> Los dispositivos reciben la máscara de red y la dirección IP de manera automática y flexible cuando se establece la conexión con una red. Con este objetivo, los dispositivos de comunicación mediadores como routers o concentradores (hubs) recurren al protocolo DHCP. En las redes locales se pueden introducir ambos datos manualmente. El **fabricante del dispositivo correspondiente** otorga la dirección de hardware, que queda vinculada a una dirección IP con ayuda del llamado Address Resolution Protocol (ARP).

> Encuentra la dirección de hardware que corresponde a una IP

![Pasted image 20240426130513](https://github.com/Frib1t/Python_Ofensivo/assets/102589078/fa56c474-703e-4457-b70e-8ce553d464e1)


```python
#!/usr/bin/env python3
import scapy.all as scapy
import argparse

def get_arguments():
	parser = argparse.ArgumentParser(description="ARP Scanner")
	parser.add_argument("-t", "--target", required=True, dest="target", help="Host / IP Range to Scan")
	args = parser.parse_args()
	return args.target


def scan(ip):
	arp_packet = scapy.ARP(pdst=ip) # protocolDestination
	broadcast_packet = scapy.Ether(dst="ff:ff:ff:ff:ff:ff") # MAC destination

	arp_packet = broadcast_packet/arp_packet # Es un operador de composicion para unir protocolos de paquetes
	answered, unanswered = scapy.srp(arp_packet, timeout=1, verbose=False)
	
	response = answered.summary()
	if response:
		print(response)
		
	scapy.arping(ip)


def main():
	target = get_arguments()
	scan(target)


if __name__ == '__main__':
	main()

```

