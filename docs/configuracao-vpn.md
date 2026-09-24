# Configuração VPN IPsec

A configuração do pfSense foi baseada no arquivo de configuração exportado pela AWS.

## Phase 1

| Parâmetro | Valor |
|---|---|
| IKE | IKEv1 |
| IP | IPv4 |
| Interface | WAN |
| Authentication | Mutual PSK |
| Negotiation | Main |
| Encryption | AES128 |
| Hash | SHA1 |
| DH | Group 2 |
| Lifetime | 28800s |
| NAT Traversal | Auto |
| DPD | Enabled |
| DPD Delay | 10s |
| DPD Retries | 3 |

## Phase 2

| Parâmetro | Valor |
|---|---|
| Mode | Tunnel |
| Protocol | ESP |
| Encryption | AES128 |
| Hash | HMAC-SHA1-96 |
| PFS | Group 2 |
| Lifetime | 3600s |

## Redes

```text
Local:
10.10.10.0/24

Remote:
10.20.10.0/24
```

## Túneis

A conexão AWS possui dois túneis. O laboratório utilizou um túnel operacionalmente para os testes e não implementou HA.

## Segurança

As PSKs presentes no arquivo original não devem ser publicadas no GitHub.
