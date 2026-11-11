# Plano de endereçamento v1

**Autor:** Duarte Sousa
**Papel:** Analista e Arquiteto
**Versão:** 1.0
**Data:** 11/11/2026
**Estado:** proposta para implementação; por testar no Packet Tracer.

## Dimensionamento

Uma rede `/24` tem `2^8 - 2 = 254` endereços de host utilizáveis. Reservando `.1` a `.10` para gateway e infraestrutura, ficam 244 endereços de DHCP (`.11` a `.254`) por VLAN de utilizadores.

Cada nó tem duas VLANs de utilizadores: cabo e Wi-Fi. A capacidade DHCP total é `2 × 244 = 488` endereços. Face a 350 equipamentos, a margem é `(488 - 350) / 350 ≈ 39%`, superior aos 25% pedidos.

Esta capacidade pressupõe uma distribuição máxima de 195 equipamentos por tipo de acesso: `244 / 1,25 = 195,2`. A divisão inicial assumida é 175 cablados e 175 Wi-Fi. Se uma VLAN precisar de mais de 195 equipamentos, deve ser redimensionada para `/23` e o plano DHCP atualizado.

Uma única `/23` teria 510 endereços utilizáveis, mas foi rejeitada porque misturaria cabo e Wi-Fi no mesmo domínio de broadcast e dificultaria o isolamento.

## Blocos e sub-redes

| Rede | Nó 1 | Nó 2 | Nó 3 | Nó 4 | Central |
|---|---|---|---|---|---|
| Bloco sumarizável | 10.1.0.0/16 | 10.2.0.0/16 | 10.3.0.0/16 | 10.4.0.0/16 | 10.0.0.0/16 |
| VLAN 10, cabo | 10.1.10.0/24 | 10.2.10.0/24 | 10.3.10.0/24 | 10.4.10.0/24 | — |
| VLAN 20, Wi-Fi | 10.1.20.0/24 | 10.2.20.0/24 | 10.3.20.0/24 | 10.4.20.0/24 | — |
| VLAN 30, servidores | — | — | — | — | 10.0.30.0/24 |
| VLAN 40, DMZ | — | — | — | — | 10.0.40.0/24 |
| VLAN 99, gestão | 10.1.99.0/27 | 10.2.99.0/27 | 10.3.99.0/27 | 10.4.99.0/27 | 10.0.99.0/27 |
| Trânsito switch-firewall | 10.1.255.0/30 | 10.2.255.0/30 | 10.3.255.0/30 | 10.4.255.0/30 | 10.0.255.0/30 |
| WAN nó-central | 172.16.1.0/30 | 172.16.2.0/30 | 172.16.3.0/30 | 172.16.4.0/30 | — |

Máscaras: `/24` = `255.255.255.0`; `/27` = `255.255.255.224` (30 endereços utilizáveis); `/30` = `255.255.255.252` (2 endereços utilizáveis). O gateway de cada VLAN é `.1`. Na WAN, a firewall central usa `.1` e a firewall do nó usa `.2`.

## Endereços fixos

| Equipamento/interface | Endereço |
|---|---|
| Nó N, switch L3, VLAN 10 | 10.N.10.1/24 |
| Nó N, switch L3, VLAN 20 | 10.N.20.1/24 |
| Nó N, switch L3, VLAN 99 | 10.N.99.1/27 |
| Nó N, switch de acesso, VLAN 99 | 10.N.99.2/27 |
| Nó N, switch L3, trânsito para firewall | 10.N.255.1/30 |
| Nó N, firewall, trânsito para switch | 10.N.255.2/30 |
| Nó N, firewall, WAN | 172.16.N.2/30 |
| Central, switch L3, VLAN 30 | 10.0.30.1/24 |
| Central, switch L3, VLAN 40 | 10.0.40.1/24 |
| Central, switch L3, VLAN 99 | 10.0.99.1/27 |
| Central, switch L3, trânsito para firewall | 10.0.255.1/30 |
| Central, firewall, trânsito para switch | 10.0.255.2/30 |
| Central, firewall, WAN N | 172.16.N.1/30 |
| Central, firewall, exterior | 203.0.113.2/30 |
| Router da Internet simulada, exterior | 203.0.113.1/30 |
| Router da Internet simulada, LAN | 198.51.100.1/24 |
| Servidor da Internet simulada | 198.51.100.10/24 |
| PC da Internet simulada | 198.51.100.20/24 |

