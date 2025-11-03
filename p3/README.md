# PRACTICA 3 - VFF
Para esta practica, he tenido que implementar desde cero una tecnica de navegacion local: VFF. Esta consiste en la conduccion del coche unicamente mediante la informacion que proporcionan los sensores (laser), y la secuencia de subobjetivos que nos proporciona la capa de navegacion global, de la cual no tenemos ni idea, solo nos interesan esos subobjetivos.
En el desarrollo he tenido mis complicaciones, por lo que voy a ir paso a paso explicando el funcionamiento de mi código y como he abordado los problemas que me han surgido.

### 1º VECTOR ATRACTIVO
Sacar el vector atractivo ha sido relativamente sencillo. Gracias a la función que hay en la guia de ayuda de la práctica, he podido pasar de coordenadas globales a polares (respecto al coche). Una vez hecho eso, tuve que hacer que el vector se atenuase respecto con su distancia a la meta.
Para ello primero me saco la distancia hacia el objetivo local, como el modulo de los vectores x e y que hemos obtenido de nuestra funcion, y el vector atractivo será _[x * 1/(distancia +1), y * 1/(distancia +1)]_.
Por lo tanto, el modulo de la fuerza atractiva total será: _∣∣atr∣∣= distancia / (1 +distancia)_ Con esto consigo que los objetivos muy lejanos tengan una fuerza atractiva que tienda a 1, y los muy cercanos tiendan a 0. 
Una vez normalizado entre 0 y 1 mi vector atractivo, me toca el repulsivo.

### 2º VECTOR REPULSIVO
Para sacar este vector, me he ayudado del código de VFF que desarrollamos en la asignatura arquitectura software para robots. No he empleado la funcion que nos proporciona la guia de ayuda ya que prefiero manipular directamente las medidas del laser.
La idea de ambas prácticas es la misma, tomando todas las medidas del laser, para ello construimos un vector con la suma de todas las distancias del, y la direeccion de ese vector será la opuesta a la que nos da la suma. Y para asegurarnos de que los obstaculos cercanos nos afectan mas, multiplicamos cada distancia por su inversa.
Una vez sacado ese vector, lo dividimos por el numero de mediciones que tenemos, y asi nos queda un vector repuslivo, normalizado también entre 0 y 1, pero como depende de la distancia a los objetivos, si tenemos alguno muy cercano, hace que el vector repulsivo sea mucho mayor, así podemos asegurarnos el no chocarse.

### 3º VECTOR TOTAL
Una vez tenemos nuestros vectores y según la teoría, el vector total = _alpha * f_atractiva + beta * f_repulsiva_. Como el objetivo principal es no chocarse, he elegido por prueba y error un alpha de 0.8 y una beta de 1.5. El comportamiento del coche es mas bien tirando a temerario, no tiene una gran velocidad, pero cumple su objetivo de no chocarse con obstaculos

### 4º CALCULO DE VELOCIDADES
Ahora normalizo mi fuerza total entre 0 y 1 para poder mandar las velocidades al coche respecto la fuerza total, que es lo que nos interesa. 
Primero saco una medida de que tan intensa es la fuerza, entre o y 1, luego calculo la velocidad deseada respecto a la diferencia la minima y la maxima y multiplicada por este factor de fuerza. Para poder disminuir la velocidad frente a la presencia de objetivos, saco del laser la distancia minima respecto a un objetivo. Si la distancia es muy grande, la velocidad va a ser mayor, si es muy pequeña, va a frenar más.
Por último suavizo la veocidad tomando muestras de la velocidad previa, para no ir a tirones

La velocidad angular depende de la componente y de mi fuerza resultante normalizada. Sé que es un calculo poco robusto, pero funcional. También la suavizo respecto a la velocidad angular anterior, y la limito entre -5 y 5

## PROBLEMAS
El principal problema ha sido el calculo de velocidades, primero las desarrollé guiandome unicamente de las componentes x e y de la resultante, pero vi que no era suficiente, sobre todo en la lineal, que solía comerme varios coches. Hice técnicas más rapidas, pero no lograba el descenso rápido de velocidades, por eso empecé a meterle filtros de casi todo tipo, y he acabado con un modelo que no es muy rapido, pero funcional. 
Otro problema son los obstaculos laterales que a veces no los detecta, y he intentando minimizar los choques con unas alpha y beta adecuadas.
También he tenido un problema. La practica se planteó como un conjunto de estados: objetivo delante, objetivo detras, y objetivo en el campo visual. Intenté hacerlo de esta forma, es decir, cuando el angulo del coche respecto a la meta sea mayor de 90, girar y 0 velocidad, pero inmediatamente cuando era 90 intentaba seguir al objetivo y se chocaba contra la pared. Probé con un rango menor (70-60) grados, y ya me funcionaba, pero en partes del circuito donde el objetivo estaba mas o menos a ese angulo, el coche se comportaba de manera indefinida.
Por ello decidí afrontarlo solo suponiendo que el coche siemrpe va a estar orientado hacia el objetivo.
Otra complicacion, y respecto a la anterior, hay momentos donde mi fuerza resultante es casi nula, o nula directamente. Como mi velocidad no esta calculada especificamente respecto a la normal, no he tenido el problema de quedarme quieto, simplemente oscila brevemente el coche.
En resumen, ha sido una práctica de mucha prueba y error, y con bastantes complicaciones, y aunque mi solucion no es la mas elegante, es la que mejor me ha funcionado

## VIDEO DEL FUNCIONAMIENTO
Como va un poco lento, recomiendo ponerlo en x2:

[Grabación de pantalla desde 2025-11-02 19-06-15.webm](https://github.com/user-attachments/assets/d61d2d73-38a5-45d4-a8f6-e183695ba627)
