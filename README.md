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

#### Indirect expansion

Se puede hacer que la shell reemplace el valor del parametro por una string si el primer caracter es un simbolo de explamacion *!* y a su vez reemplace ese valor como si fuera una variable misma, por ejemplo :

```
val="numero"
numero=254
echo ${!val}
254
```

#### Testeo de nulidad

Se puede testear los contenidos de dos valores y devolver uno de ellos dependiendo de cierta logica. Por ejemplo, en este caso, se testea si el valor del primer parametro es nulo, y en caso afirmativo, se devuelve el valor del segundo parametro utilizando el simbolo **-** :

```
echo ${num:-13}   ## Aca el valor de num nunca fue seteado, por lo tanto se devuelve un 13
13
num=20            ## Aca seteamos num a 20, por lo tanto ahora nos devuelve el valor de num.
echo ${num:-13}
20
```

Tambien se pueden utilizar variables obviamente, pero debemos anidar la expansion: 

```
precio=200                           ## Seteamos precio a 200
echo ${no_existo:-${precio}}         ## devolvemos el valor de precio 
200
echo ${no_existo:-precio}            ## Sin anidado nos devuelve el valor literal
precio
```

Con el simbolo de **=**, lo que hacemos es asignar primero y devolver despues. Por ejemplo :

```
echo ${no_existo:=200}   ## No solamente devolvemos el valor, sino que ademas lo asignamos a la variable no_existo
200
echo $no_existo
200
```

Con el simbolo de **?**, podemos devolver un mensaje que va a parar al **stderr**. Por ejemplo :

```
unset no_existo
echo ${no_existo:?'La variable no existe!'}
-bash: no_existo: La variable no existe!
```

Con el simbolo de **+**, podemos realizar el inverso de **-**; en este caso, podemos devolver el valor del segundo parametro simpre y cuando el valor del primer parametro no sea nulo, por ejemplo :

```
echo ${num:+'true'}
                     ## Nos devuelve vacio, ya que num es nulo
num="1"              ## Asignamos algo en num
echo ${num:+'true'}  ## Y ahora si devolvemos el valor del segundo parametro, o sea True
true
```

#### Substring Expansion

El foramte basico es : **${parameter:offset:length}**, donde *offset* y *length* son dos enteros, y *length* es opcional. Basicamente nos permite sustraer una cadena de caracteres desde el valor de *parameter*. Aca van algunos ejemplos ( los mismo que en la pagina de GNU ):

```
string=01234567890abcdefgh
echo ${string:7}
7890abcdefgh
echo ${string:7:0}

echo ${string:7:2}
78
echo ${string:7:-2}
7890abcdef
echo ${string: -7}
bcdefgh
echo ${string: -7:0}

echo ${string: -7:2}
bc
echo ${string: -7:-2}
bcdef
set -- 01234567890abcdefgh
echo ${1:7}
7890abcdefgh
echo ${1:7:0}

echo ${1:7:2}
78
echo ${1:7:-2}
7890abcdef
echo ${1: -7}
bcdefgh
echo ${1: -7:0}

echo ${1: -7:2}
bc
echo ${1: -7:-2}
bcdef
array[0]=01234567890abcdefgh
echo ${array[0]:7}
7890abcdefgh
echo ${array[0]:7:0}

echo ${array[0]:7:2}
78
echo ${array[0]:7:-2}
7890abcdef
echo ${array[0]: -7}
bcdefgh
echo ${array[0]: -7:0}

echo ${array[0]: -7:2}
bc
echo ${array[0]: -7:-2}
bcdef
```

Para conocer el largo de una string :

```
nombre='Matias'
echo ${#nombre}
6
```

Para conocer el largo de un array : 

```
frutas=('Banana' 'Manzana' 'Pera')
echo ${#frutas[@]}
3
```

Obtener una substring borrando desde el comienzo de la string. Con un solo simbolo **#** se busca el match mas corto, mientras que con dos **##** se busca el mas largo. Por ejemplo, para borrar un prefijo de nombres de archivo :

```
filename="data_20181229121512"
echo ${filename#*_}
20181229121512
```

A modo analogo, para borrar parte de una cadena de texto desde el final, por ejemplo para borrar la extension de una archivo :

```
filename="my_picture.jpg"
echo ${filename%.*}
my_picture
```

Aca se ve mas claro la differencia entre **#** y **##**, por ejemplo, si el nombre de archivo tiene mas de una extension :

```
filename="my_picture.jpg.txt.zip"   ## Solo nos va a borrar la primera apariencia. Para borrar todo debemos utilizar dos simbolos %
echo ${filename%.*}
my_picture.jpg.txt  
echo ${filename%%.*}                ## Ahora si!
my_picture
```

#### String substitution

El formato es  : **${parameter/pattern/string}** donde *pattern* **NO ES** una expresion regular, sino mas bien un globbing pattern. Por ejemplo :

```
address="matias@gmail.com"
echo ${address/gmail.com/yahoo.com}
matias@yahoo.com
echo ${address/matias/XXXXX}
XXXXX@gmail.com
echo ${address/[a-zA-Z]/?}   ## NO ES UNA EXPRESION REGULAR!!
?atias@gmail.com
```

#### ToUpper and ToLower

Se puede convertir el valor del primer parametro a mayusculas con **^** o a minusculas con **,**. Si se repite el simbolo, por ejemplo en el caso de **^^**, se pasa a mayusculas tdos los caracteres del string y no solo el primero. Por ejemplo : 

```
nombre='matias'
echo ${nombre^}
Matias
echo ${nombre^^^}
matias
echo ${nombre^^}
MATIAS
```

De forma analoga, se puede pasar a minusculas :

```
nombre='MATIAS'
matiasbarrios@~
echo ${nombre,}
mATIAS
echo ${nombre,,}
matias
```

Si se pasa un array como parametro, la transformacion se aplica a cada elemento del iterable :

```
frutas=('Banana' 'Manzana' 'Pera')
echo ${frutas[@],,}
banana manzana pera
```

## Arithmetic Expansion

Para realizar la expansion aritmetica se deben poner los parametros dentro de : **$ ((  ))**. Dentro de esta sintaxis los valores literales se toman como tales y los string se toman como variables. Por ejemplo : 

```
valor1=13
valor2=7
echo $(( valor1 * valor2 ))
91
echo $(( valor1 * valor2 * 2  ))
182
```

