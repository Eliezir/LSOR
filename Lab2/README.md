# Laboratório Kathará: Configuração Serviço DHCP com Roteamento Estático

![Diagrama da topologia](./topologia.png)


Topologia do laboratório (roteadores R1–R3, LANs A/B/C e enlaces D/E)

# Etapa 1 — Planejamento e Modelagem

## 1.1 Diagrama da topologia.

A figura acima corresponde ao diagrama da topologia: partimos da imagem base do enunciado e acrescentamos endereços IP e interfaces de rede nos equipamentos.

## 1.2 Tabela de endereçamento (IPs, máscaras, gateways e funções);


| Dispositivo | Interface | Rede   | Endereço IP/CIDR | Mascara Decimal | Gateway Padrão | Função           |
| ----------- | --------- | ------ | ---------------- | --------------- | -------------- | ---------------- |
| R1          | eth0      | Rede A | 192.168.1.1/24   | 255.255.255.0   | -              | Gateway da LAN A |
| R1          | eth1      | Rede E | 100.0.0.9/30     | 255.255.255.252 | -              | Enlace com R3    |
| R1          | eth2      | Rede D | 10.0.0.5/30      | 255.255.255.252 | -              | Enlace com R2    |
| R2          | eth0      | Rede B | 192.168.2.1/24   | 255.255.255.0   | -              | Gateway da LAN B |
| R2          | eth1      | Rede D | 10.0.0.6/30      | 255.255.255.252 | -              | Enlace com R1    |
| R3          | eth0      | Rede C | 192.168.3.1/24   | 255.255.255.0   | -              | Gateway da LAN C |
| R3          | eth1      | Rede E | 100.0.0.10/30    | 255.255.255.252 | -              | Enlace com R1    |
| PC4         | eth0      | Rede A | DHCP             | 255.255.255.0   | 192.168.1.1    | Host da LAN A    |
| PC5         | eth0      | Rede A | DHCP             | 255.255.255.0   | 192.168.1.1    | Host da LAN A    |
| PC0         | eth0      | Rede B | DHCP             | 255.255.255.0   | 192.168.2.1    | Host da LAN B    |
| PC1         | eth0      | Rede B | DHCP             | 255.255.255.0   | 192.168.2.1    | Host da LAN B    |
| PC2         | eth0      | Rede C | DHCP             | 255.255.255.0   | 192.168.3.1    | Host da LAN C    |
| PC3         | eth0      | Rede C | DHCP             | 255.255.255.0   | 192.168.3.1    | Host da LAN C    |


## 1.3 Faixas DHCP de cada LAN.

foi definido um bloco de enderecos dinamicos em cada sub-rede local, preservando os enderecos mais baixos para equipamentos com IP fixo (como o gateway):

- Rede A (`192.168.1.0/24`): `192.168.1.100-192.168.1.200`
- Rede B (`192.168.2.0/24`): `192.168.2.100-192.168.2.200`
- Rede C (`192.168.3.0/24`): `192.168.3.100-192.168.3.200`

Essas faixas funcionam assim: quando um host entra na rede e solicita configuracao via DHCP, o servidor (`dnsmasq`) daquela LAN oferece um IP livre dentro do intervalo configurado, junto com mascara, gateway e DNS. No projeto, isso e definido nos arquivos `Lab2/shared/dnsmasq-r1.conf`, `Lab2/shared/dnsmasq-r2.conf` e `Lab2/shared/dnsmasq-r3.conf`, por meio da diretiva `dhcp-range` (uma para cada LAN). Dessa forma, cada LAN recebe automaticamente enderecamento proprio, sem conflito entre redes e com separacao clara por sub-rede.

## 1.4 Plano de rotas estáticas.

O plano de rotas estáticas define por qual próximo salto (next-hop) cada roteador deve encaminhar pacotes para redes remotas. Como cada roteador conhece diretamente apenas suas redes conectadas, as rotas estáticas garantem a comunicação entre as LANs A, B e C. A coluna **Redes ligadas ao roteador** indica os prefixos das interfaces desse equipamento (rotas conectadas implícitas); as linhas seguintes são as rotas estáticas acrescentadas para alcançar destinos remotos.


