![Pasted image 20251027202112.png](../../_assets/61a19c9d493603cfa105ce227ec13f99c9300c26d62d9bd6ac1d515ec3d1aa9e.png)

***Jhair***
**IP**: 192.168.20.128
**Source:** VulnHub
**Level:** 
**Date:** 2 noviembre  2025
**Autor**:
**descripcion**: 

Reconfiguring machine interfaces for correct IP assignment via dhcp [Small bypass to circumvent the password]
Abusing qdPM 9.2 - Password Exposure (Unauthenticated)
Remote connection to the MYSQL service and obtaining user credentials
SSH brute force with Hydra
Abusing relative paths in a SUID binary - Path Hijacking [Privilege Escalation]
**Flags**: 

ICA{Next Generation Self Renewable Genetics}
ICA{Secret_Project}

**Begin**:

---

### **Recognition**

We star by recognizing which operating systems are on that network.
```ruby
nmap -O 192.168.20.0/24
```
![Pasted image 20251027204326.png](../../_assets/866bc4ebafe435e3fedd630d8fa57dad7c104905a5274ddc9ce056bbaae1d632.png)

```ruby
sudo arp-scan -I eth0 --localnet
```

![Pasted image 20251027205346.png](../../_assets/5876380ff8f5d4966dc1017bbb36c862b05624e5a190318fb961403c8098fc09.png)
Apparently we dont see any other ping connected.


It seems to be a vulnhub connectivity issue 
We fixed it by mocking the credentinals by giving it "E" at the time it starts.
And we configured its connectivity well
![Pasted image 20251027205948.png](../../_assets/20fc65cb407f9a51d6cb9d00d2e05e9cce6e0f843009e27f8bdb2e749aed49eb.png)
![Pasted image 20251027211537.png](../../_assets/eb5c6dc07ea0093e478247756b70e4d05432d7974322d9f933c844774144fabc.png)

We configure the interface correctly and that it 

![Pasted image 20251028225138.png](../../_assets/1a5e97ead2fe58ca6bdc72d964baab482dea9c746da0e357c708fb991386a9cb.png)
![Pasted image 20251028225151.png](../../_assets/d95678e42f1e171002f2c9fa5c94437804e97c79bc9064c841b469af02e9b877.png)


And as we see, an IP address appears and since it is 00:0c, it is of VMware origin.

![Pasted image 20251027212557.png](../../_assets/b5579cedb0c80bbd9b34b58d9e98e335ecc1481b21e20729687a2d7c140802e4.png)
![Pasted image 20251028193510.png](../../_assets/960f5f288c220f4b9373863ece9809a0878352d32877a9e62df2cf73e07eb1df.png)
and you do a mkt
mkt es una función de tu shell que crea cuatro carpetas en el directorio donde la ejecutes.
Sirve para crear rápidamente la estructura de directorios que usarías en un análisis o prueba de penetración:

![Pasted image 20251028195058.png](../../_assets/470014df5451137a22a9f5cbff9626b6bd6bf5c0e172a339cfb8b3a848d044b2.png)
We enter the nmap folder and and perform the recognition
![Pasted image 20251028195420.png](../../_assets/6e040b0ffdb383a36ba9bba67250929048bb08d6c0b7e747a8a93af302fbcccc.png)
Ping:
```
192.168.20.133
```
![Pasted image 20251028200532.png](../../_assets/bc47fa9f56e06c7e5e0abb11476ea62240f1d3833ee9d6fa954bf70dfb7a4300.png)
There is connection
TTL -> Linux
![Pasted image 20251028200843.png](../../_assets/90c3838e812872abcd8935e8c2231069ac09424cfc2ff08f50ab38b69c3d7119.png)
We can also se what S.O. is with this python script

**Port scanning**

```ruby
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.20.133 -oG allPorts
```
![Pasted image 20251028225832.png](../../_assets/84b14ac3905d8b556073ddb48f5c7d54bcbcb16b84dd2ca5e239ed92c5381dd8.png)


We use extractPorts, a shell function to see it better
![Pasted image 20251028203435.png](../../_assets/0df63a759a270fa25978bcd0508b9f5b47231d70043f991bd1656a4e706327e0.png)
![Pasted image 20251028230011.png](../../_assets/eb7924594fc86b3c889ddbf795b3e3c357641a3e8ed5379f7499582626c0720d.png)
Ports: 22, 80, 3306, 33060
Now we detect the version and service of the ports 
```ruby
nmap -sCV -p22,80,3306,33060 192.168.20.133 -oN targeted
```
![Pasted image 20251028230502.png](../../_assets/df6ec0e594538f2ceceb4c110bde5d31302e4d4d4e56f6b8ff5a11f1dfecfbf5.png)

