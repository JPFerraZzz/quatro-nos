# Rascunho anterior do pacote

**Autor original:** Duarte Sousa
**Data indicada:** 10/11/2026
**Estado:** cópia de referência do texto colado. O autor descreveu-o como cortado durante a secção 7; o material colado nesta conversa também incluía apontamentos das secções 8 e 9. Tudo estava marcado como não testado.

> PACOTE PARA O JOAO - v1 - 10/11/2026
> Autor: Duarte (Analista e Arquiteto)
>
> ESTADO: TUDO NAO TESTADO. Modelos, nomes de portas e comandos podem precisar de ajuste no Packet Tracer. Se um comando for rejeitado, copia o erro e envia ao Duarte. Guardar capturas de ecra das verificacoes (sec. 8) para o relatorio.
>
> Marcadores a substituir (escolher e nao pôr no relatorio): `<CHAVE-IPSEC>` `<CHAVE-RADIUS>` `<PASSWORD-ADMIN>` `<PSK-WIFI>`

## 1. Ordem de trabalho

1. Criar equipamentos (sec. 3) e cabos (sec. 5), usando o cabo automático.
2. Configurar SW1 e SW2 de cada nó.
3. Configurar CENTRAL-SW1, servidores e DHCP.
4. Configurar INET-RTR e as duas ASA.
5. Fazer as verificações da sec. 8.
6. Só no fim configurar AAA (sec. 6.8).

## 2. Decisões

- D1: túnel IPsec nas ASA (Plano A), para reduzir o número de equipamentos; risco de compatibilidade da ASA do Packet Tracer por confirmar.
- D2: cifrar todo o tráfego nó-central com ACL cripto ampla. Se o túnel cair, o nó fica isolado.
- D3: CENTRAL-FW1 tem uma interface por nó, com nível de segurança 10, para evitar hairpin na mesma interface; requer `same-security-traffic`.
- D4: DHCP `.11` a `.254`, ou 244 endereços por VLAN; `.2` a `.10` ficam reservados. Duas VLANs dão 488 endereços, 39% acima de 350, assumindo divisão 175/175.
- D5: AP em porta access VLAN 20 e TabletPC como cliente Wi-Fi.
- D6: Internet simulada `198.51.100.0/24`, com INET-RTR, INET-SW1, INET-SRV e INET-PC1.
- D7: segredos substituídos localmente, nunca publicados; AAA planeado nos switches com fallback local.

## 3. Equipamentos

Por nó N = 1 a 4 (oito por nó): `N<n>-FW1` ASA 5506-X; `N<n>-SW1` Catalyst 3650-24PS L3; `N<n>-SW2` Catalyst 2960-24TT; `N<n>-AP1` AccessPoint-PT; `N<n>-PC1`, `N<n>-PC2` PC-PT; `N<n>-PRN1` Printer-PT; `N<n>-TAB1` TabletPC-PT.

Central: `CENTRAL-FW1` ASA 5506-X; `CENTRAL-SW1` Catalyst 3650-24PS; `CENTRAL-SRV-DNS`, `CENTRAL-SRV-DHCP`, `CENTRAL-SRV-DC`, `CENTRAL-SRV-INTRA`, `CENTRAL-SRV-EXTRA` Server-PT.

Internet simulada: `INET-RTR` 2911; `INET-SW1` 2960-24TT; `INET-SRV` Server-PT; `INET-PC1` PC-PT. Total planeado: 43 equipamentos. O número de hosts Packet Tracer é representativo, não 350 PCs por nó.

## 4. Endereçamento recebido

Cada nó N usa o bloco `10.N.0.0/16`: VLAN 10 cabo `10.N.10.0/24`; VLAN 20 Wi-Fi `10.N.20.0/24`; VLAN 99 gestão `10.N.99.0/27`; trânsito switch-firewall `10.N.255.0/30`; WAN `172.16.N.0/30`. Gateways das VLANs `.1`; WAN central `.1` e nó `.2`.

O central usa `10.0.0.0/16`: VLAN 30 servidores `10.0.30.0/24`, gateway `.1`; VLAN 40 DMZ `10.0.40.0/24`, gateway `.1`; VLAN 99 gestão `10.0.99.0/27`; trânsito switch-firewall `10.0.255.0/30`. Exterior `203.0.113.0/30`; Internet simulada `198.51.100.0/24`.

VLAN 99: switch L3 `.1`, switch acesso do nó `.2`. Trânsito switch-firewall: switch `.1`, firewall `.2`. Impressora N1 `10.1.10.5/24`, gateway `10.1.10.1`. PCs e tablet por DHCP.