| Dispositivo | Redes ligadas ao roteador                                                                      | Destino                 | Via (Gateway)       |
| ----------- | ---------------------------------------------------------------------------------------------- | ----------------------- | ------------------- |
| R1          | 192.168.1.0/24 (Rede A), 10.0.0.4/30 (Rede D), 100.0.0.8/30 (Rede E)                           | 192.168.2.0/24 (Rede B) | 10.0.0.6 (Rede D)   |
| R1          | 192.168.1.0/24 (Rede A), 10.0.0.4/30 (Rede D), 100.0.0.8/30 (Rede E)                           | 192.168.3.0/24 (Rede C) | 100.0.0.10 (Rede E) |
| R2          | 192.168.2.0/24 (Rede B), 10.0.0.4/30 (Rede D)                                                  | 192.168.1.0/24 (Rede A) | 10.0.0.5 (Rede D)   |
| R2          | 192.168.2.0/24 (Rede B), 10.0.0.4/30 (Rede D)                                                  | 192.168.3.0/24 (Rede C) | 10.0.0.5 (Rede D)   |
| R3          | 192.168.3.0/24 (Rede C), 100.0.0.8/30 (Rede E)                                                 | 192.168.1.0/24 (Rede A) | 100.0.0.9 (Rede E)  |
| R3          | 192.168.3.0/24 (Rede C), 100.0.0.8/30 (Rede E)                                                 | 192.168.2.0/24 (Rede B) | 100.0.0.9 (Rede E)  |


Essas rotas estao definidas nos arquivos de inicializacao dos roteadores (`Lab2/r1.startup`, `Lab2/r2.startup` e `Lab2/r3.startup`) com comandos `ip route add`. Assim, quando um host de uma LAN envia trafego para outra LAN, o roteador encaminha o pacote para o roteador vizinho correto ate chegar na rede de destino.

## 1.5 Plano de testes (ping, traceroute, DHCP discover).

Para validar o funcionamento do roteamento estático, realizamos os seguintes testes:

- Ping entre hosts da mesma LAN;
- Ping entre hosts de LANs diferentes;
- Traceroute entre hosts de LANs diferentes;
- (Opcional) automatizar parte dos testes com scripts no host ou dentro dos containers.

# 2 Implementação do Serviço DHCP

## 2.1 Pesquise e escolha uma implementação DHCP para Linux (ex.:  dnsmasq, isc-dhcp-server, udhcpd, keadhcp etc.)

Optamos por utilizar o `dnsmasq` para implementar o serviço DHCP, visto que é um servidor DHCP leve e fácil de configurar e deve atender as necessidades do laboratório.

## 2.2 Instale e configure nos roteadores (R1, R2, R3).

Para configurar o `dnsmasq` em cada roteador, utilizamos os seguintes arquivos de configuração:

- `Lab2/shared/dnsmasq-r1.conf`
- `Lab2/shared/dnsmasq-r2.conf`
- `Lab2/shared/dnsmasq-r3.conf`

Esses arquivos de configuração definem as faixas DHCP para cada LAN, o gateway padrão e o servidor DNS.

**DNS (resolução de nomes):** via DHCP, cada cliente recebe como DNS o **gateway da própria LAN** (`dhcp-option=6`: `192.168.1.1`, `192.168.2.1` ou `192.168.3.1`). No **R1**, o `dnsmasq` escuta na LAN A e nos enlaces com **R2** e **R3** (`interface=eth0`, `eth1` e `eth2` em `dnsmasq-r1.conf`), encaminhando consultas para resolvedores públicos (`server=1.1.1.1` e `server=8.8.8.8`, com `no-resolv`). Nos roteadores **R2** e **R3**, o encaminhamento aponta para o **R1** (`server=10.0.0.5` e `server=100.0.0.9`), de modo que a saída para a Internet (incluindo DNS) fica centralizada no roteador de borda.

Para configurar o `dnsmasq` em cada roteador, utilizamos os seguintes comandos:

- `dnsmasq -C /shared/dnsmasq-r1.conf`
- `dnsmasq -C /shared/dnsmasq-r2.conf`
- `dnsmasq -C /shared/dnsmasq-r3.conf`

Esses comandos configuram o `dnsmasq` para cada roteador, utilizando os arquivos de configuração definidos anteriormente.

## 2.3 Arquivos .startup para automatizar

### 2.3.1 Configuracao de IPs e rotas;

- Configuracao de IPs e rotas;
Na configuração de IPs e rotas nos arquivos `.startup`, definimos os endereços das interfaces de cada roteador e os caminhos estáticos para redes remotas. Em `R1`, por exemplo, foram configuradas as interfaces das redes A, D e E, além das rotas para as LANs de `R2` e `R3`.

```bash
ifconfig eth0 192.168.1.1/24
ifconfig eth1 100.0.0.9/30
ifconfig eth2 10.0.0.5/30

ip route add 192.168.2.0/24 via 10.0.0.6
ip route add 192.168.3.0/24 via 100.0.0.10
```

