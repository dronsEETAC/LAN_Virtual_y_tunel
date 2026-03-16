# LAN Virtual y Túnel    
    
En este repositorio se introducen dos conceptos que pueden simplificar significativamente la arquitectura de comunicaciones de diferentes escenarios del ecosistema. Por una parte se explica cómo crear una LAN virtual en la que estén incluidos dispositivos conectados a diferentes LAN reales, de manera que esos dispositivos puedan conectarse directamente unos con otros sin intermediación. Por otro lado, se introduce el concepto de túnel, que permite enviar directamente mensajes MAVLink a dispositivos remotos conectados a internet, de nuevo sin necesidad de intermediación.    
 
## 1. Red LAN virtual    
 
Si dos dispositivos están conectados a la misma LAN real entonces cualquiera de ellos puede conectarse con el otro usando sus IPs dentro de la LAN. Por ejemplo, pueden abrir una conexión directa TCP o UDP para intercambiar información. Sin embargo, si los dispositivos están conectados a LANs diferentes y ninguno de ellos tiene una IP pública, entonces no pueden comunicarse directamente porque la IP local de cualquiera de ellos es inalcanzable desde el otro. La comunicación entre ambos puede hacerse a través de un intermediario. Ese es el caso, por ejemplo, de la Raspberry Pi (RPi) de abordo y uno de los ordenadores de sobremesa de un aula informática (que llamaremos estación remota). La RPi estará conectada a la WiFi del DroneLab y la estación remota a la LAN del aula. Si queremos controlar el dron desde la estación remota entonces la comunicación entre ambos dispositivos puede hacerse mediante MQTT, con la intermediación de un bróker disponible en una IP pública conocida por ambos. También pueden comunicarse  mediante sockets, en cuyo caso se necesita un servidor proxy, también con IP publica, al que se conectarán ambos dispositivos. El proxy se encargará de encaminar los mensajes de uno hacia el otro.   
 
Existen herramientas que permiten crear una LAN virtual que incluya a ambos dispositivos, cada uno de los cuales tendrá asignada una IP accesible por cualquier otro dispositivo de la LAN virtual. Una de esas herramientas es ZeroTier. Veamos cómo usarla.    
 
Para empezar hay que acceder a la web de ZeroTier:    
 
https://www.zerotier.com/    
 
Podremos logearnos con nuestras credenciales de varias plataformas, incluyendo Google o GitHub. Una vez logeados podemos elegir un nombre para la organización y luego crear una red virtual, a la que también daremos un nombre. Una vez creada la red nos proporcionará un network ID que necesitaremos para conectar dispositivos a esa red (la versión gratuita nos permitirá crear una sola red, a la que podremos conectar varios dispositivos).    
 
El siguiente paso es descargar en Windows la aplicación ZeroTier (suponiendo que el portátil que queremos conectar a la red tiene Windows). Puede descargarse desde aquí:    
 
https://www.zerotier.com/download/    
  
Una vez instalada la aplicación podemos operar desde la barra de herramientas, tal y como muestra la figura.

<img width="500" height="250" alt="Image" src="https://github.com/user-attachments/assets/b610b4d5-4830-461b-a181-5fe2b6cefabf" />    

   
 
La figura muestra que el portátil ya se ha unido a la red denominada my-first-network. Para unirse a la red hay que pulsar el item Join New Network e introducir el network ID.    
 
El proceso para el caso de la RPi también es sencillo. En primer lugar hay que instalar en la RPi ZeroTier, mediante el siguiente comando:    
```
curl -s https://install.zerotier.com | sudo bash
```
Después hay que unirse a la LAN mediante el comando siguiente:
```
sudo zerotier-cli join NETWORK_ID
```
Ahora en la web podemos verificar que ambos dispositivos pertenecen ya a la LAN, tal y como muestran las imágenes.

<img width="429" height="169" alt="Image" src="https://github.com/user-attachments/assets/e64a55d5-724b-43bf-b95a-182e1b2f1321" />   

	 
<img width="424" height="171" alt="Image" src="https://github.com/user-attachments/assets/b011e419-4647-45f7-8caf-811b0f5c55b7" />    
 

La primera imagen muestra que hay tres dispositivos conectados en la red my-first-network (además del portátil y la RPi, se conectó otro ordenador). Al clicar sobre el nombre de la red se abre una página como la que muestra la segunda imagen, en la que podemos ver los tres dispositivos conectados con las IP asignadas (columna ZT IP). En la imagen se ve que los tres dispositivos han sido autorizados (columna status). Inicialmente no estarán autorizados. Será necesario clicar en la columna status para autorizar a cada uno de ellos. A partir de ese momento, cualquiera de los dispositivos puede ahora acceder a cualquiera de los otros usando la IP asignada. Para verificarlo, basta hacer ping de cualquiera de ellos a cualquiera de los otros. Podemos trabajar ahora con los dispositivos exactamente igual que haríamos si estuviesen en la misma LAN.    

