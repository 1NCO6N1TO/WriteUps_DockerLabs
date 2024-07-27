- Tags: #flask #python #sha1 #hydra 
______
comenzamos la maquina aplicando un escaneo de nmap para ver que puertos se encuentran abiertos.

seguimos con el escaneo de servicios y versiones.
____
![](attachment/7857c445d239719d873a2befa671f643.png)
_____
obtenemos dos puertos abiertos, investigaremos primeramente el puerto 80 y vemos que encontramos.
____
![](attachment/a50f27a658182a8bbdf96326c84a1b8c.png)
_____
tenemos una pagina web sobre algún restaurante, por lo que investigaremos a ver que encontramos.

no encontramos nada relevante, intentamos aplicar fuzzing pero sin éxito.

tenemos una clase de formulario al cuales ingresaremos datos y capturaremos la petición con BurpSuite.
_____
![](attachment/619a58c89ac0b0b8221537d5a5c0bddd.png)
_____
enviamos la petición al repeter y vamos probando, como es bien sabido los errores nos pueden dar pista de a que nos enfrentamos.
______
![](attachment/caf6a3746c64327d5d3c6ab0ed71cd80.png)
_______
si probamos con solo números nos sale el siguiente error, pero si colocamos una combinación alfanumérica nos aparece lo siguiente.
_____
![](attachment/099e2b459487b6626333d1e6949a091b.png)
_____
nos indica que hubo un error al codificar la cadena en base64, lo siguiente que haremos es codificar un comando en base64 y colocarlo como imput para ver como reacciona.
____
![](attachment/0ae9e6a5e04c2d63d0c0353cbdcd9c11.png)
______
como podemos ver estamos viendo el contenido del /etc/passwd usando el comando cat, todo el comando codificado en base64.

tenemos usuarios por lo que podemos aplicar fuerza bruta para ssh con hydra, o también podemos mandarnos una reverse shell.

la primer opción fue la que elegí.
______
![](attachment/5c6f2acbccef7683120755b983649a19.png)
____
una vez dentro buscaremos la forma de escalar privilegios.

encontramos el código el cual permite que la pagina web sea vulnerable.
_____
![](attachment/99d79510e49087ab3ee4ef4f94a4170a.png)
________
nos sirve para entender como funciona la vulnerabilidad.

La función `submit_Template` permite la ejecución de comandos arbitrarios enviados por el usuario debido al uso inseguro de `subprocess.Popen` con `shell=True`. Al decodificar el `userInput` y ejecutarlo sin validación ni sanitización, cualquier comando enviado en el `userInput` será ejecutado con los permisos del usuario que ejecuta la aplicación.

en directorio encontramos una fichero que contiene hashes
_____
![](attachment/7d11c1d698f33e8cc7f2dd91f8cd78e1.png)
_____
utilizaremos una herramienta patxacSec la cual nos permitirá descifrar los hashes.
______
![](attachment/e4b724ce0baa3e7d6b136819e419fb2c.png)
______
logramos crackear un hash el cual nos da la contraseña para root
_____
![](attachment/0d933786ffee143665394c65b8d91e9c.png)