O endereço fixo da impressora do nó N é `10.N.10.5/24`, gateway `10.N.10.1`. PCs e tablet usam DHCP. Os endereços indicados para Internet são intervalos reservados para documentação e simulação, não endereços públicos reais.

## Servidores centrais e DNS

| Serviço | Nome do equipamento | Endereço | Gateway | DNS |
|---|---|---|---|---|
| DNS | CENTRAL-SRV-DNS | 10.0.30.10/24 | 10.0.30.1 | 10.0.30.10 |
| DHCP | CENTRAL-SRV-DHCP | 10.0.30.11/24 | 10.0.30.1 | 10.0.30.10 |
| AAA / serviço de domínio representativo | CENTRAL-SRV-DC | 10.0.30.12/24 | 10.0.30.1 | 10.0.30.10 |
| Intranet | CENTRAL-SRV-INTRA | 10.0.30.13/24 | 10.0.30.1 | 10.0.30.10 |
| Extranet | CENTRAL-SRV-EXTRA | 10.0.40.10/24 | 10.0.40.1 | 10.0.30.10 |

Domínio provisório: `empresa.lan` (a confirmar). Registos DNS planeados: `dns.empresa.lan` → `10.0.30.10`, `dhcp.empresa.lan` → `10.0.30.11`, `dc.empresa.lan` → `10.0.30.12`, `intranet.empresa.lan` → `10.0.30.13`, `extranet.empresa.lan` → `10.0.40.10`.

## DHCP

O servidor CENTRAL-SRV-DHCP terá oito pools, um por VLAN de utilizadores:

| Pool | Gateway | Primeiro endereço | Máscara | Máximo |
|---|---|---|---|---:|
| N1-CABO | 10.1.10.1 | 10.1.10.11 | 255.255.255.0 | 244 |
| N1-WIFI | 10.1.20.1 | 10.1.20.11 | 255.255.255.0 | 244 |
| N2-CABO | 10.2.10.1 | 10.2.10.11 | 255.255.255.0 | 244 |
| N2-WIFI | 10.2.20.1 | 10.2.20.11 | 255.255.255.0 | 244 |
| N3-CABO | 10.3.10.1 | 10.3.10.11 | 255.255.255.0 | 244 |
| N3-WIFI | 10.3.20.1 | 10.3.20.11 | 255.255.255.0 | 244 |
| N4-CABO | 10.4.10.1 | 10.4.10.11 | 255.255.255.0 | 244 |
| N4-WIFI | 10.4.20.1 | 10.4.20.11 | 255.255.255.0 | 244 |

Todos anunciam DNS `10.0.30.10`. O switch L3 de cada nó encaminha pedidos DHCP com `ip helper-address 10.0.30.11`.

## Topologia e encaminhamento

Cada nó usa um switch Catalyst 3650-24PS como switch L3: as SVIs são gateways das VLANs; uma ligação L3 `/30` liga-o à firewall. Um Catalyst 2960-24TT dá portas cabladas e transporta as VLANs 10 e 99 em trunk. O AP é ligado a uma porta de acesso na VLAN 20.

A topologia é hub-and-spoke, com uma interface da firewall central por nó. O tráfego entre nós atravessa o central. A proposta é usar IPsec IKEv1 entre as firewalls; a saída para a Internet simulada usa PAT no central. Estas funções não foram testadas.

## Limitações e pontos a confirmar

- A divisão 175/175 entre cabo e Wi-Fi é uma hipótese de dimensionamento, não um requisito conhecido.
- `empresa.lan` é provisório. O Packet Tracer não fornece Active Directory real; o servidor representa DNS/DHCP e serviço AAA.
- O funcionamento do IPsec nas ASA do Packet Tracer 9.0.0 tem de ser validado manualmente. Se falhar, rever a topologia e só então ativar o plano B com routers.
- A VLAN 99 `/27` comporta 30 endereços de host, suficiente para os equipamentos de rede previstos, sujeito a confirmação do inventário.
- O endereço de gateway do lado da rede `198.51.100.0/24` é `198.51.100.1`; o endereço `203.0.113.1` pertence à ligação exterior `/30`.
