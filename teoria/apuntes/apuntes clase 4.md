# clase 4 capa de transporte

## transporte 

la capa de transporte permite a las aplicaciones comunicarse: multiplexar y demultiplexar

el transporte en internet: best effort(intenta lo mejor que puede), es un concepto heredado de la capa de red: equipos mas simples, protocolos menos complejos, mas escalable, mas robusto, mas barato

hizo crecer a la internet dominando la tecnologia de redes de comunicaciones

proporciona: multiplexacion de la comunicaciones y verificacion de errores minima
puede proveer:
- confiabilidad de las comunicaciones
- control de flujo
- seguridad 

## UDP(definida en RFC 768)

2^16 es el numero magico, el numero maximo de puertos

```

                     
                  0      7 8     15 16    23 24    31  
                 +--------+--------+--------+--------+ 
                 |     Source      |   Destination   | 
                 |      Port       |      Port       | 
                 +--------+--------+--------+--------+ 
                 |                 |                 | 
                 |     Length      |    Checksum     | 
                 +--------+--------+--------+--------+ 
                 |                                     
                 |          data octets ...            
                 +---------------- ...                 

                      User Datagram Header Format
```


## principios de comunicacion confiable

- conectada vs no conectada
- asegurar la entrega
- asegurar el orden de los mensajes  
- asegurar integridad de los datos
- desempeño (meet por ejemplo cuando ve mucha demora junta todos los paquetes de voz y los manda juntos, aunque eso no es tan importante en la mayoria de las aplicaciones)
- control de flujo 
- compartir el canal equitativamente

ntp: protocolo de tiempo de red.

### tipos de flujo

un flujo es ese tramo de la comunicacion entre un emisor y un receptor

### stop and wait

el emisor envia un mensaje y espera a que el receptor lo reciba y envie un mensaje de confirmacion, luego el emisor envia el siguiente mensaje

### continuo

el emisor envia un mensaje y no espera a que el receptor lo reciba, sino que sigue enviando mensajes, el receptor envia mensajes de confirmacion cada cierto tiempo, el emisor puede enviar varios mensajes antes de recibir una confirmacion

### Go-back-N

el emisor envia varios mensajes sin esperar a que el receptor los reciba, el receptor envia mensajes de confirmacion cada cierto tiempo, si el emisor recibe un mensaje de confirmacion con un numero de secuencia menor al ultimo mensaje enviado, entonces el emisor vuelve a enviar todos los mensajes desde ese numero de secuencia hasta el ultimo mensaje enviado

- checksum: es un numero que se calcula a partir de los datos del mensaje, el receptor calcula el checksum del mensaje recibido y lo compara con el checksum enviado por el emisor, si son iguales, entonces el mensaje se considera valido, si no, entonces el mensaje se considera corrupto y se descarta

- CRC(Cyclic Redundancy Check): es un metodo mas avanzado que el checksum, se basa en la division de polinomios, el emisor calcula el CRC del mensaje y lo envia junto con el mensaje, el receptor calcula el CRC del mensaje recibido y lo compara con el CRC enviado por el emisor, si son iguales, entonces el mensaje se considera valido, si no, entonces el mensaje se considera corrupto y se descarta

- temporizador: es un mecanismo que se utiliza para controlar el tiempo que el emisor espera a recibir un mensaje de confirmacion, si el temporizador se agota antes de recibir un mensaje de confirmacion, entonces el emisor vuelve a enviar el mensaje

- numero de secuencia: para mantener un flujo de paquetes y detectar perdidas

- acuse de recibo(ACK): para avisar que paquetes se han recibido

- acuse de recibo negativo(NAK): para avisar que un paquete se ha recibido corrupto

- ventana deslizante: es un mecanismo que se utiliza para controlar el numero de mensajes que el emisor puede enviar sin esperar a recibir un mensaje de confirmacion, la ventana se desliza hacia adelante cada vez que el emisor recibe un mensaje de confirmacion, el tamaño de la ventana determina el numero de mensajes que el emisor puede enviar sin esperar a recibir un mensaje de confirmacion

EJEMPLO DE USO DE UDP: dns