### **Enumeración**

We scam the web

We take a look at the web service with the "whatweb" tool

![Pasted image 20251028230622.png](../../_assets/a9232ebd37fa099a6d2f1730ea65074bf2843b76e0e10dd282b6957be43d467a.png)

Older versions are often vuulnerable to 
==-> XSS | Prototype Pollution atack==

![Pasted image 20251028230954.png](../../_assets/4284e1ef1964813f66435f22da002fb66947c78d5d46fb8d6a7c3f47271c26bd.png)
We indentify if the qdPM version "9.2" has any  vulnerabilities
con:
```ruby
searchsploit qdPM 9.2
```
searchsploit: es la herramienta de búsqueda local de Exploit-DB. Busca en la copia local de la base de datos de exploits/POCs instalada en tu máquina.

![Pasted image 20251030002825.png](../../_assets/5063add24ac5fd56df844d61581b49d7ab1525ee1be29250d0a2dfc89e2d8415.png)

Password Exposure, hay una vulnerabilidad que expone contraseñas. (Unauthenticated), que no se nesesita iniciar sesión.
We perform a searchsploit on your path.
![Pasted image 20251030003449.png](../../_assets/22bf25763b5a4b88fc5bb05e5c012c9cdd7b7824925f3236a97b3f913689fca6.png)
![Pasted image 20251030003524.png](../../_assets/d8e392e9cd9a6ab7a8d9ea81215b2d6d437b36fb6aca58150f500a7c443fd058.png)

Here they indicate that the password and the database connection string are stored in
```
http://<website>/core/config/databases.yml
```

![Pasted image 20251030004557.png](../../_assets/5a3c2709420e1f7552a2e7dee49e099798eb383c9db12aca050d264e74a9fd7a.png)
![Pasted image 20251030004605.png](../../_assets/e5ee65c1b44fd6323c96527868435625d19b4d287417ec3bf138244ac8cb074d.png)

#### **Explotación**

We tried to connect via SSH
![Pasted image 20251030125609.png](../../_assets/953024974012c5cba7f38ef9ea718a65b47409f6e534ad11b6f14ae22507d0fa.png)

We tried to connect via mysql

![Pasted image 20251030131608.png](../../_assets/c25f223ce5601fae0d3e5f1e9cd8b0733238db257a5e990400cf23fd5facd04c.png)
Este mensaje indica que tu cliente MySQL intentó establecer una conexión cifrada (TLS/SSL) con el servidor, pero:

* El servidor MySQL estaba configurado para usar SSL/TLS (transport layer security).
* El certificado del servidor era autofirmado (no validado por una autoridad de confianza).
* El cliente MySQL trató de verificar ese certificado, no lo reconoció como válido y rechazó la conexión

In this case:
- El cliente intentó conectarse → MySQL negoció un canal TLS.
- MySQL envió su certificado autofirmado.
- El cliente lo comprobó → no encontró una CA válida → error 2026.

To solve this case, we use:
`--ssl-mode=DISABLED` o `--skip-ssl` → le dice al cliente **“no uses SSL”**, con lo cual ya no intenta validar ningún certificado, y la conexión se hace en texto plano (sin cifrar).
También se podría haber usado `--ssl-mode=PREFERRED`, pero la versión no lo soportaba yaque era una versión antigua

```ruby
mysql --skip-ssl -h 192.168.20.133 -P 3306 -u qdpmadmin -p
```

We cannot connect to a data base
![Pasted image 20251030132017.png](../../_assets/f51c3be4c21705f705c92c5cde7ee26c08c547c8c14c53c9932dd942061d41c1.png)
We see the exposed databases
![Pasted image 20251102181039.png](../../_assets/74793582fac8f41fce0086f9683c4320ba200ecf6d0ab2040b5af7b06af46630.png)

