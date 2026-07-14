# Taxonomia LEB — Arquitetura (ARCH)

Peso da categoria: **200** · Template de critérios: **R** (Refatoração) · IDs imutáveis.

Regra de ouro da categoria: refatorar é **mudar estrutura preservando comportamento**. A matriz de cada instância declara a *estrutura-alvo* esperada (critério R3); alcançá-la quebrando chamadas existentes anula R4 e dispara `COMP-*`.

---

### ARCH-001 · God Object — *Crítica*
**Falha:** uma classe concentra domínios não relacionados (persistência + regra + apresentação + e-mail...).
**Refatoração-alvo:** decompor por responsabilidade, mantendo a classe original como fachada compatível quando houver chamadores externos.

### ARCH-002 · Controller faz tudo — *Alta*
**Falha:** controller/handler contém regra de negócio, SQL e formatação.
**Refatoração-alvo:** extrair serviço/consulta; controller só orquestra. Rotas e parâmetros intocados.

### ARCH-003 · Repository inexistente — *Alta*
**Falha:** acesso a dados espalhado (SQL cru em views/controllers).
**Refatoração-alvo:** concentrar acesso a dados em camada dedicada, **na mesma tecnologia de acesso** (extrair repositório ≠ trocar driver).

### ARCH-004 · Lógica duplicada — *Média*
**Falha:** mesma regra de negócio implementada N vezes (e já divergindo).
**Refatoração-alvo:** unificar em um único ponto de verdade; os N chamadores passam a delegar.

### ARCH-005 · Violação SOLID — *Alta*
**Falha:** violação estrutural declarada na matriz (ex.: LSP quebrado por subclasse que lança em método herdado; DIP invertido).
**Refatoração-alvo:** a correção específica declarada; citar o princípio certo conta em R2.

### ARCH-006 · Acoplamento excessivo — *Alta*
**Falha:** módulo conhece detalhes internos de outro (acessa propriedades/tabelas alheias diretamente).
**Refatoração-alvo:** interação via interface estável do módulo dono.

### ARCH-007 · Baixa coesão — *Média*
**Falha:** módulo/classe agrupa funções sem relação entre si ("Manager", "Helper" temático).
**Refatoração-alvo:** reagrupar por afinidade real, preservando pontos de entrada públicos.

### ARCH-008 · Dependência circular — *Alta*
**Falha:** A→B→A (imports, includes ou chamadas mútuas).
**Refatoração-alvo:** quebrar o ciclo (extrair terceiro módulo ou inverter dependência).

### ARCH-009 · Magic Numbers — *Baixa*
**Falha:** constantes de negócio soltas no código (`if ($tipo == 3)`, `* 0.0725`).
**Refatoração-alvo:** constantes nomeadas no escopo certo — com os **mesmos valores**.

### ARCH-010 · Código morto — *Baixa*
**Falha:** código nunca invocado por nenhum caminho (função órfã, feature desativada há anos).
**Refatoração-alvo:** remover **apenas o que a matriz declara morto** — remover método público vivo é `COMP-004`.

### ARCH-011 · Responsabilidade múltipla — *Média*
**Falha:** um método/função faz várias coisas em sequência (valida + calcula + persiste + notifica). Escopo menor que ARCH-001.
**Refatoração-alvo:** extrair etapas em unidades nomeadas; assinatura pública original preservada.

### ARCH-012 · Classe utilitária gigante — *Média*
**Falha:** `Utils`/`Helpers` estático com dezenas de funções heterogêneas.
**Refatoração-alvo:** dividir por domínio; manter shims/encaminhadores se houver uso externo.

### ARCH-013 · Dependência global — *Alta*
**Falha:** estado global mutável (`global $db`, singleton mutável, superglobais lidas no fundo da pilha).
**Refatoração-alvo:** injetar a dependência na borda, propagando por parâmetro/construtor.

### ARCH-014 · Configuração hardcoded — *Média*
**Falha:** valores de ambiente no código (hosts, paths, limites, e-mails de admin).
**Refatoração-alvo:** externalizar em configuração, com os mesmos valores como default de runtime.

### ARCH-015 · Código inalcançável — *Baixa*
**Falha:** código após `return`/`throw`/`exit` ou sob condição estruturalmente impossível (diferente de BUG-007: aqui o defeito é o *entulho*, não a condição).
**Refatoração-alvo:** remover o trecho inalcançável declarado na matriz.
