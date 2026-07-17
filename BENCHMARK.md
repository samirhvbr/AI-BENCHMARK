# LEB — Guia de Execução (BENCHMARK)

> **Guia prático do operador.** Como rodar um modelo no LEB, passo a passo, sem
> vazar o gabarito. Complementa os documentos normativos — quando houver conflito,
> vale o normativo:
> [`SPEC.md`](SPEC.md) · [`protocol/PROTOCOL.md`](protocol/PROTOCOL.md) · [`scoring/SCORING.md`](scoring/SCORING.md) · [`scoring/JUDGE.md`](scoring/JUDGE.md).

---

## TL;DR — o ciclo em uma frase

Você entrega ao modelo **só o código legado + o manifesto público + um enunciado
neutro**; ele devolve **relatório + código alterado + justificativa**; você mede
essa entrega contra um **gabarito secreto** (a Matriz) com o harness + juiz e sai
um **scorecard de 0 a 1000**. Oficial = **mediana de 3 execuções**.

```text
   VOCÊ ENTREGA                    O MODELO DEVOLVE               VOCÊ AVALIA
┌────────────────────┐        ┌────────────────────────┐     ┌──────────────────┐
│ code/              │        │ (a) relatório técnico  │     │ harness (mecânico)│
│ manifest.md        │  ───►  │ (b) código alterado    │ ──► │ juiz (matriz+EXPL)│
│ enunciado canônico │        │ (c) justificativa      │     │ score.py → 1000   │
└────────────────────┘        └────────────────────────┘     └──────────────────┘
        PÚBLICO                                                  usa private/ (gabarito)
```

---

## A regra de ouro: o "plano" nunca vai junto

O que você chama de **"o plano"** é a **Matriz Oficial de Falhas** — o gabarito que
diz exatamente quais falhas existem, onde, com que severidade, e quais linhas são
**iscas** (falhas plausíveis que **não** existem). Ela vive em `private/` e é **o
coração do benchmark** ([SPEC §5](SPEC.md)). Se o modelo vê a matriz, o resultado
não vale nada.

Por isso a separação **público × privado** ([instances/README.md](instances/README.md)):

| Pasta / arquivo | Vai para o modelo? | Papel |
| --- | :---: | --- |
| `code/` | ✅ **SIM** | o sistema legado com as falhas plantadas |
| `manifest.md` | ✅ **SIM** | a superfície pública (o contrato que não pode quebrar) |
| enunciado canônico ([PROTOCOL §2](protocol/PROTOCOL.md)) | ✅ **SIM** | a tarefa, em texto neutro e fixo |
| `characterization/` | ❌ **NÃO** | testes de compatibilidade — só o **avaliador** usa |
| `private/matrix.json` · `matrix.md` | ❌ **NUNCA** | **o gabarito** (a matriz) |
| `private/verify/` (probes/exploits) | ❌ **NUNCA** | roteiro de verificação por falha |

O modelo também **NÃO** recebe: dicas de categoria ("procure SQL injection"), a
**contagem** de falhas, nem qualquer coisa derivada da matriz ([PROTOCOL §1](protocol/PROTOCOL.md)).

---

## O que você manda para a IA (o ponto central)

São **três coisas** — e só elas:

### 1. O pacote público (arquivos)

Monte uma pasta limpa contendo **apenas** `code/` + `manifest.md`. Nunca copie a raiz
da instância inteira (isso arrastaria `private/`).

```sh
# a partir da raiz da instância (ex.: instances/LEB-100-A/)
rm -rf /tmp/leb-pkg && mkdir -p /tmp/leb-pkg
cp -r code manifest.md /tmp/leb-pkg/

# confira que NÃO há gabarito no pacote (a saída tem de ser vazia):
find /tmp/leb-pkg \( -name 'matrix*' -o -path '*private*' -o -path '*verify*' -o -path '*characterization*' \) -print
```

Esse `/tmp/leb-pkg` é o **único** conteúdo de arquivos que o modelo pode enxergar.

### 2. O enunciado canônico (o texto da tarefa)

Fixo, **neutro** e não-parafraseável (traduzível, mas não reescrevível) — ele cita os
eixos de avaliação **sem apontar nenhuma falha** ([PROTOCOL §2](protocol/PROTOCOL.md)).
Cole exatamente isto:

> Você é responsável pela manutenção deste sistema legado em produção. Ele possui
> consumidores externos que dependem do comportamento atual, conforme o manifesto de
> superfície pública anexo.
>
> Sua tarefa:
> 1. Analise o código e reporte os problemas que encontrar (segurança, arquitetura,
>    bugs, performance, qualidade), explicando cada um, sua severidade e uma
>    **confiança de 0 a 100** de que o problema é real.
> 2. Corrija o que julgar que deve ser corrigido, preservando a compatibilidade
>    declarada no manifesto.
> 3. Entregue: (a) o relatório técnico; (b) o código alterado (diff ou arquivos
>    completos); (c) a justificativa das decisões, incluindo o que você decidiu NÃO
>    mudar e por quê.
>
> Não reescreva o sistema. Evolua-o.

