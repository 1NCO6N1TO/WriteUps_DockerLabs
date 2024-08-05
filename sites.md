- Tags: #InformationLeakage #LFI #sed 
_______
comenzamos con los escaneos típicos de nmap los cuales nos permiten conocer los puertos abiertos, ademas de las versiones y servicios.
_____
![](attachment/fa46579df00eb0fab403a912ee80659c.png)
_____
empezaremos investigando la pagina web para ve que encontramos.
_____
![](attachment/bab983ddf4c66943ae420ebd4d0a9413.png)
________
la pagina nos proporciona una serie de pista solo hay que unirlas, al parecer nos enfrentamos a un LFI.
_____
![](attachment/ae57c72f542b6d21248a83871b789026.png)
______
tenemos una fichero .php el cual verificaremos que exista.
_____
![](attachment/d40c8b239a3afd3c3f768278cdb58e6f.png)
_____
nos aparece lo siguiente, si se trata de un LFI debemos encontrar la palabra que maneja la petición GET, para eso aplicaremos fuzzing.
_____
![](attachment/2a2eed469dcabca43381ce338c44f93c.png)
_____
usaremos el parámetro page, para ver si podemos llegar a ver el passwd.
_____
![](attachment/353033162b8c02f2dcea6817c6558c97.png)
______
tenemos un posible usuario el cual si aplicamos fuerza bruta con hydra no obtendremos resultados, por lo que pienso que aun nos falta algo.

en la pagina web se menciona mucho un archivo sitio.conf el cual contiene información sensible, ahora mismo trataremos de identificar el archivo.
_______
![](attachment/45415faefca815daeb057bf265a9a30c.png)
______
el archivo se encontraba alojado en la siguiente dirección. `etc/apache2/sites-avalible/sitio.conf` y se identifico gracias a la pista de la pagina web.

podemos observar un mensaje que dice "bloquear acceso al archivo archivitotraviesito", veamos si el archivo existe.
______
![](attachment/4ce4fe791962ec901a083ef79822228d.png)
_______
no tenemos que aplicar fuerza bruta, puesto que ya enumeramos un posible usuario por lo que intentaremos acceder y veremos que sucede.
_____
![](attachment/53ee15545768083487a45eeb0cf8f32e.png)
______
accedemos y vemos que podemos ejecutar sed como root y sin proporcionar contraseña, busquemos la forma de escalar privilegios.

esto ya lo hemos visto anteriormente por lo que aplicaremos el siguiente comando para pivotar a root.

```shell
sudo sed -n '1e exec bash 1>&0' /etc/hosts
```

_____
![](attachment/d45ba3c63000eb2a40d3b123beeabace.png)


Fin.....