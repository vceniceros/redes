# apuntes 13/03

foco en los porque, 

4 de octubre de 2021 a facebook se le cayo gran parte del servicio, fallaban los DNS, concretamente el ttl de los dns era de 172800, 48 hrs, lo que paso es que con un ttl tan alto hizo que muchas entradas erroneas quedasen en cache por mucho mas tiempo haciendo que se saturen los mismo

## DNS(Es clave)

toma nombres (url) y los convierte en direcciones IP, un servidor es una direccion (con numeritos ips)

modelo osi: no existe, o no es real, asi que van 5 a joderse

- 1: aplicacion: http, ftp, smtp, dns, vpn, https
- 2: transporte: tcp, udp, ssl, quic(esta en la mitad de aplicacion y transporte)
- 3: red: **ip**, icmp
- 4: enlace (link): enlace, wifi, blueetooh, cable modem, adsl, nfc
- 5: fisica: fibra, coaxil, utp, aire

´´´
 Tins    Tproc
O->     =o->   esto es un router
 [l]    [Tbuffer]

 R = [bit/s]
´´´
R: taza de transmision por segundo

**aclaremos siempre unidades, no existen 300 megas, son 300 megabits por segundo**

tiempo de insercion depende de longitud, 

un router viene a ser esos carteles en la autopista/ruta que dicen para donde salir segun donde quieras ir

tenemos tambien el tiempo del paquete en el buffer y el tiempo de procesamiento del router

en redes nos interesea mucho el tiempo de ida y de vuelta de un paquete

```
tiempo de insersion = L[bit]/R[bit/s] (es bajo por las tasas de subida altisimas)

tiempo de transmision = D[m]/2/3C[m/s]  siendo c la velocidad de la luz (este va en funcion de la distancia de los enlaces reales, curiosamente a mas larga la distancia mas indistinta es la distancia perse en la comunicacion de los distintos enlaces) por dar un ejemplo de palermo a caballito quizas alla una distancia extra al pasar por chacarita, pero al mandarse de buenos aires a nueva york el camino seria casi el mismo

tiempo de buffer = K l/ ^ (este va en funcion del transito)

tiempo de procesamiento depende del router(esta en nano segundos ergo es desestimable)
```

## aplicaciones(el motivo por el que existe internet)

### arquitecturas

- cliente-servidor[datacenters]
- per-to-per

comunicacion entre procesos [mensajes, socket()->abre comunicaciones con archivos, otra computadoras, etc, capa de transporte]


### CARATERISTICAS QUE OFRECE EL TRANSPORTE

- transmision confiable (llega en orden, que sea integro y que mas importante, llegue)
- caudal(throughput)
- sincronizacion(si tengo dos procesos en dos equipo distintos tiene que sincronizarse)
- seguridad(que no se pueda modificar, que no se pueda leer de afuera, asegurar que el cliente o servidor sea ese(autenticar))


ip es el unico protocolo de red porque es ridiculamente barato en perspectiva

### organismos normalizadores
- IETF (internet engineer task force) -> crean las RFC, se reunen 3 veces al año, se discuten y crear protocolos
- ITU (internacional telecommunication union) -> regulan el espectro de las comunicaciones, o sea en que onda funcionara cada red
- W3C (world wide web consortium)

JITTER: cuanto varia la demora 

10ms de jitter es estandar, ya 30 es mucho

## http (hypertext transfer protocol)

se invento en el sern para documentar cosas(querian, mezclar texto, imagenm titulos y adaptarse a distintos monitores, tambien traia hipervinculos(abrir una pagina y llegar a otra)), 
- es cliente servidor 
- permite transmitir texto formateado, imagenes, etc.
- conexiones no persistentes (1 conexion por elemento) vs persistentes
- web caching
http: atiende en 80
https: atienede en 443

## DNS (internet no anda sin esto)

- relacion: nombre <-> @IP
- Base de datos jerarquia y distribuida(punto de falla, multiples consultas, administracion distribuida)
- otros servicios
    - host aliasing (para un mismo nombre halla varios nombres)

    ```
    www.fi.uba.ar -> aleph.fi.uba.ar
    smtp.fi.uba.ar ->
    ```
    - mail server aliasing (estos tienen prioridades para que no se pisen entre servidores)

    - load distribution(se encarga de que si queres acceder a una direccion esta te redirija a otras segun sea conveniente)

- top level domains(tlds): son los finales de domino, ejemplo: MIL, edu, arpa, ar, br, uba,org, etc

al final de cada direccion no se tipea pero siempre hay un punto final marcando el root, o sea las direcciones se buscan de atras hacia adelante

los root servers en el mundo se dicen que son 13, 13 direccione sip fijas a las cuales se les pregunta por cada dns para que es llame a otro hasta llegar al dns final

existen dos tipos de consultas

interativas y recursivas, en el dns local se hacen recursivas es decir se busca a si mismo mientras que las consultas externas dns son interativas, es decir para cada ip en cada dns ir buscando


5 zonas:
arin, raid, lacnic, afnic, apnic

las direcciones ip son unicas salvo para los servidores root

## tipos de consultas dns

- autorizadas o no (son si quiero hablar derecho al servidor dns o si quiero hablarle a una copia de cache)
- recursivas o iterativas
- tipo 
- - A : nombre @IP
- - NS: servidor DNS
- - CNAME: es el nombre canonico del host(pueden ser varios)
- - MX: Mail exchanger(tiene prioridad)
- - SOA: Start of a zone of authority 
- - PTR: pointer to record 

en unix existe el comando: dig usarlo para hacer una consulta dns y catchear en wireshark que te responde

## P2P 

si precisa algo robusto va p2p 