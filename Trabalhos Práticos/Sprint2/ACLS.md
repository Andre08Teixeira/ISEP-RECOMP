# Sprint2


- **ACLs**

    Nesta secção foi pedido que fosse bloqueado o spoofing interno e externo nos routers dos três edifícios. Para além disso, que fosse bloqueado todo o tráfego dirigido aos routers, com exceção do necessário para que as funcionalidades implementadas continuem em funcionamento. Por fim, foi pedido que o restante tráfego fosse permitido.

    De forma a bloquear tráfego indesejado e prevenir o spoofing, foram criadas regras para *access-lists* (ACLs), que foram configuradas nos routers de cada edificio. Estas regras são:

- OPORTO:

```shell
ACL 100 - Controlo de Tráfego Externo (inbound na interface externa):

access-list 100 deny ip any host 10.27.71.193  # Nega ping para o IP da interface Gig0/0 do router HQ
access-list 100 deny ip any host 10.27.71.197  # Nega ping para o IP da interface Gig0/1 do router HQ
access-list 100 permit gre any any             # Permite protocolo GRE
access-list 100 permit ospf any any            # Permite protocolo OSPF
access-list 100 permit eigrp any any           # Permite protocolo EIGRP
access-list 100 permit icmp any 209.165.200.129 0.0.0.31 echo-reply   # Permite respostas ICMP (ping) do router
access-list 100 permit tcp any eq 80 209.165.200.129 0.0.0.31 established   # Permite tráfego TCP para HTTP (porta 80)
access-list 100 permit tcp any eq 443 209.165.200.129 0.0.0.31 established  # Permite tráfego TCP para HTTPS (porta 443)
access-list 100 permit udp any eq 53 209.165.200.129 0.0.0.31              # Permite DNS UDP
access-list 100 deny ip any host 209.165.200.129 # Nega pacotes externos para o IP público da interface do router
access-list 100 deny ip 10.27.68.0 0.0.3.255 any  # Bloqueia spoofing de IPs internos na saída, ou seja bloqueia todos os ips que vêm de fora com um ip de rede interna
access-list 100 deny ip host 0.0.0.0 any           # Bloqueia tráfego com IP de origem 0.0.0.0
access-list 100 deny ip 127.0.0.0 0.255.255.255 any  # Bloqueia outras redes privadas
access-list 100 permit ip 172.21.72.0 0.0.1.255 any  # Permite pacotes provenientes do edificio Munich
access-list 100 deny ip 172.16.0.0 0.15.255.255 any  # Bloqueia outras redes privadas
access-list 100 permit ip 192.168.154.0 0.0.1.255 any  # Permite pacotes provenientes do edificio Warsaw 
access-list 100 deny ip 192.168.0.0 0.0.255.255 any   # Bloqueia outras redes privadas
access-list 100 deny ip 224.0.0.0 15.255.255.255 any  # Bloqueia endereços multicast
access-list 100 deny ip host 255.255.255.255 any      # Bloqueia broadcast
access-list 100 deny ip any any                       # Bloqueia todos os outros pacotes

ACL 101 - Controlo de Tráfego Interno (inbound na interface interna):

access-list 101 permit udp any any eq bootps           # Permite o protocolo DHCP
access-list 101 deny ip any host 10.27.71.193          # Nega ping para IP local do router (Gig0/0)
access-list 101 deny ip any host 10.27.71.197          # Nega ping para IP local do router (Gig0/1)
access-list 101 permit ip 10.27.68.0 0.0.3.255 any     # Permite pacotes internos para endereços válidos
access-list 101 deny ip any any                        # Bloqueia todos os outros pacotes
```
    Estas ACLs foram aplicadas às interfaces respetivas com os comandos:

```shell
interface GigabitEthernet0/0/0
 ip access-group 100 in   # Aplicar a ACL de tráfego externo na interface de entrada
interface GigabitEthernet0/0
 ip access-group 101 in   # Aplicar a ACL de tráfego interno na interface de entrada
interface GigabitEthernet0/1
 ip access-group 101 in   # Aplicar a ACL de tráfego interno na interface de entrada

```
- WARSAW

