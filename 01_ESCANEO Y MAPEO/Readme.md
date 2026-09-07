# **RESUMEN TEÓRICO Y PRACTICA: FOOTPRINTING, ESCANEO Y MAPEO** 
## **1. Paso previo: Footprinting**
Antes de ponerse a hacer escaneos contra un servidor, el primer paso en un test de intrusión es el Footprinting. Basicamente consiste en buscar la información pública que anda dando vueltas sobre nuestro objetivo, ya sea porque la publicaron a propósito o por descuido. Sin tocar directamente al objetivo/empresa, buscamos cosas como direcciones IP, dominios, cuentas de correo o que tipo de servidores usan por ejemplo. Toda esta información es la que nos va a dar la base para saber a dónde apuntar en la fase de escaneo.

## **2. Conceptos Clave de Red**
En la parte de red, tenemos que tener muy claro que estamos buscando cuando escaneamos un dispositivo.
 ~ Puerto: Es una zona donde dos hosts (computadoras) intercambian información.
 ~ Servicio: Es el tipo de información que viaja por ese puerto usando alguna utilidad específica, como puede ser SSH o Telnet.
 ~ Firewall: Es el filtro que acepta o rechaza el tráfico que entra o sale del dispositivo.

Cuando le preguntamos a un puerto cómo está, nos puede devolver tres estados principales:
 ~ Open: Podés acceder al puerto y hay un daemon (programa) escuchando del otro lado.
 ~ Closed: Podés llegar al puerto, pero no hay ningún servicio funcionando ahí. (Esto igual nos sirve porque nos confirma que hay un sistema Linux vivo en esa IP específica.)
 ~ Filtered: El puerto no es accesible. No sabemos si hay un servicio o no, porque un firewall nos está bloqueando el paso y filtrando la conexión.

## **3. NMAP: El mapeador de redes**
Acá es donde entra Nmap. Es una herramienta de software libre, gratuita y considerada el escáner de puertos más poderoso. Usa paquetes IP en bruto para ver qué máquinas están vivas en la red, qué servicios ofrecen, qué sistema operativo tienen instalado y qué tipo de firewall usan. Lo usamos para auditorías de seguridad y para juntar información clave antes de lanzar un ataque.

Como se divide el trabajo cuando usamos Nmap:

**1. Descubrimiento de Hosts:**
Después del Footprinting, necesitamos saber que máquinas están activas, porque si no responden, tenemos que pasar a otras. Nmap usa opciones como `-sn` (o `-sP`) para descubrir hosts sin escanearles los puertos, funcionando parecido a un ping para decirnos quien esta arriba en la red. Un detalle: si sos root y estás escaneando en una red local, Nmap usa peticiones ARP para encontrarlos de forma más directa.

**2. Técnicas de escaneo de puertos:**
Cuando sabemos que la maquina vive, probamos los puertos:
 ~ Escaneo TCP SYN (-sS): Es el que usa Nmap por defecto. Es mas sigiloso porque manda un paquete SYN (intentando iniciar una conexion TCP), pero nunca la termina.
 ~ Escaneo TCP Connect (-sT): Si no tienes permisos de administrador, Nmap usa este método. Lo que hace es completar toda la conexión TCP (el saludo de tres vias), pero la desventaja es que al tener éxito la conexión dejas mas registros en el servidor.
 ~ Escaneo UDP (-sU): Busca puertos UDP. Es un proceso más lento, pero te salva cuando necesitas descubrir que servicios estan detras de un firewall y usan este protocolo.
 ~ Escaneo ACK (-sA): Manda paquetes de reconocimiento (ACK) para ver como responde el sistema, muy util para mapear si un firewall esta filtrando o no esos mensajes.

**3. Reconocimiento de S.O (-O):**
Nmap le tira una serie de paquetes TCP y UDP al host y analiza casi cada bit de cómo responde (cosas como el tamaño de ventana o el ID de IP). Después agarra esos datos y los compara con una base de datos propia que tiene más de 2600 huellas para decirnos exactamente qué SO tiene la máquina.

