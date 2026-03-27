# apuntes clase 3

## app layer

- el objetivo es que varias aplicaciones corran en diferentes hosts.
- se implementa en los distintos hosts

## protocolos

son una serie de procedimientos y formatos que se siguen para que la comunicación entre hosts sea posible.

- HTTP: Hypertext Transfer Protocol

- - request line:

- - - method
- - - sp 
- - - uri
- - - sp
- - - version
- - header fields


## DNS: Domain Name System

- es lo que esta en el medio del nombre del host y la ip

- es distribuida y jerarquica

- esta a nivel de aplicacion

- las bases de datos de DNS guardan RR, tuplas de 4 elementos:
- - name: el nombre del host
- - type: el tipo de registro (A, NS, CNAME, etc)
- - value: el valor del registro (la ip, el nombre del host, etc)
- - ttl: time to live, el tiempo que el registro es valido

## tipos de registros mas comunes

- A: address record, mapea un nombre de host a una ipv4
- AAAA: address record, mapea un nombre de host a una ip version 6
- NS: name server record, mapea un nombre de host a un servidor de nombres
- TXT: text record, mapea un nombre de host a un texto, se usa para cosas como SPF, DKIM, etc
- CNAME: canonical name record, mapea un nombre de host a otro nombre de host, se usa para alias
- MX: mail exchange record, mapea un nombre de host a un servidor de correo

### provee varios servicios:

- host aliasing: mapea un nombre de host a otro nombre de host
- mail aliasing: mapea un nombre de host a un servidor de correo
- load balancing: mapea un nombre de host a varias ip, el cliente puede elegir una de ellas, lo que permite distribuir la carga entre varios servidores

## porque dice que es descentralizado?

por que esta distribuido en varios servidores, cada uno con su propia base de datos, y no hay un servidor central que controle todo el sistema.

## dns local

es el que tiene el isp que uno suele contratar, es el que se usa por defecto cuando uno hace una consulta dns, y suele ser el que tiene la cache de las consultas anteriores, lo que hace que las consultas sean mas rapidas.

## CDN: Content Delivery Network

se encarga de reducir la distancia entre el proveedor de contenido y el usuario, para mejorar la velocidad de carga de las paginas web, y reducir la latencia, son servidores distribuidos geograficamente para distribuir el contenido de manera mas eficiente, y reducir la carga en los servidores principales, con esto reducimos latencia y maximizamos el throughput, y ademas mejora la disponibilidad del contenido, ya que si un servidor cae, el contenido sigue estando disponible en otros servidores de la red. 

### arquitectura de una CDN

- servidor de origen: es el que tiene el contenido original, y se encarga de distribuirlo a los servidores perifericos, es el que tiene la base de datos de los contenidos, y se encarga de actualizarla cuando hay cambios en el contenido.

- servidores perifericos: son los que almacenan copias del contenido original, y se encargan de entregarlo a los usuarios cuando realizan solicitudes.

- puntos de presencia: son un conjunto de servidores perifericos agrupados a los cuales realmente llega el usuario, y se encargan de entregar el contenido a los usuarios, y de comunicarse con el servidor de origen para actualizar el contenido cuando hay cambios.

### tipos de cdn

- privadas: son las que son propiedad de una empresa, y se encargan de distribuir el contenido de esa empresa, como por ejemplo la cdn de netflix, o la cdn de amazon.

- de terceros: son las que son propiedad de una empresa, pero se encargan de distribuir el contenido de varias empresas, como por ejemplo cloudflare, o akamai.

### filosofias para ubicar los point of presence

- enter deep: que este lo mas cerca posible del usuario, poner los points of access lo mas cercano posible a las ISP's,para reducir la latencia, y mejorar la velocidad de carga de las paginas web, pero esto puede ser costoso, ya que se necesitan muchos servidores perifericos para cubrir una gran cantidad de usuarios.

- bring home: ponerlo lo mas cercano a los IXP's los puntos de exchange, osea los puntos donde se conectan dos o mas isp's, para reducir la latencia, y mejorar la velocidad de carga de las paginas web, pero esto puede ser menos costoso, ya que se necesitan menos servidores perifericos para cubrir una gran cantidad de usuarios, pero puede ser menos efectivo para reducir la latencia, ya que los usuarios pueden estar lejos de los IXP's.


