![Pasted image 20251110231243.png](../../_assets/b2b27aa2a864b460ee8d7631d027e7b02918267816b4b3ae2f565f4ee2ae78e7.png)

***Jhair***

**Source:** VulnHub
**Level:** 
**Date:** 10 nov 2025
**Autor**:
**descripcion**: 

Web Enumeration
Information Leakage - Log Exposure
Abusing SUID Binary (Serv-U FTP Server < 15.1.7) [Privilege Escalation]

**Flags**: 



**Begin**:

---

### **Recognition**
Comenzamos por identificar qué sistemas operativos están presentes en esa red.
```ruby
nmap -O 192.168.20.0/24
```
![Pasted image 20251110235829.png](../../_assets/1ebce3b54452a1865d616c94e2c587eda6b65f59dda7d75d9919a94bd881770a.png)

```ruby
sudo arp-scan -I eth0 --localnet
```

![Pasted image 20251110235943.png](../../_assets/44b0519943eee5e157b229526d96e3e933dd29012a75e98d508bf7bb418362f5.png)
Con `settarget` anclamos la ip con su nombre en la esquina
![Pasted image 20251111000459.png](../../_assets/f5ca4d6180700d0f4153fc6245f0a3c5cbf3b39a539771e33eb63e0f983974ca.png)
y procedemos a crear un directorio con el nombre de la maquina y con `mkt`creamos nuestros 3 directorios de trabajo.
![Pasted image 20251111000643.png](../../_assets/04b6daabb426349094768d7a71d67bbcd0bde68f35ae06e544d6b57049c0bc0b.png)
entramos a la carpeta nmap y comenzamos a hacer el reconocimiento
![Pasted image 20251111000734.png](../../_assets/2950b44fdd9ffc642c318a2d03720ad5a990a97e250aa7a3fbb96069970b0d8c.png)
hacemos ping 
```ruby
192.168.20.134
```

![Pasted image 20251111000854.png](../../_assets/804b0b6fe209499db76abd2cd00b0d7d9352525f47d51c50989ca9d3c7297e6c.png)
TTL 64 -> Linux

```ruby
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.20.130 -oG allPorts
```

![Pasted image 20251111002142.png](../../_assets/b0d71030f14ff65aa28fbd9a1347628d132dcbb857ca59533d92f684bfcd061e.png)
Hacemos un scaneo a los puertos 22 y 80
```ruby
nmap -sCV -p22,80 192.168.20.134 -oN targeted
```
![Pasted image 20251111223838.png](../../_assets/8186087a9f50c40bd9009c5907dbffbc7156f1f011e1a4077110f0245b266722.png)

### **Enumeración**

hacemos un `whatweb` a la direccion del puerto 80 abierto
![Pasted image 20251111220009.png](../../_assets/8d59305470d58a97d4ad34ea414b78556ff019a896989977b5878b77f32bcd18.png)
luego hacemos: un --script http-enum (este srcipt actua como un fuzzing )
```ruby
nmap --script http-enum -p80 192.168.20.134 -oN webScan
```
- `--script http-enum`  

Le indica a Nmap que ejecute el script del NSE llamado **http-enum**.  
`http-enum` intenta **enumerar recursos web comunes** (rutas/directorios, ficheros, aplicaciones web típicas) en el servidor HTTP objetivo, comprobando una lista de rutas conocidas y reportando las que existan. No es un escáner de vulnerabilidades profundo; es más bien un buscador de endpoints/paths comunes (p.ej. `/admin`, `/phpmyadmin`, `/robots.txt`, `/wp-login.php`, etc.).

- `-p80`  
Escanea únicamente el puerto **80** (HTTP no cifrado).

![Pasted image 20251111224034.png](../../_assets/b61c258faec20df4d2809e754a3e2e927ee6ffcc1213d79f33daeef1ac70fa64.png)

**¿Qué es http-enum?**

