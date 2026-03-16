[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/KzqfxGd5)
[![Open in Visual Studio Code](https://classroom.github.com/assets/open-in-vscode-2e0aaae1b6195c2367325f4f02e2d04e9abb55f0b24a779b69b11b9e10269abc.svg)](https://classroom.github.com/online_ide?assignment_repo_id=22830533&assignment_repo_type=AssignmentRepo)
# Lab02 - Caracterización de osciladores (externo vs. interno)


## 1. Integrantes
### •[Laura Alejandra Florian Orjuela](https://github.com/Laura-Florian)

### •[Daniel Penagos Castro](https://github.com/Daniel-Penagos)

## 2. Documentación

### 2.1 Descripción del laboratorio
♥ En este laboratorio usaremos una pickit 3,una pic 18F45K22,resistencias, cristal de cuarzo de 16MHz y condensadores de 22pF.

### ♠ Conexiones para INTOSC y HS:

Para nuestoro circuito INTOSC y HS conectamos las paticas de nuestro Cristal de cuarzo de 16MHz en los pines 13(OSC1/CLK)entrada y 14(OSC2/CLK) salida, despues conectamos en cada una de las paticas un condensador de 22pF en el pin 12 (12)tierra  y por ultimo conectamos una resistencia desde el anodo que ira en el pin 15(RC0/T1OSO/T1CLC)salida y en el catodo una resistencia de 1kOhm.
Medimos la frecuancia del Cristal en la patica que tenemos conectada en el pin 14, y en la del led en en el anodo.

### ♠ Conexiones para RC:

Para nuestoro circuito RC conectamos una resistencia de 1kOhm en el pin 12(Vss)tierra junto a el un condensador de 22pF una patica desde el pin 12(Vss)tierra y la otra al pin 13(OSC1/CLK)entrada y el pin 14(OSC2/CLK) lo dejamos "libre",  y por ultimo conectamos una resistencia desde el anodo que ira en el pin 15(RC0/T1OSO/T1CLC)salida y en el catodo una resistencia de 1kOhm.
Medimos la frecuancia de la resistencia junto con el condensador desde el pin 14, y el led nos mostrara si esta funcionando.

### 2.2 Explicación del código implementado

#include <xc.h>        // Librería principal del compilador XC8 para microcontroladores PIC

#include <stdint.h>    // Librería para usar tipos de datos estándar como uint16_t

// ========================== CONFIGURACIÓN GENERAL ========================
// Estas configuraciones se graban en el PIC cuando se programa el microcontrolador

#pragma config WDTEN = OFF      // Desactiva el Watchdog Timer (evita reinicios automáticos)
#pragma config LVP = OFF        // Desactiva la programación en bajo voltaje
#pragma config PBADEN = OFF     // Hace que los pines del puerto B inicien como digitales
#pragma config CP0 = OFF, CP1 = OFF, CP2 = OFF, CP3 = OFF  // Desactiva protección de código
#pragma config BOREN = OFF      // Desactiva el Brown-out Reset (reinicio por bajo voltaje)
#pragma config FCMEN = OFF      // Desactiva el monitor de fallo del reloj
#pragma config IESO = OFF       // Desactiva cambio automático entre osciladores

// ========================== MODO DE OSCILADOR ==========================
// Aquí se define qué tipo de reloj usará el microcontrolador

// 1 = Oscilador interno del PIC
// 2 = Cristal externo de alta velocidad
// 3 = Oscilador RC externo (resistencia + capacitor)

#define MODE 1      // En este código se selecciona el modo 1 (oscilador interno)

#if MODE == 1
#pragma config FOSC = INTIO67   // Configura el PIC para usar el oscilador interno
#define USE_PLL 0               // No usar multiplicador de frecuencia PLL

#elif MODE == 2
#pragma config FOSC = HSHP      // Configura el PIC para usar cristal externo de alta velocidad
#define USE_PLL 0               // No usar PLL

#elif MODE == 3
#pragma config FOSC = RC        // Configura el PIC para usar oscilador RC externo
#define USE_PLL 0               // No usar PLL

#else
#error "Modo de oscilador inválido" // Si se define un modo incorrecto, el compilador genera error
#endif


// ========================== FRECUENCIA DEL OSCILADOR =====================
// Esta parte define la frecuencia del reloj que usará el compilador para calcular delays

#if MODE == 1 || MODE == 2      // Si el modo es interno o cristal externo

#if USE_PLL
#define _XTAL_FREQ 64000000UL   // Si se usara PLL la frecuencia sería 64 MHz
#else
#define _XTAL_FREQ 16000000UL   // Frecuencia de trabajo del microcontrolador = 16 MHz
#endif

#else                           // Si el modo es RC externo
#define _XTAL_FREQ 16000000UL   // Frecuencia supuesta (puede no ser exacta)
#endif


// ========================== FUNCIONES ==========================

// Función para generar retardos en milisegundos
void delay_ms(uint16_t ms) {

    while(ms--) {               // Repite el ciclo ms veces
        __delay_ms(1);          // Retardo de 1 milisegundo en cada iteración
    }

}


// ========================== CONFIGURACIÓN DE PINES ==========================

void init_pins(void) {

    // Configuración del pin RC0 como salida
    TRISCbits.TRISC0 = 0;       // 0 = salida, 1 = entrada
    LATCbits.LATC0 = 0;         // Inicializa RC0 en estado bajo (0V)

    // Configuración opcional del pin RA6
    // RA6 puede usarse como salida de reloj (CLKO) dependiendo del modo

    if(MODE == 1 || (MODE == 2 && USE_PLL)) {

        TRISAbits.TRISA6 = 0;   // Configura RA6 como salida
        LATAbits.LATA6 = 0;     // Inicializa RA6 en nivel bajo

    }

}


// ========================== CONFIGURACIÓN DEL OSCILADOR ==========================

void init_oscillator(void)
### 2.3 Análisis y comparación

#### Tabla 1: Medición en frío

| Modo de oscilador | Freq. teórica Fosc | RA6 medible (CLKO)? | Freq. medida RA6 (Hz) | Freq. teórica RC0 (Hz)| Freq. medida RC0 (Hz) | Error RC0 (%) |  
|------------------|------------------|---------------------|---------------|---------------------|---------------|---------------|
| INTOSC (interno) | 16,000,000       | Sí                 |                     |                500                 |               |               | |
| HS (cristal externo 16 MHz) | 16,000,000 | No |     NA      |               500                 |               |               |
| RC externo       | ~16,000,000*     | No                                    |       N/A        | 500                 |               |               | |

#### Tabla 2: Medición con calor

| Modo de oscilador | Freq. teórica Fosc | RA6 medible (CLKO)? | Freq. medida RA6 (Hz) | Freq. teórica RC0 (Hz)| Freq. medida RC0 (Hz) | Error RC0 (%) |  
|------------------|------------------|---------------------|---------------|---------------------|---------------|---------------|
| INTOSC (interno) | 16,000,000       | Sí                 |                     |                500                 |               |               | |
| HS (cristal externo 16 MHz) | 16,000,000 | No |     NA      |               500                 |               |               |
| RC externo       | ~16,000,000*     | No                                    |       N/A        | 500                 |               |               | |

#### Tabla 3: Deriva

| Modo de oscilador |RC0 deriva (Hz) |
|------------------|--------------------|
| INTOSC (interno) |                    |                
| HS (cristal externo 16 MHz) |                |                |
| RC externo       |                 |                


<!-- Agregar tablas para valores usando PLL -->

<!-- Complemente con análisis de lo registrado en tablas -->

## 2.4 Diagramas
◘Diagrama RC

[Ver diagrama RC](diagramas/Circuito_RC.png)

◘Diagrama INTOSC y HS

[Ver diagrama INTOSC y HS](diagramas/Circuito_INTOSC_HS.png)

## 2.5 Formas de onda

### INTOSC (interno) 
[Ver Forma de onda INTOSC](formasdeonda/INTOSC_1.jpeg)
### HS
[Ver Forma de onda HS](formasdeonda/HS_Cristal.jpeg)

[Ver Forma de onda RC0](formasdeonda/HS_RC0.jpeg)
### RC
[Ver Forma de onda RC](formasdeonda/RC_1.jpeg)

## 3. Evidencias de implementación
♣ RC

[Ver video RC](videos/Circuito_RC.mp4)

♣ INTOSC/HS

[Ver video INTOSC/HS ](videos/Circuito_Osiladores.mp4)


## 4. Preguntas

* ¿En qué modo se obtuvo la medición más cercana a la frecuencia teórica?

   ○ En modo 2, con oscilador externo mediante un cristal de cuarzo de 16MHz.

* ¿Fue posible evidenciar el fenómeno de deriva? ¿Qué factores podrían explicar la variación de frecuencia al calentar el PIC?

   ○ Se pudo evidenciar solo en modo 1 y 2, con muy poca variación y en modo 3 no se pudo evidenciar por la inestabilidad.

* ¿Cuál es más preciso en cuanto a frecuencia teórica vs. medida?

   ○ El más preciso fue el modo 2 con valores muy cercanos a los teóricos y poca variación.


* Explique cómo usar RC0 para estimar la frecuencia del oscilador cuando RA6 no está disponible.

   ○ Los modos en los que aplica PLL son el modo 1 de oscilador interno y el modo 2 de oscilador externo con cristal de cuarzo.

* Si se quisiera duplicar la frecuencia del PIC usando PLL, ¿en qué modos se podría aplicar?

   ○ El modo 1 es el más "accesible" ya que el oscilador está integrado en el mismo microcontrolador,  aunque con una frecuencia limitada de 31kHz.

* Enliste ventajas y desventajas de cada modo.
  
  ○ El modo 2 es el más adecuado por su estabilidad y una frecuencia superior debido al cristal de cuarzo de 16MHZ, aunque es externo al microcontrolador.

  ○ El modo 3 permite "variar" la frecuencia de oscilación al cambiar los valores de resistencia y capacitancia, pero suele ser muy inestable al mantener frecuencia bajas.

## 5. Referencias

[1]	“Pic18f45k22 pdf”, Alldatasheet.com. [En línea]. Disponible en: https://www.alldatasheet.com/datasheet-pdf/view/348759/MICROCHIP/PIC18F45K22.html. [Consultado: 16-mar-2026].

[2]	J. Ramirez, labs/02_lab02 at main · jharamirezma/Microcontroladores_ECCI_2026-I. .