### 2.3.2 Inicialização dos serviços DHCP

Nos roteadores (`R1`, `R2` e `R3`), os serviços DHCP são iniciados automaticamente com `dnsmasq -C /shared/dnsmasq-rX.conf`, de acordo com a LAN atendida por cada nó. Nos dispositivos finais (PCs), a inicialização DHCP é feita via `dhclient eth0` nos arquivos `.startup` de cada host (`pc0` a `pc5`). Assim, ao subir o laboratório, os servidores DHCP já estão ativos e cada cliente solicita automaticamente um endereço IP da sua LAN, recebendo também máscara, gateway padrão e DNS. Esse processo permite validar o funcionamento fim a fim do DHCP sem configuração manual de IP nos hosts.

### 2.3.3 Captura automática de pacotes DHCP com tcpdump

Para registrar a troca de mensagens DHCP durante a subida do laboratório, o `tcpdump` foi configurado **apenas nos roteadores**, na interface da LAN de cada um (`eth0` de `R1`, `R2` e `R3`). Nessa interface passa o tráfego DHCP de **todos** os clientes daquela sub-rede (mensagens em broadcast na maior parte do fluxo), o que evita repetir captura em cada PC e ainda assim documenta o DORA por LAN.

O diretório `/shared/captures` é criado no `r1.startup` (`mkdir -p /shared/captures`); como o diretório é compartilhado pelo laboratório, os arquivos gravados pelos roteadores ficam acessíveis no host.

Arquivos gerados:

- `r1-dhcp.pcap` — Rede A
- `r2-dhcp.pcap` — Rede B
- `r3-dhcp.pcap` — Rede C

Comando utilizado em cada roteador (segundo plano):

- `tcpdump -i eth0 -n -U -w /shared/captures/rX-dhcp.pcap 'port 67 or port 68' >/dev/null 2>&1 &`

Pontos importantes:

- Filtro `port 67 or port 68`: captura apenas tráfego DHCP/BOOTP (UDP).
- Opção `-U`: grava os pacotes incrementalmente no arquivo `.pcap`.
- Execução em background (`&`): permite continuar o boot normalmente.
- Uso de `/shared/captures`: mantém as evidências fora dos containers, acessíveis no host.

### 2.3.4 Arquivos de configuração externos no Kathará

No Kathará, arquivos externos podem ser disponibilizados aos nós por meio do diretório compartilhado do laboratório. Neste projeto, os arquivos de configuração foram colocados em `Lab2/shared/` no host e acessados dentro dos roteadores pelo caminho `/shared/...`.

Assim, cada nó carrega sua configuração sem precisar embutir o arquivo dentro da imagem do container. No caso do DHCP, fizemos o vínculo da seguinte forma:

- `Lab2/shared/dnsmasq-r1.conf`  -> usado por `R1` como `/shared/dnsmasq-r1.conf`
- `Lab2/shared/dnsmasq-r2.conf`  -> usado por `R2` como `/shared/dnsmasq-r2.conf`
- `Lab2/shared/dnsmasq-r3.conf`  -> usado por `R3` como `/shared/dnsmasq-r3.conf`

Nos arquivos `.startup`, os roteadores iniciam o serviço apontando para esses arquivos externos:

- `dnsmasq -C /shared/dnsmasq-r1.conf`
- `dnsmasq -C /shared/dnsmasq-r2.conf`
- `dnsmasq -C /shared/dnsmasq-r3.conf`

Com isso, o gerenciamento das configurações fica centralizado, reutilizável e fácil de manter, além de permitir persistir artefatos como capturas em `/shared/captures`.

# Etapa 3 — Conectividade com a Internet - DNAT e SNAT

## 1. Como o R1 conecta à Internet e atua como Gateway Padrão da topologia

Para permitir acesso à Internet a partir de todas as LANs, o `R1` foi configurado como roteador de borda (edge router), com três funções principais:

1. **Encaminhamento IP habilitado**
  No `R1`, ativamos o roteamento entre interfaces com:
  - `echo 1 > /proc/sys/net/ipv4/ip_forward`
2. **Saída para Internet com SNAT (MASQUERADE)**
  O `R1` aplica NAT na interface de saída WAN (bridge) para traduzir os IPs privados das LANs para o IP de saída do host. No `r1.startup`, a regra usada é:
  - `iptables -t nat -A POSTROUTING -o eth3 -j MASQUERADE`
   O script também verifica se `eth3` existe e pode obter a interface da rota default como referência; se em outro ambiente a WAN não for `eth3`, a regra de `MASQUERADE` deve apontar para a interface correta.
   Isso implementa **SNAT dinâmico** (masquerading), permitindo que redes privadas (`192.168.1.0/24`, `192.168.2.0/24`, `192.168.3.0/24`) acessem destinos externos.
