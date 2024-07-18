- Tags: #fuerzaBruta #BurpSuite #python 
_____
comenzamos la maquina con los tipicos escaneos de nmap.

se nos reporta un puerto abierto, por lo que aplicamos otro escaneo de nmap para determinar la version y el servicio que corre en ese puerto.
______
![](attachment/c69444383569fc920bd64050befcbb61.png)
___
si ingresamos veremos que tenemos un panel de login de jenkins por lo que nos disponemos a ver su código fuente y comprobar si existe información útil.

no encontramos nada por lo que haremos fuzzing de directorios, pero tampoco obtenemos nada solido.

nos queda aplicar fuerza bruta al panel de login usando BurpSuite, por lo que capturamos una petición y la enviamos al repeter.
_____
![](attachment/321aa123ca60d7bd30710eac438201c4.png)
______
seleccionamos el campo al que queremos aplicar fuerza bruta, en este caso aplicaremos fuerza bruta a la contraseña y emplearemos un ataque de tipo sniper utilizando un diccionario como rockyou.
____
![](attachment/6b0891696ed310211d9921cfff233e51.png)
_______
a su vez podemos configurar que tipo de error nos arroja cuando la credencial no es correcta, esto nos ayudara para saber la credencial que si es correcta, puesto que al no aparecer el error sabremos que el correcta.
______
![](attachment/b5e5b0d72238658ba61ff00fda063cbd.png)
______
empezamos el ataque y el mismo nos reportara una contraseña para el usuario admin:rockyou.

ingresamos y buscaremos la siguiente opcion.
____
![](attachment/05c6267f8cd74a04d0299d7f5ab7464d.png)
_____
esto nos dará una consola donde podremos ejecutar código, pero no cualquier código, tenemos que utilizar groovy.

por lo que utilizaremos reverse shell generator para crear una reverse shell con nuestra ip y puerto.
_____
![](attachment/8f2cd4fc35d119e66111bcdc98a2a922.png)
_____
aplicando esta reverse shell obtendremos acceso a la maquina victima.
_____
![](attachment/922403f6e7d2db2b0000e83ceef780e0.png)
_______
buscamos por permisos SUID y vemos que tenemos a python por lo que buscaremos en gtfobins para ver como podemos escalar privilegios y utilizando python.
____
![](attachment/bcaebfb4f77e97bd393706d7d7f269b0.png)
_______
aplicando ese comando de python podremos escalar privilegios a root.
_____
![](attachment/528978a07dad0201a6c753f76bb60608.png)

