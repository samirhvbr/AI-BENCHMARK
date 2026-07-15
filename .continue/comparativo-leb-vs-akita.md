# Comparativo — LEB × llm-coding-benchmark (Akita)

> **Registrado em:** 2026-07-14 · **Autor da análise:** sessão Claude Code
> **Para que serve:** capturar de vez a comparação entre o **nosso** benchmark
> (LEB / `~/x/AI-BENCHMARK`) e o do **Fabio Akita** (`~/x/llm-coding-benchmark`,
> clone limpo do repo público dele), pra não refazer a análise.
> **Plano de ação derivado disto:** ver [`retomada.md`](./retomada.md).

---

## Resumo em uma linha

Os dois medem "LLM sabe fazer engenharia de software?", mas atacam **pontas
opostas**: o do Akita mede **construção do zero** (greenfield), o LEB mede
**manutenção de legado sem quebrar** (compatibilidade). São **complementares**,
não concorrentes — o LEB é, na prática, o benchmark que falta no espaço do Akita.

---

## O que é cada um

### `llm-coding-benchmark` (Akita) — greenfield
- **Autor / natureza:** Fabio Akita (`akitaonrails`), repo **público**, base de dados
  de uma série de posts no blog. Clonado limpo no nosso `~/x` só pra estudo.
- **Pergunta central:** quão bem o modelo **constrói do zero, autônomo**, UM app
  Rails fixo (SPA de chat com o gem RubyLLM)? O sinal real que ele caça:
  **o modelo alucina a API da biblioteca e escreve testes que mockam a própria
  alucinação** (passam verdes e mentem sobre a corretude).
- **Tese:** *"completude estrutural NÃO prediz correção em runtime"* — 9/9
  artefatos + 37 métodos de teste e ainda assim chama uma API inexistente do gem.
- **Unidade de teste:** **1 tarefa fixa × N modelos**. A "matriz" dele é de
  **modelos e configurações de harness**, não de exercícios.

### `AI-BENCHMARK` / LEB (nosso) — legado
- **Autor / natureza:** Samir, repo **público** (por design), **spec 1.1.0** +
  1 instância-referência. ~2 dias de git, 1 autor.
- **Pergunta central:** quão bem o modelo **evolui código legado existente sem
  quebrar compatibilidade**? Acha as falhas plantadas, corrige **na mesma stack**,
  explica, não regride, não cai nas iscas.
- **Tese:** *"reescrever tudo do zero não é engenharia, é fuga"* — penalizado
  explicitamente (PEN-003 −25).
- **Unidade de teste:** **N instâncias × 5 níveis**, cada uma com **falhas
  plantadas** e um gabarito oculto (matriz-hash).

---

## O insight central: são espelhos

| | Akita | LEB |
|---|---|---|
| Direção | Criar | Manter |
| Ponto de partida | Folha em branco | Código legado que já roda |
| "Erro capital" | Alucinar a API / testes mentirosos | Reescrever do zero / quebrar contrato |
| O que é difícil | Recall binário da API + disciplina de teste/Docker | Achar falha sutil **e** corrigir sem regressão |

---

## Tabela comparativa completa

| Dimensão | Akita (greenfield) | LEB (legado) |
|---|---|---|
| **Paradigma** | Construir do zero | Evoluir sem quebrar |
| **Unidade de teste** | 1 tarefa fixa × N modelos | N instâncias × 5 níveis, com falhas plantadas |
| **Desafio de referência** | App Rails 8.1 + RubyLLM (chat SPA) | LEB-100-A: painel PHP8/mysqli de um ISP fictício |
| **Stack do desafio** | Ruby 4.0 / Rails 8.1 / Hotwire / Minitest / Docker | PHP 8 + mysqli + MySQL 8 |
| **Modo de execução** | Agêntico autônomo longo (15–60 min, 100+ turnos) | Modo S (turno único) ou A (agêntico) |
| **Scoring** | Rubrica holística **0–100, 8 dimensões ponderadas** | Scorecard aditivo **1000 pts, 7 categorias** |
| **Natureza do scoring** | **Subjetivo** — auditado à mão (LLM/humano) | **Objetivo** — mecânico + pouco julgamento |
| **Gabarito** | Ler o código de integração à mão + scanner estrutural | **Matriz-hash (SHA-256) + probes + caracterização** |
| **Anti-gaming** | "Não confie no verde; leia o código" | **Iscas + enunciado neutro + caracterização + expiração** |
| **Calibração** | ❌ não tem | ✅ Brier + diagrama de confiabilidade |
| **Compatibilidade** | Não é foco (é greenfield) | **É o coração** — COMP (conduta) + penalidades |
| **Custo/tempo** | ✅ US$/run, tokens, tok/s, wall-clock | ❌ ausente |
| **Harness** | ✅ **completo** (Python stdlib + runners + gates + validação de runtime Docker/Chromium) | ❌ **não implementado** (peças prontas, falta orquestrador) |
| **Runs feitos** | ✅ **~50 modelos** rankeados | ❌ **zero** |
| **Maturidade** | Meses, PRs/issues da comunidade | 2 dias, 1 autor |
| **Público / branch** | Sim / `master` | Sim / `master` |

---

## Detalhe do scoring de cada um

