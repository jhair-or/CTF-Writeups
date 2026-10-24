
**IP**:  172.17.0.2
**S.O**: Linux
**Source:** DockeLabs
**Level:** Fácil
**Date:** 1 febrero 2026
**Autor**: Zunderrub
**Descripción**: 

Enumeración Web
[[SMB]] Enumeration; smbclient && smbmap
Fuzzing de directorios con gobuster
SMB brute forcé with crackmapexec
[[Escalada de Privilegios#**Escalada de Privilegios mediante Sudo**]] con Python Library Hijacking, 
ls y cat (Information Leakage), Hijacking (secuestro) de script

**Flags**: 


**Begin**:

---

### **Reconocimiento**:

Comenzamos haciendo un ping:
![Pasted image 20260201185509.png](../../_assets/e15d16c68d3373b499ff2f4322959e3590063e0404a5787b3839f1909757cfe3.png)
Hay conexión 
TTL: 64 -> Linux

Escaneo de Puertos TCP abiertos
![Pasted image 20260201185611.png](../../_assets/d789130647c15c351df40852b5e6f847bb44e316cb198ff79a67ac8197aedaf3.png)
Puertos abiertos: 
Puertos 22 (SSH)
Puertos 80 (HTTP)
Puertos 139 
Puertos 445 (SMB)

Escaneo de las versiones de los puertos específicos
![Pasted image 20260201185621.png](../../_assets/6943d1ee9c9a41352d1fa1597cc6d2ca02ba48e36ba99bfdc7eaa3070013ee67.png)


### **Enumeración**

Accedemos a la pagina web
![Pasted image 20260201190512.png](../../_assets/8fd3639b14c5acec69da21e1608199371d689ba7b693934b252e4f3647d72b98.png)

Hacemos un fuzzing de directorios.
![Pasted image 20260201190635.png](../../_assets/146a8fb26cabf18eafbe9408ab21ffc3cb71a96166f96789d37726ba9220b995.png)

Nos dirigimos a `/dragon`
![Pasted image 20260201190657.png](../../_assets/1575b385ef622a197826e383129058c44125db13743754bc8012606f96a709ca.png)
No fijamos en `EpisodioT1`

Y vemos lo que dice
![Pasted image 20260201190716.png](../../_assets/89db60c1cfdd58888fdc00d41398c7fc5e236bd704f27f9007dd303bb794e7b4.png)
Lo que se muestra son varios nombre de episodios nos menciona.

Enumeración al servicio SMB 
```bash
smbmap -H 172.17.0.2
```
![Pasted image 20260201190855.png](../../_assets/c0b724a261627338306ff86098c51d640936ce0225138869359cb03cde69be01.png)
Vemos que no tenemos permiso a ningún `Disk`

Después de investigar bastante.

Lo que hicimos fue enumerar los nombres que estaban en la pagina web
```bash
nano users.txt
```
![Pasted image 20260201190512.png](../../_assets/8fd3639b14c5acec69da21e1608199371d689ba7b693934b252e4f3647d72b98.png)
y colocarlos en una lista

Y los nombres de las contraseñas visto anteriormente colocarlo en una lista de notas de posibles contraseñas
```bash
nano posibles _contraseñas.txt
```
![Pasted image 20260201190716.png](../../_assets/89db60c1cfdd58888fdc00d41398c7fc5e236bd704f27f9007dd303bb794e7b4.png)

### **Explotación**

Y procedemos a hacer un ataque de fuerza bruta con crackmapexec
```bash
crackmapexec smb 172.17.0.2 -u users.txt -p posibles__contraseñas.txt --continue-on-success
```

![Pasted image 20260202130247.png](../../_assets/bad3ab40d38bb605c40205b713568e32a4947b4f99d341fc618cfb69234337e3.png)
Y encontramos las credenciales de `Jon`, las demás credenciales no son validas ya que adminten mas de una contraseña y lo mas probable es que se hayan iniciado como `guest`.
`Jon:seacercaelinvierno `

Accedemos con **smbclient** con el usuario jon y listamos los recursos compartidos
![Pasted image 20260202131257.png](../../_assets/2da488d6346f3f45b8507e8d89b16856a41ef203381bc8a83ec5dd6e565fda64.png)

Nos dirigimos a `shared`
![Pasted image 20260202131333.png](../../_assets/feb22888d031576ff8788b71f6b2ea5cfe99fe65ce88b1cbd06f2a13efc9d153.png)
Vemos un archivo llamado `proteccion_del_reino`

Lo descargamos en nuestra maquina
```bash
get proteccion_del_reino
```

Al leerlo vemos que nos dan una contraseña en B64
![Pasted image 20260202131422.png](../../_assets/c1989673dd4d55b8b7c1ed10b6114b58d4d02236b46a102ad672bc5111b083e9.png)

Lo decodificamos
![Pasted image 20260202131527.png](../../_assets/653dcacfe8d6ca3e5ed76998be72e4391d4f30bcb2159a02d261cc575aa8284c.png)

Accedemos a ssh con el usuario **jon**
![Pasted image 20260202131653.png](../../_assets/97625bf39f8257f8cc691727c10a1ac4ff47e733acbc3250afe1cce3921fc1c6.png)

### **Post-Explotación**

Revisamos los permisos sudo
![Pasted image 20260202131710.png](../../_assets/b2e5c64758b524e1bbcbdbbeff95863aa1b064a1d26b2fb8d67756a3f5ea502e.png)
Vemos que podemos ejecutar un script de python como el usuario **aria**

Leemos el script
![Pasted image 20260202132017.png](../../_assets/53d7387c5c068e872bfed62149d90df2848ec16dd5d708775e5e88ecaaf7e3d0.png)
Vemos que el script usa las librerias: import hashlib , import getpass
Procederemos a hacer un: Python Library Hijacking

Creamos una librería con código malicioso en nuestro directorio actual
![Pasted image 20260202132051.png](../../_assets/55aae45191c1fccd531a6122a4372c8d7937692862d7c722f3db46d73c149fb2.png)

Ejecutamos el script de python
![Pasted image 20260202132119.png](../../_assets/03f3f1457be5b2c3c0d3f5ba55c0a4703f5bb385d5bce39dd6390cdeedb64976.png)

Y logramos escalar al usuario **aria** y listamos los permisos sudo
![Pasted image 20260202132157.png](../../_assets/71e9f82a2d608e1762174241371f1bc2802dd6f14afa51e4384c6788dee25107.png)
Vemos que podemos utilizar `/cat` y `ls` con sudo como el usuario **daenerys** sin necesidad de una contraseña.

Usaremos esos binarios para leer si directorio del usuario **darnery**
![Pasted image 20260202133447.png](../../_assets/caadaa0b713fccbf7ee67ca2efe004995deff0ded86da46402c12f8e3fe86aac.png)
Vemos que hay un archivo llamado `mensajeParaJon`

Procedemos a leerlo
![Pasted image 20260202133740.png](../../_assets/b3fd57b0f20f69a3664940d80d3405d29d6408bc443209ce727df2cb5628bab9.png)
Y obtenemos la contraseña de **daenerys**

Nos logeamos como **daenerys**
![Pasted image 20260202133828.png](../../_assets/ebbd8764b0f4969eee64b272aba0643dc41114225193deb86a3a6756301cefb5.png)
Al listar los permisos sudo, vemos que podemos ejecutar `/.shell.sh` como root sin necesidad de una contraseña.

Reemplazamos el script legítimo por una orden para abrir una terminal como root
![Pasted image 20260202134149.png](../../_assets/f0c40d6e47a39f7e70a869ec7afcfe2709430ed208ac37f7b88184ff45b6a75b.png)

lo ejecutamos y logramos escalar a root.
![Pasted image 20260202134159.png](../../_assets/a19af79b604b7b6bcec5c682ccb9cd4f88f7448ff4f1524cc829ce64e94f6fad.png)
