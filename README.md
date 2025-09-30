# Blog-Robotica-movil

Para afrontar esta páctica, he dividido las tareas de la aspiradora en:
+ Hacer un movimiento en forma de espiral
+ Ir marcha atrás durante 2 segundos cuando el robot se haya chocado
+ Girar un ángulo aleatorio entre 0 y 360
+ Ir marcha alante durante 5 segundos (si no se choca antes)
Estas tareas las he implementado mediante una máquina de estados que se van a ir ejecutando en un bucle infinito.

# Estado espiral
Este estado es el más simple, empieza en un  punto aleatorio con unas velocidades angular lineal iniciales de 2 y 0.05. La velocidad lineal inicial debe de ser baja, pues en cada iteración del bucle, vamos a ir incrementándola, para así ir cubriendo un radio más amplio (v = r * w). Si la velocidad lineal fuese muy alta, ya sea porque el robot empieza con mucha, o porque el incremento es considerablemente alto; el robot se descontrolaría.

Este ha sido el punto más difícil de este estado, escoger unas velocidades adecuadas, puesto que la siguiente función que tiene que realizar esta tarea es más simple: cuando el bumper detecte un choque, pasamos al estado siguiente. Pero antes, tomamos el tiempo actual para el estado siguiente

# Estado ir marcha atrás
Para que el robot pueda implementar un temporizador interno, es necesario incluir la librería *time* de Python. En el estado anterior hemos tomado el tiempo en el que cambiamos de estado, ya que este estado no va a volver a ejecutarse y el bucle no va a reiniciar el tiempo inicial.
Una vez tenemos el tiempo inicial, definimos la velocidad lineal como un numero negativo y la angular como 0. Así el robot va a ir marcha atrás, y con un *if* mido el tiempo actual, y si su diferencia con el inicial es mayor que dos, pasamos al estado siguiente

# Estado girar aleatoriamente
Este es el estado que más me ha costado diseñar.
El estado anterior, antes de hacer la transición, escoge un angulo random entre 0 y 360. El estado actual, va a estar comprobando en cada iteración el ángulo actual del robot. El problema es que la función *HAL.getPose3d().yaw* devuelve el ángulo actual en radianes, por lo que he tenido que diseñar una función auxiliar a la que se le pasa un ángulo en radianes y te devuelve el ángulo en grados. Para ello no basta solo con multiplicar x pi y dividir entre 180, ya que tenemos que considerar los radianes negativos que devuelve el robot. Mi función auxiliar hace la operación anterior, y sumamos 360 al resultado (así evito angulos negativos, -20 grados pasa a ser 340 grados, por ejemplo), y a continuación hace el módulo del resultado final. Así, los ángulos que ya eran positivos, se mantienen tal y como eran al principio

Por último, si el ángulo random es mayor que el angulo actual, el robot gira hacia la izquierda, y si es mayor hacia la derecha, cuando la diferencia entre los angulos esté en un intervalo de +2 grados, pasará al estado siguiente.

# Estado de ir recto
En el estado anterior, voy a guardarme el tiempo con el que empiezo a ir marcha alante, ya que voy a implementar el cronómetro de la misma manera que hice con el estado marcha atras.
La principal diferencia, a parte del sentido de la marcha y el tiempo del cronómetro, es que si en este estado detecto una colisión, voy a pasar automáticamente al estado de marcha atrás, como rectificación, y de ahí 