**4. Deteccion de Versiones (-sV):**
Saber que el puerto 80 está abierto no alcanza. Con `-sV`, Nmap interroga al puerto para sacar el protocolo, el nombre de la aplicación y el número de versión exacto. Tener la versión precisa es fundamental, porque eso es lo que nos permite saber a qué exploit (código de explotación) es vulnerable ese servidor.

**5. Nmap Scripting Engine (NSE):**
Esta es de las funciones más copadas de Nmap. Te permite usar o escribir scripts en lenguaje LUA para automatizar tareas en la red. Sirven para encontrar puertas traseras (backdoors), hacer detecciones de versiones más sofisticadas o explotar vulnerabilidades de una. Por ejemplo, pasándole `--script vuln`, Nmap va a ejecutar todos los scripts enfocados en encontrar vulnerabilidades contra el objetivo.

**6. Opciones de uso**
Para que el escaneo sea a medida y no tardamos una eternidad, solemos combinar varios parámetros en la consola:
 ~ -p: Le indicas qué puertos querés revisar. Podés pasarle uno solo (ej. `-p 80`), varios separados por comas, o un rango completo (ej. `-p 1-65535`).
 ~ -n: Le decís a Nmap que no pierda tiempo haciendo resoluciones de DNS inversas, lo que hace que el escaneo vaya mucho más rápido.
 ~ -Pn (o -P0): Clave para evadir bloqueos. Le avisa a Nmap que no haga el ping inicial y que asuma que la máquina está viva, ideal cuando el firewall del objetivo bloquea los pings.
 ~ -oN, -oX, -oA: Son las opciones para guardar la salida del escaneo. Podés guardarlo en texto normal, formato XML, o usar `-oA` para guardarlo en los tres formatos principales a la vez.

---

# **Parte Práctica**

En esta parte vamos a hacer la fase de reconocimiento con Nmap para mapear la red y sacarle la ficha a la máquina víctima sin tocar ni explotar nada (todavía). Abríremos la terminal en Kali y vamos probando estos comandos, del más silencioso al más ruidoso.

**1. Descubrimiento de red (Host Discovery)**
* **Comando:** `~ nmap -sn 192.168.122.0/24`
* **Para qué sirve:** Hace un barrido tipo ping en toda la subred para ver qué equipos están vivos, sin escanearles los puertos. Es el primer paso ideal cuando te conectás a una red y todavía no sabés cuál es la IP de tu víctima.

**2. Escaneo Sigiloso (TCP SYN Scan)**
* **Comando:** `~ sudo nmap -sS 192.168.122.107`
* **Para qué sirve:** Es el escaneo por defecto cuando sos root. Manda paquetes SYN para iniciar una conexión TCP, pero la corta a la mitad antes de terminar el saludo de tres vías. Revisa los 1.000 puertos más comunes de forma rapidísima y es más difícil que un firewall básico deje el registro del escaneo.

**3. Búsqueda en todos los puertos**
* **Comando:** `~ nmap -p- 192.168.122.107` (también válido como `-p 1-65535`)
* **Para qué sirve:** Nmap normalmente revisa solo los mil puertos más populares para ahorrar tiempo, pero con este parámetro lo obligás a ver todos los 65.535 puertos posibles. Tarda un poco más, pero nos asegura agarrar servicios escondidos en puertos altos o poco comunes.

**4. Detección de Sistema Operativo (OS Fingerprinting)**
* **Comando:** `~ sudo nmap -O 192.168.122.107`
* **Para qué sirve:** Nmap le tira una serie de paquetes armados al host y analiza cómo responde a nivel de red. Luego compara esa respuesta con su base de datos interna para deducir qué sistema operativo está corriendo (te va a devolver que es un Linux kernel 2.6.X).

> **Nota:** Para sacar qué S.O corre la máquina, Nmap analiza la huella digital de la pila TCP/IP. Mira cómo responde el servidor a algunos paquetes armados. Para que el análisis funcione, tiene que encontrar obligatoriamente al menos un puerto abierto y uno cerrado. Al tirar el parámetro `-O` suelto, Nmap se ve obligado a lanzar primero su escaneo default por los 1000 puertos más comunes para tener de dónde agarrarse y hacer la prueba del SO. 
> Más adelante veremos cómo reducir el “ruido” que genera este análisis en la red.

