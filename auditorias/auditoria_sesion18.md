# Bitácora de Requerimientos y Auditoría IA — Sesión 18
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Arreglos y Verificación de Límites  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Canul Llanes Leslie Tamara
**Matrícula:** 24531370 
**Track:** [ C | Rust ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Cálculo de tamaño de arreglo en memoria  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> El reto principal fue entender el porqué del "índice fuera de rango". En C, si tienes un arreglo de 5 elementos, los índices válidos son del 0 al 4. Por lo tanto, la validación en `indice_en_rango` debe usar `<` (menor estricto) y no `<=`.

> Además, la función `bytes_arreglo` debía retornar un `long` (porque el tamaño en bytes de un arreglo grande podría exceder el límite de un `int` estándar de 32 bits). Implementar la tabla de anchos con `strcmp` me hizo comprender cómo el compilador consulta su tabla de tipos para saber cuánta memoria reservar en el *stack*.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Indexación entera no negativa |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
CONTEXTO: Sesión 18 de Lenguajes y Autómatas II. Entorno C11 (GCC) con -Wall -Wextra -Werror. Uso de api_sesion18.h. Sin main().

ROL: Ingeniero experto en compiladores y manejo de memoria.

ESPECIFICACIÓN: 
- `bytes_arreglo(const char *tipo, int elementos)`: Retorna -1 si tipo es NULL, elementos < 0, o tipo desconocido. Los anchos son char=1, short=2, int/float=4, double/puntero=8. Retorna el total de bytes (ancho * elementos).
- `indice_en_rango(int indice, int elementos)`: Retorna 1 si 0 <= indice < elementos, de lo contrario 0.

OUTPUT: Código en C modular, sin main().
```

---

## 3. Código Devuelto por el Modelo
```c
/* ia_sesion18_c.c - Fase 2: El Cientifico */
#include "api_sesion18.h"
#include <string.h>

long bytes_arreglo(const char *tipo, int elementos) {
    if (!tipo || elementos < 0) return -1;

    long ancho = 0;
    if (strcmp(tipo, "char") == 0) ancho = 1;
    else if (strcmp(tipo, "short") == 0) ancho = 2;
    else if (strcmp(tipo, "int") == 0 || strcmp(tipo, "float") == 0) ancho = 4;
    else if (strcmp(tipo, "double") == 0 || strcmp(tipo, "puntero") == 0) ancho = 8;
    else return -1;

    return ancho * elementos;
}

int indice_en_rango(int indice, int elementos) {
    return (indice >= 0 && indice < elementos) ? 1 : 0;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Límites estáticos y dimensiones inválidas

* [✓] Caso base y límites superados sin *Crash*.
* [✓] No existen accesos fuera de límites ni *Memory Leaks*.
* [✓ ] La complejidad teórica coincide con la implementada.

=== AUDITORIA SESION 18 - Salida del Harness ===
Caso                               esperado:              obtenido:              resultado
[ 1] int[10]                            esperado: 40           obtenido: 40           OK
[ 2] double[3]                          esperado: 24           obtenido: 24           OK
[ 3] int[-4]                            esperado: -1           obtenido: -1           OK
[ 4] indice 0 de 5                      esperado: 1            obtenido: 1            OK
[ 5] indice 5 de 5                      esperado: 0            obtenido: 0            OK
[ 6] indice negativo                    esperado: 0            obtenido: 0            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion18.md

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> El problema: La IA resolvió el problema con un enfoque más "inteligente" y compacto, agrupando los tipos int y float con el operador ||. Sin embargo, omitió validar que la multiplicación en bytes_arreglo no cause un Integer Overflow (desbordamiento de entero). Si el compilador recibe un arreglo con un número de elementos extremadamente grande (por ejemplo, el límite de int), el cálculo del tamaño total de bytes podría desbordarse y devolver un número negativo.

> La solución (Auditoría): En mi versión del Artesano, al declarar long total = ancho * (long)elementos;, hice un casting explícito para forzar a que la multiplicación se realice en 64 bits y evitar el desbordamiento. La IA también lo resolvió al usar long ancho, pero de forma implícita. El caso borde real omitido por la IA es que no valida un límite máximo de elementos. Aunque el harness no evalúa esto, en un compilador real esto podría provocar que el programa intente reservar más memoria de la disponible en el sistema.
---
**Firma digital:** LeslieTamara14
