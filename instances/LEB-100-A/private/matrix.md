# Matriz Oficial de Falhas — LEB-100-A v1.0

> **CONFIDENCIAL.** Gabarito. Nunca entregar junto de `code/`. Publicar apenas o SHA-256 de `matrix.json` no lançamento (ver `../../../matrix/MATRIX.md §4`).

| Spec LEB | Instância | Nível | Versão | Stack |
| --- | --- | --- | --- | --- |
| 1.0.0 | LEB-100-A | 100 | 1.0 | PHP 8 + mysqli + MySQL 8 |

## Falhas plantadas (13)

| # | ID | Sev. | Pts | Tpl | Localização | Evidência | Correção esperada |
| --- | --- | --- | ---: | --- | --- | --- | --- |
| 1 | SEC-001 | Crítica | 10 | C | `lib.php:82` | `"... titulo LIKE '%" . $busca . "%'"` — termo concatenado na SQL | prepared statement com bind do termo (`LIKE CONCAT('%', ?, '%')`), **em mysqli** |
| 2 | SEC-003 | Alta | 8 | C | `index.php:82` (tb. `:79`) | `echo '<p>Resultados para: ' . $busca` e `value="' . $busca . '"` | `htmlspecialchars($busca, ENT_QUOTES)` nos dois pontos de saída |
| 3 | SEC-008 | Média | 6 | C | `lib.php:140` | `titulo` gravado cru no CSV (`fputcsv`) | neutralizar células que começam com `= + - @` (prefixar `'`) sem mudar o formato |
| 4 | SEC-013 | Alta | 8 | C | `index.php:24` | login grava `$_SESSION['uid']` sem regenerar o id de sessão | `session_regenerate_id(true)` após autenticar |
| 5 | SEC-014 | Alta | 8 | C | `lib.php:15` | `md5($senha)` sem salt | `password_hash`/`password_verify` com **migração transparente no login** (aceita md5 legado e re-hash) |
| 6 | SEC-015 | Alta | 8 | C | `config.php:12` (tb. `:15`) | `DB_PASS` e `SMTP_API_KEY` no código | externalizar via env/config fora do VCS, mesmos valores em runtime; recomendar rotação |
| 7 | SEC-017 | Crítica | 10 | C | `index.php:53` (lê em `lib.php:100`) | dispatcher exibe o chamado sem checar se o solicitante é dono/técnico — IDOR via `?ver=id` | autorizar no dispatcher após carregar: negar se `papel=cliente` e `usuario_id != uid`; técnico vê todos. **Preservar** a assinatura `verChamado(mysqli,int)` |
| 8 | BUG-001 | Alta | 8 | C | `lib.php:116` | `return $soma / $qtd` com `$qtd` possivelmente 0 | tratar conjunto vazio (retornar `0.0`) antes de dividir |
| 9 | BUG-004 | Média | 6 | C | `lib.php:133` | `if ($res === false) return;` deixa `$fp` aberto | fechar `$fp` em todos os caminhos (inclusive o de erro) |
| 10 | PERF-001 | Alta | 8 | C | `lib.php:89` (tb. `:137`) | `tecnicoNome()` chamada dentro do loop → 1 query por chamado | carregar técnicos em lote (JOIN ou `WHERE id IN (...)`) |
| 11 | ARCH-002 | Alta | 10 | R | `index.php:41-97` | dispatcher faz roteamento + autorização + regra + montagem de HTML | separar apresentação (templates) e orquestração; controller magro |
| 12 | ARCH-009 | Baixa | 6 | R | `lib.php:28` | status `1/2/3` e prioridade `3/4/30` soltos | constantes nomeadas com os mesmos valores |
| 13 | CLN-007 | Média | 8 | R | `lib.php:43` | 4 níveis de `if` aninhado em `rotuloPrioridade` | guard clauses / retorno antecipado, mesma tabela-verdade |

**Brutos por categoria:** SEC 58 · BUG 14 · PERF 8 · ARCH 16 · CLN 8. (Normalizar para 250/150/150/200/100 — `SCORING.md §4`.)

## Iscas (`exists: false`) — reportar dispara PEN-004 (−5, teto −25)

| ID | Por que é plausível, mas NÃO existe |
| --- | --- |
| SEC-009 (Command Injection) | o sistema "exporta" e fala em integração de e-mail, mas não há `exec`/`system`/`shell_exec`/`proc_open` em lugar nenhum |
| PERF-006 (Sem índice) | `schema.sql` já cria `idx_status`, `idx_usuario`, `idx_criado_em`. A busca é `titulo LIKE '%termo%'`, que um índice B-tree **não** acelera — "adicionar índice em `titulo`" é engano, não correção |

## Notas de avaliação

- **Correções alternativas aceitas:**
  - SEC-001: qualquer parametrização real (mysqli prepared, `?` com bind). Escapar manualmente com `real_escape_string` dentro do `LIKE` **sem** tratar `%`/`_` é correção parcial (C3 = metade).
  - SEC-014: aceitar bcrypt **ou** argon2; o essencial é a migração transparente. Invalidar todas as senhas de uma vez = `COMP-003` (quebra a caracterização de login).
  - SEC-017: filtrar na query **ou** checar posse após o SELECT e negar; ambos válidos desde que técnico continue vendo tudo.
  - BUG-001: retornar `0.0`, `null` documentado, ou lançar exceção tratada — a matriz aceita `0.0` como canônico; qualquer um que não seja divisão por zero pontua C3.
- **Achados legítimos NÃO plantados (não penalizar; candidatos à v1.1):** `SELECT *` em `lib.php:80/132` (PERF-002), ausência de camada de repositório com SQL cru espalhado (ARCH-003), ausência de CSRF no POST de login (SEC-005). Se o modelo os reportar: 0 pontos, 0 penalidade, registrar.
- **Pegadinha de compatibilidade (a mais importante):** migrar `mysqli → PDO` para "corrigir" a SQLi dispara **COMP-010 (−20)** e, como muda a assinatura de todas as funções públicas de `lib.php` (que recebem `mysqli`), também **COMP-001 (−30 por função)**. A correção certa vive dentro do mysqli.
- **Sobreposição PERF-001:** o mesmo `tecnicoNome()` causa N+1 na listagem (`:89`) e no export (`:137`). Uma correção em lote resolve ambos — atribuir **uma** vez.
- **Ordem dos verifies:** rodar `verify/` sobre banco recém-semeado (`schema.sql` + `seed.sql`); `bug-001` usa dataset alternativo (sem `minutos_resposta`).
