# Comandos e Validações — Rede Corporativa Cisco

## 1. VLANs

Foram criadas quatro VLANs para segmentação da rede corporativa:

| VLAN | Nome | Rede |
|------|------|------|
| 10 | ADMINISTRACAO | 192.168.10.0/24 |
| 20 | TI | 192.168.20.0/24 |
| 30 | FINANCEIRO | 192.168.30.0/24 |
| 40 | VISITANTES | 192.168.40.0/24 |

### Comandos utilizados

```cisco
vlan 10
name ADMINISTRACAO

vlan 20
name TI

vlan 30
name FINANCEIRO

vlan 40
name VISITANTES

Validação:

show vlan brief


## 2. Portas de acesso

As portas destinadas aos computadores foram configuradas como portas de acesso e associadas às respectivas VLANs.

SW1
interface fa0/1
switchport mode access
switchport access vlan 10

interface fa0/2
switchport mode access
switchport access vlan 20

interface fa0/3
switchport mode access
switchport access vlan 30

interface fa0/4
switchport mode access
switchport access vlan 40

A mesma lógica de segmentação foi aplicada ao SW2.


## 3. Trunk entre os switches

O enlace entre SW1 e SW2 foi configurado como trunk para transportar múltiplas VLANs.

Os enlaces físicos utilizados foram:

SW1 Fa0/5 ↔ SW2 Fa0/5
SW1 Fa0/6 ↔ SW2 Fa0/6

Posteriormente, os dois enlaces foram agregados em um EtherChannel.

Validação: 

show interfaces trunk


## 4. EtherChannel com LACP

Foi configurado um EtherChannel utilizando LACP para agregar os enlaces entre SW1 e SW2.

SW1 e SW2
interface range fa0/5-6
channel-group 1 mode active

O Port-channel foi configurado como trunk:

interface port-channel 1
switchport mode trunk

Validação:

show etherchannel summary

O objetivo foi verificar a formação do Port-channel e o funcionamento do agrupamento dos enlaces físicos.


## 5. Router-on-a-Stick

O roteador R1 foi utilizado para realizar o roteamento entre as VLANs através de subinterfaces.

Interface física utilizada:

R1 Gi0/0/0 ↔ SW1 Gi0/1

Subinterfaces
interface gigabitEthernet 0/0/0
no shutdown

interface gigabitEthernet 0/0/0.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0

interface gigabitEthernet 0/0/0.20
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.0

interface gigabitEthernet 0/0/0.30
encapsulation dot1Q 30
ip address 192.168.30.1 255.255.255.0

interface gigabitEthernet 0/0/0.40
encapsulation dot1Q 40
ip address 192.168.40.1 255.255.255.0

Validação:

show ip interface brief

Foi verificado que as subinterfaces estavam ativas e associadas aos respectivos endereços IP.

## 6. DHCP

O roteador R1 foi configurado como servidor DHCP para fornecer endereçamento automático aos dispositivos de cada VLAN.

Exclusão de endereços

Foram reservados os primeiros endereços de cada rede para gateways e possíveis endereços estáticos:

ip dhcp excluded-address 192.168.10.1 192.168.10.10
ip dhcp excluded-address 192.168.20.1 192.168.20.10
ip dhcp excluded-address 192.168.30.1 192.168.30.10
ip dhcp excluded-address 192.168.40.1 192.168.40.10

Pools:

ip dhcp pool ADMINISTRACAO
network 192.168.10.0 255.255.255.0
default-router 192.168.10.1
dns-server 8.8.8.8

ip dhcp pool TI
network 192.168.20.0 255.255.255.0
default-router 192.168.20.1
dns-server 8.8.8.8

ip dhcp pool FINANCEIRO
network 192.168.30.0 255.255.255.0
default-router 192.168.30.1
dns-server 8.8.8.8

ip dhcp pool VISITANTES
network 192.168.40.0 255.255.255.0
default-router 192.168.40.1
dns-server 8.8.8.8

Validação:

show ip dhcp binding

Os computadores foram configurados para obter seus endereços automaticamente e receberam endereços pertencentes às respectivas VLANs.


## 7. Spanning Tree Protocol

O STP foi utilizado para fornecer uma topologia lógica livre de loops na camada 2.

O SW1 foi definido como Root Bridge para as VLANs do projeto:

spanning-tree vlan 10,20,30,40 root primary

Validação:

show spanning-tree

Foi verificado que o SW1 assumiu a função de Root Bridge e que o SW2 utilizou o Port-channel como caminho em direção ao Root Bridge.


## 8. Hardening básico

Foram aplicadas algumas boas práticas de segurança nos switches.

VLAN nativa dedicada

Foi criada uma VLAN específica para a VLAN nativa:

vlan 999
name NATIVE-BLACKHOLE

A VLAN 999 foi utilizada como VLAN nativa no trunk entre os switches.

Portas não utilizadas)

Foi criada uma VLAN separada para portas sem utilização:

vlan 1000
name UNUSED-PORTS

As interfaces sem uso foram associadas à VLAN 1000 e administrativamente desativadas:

interface range fa0/7-24
switchport mode access
switchport access vlan 1000
shutdown

Validação:

show vlan brief
show interfaces status


## 9. Testes de conectividade

Após a configuração, foram realizados testes de conectividade entre os dispositivos.

Foram verificados:

Comunicação entre computadores da mesma VLAN;
Comunicação entre diferentes VLANs;
Conectividade com os gateways;
Funcionamento do DHCP;
Comunicação através do EtherChannel;
Funcionamento do roteamento inter-VLAN.

Os testes de ping confirmaram a conectividade esperada entre as redes.


## 10. Troubleshooting

Foi simulada uma falha de configuração alterando a porta do computador da Administração para a VLAN 20.

Problema)
O computador que deveria pertencer à VLAN 10 passou a receber um endereço da rede:

192.168.20.0/24

e utilizava:

Gateway: 192.168.20.1

Como consequência, o equipamento deixou de operar na rede de Administração.

Diagnóstico

A análise do endereço IP recebido e da configuração da porta permitiu identificar que a interface estava associada à VLAN incorreta.

Correção)
A porta foi novamente associada à VLAN 10:

interface fa0/1
switchport access vlan 10

Após a correção, o computador voltou a receber um endereço da rede:

192.168.10.0/24

e a conectividade foi restaurada.


## 11. Principais comandos de validação:

VLANs
show vlan brief

Trunks
show interfaces trunk

EtherChannel
show etherchannel summary

STP
show spanning-tree

Interfaces e endereçamento
show ip interface brief

Tabela de roteamento
show ip route

DHCP
show ip dhcp binding

Estado das portas
show interfaces status


## 12. Resultado

A rede foi implementada e validada no Cisco Packet Tracer, incluindo:

Segmentação por VLAN;
Trunks 802.1Q;
EtherChannel com LACP;
Roteamento inter-VLAN com Router-on-a-Stick;
DHCP centralizado;
STP;
Hardening básico;
Troubleshooting de uma falha de VLAN;
Testes de conectividade ponta a ponta.