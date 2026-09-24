# Security Groups

## Ubuntu AWS

Regras de referência para o cenário:

| Protocolo | Porta | Origem | Finalidade |
|---|---:|---|---|
| ICMP | — | `10.10.10.0/24` | Ping |
| TCP | 22 | `10.10.10.0/24` | SSH |
| TCP | 80 | `10.10.10.0/24` | HTTP opcional |
| TCP | 443 | `10.10.10.0/24` | HTTPS opcional |

Não é necessário expor SSH para `0.0.0.0/0` neste cenário.

## pfSense

O SG do pfSense pode ser mais permissivo em um laboratório no qual o controle principal está concentrado no firewall.

Isso foi uma simplificação didática e não representa uma recomendação de produção.
