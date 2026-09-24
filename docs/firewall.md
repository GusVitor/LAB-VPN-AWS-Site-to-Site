# Firewall

O pfSense atua como firewall e roteador do ambiente On-Premises simulado.

## LAN → VPN

```text
Source:      10.10.10.0/24
Destination: 10.20.10.0/24
Protocols:   ICMP, TCP/22
```

## VPN → LAN

```text
Source:      10.20.10.0/24
Destination: 10.10.10.0/24
Protocols:   ICMP, TCP/22
```

## IPsec

Considerar o tráfego necessário à negociação IPsec, incluindo UDP 500 e UDP 4500 quando NAT-T estiver em uso.

> As regras acima são uma reconstrução técnica de referência. O ambiente original foi removido e não havia screenshots disponíveis.
