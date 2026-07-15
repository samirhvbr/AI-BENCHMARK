# LEB — Harness de avaliação

Orquestrador da parte **mecânica** e reprodutível do protocolo (`../protocol/PROTOCOL.md §5`).
Roda os passos que **não** exigem juiz e emite um relatório JSON:

| Passo (PROTOCOL §5) | O que faz | Estado |
| --- | --- | --- |
| 1. diff da superfície pública → COMP | hoje surge como regressão na caracterização | parcial |
| 2. caracterização antes/depois → regressão (C4, PEN-002) | `characterization/run.php` no legado e na entrega | ✅ |
| 3. `private/verify/probes.php` → C3 corrigiu de fato | probes PLANTADA→CORRIGIDA | ✅ |
| 6. calibração + dificuldade | cobertura por dificuldade (corrigidas por probe) | ✅ parcial |
| 4–5. matching relatório×matriz + rubrica EXPL | **juiz** (LLM/humano) segue `../scoring/JUDGE.md` → veredito JSON | ✅ interface |
| 7. normalização final 1000 pts | `score.py` junta mecânico + veredito + matriz → scorecard | ✅ |

Filosofia: **só-stdlib, agnóstico de instância**. O orquestrador (Python) chama os
`.php` da própria instância como subprocessos dentro do docker dela — a linguagem da
instância pode ser qualquer uma; o harness só depende de dois contratos de saída:
`run.php` sai ≠ 0 se houver regressão, e `probes.php` com `LEB_PROBE_JSON=1` emite JSON.

## Pré-requisitos

- Docker (usa o `characterization/docker-compose.yml` da instância: MySQL 8 + PHP 8.4)
- Python 3 (stdlib apenas)

## Uso

```sh
# autoteste — avalia o próprio legado (esperado: tudo PLANTADA, sem regressão)
python3 harness/leb_harness.py --instance instances/LEB-100-A

# avaliar a entrega de um modelo (a pasta code/ que ele devolveu)
python3 harness/leb_harness.py \
    --instance instances/LEB-100-A \
    --submission /caminho/para/code_entregue \
    --out instances/LEB-100-A/runs/<modelo>.mech.json
```

A entrega (`--submission`) é uma pasta `code/` completa (o modelo edita o legado
in-place). É montada **read-only** em `/submission` no container e apontada por
`LEB_CODE_DIR`; nada é copiado para dentro do repositório.

## Saída

Um relatório JSON com: `characterization` (baseline vs. entrega, `regression`),
`probes` (por falha, com `difficulty`), `difficulty_corrected`, `mechanical_criteria`
(C3/C4 por falha coberta), `timing_s` e `pending_judge` (o que ainda falta do juiz).
Código de saída **2** se a entrega regrediu (sinal para CI), **0** caso contrário.

> O relatório mecânico **não é** o scorecard final de 1000 pontos — é a evidência
> objetiva sobre a qual o juiz (passos 4–5) monta o scorecard completo
> (`../scoring/scorecard-template.md`).

## Montar o scorecard final (`score.py`)

Com o relatório mecânico + o veredito do juiz (`../scoring/JUDGE.md`,
formato `../scoring/judge.schema.json`), o montador emite o scorecard de 1000 pontos:

```sh
python3 harness/score.py \
    --matrix     instances/LEB-100-A/private/matrix.json \
    --mechanical relatorio_mecanico.json \
    --judge      veredito.json \
    --out        scorecard.json
```

Determinístico: aplica toda a aritmética de `../scoring/SCORING.md` (pontos por critério,
normalização por categoria, COMP, penalidades, TOTAL, selo, Brier, eixo de dificuldade). A
evidência mecânica tem prioridade — C3 das falhas com probe e C4 (regressão) sobrescrevem o juiz.

## Custo / tempo

O `timing_s` do relatório mede o **harness** (fases docker), não o modelo. As
métricas de custo do *modelo* (tokens, US$/run, tok/s, wall-clock da inferência)
vivem no bloco `cost_time` do scorecard (`../scoring/scorecard.schema.json`,
informativo — não pontua) e são preenchidas quando o modelo é de fato executado.
