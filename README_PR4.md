# PR4_PD_XavierHidalgo

## PRACTICA 4:  SISTEMAS OPERATIVOS EN TIEMPO REAL

### Ejercicio Practico 1

**PROGRAMA:**

``` cpp
void setup()
{
Serial.begin(112500);
/* we create a new task here */
xTaskCreate(
anotherTask, /* Task function. */
"another Task", /* name of task. */
10000, /* Stack size of task */
NULL, /* parameter of the task */
1, /* priority of the task */
NULL); /* Task handle to keep track of created task */
}
/* the forever loop() function is invoked by Arduino ESP32 loopTask */
void loop()
{
Serial.println("this is ESP32 Task");
delay(1000);
}
/* this function will be invoked when additionalTask was created */
void anotherTask( void * parameter )
{
/* loop forever */
for(;;)
{
Serial.println("this is another Task");
delay(1000);
}
/* delete a task when finish,
Practica4.MD 2024-03-12
6 / 6
this will never happen because this is infinity loop */
vTaskDelete( NULL );
}
```

**INFORME:**

En este programa nos encontramos con dos tareas sin fin a ejecutar. Ambas imprimen un mensaje por la salida série.

La tarea principal y la que será la primera en ejecutarse, se encuentra en el void_loop(), el cual se ejecuta indefinidamente (por definición) e imprime "this is ESP32 Task" y, tras ello, espera 1 segundo. 

La segunda tarea la definimos en el void_setup() gracias a la función xTaskCreate: 
Esta función no nos permite elegir núcleo de la CPU del processador en el que se ejecute el programa de la tarea, como sí lo hace la función xTaskCreatePinnedToCore, así que FreeRTOS trabajará con un solo núcleo (tengo entendido que el ESP32 tiene dos). De esta forma vemos que realmente las dos tareas no se ejecutaran simultáneamente (ya que estaremos usando solamente un núcleo), así que FreeRTOS organizará y decidirá cuando se ejecutan las treas para que parezca que lo están haciedo simultáneamente. 

Si observamos la salida de nuestro programa, no parece que las tareas se ejecuten simultáneamente sino consecutivamente, hecho que contradice lo dicho en el párrafo anterior, más adelante explico porqué.

Bien, vamos a ver que hace la segunda tarea. Como he dicho, la definimos en el void_setup() mediante la función xTaskCreate y, definimos, entre otras cosas, el nombre de la tarea ("another Task") y la prioridad ("1"). Como solo tendremos una tarea "auxiliar", podemos definir la prioridad que queramos (de 1 a 24) ya que no habrá ninguna más prioritaria que esta.
Al final del programa principal, vemos el programa de nuestra segunda tarea. Consiste en un bucle infinito (como el de la tarea principal) que escribimos gracias a la instancia for(;;){ }. Dentro del bucle se imprime "this is another Task" a través del puertyo série y, tras ello, espera 1 segundo. Como veis casi lo mismo que la otra tarea.

Antes he dicho que en la salida de nuestro programa, no parece que las tareas se ejecuten simultáneamente sino consecutivamente, por qué? Esto se debe al delay que ejecutamos tras imprimir un mensaje. FreeRTOS no es capaz de omitir la función delay() para ejecutar otras tareas, así que, estamos obligados a esperar un segundo entre los mensajes, hecho que hace que parezca que no estamos utilizando el multitasking.
Yo sé de dos posibles soluciones a este problema. Una es definir la tarea "auxiliar" a partir de la función xTaskCreatePinnedToCore (ya explicada) y, ejecutarla en otro núcleo distinto de la principal (si es que el ESP32 tiene más de uno). Otra es definir los delays con la función vTaskDelay(), que permite que el procesador trabaje en otras tareas mientras espera.

En conclusión, si ejecutamos el programa dado, veremos que se imprime por la salida série lo siguiente:
```
this is ESP32 Task - (espera 1 segundo) - this is another Task - (espera 1 segundo) - this is ESP32 Task - (espera 1 segundo) - etc.
```

Si aplicamos alguna de las soluciones nombradas:
```
this is ESP32 Task -  this is another Task - (espera 1 segundo) - this is ESP32 Task -  this is another Task - (espera 1 segundo) - etc.
```

Vemos que el primer delay (tras enviar "this is ESP32 Task"), no es problema en la ejecución instantanea de la segunda tarea ya que, o se esta procesando en otro núcleo, o se está ulitizando la función vTaskDelay().
