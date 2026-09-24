# AWS Site-to-Site VPN — Simulação de Ambiente Híbrido

Laboratório prático para simular a conectividade entre um ambiente **On-Premises** e a **AWS** utilizando uma **VPN Site-to-Site IPsec**.

> **Objetivo do laboratório:** estudar, na prática, VPN IPsec, roteamento, subnets, Security Groups, firewall, ENI, VPC Endpoint e troubleshooting de conectividade em um cenário híbrido.
>
> **Observação:** o ambiente On-Premises deste laboratório foi **simulado dentro da AWS**, utilizando uma VPC separada. Em um cenário real, o firewall estaria localizado no datacenter/ambiente On-Premises.

---

## 📌 Visão geral

A arquitetura foi construída com dois ambientes:

- **DTC-LOCAL** — VPC que representa o datacenter On-Premises.
- **LAB-CONECTIVIDADE** — VPC que representa o ambiente AWS.

A comunicação entre as redes é realizada através de uma **VPN Site-to-Site IPsec**, utilizando um **pfSense** como Customer Gateway no ambiente simulado On-Premises e um **Virtual Private Gateway** no lado AWS.

O laboratório foi realizado pelo **Console da AWS** e pela interface de administração do **pfSense**. Não foram utilizados procedimentos via AWS CLI na documentação.

---

## 🏗️ Arquitetura

```text
                         INTERNET
                             |
                             |
                     +-------+-------+
                     |    pfSense    |
                     |               |
                     |      WAN      |
                     +-------+-------+
                             |
                      10.10.1.0/24
                             |
                     +-------+-------+
                     |      LAN      |
                     | 10.10.10.0/24|
                     +-------+-------+
                             |
                       Ubuntu Server
                        10.10.10.10
                             |
                             |
                    =================
                       IPsec VPN
                    =================
                             |
                             |
                  Virtual Private Gateway
                             |
                     +-------+-------+
                     |   AWS VPC     |
                     |10.20.0.0/16   |
                     |               |
                     | Private       |
                     |10.20.10.0/24  |
                     |       |       |
                     | Ubuntu        |
                     |10.20.10.10    |
                     +---------------+
```

### Fluxo principal

```text
Ubuntu On-Premises
10.10.10.10
      |
      v
pfSense LAN
10.10.10.1
      |
      v
IPsec VPN
      |
      v
Virtual Private Gateway
      |
      v
AWS Route Table
      |
      v
Ubuntu AWS
10.20.10.10
```

O fluxo inverso também foi validado.

---

# 📐 Endereçamento

Os CIDRs abaixo são utilizados como **endereçamento de referência para a documentação do laboratório**. Eles não representam necessariamente os endereços originais do ambiente, que já foi removido.

| Ambiente | Componente | CIDR / IP |
|---|---|---:|
| On-Premises simulado | VPC `DTC-LOCAL` | `10.10.0.0/16` |
| On-Premises | WAN Subnet | `10.10.1.0/24` |
| On-Premises | LAN Subnet | `10.10.10.0/24` |
| On-Premises | pfSense LAN | `10.10.10.1` |
| On-Premises | Ubuntu Server | `10.10.10.10` |
| AWS | VPC `LAB-CONECTIVIDADE` | `10.20.0.0/16` |
| AWS | Private Subnet | `10.20.10.0/24` |
| AWS | Ubuntu Server | `10.20.10.10` |

A rede utilizada para representar a comunicação entre os ambientes é:

```text
10.10.10.0/24  <---- IPsec VPN ---->  10.20.10.0/24
```

---

# ☁️ Ambiente On-Premises simulado

## VPC — DTC-LOCAL

Foi criada uma VPC denominada:

```text
DTC-LOCAL
CIDR: 10.10.0.0/16
```

Essa VPC representa o datacenter corporativo.

### Subnets

Foram utilizadas duas subnets:

```text
WAN
10.10.1.0/24
```

e:

```text
LAN
10.10.10.0/24
```

A subnet WAN representa a conectividade externa do datacenter.

