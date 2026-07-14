# LEB — Modelo de Pontuação

Normativo. Complementa `../SPEC.md` §3–§6.

---

## 1. Severidades e valor-base por falha

Cada falha plantada recebe uma severidade na matriz da instância, que define seu valor-base:

### Template C — Correção (SEC, PERF, BUG)

| Critério | Crítica (10) | Alta (8) | Média (6) | Baixa (5) |
| --- | ---: | ---: | ---: | ---: |
| C1 Encontrou | 2 | 2 | 1 | 1 |
| C2 Explicou corretamente | 2 | 1 | 1 | 1 |
| C3 Corrigiu | 3 | 3 | 2 | 1 |
| C4 Não introduziu regressão | 2 | 1 | 1 | 1 |
| C5 Manteve compatibilidade | 1 | 1 | 1 | 1 |

### Template R — Refatoração (ARCH, CLN)

| Critério | Crítica (12) | Alta (10) | Média (8) | Baixa (6) |
| --- | ---: | ---: | ---: | ---: |
| R1 Identificou | 3 | 2 | 2 | 1 |
| R2 Explicou | 2 | 2 | 1 | 1 |
| R3 Refatorou | 5 | 4 | 4 | 3 |
| R4 Compatível | 2 | 2 | 1 | 1 |

---

## 2. Definição operacional de cada critério

**C1/R1 — Encontrou/Identificou.** O relatório do modelo aponta a falha certa no lugar certo (mesmo ID lógico + arquivo; tolerância de ±10 linhas ou mesma função). Nomear a classe da falha errada mas apontar o trecho certo = metade dos pontos do critério (arredonda p/ baixo).

**C2/R2 — Explicou.** A explicação descreve o mecanismo real da falha (por que é explorável / por que degrada / por que quebra), não uma descrição genérica da categoria. Explicação incorreta ou boilerplate = 0.

**C3 — Corrigiu.** A correção elimina a falha no ponto plantado **e** resiste ao teste de verificação da instância (exploit deixa de funcionar, query deixa de ser N+1, etc.). Correção parcial (mitiga mas não elimina) = metade, arredonda p/ baixo.

**R3 — Refatorou.** A estrutura-alvo declarada na matriz foi alcançada (ex.: God Object decomposto com responsabilidades separadas). Refatoração cosmética (renomear, mover sem separar responsabilidade) = 0. Parcial genuíno = metade.

**C4 — Não introduziu regressão.** Os testes de caracterização que cobrem o entorno da correção continuam verdes. Binário: qualquer teste do entorno quebrado = 0 no critério.

**C5/R4 — Manteve compatibilidade.** A correção/refatoração não disparou nenhuma violação `COMP-*` *atribuível a ela*. Binário. (A violação COMP em si também desconta da categoria COMP — o fato é um só, mas fere dois contratos distintos: o item deixa de ser "correção completa" e a conduta global fica manchada. Isso é intencional e documentado.)

---

## 3. Falhas não plantadas (iscas) e falsos positivos

- Reportar falha marcada "Existe: Não" na matriz → **PEN-004** (−5 cada, teto −25 por run).
- Reportar falha **real porém fora da matriz** (achado legítimo que os autores não plantaram): 0 pontos, 0 penalidade — e DEVERIA ser registrada para a próxima versão da instância.
- O avaliador decide "isca vs achado legítimo" **antes** de olhar o scorecard, para não enviesar.

---

## 4. Normalização por categoria

Os valores-base (§1) são pesos relativos **dentro** da categoria. A nota da categoria é:

```text
nota_categoria = round( (pontos_obtidos / pontos_possíveis_na_matriz) × peso_categoria )
```

Exemplo: instância LEB-200 planta falhas SEC somando 100 pontos-base; o modelo obtém 84 → `84/100 × 250 = 210/250`.

Isso garante que **toda instância vale exatamente 1000**, independentemente de quantas falhas foram plantadas, e mantém scorecards comparáveis entre instâncias do mesmo nível.

## 5. Compatibilidade (categoria de conduta)

```text
nota_COMP = max(0, 100 − Σ descontos COMP-* )
```

Cada violação conta **por ocorrência** (mudar a assinatura de 3 métodos públicos = 3 × −30, piso 0). Tabela de descontos em `../SPEC.md §6.1` e definições em `../taxonomy/COMP.md`.

## 6. Explicação Técnica (50 pontos)

Rubrica de 5 dimensões × 10 pontos, avaliada sobre o relatório final do modelo:

| Dimensão | 0 | 4 | 7 | 10 |
| --- | --- | --- | --- | --- |
| Clareza | incompreensível | desorganizado | legível | impecável, direto |
| Precisão técnica | erros graves | imprecisões | correto | correto e profundo |
| Causa-raiz | só sintomas | superficial | causa real | causa + mecanismo |
| Priorização | nenhuma | lista plana | por severidade | severidade × esforço × risco |
| Trade-offs | ausentes | menção vaga | reais | alternativas comparadas |

Avaliação por humano ou LLM-juíza **com esta rubrica e âncoras**; o juiz NÃO vê o scorecard das outras categorias.

## 7. Total

```text
TOTAL = Σ notas_categorias + nota_COMP + nota_EXPL − Σ PEN-*      (piso 0, teto 1000)
```

## 8. Classificação final (informativa)

| Faixa | Selo |
| ---: | --- |
| 900–1000 | **LEB Platinum** — pronta para legado crítico |
| 750–899 | **LEB Gold** — engenharia sólida |
| 600–749 | **LEB Silver** — útil com supervisão |
| 400–599 | **LEB Bronze** — requer revisão integral |
| < 400 | **Reprovada** — risco ao sistema |
