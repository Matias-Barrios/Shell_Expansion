# Shell Expansion

Esta guia busca ser una referencia rapida sobre los distintos tipos de expansiones que realiza la shell Bash durante el parseo de los comandos que introducimos en la terminal. Toda la informacion se puede encontrar aqui (en Ingles): 

* https://www.gnu.org/software/bash/manual/html_node/Shell-Expansions.html

El orden en que realiza las mismas es :

1. Brace expansion 
2. Tilde expansion
3. Parameter and variable expansion
4. Arithmetic expansion and command substitution (de izquierda a derecha)
5. Word splitting
6. Filename expansion


## Brace Expansion

#### Secuencias numericas y lexicograficas

Hay que tener en cuenta que no se puede hacer secuencias numericas pretendiendo interpolar variables. Por ejemplo, el siguiente comando es invalido : 

```
inicio=0    ## ESTO NO FUNCIONA !!
fin=10               
echo {$inicio..$fin}
```

Para obtener secuancias numericas desde variables debemos utilizar el comando **seq** :

```
inicio=0
fin=10
echo $( seq $inicio $fin )
0 1 2 3 4 5 6 7 8 9 10
```

Para obtener secuencias numericas desde valores fijos :

```
echo {1..10}
1 2 3 4 5 6 7 8 9 10
```

Se puede pasar un tercer valor que sirve como monto de cada *"salto"* :

```
echo {1..10..2}
1 3 5 7 9
```

Se pueden utilizar caracteres y la secuencia se realiza por orden lexicografico :

```
echo {a..j}
a b c d e f g h i j
```

#### Secuencias arbitrarias ( separadas por coma )

Se puede realizar la expansion de secuancias arbitrarias siempre y cuando se separen los valores con comas. Por ejemplo, para obtener una direccion de mail con un dominio distinto para un determinado usuario :

```
echo matias@{gmail.com,yahoo.com,hotmail.com}
matias@gmail.com matias@yahoo.com matias@hotmail.com
```

Se pueden anidar corchetes ( braces ) de manera de realizar combinaciones de valores simultaneos. Por ejemplo : 

```
echo matias@{gmail.{com,com.uy},yahoo.com{.us,.uy},hotmail.com}
matias@gmail.com matias@gmail.com.uy matias@yahoo.com.us matias@yahoo.com.uy matias@hotmail.com
```

Se pueden combinar secuancias numericas o lexicograficas con secuencias arbitrarias :

```
echo jugaror_{rojo_{1..5},azul_{1..5}}
jugaror_rojo_1 jugaror_rojo_2 jugaror_rojo_3 jugaror_rojo_4 jugaror_rojo_5 jugaror_azul_1 jugaror_azul_2 jugaror_azul_3 jugaror_azul_4 jugaror_azul_5
```

o tambien : 

```
echo clase_{1ero_{A..C},2ndo_{A..C}}
clase_1ero_A clase_1ero_B clase_1ero_C clase_2ndo_A clase_2ndo_B clase_2ndo_C
```

## Tilde Expansion

La shell va a reemplazar el caracter ~ ( tilde ) por el valor seteado en la variable de entorno HOME. O lo que es lo mismo que decir que se reemplazara por el path completo al directorio HOME del usuario. Por ejemplo :

```
echo ~
/home/matiasbarrios
```

Si se escribe un nombre de usuario **sin escapar** ( o sea, si no se escribe dentro de comillas ), la shell intentara reemplazarese valor por el directorio HOME de ese usuario, por ejemplo :

```
echo ~centos
/home/centos
```

Si el usuario no existe, entonces simplemente cambiara el valor por los caracteres literales :

```
echo ~fakeuser
~fakeuser
```

Si se agrega un signo de **+**, la shell cambiara el valor por el directorio donde actualmente nos encontremos :

```
echo ~+
/home/matiasbarrios/sub_carpeta
```

Si se le agrega un signo de **-**, la shell lo cambiara por el path al ultimo directorio donde estuvimos ( un valor que se guarda en la variable de entorno OLDPWD ), por ejemplo : 

```
cd /var/log
cd ~
echo ~-
/var/log
```

## Shell Parameter Expansion

En su forma mas simple, se puede utilizar esta carateristica para escapar los nombres de las variables del texto que las rodea, por ejemplo : 

```
echo "val${val}val"
val1234val
```
De otra forma la shell no tiene forma de entender donde termina el nombre la variable que queremos interpolar :

```
echo "val$valval"   
val      ## Solo imprime literalmente val, y luego busca la variable valval, la cual no existe, por lo tanto no imprime nada.
```

