# primera clase

se promociona con 8 (o 7.5 si hay nota de concepto)

20% -> concepto

nota = 0.4 * parcia + 0.4 * 1/n *Xn = 1 T pi* 0,2 * nota teorica

## repaso

- vamo a hablar de internet, protocolos,etc

- internet: es una red de redes, plataforma que brinda servicios

- icp: el proveedor de internet

- los icp tienen icp

- todos los icp tienen conexiones directas o indirectas

- comunicacion unicast: punto a punto -> host a host
- broadcast: es de una a todos -> 
- multicast: de una a un grupo -> host a varios host
- anycast: uno a lo que haiga mas cerca

- modelo tcp:ip 

es un modelo de capas de abstraccion (son 5) 

- 1: aplicacion: http, ftp, smtp, dns, vpn, https
- 2: transporte: tcp, udp, ssl, quic(esta en la mitad de aplicacion y transporte)
- 3: red: ip, icmp
- 4: enlace (link): enlace, wifi, blueetooh, cable modem, adsl, nfc
- 5: fisica: fibra, coaxil, utp, aire

una capa se conecta solo con su abyasente y no conoce a las otras 

## metricas

-- perdida de paquetes(se origina en un host pero nunca llega al destion)

-- se corto la transmision
-- se perdio integridad
-- mala configuracion
-- se enruta mal

TTL: contador de cuantas veces puede viajar un paquete hasta dropearlo

-- latencia: retardo entre el estimulo y la respuesta 

es importante porque pega directo en UX, algunas aplicaciones son sencibles a latencia, ejemplo juegos, videollamadas.

### origen

- tiempo de insercion: tiempo que demora el paque en ser insertado en el enlace

depende de:

- L = largo de paquete -> se mido en bits
- R = velocidad de serializacion

tins = L / R

se produce al serializar pero no al serializar

- tiempo de propagacion
- tiempo de procesamiento
- tiempo de encolado

- tiempo de propagacion: cuanto demora un paquete en propagarse por el enlace de un router al proximo

- depende de velocidad del medio
-- aire = velocodad de la luz(3 m/s)
-- fibra, cobre, coaxi = 2/3 velocidad de la luz


tprop = d/c


si distancia es larga predomina mas el de propagacion y si es mas corta visceversa

-- tiempo de procesamiento: el tiempo que demora en ser procesado por un router particular, entre que lee el header y decide a donde enviar

-- origen de magnitud = ns * us

-- tiempo de encolado: tiempo entre que entra la cola y es transmitido

escala segun el trafico

RTT: tiempo que tarda un paquete enviado a un recepto en volver a su emisor post ser recibido por el reseptor


Throughput: tasa de transferencia  = bits/segundo

averigurar la diferencia entre throughput y ancho de banda

jitter: variacion en el tiempo que tarda un paquete de datos en viajardesde su origen hasta destino, o sea la fluctuacion del retardo

                            l1      | l2     | l3       | l4
ditancia                 |  100m      10 km   4 km       100m
ancho de banda           |  10 mbps   200 mbps  200 mbps  10 mbps
velocidad de propagacion |  1.7*10     


RTT = ti + tp +tp + te

tiempo de procesamiento es despreciable y el tiempo de encolado tambien (asumimos que no hay trafico)

ti + tp 

no siempre so multiplica por 2 (depende de la topologia)

ping: herramienta de software de administracion de redes que prueba la accesibilidad de un host a una red

wireshark: captura paquetes(nos da informacion de cada capa)

tarea: jugar con wireshark 

