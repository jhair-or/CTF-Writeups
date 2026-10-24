
**IP**:  172.17.0.2
**S.O**: Linux
**Source:** DockeLabs
**Level:** Fácil
**Date:** 3 febrero 2026
**Autor**:
**Descripción**: 

Web Enumeration
Fuzzing with gobuster
[[SQLI]] SQL Injection
[[Escalada de Privilegios#**Escalada de Privilegios mediante SUID**]]
[[Escalada de Privilegios#**Escalada de Privilegios mediante Claves Expuestas**]]

**Flags**:  


**Begin**:

---

Comenzamos haciendo un ping:
![Pasted image 20260203115328.png](../../_assets/c527bf594892e6f6f941baea8d539cfc4e1fbe1a0f7360cdcd525cc76db191f6.png)

Hacemos un escaneo de puertos 
![Pasted image 20260203115315.png](../../_assets/f18efbcf2cb6186a2c56f06e0eefdefe5e7f8c2ac2b42b832e55101c1cafae17.png)

Accedemos a la pagina web
![Pasted image 20260203115512.png](../../_assets/eb0aea4384dc38a5e023b52253929a995d03880c281e49000fae6233b3e2691c.png)

Hacemos un fuzzing de directorios.
![Pasted image 20260203115739.png](../../_assets/0940be9d2aa951d17d0bd587cc370b581db654c7e9354048806ed1ea121d6c5e.png)
Vemos que hay una ruta llamada `/login.html`.

Nos muestra un formulario de login simple.
![Pasted image 20260203115747.png](../../_assets/b54e87e647d417f80952b0f08dbc3632360806a686f6b79059b7a90822ae2d5f.png)

Probamos si es vulnerable a un sqli
![Pasted image 20260204163325.png](../../_assets/1a134f3a97f2b583e41ba53332c7dae5ff4a21373c4702b16dd011ff46dda0c9.png)

Y nos bota un error por lo que si es vulnerable
![Pasted image 20260204150228.png](../../_assets/d20f0f96fc192c11ba419a27353483a9ccc4a7d5ed4e20c63c209b6b353bbcae.png)

Interceptamos la pagina con Burp Suite, copiamos la estructura exacta del POST
![Pasted image 20260204170602.png](../../_assets/d833fb433b0f0e4fa1166c56a7fc0592be1cb9df1f5e365931c7afabe7d66cbc.png)
y lo pegamos en un archivo llamado `login.rep`.

Ejecutamos sqlmap
```bash
sqlmap -r login.req --batch --dbs
```
- **`-r login.req`**: Lee la petición desde el archiv
- **`--batch`**: Responde "Yes" automáticamente a todas las preguntas.
- **`--dbs`**: Si tiene éxito, listará las bases de datos.

Y no lista las bases de datos del servidor.
![Pasted image 20260204170640.png](../../_assets/5d711e6e5364f7624fcf261510d7e124c9a4b0c888a3160d39759a98034f6a25.png)

Consultamos las tablas de la base de datos de `users`
```bash
sqlmap -r login.req -D users --tables
```

Y encontramos la tabla `usuarios`
![Pasted image 20260204170749.png](../../_assets/39375778159fb95c174bc59790eb304f6573831ffbfcf6eed3141f1ca60ccc04.png)

Vemos que hay en las columnas dentro de la tabla
```bash
sqlmap -r login.req -D users -T usuarios --columns --batch
```
![Pasted image 20260204170841.png](../../_assets/922c251f9d545a759f04caa10d273aebeea455394287fa96e5851d9e1216d8e8.png)

Ahora vamos a Dumpear(Extraer los datos) la tabla
```bash
sqlmap -r login.req -D users -T usuarios --dump
```
![Pasted image 20260204173156.png](../../_assets/3875b6e12735739ea942c10d26a3f667794b5c7d42fc813d9fb9ea6ff750d745.png)

Accedemos como `pepe` y buscamos binarios con permisos SUID
![Pasted image 20260204173515.png](../../_assets/c71eb450c629af261b42a1f3b74813f46ab7c86de7fb019234590e9b5fa146f0.png)
Vemos que `ls` y `grep` tienen permiso SUID

Utilizamos el `ls` para listar los archivos del directorio de `/root`.
![Pasted image 20260204174234.png](../../_assets/c5826994d800ebafeaac826ea989c8a05a557d08f62a4cc0401e3f76bbc4bd81.png)
Vemos un archivo que se llama `pass.hash`.

Y procedemos a usar `grep` para leer el archivo `pass.hash`
![Pasted image 20260204174314.png](../../_assets/99f72d2473891fd4548fac5f78bb5b095415fdf83ba06a3a45c4f6a9da1b70f4.png)
Y nos dan una cadena de texto 

Lo llevamos a la pagina hashes.com para crakearlo
![Pasted image 20260204174339.png](../../_assets/018b5f5f845b4c5bfff1cef79d5ba7981313e99903ccf7b30d961fb34733eb1f.png)
Y nos da la palabra `spongebob34`

El cual probamos como contraseña del usuario **root**
Y logramos acceder como **root**
![Pasted image 20260204174401.png](../../_assets/79ccf33760dae239a00395f94f95bc26a35a5a211c100e92176623f8da5e3840.png)


