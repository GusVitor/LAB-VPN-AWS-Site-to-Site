# Troubleshooting

Quando a VPN está configurada mas não existe conectividade, analisar em ordem:

## 1. VPN

Verificar:

- IKE
- IPsec
- estado do túnel
- negociação

## 2. Rotas

AWS:

```text
10.10.10.0/24 -> Virtual Private Gateway
```

pfSense:

```text
10.20.10.0/24 -> IPsec
```

## 3. Security Groups

Validar ICMP e TCP/22 conforme necessidade.

## 4. Firewall

Validar:

```text
LAN -> VPN
VPN -> LAN
```

## 5. Hosts

Validar:

- IP
- interface
- default gateway
- tabela de rotas
- firewall local

## Regra prática

```text
VPN UP
  ↓
Routes
  ↓
Security Groups
  ↓
Firewall
  ↓
Host
```