A subnet LAN representa a rede interna onde ficam os servidores.

---

# 🔥 pfSense

O firewall utilizado foi o **pfSense**, provisionado através do **AWS Marketplace**.

Tipo de instância utilizado:

```text
t3.micro
```

O pfSense possui duas interfaces de rede:

| Interface | Função |
|---|---|
| WAN | Comunicação externa / endpoint da VPN |
| LAN | Comunicação com a rede interna |

Um **Elastic IP** foi associado à interface WAN para representar um endereço público e fixo do ambiente On-Premises simulado.

Uma segunda **ENI** foi associada ao pfSense na subnet privada para representar a interface LAN.

---

## Source/Destination Check

A verificação de origem/destino da instância do pfSense foi desabilitada.

Essa configuração permite que a instância atue como equipamento de rede, encaminhando tráfego entre interfaces e redes.

Fluxo:

```text
LAN
 |
 v
pfSense
 |
 +---- Internet
 |
 +---- VPN
```

---

# 🖥️ Ubuntu — On-Premises

Uma instância Ubuntu Server foi criada na subnet privada do ambiente DTC-LOCAL.

Endereço utilizado para documentação:

```text
10.10.10.10
```

Gateway:

```text
10.10.10.1
```

A instância foi utilizada como origem dos testes de conectividade através da VPN.

---

# 🛣️ Roteamento — On-Premises

A tabela de rotas da LAN possui uma rota padrão direcionada ao pfSense:

```text
Destination    Target
-------------  ----------------
10.10.0.0/16   local
0.0.0.0/0      pfSense / ENI LAN
```

Além disso, o pfSense precisa possuir conhecimento da rede AWS através do túnel IPsec:

```text
Destination:
10.20.10.0/24

Interface:
IPsec / VPN
```

O princípio é:

```text
10.10.10.0/24
      |
      v
   pfSense
      |
      v
    IPsec
      |
      v
10.20.10.0/24
```

---

# ☁️ Ambiente AWS

## VPC — LAB-CONECTIVIDADE

Foi criada uma segunda VPC:

```text
LAB-CONECTIVIDADE
CIDR: 10.20.0.0/16
```

Essa VPC representa o ambiente corporativo hospedado na AWS.

---

## Private Subnet

Foi criada uma subnet privada:

```text
10.20.10.0/24
```

Dentro dela foi provisionada uma instância Ubuntu Server para os testes.

Endereço utilizado para documentação:

```text
10.20.10.10
```

A instância não precisa de exposição direta à Internet para participar dos testes da VPN.

---

# 🔐 AWS Site-to-Site VPN

A conexão VPN é composta conceitualmente por:

```text
Customer Gateway
        |
        | Internet / IPsec
        |
Virtual Private Gateway
        |
        v
AWS VPC
```

### Customer Gateway

O **Customer Gateway** representa o dispositivo do lado On-Premises.

Neste laboratório:

```text
Customer Gateway
        |
        v
     pfSense
```

O endereço público do pfSense é utilizado como endpoint do Customer Gateway.

### Virtual Private Gateway

O **Virtual Private Gateway** representa o endpoint da VPN no lado AWS e é associado à VPC `LAB-CONECTIVIDADE`.

### VPN Connection

A conexão Site-to-Site VPN estabelece os túneis IPsec entre o pfSense e a AWS.

---

# 🔑 Configuração IPsec

A configuração exportada pela AWS foi utilizada como referência para configurar o pfSense.

> **Segurança:** as chaves PSK e os endpoints públicos presentes no arquivo original não são publicados neste repositório. O ambiente original foi encerrado, mas segredos de VPN não devem ser versionados em repositórios Git.

## Phase 1 — IKE

| Parâmetro | Configuração |
|---|---|
| IKE | IKEv1 |
| IP | IPv4 |
| Interface | WAN |
| Autenticação | Mutual PSK |
| Negotiation Mode | Main |
| Encryption | AES128 |
| Hash | SHA1 |
| DH Group | 2 |
| Lifetime | 28800 segundos |
| NAT Traversal | Auto |
| DPD | Habilitado |
| DPD Delay | 10 segundos |
| DPD Retries | 3 |

