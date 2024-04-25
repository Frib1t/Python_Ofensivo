Programa para cambiar la MAC de la tarjeta de RED del sistema.

```python
#!/usr/bin/env python3
import argparse
import re
import subprocess
import signal
import sys
from termcolor import colored


def def_handler(sig, frame):
    print(colored(f"\n[!] Saliendo del Programa\n"))
    sys.exit(1)


signal.signal(signal.SIGINT, def_handler)


def get_arguments():
    parser = argparse.ArgumentParser(description="Herramienta para cambiar la dirección Mac de una interfaz de red")
    parser.add_argument("-i", "--interface", required=True, dest="interface", help="Nombre de la interfaz de red")
    parser.add_argument("-m","--mac", required=True, dest="mac_address", help="Nueva dirección MAC para la interfaz de red")

    return parser.parse_args()


def is_valid_input(interface, mac_address):

    is_valid_interface = re.match(r'^[e][n|t][s|h]\d{1,2}$', interface)
    is_valid_mac_address = re.match(r'^([A-Fa-f0-9]{2}[:]){5}[A-Fa-f0-9]{2}$', mac_address)

    return is_valid_interface and is_valid_mac_address



def change_mac_address(interface, mac_address):

    if is_valid_input(interface, mac_address):
        subprocess.run(["ifconfig", interface, "down"])
        subprocess.run(["ifconfig", interface, "hw", "ether", mac_address])
        subprocess.run(["ifconfig", interface, "up"])

        print(colored(f"\n[+] La MAC ha sido cambiada exitosamente\n", 'green'))
        """     
        Con este metodo (subproces) evitas inyecciones de comandos por ejemplo,                                                                  
        que pongan una interfaz segido de un ; y un whoami por ejemplo,                     
        y que puedan escapar, ademas que hacemos una validación previa.
        """
    else:
        print(colored(f"\n[!]Los dat6os introducidos no son correctos", 'red'))


def main():
    args = get_arguments()
    change_mac_address(args.interface, args.mac_address)



if __name__ == '__main__':
    main()
```

![Pasted image 20240425122018](https://github.com/Frib1t/Python_Ofensivo/assets/102589078/71cbc8b3-b9d3-482a-8371-6640f34980a3)


- Ejecutar el script con sudo por tema de permisos a la hora de cambiar la MAC
- Para verificar el cambio de MAC o listar un vendor de MAC usar macchanger.
- Si quieres deshacer el cambio, ejecuta macchanger y usa la listada original.

```bash
macchanger -l # lista MAC's
macchanger -s <Nuestra interfaz de red> # verifica el cambio y te lista la original
```

![Pasted image 20240425122501](https://github.com/Frib1t/Python_Ofensivo/assets/102589078/cd4e91e6-7ac6-43de-b193-0c216fcac666)

