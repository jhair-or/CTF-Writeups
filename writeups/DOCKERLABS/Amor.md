
**IP**:  172.17.0.2
**S.O**: Linux
**Source:** DockeLabs
**Level:** Fácil
**Date:** 2 Enero 2025
**Autor**: Romabri
**Descripción**: 

Web Enumeration
Fuzzing with gobuster
Information Leakage
SSH Brute Force - Hydra
Enumeración del Sistema de Archivos
Esteganografía con **steghide**
[[Escalada de Privilegios#**Escalada de Privilegios mediante Sudo**]] - ruby

**Flags**:  


**Begin**:

---

### **Reconocimiento**:

Comenzamos haciendo un ping:
![Pasted image 20260113193248.png](../../_assets/98b15047c22ef2ddee1c2ca91c4c1500259bc626a834ace4b9d4903bbe6b6b6d.png)
Hay conexión 
TTL: 64 -> Linux

Escaneamos los puertos
![Pasted image 20260113193255.png](../../_assets/1f55b5192a5789983c987396a84cf550bcd5074a08ae4a2c6c71ab361c97fcaa.png)
Puertos abiertos: 
Puertos 22 (SSH)
Puertos 80 (HTTP)

### **Enumeración**

Accedemos a la pagina web
![Pasted image 20260113193304.png](../../_assets/53002ebb7fb4df8c29a3e4f84c373b9430a201a28f5032b49a92986b2e227d1d.png)
En la pagina vemos al parecer reportes de seguridad

Hacemos un fuzzing de directorios.
![Pasted image 20260113193312.png](../../_assets/57dd0941d0f2569bca9cf7d3310e51fd5421a98422cd493e062217936006cd8e.png)
No encontramos nada interesante.

Investigando mas en la pagina vemos una parte donde dice `firmado por carlota` lo que no dice que puede ser un posible nombre de usuario.
![Pasted image 20260113193319.png](../../_assets/8b03ccb8f322fc6b9b1943779cd32b0cf10292092aadaa42761f9298e654a813.png)

### **Explotación**

Entonces hacemos una fuerza bruta con `hydra` con el usuario `carlota`
![Pasted image 20260113193326.png](../../_assets/22affdaa0b83ef3eb97d3483ee679c57237a4a5d757dba0513caebc4bc61f2ff.png)
Y logramos encontrar las credenciales: `carlota:babygirl`

Accedemos la SSH
![Pasted image 20260113193332.png](../../_assets/4abe09b42db6c4ff6a2d0402b616bd20ad0849bd24f4cce16e436e849f29d474.png)

### **Post-Explotación**

Enumeramos el Sistema de Archivos
![Pasted image 20260113193344.png](../../_assets/3045616ba2f0d4b537884b93ad16dd167acf975f2cbf3e33b276cdbfa018f3f7.png)
y encontramos una imagen 

Nos lo traemos a nuestra maquina atacante con la herramienta `scp`
![Pasted image 20260113193353.png](../../_assets/665d85ac278d0834bc52d11622d860bf9ed4572a2a5961be99ac0b2df7e9c381.png)

Hacemos un escaneo de metadatos con `exiftool`.
![Pasted image 20260113193401.png](../../_assets/c14ccbf4e3c321298b8bcc61f11399d786308a8bf28b0d9a38d3fe934770fa2f.png)
No encontramos nada 

Abrimos la imagen para ver que es
![Pasted image 20260113193417.png](../../_assets/fbd577f8aba25ede9cca0684bc20657afaaa98adfdf1beb8178aa49184bc73e3.png)
Vemos que es una imagen de una pareja, haciendo referencia al nombre de la maquina.

Intentamos extraer información oculta por esteganografía 
El cual nos pide una `passphrase` el cual al darle no ingresar nada y darle enter nos da un archivo llamado `secret.txt`
![Pasted image 20260113193426.png](../../_assets/b5936fe1f5665bb93a98d0046540f96ac7514c35c827e84fe8e781dfc56aa1eb.png)
Lo leemos y parece darnos una cadena de texto codificada en **Base64**.

El cual lo decodificamos
![Pasted image 20260113193432.png](../../_assets/aae208025587cbb968577f70e3fa6cc5e38852c8d562f08317b30b61f0b5fc8e.png)
y nos da: `eslacasadepinypon`

Vemos los usuarios del sistema desde el directorio /home
![Pasted image 20260113193438.png](../../_assets/f25836b07e773e85e4da9fe68640b74a93a222422a8451a6f77e059cad1e991f.png)

Y intentamos acceder al usuario `oscar` con aquella cadena decodificada
![Pasted image 20260113193446.png](../../_assets/abc37b82b84895908ffbdb1fb273735211b92215a82d42478f63e07a200d165c.png)
Y logramos acceder

Listamos los permisos sudo
![Pasted image 20260113193457.png](../../_assets/55d2bcfc76c683749e19089abd37dfd6710fa39cbb3274d8532eb4158b6ce3e1.png)
Y vemos que podemos ejecutar `/ruby` como **root** sin necesidad de una contraseña

Este ejecutable puede generar un shell de sistema interactivo.
Ejecutamos el comando encontrado en GTFOBins
![Pasted image 20260113193515.png](../../_assets/fe18ff9cbd965e618a7b79cc0c677dfde93920359b2f4344e21464a3d9b88f1d.png)
Escalada exitosa a root.