Servidores: DNS `10.0.30.10`; DHCP `.11`; DC/AAA `.12`; INTRA `.13`; EXTRA `10.0.40.10`. INET-RTR exterior `203.0.113.1`; CENTRAL-FW1 exterior `.2`; INET-RTR LAN `198.51.100.1`; INET-SRV `.10`, INET-PC1 `.20`. DNS `10.0.30.10`.

Pools DHCP no servidor central, oito no total: N1 a N4, cada um com CABO e WIFI, gateway correspondente `10.N.10.1` ou `10.N.20.1`, endereço inicial `.11`, máscara `/24`, DNS `10.0.30.10`, máximo 244 clientes.

Registos DNS provisórios: `dns.empresa.lan` 10.0.30.10; `dhcp.empresa.lan` 10.0.30.11; `dc.empresa.lan` 10.0.30.12; `intranet.empresa.lan` 10.0.30.13; `extranet.empresa.lan` 10.0.40.10.

## 5. Ligações

Por nó N: `N<n>-FW1 Gi1/1` a `CENTRAL-FW1 Gi1/<n>`; `N<n>-FW1 Gi1/2` a `N<n>-SW1 Gi1/0/1`; `N<n>-SW1 Gi1/0/2` a `N<n>-SW2 Gi0/1`; `N<n>-SW1 Gi1/0/3` a `N<n>-AP1 Port 0`; `N<n>-SW2 Fa0/1` a `N<n>-PC1 Fa0`; Fa0/2 a PC2; Fa0/3 a PRN1. TAB1 associa-se sem cabo.

Central: `CENTRAL-FW1 Gi1/5` a `INET-RTR Gi0/0`; `CENTRAL-FW1 Gi1/6` a `CENTRAL-SW1 Gi1/0/1`; portas Gi1/0/2-5 do switch central a DNS, DHCP, DC e INTRA em VLAN 30; Gi1/0/6 a EXTRA em VLAN 40.

Internet: `INET-RTR Gi0/1` a `INET-SW1 Gi0/1`; `INET-SW1 Fa0/1` a INET-SRV; Fa0/2 a INET-PC1.

## 6. Configuração antiga, não validada

### 6.1 N1-SW1

```text
hostname N1-SW1
ip routing
vlan 10
 name CABO
vlan 20
 name WIFI
vlan 99
 name GESTAO
ip access-list extended MGMT-BLOCK
 deny ip any 10.1.99.0 0.0.0.31
 permit ip any any
interface GigabitEthernet1/0/1
 description PARA-FW1
 no switchport
 ip address 10.1.255.1 255.255.255.252
 no shutdown
interface GigabitEthernet1/0/2
 description TRUNK-SW2
 switchport mode trunk
 switchport trunk allowed vlan 10,20,99
interface GigabitEthernet1/0/3
 description AP1
 switchport mode access
 switchport access vlan 20
interface Vlan10
 ip address 10.1.10.1 255.255.255.0
 ip helper-address 10.0.30.11
 ip access-group MGMT-BLOCK in
 no shutdown
interface Vlan20
 ip address 10.1.20.1 255.255.255.0
 ip helper-address 10.0.30.11
 ip access-group MGMT-BLOCK in
 no shutdown
interface Vlan99
 ip address 10.1.99.1 255.255.255.224
 no shutdown
ip route 0.0.0.0 0.0.0.0 10.1.255.2
```

Se necessário, tentar `switchport trunk encapsulation dot1q` antes de configurar trunk.

### 6.2 N1-SW2

```text
hostname N1-SW2
vlan 10
 name CABO
vlan 99
 name GESTAO
interface GigabitEthernet0/1
 description TRUNK-SW1
 switchport mode trunk
 switchport trunk allowed vlan 10,99
interface range FastEthernet0/1-24
 switchport mode access
 switchport access vlan 10
interface Vlan99
 ip address 10.1.99.2 255.255.255.224
 no shutdown
ip default-gateway 10.1.99.1
```

### 6.3 N1-FW1, Plano A

