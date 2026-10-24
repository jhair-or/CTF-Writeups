
**IP**:  172.17.0.2
**S.O**: Linux
**Source:** DockeLabs
**Level:** Fácil
**Date:** 28 diciembre 2025
**Autor**:
**Descripción**: 

Web Enumeration
SQLI (SQL Injection), SQLMap
Steganography challenge steguide, Fuerza bruta con stegseek
Cracking ZIP File (Extracción de Hashes, Fuerza bruta)
[[Escalada de Privilegios#**Escalada de Privilegios mediante SUID**]] (find)

**Flags**:  


**Begin**:

---

Comenzamos haciendo un ping:
![Pasted image 20260113185750.png](../../_assets/4a35141b1da8571d815979fb9777ec911b011ae0b2183fef1c6f309319b350b1.png)
Hay conexión 
TTL: 64 -> Linux

Escaneo de Puertos abiertos TCP
![Pasted image 20260113185756.png](../../_assets/b6af11505510853fdd41af74195aff9f646a51ec4e7a871934fc9da5dca33ce5.png)
Puertos abiertos: 
Puerto 22 (SSH)
Puerto 80 (HTTP)

Accedemos a la servicio web 
![Pasted image 20260113185805.png](../../_assets/d3dd7f8ea6f062dd48a7b9b115c4f1a2152b317e1c93aa12f6e80746f72dabe8.png)
Y al acceder lo que encontramos es un formulario de login simple

Probamos si es vulnerable a algún sqli con un payload simple
![Pasted image 20260113185813.png](../../_assets/9843931f7993177e84b1b02d62462981950cf68b700b3f51076cde93d3d8e478.png)

Y resulta que si es vulnerable a sqli, y no redirigio a un apartado para consultar el clima
![Pasted image 20260113185820.png](../../_assets/48cbe8b12608c0d4e52ec0ebb6839b1bacc960eecba80681febd05293c631256.png)

Procedemos a analizar con SQLMap

```bash
sqlmap -u http://172.17.0.2 --forms --batch
```
**`--forms`**: Le dice a sqlmap que escanee la página web, busque cualquier formulario HTML
**`--batch`**: Significa "modo por lotes". Hace que sqlmap corra de forma 100% automática

![Pasted image 20260113185846.png](../../_assets/b9b175e734a26779072eaf95430217cf8ed1596f6a9cad064f77ff02f67fed80.png)
Descubrió 3 tipos diferentes de Inyección SQL

```bash
sqlmap -u http://172.17.0.2 --forms --batch --dbs
```
**`--dbs`**:  Le ordena a sqlmap que, utilizando la vulnerabilidad que ya encontró en el campo `username`, nos de el nombre de todas la data bases

![Pasted image 20260113185903.png](../../_assets/73f5c4b720e2fe6a5737164fb538ada622547783e3bc0d09e5272a9a34a5d06f.png)
Y nos indico 2 data bases


```bash
sqlmap -u http://172.17.0.2 --forms --batch -D users --tables
```
Ahora que nos de las tablas de la DB `users`

![Pasted image 20260113185921.png](../../_assets/4211d67493a2183fafecea3a047445aa5eba5cdc2898d153c9a0b7060f22e322.png)
Y hay una tabla llamada `usuarios`

Dumpeamos, extracción de los datos

```bash
sqlmap -u http://172.17.0.2 --forms --batch -D users --tables -T usuarios --dump
```
**`--dump`**: Le pide a la herramienta que extraiga todas las columnas y filas de esa tabla y te las muestre.

![Pasted image 20260113185940.png](../../_assets/a72c589ce1e666127aa2708d23af2c9be1c443f69c6ed3f1e8dac9fae3809320.png)
Y nos dan 4 usuarios y 4 contraseñas, pero ninguno sirve, pero hay uno que se llama `directoriotravieso:directorio`

Nos dirigimos a aquel directorio
![Pasted image 20260113185950.png](../../_assets/5d55d5f3faab00e1396550e3e3a2e0e3dac8a01c0bb92f0a660a47b52200b124.png)
Y vemos una imagen `.jpg`

Lo extraemos con `steguide`
![Pasted image 20260113185957.png](../../_assets/e69df5869a4d4e381b451f5843eb5a5e6a7a2c340fa2448328b50921c4c8cf05.png)
Pero nos piden una passphrase

Realizamos fuerza bruta con `sregseek`
![Pasted image 20260113190004.png](../../_assets/c2b866ad1fdae87cc549e2a3a04cf46623863188079a8e2a48571982a463b3f9.png)
Y encontramos la clave y el archivo oculto

Extraemos el archivo oculto
![Pasted image 20260113190017.png](../../_assets/e898eaaa71c73a4890a51ab7450898f7691167318479f8065f2fd8f34dfbc3e7.png)

Nos dan un archivo `.zip`
![Pasted image 20260113190024.png](../../_assets/aed5344324632df1408a93d796536513b7137d38837d5a446a9d18cf9a62fa00.png)

Lo descomprimimos
![Pasted image 20260113190030.png](../../_assets/0bb9cd8b9e04691c0c2d13fc30a5742336b831ce69c3fa69565d2526b3de0226.png)
Pero al descomprimirlo nos pide una contraseña

El archivo comprimido lo pasamos a hash con John
**`zip2john`**: Lee el archivo ZIP cifrado y extrae ese hash.
![Pasted image 20260113190038.png](../../_assets/4087ac3f46d7545159624e986b33f8699e60fbcf24795d18c256301ffe129a18.png)

Ya con el hash, realizamos la fuerza bruta
![Pasted image 20260113190045.png](../../_assets/c5b682ddddd02a2c67503b7a02d5bc8ab1b2e57e2d3e434e8ad1626a2bab8d7d.png)

Y logramos encontrar la contraseña
![Pasted image 20260113190051.png](../../_assets/0e6091e32d4c9a15880897f35ba9567357d34a0287d2d7c380073d158d7dd6c4.png)

Descomprimimos y colocamos la contraseña
![Pasted image 20260113190604.png](../../_assets/f49f3c0425a5fa914119a756eb2eba32b6930349d3e9b06472ff7d9ba87946ac.png)
Nos da una archivo `.txt`

Leemos el `secret.txt`
![Pasted image 20260113190612.png](../../_assets/b9dfcb7191e1df34437bbf709f0e89264e03fe4d20fd5a2116c96f4d1d5dfeb0.png)
Y nos da una credenciales

Aquella credenciales lo probamos con el servicio de ssh
![Pasted image 20260113190619.png](../../_assets/69736b0b2e7cdfabbe130f763c014194421ce95fbcc93fe96424e29f1951187e.png)
Y logramos acceder como el usuario `calos`

Buscamos binarios con permisos SUID
![Pasted image 20260113190626.png](../../_assets/050d5472d7e727b85a3795af864f2c46f59fd1df79bb0adc71bc21f03344e271.png)
Y vemos que el binario `find` tiene permisos SUID

Con el comando encontrado en GTFOBins, ejecutamos en la terminal 
![Pasted image 20260113190631.png](../../_assets/e6104c969cb9e14a00f4b0a5713d1b9cbeffaf2661bf114ec90b51150d48ba2a.png)
Y logramos escalar exitosamente a root.
