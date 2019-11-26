# Cortafuegos personal
## Cadena FORWARD
Paquete que atraviesa el equipo.
- Cadena FORWARD de filter.
Situación típica en un cortafuegos de una red.
Cuando la política es DROP, las reglas debens er dobles.

### FORWARD
iptables -A FORWARD [-p PROTICOLO] [-s IP ORIGEN] [-d IP DESTINO] [-i INTERFAZ ENTRADA] [-j ACCEPT | DROP]

>Ejemplo:
~~~
iptables -I FORWARD -p tcp --dport 8080 -s 0.0.0.0/0 -d 192.168.0.10/32 -i eth0 -o eth1 -j ACCEPT

iptables -I FORWARD -p tcp --sport 8080 -s 192.168.0.10/32 -o th0 -i eth1 -j
~~~

## SNAT con iptables
### source NAR
- Se cambia la dirección origen por la externa del dispositivo de NAT.
- Se hace como último paso antes de enviar el paquete.
- Se difinan en la cadena POSTROUTING en la tabla nat.

### Enmascaramiento IP (IP Masquerade)
- Se denomina así se realiza SNAT pero la IP origen del dispositivo es dinámica.
- Se debe comprobar la IP externa del dispositivo de NAT antes de cambiarla en el paquete. 

## SNAT en iptables
- Suponemos que la red interna es la 192.168.100.0/24, la interfaz externa del dispositivo de NAT ka eth0 y la IP estñatica externa la 80.14.1.2
- Dos pasos:
~~~
echo 1 > /proc/sys/net/ipv4/ip_forward

iptables -t nar -A POSTROUTING -s 192.168.100.0/24 -o eth0 -j SNAT --to 80.14.1.2/32
~~~

## Enmascaramiento IP de iptables
- Suponemos que la red interna es la 192.168.100.0/24 y la interfaz externa del dispositivo de NAT la eth0.
- Dos pasos:
~~~
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -t nat -A POSTROUTING -s 192.168.100.0/24 -o eth0 -j MASQUERADE
~~~

## DNAT con iptables
- Se utiliza para exponer un puerto de un equipo interno al exteiror.
- Se cambia la dirección destino de la externa del dispositivo a la del equipo que queremos exponer.
- Se puede cambiar también el puerto destino.
- Se hace como primer paso antes de tomar la decisión de encaminamiento.
- Se definen en la cadena PREROUTING de la tabla nat.

>Ejemplo:
Suponemos que queremos enviar las peticiones al puerto 22/tcp al equipo 192.168.100.2 de la red interna.
- Primer paso:
~~~
echo 1 > /proc/sys/net/ipv4/ip_forward

iptables -t nat -A PREROUTING -p tcp --dport 22 -i eth0 -j DNAT --to 192.168.100.2
~~~

- Si fuera a un puerto diferente, por ejemplo el 2222/tcp
~~~
iptables -t nat -A PREROUTING -p tcp --dport 22 -i eth0 -j DNAAT --to 192.168.100.2:2222
~~~
