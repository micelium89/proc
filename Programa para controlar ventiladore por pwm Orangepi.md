# Codeium Chat Conversation

Note: _This is purely the output of the chat conversation and does not contain any raw data, codebase snippets, etc. used to generate the output._

**User** _at 3:56 pm, Jan 30, 2025_

Necesito crear un programa sencillo para regular de forma automática la velocidad del ventilador de una 'Orange Pi 3 lts' en función de la temperatura. Ha de usar alguna librería como lmsensors para saber la temperatura de la cpu y controlar la velocidad del ventilador por un pin GPIO. El objetivo es reducir el ruido del ventilador a bajas temperaturas y activarlo progresivamente de forma proporcional a la temperatura. Es necesario que sea ligero, sencillo e instalable desde un script en bash.

---

**Codeium** _at 3:57 pm, Jan 30, 2025_



¡Claro! Para crear un programa que controle la velocidad del ventilador en función de la temperatura de la CPU en una Orange Pi 3 LTS, puedes utilizar la librería `lm-sensors` para leer la temperatura y la librería `wiringPi` para controlar el pin GPIO del ventilador.

Aquí te dejo un ejemplo de código en C que puedes utilizar como base:
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <wiringPi.h>
#include <lm_sensors.h>

#define VENTILADOR_PIN 0 // Pin GPIO para el ventilador
#define TEMPERATURA_MAX 60 // Temperatura máxima para activar el ventilador
#define TEMPERATURA_MIN 40 // Temperatura mínima para desactivar el ventilador

int main() {
    // Inicializar wiringPi
    wiringPiSetup();

    // Inicializar lm-sensors
    sensors_init(NULL);

    while (1) {
        // Leer la temperatura de la CPU
        int temperatura = sensors_get_temp(SENSORS_SUBFEATURE_TEMP_CORE);

        // Controlar la velocidad del ventilador en función de la temperatura
        if (temperatura > TEMPERATURA_MAX) {
            // Activar el ventilador al 100%
            digitalWrite(VENTILADOR_PIN, HIGH);
        } else if (temperatura < TEMPERATURA_MIN) {
            // Desactivar el ventilador
            digitalWrite(VENTILADOR_PIN, LOW);
        } else {
            // Controlar la velocidad del ventilador de forma proporcional
            int velocidad = (temperatura - TEMPERATURA_MIN) * 100 / (TEMPERATURA_MAX - TEMPERATURA_MIN);
            analogWrite(VENTILADOR_PIN, velocidad);
        }

        // Esperar un poco antes de volver a leer la temperatura
        delay(1000);
    }

    return 0;
}
```
Este código utiliza la función `sensors_get_temp` para leer la temperatura de la CPU y la función `digitalWrite` para controlar el pin GPIO del ventilador. La función `analogWrite` se utiliza para controlar la velocidad del ventilador de forma proporcional.

Para compilar y ejecutar este código, necesitarás instalar las librerías `lm-sensors` y `wiringPi` en tu Orange Pi 3 LTS. Puedes hacer esto utilizando el siguiente comando:
```bash
sudo apt-get install lm-sensors wiringpi
```
Una vez que hayas instalado las librerías, puedes compilar el código utilizando el siguiente comando:
```bash
gcc -o ventilador ventilador.c -lwiringPi -llm_sensors
```
Finalmente, puedes ejecutar el programa utilizando el siguiente comando:
```bash
sudo ./ventilador
```
Recuerda que debes ejecutar el programa con permisos de root para que pueda acceder a los pines GPIO.

Espero que esto te ayude a crear un programa que controle la velocidad del ventilador en función de la temperatura de la CPU en tu Orange Pi 3 LTS. ¡Buena suerte!