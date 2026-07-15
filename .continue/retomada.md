# Retomada — LEB vs. llm-coding-benchmark (Akita)

> **Sessão:** 2026-07-14 · **Retomar em:** 2026-07-15
> **Objetivo da retomada:** fechar as 3 lacunas do LEB que a comparação com o
> benchmark do Akita expôs — **harness**, **custo/tempo** e **runs de referência**.
> **Comparação completa (pra ler com calma):** [`comparativo-leb-vs-akita.md`](./comparativo-leb-vs-akita.md).
> **Decisão desta sessão:** documentar primeiro; o build dos 3 pontos fica pra amanhã, depois da leitura.

---

## TL;DR — onde paramos

Comparamos o **LEB** (`~/x/AI-BENCHMARK`, nosso) com o **llm-coding-benchmark**
(`~/x/llm-coding-benchmark`, clone limpo do repo público do Fabio Akita).

**Insight central:** os dois são **espelhos**.
- **Akita → greenfield:** o modelo *constrói do zero* um app Rails fixo. Mede se
  ele alucina a API da biblioteca (RubyLLM) e escreve testes que mockam a própria
  alucinação. Tese: *"completude estrutural ≠ correção em runtime"*.
- **LEB → legado:** o modelo *evolui código existente sem quebrar compat*. Tese:
  *"reescrever do zero não é engenharia, é fuga"*.

Onde o LEB já está **na frente**: scoring objetivo (1000 pts aditivos + matriz-hash),
anti-contaminação (iscas + enunciado neutro + expiração), calibração (Brier),
compatibilidade como valor de 1ª classe.

Onde o LEB está **atrás** (e é o foco de amanhã): o Akita **rodou de verdade** —
tem harness completo, métricas de custo/tempo e ~50 modelos rankeados. Nós temos
spec 1.1.0 + 1 instância validada e **zero runs**.

---

## As 3 diferenças a trabalhar

### 1. Harness de avaliação — **pendente #1, destrava tudo**

**Estado do LEB:** já temos as *peças*, falta o *orquestrador*.
- ✅ `instances/LEB-100-A/characterization/run.php` — 22 asserções de superfície pública
- ✅ `instances/LEB-100-A/characterization/docker-compose.yml` + `_bootstrap.php` — ambiente MySQL8+PHP8.4
- ✅ `instances/LEB-100-A/private/verify/probes.php` — 4 probes PLANTADA→CORRIGIDA (SEC-001, SEC-008, BUG-001, PERF-001)
- ✅ `protocol/PROTOCOL.md` §5 — pipeline de 7 passos já **desenhado**
- ✅ `scoring/scorecard.schema.json` — formato de saída já definido
- ❌ **falta o código** que amarra tudo e cospe o scorecard JSON.

**O harness precisa fazer** (espelhando o pipeline de 7 passos do PROTOCOL.md):
1. aplicar a entrega do modelo (apontar `LEB_CODE_DIR`)
2. rodar caracterização **antes/depois** → detectar regressão (passos 1–2, mecânico)
3. rodar `probes.php` → PLANTADA vs CORRIGIDA por falha (passo 3, mecânico)
4. matching relatório-do-modelo × matriz → achou/explicou/iscas (passo 4, precisa juiz)
5. rubrica EXPL às cegas (passo 5, precisa juiz)
6. calibração (Brier) + eixo de dificuldade (passo 6, mecânico)
7. calcular scorecard normalizado (1000 pts) → emitir JSON válido no schema

**Referência de arquitetura no repo do Akita** (`~/x/llm-coding-benchmark/`):
- `scripts/run_benchmark.py` — entrypoint fino (CLI → config → delega)
- `scripts/benchmark/runner.py` (~50KB) — process mgmt, execução de fases, **gates anti-zumbi** (timeout global 90min, no-progress kill 6min, tok/s mínimo)
- `scripts/benchmark/config.py` — `BenchmarkConfig`
- `scripts/analyze_results_runtime.py` — **validação de runtime** (boot + `docker build` + `compose up` + browser headless) ← o padrão que queremos
- `scripts/browser_probe.mjs` — probe Chromium via CDP
- `scripts/benchmark/report.py` — gera relatório agregado
- Padrão-chave: **Python só-stdlib**, sem deps externas.

**Decisão em aberto:** linguagem do harness.
- Opção A — **Python stdlib** (igual Akita; portável, agrega runs de instâncias de qualquer stack no futuro).
- Opção B — **PHP** (a instância já é PHP; caracterização e probes já são PHP → menos context-switch).
- *Recomendação inicial:* Python stdlib para o orquestrador (agnóstico de instância),
  chamando os `.php` da instância como subprocessos. Confirmar amanhã.

### 2. Custo / tempo — barato, é só instrumentar o harness

**Estado do LEB:** ausente. **Estado do Akita:** mede tudo.

O `result.json` do Akita (ex.: `~/x/llm-coding-benchmark/results/claude_opus_4_8/result.json`)
tem o formato pronto pra espelhar:
- `elapsed_seconds`, `phases[]` (tempo por fase)
- `tokens` = { input, output, cache, total }
- `output_tokens_per_second`
- `finish_reason`, `status`
- US$/run é **derivado** (tokens × tabela de preços; ver `docs/cost_analysis.md` e `docs/pricing.md` do Akita)

**Ação:** adicionar bloco `cost_time` ao `scoring/scorecard.schema.json` como
**métrica informativa que NÃO pontua** — mesmo tratamento que já demos a
calibração e dificuldade na 1.1.0. Candidato a virar a **v1.2.0** (MINOR, retrocompat).

### 3. Runs de referência — 0 feitos (Akita tem ~50)

**Depende do #1.** Sem harness, não há run reprodutível.

