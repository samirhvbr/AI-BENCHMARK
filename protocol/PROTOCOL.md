# LEB — Protocolo de Execução

Normativo. Define como rodar o benchmark de forma justa e reproduzível. Resultado que não declara o protocolo usado **não é** um resultado LEB.

---

## 1. O que o modelo recebe

1. O código da instância (`code/`), o manifesto de superfície pública (`manifest.md`) e **nada de `private/`**.
2. O **enunciado canônico** (§2) — sempre o mesmo texto, sem adaptações por modelo.
3. Nos níveis 300+, acesso de leitura ao repositório via ferramentas (listar/ler arquivos), registrado em log.

O modelo **NÃO** recebe: a matriz, os testes de `private/verify/`, dicas de categoria ("procure SQL injection"), nem contagem de falhas.

## 2. Enunciado canônico

O texto abaixo é fixo (traduzível, mas não parafraseável). Ele é deliberadamente **neutro**: menciona os eixos de avaliação sem apontar falha alguma.

> Você é responsável pela manutenção deste sistema legado em produção. Ele possui consumidores externos que dependem do comportamento atual, conforme o manifesto de superfície pública anexo.
>
> Sua tarefa:
> 1. Analise o código e reporte os problemas que encontrar (segurança, arquitetura, bugs, performance, qualidade), explicando cada um, sua severidade e uma **confiança de 0 a 100** de que o problema é real.
> 2. Corrija o que julgar que deve ser corrigido, preservando a compatibilidade declarada no manifesto.
> 3. Entregue: (a) o relatório técnico; (b) o código alterado (diff ou arquivos completos); (c) a justificativa das decisões, incluindo o que você decidiu NÃO mudar e por quê.
>
> Não reescreva o sistema. Evolua-o.

A confiança por achado (item 1) é **neutra** — não revela nada da matriz — e alimenta a métrica de calibração (`../scoring/SCORING.md §9.1`), que **não** afeta os 1000 pontos.

## 3. Modos de execução

| Modo | Descrição | Uso |
| --- | --- | --- |
| **S** (single-turn) | 1 prompt → 1 resposta | LEB-100/200; mede capacidade bruta |
| **A** (agêntico) | multi-turno com ferramentas de leitura/execução, orçamento de N turnos declarado | LEB-300+; mede engenharia de verdade |

Parâmetros obrigatórios do run: modelo + versão exata, temperatura (oficial: a default do provedor, registrada), modo S/A, orçamento de turnos/tokens, data, instância + versão + hash da matriz.

## 4. Reprodutibilidade

1. Run oficial = **3 execuções independentes**; o scorecard oficial é a **mediana do TOTAL** (registrando as 3).
2. Logs completos (prompts, respostas, chamadas de ferramenta) arquivados junto do resultado.
3. Nenhum retry seletivo: descartar uma execução ruim e rodar de novo invalida o run.

## 5. Pipeline de avaliação

```text
entrega do modelo
   │
   ├─ 1. diff da superfície pública ──────────► violações COMP-* (mecânico)
   ├─ 2. testes de caracterização (antes/depois) ► C4 regressão, PEN-002 (mecânico)
   ├─ 3. private/verify por falha ─────────────► C3/R3 corrigiu de fato (mecânico)
   ├─ 4. matching relatório × matriz ──────────► C1/C2, confiança por achado, iscas → PEN-004 (avaliador)
   ├─ 5. rubrica EXPL (juiz às cegas) ─────────► 0–50
   ├─ 6. calibração + dificuldade ─────────────► Brier, discovery_index (mecânico, informativo — SCORING §9)
   └─ 7. cálculo (SCORING.md) ─────────────────► scorecard .md + .json
```

O avaliador humano (ou LLM-juíza com rubrica) só atua nos passos 4–5; todo o resto é mecânico e re-executável por terceiros. O passo 6 (calibração e cobertura por dificuldade) é derivado dos passos 3–4 e **não** entra no TOTAL.

**Ferramentas:** os passos mecânicos 1–3 e 6 rodam em [`../harness/leb_harness.py`](../harness/leb_harness.py) (relatório mecânico JSON); os passos 4–5 seguem [`../scoring/JUDGE.md`](../scoring/JUDGE.md) e produzem um veredito (`../scoring/judge.schema.json`); o passo 7 é o montador [`../harness/score.py`](../harness/score.py), que junta mecânico + veredito + matriz e emite o scorecard oficial.

## 6. Anti-gaming

1. **Iscas** na matriz punem checklist recitado sem leitura (PEN-004).
2. **Enunciado neutro** impede fishing de categoria.
3. **Caracterização** pune o modelo que "conserta" reescrevendo (PEN-002/003, COMP-*).
4. Instâncias **expiram** ao virar provável corpus de treino (MATRIX.md §4).
5. O relatório precisa **explicar** (C2/R2, EXPL): acertar por sorte não escala pontos.

## 7. Publicação de resultados

Resultado publicado DEVE conter: scorecard (md+json), parâmetros do §3, hash da matriz, logs, e a versão da spec. Formato do scorecard: `../scoring/scorecard-template.md`.
