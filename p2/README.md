# Práctica 2. SIGUELINEAS SIMPLE
  
Esta práctica tenía como objetivo diseñar un formula 1 que siguese una linea dentro de un circuito de manera reactiva. La complejidad venía de que el unico sensor que teníamos para guiarnos era la cámara. Por ello, en cada iteración del bucle tenemos que sacar la imagen, después filtrar su color para quedarnos solo con la línea, y comprobar si vemos o no la linea.

Para completar esta práctica, la he abordado en 3 fases.

### 1º fase: filtrado de imágenes

Esta parte ha sido un trabajo primordialmente de búsqueda en internet. Una vez obtenida la imagen con la funcion *HAL.getImage()*, teníamos que filtrarla de alguna manera en la que pudiese sacar de ella un punto el cual representaría la línea que quiero seguir.

Al principio opté por hacer una lista de colores más y menos intensos, pero descubrí que se puede aplicar un filtro hsv a una imagen de openCV, dejando visibles solo los tonos rojos de la imagen (linea) y el resto en negro.

El otro objetivo de esta parte es sacarse el centro de los datos obtenidos en la imagen. Para ello usé una función de la librería _numpy_ la cual te devuelve un array de columnas de una imágen (en pixeles) en las que hay un color distinto del negro. 
Si calculo la media de estas columnas, obtenemos mas o menos donde se sitúa la línea en el eje x en ese instante.

Debido a que analizo toda la imágen, puedo preveer los giros antes de pasar por ellos. Esto tiene sus ventajas y desventajas, que comentaré mas adelante.

### 2º fase: velocidad en w

Tenemos un sistema a controlar que es el propio coche. El objetivo es claro: lograr una vuelta en el menor tiempo posible sin que se salga de la linea. Para ello, he diseñado un controlador PD para la velocidad en w (giro del coche). 

En cada iteración del bucle guardo el centro de la linea y el centro de la propia imágen (Posición del coche). El error será la diferencia de ambos. Primero tuve que hallar las constantes Kp, para reducir el error proporcionalmete; y Kd para suavizar el giro si la derivada del error es pequeña, o para endurecerlo si es grande. Definimos la derivada del error como error actual - error previo.

Estos valores los elegí con una velocidad constante

### 3º fase: velocidad en v

Controlar la velocidad lineal del coche no supuso un problema, pero si lo supuso la combinación de los dos controladores. También empleé un PD para la velocidad, y como cada controlador actuaba de una forma, ya no eran solo dos constantes las que tenía que ajustar. Ahora eran 4.

Para hallarlas, empleé el método prueba y error, muchas veces, y para usar unos valores más exactos, en cada iteración del bucle normalizo el error entre -1 y 1 aprox, y lo multiplico por 2.5 para darle mas sensibilidad al giro.

Los valores que mejor me funcionaban eran una Kp no muy alta en el giro, para no sobreoscilar mucho o actuar antes de tiempo; y una Kp bastante alta en la velocidad para poder frenar lo suficiente cuando hubiese una curva.
Las constantes derivativas son considerablemente más reducidas que las proporcionales, ya que al ir a bastante velocidad, suavizar el giro solía suponer un choque inminente.

Este proceso de ajuste duró mucho tiempo, y tuve que "sacrificar" ciertas cosas. Diseñé varios controladores que seguían la linea muy bien, pero tenían un problema y es que no conseguí que fueran rápidos. También conseguí otros muy rapidos, que al no seguir la linea se chocaban en curvas muy cerradas, u otros que iban muy bien en unos circuitos porque las oscilaciones cuadraban justo con las curvas lo que permitía cogerlas muy bien, pero cuando cambiaba de circuito se chocaban a la primera.
Finalmentem en un acto de suerte, me quedé con el que es mi proyecto final. Un controlador que hace que el coche complete el circuito simple en unos 55 segundos sin chocarse. Este controlador realmente es poco estable, porque oscila bastante (no consigue coger muy bien las rectas) y se choca en 2 de los 4 circuitos, pero es el que mejor me ha funcionado. El truco está en que es capaz de entrar muy bien a las curvas cerradas, y ahí es donde gana tiempo, esto gracias a la parte de la velocidad lineal, que frena lo suficiente para que no se pase.

Como cosas a destacar de mi proyecto, aplico dos máscaras a la imagen original, cada una filtra un color para los dos tipos de lineas que tenemos en el circuito.
Otra cosa interesante es que me defino unas velocidades angular y lineal iniciales por si el coche inicia sin ver ninguna linea, que primero la busque y luego ya sigue el circuito.
Por último, cuento con un sistema de defensa si pierdo la linea. Primeramente el Kp de la velocidad va a hacer que mi coche vaya muy despacio, y para que pueda buscar la linea en un intervalo corto de tiempo, si me guardo la anterior velocidad angular que tenía y la doblo, puedo hacer que el coche gire bruscamente. Esto funciona solo en los dos primeros circuitos al tener curvas no tan cerradas.

Por último, aquí podemos ver una demostración de mi programa:

## Circuito simple

