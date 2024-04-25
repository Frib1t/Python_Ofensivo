# Escaner
## Colores
```
pip3 install termcolor
```
## Argparse

> Permitirá personalizar las opciones de ejecución.


# Código

> Escáner de puertos a través de sockets.  Uso con el Parámetro "-p" puerto, y el "-t" target:

- Parámetro "-h" (Help)

![Pasted image 20240425181349](https://github.com/Frib1t/Python_Ofensivo/assets/102589078/96fc9268-e3f8-4087-9024-c995690f8e6f)


- Uso con rango de puertos:

![Pasted image 20240425181632](https://github.com/Frib1t/Python_Ofensivo/assets/102589078/8793b607-049c-40b9-9e6b-2905acced3b6)


- Uso con un solo puerto:

![Pasted image 20240425181729](https://github.com/Frib1t/Python_Ofensivo/assets/102589078/d962a385-8272-4545-baa4-b00ae3764095)



```python
#!/usr/bin/env python3
import socket
import argparse
import signal
import sys
from concurrent.futures import ThreadPoolExecutor
from termcolor import colored

open_sockets = []

def signal_handler(sig, frame):
    print(colored(f"\n[!] Saliendo del programa...", 'red'))
    for sock in open_sockets:
        sock.close()
    sys.exit(1)

signal.signal(signal.SIGINT, signal_handler)

def get_arguments():
    parser = argparse.ArgumentParser(description='Fast TCP Port Scanner')
    parser.add_argument("-t", "--target", dest="target", required=True,
                        help="Victim target to scan (Ex: -t 192.168.1.1)")
    parser.add_argument("-p", "--port", dest="port", required=True, help="Port range to scan (Ex: -p 1-100)")
    options = parser.parse_args()

    return options.target, options.port

def create_socket():
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.settimeout(0.1)
    open_sockets.append(s)
    return s

def port_scanner(port, host):
    s = create_socket()
    try:
        s.connect((host, port))
        s.sendall(b"HEAD / HTTP/1.0 \r\n\r\n")
        response = (s.recv(1024)).decode(errors='ignore').split('\n')
        if response:
            print(colored(f"\n[+] El puerto {port} está abierto", 'green'))
            for line in response:
                print(colored(f"{line}", 'white'))
        else:
            print(colored(f"\n[+] El puerto {port} está abierto", 'green'))
    except (socket.timeout, ConnectionRefusedError):
        pass
    finally:
        s.close()

def scan_ports(ports, target):
    with ThreadPoolExecutor(max_workers=100) as executor:
        executor.map(lambda port: port_scanner(port, target), ports)

def parse_ports(ports_str):
    if '-' in ports_str:
        start, end = map(int, ports_str.split('-'))
        return range(start, end + 1)
    elif ',' in ports_str:
        return map(int, ports_str.split(','))
    else:
        return (int(ports_str),)

def main():
    target, ports_str = get_arguments()
    ports = parse_ports(ports_str)
    scan_ports(ports, target)

if __name__ == '__main__':
    main()

```


## código comentado
```python
#!/usr/bin/env python3
import socket
import argparse
import signal
import sys
from concurrent.futures import ThreadPoolExecutor
from termcolor import colored

open_sockets = []


def signal_handler(sig, frame):
    """Maneja la señal de interrupción de teclado (Ctrl+C)"""
    print(colored(f"\n[!] Saliendo del programa...", 'red'))
    for s in open_sockets:
        s.close()

    sys.exit(1)


signal.signal(signal.SIGINT, signal_handler)


def get_arguments():
    """Obtiene los argumentos del usuario"""
    parser = argparse.ArgumentParser(description='Fast TCP Port Scanner')
    parser.add_argument("-t", "--target", dest="target", required=True,
                        help="Victim target to scan (Ex: -t 192.168.1.1)")
    parser.add_argument("-p", "--port", dest="port", required=True, help="Port range to scan (Ex: -p 1-100)")
    options = parser.parse_args()

    return options.target, options.port



def create_socket():
    """Crea un nuevo socket"""
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.settimeout(0.1)

    open_sockets.append(s)

    return s


def port_scanner(port, host):
    """Escanea un puerto específico"""
    s = create_socket()

    try:
        s.connect((host, port))
        s.sendall(b"HEAD / HTTP/1.0 \r\n\r\n")
        response = (s.recv(1024)).decode(errors='ignore').split('\n')

        if response:
            print(colored(f"\n[+] El puerto {port} está abierto", 'green'))

            for line in response:
                print(colored(f"{line}", 'grey'))
        else:
            print(colored(f"\n[+] El puerto {port} está abierto", 'green'))

    except (socket.timeout, ConnectionRefusedError):
        pass

    finally:
        s.close()


def scan_ports(ports, target):
    """Escanea una lista de puertos"""
    with ThreadPoolExecutor(max_workers=100) as executor:
        executor.map(lambda port: port_scanner(port, target), ports)


def parse_ports(ports_str):
    """Analiza la cadena de puertos especificada por el usuario"""
    if '-' in ports_str:
        start, end = map(int, ports_str.split('-'))
        return range(start, end + 1)
    elif ',' in ports_str:
        return map(int, ports_str.split(','))
    else:
        return (int(ports_str),)


def main():
    """Función principal"""
    target, ports_str = get_arguments()
    ports = parse_ports(ports_str)
    scan_ports(ports, target)


if __name__ == '__main__':
    main()

```

