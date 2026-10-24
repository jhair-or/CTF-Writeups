
**IP**:  10.129.1.226
**S.O**: Linux
**Source:** HackTheBox
**Level:** Fácil
**Autor**: 
**Descripción**: 

**Flags**:  

Fuzzing de Directorios Web (Web Directory Enumeration)
Insecure Direct Object Reference (IDOR)
Análisis de Tráfico de Red (Network Traffic Analysis) - Wireshark
Information Leakage
Reutilización de Credenciales (Credential Reuse)
[[Escalada de Privilegios]] - Linux Capabilities(Enumeration, Abuso, OS Command Execution)

**Begin**:

---
### **Reconocimiento**:

Comenzamos haciendo un ping:
![Pasted image 20260218224756.png](../../_assets/7936757c31647ab7427f25c6d51b7e10b073e7f2c13fba36ea2e1f9d56e15864.png)
Hay conexión 
TTL: 64 -> Linux

Escaneo de Puertos abiertos TCP
![Pasted image 20260218224905.png](../../_assets/76d0cfb5a69fd835880542efd4814e098a23c1428925ac2faeeb1eccef956b86.png)
Puertos abiertos:
Puertos 21 (FTP)
Puertos 22 (SSH)
Puertos 80 (HTTP)

Escaneo de versión a los puertos específico 21,22,80
![Pasted image 20260218224954.png](../../_assets/a8350ab1fc669e2a99df7d9e379e834b5976cfe57cc490e959a40790400dbeb2.png)

### **Enumeración:**

Nos dirigimos al servicio web
![Pasted image 20260218225109.png](../../_assets/b082d94eff19d8cbd74d2dd1bc8ee781ce6431223cf302d36bda275d859c4572.png)
Y solo vemos un dashboard

Enumeramos el servicio FTP
![Pasted image 20260218225156.png](../../_assets/23da0cfb84b37749b6ee0539ab9bfe9d457913cd73cf2a7b3edc0b899e060010.png)
Pero nos piden credenciales para poder ver los recursos

Entonces lo que hicimos por consiguiente es hacer un fuzzing de directorios.
![Pasted image 20260218230521.png](../../_assets/2724d9dfd10248567de0227f7f6324e61a0d063c77d3263e8110e02a6885890e.png)
Pero no encontramos nada nuevo

Entonces regresamos al servicio web a analizarlo a mas profundidad
![Pasted image 20260219225531.png](../../_assets/428eb2ecdd5147cd37e8763df9f4d686d672104e6c54fc70455a4e724355f2a7.png)

Y al acceder a la parte de `Security Snapshot (5Second PCAP + Analysis)` vemos que son capturas de trafico de cada 5 segundos y notamos que en la parte de la url la parte final cambia paralelamente 1 a 2 y si recargamos la pagina cambia de de 2 a 3 y así sucesivamente 
![Pasted image 20260219225834.png](../../_assets/e3236817f43d6d3a5fdac8d2fa9c1f91434d58182051c97e246ce3dcc8117756.png)
Lo cual podria ser un indicio de una vulnerabilidad IDOR

En la parte de `IP config` al acceder nos muestra la direcciones de la interfaces de la maquina
![Pasted image 20260219225919.png](../../_assets/3c16b316ec5698d0d64e6296873024ebd3ef404c317a32272e35c4f575409216.png)
Intentamos en la parte de la url cambiar de un `ip` a un `whoami` para ver si era vulnerable a un OS Command Injection, pero la pagina nos respondio con un `Not Found`, por lo que lo descartamos

Y nos dirigimos a `Security Snapshot (5Second PCAP + Analysis)` para ver si es vulnerable al IDOR
![Pasted image 20260219225946.png](../../_assets/4f6555a518fc6f3f24e2477e837d540247e3ac9b3de2464a040451831de29edd.png)


> [!NOTE] IDOR (Insecure Direct Object Reference)
> Contents

Probando colocar el parametro 0 en la url y descargando el archivo `pcap` para analizar
![Pasted image 20260219225639.png](../../_assets/554328c408687f14f679ec51b676df13587b546ec439cc290ab366194b8767e9.png)
![Pasted image 20260219225716.png](../../_assets/a6362bc44f79aee4cb76804ce2c13d1ba5d4f840cd566ce90a352ba861e21858.png)

Lo arbimos en wireshark
![Pasted image 20260219230211.png](../../_assets/a61be4e744c930ae2cf8fb8e76e07db312a175bd04b39bde1912510d3c9421b8.png)

Lo filtramos por trafico ftp y vemos que el usuario nathan se accedio a este servicio y podemos ver sus credenciales
![Pasted image 20260219230844.png](../../_assets/448cd9586e7e064f7fdce1c6182cc4d81f6eb3c5153fdd2a2476939b34a919be.png)
![Pasted image 20260219230910.png](../../_assets/94de2d40737fda943236db877e5133c9dc41badd21a7a291287df2da8dcd668d.png)
`nathan:Buck3thH4TF0RM3!`

Accedemos al servicio ftp con esas credenciales
![Pasted image 20260219231037.png](../../_assets/e032e11450fecda0d3efc574b6432a0318b5e08f5c4b361a8f78ea547bfb33cf.png)

Listamos los archivos dentro
![Pasted image 20260219231148.png](../../_assets/757df2468d9b9365d30e32501aa8dd168ee260801d5e483bdc9907d2f2c7a04c.png)
Obtenemos la flag de usuario

Y lo descargamos
![Pasted image 20260219231205.png](../../_assets/ecfd61a38657d0a086918f54101286f78e604c3bb0fe15ea9f2f8c12072c5ba5.png)

Luego probamos si con esas mismas credenciales podemos acceder al servicio SSH
```bash
ssh nathan@10.129.3.47
```

Y si, con esas mismas credenciales logramos acceder como nathan al servicio SSH
![Pasted image 20260219231410.png](../../_assets/25e93e2568c3a66da87eefbbea02060846e823d3908c812fa33a43d904e31130.png)

Luego listamos las capacidades de los archivos de `nathan`
Comando para listar las capacidades asignadas a los archivos en Linux
```bash
getcap -r / 2>/dev/null
```
este comando busca en todo el disco duro programas que tengan permisos ocultos (conocidos como Linux Capabilities)

> [!NOTE] Linux Capabilities
> Contents

![Pasted image 20260219233354.png](../../_assets/6c2a74a5d2d470aa02e847858e6547a901b1f4a03b374e4859d6e3bcd3177413.png)
`/usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip`

**El permiso (`cap_setuid`):** Esta capacidad específica le otorga a ese binario el poder exclusivo de cambiar su Identificador de Usuario (UID) a cualquier otro, incluyendo el UID 0 (root).
Y al binario `python3.8` le dieron esta capacidad.

Con python abrimos una nueva consola, y esta nos da privilegios maximos.
![Pasted image 20260219232955.png](../../_assets/45be58a15f7eb0991d8b1f676188ffff1db6db298786b9762f638735b56876b7.png)

Y logramos obtener una shell con privilegios de root.
![Pasted image 20260219234016.png](../../_assets/40f45fad11f5280913adf51e7783ec59606edc25a4c044b0941270e5aa91d968.png)
