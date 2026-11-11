# Pacote de configuração v1 para o Packet Tracer

**Autor:** Duarte Sousa, Analista e Arquiteto
**Data:** 11/11/2026
**Versão:** 1.0
**Estado:** preparado; **não testado no Packet Tracer**. Não assumir que os comandos ASA, IPsec, serviços gráficos ou DHCP funcionam até o João os executar e registar o resultado.

## 1. O que este pacote entrega

- Configuração completa planeada para o nó 1 e configuração central.
- Indicação das alterações para os nós 2, 3 e 4.
- Configuração do router da Internet simulada.
- Passos GUI para servidores, AP e hosts, que não têm CLI IOS.
- Testes planeados e resultados esperados.

Os ficheiros `.txt` contêm apenas comandos, na ordem de aplicação, e começam com `[MCP]` ou `[À MÃO]`. O primeiro indica equipamento IOS que o MCP do João pode configurar; o segundo indica ASA, a configurar manualmente na CLI do Packet Tracer.

## 2. Mudança face ao esqueleto v0

Mantêm-se os nomes `CENTRAL-FW1` e `N1-FW1` a `N4-FW1`, mas o modelo 2911 provisório de cada firewall é substituído por ASA 5506-X. O esqueleto v0 tem ligações diretas entre routers 2911 e endereços WAN já configurados; não basta colar as novas configurações nesse ficheiro. É preciso substituir os routers provisórios pelas ASA, refazer as ligações e acrescentar os switches, APs, hosts e servidores desta topologia.

As quatro ligações WAN passam a usar portas Gigabit da ASA central, inclusive a ligação ao nó 4. Deixa de ser necessária a ligação série usada no esqueleto por falta de uma quarta porta Gigabit no router 2911. O endereço WAN de cada ligação continua igual ao esqueleto: central `172.16.N.1/30`, nó N `172.16.N.2/30`.

O nome e as portas anunciados pelo catálogo do MCP para a ASA podem divergir do Packet Tracer. Este pacote assume as portas reais `GigabitEthernet1/1` a `GigabitEthernet1/8`; confirmar no equipamento antes de ligar ou colar comandos.

## 3. Equipamentos

| Quantidade | Nome(s) | Modelo no Packet Tracer | Função |
|---:|---|---|---|
| 4 | N1-FW1 a N4-FW1 | ASA 5506-X | Firewall e endpoint IPsec de cada nó |
| 4 | N1-SW1 a N4-SW1 | Catalyst 3650-24PS | Switch L3, SVIs, encaminhamento e relay DHCP |
| 4 | N1-SW2 a N4-SW2 | Catalyst 2960-24TT | Acesso cablado |
| 4 | N1-AP1 a N4-AP1 | AccessPoint-PT | Wi-Fi |
| 8 | N1-PC1/PC2 a N4-PC1/PC2 | PC-PT | Hosts representativos cablados |
| 4 | N1-PRN1 a N4-PRN1 | Printer-PT | Impressora com IP fixo |
| 4 | N1-TAB1 a N4-TAB1 | TabletPC-PT | Host representativo Wi-Fi |
| 1 | CENTRAL-FW1 | ASA 5506-X | Firewall central, hub IPsec, PAT e saída exterior |
| 1 | CENTRAL-SW1 | Catalyst 3650-24PS | VLANs centrais e encaminhamento |
| 5 | CENTRAL-SRV-DNS, DHCP, DC, INTRA, EXTRA | Server-PT | DNS, DHCP, AAA representativo, intranet e extranet |
| 1 | INET-RTR | Cisco 2911 | Router da Internet simulada |
| 1 | INET-SW1 | Catalyst 2960-24TT | Acesso da rede simulada |
| 1 | INET-SRV | Server-PT | Destino de teste da Internet simulada |
| 1 | INET-PC1 | PC-PT | Cliente de teste externo |

Não são criados 350 PCs por nó: dois PCs cablados, um tablet Wi-Fi e uma impressora permitem testar os tipos de acesso e a configuração. A capacidade de 350 é demonstrada pelas contas de endereçamento e pelos pools DHCP, não pela quantidade de ícones no cenário.

## 4. Cablagem e portas

Usar cobre direto ou a opção automática do Packet Tracer para as ligações Ethernet.

