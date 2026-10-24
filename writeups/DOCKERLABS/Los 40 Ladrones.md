
**IP**:  172.17.0.2
**S.O**: Linux
**Source:** DockeLabs
**Level:** Muy Fácil
**Date:** 18 diciembre 2025
**Autor**:
**Descripción**: 

Fuzzing with gobuster 
Information Leakage
Port Knocking
SSH Brute Force - Hydra

**Flags**:  


**Begin**:

---

### **Reconocimiento**:

Comenzamos haciendo un ping:
![Pasted image 20260106195928.png](../../_assets/6071a33967bf44178fe1426e1e15af9225099c8a47cba32c64c150aa1d7f4091.png)
Hay conexión 
TTL: 64 -> Linux

Escaneo de Puertos abiertos TCP
![Pasted image 20260106195858.png](../../_assets/7352042ac4075a3816d4b59b66da6d2e6ac5d50d92d5e01e558fc6deb1335a83.png)
Puertos abiertos: 
Puerto 80 (HTTP)


Escaneo de versión a los puertos 80.
![Pasted image 20260106195951.png](../../_assets/f8e84b865b31e48e0da15926051607449fe92c0342670da08ceb229e81a63c33.png)

El servidor respondió con la página por defecto de Apache2
![Pasted image 20260106200044.png](../../_assets/b8510ae305d782af6899c34b089fd89814fd4aa248dca8408c51a2a772ac2cda.png)

Realizamos un fuzzing de directorios.
![Pasted image 20260108134732.png](../../_assets/e5056fca88d222c6dc64ee1c50b508a835dce749ee6a80ea5317d12337b819ee.png)

Y al dirigirnos al directorio `/qdefense.txt` aparece un mensaje
![Pasted image 20260108134756.png](../../_assets/e21e7a6682a2e035184ac997103c6d8a48a1dd0282a2f71a38506d5817d35338.png)
Que analizandolo hacer referencia a Port Knocking

Port Knocking es una técnica de seguridad en redes que sirve para ocultar servicios (como SSH) detrás de un firewall y abrirlos solo cuando el cliente correcto realiza una secuencia específica de intentos de conexión a puertos.

Entonces golpeamos los puertos **7000, 8000 y 9000** en ese orden exacto. El objetivo de esto suele ser decirle al firewall de la máquina `172.17.0.2`: _"Soy yo, tengo la clave, ábreme el puerto de acceso (el SSH)"_. Luego hacemos un escaneo con nmap
![Pasted image 20260108161121.png](../../_assets/2f8145baaedecee651f8444660a6c67d0cf046e1a29078cf3997ee1474d954b6.png)

Luego en el mensaje anterior se menciona a alguien llamado toctoc, probamos hacer fuerza bruta para ese usuario
![Pasted image 20260108161531.png](../../_assets/1f7bc96c7eb3e82dcfa927538811717ce78e918d11a4f4ded5d1aace7c0ed4b5.png)
Y conseguimos su contraseña por medio de fuerza bruta con hydra

Accedemos como el usuario toctoc
![Pasted image 20260108162014.png](../../_assets/1791e7fa777a6d353571613183d43bbe4de2a2e125d030e93272ad31d6695d8e.png)

Listamos los permisos sudo
![Pasted image 20260108162650.png](../../_assets/c1306916048a75648346c886c737975367dfd99947e0cc8f3118cda6628175be.png)
Y vemos que como cualquier usuario podemos ejecutar `/opt/bash`

Lo ejecutamos con sudo 
![Pasted image 20260108162754.png](../../_assets/5b96ea06885d3a9ec895fc63641f620c849e57c01a0f3db925d183cc19373f38.png)
Y logramos escalar a root.


