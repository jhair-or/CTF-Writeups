
**IP**:  172.17.0.2
**S.O**: Linux
**Source:** DockeLabs
**Level:** Medio
**Date:** 8 febrero 2026
**Autor**: El Pingüino de Mario
**Descripción**: 

Enumeración Web
[[SMB]] Enumeration smbclient && smbmap && enum4linux
SMB brute force with nxc 
Remote Code Execution (RCE) vía File Upload
[[Escalada de Privilegios#**Escalada de Privilegios mediante SUID**#2. Vectores de Explotación SUID]] - nano

**Flags**:  


**Begin**:

---
### **Reconocimiento**:

Comenzamos haciendo un ping:
![Pasted image 20260208203231.png](../../_assets/21523cd3e8e0cb5046d5eec371d3c8c9387226a01ba95bd5c2003bbe02b898d9.png)
Hay conexión 
TTL: 64 -> Linux

Escaneo de Puertos abiertos TCP
![Pasted image 20260208203341.png](../../_assets/1ac8eac280412f125ec7165ddaee587e7f561661ed111f233b160f7a7d1e4bdf.png)
Puertos abiertos: 
Puertos 80 (HTTP)
Puertos 139 
Puertos 445 (SMB)

Escaneo mas a profundo al puertos específico 80, 139, 445.
![Pasted image 20260208203352.png](../../_assets/9d7e978bd8b7a61739d99296236f0ad79d8799052111f17a382694bef9433d5c.png)

### **Enumeración**

Accedemos a la pagina web
![Pasted image 20260208203429.png](../../_assets/a9776c5f935f99c42898c094932473346f43c4d9d5508e8232476bba4709b751.png)

Enumeración al servicio SMB
usamos el comando `enum4linux` para obtener alguna información del servicio SMB.
```bash
enum4linux -a 172.17.0.2
```
![Pasted image 20260208213839.png](../../_assets/25fa095087d1299bfa5593f8e5471222a378fd3aca1514517f190195ae54443f.png)
Obtenemos dos usuarios: `bob` y `james`.

Ahora usaremos `smbmap` para mirar qué carpetas existen.
![Pasted image 20260208213552.png](../../_assets/669c13da994efe52c8379a038ed8586dd10e3b3210d302797990e69941e873e7.png)
Vemos que hay tres recursos compartidos a los cuales no tenemos acceso.

### **Explotación**

Hacemos un ataque de fuerza bruta con `nxc` para los usuarios encontrados

![Pasted image 20260208214408.png](../../_assets/56b09f14ced2c17d15f39dc73596cf92ced17ea29500ff52c6396add1dd139bb.png)
Y logramos encontrar las credenciales de `bob`.

Procederemos a ver qué permisos tenemos con el usuario **bob**.
![Pasted image 20260208214516.png](../../_assets/7244735ddc444033b83bc61fa81daa92a4b89f6d4edf552b81277da8c9a40d7f.png)
Y vemos que tenemos permiso de escritura en `html`.


Abrimos una sesión interactiva en la carpeta `html`.
![Pasted image 20260208214620.png](../../_assets/7f6a440133221006c54b1c1ea29b527549167a52351d2436b14c2b0fc97a7ed8.png)

Subimos un archivo malicioso `.php` para obtener una Reverse Shell
![Pasted image 20260208215547.png](../../_assets/2cc5d582afbebf2c94a60c6c6c1f4c72a0ac476d75246ecc71f4eaee01464e6c.png)

Ponemos el puerto `1234` en escucha..
```bash
nc -nvlp 1234
```

Nos dirigimos a:
`http://172.17.0.2/shell.php`

Y logramos obtener la **shell** en nuestra terminal como el usuario `www-data`
![Pasted image 20260208215828.png](../../_assets/adae2c6ca5d1a6a5caeeeeb3c688e690692687ff7d75357aaf09f3adaf5e971d.png)

### **Post-Explotación**

Intentamos acceder como el usuario `bob` con la contraseña que hemos encontrado.
![Pasted image 20260208215925.png](../../_assets/bdbad4196cb97075311f432c3d7ac84c978ac7cb94c731a02efe62cd217570f0.png)
Logramos acceder como m`bob`

Buscamos binarios con permisos SUID
![Pasted image 20260208220044.png](../../_assets/58a97828f43c1e8d60ee6b0a741f8fa0d720f76dc786dd57dabe31ee89a500bb.png)
Vemos que `/usr/bin/nano` dispone de dichos permisos

Editamos con `nano` el archivo `/etc/passwd` y eliminamos la contraseña del usuario **root** 
```bash
nano /etc/passwd
```
![Pasted image 20260208220113.png](../../_assets/17cabffd800a32bd61615bbc97c38e6f7fa918c1787255ee7084926e33b7a0e8.png)

Y accedemos como **root** y no nos pedira ninguna contraseña.
![Pasted image 20260208220205.png](../../_assets/505c0fa5b25ee9757cb108eb076f2112f2ba0f7b92e14eb978a745fb702539e0.png)

