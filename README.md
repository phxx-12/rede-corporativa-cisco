# Rede Corporativa Cisco — Packet Tracer

> Projeto prático de infraestrutura de redes desenvolvido no Cisco Packet Tracer, aplicando conceitos de segmentação, switching, roteamento, redundância, DHCP, STP, hardening e troubleshooting.

## 📌 Sobre o projeto

Este projeto consiste na implementação de uma rede corporativa segmentada em diferentes setores, desenvolvida do zero no Cisco Packet Tracer.

O laboratório foi construído com o objetivo de colocar em prática conceitos estudados em Redes e Infraestrutura, incluindo VLANs, trunking 802.1Q, EtherChannel com LACP, roteamento inter-VLAN, DHCP, Spanning Tree Protocol e boas práticas básicas de segurança.

Além da implementação, foram realizados testes de conectividade e um cenário de troubleshooting para simular e corrigir uma falha de configuração de VLAN.

---

## 🏗️ Topologia

A rede é composta por:

- 1 roteador Cisco (R1)
- 2 switches Cisco (SW1 e SW2)
- Computadores distribuídos entre diferentes VLANs
- EtherChannel entre os switches
- Router-on-a-Stick para roteamento inter-VLAN

### Segmentação da rede

| VLAN | Setor | Rede | Gateway |
|---|---|---|---|
| 10 | Administração | 192.168.10.0/24 | 192.168.10.1 |
| 20 | TI | 192.168.20.0/24 | 192.168.20.1 |
| 30 | Financeiro | 192.168.30.0/24 | 192.168.30.1 |
| 40 | Visitantes | 192.168.40.0/24 | 192.168.40.1 |

---

## 🔧 Tecnologias e conceitos aplicados

- Cisco Packet Tracer
- VLANs
- Trunk 802.1Q
- EtherChannel
- LACP
- Port-Channel
- Router-on-a-Stick
- Inter-VLAN Routing
- DHCP
- Spanning Tree Protocol (STP)
- Network Hardening
- Troubleshooting
- IPv4

---

## 1. Segmentação por VLAN

Foram criadas quatro VLANs para separar logicamente os diferentes setores da organização:

- **VLAN 10 — Administração**
- **VLAN 20 — TI**
- **VLAN 30 — Financeiro**
- **VLAN 40 — Visitantes**

As portas destinadas aos computadores foram configuradas como portas de acesso e associadas às respectivas VLANs.

### Validação
show vlan brief

2. Trunk 802.1Q

O enlace entre SW1 e SW2 foi configurado como trunk para permitir o transporte de múltiplas VLANs entre os switches.

Os enlaces físicos utilizados foram:

SW1 Fa0/5 ↔ SW2 Fa0/5
SW1 Fa0/6 ↔ SW2 Fa0/6

Posteriormente, esses enlaces foram agregados utilizando EtherChannel.

Validação
show interfaces trunk
3. EtherChannel com LACP

Foi configurado um EtherChannel entre SW1 e SW2 utilizando LACP.

Os enlaces físicos foram agrupados no:

Port-channel 1

A utilização do EtherChannel permite tratar os enlaces físicos como uma interface lógica, proporcionando agregação de links e maior resiliência caso um dos enlaces físicos apresente falha.

Configuração
interface range fa0/5-6
channel-group 1 mode active

interface port-channel 1
switchport mode trunk

Validação
show etherchannel summary
4. Router-on-a-Stick

O roteamento entre as VLANs foi implementado utilizando Router-on-a-Stick.

O roteador R1 utiliza subinterfaces na interface:

Gi0/0/0

Cada subinterface foi associada a uma VLAN através do encapsulamento 802.1Q.

Subinterface	VLAN	Endereço
Gi0/0/0.10	10	192.168.10.1/24
Gi0/0/0.20	20	192.168.20.1/24
Gi0/0/0.30	30	192.168.30.1/24
Gi0/0/0.40	40	192.168.40.1/24

Exemplo
interface gigabitEthernet 0/0/0.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0

Validação
show ip interface brief
show ip route

5. DHCP

O roteador R1 foi configurado como servidor DHCP para fornecer automaticamente endereços IP aos dispositivos das quatro VLANs.

Foram criados pools independentes para cada rede, contendo:

Rede
Gateway padrão
Servidor DNS