http-enum es un script del Nmap Scripting Engine (NSE) diseñado para enumerar rutas/directorios y recursos comunes en servidores web (p. ej. /admin, /phpmyadmin, /robots.txt, /wp-login.php, etc.). Usa una base de "huellas" (fingerprints) similar al formato de Nikto y patrones avanzados para identificar no solo rutas sino también, cuando es posible, software/versión asociado. 

**¿Cómo funciona, a grandes rasgos?**

Carga una base de caminos/huellas (incluida con Nmap) que contiene rutas y patrones a probar.
Itera sobre esa lista y hace peticiones HTTP (GET/HEAD según el caso) a cada path.
Analiza las respuestas (códigos HTTP, redirecciones, longitud de contenido, patrones en el body) para decidir si el recurso existe y, a veces, para identificar aplicaciones/versiones.
Presenta en la salida las rutas detectadas, códigos HTTP y posibles coincidencias con huellas.



luego ejecutamos en comando 
```ruby
curl -s http://192.168.20.134/robots.txt
```
* curl
Herramienta de línea de comandos para realizar peticiones HTTP(S) y otros protocolos.
* -s (--silent)
Modo silencioso: suprime la barra de progreso y mensajes de error. Solo imprime la respuesta del servidor (stdout). Ojo: si hay un error, no verás mensajes explicativos en stderr.

![Pasted image 20251111230506.png](../../_assets/27aed4553ebcf7cda0f7043ceafdd923328b7d5564421cf06946d8f450f7de15.png)
Cada palabra probablemente **indica un directorio existente o sensible** en el servidor

Probamos los directorios:
curl -I http://192.168.20.134/admin/
curl -I http://192.168.20.134/wordpress/
curl -I http://192.168.20.134/user/
curl -I http://192.168.20.134/election/
y ninguno da.

Probamos el: `/phpmyadmin/`

![Pasted image 20251111231317.png](../../_assets/68bef398e56557edca25d4013c2e0e722e83dd1005e01f2876a7ab592d45ce08.png)
![Pasted image 20251111231337.png](../../_assets/bf3d63f2ed25be052ee2bc7031c98ccd1cf49f95b8123c14cf1134576dae686e.png)

y tambien el: `/phpinfo.php`

![Pasted image 20251117173424.png](../../_assets/34c3c5fecea0c52ccced279e85fbc5dadfac1f94293b3443e444e4e2d93cc881.png)
vamos hasta la parte de `/disable_funcitons`

![Pasted image 20251117173610.png](../../_assets/2c029c575c96ce6078b9efcf7960255f8746848ee8e1411a63fac852b9ff8902.png)

 y tambien vamos a la parte de `/election`![Pasted image 20251117173752.png](../../_assets/01917da261e80850e14817b1af00b3d6c33e1c8b01313e41f887163e4514195e.png)

Utilizaremos la herramienta `gobuster` para fuzear 

Fuzzing es una técnica usada en ciberseguridad, testing de software y análisis de vulnerabilidades que consiste en:
Enviar grandes cantidades de datos aleatorios, inesperados o malformados a una aplicación, servicio o protocolo para ver cómo reacciona y encontrar fallos o vulnerabilidades.

Gobuster sirve para **descubrir recursos ocultos o no listados** en un servidor web.


```bash
gobuster dir -u http://192.168.20.134/election/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -t 20
```

`gobuster`
Es la herramienta que estás ejecutando para hacer fuzzing

`dir`
Es el modo de Gobuster.
dir = enumeración de directorios y archivos.

`-u http://192.168.20.134/election/`
-u (URL) indica la dirección objetivo.

`-w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt`
-w = wordlist.
La ruta completa indica el archivo de palabras que Gobuster va a usar.

El archivo `directory-list-2.3-medium.txt` se encontraba en 
![Pasted image 20251117182440.png](../../_assets/49c25de3ade91664b00f2d0ba428e6c3890653f25ceb6a0db5dd59bbdaaa9559.png)

`-t 20`
-t = threads (hilos de ejecución).
Significa cuántas peticiones simultáneas enviará Gobuster.

=>
![Pasted image 20251117181820.png](../../_assets/8612f7f6c1e953d90d7673f554faa0e922d778aee12a04e82cfe41172915af82.png)
 y vemos la el directorio `/admin`
