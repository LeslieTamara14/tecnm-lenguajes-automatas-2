# Bitácora de Requerimientos y Auditoría IA — Sesión 19
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Estructuras (struct) y Miembros  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Canul Llanes Leslie Tamara
**Matrícula:** 24531370 
**Track:** [ C | Rust ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Offsets de miembros en estructuras  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> El mayor reto fue comprender el concepto de **padding** (relleno) y alineación natural. Un procesador lee la memoria en bloques de 4 u 8 bytes, por lo que si declaras un `char` (1 byte) y luego un `int` (4 bytes), el `int` no puede empezar justo después del `char` (posición 1), sino que el compilador inserta 3 bytes de relleno para que el `int` empiece en la posición 4 (múltiplo de su tamaño).

> Además, para calcular el `tamano_struct`, descubrí que el total también debe rellenarse al final (*padding* final) para que el arreglo de estructuras quede alineado en memoria. Utilicé la fórmula del doble módulo `(ancho - (offset % ancho)) % ancho` para calcular el relleno sin necesidad de usar condiciones. Cabe destacar que codifiqué una función auxiliar `ancho_de_tipo` para no repetir el `if-else` en ambas funciones, lo cual es una técnica básica de reutilización de código sin llegar a optimizaciones complejas. 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Alineación de memoria por palabras |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
CONTEXTO: Sesión 19 de Lenguajes y Autómatas II. Entorno C11 (GCC) con -Wall -Wextra -Werror. Uso de api_sesion19.h. Sin main().

ROL: Ingeniero experto en compiladores y arquitectura de memoria.

ESPECIFICACIÓN: 
- Implementar int offset_campo(const Campo *campos, int total, const char *nombre): Retorna el offset (alineado) del campo solicitado. Devuelve -1 si es nulo o no existe.
- Implementar int tamano_struct(const Campo *campos, int total): Retorna el tamaño total. Aplica padding final al múltiplo de la alineación mayor.
- Anchos: char=1, short=2, int/float=4, double/puntero=8.

OUTPUT: Código en C modular, sin main().
```

---

## 3. Código Devuelto por el Modelo
```c
/* ia_sesion19_c.c - Fase 2: El Cientifico */
#include "api_sesion19.h"
#include <string.h>

static int ancho(const char *t) {
    if (!t) return -1;
    if (!strcmp(t, "char")) return 1;
    if (!strcmp(t, "short")) return 2;
    if (!strcmp(t, "int") || !strcmp(t, "float")) return 4;
    if (!strcmp(t, "double") || !strcmp(t, "puntero")) return 8;
    return -1;
}

int offset_campo(const Campo *campos, int total, const char *nombre) {
    if (!campos || !nombre || total <= 0) return -1;
    int off = 0;
    for (int i = 0; i < total; i++) {
        int w = ancho(campos[i].tipo);
        if (w == -1) return -1;
        off = (off + w - 1) & ~(w - 1);
        if (!strcmp(campos[i].nombre, nombre)) return off;
        off += w;
    }
    return -1;
}

int tamano_struct(const Campo *campos, int total) {
    if (!campos || total < 0) return -1;
    if (total == 0) return 0;
    int off = 0, max_w = 1;
    for (int i = 0; i < total; i++) {
        int w = ancho(campos[i].tipo);
        if (w == -1) return -1;
        if (w > max_w) max_w = w;
        off = (off + w - 1) & ~(w - 1);
        off += w;
    }
    off = (off + max_w - 1) & ~(max_w - 1);
    return off;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Acceso a miembros inexistentes

* [✓] Caso base y límites superados sin *Crash*.
* [✓] No existen accesos fuera de límites ni *Memory Leaks*.
* [✓] La complejidad teórica coincide con la implementada.

=== AUDITORIA SESION 19 - Salida del Harness ===
Caso                               esperado:              obtenido:              resultado
[ 1] struct char e int: b               esperado: 4            obtenido: 4            OK
[ 2] struct de tres campos: c           esperado: 8            obtenido: 8            OK
[ 3] tamano total con padding           esperado: 12           obtenido: 12           OK
[ 4] double seguido de char             esperado: 16           obtenido: 16           OK
[ 5] miembro inexistente                esperado: -1           obtenido: -1           OK
[ 6] struct sin campos                  esperado: 0            obtenido: 0            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion19.md

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> El problema: La IA implementó el cálculo de alineación con el operador de mascaramiento de bits (off + w - 1) & ~(w - 1), el cual es una técnica avanzada que asume que el ancho w siempre es una potencia de 2.

> La solución (Auditoría): En C, la alineación de tipos básicos siempre es potencia de 2, así que el código de la IA funciona correctamente. Sin embargo, el caso borde crítico es que la IA no validó que la suma total <= 0 en offset_campo, entonces, si el usuario pasa un total de 0, el ciclo no se ejecuta y retorna -1 correctamente, pero no lanza advertencia de "estructura vacía". En el Artesano, usé la fórmula de módulo clásica, que es más legible y no depende de que los anchos sean potencias de 2, lo que la hace más robusta y fácil de auditar por un humano.

---
**Firma digital:** LeslieTamara14
