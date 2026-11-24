# Bitácora de Requerimientos y Auditoría IA — Sesión 13
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Manejo de Ámbitos (Scope Global/Local)  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Canul Llanes Leslie Tamara
**Matrícula:** 24531370
**Track:** [ C | Rust ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Pila de tablas de símbolos (Stack Scopes)  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> El mayor reto a nivel lógico fue comprender la diferencia entre el **ámbito actual** y **la cadena de ámbitos**. En la función `declarar`, es crucial *no* buscar en los padres, porque si el padre ya tiene la variable `x`, el hijo aún puede declarar su propia `x` (esto es el *shadowing* o ocultamiento).
 
> Por otro lado, en `buscar`, la función debe devolver el **nombre del ámbito** donde se encontró la variable, no la variable en sí (el contrato pide `"global"`, `"bloque"`, etc.). Por último, tuve que asegurarme de no desbordar el arreglo `nombres[AMBITO_MAX]` validando que `ambito->total < AMBITO_MAX` antes de insertar.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Ocultamiento de variables (Shadowing) |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
CONTEXTO: Sesión 13 de Lenguajes y Autómatas II. Entorno C11 con GCC y banderas estrictas (-Wall -Wextra -Werror). Uso de api_sesion13.h (Ambito tiene un arreglo de nombres y un puntero al padre). Sin main().

ROL: Ingeniero experto en compiladores y tablas de símbolos.

ESPECIFICACIÓN: 
- `declarar`: Recibe un ámbito y un nombre. Devuelve -1 si el ámbito/nombre es nulo, o si el ámbito está lleno (total >= 8). Devuelve 0 si el nombre ya existe en ESE ámbito (no revisar padres). Devuelve 1 si se inserta correctamente en el ámbito actual.
- `buscar`: Recibe un ámbito y un nombre. Busca en el ámbito actual y luego en los padres (subiendo por el puntero `padre`). Devuelve el nombre del ámbito (`ambito->nombre`) donde se encontró. Si no existe, devuelve NULL.

OUTPUT: Código en C modular, sin main().
```

---

## 3. Código Devuelto por el Modelo
```c
/* ia_sesion13_c.c - Fase 2: El Cientifico */
#include "api_sesion13.h"
#include <string.h>

int declarar(Ambito *ambito, const char *nombre) {
    if (!ambito || !nombre || nombre[0] == '\0' || ambito->total >= AMBITO_MAX) {
        return -1;
    }
    
    for (int i = 0; i < ambito->total; i++) {
        if (strcmp(ambito->nombres[i], nombre) == 0) {
            return 0;
        }
    }
    
    ambito->nombres[ambito->total++] = nombre;
    return 1;
}

const char *buscar(const Ambito *ambito, const char *nombre) {
    if (!ambito || !nombre) return NULL;
    
    for (const Ambito *a = ambito; a != NULL; a = a->padre) {
        for (int i = 0; i < a->total; i++) {
            if (strcmp(a->nombres[i], nombre) == 0) {
                return a->nombre;
            }
        }
    }
    return NULL;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Búsqueda jerárquica de identificadores

* [✔] Caso base y límites superados sin *Crash*.
* [✔] No existen accesos fuera de límites ni *Memory Leaks*.
* [✔] La complejidad teórica coincide con la implementada.

=== AUDITORIA SESION 13 - Salida del Harness ===
Caso                               esperado:              obtenido:              resultado
[ 1] declarar(&g, "x")                  esperado: 1            obtenido: 1            OK
[ 2] buscar en el mismo ambito          esperado: global       obtenido: global       OK
[ 3] buscar subiendo al padre           esperado: global       obtenido: global       OK
[ 4] shadowing en el hijo               esperado: bloque       obtenido: bloque       OK
[ 5] tres niveles hasta la raiz         esperado: raiz         obtenido: raiz         OK
[ 6] nombre no declarado                esperado: (nulo)       obtenido: (nulo)       OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion13.md

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> El problema: La IA resolvió los casos del harness a la perfección. Sin embargo, en su función declarar, omitió validar que el nombre no sea una cadena vacía (nombre[0] == '\0').

> La solución (Auditoría): Aunque el harness no introduce cadenas vacías, en un compilador real, intentar declarar una variable con un identificador vacío es un error sintáctico que debe rebotar. En mi versión del Artesano agregué la validación nombre[0] == '\0' junto con los nulos para devolver -1 y evitar meter basura a la tabla de símbolos. En la versión de la IA esto fue un " caso borde" omitido, aunque no afectara el resultado de la prueba.

---
**Firma digital:** LeslieTamara14
