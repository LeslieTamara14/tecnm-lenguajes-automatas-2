# Bitácora de Requerimientos y Auditoría IA — Sesión 16
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Firmas de Función y Parámetros  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Canul Llanes Leslie Tamara
**Matrícula:** 24531370
**Track:** [ C | Rust ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Registro de prototipos y aridad  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> El mayor reto a nivel lógico fue entender que la función tiene **tres estados de retorno** distintos, no solo dos. A diferencia de un simple booleano (verdadero/falso), aquí debía distinguir entre "la función no existe" (-1), "la función existe pero le pasaste mal el número de argumentos" (0), y "la llamada es válida" (1).

> También fue crucial validar el parámetro `nargs < 0`, ya que en C es fácil que un ciclo o una variable no inicializada pase un número negativo; detectar esto antes de buscar en el catálogo evita comportamientos indefinidos.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Coincidencia estricta de parámetros |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
CONTEXTO: Sesión 16 de Lenguajes y Autómatas II. Entorno C11 (GCC) con -Wall -Wextra -Werror. Uso de api_sesion16.h (Estructura Funcion con nombre y aridad). Sin main().

ROL: Ingeniero experto en compiladores y análisis semántico.

ESPECIFICACIÓN: 
- Implementar int verificar_llamada(const Funcion *catalogo, int total, const char *nombre, int nargs).
- Devuelve -1 si catalogo o nombre son NULL, o si nargs es negativo.
- Recorre el catálogo buscando coincidencia exacta con strcmp.
- Si encuentra la función, devuelve 1 si catalogo[i].aridad == nargs, y 0 si no coincide.
- Si termina el ciclo sin encontrar la función, devuelve -1.

OUTPUT: Código en C modular, sin main().
```

---

## 3. Código Devuelto por el Modelo
```c
/* ia_sesion16_c.c - Fase 2: El Cientifico */
#include "api_sesion16.h"
#include <string.h>

int verificar_llamada(const Funcion *catalogo, int total, const char *nombre, int nargs) {
    if (!catalogo || !nombre || nargs < 0) {
        return -1;
    }

    for (int i = 0; i < total; i++) {
        if (strcmp(catalogo[i].nombre, nombre) == 0) {
            return (catalogo[i].aridad == nargs) ? 1 : 0;
        }
    }

    return -1;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Llamadas con número incorrecto de args

* [✓] Caso base y límites superados sin *Crash*.
* [✓] No existen accesos fuera de límites ni *Memory Leaks*.
* [✓] La complejidad teórica coincide con la implementada.

=== AUDITORIA SESION 16 - Registro de prototipos y aridad ===
Caso                               esperado:              obtenido:              resultado
[ 1] suma con 3 argumentos              esperado: 1            obtenido: 1            OK
[ 2] suma con 2 argumentos              esperado: 0            obtenido: 0            OK
[ 3] suma con 4 argumentos              esperado: 0            obtenido: 0            OK
[ 4] funcion no registrada              esperado: -1           obtenido: -1           OK
[ 5] main sin argumentos                esperado: 1            obtenido: 1            OK
[ 6] nombre NULL                        esperado: -1           obtenido: -1           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion16.md

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> El problema: La IA resolvió la lógica del algoritmo a la perfección, e incluso optimizó el código con un operador ternario. Sin embargo, omitió validar el caso en el que total sea un número negativo.

> La solución (Auditoría): Aunque el harness no prueba este caso (asume que el programador pasa un tamaño de arreglo válido), en un compilador real, si se recibe un total negativo por un error en otra parte del sistema, el ciclo for (int i = 0; i < total; i++) simplemente no se ejecutaría y devolvería -1 (función no encontrada), lo cual enmascara el error real.
En mi versión del Artesano tampoco validé total < 0 porque el contrato no lo exigía, pero lo anoto como un caso borde pendiente para hacer el sistema completamente robusto.

---
**Firma digital:** LeslieTamara14
