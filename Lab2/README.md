## Topologia

![Topologia do Lab (referência)](../Lab2/topologia.png)

## Etapa 1 — Planejamento e Modelagem

Crie um documento descrevendo:

- Diagrama da topologia (pode usar a imagem fornecida).
- Tabela de endereçamento (IPs, máscaras, gateways e funções).
- Faixas DHCP para cada LAN (ex.: `192.168.10.100–192.168.10.200`).
- Plano de rotas estáticas.
- Plano de testes (pings, traceroutes, DHCP discover).

## Etapa 2 — Implementação do Serviço DHCP

- Pesquise e escolha uma implementação DHCP para Linux (ex.: `dnsmasq`, `isc-dhcp-server`, `udhcpd`, `kea` etc.).
- Instale e configure nos roteadores (R1, R2, R3).
- Crie arquivos `.startup` para automatizar:
  - Configuração de IPs e rotas;
  - Inicialização dos serviços DHCP;
  - Captura automática de pacotes DHCP com `tcpdump`.
- Pesquise como o Kathará carrega arquivos de configuração externos (ex.: `dhcpd.conf`, `dnsmasq.conf`) e vincule-os aos nós.

## Etapa 3 — Conectividade com a Internet (DNAT e SNAT)

1. Pesquise como o R1 pode conectar-se à Internet e servir como **Gateway Padrão** para todos os demais nós da topologia.
2. Faça um teste pingando o IP `8.8.8.8` de qualquer PC e verifique o retorno para a Internet.
3. Dica: busque sobre masquerading, NAT estático e dinâmico no Linux. Há outra trilha no ALEX tratando sobre NAT — verifique lá.

## Etapa 4 — Testes e Capturas

- Execute `kathara lstart` e verifique:
  - IPs obtidos via DHCP (`ip addr show`);
  - Conectividade entre sub-redes (`ping`/`traceroute`).
- Capture pacotes DHCP nos clientes e servidores.
- Analise no Wireshark os estágios DORA (Discover, Offer, Request, Ack).
- Observe os campos `xid`, `yiaddr`, `chaddr`, `siaddr` e as camadas envolvidas.
