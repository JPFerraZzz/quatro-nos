# Esqueleto v0 da rede (Packet Tracer 9.0.0)

**Estado: esqueleto, não testado além do ping.** Só há ligação WAN entre o central e cada nó. Não há LAN, VLANs, Wi-Fi, DHCP, VPN, firewall, encaminhamento nem servidores.

Ficheiro: `esqueleto-v0.pkt`. Todos os valores são inventados, de laboratório. Não há passwords nem chaves.

## Equipamentos

| Nome | Modelo usado | Papel | Nota |
|---|---|---|---|
| CENTRAL-FW1 | Cisco 2911 + HWIC-2T | Fronteira do nó central | **Provisório**: devia ser firewall (ver abaixo) |
| N1-FW1 | Cisco 2911 | Fronteira do nó 1 | **Provisório** |
| N2-FW1 | Cisco 2911 | Fronteira do nó 2 | **Provisório** |
| N3-FW1 | Cisco 2911 | Fronteira do nó 3 | **Provisório** |
| N4-FW1 | Cisco 2911 + HWIC-2T | Fronteira do nó 4 | **Provisório** |

### Porque são provisórios

O plano é usar a ASA 5506-X. Neste MCP ela só se pode adicionar e ligar: a API de consola não a alcança (`the device has no console`), por isso não é possível configurar IPs nem fazer ping a partir dela. As ASA foram substituídas por routers 2911, marcados como provisórios, para ter o esqueleto a funcionar. Devem ser trocados quando houver forma de configurar as ASA (por exemplo, à mão na CLI do PT).

O catálogo do MCP anuncia as portas da ASA 5506-X como `Gi1/0` a `Gi1/7`, mas no PT real são `Gi1/1` a `Gi1/8` mais `Management1/1`.

O switch Catalyst 3650-24PS existe no catálogo (28 portas). Não foi usado neste esqueleto e não foi testado.

## Cablagem

| # | Ponta A | Ponta B | Cabo | Rede |
|---|---|---|---|---|
| 1 | CENTRAL-FW1 Gi0/0 | N1-FW1 Gi0/0 | cross | 172.16.1.0/30 |
| 2 | CENTRAL-FW1 Gi0/1 | N2-FW1 Gi0/0 | cross | 172.16.2.0/30 |
| 3 | CENTRAL-FW1 Gi0/2 | N3-FW1 Gi0/0 | cross | 172.16.3.0/30 |
| 4 | CENTRAL-FW1 Serial0/0/0 | N4-FW1 Serial0/0/0 | serial | 172.16.4.0/30 |

A ligação 4 é série porque o 2911 só tem 3 portas Gigabit fixas. A tentativa com `HWIC-1GE-SFP` e fibra não deu ligação: as portas SFP ficaram `down (disabled)`, sem transceiver. O lado do central (DCE) tem `clock rate 64000`.

## Endereçamento WAN

| Rede | CENTRAL-FW1 | Nó | Máscara |
|---|---|---|---|
| 172.16.1.0/30 | 172.16.1.1 (Gi0/0) | N1-FW1 172.16.1.2 (Gi0/0) | 255.255.255.252 |
| 172.16.2.0/30 | 172.16.2.1 (Gi0/1) | N2-FW1 172.16.2.2 (Gi0/0) | 255.255.255.252 |
| 172.16.3.0/30 | 172.16.3.1 (Gi0/2) | N3-FW1 172.16.3.2 (Gi0/0) | 255.255.255.252 |
| 172.16.4.0/30 | 172.16.4.1 (Serial0/0/0) | N4-FW1 172.16.4.2 (Serial0/0/0) | 255.255.255.252 |

## Disposição no canvas

Central ao meio (450, 300). N1 em cima à esquerda (150, 120), N2 em cima à direita (750, 120), N3 em baixo à esquerda (150, 480), N4 em baixo à direita (750, 480).

## Testes feitos

Ping de cada nó ao IP do central na sua rede WAN:

| Nó | Destino | Resultado |
|---|---|---|
| N1-FW1 | 172.16.1.1 | 4/5 (80%) |
| N2-FW1 | 172.16.2.1 | 4/5 (80%) |
| N3-FW1 | 172.16.3.1 | 4/5 (80%) |
| N4-FW1 | 172.16.4.1 | 5/5 (100%) |

Os 80% são o primeiro pacote, que se perde no ARP. Não foi testado mais nada.
