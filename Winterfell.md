- Tags: #InformationLeakage #LibraryHijackingPython 
_________
comenzamos la maquina con los escaneos de nmap que nos reportan 4 puertos abiertos en lo que destacan los servicios de smb.
____
![](attachment/c079345ab0bd92be784e45db147a0205.png)
____
investiguemos primero el puerto 80, para ver que encontramos.
_____
![](attachment/d2a2c00187a3f5e32345975e8232f63d.png)
____
tenemos de inicio un escrito donde se pueden apreciar 3 posibles usuarios, sigamos investigando para ver que mas nos encontramos.
_____
![](attachment/37a2e0ff064c1ac270b74b6810892873.png)
______
tenemos un directorio llamada dragon, por lo que accedamos para ver que contiene.
_____
![](attachment/7f87ff3bea145f96a8410953dc57a34e.png)
_____
contenía un archivo con este texto, podemos armar un diccionario pues no es casualidad que todas las palabras estén juntas.

esto es todo lo que encontraremos en la web, ahora podemos enumerar el servicio de samba, porque obviamente probamos los usuarios con las posibles contraseñas pero no obtuvimos resultados.

por lo que procedamos con samba y ver si podemos enumerar usuarios de ese servicio.

podemos utilizar rpcclient para ver si nos reporta usuarios.

```bash
rpcclient -U "" -N 172.17.0.2 
enumdomusers
```

utilizando `enumdomusers` para listar usuarios disponibles.
_____
![](attachment/e1a47e54a6f1c380f202e901eef95091.png)
___
por lo que confirmamos un usuario de los 3 presuntos anteriores.

podemos utilizar crackmapexec para aplicar fuerza bruta al servicio de smb con el diccionario que creamos, para ver si podemos acceder al servicio en si.
_____
![](attachment/30e97d81564a35f70939204438c3b323.png)
______
podemos acceder al servicio con esas credenciales.
_____
![](attachment/583e917239cee5617d9c6c35e0ecca60.png)
_____
podemos descargar el archivo con get y ver que contiene.
______
![](attachment/1a63a11aec35a33734c69ad0e9edf773.png)
_______
tenemos una credencial codificada en base64 por lo que aplicaremos `base64 -d` para decodificar la credencial.
______
![](attachment/8b8f645197dbc3f87e05bf8c0a65a9fb.png)
_____
obtenemos la clave vemos si podemos acceder por ssh con el usuario jon
____
![](attachment/220d8d883a31a8616257a1958396c0ef.png)
_____
ahora tenemos que ver como pivotar entre usuarios.
_____
![](attachment/9cfb0660f1fb20a2ffe345db3ae49358.png)
_____
podemos observar el contenido de mensaje.py para ver que nos encontramos.
_____
![](attachment/d212654a25b3ea947f88ad5bc89d5d62.png)
____
se están importando dos librerías pero no tienen su ruta absoluta por lo que se podría acontecer un Library Hijacking de python, lo que vamos hacer a continuación es comprobar las rutas donde python esta buscando estas librerías.

lo podemos hacer con el siguiente comando.

```python
python3 -c 'import sys; print(sys.path)'
```

______
![](attachment/2e4b69cac38bc143bd894f81ec0f3155.png)
_____
esas comillas vacías nos indican que python primero busca por el directorio actual y si no encuentra nada prosigue con los demás directorios.

se evalúa de izquierda a derecha.

lo que haremos es crear un fichero .py con el nombre de una de las librerías que contiene el script con el siguiente contenido.

```python
import os
os.system("/bin/bash")
```

de esta forma cuando el script busque la librería que esta importando va a verificar primeramente nuestro directorio actual y si encuentra un fichero con el mismo nombre que el de la librería que busca pues utiliza ese.
_____
![](attachment/59f66824746f9c349ae015de6f2dfb7a.png)
____
ahora ejecutamos el script como aria.
____
![](attachment/7ac713d614bba9d02f48fc06b4488a39.png)
_____
ahora que somos aria solo nos falta un usuario al cual pivotar.
______
![](attachment/d236d504e3ccbb61733b4468fbeaac57.png)
_____
aplicando el comando sudo -l vemos que podemos ejecutar cat y ls como daenerys, esto nos servirá para poder ver el contenido de su directorio, así podremos listar archivos y poder ver su contenido con cat.
_____
![](attachment/e749024c6adfd3c2fc4fda419bd93814.png)
_____
tenemos un archivo, ahora veamos su contenido.
_____
![](attachment/de44830fc5f1bff9eb7e1e788cdbf95b.png)
_____
obtenemos una credencial, ahora pivotaremos al usuario daenerys, y vemos como llegar a root.
_____
![](attachment/66cd7d1bb3bb44388953ca436f698714.png)
_____
tenemos un script en bash llamado shell vemos que permisos tenemos sobre el archivo.
____
![](attachment/978ca5c18c9487cd09796dc3e36cee22.png)
_____
podemos modificar el script por lo que usaremos esto para escalar privilegios.
____
![](attachment/3de00c66cf9cfcc0c23a31dc53b3454b.png)
_____
guardamos, ejecutamos y aplicamos un bash -p y asi obtendremos root.
____
![](attachment/29a9ab185b15535d310c22e7521c5dc5.png)
