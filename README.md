# Cortafuegos Perimetral
## 1. [Cortafuegos Personal](https://github.com/PalomaR88/Cortafuegos_perimetral/blob/master/Cortafuegos_perimetral.md#cortafuegos-personal) 
#### 1.1. [Cadena FORWARD](https://github.com/PalomaR88/Cortafuegos_perimetral/blob/master/Cortafuegos_perimetral.md#cadena-forward) 
##### 1.1.1. [FORWARD](https://github.com/PalomaR88/Cortafuegos_perimetral/blob/master/Cortafuegos_perimetral.md#forward) 
#### 1.2. [SNAT con iptables](https://github.com/PalomaR88/Cortafuegos_perimetral/blob/master/Cortafuegos_perimetral.md#snat-con-iptables) 
##### 1.2.1. [Source NAT](https://github.com/PalomaR88/Cortafuegos_perimetral/blob/master/Cortafuegos_perimetral.md#source-Nat) 
##### 1.2.2. [Enmascaramiento IP](https://github.com/PalomaR88/Cortafuegos_perimetral/blob/master/Cortafuegos_perimetral.md#enmascaramiento-ip-ip-masquerade) 
#### 1.3. [SNAT en iptables](https://github.com/PalomaR88/Cortafuegos_perimetral/blob/master/Cortafuegos_perimetral.md#SNAT-en-iptables) 
#### 1.4. [Enmascaramiento IP de iptables](https://github.com/PalomaR88/Cortafuegos_perimetral/blob/master/Cortafuegos_perimetral.md#enmascaramiento-ip-de-iptables) 
#### 1.5. [SNAT con iptables](https://github.com/PalomaR88/Cortafuegos_perimetral/blob/master/Cortafuegos_perimetral.md#dnat-con-iptables) 
---------------------------------------------

## 2. [Implementación de un cortafuegos Perimetral](https://github.com/PalomaR88/Cortafuegos_perimetral/blob/master/Practica.md#implementaci%C3%B3n-de-un-cortafuego-perimetral) 
#### 2.1. [Esquema de Red](https://github.com/PalomaR88/Cortafuegos_perimetral/blob/master/Practica.md#esquema-de-red) 
#### 2.2. [Limpieza de las reglas previas](https://github.com/PalomaR88/Cortafuegos_perimetral/blob/master/Practica.md#limpieza-de-las-reglas-previas) 
#### 2.3. [Permitir ssh al cortafuegos](https://github.com/PalomaR88/Cortafuegos_perimetral/blob/master/Practica.md#Permitir-ssh-al-cortafuegos) 
#### 2.4. [Política por defecto](https://github.com/PalomaR88/Cortafuegos_perimetral/blob/master/Practica.md#pol%C3%ADtica-por-defecto) 
#### 2.5. [Activar el bit de forward](https://github.com/PalomaR88/Cortafuegos_perimetral/blob/master/Practica.md#activar-el-bit-de-forward) 
---------------------------------------------
## 3. [SNAT](https://github.com/PalomaR88/Cortafuegos_perimetral/blob/master/Practica.md#snat) 
#### 3.1. [Permitir ssh desde el cortafuegos a la LAN](https://github.com/PalomaR88/Cortafuegos_perimetral/blob/master/Practica.md#PERMITIR-SSH-DESDE-EL-CORTAFUEGO-A-LA-LAN) 
#### 3.2. [Permitir tráfico para la interfaz loopback](https://github.com/PalomaR88/Cortafuegos_perimetral/blob/master/Practica.md#permitir-tr%C3%A1fico-para-la-interfaz-loopback) 
#### 3.3. [Peticiones y respuestas protocolo ICMP](https://github.com/PalomaR88/Cortafuegos_perimetral/blob/master/Practica.md#peticiones-y-respuestas-protocolo-icmp) 
#### 3.4. [Ping a la LAN](https://github.com/PalomaR88/Cortafuegos_perimetral/blob/master/Practica.md#ping-a-la-lan) 
#### 3.5. [Reglas FORWARD](https://github.com/PalomaR88/Cortafuegos_perimetral/blob/master/Practica.md#reglas-FORWARD) 
#### 3.6. [Ping desde la LAN](https://github.com/PalomaR88/Cortafuegos_perimetral/blob/master/Practica.md#permitir-hacer-ping-desde-la-lan) 
#### 3.7. [Consultas y respuestas DNS desde la LAN](https://github.com/PalomaR88/Cortafuegos_perimetral/blob/master/Practica.md#consultas-y-respuestas-dns-desde-la-lan) 
#### 3.7. [Permitir la navegación web desde la LAN](https://github.com/PalomaR88/Cortafuegos_perimetral/blob/master/Practica.md#permitimos-la-navegaci%C3%B3n-web-desde-la-lan) 
#### 3.8. [Permitir acceso a nuestro navegador web de la LAN desde el exterior](https://github.com/PalomaR88/Cortafuegos_perimetral/blob/master/Practica.md#permitimos-el-acceso-a-nuestro-servidor-web-de-la-lan-desde-el-exterior) 

---------------------------------------------
## 4. [Configuración en un solo paso y practica](https://github.com/PalomaR88/Cortafuegos_perimetral/blob/master/Practica.md#configuraci%C3%B3n-en-un-solo-paso) 





