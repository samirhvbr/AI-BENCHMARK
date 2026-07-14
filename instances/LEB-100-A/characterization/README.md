# Caracterização — LEB-100-A

Os testes de **caracterização** travam a superfície pública (`../manifest.md`): eles capturam o comportamento *contratual* do sistema, não as falhas. Servem para medir **regressão** e **compatibilidade** — devem passar no código legado intocado **e** na entrega do modelo, se ela preservou os contratos.

> Se um teste daqui fica vermelho depois da correção, houve regressão (C4/PEN-002) ou violação `COMP-*`.

## Rodar

O ambiente sobe um MySQL 8 e um PHP 8.4 com `mysqli`, sem depender de nada instalado na máquina:

```sh
cd characterization
docker compose up -d                                   # sobe o MySQL (aguarda healthcheck)
docker compose run --rm php php characterization/run.php
docker compose down -v                                 # encerra
```

Saída esperada no legado: **22 verificações ok, 0 falharam** (exit 0).

## Avaliar a entrega de um modelo

Aponte `LEB_CODE_DIR` para o `code/` entregue pelo modelo (montado no container) e rode a mesma suíte:

```sh
docker compose run --rm \
  -v /caminho/da/entrega/code:/entrega -e LEB_CODE_DIR=/entrega \
  php php characterization/run.php
```

- Verde → contratos preservados.
- Vermelho → regressão/compatibilidade quebrada; anotar no scorecard (`../../../scoring/`).

## O que cada bloco protege

| Bloco | Contrato |
| --- | --- |
| `autenticar()` | login funciona e retorna `id,nome,papel` — protege contra "invalidei as senhas" ao corrigir o hash |
| `formatarStatus()` | rótulos `Aberto/Em atendimento/Resolvido` |
| `rotuloPrioridade()` | tabela-verdade do SLA — protege a refatoração do `if` aninhado |
| `listarChamados()` | 5 chamados, chave `tecnico_nome`, busca funcional |
| `verChamado()` | detalhe por id (a assinatura não muda ao corrigir o IDOR) |
| `mediaResposta()` | média correta com dados normais |
| `exportarCsv()` | colunas do CSV e ordem por id (comparadas por campo, não por bytes) |

## Notas

- A comparação do CSV usa `str_getcsv` (campos), não igualdade textual — quoting é detalhe de serialização e varia por versão do PHP.
- `_bootstrap.php` é compartilhado com os probes de `private/verify/`; recria o banco a partir de `../code/schema.sql` + `../code/seed.sql` a cada execução.
