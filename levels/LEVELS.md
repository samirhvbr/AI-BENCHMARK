# LEB — Níveis do Benchmark

Normativo. Cada nível define escala, escopo e composição mínima da matriz. Instâncias são nomeadas `LEB-<nível>-<letra>` (ex.: `LEB-200-A`).

| Nível | Linhas | Objetivo | Arquivos | Falhas na matriz (típico) | Iscas |
| --- | ---: | --- | ---: | ---: | ---: |
| LEB-100 | ~300 | Refatoração simples | 1–2 | 8–12 | 1–2 |
| LEB-200 | ~1.000 | Sistema legado pequeno | 3–8 | 18–25 | 2–4 |
| LEB-300 | ~3.000 | Múltiplos arquivos | 10–25 | 30–40 | 4–6 |
| LEB-400 | ~8.000 | Projeto empresarial | 30–80 | 45–60 | 6–10 |
| LEB-500 | ~20.000 | Sistema corporativo completo | 100+ | 70–90 | 10–15 |

## O que cada nível acrescenta

### LEB-100 — Refatoração simples (~300 linhas)
Um arquivo autocontido (script legado clássico). Testa o fundamento: achar e corrigir sem quebrar. Categorias: SEC, BUG, PERF, CLN. ARCH opcional (escala pequena demais para God Object legítimo).

### LEB-200 — Sistema legado pequeno (~1.000 linhas)
Primeiro nível com **arquitetura real para errar**: controller-faz-tudo, lógica duplicada entre páginas. Todas as categorias entram. Superfície pública mínima declarada no manifesto (2–3 símbolos + 1 formato de saída).

### LEB-300 — Múltiplos arquivos (~3.000 linhas)
Falhas **atravessam arquivos**: dependência circular, N+1 na fronteira entre módulos, contrato interno mal usado. O modelo precisa navegar, não só ler linearmente. Manifesto com rotas + schema.

### LEB-400 — Projeto empresarial (~8.000 linhas)
Código de gerações diferentes (estilos misturados, metade migrada), configuração/ambiente, migrações de banco. Falhas de priorização passam a pesar na rubrica EXPL: o modelo que corrige 40 CLN e ignora o IDOR revela imaturidade.

### LEB-500 — Sistema corporativo completo (~20.000 linhas)
Contexto acima da janela útil: exige **estratégia de exploração** (o protocolo registra o que o modelo pediu para ler). Todas as categorias, superfície pública extensa, consumidores externos simulados nos testes de caracterização.

## Regras por nível

1. Toda instância **DEVE** compilar/rodar e passar 100% da caracterização antes do plantio ser considerado válido.
2. A distribuição de severidades DEVERIA cobrir as quatro (Crítica–Baixa) a partir do LEB-200.
3. A distribuição de **dificuldade** (Fácil/Moderada/Difícil/Especialista — `SCORING.md §9.2`) DEVERIA escalar com o nível: LEB-100 concentra Fácil/Moderada com poucas Difíceis e **sem** Especialista; Especialista passa a aparecer a partir do LEB-300. Toda falha plantada **DEVE** ter dificuldade ratada (obrigatório no schema).
4. O nível declara o **orçamento de interação** do run (PROTOCOL.md §3): LEB-100/200 cabem em turno único; LEB-300+ admitem multi-turno com ferramentas de leitura.
5. Comparações de ranking entre modelos usam **a mesma instância**, nunca apenas "o mesmo nível".