3. **Rotas padrão dos demais roteadores apontando para o R1**
  Para centralizar a saída no `R1`, definimos:
  - Em `R2`: `ip route add default via 10.0.0.5` (R1)
  - Em `R3`: `ip route add default via 100.0.0.9` (R1)

Com essa arquitetura, qualquer tráfego de Internet gerado por PCs das LANs B e C chega primeiro ao seu roteador local (`R2`/`R3`), é encaminhado ao `R1` pela rota default, e sai para Internet após NAT no `R1`.

# Etapa 4 — Testes e Capturas

## 1. Verificação de IPs obtidos via DHCP (`ip addr show`)

Com o laboratório iniciado (`kathara lstart`), verificamos os endereços atribuídos dinamicamente nas interfaces dos clientes.

Evidências coletadas:

- `pc0` recebeu `192.168.2.172/24` na `eth0` (Rede B), dentro da faixa `192.168.2.100-192.168.2.200`;
- `pc5` recebeu `192.168.1.194/24` na `eth0` (Rede A), dentro da faixa `192.168.1.100-192.168.1.200`.

Os dois resultados aparecem como `scope global dynamic`, confirmando concessão DHCP válida nos clientes.

Evidência de IPs via DHCP (pc0 e pc5)

### Evidência adicional — DHCP e conectividade na Rede B

Foi verificado o endereço da interface `eth0` em dois hosts da Rede B:

- `pc0`: `192.168.2.172/24` (`scope global dynamic`)
- `pc1`: `192.168.2.175/24` (`scope global dynamic`)

Ambos os endereços estão dentro da faixa DHCP configurada para a Rede B (`192.168.2.100-192.168.2.200`), confirmando concessão dinâmica válida.

Também foi testada a conectividade entre os hosts da mesma LAN com:

- `ping -c 4 192.168.2.175` (a partir de `pc0`)

Resultado:

- `4 transmitted, 4 received, 0% packet loss`, confirmando comunicação correta na sub-rede.

Evidência DHCP e ping na Rede B

### Evidência — conectividade entre redes diferentes (Rede C -> Rede A)

Para validar o roteamento entre sub-redes, verificamos primeiro os IPs dinâmicos dos hosts envolvidos:

- `pc5` (Rede A): `192.168.1.194/24` (`scope global dynamic`)
- `pc3` (Rede C): `192.168.3.134/24` (`scope global dynamic`)

Em seguida, executamos o teste:

- `ping -c 4 192.168.1.194` (a partir de `pc3`)

Resultado:

- `4 packets transmitted, 4 received, 0% packet loss`
- RTT médio de `5.886 ms`

Conclusão:
A comunicação entre a Rede C e a Rede A está funcional, confirmando que o roteamento estático entre os roteadores está operando corretamente.

Evidência de conectividade entre LANs (pc3 -> pc5)

### Evidência — análise no Wireshark (estágios DORA e camadas)

A captura `r2-dhcp.pcap` (interface `eth0` do **R2**, Rede B) foi analisada no Wireshark com filtro DHCP, evidenciando os estágios do processo DORA observados na LAN B (incluindo o fluxo do cliente `pc0`):

- `DHCP Discover`
- `DHCP Offer`
- `DHCP Request`
- `DHCP ACK`

Também foi possível observar, no pacote selecionado, as camadas envolvidas no processo:

- Ethernet II
- IPv4
- UDP (porta 68 -> 67 no Discover)
- BOOTP/DHCP

Além disso, a lista de pacotes mostra os IDs de transação (`xid`) associados às trocas DHCP.

Evidência Wireshark: sequência DORA e camadas

No pacote `DHCP ACK` selecionado, os campos solicitados foram observados com os seguintes valores:

- `xid` (`Transaction ID`): `0x627db826`
- `yiaddr` (`Your client IP address`): `192.168.2.172`
- `chaddr` (`Client MAC address`): `06:3c:96:5c:cc:06`
- `siaddr` (`Next server IP address`): `192.168.2.1`

Esses dados confirmam a associação entre cliente, servidor e endereço atribuído durante a etapa final da negociação DHCP.

Evidência Wireshark: campos xid, yiaddr, chaddr e siaddr