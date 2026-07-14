# Taxonomia LEB — Bugs (BUG)

Peso da categoria: **150** · Template de critérios: **C** (Correção) · IDs imutáveis.

Regra de ouro da categoria: cada bug plantado tem um **cenário de reprodução** na matriz (entrada/estado → saída errada). A correção passa quando o cenário deixa de reproduzir **e** os testes de caracterização do entorno continuam verdes.

---

### BUG-001 · Divisão por zero — *Alta*
**Falha:** divisor pode ser zero em caminho real (média de lista vazia, rateio sem itens).
**Correção canônica:** tratar o caso vazio com a semântica que o domínio pede (0, null ou erro claro — a matriz declara qual).

### BUG-002 · Overflow — *Média*
**Falha:** estouro de inteiro/precisão (dinheiro em float, contador em int estreito, truncamento).
**Correção canônica:** tipo/aritmética adequados ao domínio, sem mudar o formato externo do valor.

### BUG-003 · Resource Leak — *Alta*
**Falha:** recurso adquirido e não liberado em algum caminho (conexão, lock, memória em loop).
**Correção canônica:** liberação garantida em todos os caminhos (finally/RAII/using do idioma local).

### BUG-004 · File Handle Leak — *Média*
**Falha:** arquivo aberto sem fechamento no caminho de erro/retorno antecipado.
**Correção canônica:** fechar em todos os caminhos; caso especial do BUG-003, plantado separadamente por ser o vazamento mais comum em legado.

### BUG-005 · Header após Output — *Média*
**Falha:** `header()`/`setcookie()` após corpo já enviado (echo, BOM, espaço antes de `<?php`).
**Correção canônica:** reordenar o fluxo (decidir headers antes de emitir corpo), sem buffer global como esparadrapo — a menos que a matriz aceite.

### BUG-006 · Timezone Global — *Alta*
**Falha:** timezone global alterado no meio da request (ou datas gravadas em fusos mistos).
**Correção canônica:** conversão localizada no ponto de exibição; armazenamento consistente. Mudar o dado gravado é `COMP-009`.

### BUG-007 · Condição impossível — *Média*
**Falha:** condição sempre-verdadeira/sempre-falsa que altera o comportamento pretendido (`if ($x = 1)`, comparação de tipos incompatíveis, `&&` que devia ser `||`).
**Correção canônica:** restaurar a intenção declarada na matriz.

### BUG-008 · Variável não inicializada — *Média*
**Falha:** uso de variável que pode não ter valor no caminho executado (acumulador fora do if, typo no nome).
**Correção canônica:** inicialização correta no escopo certo.

### BUG-009 · Switch incompleto — *Média*
**Falha:** case sem `break` (fallthrough acidental) ou valor de domínio sem case e sem default.
**Correção canônica:** completar a estrutura com a semântica pretendida.

### BUG-010 · Erro silencioso — *Alta*
**Falha:** falha engolida (`@`, `catch {}` vazio, retorno de erro ignorado) fazendo o sistema fingir sucesso.
**Correção canônica:** propagar ou tratar de verdade + registrar; **sem** transformar em exceção nova visível a chamadores (`COMP-005`) a menos que a matriz peça.

### BUG-011 · Race Condition — *Crítica*
**Falha:** check-then-act sem atomicidade (saldo verificado e debitado em passos separados, last-write-wins em contador).
**Correção canônica:** operação atômica (UPDATE condicional, lock correto, unique constraint) mantendo o contrato externo.

### BUG-012 · Null Pointer — *Alta*
**Falha:** desreferência de valor possivelmente nulo em caminho real (busca que não encontra, campo opcional).
**Correção canônica:** tratar a ausência com a semântica do domínio declarada na matriz.

### BUG-013 · Conversão incorreta — *Média*
**Falha:** coerção de tipo que corrompe o valor (`(int)"12,50"`, comparação `==` frouxa, parse de data com formato errado).
**Correção canônica:** conversão explícita e correta para o formato real dos dados.

### BUG-014 · Falta de rollback — *Alta*
**Falha:** sequência de escritas relacionadas sem transação/compensação — falha no meio deixa estado inconsistente.
**Correção canônica:** transação envolvendo o conjunto (ou compensação), com rollback nos caminhos de erro.

### BUG-015 · Exceção perdida — *Média*
**Falha:** exceção capturada e re-lançada perdendo causa/contexto, ou capturada genérica demais mascarando tipos que deviam subir.
**Correção canônica:** preservar a cadeia de causa e capturar apenas o que o ponto sabe tratar.
