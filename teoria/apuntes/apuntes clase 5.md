# clase 5

para crear una nuevo protocolo de transporte este deberia correr en todos los demas OS que ya existen. 

## tcp

como se numran los segmentos de TCP?

como se calculan los timeouts en TCP?

como se logra la tansmision confiable en TCP?

la cantidad minima de informacion que se puede mandar es un byte

MSS: maximum segment size: tamaño maximo de bytes enviables, los mss son negociados

en el primer paquete se envia el primer numero de secuencia que es un byte en un valor aleatoriamente generado

el mss lo determina el host que inicia la conexion, el host que responde a la conexion va a usar el mss que le envio el host que inicio la conexion

todo paquete tiene numero de secuencia y numero de ack que es el numero de secuencia del ultimo byte que se envio, y el numero de ack es el numero de secuencia del ultimo byte que se recibio mas uno, esto permite a TCP saber que bytes fueron recibidos correctamente y cuales no, y permite a TCP retransmitir los bytes que no fueron recibidos correctamente.


lo maximo que permite tcp en el tamaño de los bytes el es 32 bits, esto es porque el numero de secuencia es un numero de 32 bits, y el numero de ack es un numero de 32 bits, y el mss es un numero de 16 bits, esto permite a TCP tener un espacio de direccionamiento de 2^32 bytes, lo que es suficiente para la mayoria de las aplicaciones.

paradoja del general: si el general A envia un mensaje al general B, y el general B responde con un ack, el general A no sabe si el ack fue recibido correctamente por el general B, y el general B no sabe si el mensaje fue recibido correctamente por el general A, esto es una paradoja porque ambos generales necesitan saber que el otro recibio el mensaje correctamente para poder tomar una decision, pero ninguno de los dos puede estar seguro de que el otro recibio el mensaje correctamente, esto es un problema de comunicacion confiable en redes.

## calculo del timeout en TCP

```math
T(i) = RTT(i)* \alpha + T(i-1)(\alpha - 1) / 0 <= \alpha <= 1
```

normalmente se usa alpha = 1/8

```math
D(i) = RTT(i)* \beta + D(i-1)(\beta - 1) / 0 <= \beta <= 1
```

normalmente se usa beta = 1/4

```math
timeout(i) = T(i) + 4*D(i)
```

lo unico que define quien es el cliente y quien es el servidor y quies es el cliente es quien espera a la conexion y quien envia el primer paquete, ambos pueden enviarse paquetes hasta de manera simultanea.

en tcp cualquiera puede cerra la conexion mandando un paquete con el flag de fin.


### como gestionar congestion en TCP?

```math
w = min(rwnd, cwnd)-> reset window y congestion window

cwind = 1 mss --> se inicializa en 1 mss
ssthreshold = 65335 bytes --> es el numero magico, siendo este el valor maximo que puede tener el cwind

2cwin < sshreshold -> slow start
cwin >= sshreshold -> congestion avoidance
cwind(i) = cwind(i-1) + 1 mss / cwind(i-1) -> slow start
cwind(i) = cwind(i-1) + 1 mss / cwind(i-1) -> congestion avoidance
cwind(i) = cwind(i-1)/2 -> timeout
cwind(i) = cwind(i-1)/2 -> 3 duplicate ack


```

por cada ack recibido el cwin crece 1 mss, tod esto deriva de la version reno de tcp.
