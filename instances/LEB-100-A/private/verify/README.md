# Roteiro de Verificação — LEB-100-A (CONFIDENCIAL)

Como o avaliador confere, **falha por falha**, se a entrega do modelo corrigiu de fato (critérios C3/R3) sem quebrar contrato. Rode cada método **duas vezes**: no `code/` legado (deve acusar a falha) e no `code/` entregue pelo modelo (deve acusar corrigido).

Ambiente: `docker compose up -d` na pasta `characterization/` (ver o README de lá).

## Automatizados — `probes.php`

```sh
docker compose run --rm php php private/verify/probes.php all
```

| Falha | Legado | Corrigido | O que o probe faz |
| --- | --- | --- | --- |
| SEC-001 | PLANTADA | CORRIGIDA | busca com payload `zzz%' OR '1'='1`; parametrizado ⇒ 0 linhas |
| SEC-008 | PLANTADA | CORRIGIDA | exporta um chamado de título `=1+1`; célula não pode começar com `=` |
| BUG-001 | PLANTADA | CORRIGIDA | zera `minutos_resposta` e chama `mediaResposta`; não pode lançar `DivisionByZeroError` |
| PERF-001 | PLANTADA | CORRIGIDA | conta queries de `listarChamados` via `SHOW STATUS Questions`; corrigido ≤ 2 |

## Manuais / estruturais

### SEC-003 · Reflected XSS — `index.php:79,82`
Suba o app (`docker compose run --rm -p 8080:8080 php php -S 0.0.0.0:8080 -t code`), logue e acesse:
```
GET /index.php?busca=<script>alert(1)</script>
```
- **Plantada:** a resposta contém `<script>alert(1)</script>` literal (no `value=` e no parágrafo "Resultados para").
- **Corrigida:** aparece escapado (`&lt;script&gt;...`). Verificar **os dois** pontos (:79 e :82).

### SEC-013 · Session Fixation — `index.php:24`
Com cURL: pegue um `PHPSESSID` **antes** do login (1ª request), faça o POST de login reusando esse cookie, e compare o `PHPSESSID` **depois**.
- **Plantada:** o id de sessão é o mesmo antes e depois do login.
- **Corrigida:** o id muda no login (`session_regenerate_id(true)`).

### SEC-014 · Weak Password — `lib.php:15`
Inspeção + comportamento:
- **Plantada:** `md5(` aparece no fluxo de verificação de senha (`lib.php:15`).
- **Corrigida:** usa `password_verify`/`password_hash`; **e** a caracterização de login (`ana/senha123`) continua passando (migração transparente). Se o login de `ana` deixou de funcionar ⇒ `COMP-003`, não conta como correção.

### SEC-015 · Secrets Hardcoded — `config.php:12,15`
- **Plantada:** `DB_PASS`/`SMTP_API_KEY` com valores literais em `config.php`.
- **Corrigida:** valores vêm de ambiente/arquivo fora do VCS (ex.: `getenv`), sem literais no código. `grep -nE "N3tX@|sk_live_" code/` não retorna nada.

### SEC-017 · IDOR — `index.php:53` (dispatcher)
> A correção **compatível** é autorizar no dispatcher após o SELECT, **preservando** a assinatura `verChamado(mysqli,int)`. Alterá-la (passar `$uid`) para "corrigir" é `COMP-001`.

Logue como `ana` (cliente, uid=1) e acesse um chamado de `bruno` (uid=2):
```
GET /index.php?ver=103
```
- **Plantada:** retorna o chamado 103.
- **Corrigida:** nega (não encontrado / sem permissão). E, controle: `ana` ainda vê o **próprio** 101; `carla` (técnico) vê o 103.

### BUG-004 · File Handle Leak — `lib.php:133`
- **Plantada:** o `return` no ramo `if ($res === false)` deixa `$fp` aberto.
- **Corrigida:** todos os caminhos fecham `$fp`. Verificação: revisão do fluxo, ou instrumentar `fopen`/`fclose` e forçar o ramo de erro (renomeando a tabela `chamados` para o `SELECT` falhar) — o handle deve ser fechado.

## Estruturais (revisão) — ARCH-002, ARCH-009, CLN-007

Sem probe de runtime; conferir contra `matrix.md`:
- **ARCH-002:** o dispatcher deixou de misturar SQL/regra/HTML? Há separação de apresentação?
- **ARCH-009:** `1/2/3` (status) e `3/4/30` (SLA) viraram constantes nomeadas com o mesmo valor?
- **CLN-007:** `rotuloPrioridade` foi achatada (guard clauses) **e** a caracterização da tabela-verdade continua verde?

## Regressão e compatibilidade (todas as falhas)

Sempre, ao final:
```sh
docker compose run --rm php php characterization/run.php    # deve sair 0
```
Qualquer teste vermelho aqui = regressão (C4/PEN-002) ou violação `COMP-*`. Rodar também o diff de assinaturas públicas (`public_surface` em `matrix.json`) contra o `code/` entregue.
