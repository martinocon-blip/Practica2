# Práctica 2 - Interrupciones GPIO en ESP32

## Objetivo

El objetivo de esta práctica es comprender el funcionamiento de las interrupciones hardware en el ESP32 utilizando una entrada GPIO.

Se implementa un sistema basado en un pulsador conectado al GPIO18. Cada vez que el usuario pulsa el botón, se genera una interrupción que ejecuta una rutina de servicio (ISR), incrementando un contador y mostrando información por el puerto serie.

---

# Introducción teórica

Una interrupción es un mecanismo que permite al microcontrolador detener temporalmente la ejecución normal del programa para atender un evento importante.

En esta práctica se utilizan interrupciones GPIO, que se activan cuando cambia el estado eléctrico de un pin.

El ESP32 permite asociar una función ISR a cualquier pin GPIO mediante la función:

```cpp
attachInterrupt(GPIOPin, ISR, Mode);
```

En este caso:

- `GPIOPin` → GPIO18
- `ISR` → función `isr()`
- `Mode` → `FALLING`

El modo `FALLING` indica que la interrupción se dispara cuando la señal pasa de HIGH a LOW.

---

# Material utilizado

- ESP32-S3-DevKitC-1
- Pulsador
- Visual Studio Code
- PlatformIO
- Framework Arduino

---

# Configuración del proyecto

## Archivo `platformio.ini`

```ini
[env:esp32-s3-devkitc-1]
platform = espressif32
board = esp32-s3-devkitc-1
framework = arduino
monitor_speed = 115200
```

---

# Código implementado

## Archivo `main.cpp`

```cpp
#include <Arduino.h>

struct Button {
  const uint8_t PIN;
  uint32_t numberKeyPresses;
  bool pressed;
};

Button button1 = {18, 0, false};

void IRAM_ATTR isr() {
  button1.numberKeyPresses += 1;
  button1.pressed = true;
}

void setup() {

  Serial.begin(115200);

  pinMode(button1.PIN, INPUT_PULLUP);

  attachInterrupt(button1.PIN, isr, FALLING);
}

void loop() {

  if (button1.pressed) {

      Serial.printf(
        "Button 1 has been pressed %u times\n",
        button1.numberKeyPresses
      );

      button1.pressed = false;
  }

  // Desconectar interrupción después de 1 minuto
  static uint32_t lastMillis = 0;

  if (millis() - lastMillis > 60000) {

    lastMillis = millis();

    detachInterrupt(button1.PIN);

    Serial.println("Interrupt Detached!");
  }
}
```

---

# Funcionamiento del programa

El programa configura el GPIO18 como entrada utilizando la resistencia interna `INPUT_PULLUP`.

Cuando el botón es pulsado:

1. El pin cambia de HIGH a LOW.
2. Se genera una interrupción.
3. El ESP32 ejecuta automáticamente la función `isr()`.
4. La ISR incrementa el contador de pulsaciones.
5. Se activa la variable `pressed`.

Posteriormente, en el `loop()`, el programa detecta el cambio y muestra por puerto serie el número total de pulsaciones realizadas.

Después de 60 segundos, la interrupción se desconecta automáticamente mediante:

```cpp
detachInterrupt(button1.PIN);
```

A partir de ese momento, el botón deja de generar interrupciones.

---

# Explicación de elementos importantes

## ISR (Interrupt Service Routine)

```cpp
void IRAM_ATTR isr()
```

Esta función se ejecuta automáticamente cuando ocurre la interrupción.

La etiqueta `IRAM_ATTR` indica que la función se almacene en memoria RAM interna para ejecutarse más rápido.

---

## INPUT_PULLUP

```cpp
pinMode(button1.PIN, INPUT_PULLUP);
```

Activa la resistencia pull-up interna del ESP32.

Esto mantiene el pin normalmente en HIGH y evita estados flotantes.

---

## attachInterrupt()

```cpp
attachInterrupt(button1.PIN, isr, FALLING);
```

Asocia la interrupción al GPIO18 utilizando el flanco descendente (`FALLING`).

---

## detachInterrupt()

```cpp
detachInterrupt(button1.PIN);
```

Desactiva completamente la interrupción del pin.

---

# Diagrama de flujo

```text
            ┌──────────────┐
            │   Inicio     │
            └──────┬───────┘
                   │
                   ▼
      ┌────────────────────────┐
      │ Inicializar puerto     │
      │ serie                  │
      └──────────┬─────────────┘
                 │
                 ▼
      ┌────────────────────────┐
      │ Configurar GPIO18      │
      │ como INPUT_PULLUP      │
      └──────────┬─────────────┘
                 │
                 ▼
      ┌────────────────────────┐
      │ Activar interrupción   │
      │ FALLING                │
      └──────────┬─────────────┘
                 │
                 ▼
        ┌──────────────────┐
        │ Esperar pulsador │
        └────────┬─────────┘
                 │
        Pulsación detectada
                 │
                 ▼
      ┌────────────────────────┐
      │ Ejecutar ISR           │
      │ Incrementar contador   │
      └──────────┬─────────────┘
                 │
                 ▼
      ┌────────────────────────┐
      │ Mostrar contador por   │
      │ puerto serie           │
      └──────────┬─────────────┘
                 │
                 ▼
      ┌────────────────────────┐
      │ ¿Han pasado 60 s?      │
      └──────────┬─────────────┘
                 │ Sí
                 ▼
      ┌────────────────────────┐
      │ detachInterrupt()      │
      └────────────────────────┘
```

---

# Salida obtenida por puerto serie

Ejemplo de salida observada:

```text
Button 1 has been pressed 1 times
Button 1 has been pressed 2 times
Button 1 has been pressed 3 times
Button 1 has been pressed 4 times
Interrupt Detached!
```

---

# Ventajas de las interrupciones

Las interrupciones permiten reaccionar inmediatamente a eventos externos sin necesidad de comprobar continuamente el estado de un pin.

Esto mejora:

- La eficiencia del procesador.
- La velocidad de respuesta.
- La organización del código.

---

# Conclusiones

En esta práctica se ha aprendido:

- El funcionamiento de las interrupciones GPIO.
- Cómo asociar una ISR a un pin.
- El uso de `attachInterrupt()`.
- El uso de `detachInterrupt()`.
- La utilidad de `INPUT_PULLUP`.
- Cómo utilizar interrupciones para detectar eventos externos.

Además, se ha comprobado cómo el ESP32 puede responder de forma inmediata a una pulsación sin necesidad de utilizar polling continuo.