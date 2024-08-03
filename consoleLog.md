- Tags: #InformationLeakage #hydra 
_____
comenzamos la maquina con el tipico escaneo de nmap, y nos reporta lo siguiente.
_______
![](attachment/2d5b945d90ee5688f73e4afd6dddae40.png)
______
tenemos tres puertos abierto, y tenemos un ssh pero reasignado a otro puerto.

investigaremos la web para ver que obtenemos.
_____
![](attachment/46cd32daf9e29e68b6f3e02bb84bb1b6.png)
_____
no obtenemos nada relevante por lo que investigaremos el código fuente.
_____
![](attachment/ffedc506bd1a5d7bf32910bba650f30e.png)
______
también podemos aplicar fuzzing para ver que encontramos.
_____
![](attachment/1edfccf6576131d09056407518e71081.png)
_____
si accedemos a ese recurso podremos visualizar lo siguiente.
____
![](attachment/59f530042932740b58cb3ff6730a695f.png)
____
obtenemos una password pero nos falta el usuario, por lo aplicaremos fuerza bruta con hydra para ver que nos reporta.
_____
![](attachment/4c7003fe5b18ffe656d43afd84eaae17.png)
______
ahora podemos conectarnos con ssh para después enumerar el sistema y ver como podemos escalar privilegios.
_____
![](attachment/2a4efcf32af47778eb83a17c62a2c77d.png)
_____
tenemos nano con permisos SUID por lo que podemos alterar el archivo passwd para inhabilitar la contraseña para ser usuario root, así podremos pivotar a root sin contraseña.
______
![](attachment/633f2298e280ae8166c5e0594727e150.png)
____
le quitamos la x, y ahora aplicamos un su root y automáticamente nos convertimos a root.
____
![](attachment/f8b3568f2e34eff405d5c90ecb7cfbc3.png)