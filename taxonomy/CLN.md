# Taxonomia LEB — Clean Code (CLN)

Peso da categoria: **100** · Template de critérios: **R** (Refatoração) · IDs imutáveis.

Regra de ouro da categoria: limpeza é a categoria de **menor** peso do LEB — de propósito. Polir código sem antes garantir segurança/comportamento é inversão de prioridade, e o modelo que só entrega limpeza pontua pouco.

---

### CLN-001 · Função gigante — *Média*
**Falha:** função/método muito acima do limite declarado na matriz (tipicamente >80 linhas com múltiplos níveis de abstração misturados).
**Refatoração-alvo:** extrair blocos coesos em funções nomeadas; assinatura pública original intacta.

### CLN-002 · Método duplicado — *Média*
**Falha:** dois métodos com corpo idêntico/quase idêntico no mesmo escopo (cópia que divergirá).
**Refatoração-alvo:** um delega ao outro (ou ambos a um privado comum); **ambos continuam existindo** se forem públicos (`COMP-004`).

### CLN-003 · Comentário incorreto — *Baixa*
**Falha:** comentário afirma o contrário do que o código faz (armadilha ativa para manutenção).
**Refatoração-alvo:** corrigir/remover o comentário — **sem "corrigir" o código para obedecer ao comentário** (isso mudaria comportamento: `COMP-003`), salvo se a matriz declarar o código como o errado (aí é BUG).

### CLN-004 · Nome ruim — *Baixa*
**Falha:** identificador que mente ou nada diz (`$data2`, `processar()`, `flag`), no escopo declarado na matriz.
**Refatoração-alvo:** renomear com significado — apenas identificadores **internos**; renomear símbolo público é `COMP-001/004`.

### CLN-005 · Código comentado — *Baixa*
**Falha:** blocos de código morto em comentário ("por via das dúvidas").
**Refatoração-alvo:** remover; o histórico é papel do VCS.

### CLN-006 · Comentário inútil — *Baixa*
**Falha:** comentário que repete o óbvio (`// incrementa i`), ruído que dilui os comentários que importam.
**Refatoração-alvo:** remover ruído, **preservando** comentários com informação real (o gabarito lista quais ficam).

### CLN-007 · if aninhado — *Média*
**Falha:** aninhamento profundo (≥4 níveis) escondendo o fluxo principal.
**Refatoração-alvo:** guard clauses/retorno antecipado/extração — mesma tabela-verdade (o teste de caracterização confere).

### CLN-008 · else desnecessário — *Baixa*
**Falha:** `else` após bloco que já retorna/lança.
**Refatoração-alvo:** achatar o fluxo removendo o else.

### CLN-009 · DRY violado — *Média*
**Falha:** o mesmo conhecimento (constante, formato, regra pequena) repetido em vários pontos — escala menor que ARCH-004.
**Refatoração-alvo:** ponto único de verdade referenciado pelos usos.

### CLN-010 · Sem tipagem — *Baixa*
**Falha:** assinaturas sem tipos onde a linguagem suporta (parâmetros/retornos), no conjunto declarado na matriz.
**Refatoração-alvo:** adicionar tipos que reflitam o comportamento **atual** (inclusive nullables) — apertar o tipo além do real quebra chamadores (`COMP-001`).
