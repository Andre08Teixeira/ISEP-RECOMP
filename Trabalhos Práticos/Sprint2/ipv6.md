# Relatório IPv6 para a RECOMP Corporation

## Introdução

A RECOMP Corporation planeia migrar de IPv4 para IPv6 para satisfazer as necessidades de crescimento e escalabilidade da rede. Durante o processo de transição, ambos os protocolos (IPv4 e IPv6) devem coexistir. Este relatório apresenta as recomendações de configuração necessárias para suportar IPv6 em cada filial e garantir uma transição tranquila entre os protocolos.

## 1. Planeamento de Sub-redes IPv6
Prefixo `2001:db8::/32:` Este prefixo é reservado pelo IETF(Internet Engineering Task Force) para documentação e exemplos, como uma “rede fictícia” para ensino e teste. Ou seja, não deve ser usado em produção porque é designado para documentação. Para um ambiente real, a organização deve solicitar um prefixo IPv6 a um Provedor de Serviços de Internet (ISP).
  
Para a estrutura de endereçamento IPv6, a RECOMP deve utilizar um **prefixo /48** para cada localização, com divisões em sub-redes /64 para cada VLAN interna o que permite que cada rede VLAN tenha endereços IPv6 únicos. Abaixo estão as sugestões de prefixos IPv6 para cada localidade:

- **Oporto**: `2001:db8:10::/48`
- O sufixo 10 foi escolhido para identificar a rede de Oporto.
- A máscara /48 significa que os primeiros 48 bits do endereço definem a rede, deixando os bits restantes para criar sub-redes.
- Abaixo deste prefixo /48, é possível criar várias sub-redes /64 para as VLANs de Oporto.
  - VLAN STAFF: `2001:db8:10:10::/64`
  - VLAN ACCOUNTING: `2001:db8:10:20::/64`
  - VLAN HR: `2001:db8:10:30::/64`
  - VLAN USERS: `2001:db8:10:40::/64`

- **Warsaw**: `2001:db8:11::/48`
- O sufixo 11 foi usado para identificar a rede de Varsóvia.
- Cada VLAN em Warsaw recebe um novo sufixo, permitindo que haja redes IPv6 exclusivas para cada departamento ou VLAN.
  - VLAN ACCOUNTING: `2001:db8:11:20::/64`
  - VLAN USERS: `2001:db8:11:40::/64`

- **Munich**: `2001:db8:12::/48`
- O sufixo 12 identifica a rede de Munique.
- Esse prefixo /48 pode ser dividido em sub-redes /64, criando identificadores específicos para VLANs nesta localidade.
  - VLAN STAFF: `2001:db8:12:10::/64`
  - VLAN USERS: `2001:db8:12:40::/64`

Essas sub-redes devem ser aplicadas de forma consistente em todas as filiais para facilitar o roteamento e a administração.

**No Ambiente Real**

Para uma rede corporativa real, a empresa deve: 
- Obter um prefixo IPv6 de um ISP, que pode ser um /48 para grandes empresas ou um /56 para empresas menores.
- Subdividir esse prefixo para suas redes internas, mantendo a hierarquia, tal como descrito, para facilitar o roteamento e a administração interna.

## 2. Atribuição de Endereços IPv6 aos Dispositivos

Para distribuir endereços IPv6 aos dispositivos, recomendamos duas abordagens:

1. **SLAAC (Stateless Address Autoconfiguration)**: Permite que os dispositivos configurem automaticamente os seus endereços IPv6 com base nos prefixos anunciados pelos routers.
   - Usado para redes que não necessitam de um controlo rígido de IP, como redes de uso geral (ex.: VLAN USERS).
   
2. **DHCPv6**: Utilizado para controlo centralizado, como nos segmentos de rede `ACCOUNTING`, onde é necessária uma gestão mais cuidadosa dos endereços IP.
   - DHCPv6 permite o rastreamento de dispositivos e oferece opções adicionais, como atribuição de DNS.
   
Cada VLAN deve ser configurada para suportar a distribuição de endereços IPv6, de acordo com a necessidade de gestão e segurança de cada rede.

## 3. Roteamento Dinâmico com IPv6

Para garantir a conectividade e o roteamento eficiente entre localidades, a RECOMP deve implementar roteamento dinâmico com IPv6. As recomendações são:

- **OSPFv3**: Recomendado para roteamento dentro de cada localidade, com áreas específicas para redes locais e túneis GRE que conectam as localidades.
  - **Área 0** para túneis entre Oporto e Warsaw, e entre Oporto e Munich.
  - **Área 1** para redes locais de Oporto.
  - **Área 2** para redes locais de Warsaw.
  - **Área 3** para redes locais de Munich.

- **EIGRP para IPv6**: Deve ser configurado nas interfaces dos túneis GRE para garantir conectividade entre as localidades (Oporto-Munich e Oporto-Warsaw) e facilitar a redistribuição entre OSPF e EIGRP.

## 4. Questões de Segurança para IPv6

A introdução de IPv6 traz novos desafios de segurança. Recomendamos as seguintes medidas:

- **RA Guard e DHCP Guard**: Para evitar ataques de anúncios de router e DHCP não autorizados, que poderiam comprometer a rede.
- **ACLs IPv6**: Assim como em IPv4, as ACLs devem ser configuradas para controlar o tráfego IPv6. As ACLs para IPv6 devem incluir:
  - Bloqueio de endereços multicast não necessários.
  - Controlo de tráfego entre as VLANs e redes internas.
  - Restrição de acesso aos routers e outros dispositivos críticos.

## 5. Transição e Coexistência com IPv4

Durante a transição, recomendamos o uso de uma **configuração dual-stack** (suporte simultâneo a IPv4 e IPv6) em todos os dispositivos e routers, até que a rede esteja pronta para operar exclusivamente com IPv6. Isso permitirá:

- **Acesso contínuo** a serviços que ainda dependem de IPv4.
- **Monitorização e ajustes** de desempenho e segurança para IPv6 em um ambiente controlado.

## Conclusão

A implementação de IPv6 proporcionará maior flexibilidade e escalabilidade para a RECOMP Corporation. Esse plano de transição considera as necessidades de segurança, roteamento e gestão de endereços para uma coexistência com IPv4 sem interrupções. Recomendamos um período de teste e monitorização, com atualizações regulares para garantir que a migração ocorra de maneira eficaz e segura.