| Ponta A | Ponta B | Rede/observação |
|---|---|---|
| N1-FW1 Gi1/1 | CENTRAL-FW1 Gi1/1 | 172.16.1.0/30 |
| N2-FW1 Gi1/1 | CENTRAL-FW1 Gi1/2 | 172.16.2.0/30 |
| N3-FW1 Gi1/1 | CENTRAL-FW1 Gi1/3 | 172.16.3.0/30 |
| N4-FW1 Gi1/1 | CENTRAL-FW1 Gi1/4 | 172.16.4.0/30 |
| N1-FW1 Gi1/2 | N1-SW1 Gi1/0/1 | 10.1.255.0/30 |
| N1-SW1 Gi1/0/2 | N1-SW2 Gi0/1 | Trunk VLAN 10, 20, 99 |
| N1-SW1 Gi1/0/3 | N1-AP1 Port 0 | Acesso VLAN 20 |
| N1-SW2 Fa0/1 | N1-PC1 Fa0 | VLAN 10 |
| N1-SW2 Fa0/2 | N1-PC2 Fa0 | VLAN 10 |
| N1-SW2 Fa0/3 | N1-PRN1 Fa0 | VLAN 10 |
| N1-TAB1 | N1-AP1 | Sem cabo; associação Wi-Fi |
| CENTRAL-FW1 Gi1/5 | INET-RTR Gi0/0 | 203.0.113.0/30 |
| CENTRAL-FW1 Gi1/6 | CENTRAL-SW1 Gi1/0/1 | 10.0.255.0/30 |
| CENTRAL-SW1 Gi1/0/2 | CENTRAL-SRV-DNS Fa0 | VLAN 30 |
| CENTRAL-SW1 Gi1/0/3 | CENTRAL-SRV-DHCP Fa0 | VLAN 30 |
| CENTRAL-SW1 Gi1/0/4 | CENTRAL-SRV-DC Fa0 | VLAN 30 |
| CENTRAL-SW1 Gi1/0/5 | CENTRAL-SRV-INTRA Fa0 | VLAN 30 |
| CENTRAL-SW1 Gi1/0/6 | CENTRAL-SRV-EXTRA Fa0 | VLAN 40 |
| INET-RTR Gi0/1 | INET-SW1 Gi0/1 | 198.51.100.0/24 |
| INET-SW1 Fa0/1 | INET-SRV Fa0 | 198.51.100.10 |
| INET-SW1 Fa0/2 | INET-PC1 Fa0 | 198.51.100.20 |

Para os nós 2 a 4, substituir N1 por N2/N3/N4, os endereços `10.1` por `10.2`/`10.3`/`10.4`, e ligar a firewall do nó à porta central Gi1/2, Gi1/3 ou Gi1/4 respetivamente. As alterações estão resumidas em `variacoes-nos-2-4.md`; esse ficheiro é uma referência, não um bloco de comandos completo para aplicar diretamente.

## 5. VLANs e endereços principais

| Segmento | Nó 1 | Central | Gateway |
|---|---|---|---|
| VLAN 10, cabo | 10.1.10.0/24 | — | 10.1.10.1 |
| VLAN 20, Wi-Fi | 10.1.20.0/24 | — | 10.1.20.1 |
| VLAN 30, servidores | — | 10.0.30.0/24 | 10.0.30.1 |
| VLAN 40, DMZ | — | 10.0.40.0/24 | 10.0.40.1 |
| VLAN 99, gestão | 10.1.99.0/27 | 10.0.99.0/27 | .1 no switch L3 |
| Trânsito switch-firewall | 10.1.255.0/30 | 10.0.255.0/30 | Switch .1, ASA .2 |
| WAN nó-central | 172.16.1.0/30 | 172.16.1.0/30 | Central .1, nó .2 |

Cada nó usa o seu bloco `10.N.0.0/16`, com N de 1 a 4; o central usa `10.0.0.0/16`. Os gateways das VLANs de utilizadores são SVIs no Catalyst 3650. O resumo detalhado, endereços fixos, pools e contas estão em `../../relatorios/enderecamento-v1.md`.

## 6. Serviços gráficos e hosts no Packet Tracer

Os seguintes passos são na interface gráfica do Packet Tracer, não em ficheiros de comandos. Estão **não testados**.

### Servidores centrais

Configurar em cada Server-PT, em **Config > FastEthernet0**, o IP, máscara, gateway e DNS constantes no plano de endereçamento. Ativar os serviços correspondentes:

