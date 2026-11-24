# Bitácora de Requerimientos y Auditoría IA — Sesión 12
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Atributos de Símbolos y Memoria  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Canul Llanes Leslie Tamara
**Matrícula:** 24531370
**Track:** [ C | Rust ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Estructura de registro de símbolos  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> El mayor reto a nivel de lógica fue comprender la fórmula matemática para la alineación de memoria. Si el cursor está en la posición 1 y queremos meter un `int` (que ocupa 4 bytes y requiere estar alineado a múltiplos de 4), no podemos simplemente sumar. Necesitamos calcular el *padding* (relleno).

> Implementé la fórmula `padding = (ancho - (cursor % ancho)) % ancho` para evitar que, cuando el cursor ya esté perfectamente alineado (ej. cursor=4, ancho=4), le sume un bloque entero de 4 bytes por error. El doble módulo asegura que si ya está alineado, el padding sea 0.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Cálculo de desplazamientos y alineación |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
CONTEXTO: Sesión 12 de la materia Lenguajes y Autómatas II. Entorno C11 (GCC) con -Wall -Wextra -Werror. No incluir main(). Uso del archivo api_sesion12.h.

ROL: Ingeniero experto en compiladores y arquitectura de computadoras.

ESPECIFICACIÓN: 
- Implementar int ancho_de_tipo(const char *tipo) devolviendo 1 (char), 2 (short), 4 (int, float), 8 (double, puntero) o -1 si es inválido o NULL.
- Implementar int asignar_offset(Simbolo *simbolo, int cursor) que rechace NULL y cursores negativos devolviendo -1.
- Calcular el padding necesario para que el offset sea múltiplo del ancho del tipo, guardarlo en simbolo->offset y retornar el nuevo cursor (offset + ancho).

OUTPUT: Código en C modular, listo para compilar.
```

---

## 3. Código Devuelto por el Modelo
```c
/* ia_sesion12_c.c - Fase 2: El Cientifico */
#include "api_sesion12.h"
#include <string.h>

int ancho_de_tipo(const char *tipo) {
    if (!tipo) return -1;
    if (strcmp(tipo, "char") == 0) return 1;
    if (strcmp(tipo, "short") == 0) return 2;
    if (strcmp(tipo, "int") == 0) return 4;
    if (strcmp(tipo, "float") == 0) return 4;
    if (strcmp(tipo, "double") == 0) return 8;
    if (strcmp(tipo, "puntero") == 0) return 8;
    return -1;
}

int asignar_offset(Simbolo *simbolo, int cursor) {
    if (!simbolo || cursor < 0) return -1;
    
    int w = ancho_de_tipo(simbolo->tipo);
    if (w == -1) return -1;
    
    int offset = (cursor + w - 1) & ~(w - 1);
    simbolo->offset = offset;
    
    return offset + w;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Alineación de tipos y tamaños en bytes

* [✔] Caso base y límites superados sin *Crash*.
* [✔] No existen accesos fuera de límites ni *Memory Leaks*.
* [✔] La complejidad teórica coincide con la implementada.

Salida del Harness
=== AUDITORIA SESION 12 - Registro de simbolo y offsets alineados ===
Caso                               esperado:              obtenido:              resultado
[ 1] ancho_de_tipo("int")               esperado: 4            obtenido: 4            OK
[ 2] ancho_de_tipo("double")            esperado: 8            obtenido: 8            OK
[ 3] int en cursor 0                    esperado: 4            obtenido: 4            OK
[ 4] char e int: offset de b            esperado: 4            obtenido: 4            OK
[ 5] double desde cursor 4              esperado: 8            obtenido: 8            OK
[ 6] registro NULL                      esperado: -1           obtenido: -1           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion12.md

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> El problema: La IA no cometió errores funcionales en los casos del harness, pero implementó una técnica de alineación mediante mascaramiento de bits ((cursor + w - 1) & ~(w - 1)). Esta técnica es muy rápida, pero tiene un caso borde crítico: solo funciona si el ancho w es una potencia de 2 (1, 2, 4, 8).

> La solución (Auditoría): Aunque en esta sesión los tipos cumplen esa regla, en la arquitectura real de un compilador podrían existir tipos con anchos no potencias de 2 (por ejemplo, estructuras empaquetadas o tipos de 12 bytes). La IA fallaría con esos valores, calculando offsets negativos o desalineados. En la versión del Artesano utilicé la fórmula general con módulo, que funciona para cualquier ancho y es la que usaría en un compilador real.

---
**Firma digital:** LeslieTamara14
