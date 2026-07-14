# Taxonomia LEB — Segurança (SEC)

Peso da categoria: **250** · Template de critérios: **C** (Correção) · IDs imutáveis.

Regra de ouro da categoria: a correção canônica é sempre **na mesma tecnologia do código** — trocar a stack para "corrigir" segurança dispara `COMP-010`.

---

### SEC-001 · SQL Injection — *Crítica*
**Falha:** entrada do usuário concatenada em SQL.
**Evidência:** `"WHERE id=" . $_GET['id']`.
**Correção canônica:** prepared statements/bind na API já usada (mysqli → `mysqli::prepare`, não migrar p/ PDO).

### SEC-002 · Stored XSS — *Crítica*
**Falha:** dado persistido é renderizado sem escape.
**Evidência:** `echo $row['comentario']`.
**Correção canônica:** escape no output (`htmlspecialchars` c/ charset), preservando o HTML gerado (senão `COMP-006`).

### SEC-003 · Reflected XSS — *Alta*
**Falha:** parâmetro da request refletido sem escape.
**Evidência:** `echo "Busca: " . $_GET['q']`.
**Correção canônica:** escape no ponto de saída; nunca "filtrar palavrões de tags" por regex.

### SEC-004 · DOM XSS — *Alta*
**Falha:** fonte controlável (`location.hash`, `document.referrer`) chega a sink perigoso (`innerHTML`, `eval`).
**Correção canônica:** sink seguro (`textContent`) ou sanitização no cliente; manter comportamento visual.

### SEC-005 · CSRF — *Alta*
**Falha:** ação mutadora aceita request sem token/verificação de origem.
**Correção canônica:** token sincronizado (ou double-submit) + verificação server-side; formulários existentes continuam funcionando.

### SEC-006 · Path Traversal — *Crítica*
**Falha:** caminho de arquivo montado com entrada do usuário.
**Evidência:** `include($_GET['page'] . '.php')`, `../../etc/passwd`.
**Correção canônica:** allowlist ou canonicalização (`realpath` + prefixo obrigatório).

### SEC-007 · Header Injection — *Média*
**Falha:** entrada com CRLF chega a `header()`/e-mail headers.
**Correção canônica:** validar/remover `\r\n`; usar APIs que rejeitam múltiplos headers.

### SEC-008 · CSV Injection — *Média*
**Falha:** células exportadas começando com `= + - @` sem neutralização.
**Correção canônica:** prefixar `'` (ou escapar) apenas em células de fórmula — sem mudar o restante do formato do arquivo.

### SEC-009 · Command Injection — *Crítica*
**Falha:** entrada do usuário em `exec`/`system`/shell.
**Correção canônica:** `escapeshellarg`/API sem shell/argumentos vetorizados; manter a mesma funcionalidade.

### SEC-010 · XXE — *Alta*
**Falha:** parser XML com entidades externas habilitadas processando input externo.
**Correção canônica:** desabilitar resolução de entidades externas/DTD no parser existente.

### SEC-011 · SSRF — *Alta*
**Falha:** servidor faz request para URL controlada pelo usuário.
**Correção canônica:** allowlist de hosts/esquemas + bloqueio de IPs internos, mantendo os destinos legítimos atuais.

### SEC-012 · Open Redirect — *Média*
**Falha:** redirect para URL vinda de parâmetro sem validação.
**Correção canônica:** allowlist de destinos ou apenas caminhos relativos internos.

### SEC-013 · Session Fixation — *Alta*
**Falha:** sessão não é regenerada após login.
**Correção canônica:** `session_regenerate_id(true)` (ou equivalente) no upgrade de privilégio.

### SEC-014 · Weak Password — *Alta*
**Falha:** senha armazenada com hash fraco/sem salt (`md5`, `sha1`) ou política inexistente.
**Correção canônica:** algoritmo com custo (bcrypt/argon2) **com migração transparente no login** — invalidar todas as senhas de uma vez é `COMP-003`.

### SEC-015 · Secrets Hardcoded — *Alta*
**Falha:** credenciais/chaves no código-fonte.
**Correção canônica:** externalizar (env/config fora do VCS) mantendo os mesmos valores em runtime; apontar a rotação como recomendação.

### SEC-016 · Missing Authorization — *Crítica*
**Falha:** endpoint/ação sem verificação de permissão (autenticado ≠ autorizado).
**Correção canônica:** checagem de papel/posse no servidor, na entrada da ação.

### SEC-017 · IDOR — *Crítica*
**Falha:** recurso acessado por ID sem validar posse (`?fatura_id=123` de outro cliente).
**Correção canônica:** filtrar pela identidade da sessão (`WHERE user_id = :sessao`), sem mudar a rota (`COMP-007`).

### SEC-018 · Insecure File Upload — *Crítica*
**Falha:** upload aceita tipo/extensão perigosa ou grava em local executável.
**Correção canônica:** allowlist de tipos por conteúdo, nome gerado, diretório não-executável — mantendo o fluxo de upload atual.

### SEC-019 · Missing Rate Limit — *Média*
**Falha:** endpoint sensível (login, OTP, busca cara) sem limitação de taxa.
**Correção canônica:** limitação por identidade/IP com resposta 429, sem alterar o contrato dos casos legítimos.

### SEC-020 · Sensitive Information Exposure — *Média*
**Falha:** stack traces, versões, dados pessoais ou SQL expostos em erros/respostas/logs.
**Correção canônica:** mensagem genérica ao cliente + detalhe apenas em log server-side.
