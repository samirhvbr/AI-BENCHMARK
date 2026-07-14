# Scorecard Oficial LEB

| Campo | Valor |
| --- | --- |
| Modelo | <nome + versão exata> |
| Instância | LEB-___-_ v___ |
| Hash da matriz | `sha256:…` |
| Modo | S / A (orçamento: __ turnos) |
| Temperatura | ___ |
| Data | AAAA-MM-DD |
| Execuções | 3 (totais: ___ / ___ / ___ → mediana ___) |
| Spec LEB | 1.0.0 |

---

## Detalhe por falha

Legenda: ✔ pleno · ◐ parcial · ✘ zero — por critério (C1..C5 / R1..R4).

### Segurança — ___ / 250

| ID | C1 | C2 | C3 | C4 | C5 | Pts |
| --- | :-: | :-: | :-: | :-: | :-: | ---: |
| SEC-001 | ✔ | ✔ | ✔ | ✔ | ✔ | 10/10 |
| SEC-002 | ✔ | ✔ | ✘ | — | — | 4/10 |
| SEC-003 | ✘ | ✘ | ✘ | ✘ | ✘ | 0/8 |
| … | | | | | | |

Bruto: ___ / ___ → normalizado ×250.

### Arquitetura — ___ / 200

| ID | R1 | R2 | R3 | R4 | Pts |
| --- | :-: | :-: | :-: | :-: | ---: |
| ARCH-002 | ✔ | ✔ | ◐ | ✔ | 8/10 |
| … | | | | | |

### Bugs — ___ / 150
### Performance — ___ / 150
### Clean Code — ___ / 100

(mesmo formato)

### Compatibilidade — ___ / 100

| Violação | Ocorrências | Desconto |
| --- | ---: | ---: |
| COMP-010 (mysqli→PDO) | 1 | −20 |
| … | | |

`100 − Σ = ___`

### Explicação Técnica — ___ / 50

| Dimensão | Nota |
| --- | ---: |
| Clareza | _/10 |
| Precisão técnica | _/10 |
| Causa-raiz | _/10 |
| Priorização | _/10 |
| Trade-offs | _/10 |

### Penalidades gerais

| ID | Ocorrências | Desconto |
| --- | ---: | ---: |
| PEN-001 | _ | −__ |
| PEN-004 (iscas: <IDs>) | _ | −__ |

---

## Resumo

```text
Modelo: <nome>
--------------------
Segurança        ___ / 250
Arquitetura      ___ / 200
Performance      ___ / 150
Bugs             ___ / 150
Clean Code       ___ / 100
Compatibilidade  ___ / 100
Explicação       ___ /  50
Penalidades      −___
--------------------
TOTAL            ___ / 1000    →  <selo SCORING.md §8>
```

## Achados fora da matriz (informativo)

| Achado | Localização | Destino |
| --- | --- | --- |
| <descrição> | arq:linha | candidato à v seguinte da instância |