A configuração exportada da AWS especifica IKEv1, AES128, SHA1, DH Group 2, lifetime de 28800 segundos, NAT Traversal em modo automático e DPD habilitado. fileciteturn0file0L40-L67

---

## Phase 2 — IPsec

| Parâmetro | Configuração |
|---|---|
| Mode | Tunnel |
| Protocol | ESP |
| Encryption | AES128 |
| Hash | HMAC-SHA1-96 |
| PFS | Group 2 |
| Lifetime | 3600 segundos |

Esses parâmetros constam na configuração exportada pela AWS. 

### Redes da Phase 2

```text
Local Network:
10.10.10.0/24

Remote Network:
10.20.10.0/24
```

---

# 🔄 Túneis VPN

A conexão AWS disponibiliza dois túneis IPsec.

A configuração exportada contém parâmetros para:

```text
Tunnel 1
Tunnel 2
```

e a própria configuração informa que os dois túneis devem ser configurados para redundância. fileciteturn0file0L14-L18

Neste laboratório, entretanto, **somente um túnel foi utilizado operacionalmente para os testes**.

Portanto:

> **Alta disponibilidade não fez parte do escopo deste laboratório.**

Arquitetura utilizada:

```text
              AWS
               |
       Virtual Private
           Gateway
               |
          Tunnel IPsec
               |
            pfSense
               |
          On-Premises
```

Em um ambiente produtivo, os dois túneis disponibilizados pela VPN devem ser considerados na arquitetura de redundância.

---

# 🛣️ Route Table — AWS

A tabela de rotas associada à subnet privada precisa possuir uma rota para a LAN do ambiente On-Premises.

```text
Destination        Target
-----------------  ---------------------------
10.20.0.0/16       local
10.10.10.0/24      Virtual Private Gateway
```

Com isso, quando a instância AWS precisa acessar:

```text
10.10.10.0/24
```

o tráfego é encaminhado para o Virtual Private Gateway e, posteriormente, para o túnel IPsec.

Fluxo:

```text
Ubuntu AWS
10.20.10.10
      |
      v
Route Table
      |
      v
Virtual Private Gateway
      |
      v
IPsec
      |
      v
pfSense
      |
      v
10.10.10.0/24
```

---

# 🛡️ Security Groups

Os Security Groups foram utilizados como camada de controle de acesso no ambiente AWS.

Para o cenário documentado, as liberações necessárias são:

## Ubuntu AWS

| Protocolo | Porta | Origem | Finalidade |
|---|---:|---|---|
| ICMP | — | `10.10.10.0/24` | Teste de ping |
| TCP | 22 | `10.10.10.0/24` | SSH |
| TCP | 80 | `10.10.10.0/24` | Teste HTTP opcional |
| TCP | 443 | `10.10.10.0/24` | Teste HTTPS opcional |

A regra de SSH deve estar limitada à rede necessária para o laboratório, evitando exposição desnecessária à Internet.

---

# 🔥 Regras do pfSense

O pfSense foi utilizado como principal ponto de controle de tráfego no ambiente On-Premises simulado.

Para a documentação do cenário, as regras necessárias são:

### LAN → VPN

Permitir:

```text
Source:
10.10.10.0/24

Destination:
10.20.10.0/24
```

Protocolos utilizados nos testes:

```text
ICMP
TCP/22
```

### VPN → LAN

Permitir:

```text
Source:
10.20.10.0/24

Destination:
10.10.10.0/24
```

Protocolos:

```text
ICMP
TCP/22
```

### VPN / IKE

Para estabelecimento/manutenção da VPN, devem ser considerados os protocolos necessários à negociação IPsec, incluindo UDP 500 e, quando NAT-T estiver sendo utilizado, UDP 4500. A configuração exportada da AWS também orienta essa necessidade para NAT-T. fileciteturn0file0L31-L35