```text
hostname N1-FW1
interface GigabitEthernet1/1
 nameif outside
 security-level 0
 ip address 172.16.1.2 255.255.255.252
 no shutdown
interface GigabitEthernet1/2
 nameif inside
 security-level 100
 ip address 10.1.255.2 255.255.255.252
 no shutdown
route outside 0.0.0.0 0.0.0.0 172.16.1.1
route inside 10.1.0.0 255.255.0.0 10.1.255.1
access-list FROM-TUNNEL extended deny ip any 10.1.99.0 255.255.255.224
access-list FROM-TUNNEL extended permit ip 10.0.0.0 255.0.0.0 any
access-group FROM-TUNNEL in interface outside
no sysopt connection permit-vpn
policy-map global_policy
 class inspection_default
  inspect icmp
crypto ikev1 policy 10
 authentication pre-share
 encryption aes
 hash sha
 group 2
 lifetime 86400
crypto ipsec ikev1 transform-set TS-EMPRESA esp-aes esp-sha-hmac
access-list VPN-CENTRAL extended permit ip 10.1.0.0 255.255.0.0 any
crypto map CMAP 10 match address VPN-CENTRAL
crypto map CMAP 10 set peer 172.16.1.1
crypto map CMAP 10 set ikev1 transform-set TS-EMPRESA
crypto map CMAP interface outside
crypto ikev1 enable outside
tunnel-group 172.16.1.1 type ipsec-l2l
tunnel-group 172.16.1.1 ipsec-attributes
 ikev1 pre-shared-key <CHAVE-IPSEC>
```

Nos nós 2-4, trocar os endereços e o hostname pelo número do nó. O rascunho propunha sintaxe antiga IKE/ISAKMP como alternativa, sem validação.

### 6.4 CENTRAL-SW1

```text
hostname CENTRAL-SW1
ip routing
vlan 30
 name SERVIDORES
vlan 40
 name DMZ
vlan 99
 name GESTAO
ip access-list extended DMZ-IN
 permit tcp 10.0.40.0 0.0.0.255 eq 80 10.0.0.0 0.255.255.255
 permit icmp 10.0.40.0 0.0.0.255 10.0.0.0 0.255.255.255 echo-reply
 deny ip 10.0.40.0 0.0.0.255 10.0.0.0 0.255.255.255
 permit ip any any
interface GigabitEthernet1/0/1
 no switchport
 ip address 10.0.255.1 255.255.255.252
 no shutdown
interface range GigabitEthernet1/0/2-5
 switchport mode access
 switchport access vlan 30
interface GigabitEthernet1/0/6
 switchport mode access
 switchport access vlan 40
interface Vlan30
 ip address 10.0.30.1 255.255.255.0
 no shutdown
interface Vlan40
 ip address 10.0.40.1 255.255.255.0
 ip access-group DMZ-IN in
 no shutdown
interface Vlan99
 ip address 10.0.99.1 255.255.255.224
 no shutdown
ip route 0.0.0.0 0.0.0.0 10.0.255.2
```

### 6.5 CENTRAL-FW1, Plano A

```text
hostname CENTRAL-FW1
same-security-traffic permit inter-interface
interface GigabitEthernet1/1
 nameif WAN1
 security-level 10
 ip address 172.16.1.1 255.255.255.252
 no shutdown
interface GigabitEthernet1/2
 nameif WAN2
 security-level 10
 ip address 172.16.2.1 255.255.255.252
 no shutdown
interface GigabitEthernet1/3
 nameif WAN3
 security-level 10
 ip address 172.16.3.1 255.255.255.252
 no shutdown
interface GigabitEthernet1/4
 nameif WAN4
 security-level 10
 ip address 172.16.4.1 255.255.255.252
 no shutdown
interface GigabitEthernet1/5
 nameif outside
 security-level 0
 ip address 203.0.113.2 255.255.255.252
 no shutdown
interface GigabitEthernet1/6
 nameif inside
 security-level 100
 ip address 10.0.255.2 255.255.255.252
 no shutdown
route outside 0.0.0.0 0.0.0.0 203.0.113.1
route inside 10.0.0.0 255.255.0.0 10.0.255.1
route WAN1 10.1.0.0 255.255.0.0 172.16.1.2
route WAN2 10.2.0.0 255.255.0.0 172.16.2.2
route WAN3 10.3.0.0 255.255.0.0 172.16.3.2
route WAN4 10.4.0.0 255.255.0.0 172.16.4.2
object network OBJ-PRIVADA
 subnet 10.0.0.0 255.0.0.0
 nat (any,outside) dynamic interface
object network OBJ-EXTRANET
 host 10.0.40.10
 nat (inside,outside) static interface service tcp 80 80
access-list FROM-NODES extended deny ip any 10.0.99.0 255.255.255.224
access-list FROM-NODES extended deny ip any 10.1.99.0 255.255.255.224
access-list FROM-NODES extended deny ip any 10.2.99.0 255.255.255.224
access-list FROM-NODES extended deny ip any 10.3.99.0 255.255.255.224
access-list FROM-NODES extended deny ip any 10.4.99.0 255.255.255.224
access-list FROM-NODES extended permit ip 10.0.0.0 255.0.0.0 any
access-group FROM-NODES in interface WAN1
access-group FROM-NODES in interface WAN2
access-group FROM-NODES in interface WAN3
access-group FROM-NODES in interface WAN4
access-list FROM-OUTSIDE extended permit tcp any host 10.0.40.10 eq 80
access-group FROM-OUTSIDE in interface outside
no sysopt connection permit-vpn
crypto ikev1 policy 10
 authentication pre-share
 encryption aes
 hash sha
 group 2
 lifetime 86400
crypto ipsec ikev1 transform-set TS-EMPRESA esp-aes esp-sha-hmac
```

