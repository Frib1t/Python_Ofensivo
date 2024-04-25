Creación de un escáner de red por ICMP, para detectar dispositivos activos en una red.

> El uso es mediante el parámetro "-t" para pasarle la IP objetivo o el rango con "-". El menú de ayuda se despliega con -h.

-  "-h":
![Pasted image 20240425174753](https://github.com/Frib1t/Python_Ofensivo/assets/102589078/2c22dda7-120a-4bef-a928-a8c66f6e14b7)


- "-" Rango:
![Pasted image 20240425174538](https://github.com/Frib1t/Python_Ofensivo/assets/102589078/fb78bfef-535b-45a9-add7-7052b4f0020b)


- "-t" IP única:
![Pasted image 20240425174647](https://github.com/Frib1t/Python_Ofensivo/assets/102589078/2cb24671-f643-4029-9963-dd11cc8a5bca)



```python
#!/usr/bin/env python3

import signal
import argparse
import subprocess
import sys
from concurrent.futures import ThreadPoolExecutor
from termcolor import colored

def def_handler(sig, frame):
	print(colored(f"\n[!] Saliendo del Programa", 'red'))
	sys.exit(1)


signal.signal(signal.SIGINT, def_handler)

def get_arguments():
	parser = argparse.ArgumentParser(description="Herramienta para descubrir hasost activos en una red (ICMP)")
	parser.add_argument("-t", "--target", required=True, dest="target", help="Host o rango de red a escanear")

	args = parser.parse_args()
	return args.target


def parse_target(target_str):
	# 192.168.1.1-100
	target_str_splitted = target_str.split('.') # ["192","168","1","1-100"]
	first_three_octets = '.'.join(target_str_splitted[:3]) # 192.168.1
	
	if len(target_str_splitted) == 4:
		if "-" in target_str_splitted[3]:
			start, end = target_str_splitted[3].split('-')
			return [f"{first_three_octets}.{i}" for i in range(int(start), int(end)+1)]
		else:
			return [target_str]
	else:
		print(colored(f"\n[!] El fomrato de IP o rango de IP no es valido\n", 'red'))


def host_discovery(target):
	try:
		ping = subprocess.run(["ping", "-c", "1", target], timeout=1, stdout=subprocess.DEVNULL) # codigo de estado 0 exito, 1 fracaso
		if ping.returncode == 0:
			print(colored(f"\t[i] La IP {target} está activa", 'green'))
			
	except subprocess.TimeoutExpired:
		pass


def main():
	target_str = get_arguments()
	targets = parse_target(target_str)
	print(f"\n[+] Hosts activos en la red:\n")

	max_threads = 100

	with ThreadPoolExecutor(max_workers=max_threads) as executor:
		executor.map(host_discovery, targets) # funcion, iterable


if __name__ == '__main__':
	main()
	
```

----
