## Task 1.1A:
Nesta tarefa, começamos por abrir o docker do seedlab desta semana e verificar que aparecem os 3 containers essenciais com o comando `dockps`: seed-attacker, hostA e hostB.

Com o comando `ifconfig`, nós conseguimos identificar a interface da VM que estamos a usar para os containers: `br-4db7ef92b2f9`.

De seguida, dentro da pasta `volumes`, críamos um ficheiro chamado `sniffer.py` e inserimos o seguinte excerto com a nossa interface:

```python
from scapy.all import *

def print_pkt(pkt):
    pkt.show()
    
pkt = sniff(iface='br-4db7ef92b2f9', filter='icmp', prn=print_pkt)
```

Agora, vamos enviar pacotes usando o comando `ping`. Para isso, vamos ao container que corresponde ao hostA e damos `ping` do hostB. Se abrirmos o wireshark, e selecionarmos a interface descoberta obtémos isto:

![](img/wireshark.png)

Com isto, sabemos que os pacotes estão a ser enviados. Agora, vamos usar o excerto de python e testar com premissões root e sem permissões e ver o que acontece com este comando `python3 sniffer.py`:

- Com privilégios root (usando `sudo` antes):

![](img/t1.1A_p1_lb13.png)


- Se mudarmos para a conta seed e não usarmos privilégios root, isto acontece:

![](img/t1.1A_p2_lb13.png)


## Task 1.1B:
Nesta tarefa, começamos por relembrar que fizemos a captura de pacotes ICMP filtrados com o sniffer.py na tarefa anterior com os privilégios root.

A seguir, para capturar os pacotes TCP, começamos por substituir na varíavel `pkt` o filter usado da seguinte forma:
```
pkt = sniff(iface='br-4db7ef92b2f9', filter='tcp and dst port 23', prn=print_pkt)
```

Depois, executamos o ficheiro `sniffer.py` novamente, e paralelamente, abrímos o container do hostA e usámos o comando `telnet hostB-10.9.0.6 23`. Com isto, são imprimos pacotes com este formato:

![](img/t1.1B_p2_lb13.png)

De seguida, para capturar pacotes que veêm de uma subnete particular, voltamos a mudar o filter do `pkt` no ficheiro `sniffer.py` usado anteriormente, da seguinte maneira:
```
pkt = sniff(iface='br-4db7ef92b2f9', filter='net 128.230.0.0/16', prn=print_pkt)
```

No final, executamos o ficheiro `sniffer.py` e, com container hostA, fizemos ping do endereço `128.230.0.1`. Na shell, dentro da pasta volumes onde executamos o ficheiro, obtiemos pacotes assim:

![](img/t1.1B_p3_lb13.png)


## Task 1.2

Nesta tarefa, começamos por criar um ficheiro `spoof.py` da seguinte forma:

```python
#!/usr/bin/env python3
from scapy.all import *

ip = IP()
ip.src = "10.9.0.5"  # host A
ip.dst = "10.9.0.6"  # host B

icmp = ICMP()
packet = ip / icmp

print(f"Sending spoofed ICMP packet from {ip.src} to {ip.dst}...")
send(packet) 
print("Packet sent!")
```

Depois abrimos o wireshark no caminho `any`. Paralelamente, executamos o ficheiro criado com o comando `sudo python3 spoof.py`
e verificamos o resultado pretendido:

![](img/t1.2_p1_lb13.png)

![](img/t1.2_p2_lb13.png)

Assim, nós conseguimos enviar um pacote ICMP falsificado com o script criado.

## Task 1.3

