# P6 - AUTOLOCALIZACIÓN CON APRILTAGS
Esta práctica ha sido sin duda alguna la más difícil, ya que conceptualmente es fácil pero he tenido que buscar por mi cuenta como hacer el PnP, multiplicación de matrices... Además, el laser y el bumper del robot no estaban habilitados (o por lo menos tal y como lo hemos implementado en prácticas anteriores), ya que las funciones no se reconocían.
Por eso, tras mucho esfuerzo, mi autolocalización es errónea, aunque he maximizado la similaritud, no se acerca ni mucho menos al video explicativo, pero aun así, voy a comentar mi práctica, ya que conceptualmente se acerca al objetivo.

### 1º parte: navegación
Como ya he dicho, no he podido navegar con laser ni bumper, por lo que he modificado el algoritmo de navegación aleatoria de la p1, es decir, implementando una máquina de 2 estados. Primeramente me calculo un tiempo aleatorio entre 1 y 5 segundos, y empiezo avanzando recto. En cada iteración compruebo si ya han pasado ese tiempo aleatorio, y cuando sea así, calculo otro tiempo random pero esta vez para girar y así sucesivamente.
Esto ha causado varios problemas, como por ejemplo el giro. Si la dirección del giro también es aleatoria, he comprobado que casi siempre va a seguir una "linea recta", cosa que pretendo evitar. Por eso, los giros que hago siempre son hacia la izquierda. Esta solución es muy pobre, pero es la más efectiva que he probado

### 2º parte: localización de balizas
Para esta parte, me he ayudado casi exclusivamente del código de ayuda proporcionado por el profesor. Lo único que he añadido, es detectar el area para todos los apriltags que detecto. Así, tal y como dice el enunciado, si detecto 2, me quedo con el más cercano (el que mayor area tenga)

### 3º parte: PnP
El objetivo principal es calcular la posicion en el mundo del robot, sabiendo la posición de los tags en el mundo. Para ello, tengo que primero calcular la posicón de los tags respecto de la cámara, y luego con una multiplicación de matrices lo saco facilmente.

El código de ayuda nos da la matriz de la cámara y los coeficientes de distorsión de la lente. Yo he tenido que definir los puntos en 3d (esquinas del tag) del tag calculado previamente (el mas cercano) siempre que haya tag, y la proyeccion 2d de estos puntos 3d en la imagen.

Con la función de openCV: *solvePnP*, que recibe estos 4 argumentos, obtenemos un vector de rotacion (rvec), que representa la orientación del tag vista desde la camara, y un vector de traslación 3d (tvec), que representa la posición del origen del tag vista desde la camara.

Para pasar del vector rvec a una matriz de rotación, he usado la función de openCV: *Rodrigues*. 
Como queremos hallar la posición de la cámara respecto del tag, la matriz de rotación que me interesa es la traspuesta de la que hallada con *Rodrigues*, y el vector de traslación es: *t_tc = -R_tc.T @ tvec*, es decir, la posición del tag en la camara (traslación) es: - matriz de rotación traspuesta del tag en la cámara * vector hallado con el *solvePnP*. Esto ha sido conceptualmente muy difícil de entender, y sin documentación externa habría sido imposible hallarlo por mi cuenta.

Una vez hallada la posicion del la camara respecto al tag, para hallar la camara respecto al mundo, he empleado lo siguiente en el codigo: *cam_world = R_wt @ t_tc + tag_pos_world*, donde R_wt es una matriz de rotación que creo yo mismo cuando elijo al tag de mayor área, ya que la posición del tag, tiene 4 campos, y el último de ellos es el yaw, por lo que la matriz de rotación la puedo sacar como: (cos(yaw), -sin(yaw), 0; sin(yaw), cos(yaw), 0; 0, 0, 1).

Así puedo hallar la posición de la camara respecto al mundo, al menos teóricamente, y también hallo el yaw de la cámara respecto al mundo, para poder suponer no solo la posición, sino también la orientación.

### 4º par

