# PRÁCTICA 2 ASR : VM

Luis Foncillas Gutiérrez -- arquitecturas de servicios de red 2023

## SOLUCIÓN 1
### VMs y firewall básico
Se monta una máquina de salto y un servidor web, instalando un firewall de capa 4 básico
![vmssol1](https://github.com/luisfoncig/asr-23/assets/145972379/03add51d-fbe9-42c6-8eee-e7c5e4c12046)
![firewallsol1](https://github.com/luisfoncig/asr-23/assets/145972379/e819eb90-d611-4e86-bbb1-d7c27149f55f)

### Conexiones
tras esto se comprueban conexiones: primero de local a servidor salto, y después de salto a servidor web
![sshajumpserver](https://github.com/luisfoncig/asr-23/assets/145972379/b8a7b242-ca59-41a0-9a18-7ec3927a3295)
![sshjumpweb](https://github.com/luisfoncig/asr-23/assets/145972379/0d1dfd8e-84d7-46d3-8ca9-8f4f0bd666bc)


## SOLUCIÓN 2
### Balanceador de carga
antes de comenzar con el balanceador de carga, le quitamos la ip pública al servidor web. Después de esto solo podremos acceder al servidor web a través de la ip privada (10.0.0.2)

![servwebnoip](https://github.com/luisfoncig/asr-23/assets/145972379/6f5136d0-9435-4c02-a46c-f0b84a9b1d73)

### Instalación nginx
La siguiente tarea es instalar nginx en el servidor web, pero al no tener ip pública el acceso a internet se dificulta. 
Para solucionar esto, hay que instalar una cloud NAT asignada a nuestra subred:

![cloudnat](https://github.com/luisfoncig/asr-23/assets/145972379/fd86604c-ff09-4c78-a93f-9015ef02a785)

lo que nos permite instalar nginx correctamente.

![installnginx](https://github.com/luisfoncig/asr-23/assets/145972379/35f9bcec-df5a-4a52-b060-95d3853405b7)

### Certificados

Tras esto, se generan 
las claves ssl necesarias para tener la correcta certificación, las cuales insertamos en el backend de nuestro 
balanceador de carga.

![certs](https://github.com/luisfoncig/asr-23/assets/145972379/68c25cbf-15c9-4bbd-8caf-7a4ba24fe545)

### WAF

Para finalizar el apartado, hay que ajustar la firewall para tener condiciones a nivel de aplicación.
Se nos requiere restringir tráfico a países que no sean de confianza de la UE, y proteger de ataques SQL injection y cross syte scripting:

![WAFRules](https://github.com/luisfoncig/asr-23/assets/145972379/39adf475-827d-4063-962f-75f6ae580327)

Tras instalar todas estas cosas se comprueba la conexión al balanceador de carga:

![resultadosol2](https://github.com/luisfoncig/asr-23/assets/145972379/fc1220e6-43e1-4688-9d05-04967e9464cd)

## SOLUCIÓN 3
Se requiere instalar una solución zero trust, es decir cambiar todo a HTTPS, y el puerto al 443, y modificar los parámetros de nginx para se acceda por el puerto 443, y que utilice los certificados generados anteriormente.
Esto se consigue transferiendo los certificados al servidor web.

![loadbalancersol3](https://github.com/luisfoncig/asr-23/assets/145972379/5e57c85b-5966-4873-bd04-77e1a8537f96)

Si accedemos por el puerto 80 no permite tráfico

![primerintentosol3](https://github.com/luisfoncig/asr-23/assets/145972379/2e532960-6fb4-4e68-ae9e-2c8fa10a4e4c)

en cambio por el 443 sí

![finalsol3](https://github.com/luisfoncig/asr-23/assets/145972379/fcea90a7-d32a-45b0-b7c9-df4c73959329)

## SOLUCIÓN 4
Para mejorar la seguridad del sistema se podrían instalar más reglas WAF, como por ejemplo protegiendo de ataques de escaneo de puerto,
los cuales exploran los puertos de los servidores buscando vulnerabilidades por las que acceder al sistema.

Para mejorar disponibilidad se podrían generar copias backup del servidor en regiones distintas a la principal, no sólo asegurando el servicio con
números por si falla alguna máquina, sino protegiendo también de problemas locales, ya que si nuestra red de recuperación está desplegada por varias zonas,
es mucho más improbable que fallen todas a la vez.
