# Onix: A Distributed Control Platform for Large-scale Production Networks — Resumen

> Koponen, Casado, Gude, Stribling, Poutievski, Zhu, Ramanathan, Iwata, Inoue, Hama, Shenker.
> *9th USENIX Symposium on Operating Systems Design and Implementation (OSDI '10)*, Vancouver, BC, octubre de 2010.
> Autores de Nicira Networks, Google, NEC e ICSI/UC Berkeley.

---

## 1. Contexto histórico

Durante décadas, la tecnología de red mejoró enormemente en velocidad de línea, densidad de puertos y relación precio/rendimiento, pero los **mecanismos del plano de control avanzaron mucho más lento**: diseñar y desplegar un protocolo de control nuevo podía llevar años (el paper cita TRILL, que estuvo más de seis años en fase de diseño y especificación).

El modelo tradicional tenía un problema estructural: la lógica de control vivía **embebida dentro de cada router/switch**, mezclada con el plano de datos. Cada dispositivo corría sus propios protocolos distribuidos (OSPF, IS-IS, BGP), pero ese intercambio estaba **limitado a información de enlace y alcanzabilidad**, con un modelo de distribución fijo. Como consecuencia, cada función de control nueva (por ejemplo, ruteo escalable de direcciones planas) obligaba a:

1. resolver un problema de diseño difícil y de bajo nivel (su propio protocolo distribuido), y luego
2. enfrentar la dificultad de desplegarlo sobre el hardware de los switches.

El resultado era una colección "barroca" de protocolos con propiedades de escalabilidad y convergencia distintas, y una innovación lentísima.

**La línea de investigación previa** que desemboca en Onix: 4D → RCP → SANE → Ethane → **NOX**. De todos ellos, solo NOX podía considerarse una plataforma de control con una API de propósito general. La cronología relevante:

- **2008** — OpenFlow (Stanford) y NOX (Nicira), el primer controlador OpenFlow/SDN.
- **2010** — Onix. En esencia, **Onix toma NOX y lo distribuye sobre múltiples servidores**, agregando lo que a NOX le faltaba: confiabilidad y flexibilidad para escalar.

> **SDN (Software-Defined Networking)**, tal como lo usa el paper, es el paradigma donde el plano de control se **desacopla** del plano de forwarding y se construye como un **sistema distribuido** sobre una o más máquinas, que supervisa switches "tontos".

---

## 2. El problema y la propuesta

Las redes no tenían un **paradigma de control general** ni abstracciones de gestión a nivel de toda la red. Onix se propone como una **plataforma de control de calidad productiva** sobre la cual el plano de control se implementa como sistema distribuido, operando sobre una **vista global de la red** y usando primitivas de distribución de estado provistas por la plataforma.

La filosofía central de SDN según el paper: **las primitivas básicas de distribución de estado deben implementarse una sola vez en la plataforma**, usando técnicas conocidas de la literatura de sistemas distribuidos, en lugar de reinventarse para cada tarea de control con algoritmos especializados.

### Desafíos de una plataforma de control productiva
- **Generalidad** — la API debe permitir aplicaciones muy diversas en contextos diversos.
- **Escalabilidad** — los límites de escala deben venir del problema inherente de gestión de estado, no de la implementación de la plataforma.
- **Confiabilidad** — manejar fallos de equipo (y otros) con gracia.
- **Simplicidad** — facilitar la construcción de aplicaciones de gestión.
- **Rendimiento del plano de control** — adecuado, no necesariamente óptimo (se prioriza generalidad sobre rendimiento cuando hay que elegir).

### Contribuciones principales de Onix
1. Una **API mucho más general** que sistemas previos (apunta a WAN, nube pública y datacenter empresarial).
2. **Primitivas de distribución flexibles** (almacenamiento DHT, membresía de grupo) que permiten construir aplicaciones de control sin reinventar los mecanismos de distribución, conservando la libertad de hacer trade-offs de rendimiento/escalabilidad.

---

## 3. Componentes (los 4 elementos de una red controlada por Onix)

1. **Infraestructura física** — switches, routers y otros elementos (incluso balanceadores) que exponen una interfaz para que Onix lea y escriba el estado que controla su comportamiento (ej. entradas de la tabla de forwarding). No necesitan correr nada más que esa interfaz y conectividad básica.
2. **Infraestructura de conectividad** — el canal por donde viaja el "tráfico de control" entre el hardware y Onix. Puede ser *in-band* (comparte la red de datos) o *out-of-band* (red física separada). Protocolos estándar como IS-IS u OSPF sirven para mantenerla.
3. **Onix** — el sistema distribuido en sí, corriendo sobre un clúster de uno o más servidores (cada uno puede correr múltiples instancias). Da a la lógica de control acceso programático a la red y disemina el estado entre instancias para escalar (millones de puertos) y tolerar fallos.
4. **Lógica de control** — implementada *sobre* la API de Onix; determina el comportamiento deseado de la red. Onix solo provee las primitivas para acceder al estado.

> **Rango de aplicabilidad:** se asume que la infraestructura física reenvía paquetes mucho más rápido (dos o más órdenes de magnitud) de lo que Onix puede procesarlos. Por eso Onix **no** está pensado para funciones que requieran conocer cambios de estado por-paquete.

---

## 4. La NIB (Network Information Base) — el corazón del sistema

La copia del estado de red que mantiene Onix se guarda en la **NIB**, análoga (a grandes rasgos) a la RIB de los routers IP. Pero a diferencia de la RIB —que solo guarda prefijos a destinos— la **NIB es un grafo de todas las entidades de red de la topología**.

La NIB es a la vez:
- el **modelo de control** (las aplicaciones se implementan leyendo y escribiendo la NIB), y
- la **base del modelo de distribución** (Onix replica y distribuye la NIB entre instancias).

La lógica de control típica es sencilla: se **registra** para ser notificada ante un cambio de estado (ej. la aparición de nuevos switches y puertos) y, cuando salta la notificación, manipula el estado modificando los pares clave-valor de las entidades afectadas.

Características importantes de la NIB:
- Mantiene un **índice** por identificador de entidad para consulta directa.
- Soporta **registro de notificaciones** ante cambios y altas/bajas de entidades.
- Provee **acceso exclusivo local** (no locking distribuido ni fino): garantiza que ningún otro hilo de *la misma* instancia toca la NIB, pero **no** garantiza que otras instancias o elementos de red no la modifiquen. La coordinación entre instancias se resuelve por mecanismos externos a la NIB.
- Todas las operaciones son **asincrónicas**: actualizar una entidad solo garantiza que el mensaje *eventualmente* se enviará, sin garantías de orden ni latencia. Existe una **primitiva de sincronización** (callback) para saber cuándo un cambio efectivamente se empujó al hardware u otras instancias.

### Funciones de la API de la NIB (Tabla 1 del paper)
| Categoría | Propósito |
|---|---|
| Query | Encontrar entidades. |
| Create / destroy | Crear y eliminar entidades. |
| Access attributes | Inspeccionar y modificar entidades. |
| Notifications | Recibir avisos de cambios. |
| Synchronize | Esperar a que las actualizaciones se exporten a elementos de red y controladores. |
| Configuration | Configurar cómo se importa/exporta el estado de la NIB. |
| Pull | Pedir que se importen entidades bajo demanda. |

---

## 5. Paralelismo con el paradigma orientado a objetos (POO) ⭐

Este es, a mi juicio, uno de los aspectos más elegantes del diseño, y **no es una analogía superpuesta: Onix está literalmente implementado con POO en C++.**

- **Clase base**: la NIB contiene entidades de red genéricas, cada una con un conjunto de pares **clave-valor** y un identificador global plano de **128 bits**. Esa es la estructura base de la que derivan todos los tipos.
- **Herencia**: las **entidades tipadas** (`Node`, `Port`, `Link`, `ForwardingEngine`, `ForwardingTable`, `Host`) heredan de esa clase base. En la **Figura 2** del paper, las **líneas sólidas representan herencia** y todas las clases tipadas comparten la base que provee el acceso genérico clave-valor.
- **Atributos + métodos = objeto**: las entidades tipadas contienen un conjunto predefinido de atributos (los pares clave-valor) **y métodos** para operar sobre ellos.
- **Asociaciones / composición**: las **líneas punteadas** de la Figura 2 son **relaciones referenciales entre instancias** (un `Node` tiene una lista de `Port`s; un `Link` mapea a dos `Port`s, y dos `Port`s pueden mapear al mismo `Link`). Es composición/asociación entre objetos, no herencia.
- **Extensibilidad por subclassing**: el conjunto de tipos **no es fijo**; las aplicaciones pueden **subclasear** las clases base para extender el modelo de datos (e incluso controlar el trade-off memoria/CPU sobre cómo se almacenan los pares clave-valor).
- **Encapsulamiento como herramienta de diseño**: la resolución de inconsistencias se **embebe dentro de las propias entidades vía herencia**. La aplicación extiende la clase para meter ahí la lógica de detección de inconsistencias referenciales, de modo que la lógica de control nunca quede expuesta a estado inconsistente.

**El matiz clave:** estos objetos **no son objetos en memoria comunes**, sino el **espejo vivo del estado real del hardware**. Cuando la aplicación agrega una entrada de flujo a una entidad `ForwardingTable`, el componente de exportación de OpenFlow lo traduce en una operación que escribe la entrada en la **TCAM** del switch; y a la inversa, las entradas de la TCAM quedan accesibles leyendo esa misma entidad. Conceptualmente se parece más a un **ORM** (objeto respaldado por estado externo) que a un objeto puramente en RAM.

> **Cómo decirlo en una línea:** Onix modela la red como un **grafo de objetos** (la NIB), donde cada elemento físico se mapea a uno o más objetos con atributos y métodos, organizados por **herencia** y extensibles por **subclassing**, igual que en POO; la diferencia es que esos objetos están respaldados por estado real de la red y actúan como punto de integración y sincronización con el hardware.

---

## 6. Escalabilidad

Onix ofrece tres estrategias para escalar:

- **Partición (Partitioning)** — una instancia mantiene solo un **subconjunto** de la NIB y/o se conecta solo a un subconjunto de elementos, procesando menos eventos. Agregar instancias **reduce trabajo** en lugar de solo replicarlo.
- **Agregación (Aggregation)** — un clúster de Onix puede exponerse como **un único nodo agregado** en la NIB de otro clúster. Permite estructuras **federadas y jerárquicas** (ej. cada edificio de un campus gestionado por un Onix que se presenta como un solo nodo a un Onix global de ingeniería de tráfico). Espíritu similar a PNNI de ATM.
- **Consistencia y durabilidad** — la lógica de control **decide** los requisitos de consistencia de cada porción de estado. Por defecto Onix ofrece **dos data stores**: una base de datos transaccional replicada (durabilidad/consistencia fuerte) y un DHT en memoria (estado volátil tolerante a inconsistencias).

**Ejemplo guía del paper** (app que establece rutas entre switches): se empieza con una sola instancia → se *particiona* el estado de forwarding cuando la memoria no alcanza → se *agrega* la topología en nodos lógicos cuando la CPU no da abasto con los eventos → se *particiona aún más* la NIB para dominios grandes → hasta llegar a escenarios **inter-AS**, donde por privacidad ya no se comparte la topología completa y las instancias de ASes adyacentes "pelean" peering compartiendo topología a cierto nivel de detalle y estableciendo rutas entre sí. La estrategia resultante se parece mucho a **MPLS jerárquico**.

---

## 7. Confiabilidad

Las aplicaciones sobre Onix deben manejar **cuatro tipos de fallo**:

1. **Fallos de elemento de forwarding** y **fallos de enlace** — se manejan con los mismos mecanismos que los planos de control modernos (desviar tráfico). Conviene apoyarse en **backup paths** con failover rápido en el propio elemento.
2. **Fallos de instancia Onix** — dos opciones: que instancias vivas detecten al nodo caído y asuman sus responsabilidades, o que **más de una instancia gestione** simultáneamente un elemento (manejando condiciones de carrera de *lost update*). Si toda instancia computa el estado de forma **determinista** (mismo algoritmo en todas), la inconsistencia es solo transitoria. Enfoque similar a RCP.
3. **Fallos de la infraestructura de conectividad** — los mecanismos de distribución se desacoplan de la topología subyacente y necesitan conectividad para recuperarse. Soluciones comunes: red/VLAN de gestión dedicada (aislada del plano de datos), o estado de forwarding estático combinado con **source routing + multipathing** para conectividad ultra-confiable.

---

## 8. Distribución de la NIB

Dos observaciones guiaron el diseño: (a) las aplicaciones difieren en escalabilidad, frecuencia de actualización y durabilidad; (b) difieren en requisitos de **consistencia** (un flag de enlace transitoriamente inconsistente es fácil de resolver; una inconsistencia en una política de red puede requerir intervención humana).

Onix provee **dos mecanismos de distribución de estado entre instancias**:

- **Base de datos transaccional replicada** (respaldada por una máquina de estados replicada) — para estado que requiere **durabilidad y consistencia simplificada**, cambia lentamente. API tipo **SQL** con triggers y modelos de datos ricos. Tiene **limitaciones severas de rendimiento** (es solo un mecanismo de diseminación confiable de estado lento). Se integra a la NIB con módulos import/export.
- **DHT en memoria, de un salto, eventualmente consistente** (similar a **Dynamo**) — para estado con **alta tasa de actualización y disponibilidad**, relajando consistencia/durabilidad. API `get/put` más **soft-state triggers** (callbacks que hay que reinstalar). Puede devolver **múltiples valores** para una clave, y la **resolución de conflictos queda a cargo de la aplicación**.

Onix también puede soportar **sistemas de almacenamiento arbitrarios** si la aplicación escribe sus propios módulos import/export.

### Gestión del estado del elemento de red
Onix **no impone** un protocolo único hacia los switches; la interfaz primaria es la NIB. Soporta:
- **OpenFlow** — canal optimizado para gestionar tablas de forwarding y aprender cambios de estado de puertos. Onix traduce eventos/operaciones OpenFlow a entidades de la NIB (y viceversa con la TCAM).
- **Protocolo de base de datos de configuración** (estilo **Open vSwitch / OVSDB**) — para configuración y estado general del switch que OpenFlow no expone. Semántica parecida a la base transaccional entre controladores, pero con un lenguaje de consulta más restringido (sin joins).

### Consistencia y coordinación
- La NIB es el **punto central de integración** de múltiples fuentes (otras instancias + elementos de red): las fuentes **no interactúan entre sí directamente**, sino que importan/exportan estado hacia la NIB.
- La integración **no exige consistencia fuerte**; la aplicación registra **lógica de resolución de inconsistencias** de dos formas: (1) por **herencia** en las entidades (C++), y (2) vía **plugins** de conflicto en los módulos import/export.
- Para coordinación, Onix embebe **Zookeeper**, exponiendo una API orientada a objetos sobre su namespace jerárquico (consenso, membresía de grupo, detección de fallos).

---

## 9. Implementación

- Unas **150.000 líneas de C++** integrando varias librerías de terceros.
- En su forma más simple, Onix es un *harness*: se comunica con los elementos de red, agrega esa información en la NIB y ofrece un **framework** donde el programador escribe la aplicación de gestión.
- Una instancia puede correr en **múltiples procesos**, cada uno en un **lenguaje distinto** (soporta **C++, Python y Java**), interconectados por el mismo sistema de RPC que usan las instancias entre sí, pero sobre **IPC local** en vez de TCP/IP.
- Módulos como **componentes débilmente acoplados**, cargables/descargables dinámicamente sin recompilar, mientras se respete la interfaz binaria.

> **Sobre "¿es un framework de C++?":** el núcleo sí es C++ y expone un framework/API, pero la categoría correcta es **plataforma de control distribuida** (un *runtime* que se despliega y corre como sistema distribuido), no una mera librería; y de cara al desarrollador es **políglota** (C++/Python/Java).

---

## 10. Aplicaciones (casos productivos)

| Aplicación | Qué hace | Aspectos de Onix que estresa |
|---|---|---|
| **Ethane** | Política de seguridad de red empresarial; declara políticas centralizadas con nombres de alto nivel (lenguaje **FML**). Procesa el primer paquete de cada flujo, ubica hosts, aplica política y arma el forwarding. Usa **DHT** para link-state (decenas de miles de updates/s). | Flow Setup, Distribución, Disponibilidad |
| **Distributed Virtual Switch (DVS)** | Switch lógico distribuido en hipervisores; las políticas siguen a las VMs al migrar. Onix solo interviene al crear/destruir/migrar VMs. Config persistida en la base transaccional; alta disponibilidad por *cold standby*. | Integración |
| **Datacenter virtualizado multi-tenant** | Redes L2 por inquilino con aislamiento de direcciones/recursos; encapsula en el borde (túneles). Túneles pair-wise → **O(N²)** → requiere particionar y publicar endpoints por DHT. | Distribución, Disponibilidad |
| **Router IP carrier-grade scale-out** | Router BGP construido con switches commodity como "line-cards"; Onix es el "pegamento" entre hardware y un stack BGP open source, traduciendo la RIB a entradas de flujo. (En fase de diseño.) | Flow Setup / control tradicional |

---

## 11. Métricas y evaluación

> **NOTA:** dejar aquí las imágenes de los gráficos del paper. Cada bloque tiene su placeholder y la referencia a la figura/tabla correspondiente.

La evaluación combina **micro-benchmarks** (rendimiento como plataforma general) y mediciones **end-to-end** de una aplicación en desarrollo.

### 11.1 Escalabilidad — nodo único

**Throughput de modificación de atributos de la NIB** (según cantidad de listeners; curvas para 1, 10 y 1000 atributos). Con una sola modificación el test mide esencialmente la librería de hilos (el acceso exclusivo a la NIB equivale a un context switch); con más atributos por context switch sube el throughput efectivo.

<img width="304" height="160" alt="image" src="https://github.com/user-attachments/assets/44d21369-8614-487f-b4ff-b26e6167d789" />


**Uso de memoria de la NIB** (según cantidad de entidades y atributos por entidad). Una entidad de cero atributos consume **191 bytes** (incluyendo overhead de indexado); cada atributo ~16 bytes + 8 de identificador. Conclusión: una sola instancia en máquina servidor maneja **millones de entidades**.

<img width="284" height="154" alt="image" src="https://github.com/user-attachments/assets/9cdcb876-450f-4729-83b3-507e7ef6b108" />


**Paquetes reenviados por segundo** según número de conexiones OpenFlow (benchmark del stack OpenFlow). Más de **100.000 paquetes/s** con hasta ~1000 conexiones concurrentes; los "escalones" se deben al scheduling del SO sobre múltiples cores.

<img width="290" height="146" alt="image" src="https://github.com/user-attachments/assets/fa8f24e9-eb32-43f1-9a91-7f31ac81fe3e" />


### 11.2 Escalabilidad — multi-nodo

**Throughput de RPC** (el techo del DHT en memoria está limitado por el stack RPC de Onix). Ejemplo del paper: con NIB replicada a 5 instancias, cada update genera **22 pares request-response** → ~**24.000 updates pequeños/s** en agregado (ej. actualizar un atributo de carga en 24.000 enlaces por segundo).

<img width="302" height="176" alt="image" src="https://github.com/user-attachments/assets/e1d1714f-a07b-4331-83de-287553c9aba4" />

**Latencia de propagación de un valor en el DHT** (CDF, red de 5 nodos en LAN). En el peor caso, un update puede tardar hasta **4×** lo que tarda un push (1 hop para poner el valor, 1 para notificar, 2 para hacer get).

<img width="294" height="146" alt="image" src="https://github.com/user-attachments/assets/3bfc3ed9-000a-4d3e-a74a-1a4331a00fde" />

**Throughput de la base de datos transaccional replicada** (no optimizada para throughput; datos relativamente estáticos). Tabla 3 del paper:

| Queries/transacción | 1 | 10 | 20 | 50 | 100 |
|---|---|---|---|---|---|
| **Queries/s** | 49.7 | 331.9 | 520.1 | 541.7 | 494.4 |

<!-- (Opcional) PEGAR IMAGEN AQUÍ si querés la tabla como captura -->

### 11.3 Confiabilidad

- **Fallos de enlace/switch:** Onix monitorea conexiones con *keepalives* agresivos; los switches monitorean enlaces/túneles con sondeo por hardware (ej. **802.1ag CFM**).
- **Fallos de instancia:** detección y reconfiguración vía Zookeeper.
- **Test end-to-end (app multi-tenant):** ante la caída de un switch que hospeda un túnel, la **disrupción mediana host-a-host fue de 1120 ms**. Dado el timeout de detección configurado en 1 segundo, Onix tarda ~**120 ms** en reparar el túnel una vez detectado el fallo. Resultados prometedores, comparables a implementaciones de ruteo tradicionales (y la app está sin optimizar).

<img width="302" height="128" alt="image" src="https://github.com/user-attachments/assets/3548eb1c-3c4e-4b3b-b619-8015fe468a61" />


> **Escala probada:** redes de hasta **64 switches** con una sola instancia, y clústeres de hasta **5 instancias**.

---

## 12. Trabajo relacionado (en breve)

Onix desciende de la línea que separa control de dataplane (4D, RCP, SANE, Ethane, NOX, OpenFlow), pero con foco en ser **productivo a gran escala** (confiabilidad, escalabilidad, generalidad). Es **complementario** a la línea de planos de forwarding extensibles (RouteBricks, Click, XORP) y puede servir de plataforma para arquitecturas de datacenter como SEATTLE, VL2 y PortLand. Sigue además la tradición de sistemas distribuidos que **relajan la consistencia** con ayuda de la aplicación (Bayou, PRACTI, WheelFS, PNUTS).

---

## 13. Conclusión e ideas para llevarse

- SDN usa la **plataforma de control** para **simplificar** la implementación del control de red: en vez de lidiar con el hardware, el desarrollador programa su lógica sobre una **API de alto nivel**.
- Onix **convierte los problemas de red en problemas de sistemas distribuidos**, resolubles con conceptos familiares para esa comunidad.
- El paper **no trata sobre la ideología de SDN, sino sobre su implementación**: Onix es una **prueba de existencia** de que estas plataformas son factibles, y **no requirió mecanismos novedosos**, solo el uso juicioso de prácticas estándar de diseño de sistemas distribuidos.
- **Honestidad del paper:** Onix **no resuelve por sí solo** todos los problemas de gestión de red; el diseñador de aplicaciones aún debe entender las implicancias de escala. Onix da herramientas generales para gestionar estado, pero no hace desaparecer mágicamente los problemas de escala y consistencia. Aun así, en los casos vistos, construir aplicaciones de gestión es **mucho más fácil con Onix que sin él**.

---

### Apéndice: linaje y sucesores (contexto para la presentación)
- **OpenFlow / NOX (2008)** → **Onix (2010)** → ramas:
  - **Comercial (VMware/Nicira):** Onix fue base del controlador SDN de VMware (linaje hacia **VMware NSX**).
  - **Google:** Onix fue su controlador de 1ª generación; sucesor directo **Orion** (microservicios, modelo reconciliation-driven, conceptualmente cercano a Kubernetes).
  - **Open source (ideas heredadas):** **ONOS** (open-sourceado en 2014, foco carrier) y **OpenDaylight** (fundado en 2013, Linux Foundation, multi-protocolo).