- **CENTRAL-SRV-DNS:** `10.0.30.10/24`, gateway `10.0.30.1`. Em **Services > DNS**, ativar DNS e criar os registos do domínio `empresa.lan` indicados no plano.
- **CENTRAL-SRV-DHCP:** `10.0.30.11/24`, gateway `10.0.30.1`. Em **Services > DHCP**, ativar o serviço e criar os oito pools indicados em `../../relatorios/enderecamento-v1.md`, com máscara `/24`, DNS `10.0.30.10`, gateway específico da VLAN, primeiro endereço `.11` e máximo 244 clientes.
- **CENTRAL-SRV-DC:** `10.0.30.12/24`, gateway `10.0.30.1`. Usar **Services > AAA** para representar autenticação RADIUS. Criar os clientes de rede com os endereços de trânsito dos switches (10.N.255.1 e 10.0.255.1) e uma chave escolhida localmente, fora do repositório. O Packet Tracer não implementa Active Directory real.
- **CENTRAL-SRV-INTRA:** `10.0.30.13/24`, gateway `10.0.30.1`. Ativar HTTP e usar uma página simples identificada como “Intranet”.
- **CENTRAL-SRV-EXTRA:** `10.0.40.10/24`, gateway `10.0.40.1`. Ativar HTTP e usar uma página simples identificada como “Extranet”.

### AP, PCs, tablet e impressora

- N1-AP1: configurar SSID `EMPRESA-N1` e WPA2-PSK. Escolher a chave localmente; não a guardar no repositório. Associar N1-TAB1 a este SSID e ativar DHCP.
- PCs dos nós: ativar DHCP. A impressora N1-PRN1 usa `10.1.10.5/24`, gateway `10.1.10.1`, DNS `10.0.30.10`.
- Nos outros nós, usar SSID `EMPRESA-N2`, `EMPRESA-N3` ou `EMPRESA-N4` e os endereços do respetivo bloco.
- INET-SRV: `198.51.100.10/24`, gateway `198.51.100.1`; ativar HTTP para o teste de saída.
- INET-PC1: `198.51.100.20/24`, gateway `198.51.100.1`.

### AAA nos switches (opcional, executar depois dos testes IP)

O AAA é o último passo porque depende da conectividade ao servidor, da definição local de credenciais e de validação de sintaxe no IOS do Packet Tracer. Os ficheiros não contêm palavras-passe nem chaves. Não aplicar antes de guardar uma sessão de consola local de recuperação. A configuração AAA e o login de prova ainda não foram testados.

## 7. Configurar as ASA manualmente

Os ficheiros `N1-FW1.txt` e `CENTRAL-FW1.txt` estão marcados `[À MÃO]`. No Packet Tracer, abrir a ASA, selecionar **CLI**, entrar em modo privilegiado e aplicar linha a linha em modo de configuração. Não limpar uma configuração existente sem guardar uma cópia.

As linhas com `<CHAVE-IPSEC-N1>` ou equivalentes são marcadores, não comandos prontos. Antes de as executar, substituir localmente cada marcador por uma chave aleatória de laboratório. A mesma chave deve ser usada nas duas pontas do túnel correspondente. Não incluir a chave no Git, nas capturas de ecrã nem nos relatórios. Se a ASA rejeitar `crypto ikev1`, registar o erro e a versão/sintaxe mostrada; não assumir que a alternativa IKEv1 antiga funciona sem a testar.

## 8. Verificações planeadas

**Estado de todos os testes do pacote: não testado.** Guardar resultados e capturas só depois da execução. `show` varia entre IOS e ASA; correr cada comando no equipamento indicado.