> **Nota:** essas regras são uma reconstrução técnica de referência baseada no cenário informado. Como o ambiente foi excluído e não existem screenshots das regras originais, elas não devem ser tratadas como cópia exata da configuração histórica.

---

# 🔌 EC2 Instance Connect Endpoint

Foi utilizado um **EC2 Instance Connect Endpoint** para permitir o acesso à instância Ubuntu sem necessidade de disponibilizar a instância diretamente na Internet.

O componente foi utilizado principalmente para:

- acesso administrativo;
- realização dos testes;
- manter a instância em subnet privada;
- reduzir a necessidade de exposição pública.

Fluxo de acesso:

```text
Administrador
      |
      v
EC2 Instance Connect Endpoint
      |
      v
Private Subnet
      |
      v
Ubuntu
```

---

# 🧪 Testes de conectividade

Após a configuração dos componentes, foram realizados testes de conectividade entre as redes.

## Teste 1 — ICMP

Origem:

```text
Ubuntu On-Premises
10.10.10.10
```

Destino:

```text
Ubuntu AWS
10.20.10.10
```

Resultado:

```text
PING
    |
    +---- OK
```

---

## Teste 2 — ICMP no sentido inverso

Origem:

```text
Ubuntu AWS
10.20.10.10
```

Destino:

```text
Ubuntu On-Premises
10.10.10.10
```

Resultado:

```text
PING
    |
    +---- OK
```

---

# 🔐 Teste SSH

Também foi validada a comunicação utilizando SSH.

Fluxo:

```text
Ubuntu On-Premises
        |
        | TCP/22
        v
     IPsec VPN
        |
        v
   Ubuntu AWS
```

Resultado:

```text
SSH
 |
 +---- OK
```

Os testes confirmaram comunicação entre as redes através da VPN.

---

# 🔎 Troubleshooting

Uma das principais finalidades deste laboratório foi compreender que uma VPN estabelecida não significa necessariamente que existe conectividade entre os hosts.

O troubleshooting deve seguir uma análise em camadas.

## 1. Estado da VPN

Verificar:

```text
IKE
IPsec
Tunnel Status
```

Primeiro deve-se confirmar se o túnel está estabelecido.

---

## 2. Rotas

Verificar no lado AWS:

```text
Route Table
      |
      +---- 10.10.10.0/24
             |
             +---- Virtual Private Gateway
```

No lado On-Premises:

```text
pfSense
      |
      +---- 10.20.10.0/24
             |
             +---- IPsec
```

---

## 3. Security Groups

Confirmar se o tráfego necessário está autorizado.

Principalmente:

```text
ICMP
TCP/22
```

---

## 4. Firewall

Confirmar as regras do pfSense:

```text
LAN → VPN
VPN → LAN
```

---

## 5. Sistema operacional

Validar:

```text
Interface de rede
IP
Default Gateway
Routing Table
Firewall local
```

---

# 🧠 Conceitos praticados

## AWS

- Amazon VPC
- Amazon EC2
- Subnets
- Route Tables
- Security Groups
- Elastic Network Interface — ENI
- Elastic IP
- Customer Gateway
- Virtual Private Gateway
- Site-to-Site VPN
- EC2 Instance Connect Endpoint

## Redes

- IPv4
- CIDR
- Subnetting
- LAN
- WAN
- Routing
- Default Gateway
- IP público
- IP privado
- VPN
- IPsec
- IKE
- ESP
- NAT-T

## Segurança

- Firewall
- Security Groups
- Segmentação de rede
- Controle de tráfego
- Acesso privado
- Princípio do menor privilégio
- Redução de superfície de exposição

---

# ⚠️ Limitações do laboratório

Este laboratório foi desenvolvido com foco em **aprendizado técnico e compreensão dos conceitos**.

Algumas configurações foram simplificadas propositalmente.

### Ambiente On-Premises

O datacenter foi simulado dentro da AWS.

```text
AWS VPC
   |
   +---- pfSense
   |
   +---- LAN
   |
   +---- Ubuntu
```