Para cada N, a configuração antiga acrescentava ACL cripto `permit ip any 10.N.0.0 255.255.0.0`, um crypto map na WAN N, peer `172.16.N.2`, ativação IKEv1 nessa WAN e tunnel-group com chave partilhada local. A chave não deve ser publicada.

### 6.6 INET-RTR

```text
hostname INET-RTR
interface GigabitEthernet0/0
 ip address 203.0.113.1 255.255.255.252
 no shutdown
interface GigabitEthernet0/1
 ip address 198.51.100.1 255.255.255.0
 no shutdown
```

### 6.7 Servidores, AP e clientes

Configurar IPs dos Server-PT em Config > FastEthernet0; ativar DNS em CENTRAL-SRV-DNS, DHCP e oito pools em CENTRAL-SRV-DHCP, AAA em CENTRAL-SRV-DC, HTTP em INTRA e EXTRA. Adicionar clientes RADIUS dos switches com endereços 10.N.255.1 e 10.0.255.1. Configurar SSID `EMPRESA-N<n>` e WPA2-PSK no AP; associar o tablet; PCs por DHCP; impressora estática. INET-SRV e INET-PC1 usam `198.51.100.10` e `.20`, gateway `.1`.

### 6.8 AAA (só no fim)

O rascunho previa, nos switches 3650:

```text
ip domain-name empresa.lan
username admin privilege 15 secret <PASSWORD-ADMIN>
radius-server host 10.0.30.12 key <CHAVE-RADIUS>
aaa new-model
aaa authentication login default group radius local
```

Nos switches 2960, previa apenas utilizador admin local. Credenciais e chaves são marcadores e não devem ser publicadas.

## 7. Plano B proposto no rascunho

Usar só se os comandos `show crypto isakmp sa` e `show crypto ipsec sa` não mostrarem associações. O rascunho sugeria routers 2911 adicionais nos nós e dois routers centrais, ativação da licença `securityk9` e túneis IPsec nos routers. Indicava redes novas `10.N.254.0/30`, `10.0.254.0/30` e `10.0.253.0/30`, e a WAN `172.16.N.0/30` como ligação router-router.

**Aviso:** este plano estava incompleto e a própria nota indicava que N1-N2 e N3-N4 poderiam deixar de atravessar CENTRAL-FW1. Não é uma configuração aprovada nem deve ser aplicada sem redesenho.

## 8. Verificações rápidas recebidas

Registar PASSOU/FALHOU e guardar capturas após execução:

1. RF1/RNF1: em N1-FW1, `show crypto isakmp sa` e `show crypto ipsec sa`; esperar associações ativas e contadores a subir.
2. RF1: N1-PC1 faz ping a PCs dos nós 2, 3 e 4 e a CENTRAL-SRV-DNS; em Simulation mode, observar passagem por CENTRAL-FW1.
3. RF2/RF3: N1-PC1 recebe `10.1.10.11` a `.254`, N1-TAB1 `10.1.20.11` a `.254`; repetir nos quatro nós.
4. RF4: ping de N1-PC1 a `10.1.99.1` e `10.2.99.1` deve falhar.
5. RF5: ping de N1-PC1 a `198.51.100.10` deve funcionar; verificar `show xlate` no central.
6. RF5: abrir intranet e extranet por nome; de INET-PC1 abrir `http://203.0.113.2`.
7. DMZ: CENTRAL-SRV-EXTRA não deve iniciar ping para `10.0.30.10`.
8. Domínio: `nslookup dc.empresa.lan` e ping ao DC.
9. AAA: login de teste no N1-SW1; com DC desligado, fallback local.

## 9. Limitações indicadas

- Packet Tracer não tem Active Directory real; DNS + AAA representam parte do desenho.
- DHCP centralizado não atribui novos endereços quando o túnel está em baixo.
- ACLs nos switches são sem estado; a ACL da DMZ tem de permitir respostas necessárias.
- Redundância é Should e continua por decidir.
- “Modelos de BI” está por esclarecer com o professor.

**Nota sobre a versão atual:** este rascunho foi guardado como referência. Há correções e decisões revistas no `README.md` e nos ficheiros de comandos atuais. O pacote atual também está totalmente por testar.
