# Taxonomia LEB — Performance (PERF)

Peso da categoria: **150** · Template de critérios: **C** (Correção) · IDs imutáveis.

Regra de ouro da categoria: otimização **com evidência e sem mudar resultado**. A correção deve produzir a mesma saída observável (mesmos dados, mesma ordem quando contratual) — só que mais barata.

---

### PERF-001 · N+1 Query — *Alta*
**Falha:** consulta dentro de loop sobre resultado de outra consulta.
**Correção canônica:** JOIN ou busca em lote (`WHERE id IN (...)`) mantendo o mesmo conjunto de dados.

### PERF-002 · SELECT * — *Baixa*
**Falha:** `SELECT *` onde poucas colunas são usadas (agravado por colunas BLOB/TEXT).
**Correção canônica:** projetar apenas as colunas consumidas — conferindo todos os consumidores do resultado antes.

### PERF-003 · Sem paginação — *Alta*
**Falha:** listagem carrega a tabela inteira para a memória/tela.
**Correção canônica:** paginação no banco (LIMIT/OFFSET ou keyset). Se a saída é contrato público, paginar sem quebrar consumidores (senão `COMP-002/008`).

### PERF-004 · Query repetida — *Média*
**Falha:** mesma consulta idêntica executada N vezes na mesma request.
**Correção canônica:** executar uma vez e reutilizar (memoização por request).

### PERF-005 · Loop SQL — *Alta*
**Falha:** INSERT/UPDATE um-a-um dentro de loop.
**Correção canônica:** operação em lote (multi-row insert, UPDATE com CASE/JOIN) com a mesma semântica transacional.

### PERF-006 · Sem índice — *Média*
**Falha:** consulta frequente filtra/ordena por coluna sem índice (a instância fornece o schema).
**Correção canônica:** propor o índice certo (coluna(s) e ordem) via migração aditiva — DDL destrutivo é `COMP-009`.

### PERF-007 · Recalcula estatísticas — *Média*
**Falha:** agregação cara recomputada a cada acesso (dashboard que soma a tabela toda por hit).
**Correção canônica:** cache com invalidação correta ou materialização incremental — valores continuam corretos após escrita.

### PERF-008 · Compressão dupla — *Baixa*
**Falha:** conteúdo comprimido re-comprimido (gzip sobre gzip, imagem re-encodada).
**Correção canônica:** comprimir uma única vez no ponto certo da cadeia.

### PERF-009 · Carregamento desnecessário — *Média*
**Falha:** recurso pesado carregado sem uso no caminho executado (biblioteca inteira, arquivo lido e descartado).
**Correção canônica:** carregar apenas quando o caminho precisa.

### PERF-010 · Cache incorreto — *Alta*
**Falha:** cache que serve dado errado (chave sem variação por usuário/parâmetro, TTL infinito para dado mutável).
**Correção canônica:** chave completa + invalidação/TTL coerentes. (É falha de performance *e* corretude — o teste de verificação cobra o dado certo.)

### PERF-011 · Uso excessivo de memória — *Média*
**Falha:** dataset inteiro materializado quando streaming/iteração bastaria (`fetchAll` + foreach único, `file_get_contents` de arquivo gigante).
**Correção canônica:** processar por fluxo/lotes com o mesmo resultado final.

### PERF-012 · Algoritmo O(n²) — *Média*
**Falha:** busca linear aninhada onde índice/estrutura adequada existe (`in_array` dentro de loop).
**Correção canônica:** estrutura de busca O(1)/O(log n) (set/mapa) preservando ordem de saída contratual.

### PERF-013 · Consulta redundante — *Baixa*
**Falha:** buscar dado que já está disponível no contexto (re-SELECT do registro recém-carregado).
**Correção canônica:** reutilizar o dado em mãos.

### PERF-014 · Falta de lazy loading — *Baixa*
**Falha:** relações/objetos caros instanciados ansiosamente em caminho que raramente os usa.
**Correção canônica:** adiar a criação para o primeiro uso, sem alterar a interface pública.

### PERF-015 · Bloqueio desnecessário — *Alta*
**Falha:** lock/transação segurados durante trabalho lento não relacionado (I/O externo, e-mail) serializando todo o sistema.
**Correção canônica:** encolher a seção crítica para o mínimo que exige atomicidade.
