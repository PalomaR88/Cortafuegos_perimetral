# Implementación de un cortafuego perimetral
Vamos a realizar los primeros pasos para implementar un cortafuegos que protege la red interna.

## Esquema de red
Vamos a utilizar dos máquinas en openstack, que vamos a crear con la receta heat: [escenario2.yaml](https://fp.josedomingo.org/seguridadgs/u03/escenario2.yaml). La receta heat ha deshabilitado el cortafuego que nos ofrece openstack (todos los puertos de todos los protocolos están abiertos). Una máquina (que tiene asignada una IP flotante) hará de cortafuegos, y la otra será una máquina de la red interna 192.168.100.0/24.

## Limpieza de las reglas previas
~~~
iptables -F
iptables -t nat -F
iptables -Z
iptables -t nat -Z
~~~

## Permitir ssh al cortafuegos 
Cómo estamos conectado a la máquina por ssh, vamos a permitir la conexión ssh desde la red 172.22.0.0/16, antes de cambiar las políticas por defecto a DROP, para no perder la conexión:
~~~
iptables -A INPUT -s 172.22.0.0/16 -p tcp -m tcp --dport 22 -j ACCEPT
iptables -A OUTPUT -d 172.22.0.0/16 -p tcp -m tcp --sport 22 -j ACCEPT
~~~

## Política por defecto
~~~
iptables -P INPUT DROP
iptables -P OUTPUT DROP
iptables -P FORWARD DROP
~~~

Comprobamos que el equipo no puede acceder a ningún servicio ni de Internet ni de la red local, ya que la política lo impide.

## Activar el bit de forward
El primer paso común es habilitar el bit de forward mediante la instrucción:
~~~
echo 1 > /proc/sys/net/ipv4/ip_forward
~~~

Para habilitar el forward de forma permanente, habilitamos la línea net.ipv4.ip_forward=1 del fichero /etc/sysctl.conf y ejecutando posteriormente sysctl -p /etc/sysctl.conf.


## SNAT
Hacemos SNAT para que los equipos de la LAN puedan acceder al exterior:
~~~
iptables -t nat -A POSTROUTING -s 192.168.100.0/24 -o eth0 -j MASQUERADE
~~~

## Permitir ssh desde el cortafuego a la LAN
~~~
iptables -A OUTPUT -p tcp -o eth1 -d 192.168.100.0/24 --dport 22 -j ACCEPT
iptables -A INPUT -p tcp -i eth1 -s 192.168.100.0/24 --sport 22 -j ACCEPT
~~~

## Permitir tráfico para la interfaz loopback
~~~
iptables -A INPUT -i lo -p icmp -j ACCEPT
iptables -A OUTPUT -o lo -p icmp -j ACCEPT
~~~

## Peticiones y respuestas protocolo ICMP
~~~
iptables -A OUTPUT -o eth0 -p icmp -j ACCEPT
iptables -A INPUT -i eth0 -p icmp -j ACCEPT
~~~

## Ping a la LAN:
~~~
iptables -A OUTPUT -o eth1 -p icmp -j ACCEPT
iptables -A INPUT -i eth1 -p icmp -j ACCEPT
~~~

## Reglas forward
Aunque ya hemos configurado SNAT, como hemos puesto la política por defecto FORWARD a DROP, los equipos de la LAN están incomunicadas, ya que no permitimos que ningún paquete pase por el cortafuego. Por lo tanto ahora tenemos que ir configurando los pares de reglas (forward en ambas direcciones) para ir permitiendo distintos protocolos, puertos,… a la LAN.

## Permitir hacer ping desde la LAN
Actualmente estamos permitiendo que desde la LAN se pueda hacer ping al cortafuego, pero para que la LAN haga ping al exterior los paquetes ICMP tiene que estar permitidos que pasen por el cortafuego:
~~~
iptables -A FORWARD -o eth0 -i eth1 -s 192.168.100.0/24 -p icmp -j ACCEPT
iptables -A FORWARD -i eth0 -o eth1 -d 192.168.100.0/24 -p icmp -j ACCEPT
~~~

## Consultas y respuestas DNS desde la LAN
~~~
iptables -A FORWARD -i eth1 -o eth0 -s 192.168.100.0/24 -p udp --dport 53 -j ACCEPT
iptables -A FORWARD -o eth1 -i eth0 -d 192.168.100.0/24 -p udp --sport 53 -j ACCEPT
~~~

## Permitir la navegación web desde la LAN
~~~
iptables -A FORWARD -i eth1 -o eth0 -s 192.168.100.0/24 -p tcp --dport 80 -j ACCEPT
iptables -A FORWARD -o eth1 -i eth0 -d 192.168.100.0/24 -p tcp --sport 80 -j ACCEPT
iptables -A FORWARD -i eth1 -o eth0 -s 192.168.100.0/24 -p tcp --dport 443 -j ACCEPT
iptables -A FORWARD -o eth1 -i eth0 -d 192.168.100.0/24 -p tcp --sport 443 -j ACCEPT
~~~
Podemos comprobar que está haciendo resolución DNS y navegación web desde la máquina de la LAN instalando un servidor web apache2.


## Permitimos el acceso a nuestro servidor web de la LAN desde el exterior
En un primer momento tenemos que permitir que la consulta pase por el cortafuegos:
~~~
iptables -A FORWARD -i eth0 -o eth1 -d 192.168.100.0/24 -p tcp --dport 80 -j ACCEPT # en este caso lo correcto sería indicar la IP del servidor 
iptables -A FORWARD -i eth1 -o eth0 -s 192.168.100.0/24 -p tcp --sport 80 -j ACCEPT
~~~
Y necesitamos configurar una regla DNAT:
~~~
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j DNAT --to 192.168.200.10
~~~

## Configuración en un solo paso

Editamos un fichero y añadimos todas las reglas anteriores:
~~~
# Limpiamos las tablas
iptables -F
iptables -t nat -F
iptables -Z
iptables -t nat -Z
# Establecemos la política
iptables -P INPUT DROP
iptables -P OUTPUT DROP
iptables -P FORWARD DROP

# SNAT
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -t nat -A POSTROUTING -s 192.168.100.0/24 -o eth0 -j MASQUERADE

# DNAT
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j DNAT --to 192.168.200.10

# Reglas INPUT/OUTPUT

iptables -A OUTPUT -p tcp -o eth1 -d 192.168.100.0/24 --dport 22 -j ACCEPT
iptables -A INPUT -p tcp -i eth1 -s 192.168.100.0/24 --sport 22 -j ACCEPT

iptables -A INPUT -i lo -p icmp -j ACCEPT
iptables -A OUTPUT -o lo -p icmp -j ACCEPT

iptables -A OUTPUT -o eth0 -p icmp -j ACCEPT
iptables -A INPUT -i eth0 -p icmp -j ACCEPT

iptables -A OUTPUT -o eth1 -p icmp -j ACCEPT
iptables -A INPUT -i eth1 -p icmp -j ACCEPT

# Reglas FORWARD

iptables -A FORWARD -o eth0 -i eth1 -s 192.168.100.0/24 -p icmp -j ACCEPT
iptables -A FORWARD -i eth0 -o eth1 -d 192.168.100.0/24 -p icmp -j ACCEPT

iptables -A FORWARD -i eth1 -o eth0 -s 192.168.100.0/24 -p udp --dport 53 -j ACCEPT
iptables -A FORWARD -o eth1 -i eth0 -d 192.168.100.0/24 -p udp --sport 53 -j ACCEPT

iptables -A FORWARD -i eth1 -o eth0 -s 192.168.100.0/24 -p tcp --dport 80 -j ACCEPT
iptables -A FORWARD -o eth1 -i eth0 -d 192.168.100.0/24 -p tcp --sport 80 -j ACCEPT
iptables -A FORWARD -i eth1 -o eth0 -s 192.168.100.0/24 -p tcp --dport 443 -j ACCEPT
iptables -A FORWARD -o eth1 -i eth0 -d 192.168.100.0/24 -p tcp --sport 443 -j ACCEPT

iptables -A FORWARD -i eth0 -o eth1 -d 192.168.100.0/24 -p tcp --dport 80 -j ACCEPT
iptables -A FORWARD -i eth1 -o eth0 -s 192.168.100.0/24 -p tcp --sport 80 -j ACCEPT
~~~

> Ejercicios

**1. Permite realizar conexiones ssh desde los equipos de la LAN**

~~~
debian@router-fw:~$ sudo iptables -t nat -I POSTROUTING -s 192.168.100.0/24 -o eth0 -p tcp --dport 22 -j MASQUERADE
debian@router-fw:~$ 
debian@router-fw:~$ sudo iptables -A FORWARD -i eth1 -o eth0 -p tcp --dport 22 -j ACCEPT
debian@router-fw:~$ sudo iptables -A FORWARD -i eth0 -o eth1 -p tcp --sport 22 -j ACCEPT
~~~

> Prueba:
~~~
debian@lan:~$ ssh paloma@172.23.0.22
The authenticity of host '172.23.0.22 (172.23.0.22)' can't be established.
ECDSA key fingerprint is SHA256:TMp5TPIjjKNSGQrf1jUMXXOo36syVo4Uf54L4QKdAIM.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '172.23.0.22' (ECDSA) to the list of known hosts.
paloma@172.23.0.22's password: 
Linux coatlicue 4.19.0-6-amd64 #1 SMP Debian 4.19.67-2+deb10u2 (2019-11-11) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
You have mail.
Last login: Tue Nov 19 12:24:28 2019 from 172.22.201.14
paloma@coatlicue:~$ 
~~~


**2.  Instala un servidor de correos en la máquina de la LAN. Permite el acceso desde el exterior y desde el cortafuego al servidor de correos. Para probarlo puedes ejecutar un telnet al puerto 25 tcp.**

~~~
debian@lan:~$ sudo apt install postfix

 ┌──────────────────────┤ Postfix Configuration ├──────────────────────┐
 │ Please select the mail server configuration type that best meets    │ 
 │ your needs.                                                         │ 
 │                                                                     │ 
 │  No configuration:                                                  │ 
 │   Should be chosen to leave the current configuration unchanged.    │ 
 │  Internet site:                                                     │ 
 │   Mail is sent and received directly using SMTP.                    │ 
 │  Internet with smarthost:                                           │ 
 │   Mail is received directly using SMTP or by running a utility      │ 
 │ such                                                                │ 
 │   as fetchmail. Outgoing mail is sent using a smarthost.            │ 
 │  Satellite system:                                                  │ 
 │   All mail is sent to another machine, called a 'smarthost', for    │ 
 │ delivery.                                                           │ 
 │  Local only:                                                        │ 
 │   The only delivered mail is the mail for local users. There is no  │ 
 │ network.                                                            │ 
 │                                                                     │ 
 │ General type of mail configuration:                                 │ 
 │                                                                     │ 
 │                      No configuration                               │ 
 │                      Internet Site                                  │ 
 │                      Internet with smarthost                        │ 
 │                      Satellite system                               │ 
 │                      Local only                                     │ 
 │                                                                     │ 
 │                                                                     │ 
 │                  <Ok>                      <Cancel>                 │ 
 │                                                                     │ 
 └─────────────────────────────────────────────────────────────────────┘ 

┌──────────────────────┤ Postfix Configuration ├──────────────────────┐
 │ The "mail name" is the domain name used to "qualify" _ALL_ mail     │ 
 │ addresses without a domain name. This includes mail to and from     │ 
 │ <root>: please do not make your machine send out mail from          │ 
 │ root@example.org unless root@example.org has told you to.           │ 
 │                                                                     │ 
 │ This name will also be used by other programs. It should be the     │ 
 │ single, fully qualified domain name (FQDN).                         │ 
 │                                                                     │ 
 │ Thus, if a mail address on the local host is foo@example.org, the   │ 
 │ correct value for this option would be example.org.                 │ 
 │                                                                     │ 
 │ System mail name:                                                   │ 
 │                                                                     │ 
 │ lan.novalocal______________________________________________________ │ 
 │                                                                     │ 
 │                  <Ok>                      <Cancel>                 │ 
 │                                                                     │ 
 └─────────────────────────────────────────────────────────────────────┘ 
~~~

En el router:
~~~
debian@router-fw:~$ sudo iptables -A FORWARD -i eth0 -o eth1 -d 192.168.100.10/24 -p tcp --dport 25 -j ACCEPT
debian@router-fw:~$ sudo iptables -A FORWARD -i eth1 -o eth0 -p tcp --sport 25 -j ACCEPT
debian@router-fw:~$ sudo iptables -A OUTPUT -o eth1 -d 192.168.100.10/24 -p tcp --dport 25 -j ACCEPT
debian@router-fw:~$ sudo iptables -A INPUT -i eth1 -p tcp --sport 25 -j ACCEPT
debian@router-fw:~$ sudo iptables -t nat -I PREROUTING -i eth0 -p tcp --dport 25 -j DNAT --to 192.168.100.10
~~~

> Prueba, desde el router:
~~~
debian@router-fw:~$ telnet 192.168.100.10 25
Trying 192.168.100.10...
Connected to 192.168.100.10.
Escape character is '^]'.
220 lan.novalocal ESMTP Postfix (Debian/GNU)
~~~

> Prueba, desde otro equipo:
~~~
paloma@coatlicue:~$ telnet 172.22.201.78 25
Trying 172.22.201.78...
Connected to 172.22.201.78.
Escape character is '^]'.
220 lan.novalocal ESMTP Postfix (Debian/GNU)
~~~


**3. Permite poder hacer conexiones ssh desde exterior a la LAN**

~~~
debian@router-fw:~$ sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 22 -j DNAT --to 192.168.100.10
debian@router-fw:~$ sudo iptables -A FORWARD -i eth0 -o eth1 -p tcp --dport 22 -j ACCEPT
debian@router-fw:~$ sudo iptables -A FORWARD -i eth1 -o eth0 -p tcp --sport 22 -j ACCEPT
~~~

> Prueba:
~~~
paloma@coatlicue:~$ ssh debian@172.22.201.78
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ECDSA key sent by the remote host is
SHA256:QIYecH27rmU5Urfy4zoUoOwBMF+l3++NVAmaHRGvg3Q.
Please contact your system administrator.
Add correct host key in /home/paloma/.ssh/known_hosts to get rid of this message.
Offending ECDSA key in /home/paloma/.ssh/known_hosts:46
  remove with:
  ssh-keygen -f "/home/paloma/.ssh/known_hosts" -R "172.22.201.78"
ECDSA host key for 172.22.201.78 has changed and you have requested strict checking.
Host key verification failed.
paloma@coatlicue:~$ ssh-keygen -f "/home/paloma/.ssh/known_hosts" -R "172.22.201.78"
# Host 172.22.201.78 found: line 46
/home/paloma/.ssh/known_hosts updated.
Original contents retained as /home/paloma/.ssh/known_hosts.old
paloma@coatlicue:~$ ssh debian@172.22.201.78
The authenticity of host '172.22.201.78 (172.22.201.78)' can't be established.
ECDSA key fingerprint is SHA256:QIYecH27rmU5Urfy4zoUoOwBMF+l3++NVAmaHRGvg3Q.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '172.22.201.78' (ECDSA) to the list of known hosts.
Linux lan 4.19.0-6-cloud-amd64 #1 SMP Debian 4.19.67-2+deb10u1 (2019-09-20) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu Dec  5 15:50:21 2019 from 192.168.100.2
debian@lan:~$ 
~~~



**4. Modifica la regla anterior, para que al acceder desde el exterior por ssh tengamos que conectar al puerto 2222, aunque el servidor ssh este configurado para acceder por el puerto 22.**

~~~
debian@router-fw:~$ sudo iptables -t nat -I PREROUTING -i eth0 -p tcp --dport 2222 -j DNAT --to 192.168.100.10:22
debian@router-fw:~$ 
debian@router-fw:~$ sudo iptables -I FORWARD -i eth0 -o eth1 -p tcp --dport 22 -j ACCEPT
debian@router-fw:~$ sudo iptables -I FORWARD -i eth1 -o eth0 -p tcp --sport 22 -j ACCEPT
~~~

> Para probarlo, se hace una conexión ssh al puerto 2222:
~~~
paloma@coatlicue:~$ ssh -p 2222 debian@172.22.201.78
Linux lan 4.19.0-6-cloud-amd64 #1 SMP Debian 4.19.67-2+deb10u1 (2019-09-20) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu Dec  5 17:25:16 2019 from 172.23.0.22
debian@lan:~$ 
~~~


**5. Permite hacer consultas DNS sólo al servidor 192.168.202.2. Comprueba que no puedes hacer un dig @1.1.1.1.**

~~~
debian@router-fw:~$ sudo iptables -t nat -A POSTROUTING -s 192.168.100.0/24 -o eth0 -p udp --dport 53 -j MASQUERADE
debian@router-fw:~$ sudo iptables -I FORWARD -i eth1 -o eth0 -d 192.168.202.2 -p udp --dport 53 -j ACCEPT
debian@router-fw:~$ sudo iptables -R FORWARD 2 -i eth0 -o eth1 -p udp --sport 53 -j ACCEPT
~~~

>Prueba:
~~~
dig @1.1.1.1

   ; <<>> DiG 9.11.5-P4-5.1-Debian <<>> @1.1.1.1
   ; (1 server found)
   ;; global options: +cmd
   ;; connection timed out; no servers could be reached
~~~

**6. ¿Tendría resolución de nombres y navegación web el cortafuego? ¿Sería necesario? ¿Tendrían que estar esas de reglas de forma constante en el cortafuego?**

No tendría y no sería necesario pero es aconsejable tenerlos para controlar la seguridad. Y estas reglas deberían estar constantemente. 
