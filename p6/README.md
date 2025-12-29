# P6 - AUTOLOCALIZACIÓN CON APRILTAGS

Esta práctica ha sido sin duda alguna la más difícil, ya que conceptualmente es fácil pero he tenido que buscar por mi cuenta cómo hacer el PnP y la multiplicación de matrices. Además, el laser y el bumper del robot no estaban habilitados, así que he tenido que usar odometría para ciertas partes.

### 1º parte: navegación

He modificado el algoritmo de navegación aleatoria de la P1 implementando una máquina de 2 estados. Primeramente me calculo un tiempo aleatorio entre 1 y 5 segundos y empiezo avanzando recto. En cada iteración compruebo si ya han pasado ese tiempo y, cuando sea así, calculo otro tiempo random pero esta vez para girar y así sucesivamente.

Para simplificar, los giros siempre son hacia la izquierda. No es la solución más elegante, pero funciona de manera consistente y evita que el robot quede atrapado en rectas.

### 2º parte: localización de balizas

En todas las iteraciones detecto todos los AprilTags en la imagen y me quedo con el de mayor área, que suele ser el más cercano. Una vez hallado este tag, obtengo su posición en el mundo gracias al ID del tag y la lista de posiciones de los tags proporcionada en el archivo YAML. 

Siempre que haya al menos un tag en la imagen, guardo la posición en ese momento del robot, y eso me servirá para la autolocalizacion sin balizas, que explicaré mas adelante.

### 3º parte: PnP y matrices homogéneas

Para calcular la posición de la cámara respecto al mundo, primero calculo la posición del tag respecto a la cámara usando solvePnP de OpenCV. Esto me da:

- rvec: vector de rotación del tag respecto a la cámara.

- tvec: vector de traslación del tag respecto a la cámara.

A partir de rvec genero la matriz de rotación con *Rodrigues* y construyo una matriz homogénea 4x4, que combina rotación y traslación.

Para hallar la posición de la cámara respecto al tag, invierto esta matriz. Además, aplico una corrección de ejes porque OpenCV usa un sistema de coordenadas diferente al del robot. Esta corrección también es una matriz homogénea 4x4, que transforma las coordenadas de la cámara de OpenCV al sistema de coordenadas del robot.

Finalmente, multiplico la matriz de la posición del tag en el mundo T_world_tag por la matriz de la cámara respecto al tag y la matriz de corrección de ejes. Esto me da T_world_cam, la posición y orientación de la cámara en el mundo. El yaw de la cámara se obtiene con *atan2(T_world_cam[1,0], T_world_cam[0,0])* y le sumo 90° para alinear correctamente la orientación, ya que me dí cuenta de que esta corrección era necesaria para no solo autolocalizarse bien en cuanto a posición, sino también en cuanto a orientación.

Esta parte ha sido sin duda alguna la más dificil. Inicialmente, mis matrices eran 3x3, pero opté por hacerlas homogéneas, tal y como vimos en clase.
Lo que más me costó pillar fue la diferencia de ejes del robot y del simulador, ya que eran distintos, y eso hacía que mi robot se autolocalizase mal, aunque el PnP estuviese teóricamente correcto, por lo que depurando mis matrices de posición finales en distintos escenarios (robot alejandose y acercandose a la baliza, girando sobre su pripio eje con una baliza delante...) logré hallar la correción de ejes correcta

### 4º parte: autolocalización sin baliza

Cuando no se detecta ninguna baliza, uso la odometría para estimar la posición de la cámara. La posición actual se calcula como la última posición conocida con baliza más la diferencia de x, y y yaw obtenida desde odometría.

De esta manera, la autolocalización sigue funcionando aunque el robot no vea tags, y cuando detecta uno nuevamente, la posición se corrige automáticamente.

### 5º parte: resumen de matrices

En todo el código utilizo matrices homogéneas 4x4 para representar transformaciones (rotación + traslación).

- T_world_tag: posición y orientación del tag en el mundo.

- T_ct / T_tag_cam: posición del tag respecto a la cámara (tras inversión).

axes_correction: corrige la diferencia de ejes entre OpenCV y el sistema del robot.

T_world_cam: posición final de la cámara respecto al mundo.

Con esta combinación de matrices y la selección del tag más cercano, la autolocalización funciona correctamente.

## VIDEO
En mi demostración, la autolocalización es bastante correcta, aunque siempre hay un pequeño error, posiblemente por el error acomulado, o que interprete que el centro del robot es la cámara, por lo que estimo la posición de la camara en el mundo, y no la del robot en el mundo. 
Otro problema es que a grandes distancias, el error aumenta, y además, al calcularse la posición en cada iteración y no haber ningún tipo de suavizado, se puede observar ese ruido de la posición, no es una estimación suave.

En el video, se ve como el robot, cuando deja de detectar una baliza, se autolocaliza gracias al incremento de la odometría correctamente, y ahí el ruido es menor. Pero cuando detecta una baliza, como está haciendo cálculos constantemente, hay más ruido. Aun así las posiciones estimadas son bastante correctas en ambos casos

[Grabación de pantalla desde 2025-12-29 18-24-16.webm](https://github.com/user-attachments/assets/b1165662-fd84-414a-a739-e0fdb433d1a9)