Em produção, o firewall normalmente estaria no próprio datacenter ou em outro ambiente de infraestrutura.

### Alta disponibilidade

Não foi implementada alta disponibilidade.

Somente um túnel foi utilizado operacionalmente nos testes.

### Security Groups

O Security Group do pfSense foi tratado de maneira mais permissiva para facilitar a função de firewall/router do laboratório.

Essa abordagem não deve ser interpretada como recomendação de segurança para produção.

### Dados históricos

O ambiente foi posteriormente excluído e não foram mantidas screenshots da implementação.

Por isso, algumas regras de firewall, Security Groups e tabelas de rotas apresentadas neste documento são **reconstruções técnicas de referência**, baseadas no cenário realizado e não cópias exatas do ambiente original.

---

# 🚀 Considerações para produção

Em um ambiente produtivo, a arquitetura deveria considerar, conforme o contexto:

- utilização dos dois túneis da VPN;
- redundância do firewall;
- múltiplas Availability Zones quando aplicável;
- monitoramento da VPN;
- logs;
- CloudWatch;
- controle rigoroso de Security Groups;
- princípio do menor privilégio;
- segmentação de redes;
- ausência de exposição pública desnecessária;
- monitoramento de disponibilidade;
- documentação das rotas;
- processo formal de mudança;
- gerenciamento seguro de credenciais;
- rotação de chaves e segredos;
- políticas de acesso administrativo.

---

# 🔐 Segurança das informações

**Nunca publique no GitHub:**

- Pre-Shared Keys;
- credenciais AWS;
- Access Keys;
- Secret Keys;
- tokens;
- senhas;
- ARNs desnecessários;
- IDs internos quando não forem necessários;
- IPs públicos que ainda estejam em uso;
- arquivos de configuração contendo segredos.

A configuração exportada utilizada neste laboratório continha **PSKs reais da VPN**. Esses valores foram deliberadamente omitidos desta documentação.

Como o ambiente original foi destruído, os valores históricos não fazem parte deste repositório.

---

# 📁 Estrutura do projeto

```text
aws-vpn-site-to-site-lab/
│
├── README.md
│
├── docs/
│   ├── arquitetura.md
│   ├── configuracao-vpn.md
│   ├── roteamento.md
│   ├── firewall.md
│   ├── security-groups.md
│   └── troubleshooting.md
│
├── diagrams/
│   └── arquitetura.txt
│
└── screenshots/
    └── README.md
```

---

# 📚 Documentação complementar

- [Arquitetura](docs/arquitetura.md)
- [Configuração VPN IPsec](docs/configuracao-vpn.md)
- [Roteamento](docs/roteamento.md)
- [Firewall](docs/firewall.md)
- [Security Groups](docs/security-groups.md)
- [Troubleshooting](docs/troubleshooting.md)

---

# 🎯 Resultado final

O laboratório demonstrou com sucesso a criação de uma conectividade híbrida simulada entre:

```text
┌─────────────────────────────┐
│  ON-PREMISES SIMULADO       │
│                             │
│  10.10.10.0/24              │
│         │                   │
│      pfSense                │
└─────────┼───────────────────┘
          │
          │ IPsec VPN
          │
┌─────────┼───────────────────┐
│         │                   │
│  AWS    │                   │
│                             │
│  10.20.10.0/24              │
│                             │
│  Ubuntu                     │
└─────────────────────────────┘
```

Os testes de **PING** e **SSH** foram realizados com sucesso, validando a comunicação entre os ambientes.

---

## 💡 Objetivo de aprendizado

O principal objetivo deste laboratório não foi apenas estabelecer uma VPN, mas compreender o caminho completo que um pacote percorre em uma arquitetura híbrida:

```text
Host
 ↓
Gateway
 ↓
Route Table
 ↓
Firewall / Security Group
 ↓
VPN
 ↓
Virtual Private Gateway
 ↓
Route Table
 ↓
Host de destino
```

Esse entendimento é fundamental para diagnosticar problemas de conectividade em ambientes AWS híbridos.
