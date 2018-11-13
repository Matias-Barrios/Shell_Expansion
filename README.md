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

#### Secuencias numericas

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

Se pueden utilizar caracteres y la secuencia se 
