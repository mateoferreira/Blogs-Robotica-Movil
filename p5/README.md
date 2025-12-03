# P5 - CREACIÓN DE UN MAPA GLOBAL

Para esta práctica, he tenido que construir mi propio mapa de manera probabilística. No ha sido muy complicada ya que ya me había creado un mapa en la anterior práctica.

La diferencia con la anterior, es que el robot se va construyendo el mapa con las lecturas del laser 360 que tiene, en vez de basarse en otro mapa para crear uno nuevo. Para ello voy a dividir la explicación de mi funcionamiento en 3 partes:

### 1 - NAVEGACIÓN
Para la navegación me he complicado muy poco. Si bien podría haber empleado la pseudo-aleatoria de la p1 o un movimiento en S, ya que sabemos la posicion del robot, como en la demo; he optado por algo mas simple y casi siempre funcional. Si hago que el robot vaya recto hasta que alguna de las medidas del laser frontal sean menores a x distancia, y en ese punto gira para la derecha o para la izquierda, el robot se mueve aleatoriamente cubriendo la mayor parte de la superficie. Gasto mucho mas tiempo en recorrer todo el mapa, pero la complejidad es menor y el robot no se choca.

### 2 - LECTURA DE LÁSER
Mientras que mi robot comprueba el laser para ver si hay obstaculos para la navegación dentro del bucle principal, se usan esas mismas medidas para comprobar los obstaculos en el instante t. Entonces me defino dos casos por cada medida. Si es menor que una distancia máxima que yo me defino, entonces es que tenemos un obstaculo en el angulo correspondiente a esa medida a la distancia de esa medida. Si es mayor a la distancia maxima, tenemos un espacio libre desde el robot hasta la distancia maxima en ese angulo. Entonces, como podremos ver en el video, cuando el robot se va moviendo va actualizando de forma circular con radio: *max_distance* en los angulos donde no hay obstaculos, y que se cortan en los angulos que si hay obstaculos.

### 3 - ACTUALIZACIÓN DEL MAPA
La parte más dificil sin duda es la de la implementación del método probabilístcico para la actualización del mapa, ya que no solo tienes que actualizar el mapa, sino también tienes que considerar que pixeles actualizar.
Para ello es esencial el uso de odometría o GPS, ya que por calculos matemáticos, sabiendo la posición x, y, theta del robot global y la distancia y el angulo de los obstaculos respecto al robot, puedes saber la posicion global de los obstaculos.