Também foram excluídos os primeiros endereços de cada rede para reservar os gateways e possíveis endereços estáticos.

Validação
show ip dhcp binding

Os computadores foram configurados para utilizar DHCP e receberam endereços pertencentes às respectivas redes.

6. Spanning Tree Protocol

O STP foi utilizado para manter uma topologia lógica livre de loops na camada 2.

O SW1 foi configurado como Root Bridge para as VLANs utilizadas no projeto.

spanning-tree vlan 10,20,30,40 root primary

Validação
show spanning-tree

Foi verificado o papel dos switches e das portas na topologia STP, incluindo Root Bridge, Root Port e Designated Ports.

7. Hardening

Foram aplicadas algumas boas práticas básicas de segurança nos switches.

VLAN nativa dedicada

Foi criada uma VLAN específica para utilização como VLAN nativa:

VLAN 999 — NATIVE-BLACKHOLE
Portas não utilizadas

Também foi criada uma VLAN específica para portas sem utilização:

VLAN 1000 — UNUSED-PORTS

As portas sem uso foram associadas a essa VLAN e administrativamente desativadas.

interface range fa0/7-24
switchport mode access
switchport access vlan 1000
shutdown

Validação
show vlan brief
show interfaces status

8. Troubleshooting

Além da configuração da rede, foi realizado um cenário prático de troubleshooting.

Uma porta que deveria estar associada à VLAN 10 — Administração foi propositalmente configurada na VLAN 20 — TI.

Como consequência, o computador recebeu um endereço da rede:

192.168.20.0/24

em vez da rede:

192.168.10.0/24
Diagnóstico

A análise do endereço IP, gateway e configuração da porta permitiu identificar que o equipamento estava associado à VLAN incorreta.

Correção

A porta foi novamente associada à VLAN 10:

interface fa0/1
switchport access vlan 10

Após a correção, o computador voltou a receber um endereço da rede correta e a conectividade foi restaurada.

🧪 Validações realizadas

Durante o desenvolvimento foram utilizados diversos comandos de verificação:

show vlan brief
show interfaces trunk
show etherchannel summary
show spanning-tree
show ip interface brief
show ip route
show ip dhcp binding
show interfaces status

Também foram realizados testes de ping para verificar:

Conectividade com os gateways;
Comunicação entre diferentes VLANs;
Funcionamento do roteamento inter-VLAN;
Funcionamento do DHCP;
Conectividade após correção do cenário de troubleshooting.

## 📸 Evidências

### Topologia final

![Topologia final]
(imagens/topologia-final.png)

### EtherChannel e LACP

![EtherChannel e LACP]
(imagens/etherchannel-lacp.png)

### Trunk 802.1Q

![Trunk 802.1Q]
(imagens/trunk.png)

### Router-on-a-Stick

![Router-on-a-Stick]
(imagens/router-on-a-stick.png)

### Interfaces do R1

![Interfaces do R1]
(imagens/interfaces-r1.png)

### DHCP

![DHCP]
(imagens/dhcp-pools.png)

### Validação de conectividade

![Ping]
(imagens/ping-vlan-10.png)

### Hardening

![Hardening]
(imagens/hardening.png)

📂 Estrutura do projeto
rede-corporativa-cisco/
│
├── README.md
│
├── packet-tracer/
│   └── rede-corporativa-cisco.pkt
│
├── imagens/
│   ├── topologia-final.png
│   ├── etherchannel-lacp.png
│   ├── router-on-a-stick-dhcp.png
│   ├── hardening.png
│   └── troubleshooting.png
│
└── documentacao/
    └── comandos-e-validacoes.md

🎯 Objetivos de aprendizagem

Com este laboratório, pratiquei:

Segmentação de redes utilizando VLANs;
Configuração de trunks 802.1Q;
Agregação de links com EtherChannel/LACP;
Roteamento inter-VLAN;
Configuração de Router-on-a-Stick;
Implementação de DHCP;
Funcionamento do STP;
Aplicação de hardening básico em switches;
Análise de problemas de conectividade;
Troubleshooting de configuração de VLANs;
Validação de redes utilizando comandos Cisco IOS.

🛠️ Ambiente

Ferramenta: Cisco Packet Tracer

Área: Redes e Infraestrutura

Tipo: Laboratório prático / Projeto de portfólio