```shell
ACL 100 - Controlo de Tráfego Externo (Aplicada na interface externa GigabitEthernet0/0/0):

access-list 100 permit gre any any                          
access-list 100 permit ospf any any                         
access-list 100 permit eigrp any any                        
access-list 100 permit icmp any 192.0.2.97 0.0.0.15 echo-reply  # Permite respostas de ICMP (ping) para a rede pública do router
access-list 100 permit tcp any eq 80 192.0.2.97 0.0.0.15 established  # Permite tráfego TCP para conexões HTTP
access-list 100 permit tcp any eq 443 192.0.2.97 0.0.0.15 established # Permite tráfego TCP para conexões HTTPS
access-list 100 permit udp any eq 53 192.0.2.97 0.0.0.15              # Permite consultas DNS UDP

# Nega pacotes externos direcionados ao IP da interface pública de Varsóvia
access-list 100 deny ip any host 192.0.2.97

# Bloqueia spoofing de endereços internos saindo pelo router de Warsaw
access-list 100 deny ip 192.168.154.0 0.0.1.255 any  # Bloqueia tráfego spoofing das sub-redes locais de Warwaw
access-list 100 deny ip host 0.0.0.0 any             # Bloqueia IPs de origem 0.0.0.0
access-list 100 deny ip 127.0.0.0 0.255.255.255 any  # Bloqueia outras redes privadas

# Permite tráfego de outras localidades (Oporto e Munich)
access-list 100 permit ip 10.27.68.0 0.0.3.255 any    # Permite tráfego da rede de Oporto (HQ)
access-list 100 permit ip 172.21.72.0 0.0.1.255 any   # Permite tráfego da rede de Munich

# Bloqueia outras redes privadas e endereços não roteáveis
access-list 100 deny ip 172.16.0.0 0.15.255.255 any    # Bloqueia redes privadas da faixa 172.16.0.0/12
access-list 100 deny ip 192.168.0.0 0.0.255.255 any    # Bloqueia redes privadas da faixa 192.168.0.0/16
access-list 100 deny ip 224.0.0.0 15.255.255.255 any   # Bloqueia endereços multicast
access-list 100 deny ip host 255.255.255.255 any       # Bloqueia broadcasts

# Bloqueia todos os pacotes restantes (padrão de segurança)
access-list 100 deny ip any any

ACL 101 - Controlo de Tráfego Interno (Aplicada nas interfaces internas GigabitEthernet0/0.20 e GigabitEthernet0/0.40):

access-list 101 permit udp any any eq bootps                    # Permite DHCP (para atribuição de IPs dinâmicos)
access-list 101 deny ip any host 192.168.155.254                # Nega ping para o IP da VLAN ACCOUNTING (interface local)
access-list 101 deny ip any host 192.168.154.254                # Nega ping para o IP da VLAN USERS (interface local)
access-list 101 permit ip 192.168.154.0 0.0.1.255 any           # Permite pacotes internos das sub-redes locais VLAN USERS e ACCOUNTING
access-list 101 deny ip any any                                # Bloqueia todos os pacotes restantes
```
    Estas ACLs foram aplicadas às interfaces respetivas com os comandos:

```shell
interface GigabitEthernet0/0/0
 ip access-group 100 in      # Aplicar ACL de controlo de tráfego externo na entrada

interface GigabitEthernet0/0.20
 ip access-group 101 in      # Aplicar ACL de controlo de tráfego interno (VLAN ACCOUNTING)

interface GigabitEthernet0/0.40
 ip access-group 101 in      # Aplicar ACL de controlo de tráfego interno (VLAN USERS)
```

```shell
Em suma...
ACL 100:
Controla o tráfego de entrada na interface pública (GigabitEthernet0/0/0).
Permite protocolos essenciais como GRE, OSPF, EIGRP e ICMP (echo-reply).
Permite conexões HTTP, HTTPS e DNS para a rede pública de Varsóvia.
Bloqueia o spoofing de endereços internos e redes privadas não autorizadas.

ACL 101:
Controla o tráfego de entrada nas interfaces internas VLAN ACCOUNTING e VLAN USERS.
Permite o tráfego DHCP para configuração automática de IPs.
Restringe ping para o IP das interfaces locais e bloqueia pacotes não autorizados.
Essas configurações de ACL protegem o router de Varsóvia, restringindo o tráfego externo e protegendo as sub-redes internas de acordo com as diretrizes de segurança do projeto.
```

- MUNICH

