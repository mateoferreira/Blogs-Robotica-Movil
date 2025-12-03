# P5 - CREACIÓN DE UN MAPA GLOBAL

Para esta práctica, he tenido que construir mi propio mapa con enfoque probabilístico. No ha sido muy complicada ya que ya me había creado un mapa en la anterior práctica, y he podido reutilizar algo de código.

La diferencia con la anterior, es que el robot se va construyendo el mapa con las lecturas del laser 360 que tiene, en vez de basarse en otro mapa para crear uno nuevo. Para ello voy a dividir la explicación de mi funcionamiento en 3 partes:

### 1 - NAVEGACIÓN
Para la navegación me he complicado muy poco. Si bien podría haber empleado la pseudo-aleatoria de la p1 o un movimiento en S, ya que sabemos la posicion del robot, como en la demo; he optado por algo mas simple y casi siempre funcional. Si hago que el robot vaya recto hasta que alguna de las medidas del laser frontal sean menores a x distancia, y en ese punto gira para la derecha o para la izquierda, el robot se mueve aleatoriamente cubriendo la mayor parte de la superficie. Gasto mucho mas tiempo en recorrer todo el mapa, pero la complejidad es menor y el robot no se choca.

### 2 - LECTURA DE LÁSER
Mientras que mi robot comprueba el laser para ver si hay obstaculos para la navegación dentro del bucle principal, se usan esas mismas medidas para comprobar los obstaculos en el instante t. Entonces me defino dos casos por cada medida. Si es menor que una distancia máxima que yo me defino, entonces es que tenemos un obstaculo en el angulo correspondiente a esa medida a la distancia de esa medida. Si es mayor a la distancia maxima, tenemos un espacio libre desde el robot hasta la distancia maxima en ese angulo. Entonces, como podremos ver en el video, cuando el robot se va moviendo va actualizando de forma circular con radio: *max_distance* en los angulos donde no hay obstaculos, y que se cortan en los angulos que si hay obstaculos.

### 3 - ACTUALIZACIÓN DEL MAPA
La parte más dificil sin duda es la de la implementación del método probabilístcico para la actualización del mapa, ya que no solo tienes que actualizar el mapa, sino también tienes que considerar que pixeles actualizar.
Para ello es esencial el uso de odometría o GPS, ya que por calculos matemáticos, sabiendo la posición x, y, theta del robot global y la distancia y el angulo de los obstaculos respecto al robot, puedes saber la posicion global de los obstaculos.

Para hacer estos calculos, primero me calculo una escala, que es de 25, es decir, un metro son 25 pixeles. 
Luego me calculo las x y las y en el mundo con una relacion de senos, cosenos y sumas y restas. Recalco lo de "en el mundo" ya que nuestro GPS no nos da una posicion del robot donde el centro del mapa es el 0, 0, asique para pasar de las x e y que te da el GPS a las de nuestro mapa, tenemos que sumarle la posicion inicial que nos definimos nosotros, que sería el centro del mapa.

Una vez tengo las posiciones en el mapa de donde se encuentran nuestros obstaculos, ya puedo empezar a implementar mi método de creación probabilístico. Para ello me defino al principio los ratios de ocupación de las celdas, donde la probabilidad de que este ocupada si detectamos un obstaculo es de 0.8, ya que estamos con un sensor fiable como es el laser.

Entonces con este método voy actualizando mi mapa según las lecturas que tenga de las distintas celdas, pero no lo voy actualizando constantemente para no saturar, sino que me voy a guardar la posición del robot cuando actualizo el mapa, y en cada iteración del bucle compruebo si se ha movido más de 1.5 metros desde la ultima actualización. Si es así, me guardo la posición actual, actualizo el mapa con las lecturas en ese momento, y continuo sucesivamente.
La actualización consiste en multiplicar el ratio de cada celda por el de la lectura actual. El ratio cuando está ocupada es p(ocupada)/p(vacia), y el inverso cuando está libre. Importante que siempre trunco la probabilidad entre 0 y 1, y las probabilidades de ocupación iniciales son de 0.5 ya que no tenemos ni idea, es decir, el ratio inicial de todo el mapa es 1.

A partir de este punto es donde empezaron a venir los problemas. El primero y más importante es que sabía como detectar un obstaculo, pero no sabía como hacer una linea recta entre ese obstaculo (o la maxima distanca del laser) y el robot con el angulo corresponiente para saber las celdas intermedias que estan libres. Para ello, la función *linspace* de numpy me ha ayudado, aunque he intentado entender al 100% el comportamiento de mi codigo en esta parte, la verdad es que no lo he conseguido, pero el resultado es bueno.

Otro problema, y el mas notable, es la rotación del mapa. He intentado con multitud de combinaciones de x e y positivas o negativas, pero siempre había una rotación o inversión. Por eso lo he dejado rotado 90º hacia la izqueirda. Esto lo considero error de visualización, no del algoritmo, por eso lo dejo así.

Podemos ver como el mapa tarda más en tender a 0 o a 1 por cada celda depende de la fiabilidad del sensor. Si empezamos con 0.8 podemos ver como ya casi las celdas libres estan blancas, si lo ponemos a 0.6 por que nuestros sensores fuesen menos fiables, los valores recién descubiertos se ven mas grisaceos.

Un ultimo detalle es que dependiendo de si la posición la sacamos con el GPS o la odometría se ve mejor o peor el mapa, ya que como hemos visto, es indispensable tener una buena autolocalización para ubicar correctamente los obstáculos. Con Odom2, mi mapa se ve así:

<img width="1103" height="279" alt="Captura desde 2025-12-03 20-09-16" src="https://github.com/user-attachments/assets/8a6dca1d-c99a-48b0-9a96-6df4f36a6dd5" />

Vemos claramente que la mala autolocalización desemboca en un mal mapa
Con Odom:

<img width="1103" height="279" alt="Captura desde 2025-12-03 19-57-42" src="https://github.com/user-attachments/assets/1e43b619-5286-4c21-ba49-c5b9ff58ee28" />

El mapa es bastante mejor, pero las estanterías que son obstaculos más finos, son casi imperceptibles en mi mapa, por que no está tan bien ubicado. Un buen ejemplo de como se debe de ver es localizandose mediante GPS:

<img width="559" height="342" alt="Captura desde 2025-12-03 18-32-35" src="https://github.com/user-attachments/assets/37765d6e-eb21-4991-a81e-e8c80d0bff84" />

Donde se ve claramente la delimitación de los obstáculos.

Un video de mi funcionamiento:

[Grabación de pantalla desde 2025-12-03 18-16-03.webm](https://github.com/user-attachments/assets/76ccf540-3af6-4ecf-b0dd-866f7e230b99)