| Fluxo / requisito | Ação e comando | Resultado esperado |
|---|---|---|
| Estado físico e SVIs | Em N1-SW1 e CENTRAL-SW1: `show ip interface brief`; `show vlan brief`; `show interfaces trunk` | Portas cabladas e SVIs relevantes `up/up`; VLANs 10/20/30/40/99 e trunks apresentados. |
| Encaminhamento | Em N1-SW1 e CENTRAL-SW1: `show ip route` | Rotas conectadas às VLANs e rota predefinida para a ASA respetiva. |
| DHCP por cabo | Em N1-PC1, Desktop > Command Prompt: `ipconfig /renew`, depois `ipconfig` | Endereço `10.1.10.11` a `10.1.10.254`, máscara `/24`, gateway `10.1.10.1`, DNS `10.0.30.10`. Se outro host tiver recebido o primeiro endereço, qualquer endereço dentro do pool é válido. |
| DHCP por Wi-Fi | Em N1-TAB1: confirmar associação a `EMPRESA-N1` e renovar DHCP | Endereço `10.1.20.11` a `10.1.20.254`, máscara `/24`, gateway `10.1.20.1`, DNS `10.0.30.10`. |
| Cabo e Wi-Fi até VLANs | Em N1-PC1 e N1-TAB1: `ping 10.1.10.1` ou `ping 10.1.20.1`, conforme o host | Resposta do gateway da VLAN. |
| Host do nó para servidor central | Em N1-PC1: `ping 10.0.30.13`; depois abrir `http://10.0.30.13` no browser | Respostas e página da intranet. Repetir com tablet. |
| DNS | Em N1-PC1: `ping intranet.empresa.lan`; abrir `http://intranet.empresa.lan` | O nome resolve para `10.0.30.13` e a página abre. Se o comando `ping` não resolver nomes no PC-PT, usar o browser e confirmar o endereço no serviço DNS. |
| Internet simulada com PAT | Em N1-PC1: `ping 198.51.100.10`; abrir `http://198.51.100.10`. Na ASA central: `show xlate` | Servidor de teste alcançável; tabela de traduções inclui PAT para o endereço exterior `203.0.113.2`. |
| Túnel IPsec | Gerar tráfego entre N1-PC1 e servidor central; em N1-FW1 e CENTRAL-FW1: `show crypto ikev1 sa`; `show crypto ipsec sa` | Associação IKE/IPsec ativa e contadores de encapsulamento/descapsulamento aumentam. A sintaxe ou suporte no Packet Tracer pode diferir. |
| Passagem pelo central | No Packet Tracer, Simulation mode, enviar PDU de N1-PC1 para N2-PC1 | O caminho observado inclui CENTRAL-FW1. Não usar apenas `tracert` para concluir que passou pela ASA. |
| Comunicação nó a nó | Em N1-PC1: `ping 10.2.10.11` (ajustar ao IP DHCP efetivamente recebido) | Resposta se o hub-and-spoke e o reencaminhamento de tráfego entre túneis funcionarem. Repetir para N3 e N4; comportamento ainda não testado. |
| Bloqueio de gestão local | Em N1-PC1: `ping 10.1.99.1` | Sem resposta, devido à ACL na SVI de entrada da VLAN de utilizadores. |
| Bloqueio de gestão remota | Em N1-PC1: `ping 10.2.99.1` | Sem resposta; a ACL da firewall central deve impedir acesso às VLANs de gestão. |
| Isolamento DMZ | Em CENTRAL-SRV-EXTRA, testar acesso a `10.0.30.10` | Sem acesso iniciado pela DMZ à VLAN de servidores; confirmar que a ACL DMZ foi aplicada. |
| Extranet externa | Em INET-PC1, abrir `http://203.0.113.2` | Página “Extranet”, caso o Packet Tracer suporte a tradução estática de serviço e HTTP. |
| AAA (posterior) | Iniciar sessão no N1-SW1 com utilizador de teste; desligar temporariamente o servidor AAA e verificar fallback local | Login RADIUS e fallback local conforme configuração. Só fazer depois de criar credenciais e garantir acesso de recuperação. |

Se um teste falhar, registar o comando, a mensagem exata, o equipamento e a versão do Packet Tracer. Corrigir só depois de confirmar se o erro é de sintaxe, cablagem ou desenho. Não marcar como “passou” com base apenas na configuração visível.

## 9. Plano B e limitações

O plano A é IPsec entre as ASA 5506-X. O MCP não acede à consola das ASA, por isso a configuração é manual. A compatibilidade de IKEv1, ACLs, NAT e tráfego entre túneis no Packet Tracer 9.0.0 é desconhecida até ser testada.

Se o IPsec ASA falhar, parar e guardar os erros. O rascunho anterior propõe routers 2911 dedicados aos túneis, com `securityk9`, mas esse plano ainda não tem uma topologia completa validada: pode permitir tráfego nó-a-nó sem atravessar a firewall central e altera cablagem e sub-redes. Não ativar o plano B sem rever esses efeitos com Duarte e João.

Limitações conhecidas: serviços de domínio são representações do Packet Tracer; DHCP central não atribui novos endereços se o caminho ao central cair; IPsec e PAT não foram testados; a redundância dinâmica é Should e não faz parte deste pacote v1.
