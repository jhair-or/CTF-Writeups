
**IP**:  172.17.0.3
**S.O:** Linux
**Source:** DockeLabs
**Level:** Fácil
**Date:** 13 enero 2026
**Autor**:
**Descripción**: 

Enumeración Web
Crypto Challenge - B64 Cipher
Enumeración del Sistema de Archivos
[[Escalada de Privilegios]]

**Flags**:  


**Begin**:

---

Comenzamos haciendo un ping:
![Pasted image 20260110160513.png](../../_assets/01c19c43ecbcd1c9b0b2c3cc24c8fb0810957261512e4c5e8e513fd72cae3e20.png)
Hay conexión 
TTL: 64 -> Linux

Escaneo de Puertos abiertos TCP
![Pasted image 20260110160634.png](../../_assets/bc5f7daa925063e0a5051d1b010f2f2f874959ed217cb38e5117276b612fe76d.png)
Puertos abiertos: 
Puertos 22 (SSH)
Puertos 80 (HTTP)

Escaneo de versión a los puertos específicos.
![Pasted image 20260110160642.png](../../_assets/7cf5238affbc329b62de2906f577df7079179b5ed3c43021efdcbbc628c57e11.png)


Accedemos al servicio web
![Pasted image 20260110160824.png](../../_assets/7e2e56f2317b06beb8d54faff06899008cc317053177dad36e443af12d2a4337.png)
El servidor respondió con la página por defecto de Apache2

Realizamos un fuzzing de directorios.
![Pasted image 20260110161122.png](../../_assets/56d99ed92b53e46aea3df502efb9eb3a9a2d18b27b6557491c8593ad8acaf344.png)
No encontramos nada.

Inspeccionando mas la pagina web, vemos que abajo del todo vemos una cadena de texto en B64.
![Pasted image 20260110161509.png](../../_assets/ecc4abcb6bee34b92b327c30c71ce37b6c85b4ac445304021e1d49dabc92c0f3.png)

Lo llevamos a `CyberChef`
![Pasted image 20260110161611.png](../../_assets/7aa5669c3c7122509b11d1f397a0eb59d9ce704e26816945d9ff51fa10710212.png)
Y nos da una cadena de texto que parece ser unas credenciales
`daniela:focaroja`

Accedemos al servicio SSH
![Pasted image 20260110162230.png](../../_assets/1cbc9633f39a1f1f9884d3ae25f8efe4f9acc0eef55b77407b2b4476b4483a84.png)

Listamos los archivos y directorios del usuario
![Pasted image 20260110162445.png](../../_assets/da3d38ebd48b9647a7b62b2a9016aa0f0f4bfb6211978da316e2777c98cb77fb.png)
Y vemos un directorio llamado `.secreto`.

Entramos al directorio `.secreto`
![Pasted image 20260110162706.png](../../_assets/c4f06110086930dc6f19573ee388b9058084e4c1cec1ad5a5258a517e25aecf5.png)
Y dentro de ello hay una carpeta llamada `passdiego` que parece ser la contraseña del usuario `diego` al leerlo nos da una cadena de texto.

La cadena de texto lo pasamos por `CyberChef` y nos da la palabra `ballenanegra`
![Pasted image 20260110162727.png](../../_assets/254392b7c05004fee157ccd5bcbeccd68fa3343fa8a88f8aca078ea39db4d238.png)

Accedemos como el usuario `diego`
![Pasted image 20260110162830.png](../../_assets/05d7e3a3222a88e77d04e4c539440dc1b92f1b6c3329a0299d04abe0f705db13.png)

Listamos los archivos del usuario
![Pasted image 20260110162950.png](../../_assets/5dfe81a2bf7b7733e306da8b4a1c92808ea7efa943b9b4209fd8ad82a6448972.png)
Vemos un archivo `pass` que solo dice: `donde estara`

Listamos los archivos ocultos tambien.
![Pasted image 20260110163951.png](../../_assets/d7c4da38d48225397c2679de41410be85f00f7d80de2ca100a03244caa14ec60.png)

Leemos el `.bash_logout`
![Pasted image 20260110164226.png](../../_assets/8616fab1f0f66cec1a77f78a808f7b0150fd5515c580fd33004dfe874f3af27d.png)
Vemos una cadena de texto de B64

Lo llevamos a `CyberChef`
![Pasted image 20260110164248.png](../../_assets/d09d661e890d0579a246f83050758deee8ebdcab0614f4a850d8cc8d921f35e1.png)
Nos da la palabra `b3oazul`

Intentamos acceder a **root** de contraseña `b3oazul`
![Pasted image 20260110164259.png](../../_assets/0b761662445fef31d5e8ab4af2732807477055de9c720793580bb1a77e0ee61c.png)
Y al parecer no es 

Después de buscar en varios lugares, en el directorio `share` se encuentra un archivo llamado `.-`  
![Pasted image 20260110164003.png](../../_assets/f6ecaa0eea7da8f47b49d291425b4772e07779f779aa4abd5ddcd6dfd830baa1.png)

Que al leerlo nos da una adivinanza sobre la contraseña de **root**
![Pasted image 20260110164012.png](../../_assets/fb64c2e2efa1e24223e35db537934bffc0ced02693ef49d196d7bbc203a08f7a.png)
Buscando la respuesta a la contraseña y guiándome de `b3oazul`
Se logra encontrar la respuesta que es: `osoazul`

Accedemos con **root**
![Pasted image 20260110164343.png](../../_assets/d6e6a48781e592d3cea36998241af2fc70117986685d5b85cf9a3b7eca6fab04.png)
Escalada exitosa a root.
