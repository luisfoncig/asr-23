# PRÁCTICA 2 ASR : VM

Luis Foncillas Gutiérrez -- arquitecturas de servicios de red 2023

## SOLUCIÓN 1
### VMs y firewall básico
Se monta una máquina de salto y un servidor web, instalando un firewall de capa 4 básico
![vmssol1](pract2/vmssol1.png)
![firewallsol1](pract2/firwallsol1.png)

### Conexiones
tras esto se comprueban conexiones: primero de local a servidor salto, y después de salto a servidor web
![sshajumpserver](/pract2/sshajumpserver.png)
![sshjumpweb](/pract2/sshjumpweb.png)


## SOLUCIÓN 2
### Balanceador de carga
antes de comenzar con el balanceador de carga, le quitamos la ip pública al servidor web. Después de esto solo podremos acceder al servidor web a través de la ip privada (10.0.0.2)

![servwebnoip](/pract2/servwebnoip.png)

### Instalación nginx
La siguiente tarea es instalar nginx en el servidor web, pero al no tener ip pública el acceso a internet se dificulta. 
Para solucionar esto, hay que instalar una cloud NAT asignada a nuestra subred:

![cloudnat](/pract2/cloudnat.png)

lo que nos permite instalar nginx correctamente.

![installnginx](/pract2/installnginx.png)

### Certificados

Tras esto, se generan 
las claves ssl necesarias para tener la correcta certificación, las cuales insertamos en el backend de nuestro 
balanceador de carga.

![certs](/pract2/certs.png)

### WAF

Para finalizar el apartado, hay que ajustar la firewall para tener condiciones a nivel de aplicación.
Se nos requiere restringir tráfico a países que no sean de confianza de la UE, y proteger de ataques SQL injection y cross syte scripting:

![WAFRules](/pract2/WAFRules.png)

Tras instalar todas estas cosas se comprueba la conexión al balanceador de carga:

![resultadosol2](/pract2/resultadosol2.png)

## SOLUCIÓN 3
Se requiere instalar una solución zero trust, es decir cambiar todo a HTTPS, y el puerto al 443, y modificar los parámetros de nginx para se acceda por el puerto 443, y que utilice los certificados generados anteriormente.
Esto se consigue transferiendo los certificados al servidor web.

![loadbalancersol3](/pract2/loadbalancersol3)

Si accedemos por el puerto 80 no permite tráfico

![primerintentosol3](/pract2/primerintentosol3.png)

en cambio por el 443 sí

![finalsol3](/pract2/finalsol3.png)

## SOLUCIÓN 4
Para mejorar la seguridad del sistema se podrían instalar más reglas WAF, como por ejemplo protegiendo de ataques de escaneo de puerto,
los cuales exploran los puertos de los servidores buscando vulnerabilidades por las que acceder al sistema.

Para mejorar disponibilidad se podrían generar copias backup del servidor en regiones distintas a la principal, no sólo asegurando el servicio con
números por si falla alguna máquina, sino protegiendo también de problemas locales, ya que si nuestra red de recuperación está desplegada por varias zonas,
es mucho más improbable que fallen todas a la vez.
