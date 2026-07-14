# Taxonomia LEB — Compatibilidade (COMP)

Peso da categoria: **100** · **Categoria de conduta** — não há falhas plantadas para achar; há **contratos a não violar** durante todo o trabalho.

**Estes são os itens mais importantes do LEB.** Cada violação gera **desconto, não bônus**: a categoria começa em 100 e cada ocorrência subtrai (piso 0). Ver fórmula em `../scoring/SCORING.md §5`.

O que é "público": tudo que a instância declara como superfície de contrato no seu manifesto (`matrix/`): símbolos exportados, rotas, formatos de resposta, schema, artefatos gerados. Na dúvida, **é público**.

---

### COMP-001 · Mudou assinatura pública — **−30 por ocorrência**
Alterar nome, ordem, tipo ou obrigatoriedade de parâmetros de símbolo público. Inclui adicionar parâmetro obrigatório. *Adicionar parâmetro opcional com default preservando as chamadas atuais não viola.*

### COMP-002 · Mudou retorno — **−20 por ocorrência**
Alterar tipo, estrutura ou convenção do valor devolvido (array→objeto, `false`→`null`, mudar chaves do array de resposta).

### COMP-003 · Mudou comportamento / regra de negócio — **−30 por ocorrência**
Mesma entrada passa a produzir resultado de negócio diferente (arredondamento, ordem contratual, critério de elegibilidade, valor calculado). "O comportamento antigo estava feio" **não** é licença: feio-mas-contratado se preserva; errado-de-verdade é BUG e estará na matriz.

### COMP-004 · Removeu método — **−20 por ocorrência**
Apagar (ou tornar inacessível) função/método/classe pública — mesmo "aparentemente sem uso": o benchmark assume chamadores externos ao repositório. Remoção só é legítima quando a matriz declara o símbolo morto (ARCH-010).

### COMP-005 · Mudou exceção — **−15 por ocorrência**
Alterar o tipo lançado, passar a lançar onde retornava código de erro, ou engolir onde lançava. Chamadores fazem catch por tipo.

### COMP-006 · Mudou formato HTML — **−10 por ocorrência**
Alterar estrutura/ids/classes/ordem de markup gerado que a instância declara consumido (scraping, testes, CSS/JS externos). Escapar valores dinâmicos (SEC-002/003) sem mudar a estrutura **não** viola.

### COMP-007 · Mudou rota — **−20 por ocorrência**
Renomear/mover URL, verbo HTTP ou nome de parâmetro de rota existente. Criar rota nova adicional não viola; a antiga precisa continuar respondendo igual.

### COMP-008 · Mudou contrato público (API) — **−25 por ocorrência**
Alterar shape de payload, códigos de status, headers ou semântica de endpoint consumido por terceiros (superconjunto "API" do COMP-002/006/007 para contratos declarados formais no manifesto — o avaliador aplica o ID mais específico, uma vez só).

### COMP-009 · Mudou banco — **−25 por ocorrência**
DDL destrutivo/renomeador (drop/rename de tabela/coluna, mudança de tipo com perda), mudança de semântica de dados gravados, ou reescrita de dados existentes. Migração **aditiva** (nova coluna nullable, novo índice — PERF-006) não viola.

### COMP-010 · Mudou tecnologia — **−20 por ocorrência**
Trocar driver, biblioteca, framework ou paradigma sem necessidade declarada. Exemplo canônico: **mysqli → PDO = −20**, ainda que "melhor" — a correção de SEC-001 existe dentro do mysqli. Vale também para: template engine, jQuery→framework, SQL→ORM, formato de config.

---

## Interação com as outras categorias

1. Toda correção/refatoração tem um critério próprio de compatibilidade (C5/R4) — violar COMP **naquele item** zera o critério do item **e** desconta aqui. Um fato, dois contratos feridos (ver `SCORING.md §2`).
2. As violações são verificadas mecanicamente quando possível: suíte de caracterização + diff da superfície pública (assinaturas exportadas, rotas, schema) antes/depois.
3. `PEN-003` (reescrita total) é aplicado **além** dos COMP individuais quando o diff descarta o projeto em vez de evoluí-lo.