Nesta tarefa, começamos por criar um ficheiro `traceroute.py` que substitui o comando traceroute da VM, desta forma:
```python
from scapy.all import *

def simple_traceroute(destination, max_hops=30):
    print(f"Tracing route to {destination} with a maximum of {max_hops} hops...\n")
    for ttl in range(1, max_hops + 1):
        pkt = IP(dst=destination, ttl=ttl) / ICMP()  # Create ICMP packet with TTL
        reply = sr1(pkt, verbose=0, timeout=2)  # Send packet and wait for a reply

        if reply is None:
            print(f"{ttl}: * * * (Request timed out)")
            continue

        print(f"{ttl}: {reply.src}")

        if reply.type == 0:  
            print(f"Destination {destination} reached in {ttl} hops.")
            break
    else:
        print(f"Could not reach {destination} within {max_hops} hops.")

destination = "8.8.8.8" 
simple_traceroute(destination)
```
Nós críamos um loop que faz variar o ttl até ao limite máximo de 30, semelhante ao que que acontece com o comando traceroute.

Depois, executamos o ficheiro e comparamos os resultados com o que aconteceria se usássemos o comando `traceroute 8.8.8.8 -I`. A flag `-I` foi usada para reforçar o uso de pacotes ICMP ao invés de usar pacotes UDP.

![](img/t1.3_lb13.png)

Podemos concluir, que o traceroute foi criado corretamente uma vez que os resultados são idênticos.

## Task 1.4
Nesta tarefa, começamos por criar o ficheiro `sniff_spoof.py` que envia um pacote falso quando vê um pacote a entrar.

```python
#!/usr/bin/env python3
from scapy.all import *

def send_reply(packet):
    if packet.haslayer(ICMP) and packet[ICMP].type == 8:
       
        ip = IP(src=packet[IP].dst, dst=packet[IP].src)
        icmp = ICMP(type=0, id=packet[ICMP].id, seq=packet[ICMP].seq)
        
        data = packet[Raw].load if packet.haslayer(Raw) else b''

        reply = ip / icmp / data
        send(reply, verbose=0)
        print(f"Sent ICMP reply to {packet[IP].src}")
        

packet = sniff(iface='br-4db7ef92b2f9', filter='icmp', prn=send_reply)
```

Para criar este ficheiro, usámos o codigo usado na task 1.1 e na task 1.2 e adaptamos para esta tarefa (tendo em conta os valores obtidos nessas mesmas tarefas, nomeadamente, o valor da interface descoberto `br-4db7ef92b2f9`).

De seguida, nós executamos este ficheiro na VM com o comando `sudo python3 sniff_spoof.py` e paralelamente testamos o `ping` para endereços diferentes no container `hostA`:

#### `1.2.3.4`

![](img/1.2.3.4_lb13.png)

![](img/1.2.3.4_ws.png)

Foram capturados pacotes ICMP enviados para um endereço fictício. O Wireshark mostra:

- Pacotes Echo Request de 10.9.0.5 para 1.2.3.4.
- Pacotes Echo Reply falsificados de 1.2.3.4 para 10.9.0.5, simulando que o endereço estava ativo.

O script funcionou conforme esperado, enviando respostas ICMP falsificadas para um endereço fictício.

#### `10.9.0.99`

![](img/10.9.0.99_lb13.png)

![](img/10.9.0.99_ws.png)

Foram enviados apenas pacotes ARP (Who has 10.9.0.99? Tell 10.9.0.5) para a resolução de endereço MAC. Nenhum pacote ICMP foi enviado ou capturado para este endereço. Isto acontece porque o endereço não existe em LAN e por isso é impossivel estabelecer uma conexão (não é possivel encontrar o MAC de destino ao usar pacotes ARP).

#### `8.8.8.8`

![](img/8.8.8.8_lb13.png)

![](img/8.8.8.8_ws.png)

Foram capturados pacotes ICMP enviados para o servidor DNS público do Google. O Wireshark mostra:

- Pacotes Echo Request saindo do endereço 10.9.0.5 em direção a 8.8.8.8.
- Pacotes Echo Reply falsificados, retornando de 8.8.8.8 para 10.9.0.5.

O script conseguiu interceptar e falsificar as respostas ICMP mesmo quando o destino era um servidor externo real.