Por que é neutro (e por que isso importa): mencionar "procure SQLi" ou "há 13 falhas"
seria **fishing de categoria** e vazaria a matriz — o enunciado neutro é uma das
defesas anti-gaming do padrão ([PROTOCOL §6](protocol/PROTOCOL.md)). A **confiança de
0 a 100** por achado é deliberadamente neutra (não revela nada da matriz) e só
alimenta a métrica de calibração, que **não** entra nos 1000 pontos ([SPEC §8.1](SPEC.md)).

### 3. Nada mais

Sem system prompt que injete pistas, sem "dica de amigo", sem link para este
repositório (o repo tem a taxonomia e o formato da matriz). Se o modelo tiver
ferramentas de leitura (modo agêntico), aponte-as **só** para `/tmp/leb-pkg` — nunca
para a árvore da instância, senão ele pode `ler private/`.

---

## Passo a passo completo

### Passo 0 — Pré-requisitos

- `git pull` (o repo pede isso no topo do README).
- **Docker** (a caracterização sobe MySQL 8 + PHP 8.4 via `characterization/docker-compose.yml`).
- **Python 3** (stdlib apenas; o harness não tem dependências externas).
- Escolha o modo ([PROTOCOL §3](protocol/PROTOCOL.md)):
  - **Modo S** (turno único: 1 prompt → 1 resposta) — para LEB-100/200. É o sugerido
    para a instância de referência **LEB-100-A**, que cabe numa janela de contexto.
  - **Modo A** (agêntico: multi-turno com ferramentas de leitura/execução, orçamento
    de N turnos declarado) — para LEB-300+.

### Passo 1 — Escolher a instância

Use uma instância **ativa** (não aposentada). A de referência é
[`instances/LEB-100-A`](instances/LEB-100-A/) — painel PHP legado, 13 falhas + 2 iscas,
modo S. Anote o **hash SHA-256 da matriz** do README da instância: ele entra no
resultado publicado e prova qual gabarito foi usado.

### Passo 2 — Montar o pacote público

Exatamente como na seção acima ("O que você manda para a IA" → item 1). Rode o `find`
de conferência; ele **tem** de sair vazio.

### Passo 3 — Entregar ao modelo e coletar a entrega

Entregue **pacote público + enunciado canônico**. Registre os parâmetros obrigatórios
do run ([PROTOCOL §3](protocol/PROTOCOL.md)): modelo + versão exata, temperatura (oficial
= a default do provedor, registrada), modo S/A, orçamento de turnos/tokens, data,
instância + versão + hash da matriz.

Colete a **entrega** completa:
- **(a)** relatório técnico (com a confiança 0–100 por achado);
- **(b)** o `code/` alterado — o modelo edita o legado *in-place*; você quer a pasta
  `code/` completa de volta;
- **(c)** a justificativa (inclusive o que ele decidiu **não** mudar).

Guarde os **logs completos** (prompts, respostas, chamadas de ferramenta) junto do
resultado ([PROTOCOL §4](protocol/PROTOCOL.md)).

### Passo 4 — Avaliação mecânica (harness)

Roda os passos objetivos e re-executáveis do pipeline (caracterização antes/depois,
probes de correção, cobertura por dificuldade, timing) e cospe um relatório JSON:

```sh
python3 harness/leb_harness.py \
    --instance   instances/LEB-100-A \
    --submission /caminho/para/o/code_devolvido \
    --out        instances/LEB-100-A/runs/<modelo>.mech.json
```

A entrega é montada **read-only** no container; nada é copiado para dentro do repo.
**Exit 2** = a entrega regrediu (quebrou caracterização → sinal para CI); **exit 0** =
sem regressão. Detalhes e contrato de saída em [`harness/README.md`](harness/README.md).

> Autoteste opcional (sanidade): rodar sem `--submission` avalia o próprio legado —
> esperado tudo **PLANTADA**, sem regressão.

### Passo 5 — Juiz (matriz + explicação)

Os dois passos que precisam de julgamento (humano OU LLM-juíza) seguem
[`scoring/JUDGE.md`](scoring/JUDGE.md) e produzem um **veredito JSON**
(`scoring/judge.schema.json`):
- **matching relatório × matriz** — o que o modelo achou/explicou de verdade; iscas
  reportadas viram **PEN-004**; falsos positivos idem;
- **rubrica EXPL** — qualidade da explicação, avaliada **às cegas** (0–50).

O juiz **só** atua aqui; todo o resto é mecânico. A evidência mecânica tem prioridade
(C3 das falhas com probe e C4 de regressão sobrescrevem o juiz).

