Variações a aplicar depois da configuração completa do nó 1.
Este ficheiro é uma referência, não um bloco de comandos para colar.

Para cada nó N, usar N = 2, 3 ou 4. Nos ficheiros N1-SW1.txt,
N1-SW2.txt e N1-FW1.txt, alterar:

- Hostname: N1-SW1/N1-SW2/N1-FW1 para N-SW1/N-SW2/N-FW1.
- VLAN 10: 10.1.10.1 para 10.N.10.1.
- VLAN 20: 10.1.20.1 para 10.N.20.1.
- VLAN 99: 10.1.99.0/27 e .1/.2 para 10.N.99.0/27 e .1/.2.
- Trânsito switch-firewall: 10.1.255.1/.2 para 10.N.255.1/.2.
- ACL BLOQUEIA-GESTAO: destino 10.1.99.0 para 10.N.99.0.
- Rota predefinida do switch: next-hop 10.N.255.2.
- Relay DHCP continua a apontar para 10.0.30.11.
- WAN da firewall do nó: 172.16.N.2/30; gateway 172.16.N.1.
- Peer e crypto ACL da firewall do nó: peer 172.16.N.1 e rede local
  10.N.0.0/16. Usar a chave local correspondente CHAVE-IPSEC-N.

Portas da firewall central por nó:

| Nó | Firewall do nó | Interface WAN central | Rede WAN |
|---|---|---|---|
| N2 | N2-FW1 Gi1/1 | CENTRAL-FW1 Gi1/2 | 172.16.2.0/30 |
| N3 | N3-FW1 Gi1/1 | CENTRAL-FW1 Gi1/3 | 172.16.3.0/30 |
| N4 | N4-FW1 Gi1/1 | CENTRAL-FW1 Gi1/4 | 172.16.4.0/30 |

Na configuração central já estão planeadas as rotas, ACLs e entradas
IPsec para N2, N3 e N4. As chaves correspondentes têm de coincidir
entre a ASA central e a ASA do nó.

Endereços dos equipamentos finais:

- N-SW2: VLAN 99, 10.N.99.2/27; gateway 10.N.99.1.
- N-PRN1: 10.N.10.5/24; gateway 10.N.10.1; DNS 10.0.30.10.
- N-PC1, N-PC2 e N-TAB1: DHCP.
- N-AP1: SSID EMPRESA-N; ligado a uma porta de acesso VLAN 20.

Os comandos completos específicos dos switches N2-N4 ainda têm de
ser gerados ou obtidos por substituição cuidadosa no texto de N1 antes
de serem aplicados. Não colar apenas os valores desta referência como
se fossem uma configuração completa.
