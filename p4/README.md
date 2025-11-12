# PRACTICA 4 - NAVEGACIÓN GLOBAL
Esta práctica la he dividido en partes para afrontarla lo mejor posible.
- Hacer el descenso por gradiente desde el objetivo global
- Engordar las paredes del mapa obtenido en el descenso por gradiente
- Navegar hasta el destino global mediante la tecnica de navegacion global: Gradint Path Planning

### DESCENSO POR GRADIENTE
Lo que más me costó de este paso fue interpretar las coordenadas del mapa global original.
Este mapa es un array bidimensional de *numpy*, pero en vez acceder a una posicion como: mapa[x,y], hay que hacer: mapa[y,x].
Por eso, las primeras veces que intentaba ejecutar mi codigo, tenia valores de destino bastante incoherentes, y a la hora de visualizar el mapa, estaba invertido.
Lo primero de mi programa, es hacer un bucle, con frecuencia de 10 hz para no saturar el programa, donde miro si el usuario elige un target con el mapa *(while target_global is None:)*
Una vez el usuario clica, siempre que clique en una calle vacía, nos guardamos el target e iniciamos la creacion de nuestro mapa.
Para poder crear este mapa a partir del descenso por gradiente, he implementado, como recomienda el guion de la practica, el algoritmo BFS de expansión por anchura, es decir, expandir todos los puntos que esten equidistantes al centro, y una vez expandidos, expandes la siguiente capa, y la siguiente hasta llegar a la posición del coche. Esta posicion me la guardo antes de realizar el algoritmo, y cuando llego hasta la posicion del coche en la expansion, expando mi gradiente solo 3 veces más, para poder permitir un giro de coche, en caso de que el coche tenga que girar 180º, por ejemplo.
La implementacion del BFS, ha sido en gran parte ayudada por mis apuntes de la asigantura Inteligencia Artificial, pero un poco modificado para este ejercicio. Para poder realizarlo correctamente, tengo que iniciar una cola de la libreria collections, y al principio del algoritmo inserto en la cola el target global que me he guardado. Tambien me inicio un array de numpy donde cada pixel vale infinito, que va a ser el mapa que voy a ir creando. La posicion del target va a tener el valor 0 en nuestro mapa.
Luego, mientras queden pixeles que expandir en nuestra cola, mira los posibles pixeles expandidos, y siempre que no sean un edificio en el mapa original, los pintamos en nuestro mapa con coste: coste del pixel expandido + coste del movimiento que hago en la expansion (1 si es alante, atras, izquierda, derecha, y raiz de dos si a una de las esquinas), siempre y cuando el nuevo coste sea menor que el que ya tiene, asi nos aseguramos de no pisar nuestro coste constantemente.

Estos pixeles sin expandir los voy metiendo a la cola hasta que los expanda todos, y cuando los expanda los siguientes, y cuando llegamos a la posicion del coche, seguimos expandiendo hasta que queramos expandir un pixel cuyo coste sea la posicion del coche +3, como he dicho antes. Cuando llegamos a este punto, paramos la expansion y continuamos hasta el paso siguiente:

### PENALIZACION DE PAREDES
Para no ir muy pegados a las paredes en nuestra conduccion, ya que en una misma calle seguramente los valores de un lateral sean menores por como funciona la expansión, penalizamos los pixeles que estan hasta dos posiciones mas lejanas de las paredes. Para ello, miro si en el mapa original tenemos pared (valor 255), y si es asi, penalizamos los pixeles de nuestro mapa que están pegados a las paredes sumandole 1000, y los siguientes con un valor de 500. Asi nos aaseguramos que nunca vamos a ir por esos pixeles, ya que sería contrario a nuestro algoritmo.
Una vez tengamos estas dos fases, ya tenemos completado nuestro mapa y podemos usarlo para la navegacion, asique lo mostramos por pantalla (los valores mas oscuros son los mas pequeños, los blancos los mas grandes, y los negros que bordean nuestra expansion, son infinito)

## NAVEGACION
Para esta parte, he tenido otra complicacion que tiene que ver con las coordenadas, y esque el yaw del coche se mide desde el maletero, no desde el capó.
El algoritmo de navegacion que he hecho, comprueba en un "ovalo" de radio menor 4 y de radio mayor 9 los costes de los pixeles interiores, y va buscando el coste minimo, asi navegamos siguiendo subobjetivos.
Una vez hallado el coste minimo dentro del radio definido, normalizo el angulo entre el coche y ese pixel con coste minimo, para ver lo bien orientado que voy, y calculo la distancia entre mi coche y el subobjetivo, si es menor que 4, interpreto que he llegado a mi destino.
Para elegir las velocidades, opero sobre todo con el angulo