### Akita — rubrica holística 0–100 (8 dimensões)
| Dimensão | Peso |
|---|---:|
| Deliverable completeness | 25 |
| RubyLLM correctness (vs. source do gem 1.14.1) | 20 |
| Test quality (mocka a API real? cobre erros?) | 15 |
| Error handling | 10 |
| Persistence / multi-turn | 10 |
| Hotwire/Turbo/Stimulus | 10 |
| Architecture | 5 |
| Production readiness | 5 |

Tiers: **A** 80–100 · **B** 60–79 · **C** 40–59 · **D** <40. Coluna binária extra
"RubyLLM OK" (API correta vs. alucinada). **Score final auditado à mão.**
Exemplos de ranking: Opus 4.7 = 97/A · GPT 5.4 xHigh = 95/A · … · Grok 4.20 = 25/D ·
GPT-OSS 20B = 11/D.

### LEB — scorecard aditivo 1000 pts (7 categorias)
| Categoria | Peso |
|---|---:|
| Segurança (SEC) | 250 |
| Arquitetura (ARCH) | 200 |
| Bugs (BUG) | 150 |
| Performance (PERF) | 150 |
| Clean Code (CLN) | 100 |
| Compatibilidade (COMP, só desconta) | 100 |
| Explicação Técnica (EXPL, às cegas) | 50 |

Cada falha pontua por **critérios cumulativos** (achou/explicou/corrigiu/sem-regressão/
compatível). Normalização garante que toda instância vale **exatamente 1000**, não
importa quantas falhas. Penalidades descontam do total (bug novo, regressão, rewrite,
falso-positivo contra isca). Selos: **Platinum** 900+ · Gold 750+ · Silver 600+ ·
Bronze 400+ · Reprovada <400.
Métricas **informativas (não pontuam):** calibração (Brier) + eixo de dificuldade.

---

## Onde o LEB já está NA FRENTE

1. **Objetividade.** O Akita depende de auditoria humana da rubrica 0–100 (ele
   mesmo admite: *"o único sinal confiável é ler o código à mão"*). O nosso
   scorecard é **aditivo e mecânico**, com matriz-gabarito hasheada —
   reprodutível por terceiros sem julgamento subjetivo na maior parte.
2. **Anti-contaminação estruturado.** Iscas obrigatórias (10–20%), enunciado
   neutro que impede *fishing* de categoria, e **expiração por hash** quando a
   instância vira provável corpus de treino. O Akita não tem defesa contra
   contaminação — a tarefa dele é fixa e pública há meses.
3. **Calibração (Brier).** Medir *"o modelo sabe o que não sabe?"* é um eixo que
   o Akita não mede.
4. **Compatibilidade como valor de 1ª classe.** A pegadinha `mysqli→PDO` (dispara
   COMP-010 −20 **e** COMP-001 por assinatura pública) não tem equivalente no
   mundo greenfield. É o teste central de maturidade do LEB.

---

## Onde o LEB está ATRÁS — os 3 pontos a "roubar"

O ponto forte do Akita é o nosso ponto fraco: ele **rodou de verdade**.

### 1. Harness de avaliação
- **Akita:** harness completo em **Python só-stdlib** (`scripts/benchmark/runner.py`
  ~50KB), com **gates anti-zumbi** (timeout global, no-progress kill, tok/s mínimo)
  e **validação de runtime** (`docker build` + `compose up` + probe headless de
  browser via CDP em `scripts/browser_probe.mjs`).
- **LEB:** temos as **peças** (probes.php, caracterização 22/22, docker-compose,
  pipeline de 7 passos desenhado no PROTOCOL.md, scorecard.schema.json), mas
  **falta o orquestrador** que amarra tudo e cospe o scorecard JSON. É o
  pendente #1 declarado no nosso próprio README.

### 2. Custo / tempo
- **Akita:** o `result.json` já traz `elapsed_seconds`, `phases[]`, `tokens`
  {input/output/cache/total}, `output_tokens_per_second`; US$/run é derivado
  (tokens × tabela de preços — `docs/cost_analysis.md`, `docs/pricing.md`).
- **LEB:** **ausente.** Barato importa tanto quanto correto. Entra como métrica
  **informativa que não pontua** — mesmo tratamento de calibração/dificuldade.

### 3. Runs de referência
- **Akita:** ~50 modelos rankeados, com achados ricos (ex.: destilação de
  "reasoning" do Claude NÃO transfere recall de API de biblioteca; "o harness
  importa" — mesmo modelo, harness diferente → corretude diferente).
- **LEB:** **zero.** Depende do harness (#1). Regra do protocolo: 1 run oficial =
  **3 execuções**, nota = **mediana do total**.

---

## Dois pontos que reforçam a estratégia do LEB

- **A tese empírica do Akita valida a nossa por construção.** O achado dele
  ("completude estrutural ≠ correção em runtime": 9/9 artefatos + testes que
  mockam uma API inexistente) é *exatamente* o tipo de mentira que a nossa
  **caracterização + probes** pegam automaticamente. Vale citar o achado dele
  como motivação no README do LEB.
- **Detalhe operacional (sem risco pra nós):** o harness do Akita roda o Claude
  Code headless com `claude -p --dangerously-skip-permissions` como um dos
  runners. É legítimo (é assim que se automatiza CLI de agente em jaula) — só
  explica por que um scanner do ambiente sinalizou o padrão ao ler o repo dele.

---

## Próximo passo

O plano concreto pra fechar as 3 lacunas (ordem, decisões em aberto, paths de
referência) está em [`retomada.md`](./retomada.md).