**5. Extracción de Versiones**
* **Comando:** `~ nmap -sV 192.168.122.107`
* **Para qué sirve:** A nmap no le alcanza con saber que el puerto 21 está abierto; interactúa con el servicio para leer los banners y decirnos el software y versión exacta (ej. vsftpd 2.3.4). Esta es la información de oro que un pentester usa después para buscar las CVEs en internet.

**6. El Escaneo Agresivo (Todo en uno)**
* **Comando:** `~ sudo nmap -A -v 192.168.122.107`
* **Para qué sirve:** El modificador `-A` es un combo: activa la detección de SO, el escaneo de versiones, trazado de rutas y corre unos scripts básicos (NSE) de Nmap al mismo tiempo. Le sumamos la `-v` (verbose) para que nos vaya escupiendo los resultados en la pantalla en tiempo real y no te quedes mirando la terminal negra pensando que se tildó.

**7. Buscador automático de vulnerabilidades (NSE)**
* **Comando:** `~ sudo nmap --script vuln 192.168.122.107`
* **Para qué sirve:** Utiliza el Nmap Scripting Engine (NSE), una de las funciones más interesantes de la herramienta. Además de darnos qué está abierto, ejecuta programas automatizados para buscar vulnerabilidades conocidas sobre esos mismos servicios en tiempo real.


**8. Guardar la evidencia (Output Format)**
* **Comando:** `~ nmap -sV -oN resultado.txt 192.168.122.107` (o `-oA` para guardar en todos los formatos a la vez).
* **Para qué sirve:** Redirige toda la salida a un archivo de texto en tu máquina atacante. Nunca conviene hacer un escaneo en el aire; siempre lo guardamos para no tener que escanear todo de nuevo.

Conociendo ya como funciona básicamente nmap, podemos hacer combinaciones mas personalizadas. Lo que nos permite por ejemplo reducir el ruido que hace el escaneo en la señal. Entonces si estamos en una auditoría y por ejemplo: queremos sacar el sistema operativo pasando lo más desapercibido posible, no dejamos que Nmap vea los 1000 puertos. Lo que haríamos es combinar el parámetro `-O` con `-p`, indicando un par de puertos que están abiertos y cerrados. Por ejemplo: `sudo nmap -O -p 22,80,9999 192.168.122.107`. De esta forma, Nmap hace el análisis golpeando solo esas 3 puertas en lugar de mil.

**9. Comandos Bonus**

 ~ Evadir el bloqueo de Ping (Host Discovery Disabled)
  * **Comando:** `~ nmap -Pn 192.168.122.107` (también válido como `-P0`).
  * **Para qué sirve:** Muchos administradores configuran el firewall para que ignore los paquetes ICMP (ping) y la máquina parezca "apagada". Este modificador le avisa a Nmap que saltee la prueba inicial, asuma que el servidor está vivo, y vaya a golpear directo a los puertos.    

 ~ Escaneo de puertos UDP
  * **Comando:** `~ sudo nmap -sU 192.168.122.107`
  * **Para qué sirve:** Por defecto, Nmap solo prueba puertos TCP. Este comando envía paquetes UDP para descubrir servicios que trabajan bajo ese protocolo. Se demora por lo que es más lento, pero asegura no dejar puntos ciegos y permite auditorías mucho más exactas.  

 ~ Fragmentación de paquetes
  * **Comando:** `~ sudo nmap -f 192.168.122.107`
  * **Para qué sirve:** Permite partir los paquetes de red en pedazos muy chicos. Es una técnica de evasión que hace muchísimo más complejo para un firewall lograr hacer el rastreo o detectar que lo estás escaneando.

> **Nota:** El desarrollo paso a paso del laboratorio, las capturas de pantalla de las terminales probando NMAP se encuentran documentadas en detalle en el archivo `.pdf` adjunto en esta misma carpeta.
