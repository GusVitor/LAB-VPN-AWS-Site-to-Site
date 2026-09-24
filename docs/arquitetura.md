# Arquitetura

## Objetivo

Simular uma arquitetura híbrida entre um ambiente On-Premises e a AWS utilizando VPN Site-to-Site IPsec.

## Ambientes

### DTC-LOCAL

VPC que representa o datacenter:

- CIDR: `10.10.0.0/16`
- WAN: `10.10.1.0/24`
- LAN: `10.10.10.0/24`
- Firewall/router: pfSense
- Servidor de testes: Ubuntu `10.10.10.10`

### LAB-CONECTIVIDADE

VPC que representa o ambiente AWS:

- CIDR: `10.20.0.0/16`
- Private Subnet: `10.20.10.0/24`
- Servidor de testes: Ubuntu `10.20.10.10`
- Endpoint administrativo: EC2 Instance Connect Endpoint

## Topologia

```text
DTC-LOCAL
10.10.0.0/16
 |
 +-- WAN 10.10.1.0/24
 |      |
 |   pfSense
 |      |
 +-- LAN 10.10.10.0/24
        |
     Ubuntu
     10.10.10.10
        |
      IPsec
        |
Virtual Private Gateway
        |
LAB-CONECTIVIDADE
10.20.0.0/16
        |
Private Subnet
10.20.10.0/24
        |
     Ubuntu
     10.20.10.10
```

## Observação

O ambiente On-Premises foi simulado dentro da AWS exclusivamente para fins de laboratório.