```shell
ACL 100 - Controlo de Tráfego Externo (Aplicada na interface externa GigabitEthernet0/0/0):

access-list 100 permit gre any any                          
access-list 100 permit ospf any any                         
access-list 100 permit eigrp any any                        
access-list 100 permit icmp any 193.136.60.147 0.0.0.7 echo-reply  # Permite respostas de ICMP (ping) para a rede pública do router
access-list 100 permit tcp any eq 80 193.136.60.147 0.0.0.7 established  # Permite tráfego TCP para conexões HTTP
access-list 100 permit tcp any eq 443 193.136.60.147 0.0.0.7 established # Permite tráfego TCP para conexões HTTPS
access-list 100 permit udp any eq 53 193.136.60.147 0.0.0.7              # Permite consultas DNS UDP

# Nega pacotes externos direcionados ao IP da interface pública de Munich
access-list 100 deny ip any host 193.136.60.147

# Bloqueia spoofing de endereços internos saindo pelo router de Munich
access-list 100 deny ip 172.21.72.0 0.0.0.255 any  # Bloqueia tráfego spoofing da sub-rede VLAN USERS
access-list 100 deny ip 172.21.73.0 0.0.0.255 any  # Bloqueia tráfego spoofing da sub-rede VLAN STAFF
access-list 100 deny ip host 0.0.0.0 any           # Bloqueia IPs de origem 0.0.0.0
access-list 100 deny ip 127.0.0.0 0.255.255.255 any  # Bloqueia outras redes privadas

# Permite tráfego de outras localidades (Oporto e Warsaw)
access-list 100 permit ip 10.27.68.0 0.0.3.255 any   # Permite tráfego da rede de Oporto 
access-list 100 permit ip 192.168.154.0 0.0.1.255 any  # Permite tráfego da rede de Warsaw 

# Bloqueia outras redes privadas e endereços não roteáveis
access-list 100 deny ip 172.16.0.0 0.15.255.255 any    # Bloqueia redes privadas da faixa 172.16.0.0/12
access-list 100 deny ip 192.168.0.0 0.0.255.255 any    # Bloqueia redes privadas da faixa 192.168.0.0/16
access-list 100 deny ip 224.0.0.0 15.255.255.255 any   # Bloqueia endereços multicast
access-list 100 deny ip host 255.255.255.255 any       # Bloqueia broadcasts

# Bloqueia todos os pacotes restantes (padrão de segurança)
access-list 100 deny ip any any

ACL 101 - Controlo de Tráfego Interno (Aplicada nas interfaces internas GigabitEthernet0/0.10 e GigabitEthernet0/0.40):

access-list 101 permit udp any any eq bootps                    # Permite DHCP (para atribuição de IPs dinâmicos)
access-list 101 deny ip any host 172.21.73.1                    # Nega ping para o IP da VLAN STAFF (interface local)
access-list 101 deny ip any host 172.21.72.1                    # Nega ping para o IP da VLAN USERS (interface local)
access-list 101 permit ip 172.21.72.0 0.0.1.255 any            # Permite pacotes internos das sub-redes locais VLAN USERS e STAFF
access-list 101 deny ip any any                                # Bloqueia todos os pacotes restantes
```

    Estas ACLs foram aplicadas às interfaces respetivas com os comandos:

```shell
interface GigabitEthernet0/0/0
 ip access-group 100 in      # Aplicar ACL de controlo de tráfego externo na entrada

interface GigabitEthernet0/0.10
 ip access-group 101 in      # Aplicar ACL de controlo de tráfego interno (VLAN STAFF)

interface GigabitEthernet0/0.40
 ip access-group 101 in      # Aplicar ACL de controlo de tráfego interno (VLAN USERS)
```

```shell
Em suma...
ACL 100:
Controla o tráfego de entrada na interface pública (GigabitEthernet0/0/0).
Permite protocolos essenciais como GRE, OSPF, EIGRP e ICMP (echo-reply).
Permite conexões HTTP, HTTPS e DNS para a rede pública de Munique.
Bloqueia o spoofing de endereços internos e redes privadas não autorizadas.

ACL 101:
Controla o tráfego de entrada nas interfaces internas VLAN STAFF e VLAN USERS.
Permite o tráfego DHCP para configuração automática de IPs.
Restringe ping para o IP das interfaces locais e bloqueia pacotes não autorizados.
Essas configurações de ACL fornecem segurança ao router de Munique, limitando o tráfego externo e protegendo as sub-redes internas.
```