We use the **qdpm** database
![Pasted image 20251102181227.png](../../_assets/0a726837cbbddc97cdade26b02e7a9840bbff002e92bbef65489e87e378ab841.png)
show tables
![Pasted image 20251102181246.png](../../_assets/9433e0d5b28db0f1fb639a1026de3fead3a473bc80127f482f33347595ca75d5.png)
table users
![Pasted image 20251102185416.png](../../_assets/60796c6f9c17b7d7c2cb40d7e3a9cfb3e6e07b8d82875b1ac30db801f72d9f19.png)
Staff
![Pasted image 20251102185441.png](../../_assets/67713aa13a22024da7802ca9d42fb7153ceaf9cf9c3f28ce452d71d5d4a4d34f.png)
![Pasted image 20251102185515.png](../../_assets/5332babe1efabeccda218d9ccf285682cc85e76335cda86d5e4965a036a37748.png)
![Pasted image 20251102185548.png](../../_assets/93f48190673bf692058883cd47acaf1146cef5687cc1f54b546148bf5b3e105f.png)
Here wen can see users and roles
In login:
![Pasted image 20251102185935.png](../../_assets/d34501f081733a84f044db43109ded5c2c434404c2281744c9b581436dcfa715.png)
We see that the passwords have been encrypted in base64
Decrypting:
![Pasted image 20251102190332.png](../../_assets/c3c2cefdb26e0d2a7f67b37c583e57b3c39084add25d029e257d0cb8278c4768.png)
![Pasted image 20251102191225.png](../../_assets/305d93345946515b505df3ad5b47ef8de2f75b4f9fe234f00465ffa8f357e0cd.png)
We saved it 
![Pasted image 20251102192654.png](../../_assets/b2e200dc9ab8933d4044477d4e66384f87cbd9b4c6b360b588fbc30184345570.png)
Users also save it in content
![Pasted image 20251102200354.png](../../_assets/241f1271b384e8bd03dd9693d259f42c814f8c9f7d1383e318483cd3c14155ae.png)
![Pasted image 20251102192738.png](../../_assets/1ce44e5d8cea4cc9df19e4f447d3068043e56f3ce3899e4f0ed3f7a729e22f44.png)
![Pasted image 20251102192827.png](../../_assets/c31ef1dc23726b8437a0b8b87a31168aaa285e9efca444f9faabcce10e608da9.png)
Since we have a list of users and a list of passwords, we will proceed to use hydra
**Hydra**:
permite probar listas de usuarios y contraseñas rápidamente contra uno o varios hosts.
- Intenta combinaciones usuario/contraseña de forma automática contra un servicio.
- Soporta muchos protocolos y modos (incluye ataque por formulario web).
- Permite usar listas de usuarios y/o contraseñas (wordlists).
- Tiene opciones para controlar concurrencia, SSL, salida a fichero, detenerse al encontrar credenciales, etc.
```ruby
hydra -L users -P passwords ssh://192.168.20.133
```
`-L users.txt` → archivo con usuarios (uno por línea).
`-P passwords.txt` → archivo con contraseñas.
`ssh://192.168.20.133 → protocolo y objetivo.
![Pasted image 20251102195346.png](../../_assets/e8856328a464e034be3b657fbd38eee5f73733047a23b3955715e134cd22ad52.png)
There are two users with matching passwords.
We connect
**For travis:**
![Pasted image 20251102200737.png](../../_assets/68890df17eff08587ee481ffebd9e4cd3be420fb9c71e47d5b44e36debe073cc.png)
![Pasted image 20251102202645.png](../../_assets/da4fc57da5b9952264cc9e8ea634332ea64b43133f9624a48d3ae31a40bb32ad.png)
First flag found
**ICA{Secret_Project}**

### **Post-Explotación**

We enter the user dexter
![Pasted image 20251102202835.png](../../_assets/c343b44140637a4571bbc2cd8ed33aa0a6ba9b3a1f75fafcc294d0851774b6e3.png)
inisde it we have a note
![Pasted image 20251102203030.png](../../_assets/e9b087cdc69bca558daf5c325dad7115c0031a92037fc6f61dada507c586a2e9.png)
Tell us this
![Pasted image 20251102203232.png](../../_assets/0e1250be4eab45c90f804c26227c3eb08358c52754c724b7ed2bb44658f997ea.png)
The text gives us the executables files as a clue
Then we look for binaries with SUID whose owners are root
```ruby
find \-perm -4000 -user root 2>/dev/null
```
**`find`**  Es la utilidad de Linux para buscar ficheros y directorios recursivamente a partir del directorio actual
`\-perm -4000`:
* `-perm` es la prueba de permisos de `find`.
- `4000` es un número octal que representa el **bit SUID (set-user-ID)**. En octal: `4000` = setuid, `2000` = setgid, `1000` = sticky.
- El guion delante (`-4000`) le dice a `find` “todos los bits indicados están _presentes_” (es decir, el archivo tiene el bit SUID activo).
- El `\` delante del `-` en tu ejemplo (`\-perm`) es innecesario en la mayoría de shells; quizá lo puso alguien para evitar que `-perm` se interprete como opción del shell, pero `find -perm` está bien sin escape.
**`-user root`**  Filtra para que sólo muestre archivos cuyo propietario (dueño) sea `root`.
**`2>/dev/null`**  Redirige la salida de error estándar (stderr) al dispositivo nulo. Esto oculta mensajes de error como **Permission denied** que aparecen cuando `find` intenta entrar en directorios sin permisos. Así la salida sólo muestra los ficheros encontrados, sin errores molestos.
![Pasted image 20251102210035.png](../../_assets/dd47ee316370c2937c141f9e83fe0d41b87d9ca2dda22f5e2606b82ec9d90c6c.png)
We see that there is a strange one:  `./opt/get_access`
We see your privilege
![Pasted image 20251102210521.png](../../_assets/7ac4d14bc2fc3015be11f6bcbb1ee6adf4ce7f0456d38d81e48a5a098de4ac18.png)
![Pasted image 20251102210620.png](../../_assets/94182021d2ec1c4790ac54d4394eef98ba32cec081e22d33534eaaa5848190a3.png)
we see that it is a compiled linux binary
![Pasted image 20251102210816.png](../../_assets/62c1ac7ab9fb8bbb0e7f6eca405f4fd5a2f693a31b3f30871d3bcb72aa42a8a6.png)
we use the string command to the executable
Al ejecutar strings te mostrara:
- Mensajes de error y `printf`/`fprintf` que el programa usa.
- Rutas de archivos embebidas (p. ej. `/etc/config`) y nombres de recursos.
- Versiones de librerías, cadenas de licencia, URLs o keys en texto plano (si el autor las dejó ahí).
- Nombres de símbolos si el binario no está totalmente "stripped".
- Texto de archivos de scripts (si aplicas `strings` a un script verás el código legible), de forma legible, etc.
![Pasted image 20251102212918.png](../../_assets/b82db35cb8649493e0edf7d120e57dea7ea0b519999c51c5b0b87c5caf77e44b.png)
We see that we have root permission to execute cat on system.info
![Pasted image 20251102213432.png](../../_assets/6acaa7bc0b00a1d7c54ebb0443057b86d19dbc353c659161975d2abcf2439d4b.png)
We see that in the executable it uses `cat`and not `user/bin/cat` so we can hijack the `cat`binary.

gonig to the /tmp/ directory
**tmp**:contiene archivos temporales creados por aplicaciones y el sistema operativo. Estos archivos son para uso provisional y suelen eliminarse automáticamente al reiniciar el sistema o cuando el programa ya no los necesita.
![Pasted image 20251102214146.png](../../_assets/b945c5f803582b00d21804db50d455a13c15f0ed1cde427fcae84f5d869a5e1f.png)
We create our own cat and give it execution permissions
![Pasted image 20251102214318.png](../../_assets/7896b602207d84d746eb8bcbf0edb75047e3c7f6803c1ff97f61348439fa16e2.png)
```ruby
echo $PATH
```
Este comando  sirve para **mostrar las rutas donde Linux busca los programas ejecutables** cuando escribes un comando en la terminal.

Esto hara que comience buscando primero por el directorio /tmp
```ruby
export PATH=/tmp:$PATH
```
![Pasted image 20251102215118.png](../../_assets/afdc586d7e4d065c88aea571c8d9af1155df365e51d547dec23411992c1cdfb1.png)
ahora alteramos el contenido de cat para que asigne privilegios suid a la bash y no solo muestre el system.info
![Pasted image 20251105111836.png](../../_assets/41493eb3726896d9b3fb5dba9eed0c39b3180d83a6536d085e058f59f97b5058.png)
![Pasted image 20251105111819.png](../../_assets/bfeb51baf7b74332261939c700afab6dccb2d60b18d66375d58f2ac5e46310bc.png)

![Pasted image 20251106000417.png](../../_assets/e2deea69a5c4135677271a9ea1e85442f1a12230e708683d9ccbdb5226a7d23c.png)
Tenemos root en la bash

Una vez dado esto, volvemos el path a su normalidad para que el cat ya no sigua secuestrado en /tmp 
![Pasted image 20251105112402.png](../../_assets/27afd71dcc725c9a82145fd06f95810f5d26f14c75b6440af89924558e096d17.png)
Y entramos al modo privilegiado del bash.
```ruby
bash -p
```
![Pasted image 20251105112810.png](../../_assets/3b21e257ab3129c18d242b6b76af5122016f1f1a03987ec3af7acfcfcee81686.png)
![Pasted image 20251106000649.png](../../_assets/49c51c569d3df726fe8400e58309aa5e155348dff22a34250654a757ffbf8aa7.png)

Entramos al directorio de root
![Pasted image 20251106000731.png](../../_assets/59458d7500b1933e28d1a5317d88469c0bcc7e5978abb18ebe16cf8ca083507a.png)
abrimos el .txt
![Pasted image 20251106000743.png](../../_assets/8bb3d1710384778bfa69b697e66e8990ae0c75bad697f15647b4d0e4827719ab.png)
Bandera encontrada

**ICA{Next Generation Self Renewable Genetics}**