## 2. Comunicación vía túnel    
 
En cualquiera de los escenarios con los que trabajamos en el ecosistema, el dron está conectado físicamente a la estación de tierra que tiene conectada la radio de telemetría. También puede estar conectado por cable a la RPi de abordo, a través de un puerto serie. En principio, solo estos dispositivos conectados al dron pueden enviar los comandos MAVLink (por ejemplo, usando la librería DronLink). Si queremos controlar el dron desde una estación remota conectada a internet entonces tenemos que usar algún mecanismo de comunicación entre esa estación remota y el dispositivo que está conectado directamente al dron. La figura muestra un ejemplo de esta situación.    
<img width="1500" height="700" alt="Image" src="https://github.com/user-attachments/assets/9fe4b8a5-d7c2-4e57-a292-44d0dc17e097" />

El dron está conectado a la estación de tierra mediante el enlace de telemetría. Por tanto, podemos controlar el dron enviando por ese enlace mensajes MAVLink usando la librería DronLink, por ejemplo, para hacer que el dron despegue. Tenemos también una estación remota con la misma interfaz gráfica que la estación de tierra. Pero si pulsamos el botón de despegue en la estación remota, no podemos ahora usar la librería DronLink, porque esa estación no está conectada con el dron. La aplicación tiene que enviar un mensaje a la estación de tierra para que sea ella la que ordene la operación a través del enlace de telemetría. Esa comunicación entre la estación remota y la estación de tierra puede hacerse vía MQTT o vía sockets. Su implementación dependerá de si ambos dispositivos están en la misma LAN física o no, o si se ha creado una LAN virtual, tal y como se ha explicado en el apartado anterior.    
 
Sin embargo, es posible crear un túnel que conecte la estación remota con la estación de tierra, de manera que la remota pueda enviar directamente mensajes MAVLink (usando DronLink), que serán redirigidos al dron por la estación de tierra. El túnel puede ser simplemente un socket que conecta ambas estaciones. La estación remota se conecta al dron usando la IP:puerto que ofrece la estación de tierra, que a su vez se habrá conectado al puerto COM en el que está conectada la radio de telemetría. Los mensajes MAVLink que envíe la estación remota usando DronLink viajarán por el túnel de manera que, cuando los reciba, la estación de tierra los redirigirá al puerto COM para que lleguen al dron vía enlace de telemetría. La figura muestra esta situación.    
<img width="2000" height="1000" alt="Image" src="https://github.com/user-attachments/assets/e84f7038-3140-4a82-a664-3acfe7e50ebb" />

De nuevo, la conexión entre ambas estaciones puede ser directa si ambas están en la misma LAN real o virtual, o necesitar de un intermediario si no es el caso.
Una ventaja de este planteamiento es que podemos eliminar la intermediación si los dispositivos están en la misma LAN real o virtual. Pero la ventaja más importante es que el código de ambas estaciones puede ser el mismo, con la única diferencia del string de conexión que se usa en cada caso. Veamos a continuación cómo implementar este esquema.    
 
### 2.1 Un túnel con MAVProxy     
 
La forma más sencilla de poner en práctica el concepto de túnel es usar MAVProxy. Cuando hacemos, por ejemplo:
```
mavproxy --master=com3 --out=udp:127.0.0.1:14550
```
estamos poniendo en marcha un túnel que conecta el puerto com3 (al que está conectado el enlace de telemetría) con el puerto udp 14500. Los mensajes MAVLink fluirán de un puerto al otro. El puerto udp está configurado para conexiones locales (programas en la misma máquina en la que se pone en marcha MAVProxy). Pero supongamos ahora que tenemos tres portátiles A, B y C, todos conectados en la misma LAN. En el portátil A tenemos ejecutándose el simulador SITL, en el portátil B ponemos en marcha un túnel con MAVProxy y en el portátil C vamos a ejecutar nuestro programa que usa DronLink. Pondremos en marcha MAVProxy en el portátil B con este comando:    
```
mavproxy --master=tcp:192.168.1.81:5763 --out=udpin:0.0.0.0:14550
```
suponiendo que la IP del portátil A en la LAN es 192.168.1.81. Vemos que el puerto udp está configurado para escuchar en el puerto 14500 mensajes que lleguen a través de cualquier interfaz. Por otra parte, el programa que se ejecuta en el portátil C tiene que conectarse al dron (en realidad se conecta al túnel) con la siguiente llamada a DronLink:    
```
dron.connect (“udpout:192.168.1.77:14550”, 115200)
```
siendo 192.168.1.77 la IP de B en la LAN. Naturalmente, si A, B y C no pertenecen a la misma LAN real podemos crear una LAN virtual, tal y como se ha explicado en la sección 1, y usar las IP asignadas en esa LAN virtual.   
 