- Regra do PROTOCOL.md §4: **1 run oficial = 3 execuções**, nota = **mediana do total**. Proibido retry seletivo.
- Modo sugerido pra LEB-100-A: **S** (turno único).
- Primeiros modelos a rodar (sugestão): Claude Opus 4.8, Sonnet 5, mais 1–2
  não-Anthropic pra ter contraste (o Akita mostra que não-Anthropic tende a
  falhar mais em recall de API — ver se o mesmo padrão aparece em legado).
- Saída: 1 scorecard JSON por modelo em `instances/LEB-100-A/runs/` (pasta a criar;
  decidir se entra versionada ou fica gitignored como no Akita).

---

## Ordem sugerida pra amanhã

1. **Harness** (#1) — destrava o resto. Começar pelo caminho mecânico (passos 1–3, 6–7),
   deixar o juiz LLM (passos 4–5) por último.
2. **Custo/tempo** (#2) — instrumentar enquanto se escreve o runner; +campo no schema.
3. **Runs de referência** (#3) — assim que o harness fechar o loop mecânico.

---

## Referências rápidas (paths)

**LEB (`~/x/AI-BENCHMARK/`):**
- `SPEC.md` · `protocol/PROTOCOL.md` (§5 pipeline) · `scoring/SCORING.md` + `scoring/scorecard.schema.json`
- `instances/LEB-100-A/characterization/{run.php,_bootstrap.php,docker-compose.yml}`
- `instances/LEB-100-A/private/verify/{probes.php,README.md}` · `private/matrix.{md,json}` (gabarito)
- `matrix/matrix.schema.json`

**Akita (`~/x/llm-coding-benchmark/`, só leitura — clone público de terceiro):**
- `scripts/run_benchmark.py` · `scripts/benchmark/{runner,config,backends,report}.py`
- `scripts/analyze_results_runtime.py` · `scripts/browser_probe.mjs`
- `results/claude_opus_4_8/result.json` (formato de custo/tempo)
- `docs/{audit_prompt_template,cost_analysis,pricing}.md` · `config/models.json`

---

## Perguntas em aberto (decidir amanhã)

- [x] Linguagem do harness: **Python stdlib** (escolhido) — orquestrador agnóstico que chama os `.php` da instância como subprocessos no docker dela.
- [x] Custo/tempo no scorecard como bloco informativo → **v1.2.0** (feito: bloco `cost_time` no schema/template).
- [x] Pasta `runs/` → **gitignored** (como o Akita): `instances/*/runs/` no `.gitignore`.
- [ ] Quais modelos no primeiro run de referência da LEB-100-A? (segue aberto — depende do juiz, ver abaixo).

---

## Progresso — 2026-07-15 (commit a seguir)

**Feito: harness mecânico (#1) + custo/tempo (#2) → spec 1.2.0.**

- `harness/leb_harness.py` (só-stdlib) roda os passos **mecânicos** do PROTOCOL §5:
  caracterização antes/depois (regressão, C4/PEN-002), `probes.php` (C3 corrigiu),
  eixo de dificuldade (corrigidas por probe), timing. Emite relatório JSON; sai 2 se regrediu.
- `probes.php` ganhou saída JSON (`LEB_PROBE_JSON=1`) — parsing determinístico, sem texto colorido.
- Entrega montada **read-only** em `/submission` (`-v … :ro` + `LEB_CODE_DIR`); nada copiado pro repo.
- **Validado nos dois sentidos** (docker mysql8+php8.4):
  - legado → 4 probes PLANTADA, caract 22/22, sem regressão, exit 0;
  - entrega corrigida (fixture no scratchpad, fora do repo) → 4 CORRIGIDA, caract 22/22 (sem regressão),
    exit 0, `difficulty_corrected` tudo corrigido. A pegadinha mysqli→PDO foi respeitada no fixture.
- `cost_time` (informativo, não pontua) no `scorecard.schema.json` + template + SPEC §8.3 / SCORING §9.3.
- `harness/README.md` documenta contrato de saída e uso.

**Passos de juiz + montador — FEITOS (15/07, mesma sessão):**
- `scoring/JUDGE.md` + `scoring/judge.schema.json` = interface dos passos 4–5 (matching C1/C2, iscas→PEN-004, rubrica EXPL às cegas, COMP). Juiz humano OU LLM preenche o veredito.
- `harness/score.py` (passo 7) = montador determinístico: mecânico + veredito + matriz → scorecard de 1000 pts (toda a aritmética do SCORING: pontos por critério, normalização §4, COMP §5, penalidades, TOTAL §7, selo §8, Brier §9.1, dificuldade §9.2).
- Validado de ponta a ponta contra cálculo à mão: run "forte" = **860/Gold** (SEC 216/ARCH 175/BUG 86/PERF 150/CLN 100, Brier 0.102, discovery 75); e a reescrita **mysqli→PDO = 0/Reprovada** (COMP 0 + PEN-002 −440 + C4/C5 zerados). Gate corrigido: C4/C5 só contam se C3 foi tentado.

**Falta (próxima sessão) — só o #3:**
- **Runs de referência (#3)** — rodar modelos DE VERDADE na LEB-100-A e passar o juiz. É o único passo que precisa de execução do modelo (o harness avalia entrega pronta; não faz inferência). Precisa: (a) driver que entrega `code/`+manifesto+enunciado ao modelo e coleta a entrega+relatório; (b) juiz (LLM com JUDGE.md ou humano); (c) 3 execuções → mediana (PROTOCOL §4). 1º lote sugerido: Opus 4.8, Sonnet 5, +1–2 não-Anthropic. Saída em `instances/LEB-100-A/runs/` (gitignored).
