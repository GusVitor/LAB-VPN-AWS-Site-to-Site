# Roteamento

## AWS

Route Table da subnet privada:

```text
10.20.0.0/16     local
10.10.10.0/24    Virtual Private Gateway
```

## On-Premises

Route Table da LAN:

```text
10.10.0.0/16     local
0.0.0.0/0        pfSense / ENI LAN
```

No pfSense, a rede AWS é alcançada pelo IPsec:

```text
10.20.10.0/24 -> IPsec
```

## Princípio

A conectividade exige rotas válidas nos dois sentidos.

```text
10.10.10.0/24
      ↕
    IPsec
      ↕
10.20.10.0/24
```
