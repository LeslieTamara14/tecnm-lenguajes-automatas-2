# Bitácora de Requerimientos y Auditoría IA — Sesión 14
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Declaración vs Uso de Variables  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Canul LLanes Leslie Tamara  
**Matrícula:** 24531370  
**Track:** [ C | Rust ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Detección de variables no declaradas  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> El reto principal fue entender la diferencia entre el ámbito de "escritura" y el de "lectura". En la función `declarar`, es vital verificar que el nombre no exista previamente para no duplicar símbolos, pero además hay que validar que no se exceda la capacidad física del arreglo (`TABLA_MAX`). Por otro lado, en `usar`, el reto fue asegurarme de que la comparación sea exacta; un error común es usar funciones que busquen subcadenas, pero el contrato exige que "x" y "xy" se consideren variables totalmente distintas, por lo que la función `strcmp` es obligatoria aquí.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Invariante de definición previa |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
CONTEXTO: Sesión 14 de Lenguajes y Autómatas II. Entorno C11 con GCC y banderas estrictas (-Wall -Wextra -Werror). Uso de api_sesion14.h (Tabla con un arreglo de nombres y un contador total). Sin main().

ROL: Ingeniero experto en compiladores y tablas de símbolos.

ESPECIFICACIÓN: 
- `declarar(Tabla *tabla, const char *nombre)`: Devuelve -1 si tabla o nombre son NULL, o si el nombre es una cadena vacía. Devuelve 0 si el nombre ya existe en la tabla. Devuelve 1 si se inserta correctamente.
- `usar(const Tabla *tabla, const char *nombre)`: Devuelve 1 si el nombre ya fue declarado (existe en la tabla), de lo contrario devuelve 0.
- La comparación debe ser exacta usando strcmp (el prefijo "x" no es igual a "xy").

OUTPUT: Código en C modular, sin main().
```

---

## 3. Código Devuelto por el Modelo
```c
/* ia_sesion14_c.c - Fase 2: El Cientifico */
#include "api_sesion14.h"
#include <string.h>

int declarar(Tabla *tabla, const char *nombre) {
    if (!tabla || !nombre || nombre[0] == '\0') {
        return -1;
    }
    
    if (tabla->total >= TABLA_MAX) {
        return -1;
    }

    for (int i = 0; i < tabla->total; ++i) {
        if (strcmp(tabla->nombres[i], nombre) == 0) {
            return 0;
        }
    }

    tabla->nombres[tabla->total++] = nombre;
    return 1;
}

int usar(const Tabla *tabla, const char *nombre) {
    if (!tabla || !nombre) return 0;
    
    for (int i = 0; i < tabla->total; ++i) {
        if (strcmp(tabla->nombres[i], nombre) == 0) {
            return 1;
        }
    }
    return 0;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Uso antes de declarar y redeclaración

* [✔] Caso base y límites superados sin *Crash*.
* [✔] No existen accesos fuera de límites ni *Memory Leaks*.
* [✔] La complejidad teórica coincide con la implementada.

=== AUDITORIA SESION 14 - Salida del Harness ===
Caso                               esperado:              obtenido:              resultado
[ 1] usar sin declarar                  esperado: 0            obtenido: 0            OK
[ 2] usar despues de declarar           esperado: 1            obtenido: 1            OK
[ 3] redeclarar el mismo nombre         esperado: 0            obtenido: 0            OK
[ 4] total tras redeclarar              esperado: 1            obtenido: 1            OK
[ 5] prefijo no es el mismo nombre      esperado: 0            obtenido: 0            OK
[ 6] declarar nombre vacio              esperado: -1           obtenido: -1           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion14.md

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> El problema: En esta sesión, la IA resolvió el algoritmo base correctamente. Sin embargo, en el código de la IA (y del artesano) falta una validación silenciosa: no se comprueba si la variable que se intenta "usar" está vacía o es un puntero NULL antes de buscar en el ciclo.

> La solución (Auditoría): Aunque el harness pasa el caso usar(&t, "xy"), en un compilador real, si el usuario intenta usar una variable con un nombre mal formado (por ejemplo, un string vacío ""), el compilador no debería arrojar "variable no declarada", sino "error sintáctico". La IA omitió validar que el nombre en usar no sea vacío. En el Artesano tampoco lo puse porque el contrato no lo pedía explícitamente para usar, pero lo anoto como una mejora futura para evitar falsos positivos. La IA tampoco incluyó la librería <stdio.h>, pero en este caso no la necesitaba porque no imprimió nada.

---
**Firma digital:** LeslieTamara14