Fuzzeamos ahora con ese directorio para ver si hay mas 

```bash
gobuster dir -u http://192.168.20.134/election/admin -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -t 20
```


![Pasted image 20251218000034.png](../../_assets/aa83cb6746582c30cdda4b72729db255c10cda2ecc3267b3b6b1ab1a10077859.png)
Vemos que hay el directorio `/logs`.
Nos dirigimos donde ello
![Pasted image 20251218000206.png](../../_assets/80cae64d1fb1fd79c453ce4a00b5b5290669661afbd85ac9fb06e676ab989b4d.png)

y vemos un archivo `system.log` lo descargamos y lo concatenamos
y nos da esto:
![Pasted image 20251218000349.png](../../_assets/d108a7a7599907e375042fe649ac6ca728f9b2567430afa5870f9384c0bdf496.png)
Nos da un user y contraseña: 
`love:P@$$w0rd@123`

### **Explotación**

Entonces nos conectamos al SSH
![Pasted image 20251218001056.png](../../_assets/685c5533edc4928865d24a0ded6bb35f750c8427ae9d29883d52268e9b898aeb.png)
y buscamos la primera flag de user:
![Pasted image 20251218001756.png](../../_assets/cb3ca8ef8749c0ec4dabe4f53c3f19545c2b51fd1e7b9acc4e623b5ed663eaf4.png)

**election{cd38ac698c0d793a5236d01003f692b0}**

### **Post-Explotación**

Ahora la siguiente flag a hallar seria la de root

Escalada de privilegios:
Vemos que privilegios tenemos

![Pasted image 20251218002332.png](../../_assets/6f31bb5a515912d144f43498644bc097fcdfb36796694b99144694bc998ab8ee.png)
nada interesante

![Pasted image 20251218002305.png](../../_assets/7dcdf762b52c41d86a9a61692405bbe04abc6cc0c355f453ad585cc5bd049028.png)
no tenemos permiso para ver esto

Buscamos los privilegios SUID
```bash
find / -perm -4000 2>/dev/null | grep -v sanp
```

![Pasted image 20251218003103.png](../../_assets/2bd6e8959f5b41fc3fdcf019b29bcdb0602c056e4b565488ebab2c41d9c5cd25.png)

El mas raro es el `/Serv-U/Serv-U`
Y a traves de internet encontramos que apartir de `/Serv-U/Serv-U` podemos realizar una escalada de privilegio.

![Pasted image 20251218003547.png](../../_assets/54449d0fe3c70f153e44eecea7eb40ea05782e921e291170cb9103c831192088.png)

![Pasted image 20251218003656.png](../../_assets/ee5954be04fd5cad4df43d285123a4c697c2e6d8165937cd14f486cfe0cc4c74.png)

Copiamos el script y lo ejecutamos en **GCC**.
![Pasted image 20251218005011.png](../../_assets/614d8b8338320f0a2d8c853b61b7753b085e7e52730c07b8e575de3d72b6599a.png)

![Pasted image 20251218005045.png](../../_assets/37b4cdbba171d7fc051665359eccee496a9f2cfa3ef65c381825c7d1cd031207.png)

![Pasted image 20251218005107.png](../../_assets/688bb0dcbffff54de6af6428b006171a4a2592dcc5c0610f2cfe6619daf18251.png)
![Pasted image 20251218005127.png](../../_assets/debef4fbcc81af172d576c36b58af9f31cb6c82dc9c5b042d75f6dedc3216c7f.png)
![Pasted image 20251218005203.png](../../_assets/44327c8d81cfe76bfb2aae84dc914ca5d6532ebec845fc10ea25efc21cd08aed.png)
Y vamos por la flag:
![Pasted image 20251218005301.png](../../_assets/cd33908d567b2b9cf87055cad7b4ef8ba1b93c137f3d17087a444afb1701ac3e.png)
Flag de root encontrada:

**election{5238feefc4ffe09645d97e9ee49bc3a6}**