Es importante tener presente que en cuanto MAVProxy reciba el primer mensaje por el socket udp, registrará la IP del emisor y la usará cuando tenga que re-enviarle mensajes MAVLink procedentes del SITL. Por eso, el método connect de DronLink hace lo siguiente:    
```
self.vehicle = mavutil.mavlink_connection((“udpin: 192.168.1.71:14550”, baud)

self.vehicle.mav.heartbeat_send(
    mavutil.mavlink.MAV_TYPE_GCS,
    mavutil.mavlink.MAV_AUTOPILOT_INVALID,
    0,
    0,
    0
)

self.vehicle.wait_heartbeat()
```

Inmediatamente después de conectarse envía un mensaje MAVLink arbitrario para que MAVProxy registre la IP del emisor. Sin ese mensaje, MAVProxy no podría re-enviar al portátil C los mensajes MAVLink procedentes del SITL y el programa que ha intentado conectarse fallaría porque nunca recibiría el heartbeat que espera después de la conexión y el envío del mensaje arbitrario.    

### 2.2 Un túnel con MAVProxy en la RPi    

En el caso particular de que queramos poner en marcha MAVProxy en la RPi conviene tener en cuenta varias cosas. Por un lado, hay que instalar el paquete *mavproxy* en el entorno virtual. También será necesario instalar el paquete *future*. El túnel se pone en marcha con este comando:     
```
mavproxy.py --master=tcp:192.168.1.81:5763 --out=udpin:0.0.0.0:14550
```
Si en algún momento debe abortarse mavproxy, puede hacerse así:    
```
killall mavproxy.py
```
Puesto que el programa que va a conectarse al túnel debe conocer la IP de la RPi en la LAN, el siguiente código puede ser útil para descubrir cuál es esa IP:
```
import subprocess
arp = subprocess.check_output("arp -a").decode("cp1252")

for line in arp.split("\n"):

    if "b8-27-eb" in line.lower() or 
       "dc-a6-32" in line.lower() or 
       "e4-5f-01" in line.lower():
       
       print("Raspberry Pi encontrada:", line)
```
El código identifica que la IP es de una RPi a partir de los primeros dígitos de la MAC, que suele tener una de las tres combinaciones que se consultan.   
 
### 2.3 Nuestro propio túnel   
 
Para acabar, es fácil preparar nuestro propio túnel, implementando la misma operación que hace MAVProxy. El código de ese túnel, que se ejecutaría en B, es el siguiente:
```
import socket
import threading

# IP y Puerto del SITL (ejecutándose en el portátil A)
TCP_HOST = "192.168.1.81"
TCP_PORT = 5763

# Puerto UDP donde escuchará el túnel (que se ejecuta en el portátil B)
UDP_PORT = 14550

print("Conectando al SITL...")
tcp_sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
tcp_sock.connect((TCP_HOST, TCP_PORT))
print("Conectado al SITL")

# preparo el socket udp para escuchar mensajes que vienen del portátil C
udp_sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
udp_sock.bind(("0.0.0.0", UDP_PORT))
print("Escuchando UDP en puerto", UDP_PORT)

client_addr = None
def udp_to_tcp():
    global client_addr
    while True:
	  # espero mensaje por el puerto UDP
        data, addr = udp_sock.recvfrom(4096)
	  # me guardo la IP del que envía el mensaje (el portátil C)
        client_addr = addr
	  # re-envío en mensaje MAVLink al SITL
        tcp_sock.sendall(data)


def tcp_to_udp():
    global client_addr
    while True:
	  # Espero mensaje del SITL
        data = tcp_sock.recv(4096)

        if not data:
            break

        if client_addr:
	      # si ya tengo la IP de C le re-envio el mensaje 
            udp_sock.sendto(data, client_addr)


t1 = threading.Thread(target=udp_to_tcp)
t2 = threading.Thread(target=tcp_to_udp)

t1.start()
t2.start()

t1.join()
t2.join()
```


