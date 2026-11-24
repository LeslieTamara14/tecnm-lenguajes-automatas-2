# Bitácora de Requerimientos y Auditoría IA — Sesión 17
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Comprobación Semántica de Retornos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Canul Llanes Leslie Tamara
**Matrícula:** 24531370
**Track:** [ C | Rust ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Validación de tipo de sentencia return  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> El reto principal aquí fue doble. Primero, la **compatibilidad de tipos**: en C, `void` no puede devolver nada, y un `int` no puede almacenar un `double` sin perder información (por eso el compilador lo marca como warning o error, dependiendo del estándar).

> Segundo, el **análisis de flujo de control**: si una función tiene 2 rutas lógicas (un `if` y un `else`) pero solo hay 1 `return` dentro del `if`, el `else` no está retornando, lo que genera un error de "ruta sin retorno". La lógica del Artesano compara primero los tipos, y solo si los tipos son correctos, evalúa las rutas.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Rutas de ejecución con retorno garantizado |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
CONTEXTO: Sesión 17 de Lenguajes y Autómatas II. Entorno C11 (GCC) con -Wall -Wextra -Werror. Uso de api_sesion17.h. Sin main().

ROL: Ingeniero experto en compiladores y análisis semántico.

ESPECIFICACIÓN: 
- Implementar int diagnosticar_retorno(...).
- Devuelve -1 si tipo_funcion o tipo_expresion son NULL, si rutas_totales <= 0, o si rutas_con_return < 0 o > rutas_totales.
- Devuelve 1 si el tipo de la expresión no es compatible con el tipo de la función (ej. "void" con un valor, "int" con "double").
- Devuelve 2 si las rutas_con_return son estrictamente menores que las rutas_totales.
- Devuelve 0 en caso de que todo sea válido.

OUTPUT: Código en C modular, sin main().
```

---

## 3. Código Devuelto por el Modelo
```c
/* ia_sesion17_c.c - Fase 2: El Cientifico */
#include "api_sesion17.h"
#include <string.h>

int diagnosticar_retorno(const char *tipo_funcion, const char *tipo_expresion, int rutas_totales, int rutas_con_return) {
    if (!tipo_funcion || !tipo_expresion) return -1;
    if (rutas_totales <= 0 || rutas_con_return < 0 || rutas_con_return > rutas_totales) return -1;

    if (strcmp(tipo_funcion, "void") == 0 && strcmp(tipo_expresion, "void") != 0) {
        return 1;
    }
    
    if (strcmp(tipo_funcion, "int") == 0 && strcmp(tipo_expresion, "double") == 0) {
        return 1;
    }

    if (rutas_con_return < rutas_totales) {
        return 2;
    }

    return 0;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Funciones sin retorno o retornos mixtos

* [✓] Caso base y límites superados sin *Crash*.
* [✓] No existen accesos fuera de límites ni *Memory Leaks*.
* [✓] La complejidad teórica coincide con la implementada.

=== AUDITORIA SESION 17 - Salida del Harness ===
Caso                               esperado:              obtenido:              resultado
[ 1] int con int y dos rutas            esperado: 0            obtenido: 0            OK
[ 2] void devuelve valor                esperado: 1            obtenido: 1            OK
[ 3] ruta sin retorno                   esperado: 2            obtenido: 2            OK
[ 4] double acepta int                  esperado: 0            obtenido: 0            OK
[ 5] int no acepta double               esperado: 1            obtenido: 1            OK
[ 6] cero rutas de ejecucion            esperado: -1           obtenido: -1           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion17.md

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> El problema: La IA omitió validar el tipo de retorno float y también tipos como char o short. En el diseño de compiladores, un float (4 bytes) no puede aceptar un double (8 bytes) sin una conversión explícita (que no se contempla en el contrato). Sin embargo, la IA solo validó "void" e "int" contra "double".

> La solución (Auditoría): Al revisar el código, me di cuenta de que la IA generalizó mal la validación. En la versión del Artesano, el código es más robusto porque agrupa los casos de double y int, pero le falta la validación explícita para cuando el tipo sea float. En un compilador real, esto requeriría una función aparte que compare los anchos de los tipos, pero para el alcance de esta sesión, los casos del harness quedaron cubiertos. Lo anoto como una omisión importante: la IA no valida los tipos de datos más allá de los que están en el harness.

---
**Firma digital:** LeslieTamara14
