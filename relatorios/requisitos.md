# Requisitos do projeto

**Autor:** Duarte Sousa
**Papel:** Analista e Arquiteto
**Versão:** 1.0
**Data:** 11/11/2026
**Estado:** reconstruído a partir dos requisitos fornecidos; confirmar o texto exato com o enunciado.

## Requisitos funcionais

| Código | Descrição | Prioridade | Critério de aceitação |
|---|---|---|---|
| RF1 | Interligar os quatro nós ao central, em estrela. | Must | Um host de cada nó faz ping a um host dos outros três nós e a um host central. Em Simulation mode, o PDU passa pela firewall central. O traceroute não é prova de passagem pela ASA. |
| RF2 | Cada nó suporta 350 equipamentos geridos, com margem de crescimento. | Must | O endereçamento fornece pelo menos 350 endereços úteis por nó e margem mínima de 25%; as contas ficam documentadas e um host representativo recebe endereço DHCP. |
| RF3 | Cada nó tem redes cablada e Wi-Fi em VLANs separadas. | Must | Um PC cablado e um dispositivo Wi-Fi por nó recebem DHCP da VLAN respetiva e chegam a um servidor central. |
| RF4 | Cada nó tem uma firewall que o liga ao central. | Must | A firewall tem interfaces inside e outside e o túnel está ativo. Demonstra-se um bloqueio, por exemplo, Wi-Fi para a VLAN de gestão. |
| RF5 | O central tem firewall, servidores e saída para a Internet. | Must | Os servidores são acessíveis a partir dos quatro nós; um host de cada nó chega ao servidor de Internet simulado e a saída passa pelo central. |
| RF6 | Estudar a ligação à Internet. | Must | O relatório compara pelo menos duas alternativas de acesso e justifica a escolhida. |
| RF7 | Intranet e extranet (SharePoint ou Google Drive). | Must (estudo), Should (simulação) | O estudo fica documentado; a intranet é acessível a partir dos nós e a extranet fica numa DMZ, acessível a partir da Internet simulada. |
| RF8 | Domínio que valida PCs e servidores. | Must | O desenho do domínio fica documentado, servidores e PCs de exemplo resolvem nomes e é demonstrada autenticação no servidor AAA. |
| RF9 | Testes de exemplo e de aceitação. | Must | Existe pelo menos um teste por requisito Must; cada teste é executado e registado como passou ou falhou. |

## Requisitos não funcionais

| Código | Descrição | Prioridade | Critério de aceitação |
|---|---|---|---|
| RNF1 | Proposta da equipa: proteger o tráfego nó-central com VPN IPsec. | Should | O tráfego nó-central passa pelo túnel e o estado ativo é verificado no equipamento. |
| RNF2 | Encaminhamento dinâmico e redundância. | Should | As rotas são aprendidas dinamicamente; após falha de uma ligação, o serviço recupera e o comportamento é testado. |
| RNF3 | Endereçamento sem sobreposições e sumarizável. | Must | Não há sub-redes sobrepostas e cada nó pode ser resumido num prefixo. |
| RNF4 | Gestão segura dos equipamentos. | Could | Os equipamentos são geridos por SSH a partir da VLAN de gestão. |

## Dúvidas para o professor

O documento de avaliação atribui pontos a **Modelos de BI**, **interfaces gráficos**, **Easy-to-use** e **PEN**, mas o significado e os entregáveis destes termos não estão esclarecidos no material disponível. Não lhes foi atribuído um significado por suposição. Pedir definição ao professor.

## Limites e pressupostos

- “Must”, “Should” e “Could” são as prioridades registadas para este trabalho; confirmar se correspondem à nomenclatura formal do enunciado.
- O Packet Tracer representa os servidores de domínio e autenticação com serviços simulados. Não equivale a uma instalação real de Active Directory.
- Os testes e critérios descritos são planeados; só devem ser marcados como concluídos depois de executados e registados.