### Passo 6 — Montar o scorecard (score.py)

Determinístico: junta mecânico + veredito + matriz e aplica toda a aritmética do
[`SCORING.md`](scoring/SCORING.md) (pontos por critério, normalização por categoria,
COMP, penalidades, TOTAL, selo, Brier, eixo de dificuldade):

```sh
python3 harness/score.py \
    --matrix     instances/LEB-100-A/private/matrix.json \
    --mechanical instances/LEB-100-A/runs/<modelo>.mech.json \
    --judge      instances/LEB-100-A/runs/<modelo>.judge.json \
    --out        instances/LEB-100-A/runs/<modelo>.scorecard.json
```

Saída no formato de [`scoring/scorecard-template.md`](scoring/scorecard-template.md)
(+ JSON). Inclui os blocos **informativos** que não pontuam: calibração (Brier),
cobertura por dificuldade e `cost_time` (tokens, tok/s, US$/run, wall-clock).

### Passo 7 — Mediana de 3 (resultado oficial)

Um **run oficial = 3 execuções independentes**; a nota oficial é a **mediana do
TOTAL** (registrando as 3). **Proibido retry seletivo**: descartar uma execução ruim e
rodar de novo invalida o run ([PROTOCOL §4](protocol/PROTOCOL.md)). Repita os passos
3–6 três vezes.

### Passo 8 — Publicar

O resultado publicado **DEVE** conter ([PROTOCOL §7](protocol/PROTOCOL.md)): scorecard
(`.md` + `.json`), os parâmetros do Passo 3, o **hash da matriz**, os logs e a **versão
da spec**. Comparações entre modelos só valem **na mesma instância e mesmo protocolo**.
As pastas `instances/*/runs/` são **gitignored** — os scorecards não sobem para o repo
público junto do código.

---

## Checklist anti-vazamento (antes de apertar "enviar")

- [ ] O pacote tem **só** `code/` + `manifest.md`? (rodei o `find` e saiu vazio)
- [ ] **Nenhum** arquivo `matrix*`, `private/`, `verify/` ou `characterization/` no que o modelo vê?
- [ ] O enunciado é o **canônico**, sem dica de categoria e sem contagem de falhas?
- [ ] Sem system prompt / contexto extra injetando pistas?
- [ ] Em modo agêntico: as ferramentas de leitura estão **presas ao pacote**, sem acesso à instância nem a este repo?
- [ ] Anotei modelo+versão, temperatura, modo, orçamento, data e **hash da matriz**?

---

## Armadilhas que o benchmark planta de propósito

Estas não são bugs do processo — são o teste funcionando. Não "ajude" o modelo a evitá-las.

- **mysqli → PDO** para "consertar" a SQLi: dispara `COMP-010` **e** `COMP-001` (muda a
  assinatura de todas as funções públicas de `lib.php`). A correção certa vive **dentro
  do mysqli**. No LEB-100-A essa reescrita pontua **0 / Reprovada**.
- **IDOR (SEC-017):** a correção deve autorizar **no dispatcher**, preservando
  `verChamado(mysqli, int)`. Passar `$uid` para a função é `COMP-001`.
- **Iscas** (`SEC-009`, `PERF-006` na LEB-100-A): não existem no código. Reportá-las com
  confiança alta é **PEN-004** e machuca a calibração. É exatamente o que pega o modelo
  que recita checklist sem ler o código.
- **Reescrever tudo** para "modernizar": `PEN-003` + provável enxame de `PEN-002`
  (regressões). O padrão penaliza fuga por reescrita — *evoluir* é o objetivo.

---

## Referência rápida de comandos

```sh
# 1) montar pacote público (a partir da raiz da instância)
rm -rf /tmp/leb-pkg && mkdir -p /tmp/leb-pkg && cp -r code manifest.md /tmp/leb-pkg/
find /tmp/leb-pkg \( -name 'matrix*' -o -path '*private*' -o -path '*verify*' \) -print   # deve sair VAZIO

# 2) (você entrega /tmp/leb-pkg + enunciado canônico ao modelo e coleta a entrega)

# 3) avaliação mecânica
python3 harness/leb_harness.py --instance instances/LEB-100-A \
    --submission /caminho/code_devolvido --out instances/LEB-100-A/runs/<modelo>.mech.json

# 4) juiz → veredito JSON (segue scoring/JUDGE.md)

# 5) scorecard final de 1000 pontos
python3 harness/score.py --matrix instances/LEB-100-A/private/matrix.json \
    --mechanical instances/LEB-100-A/runs/<modelo>.mech.json \
    --judge instances/LEB-100-A/runs/<modelo>.judge.json \
    --out instances/LEB-100-A/runs/<modelo>.scorecard.json

# repetir 2–5 três vezes → nota oficial = mediana do TOTAL
```
