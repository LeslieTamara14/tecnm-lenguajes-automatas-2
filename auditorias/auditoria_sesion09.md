# Bitácora de Requerimientos y Auditoría IA — Sesión 09
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Reporte Formal de Errores Semánticos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Leslie Tamara Canul Llanes 
**Matrícula:** 24531370 
**Track:** [ C | Rust ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Diagnóstico con línea y columna  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> En mi primer intento la función no hacía nada y el buffer se quedaba vacío o con basura. Para corregirlo apliqué dos reglas de memoria básicas: primero revisar que salida no sea nulo y que cap sea mayor a cero, para no intentar escribir en un lugar inválido. Después usé snprintf en lugar de sprintf; si usara sprintf con un buffer de 10 (como en el caso 4) el programa explotaría con un error de memoria, porque el texto es más largo. snprintf me deja escribir hasta cap - 1 letras y siempre pone el '\0' al final, así que el caso 4 se trunca de forma segura. Además, armé el texto exactamente igual que GCC, con los dos puntos y la palabra error para que coincidiera exacto con lo que pide el harness.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Estructura estándar de errores GCC |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
[Actúa como programador de un formateador de diagnósticos en C. Usa
api_sesion09.h e implementa la función reportar:

  void reportar(const char *archivo, int linea, int columna,
                const char *mensaje, char *salida, int cap);

El formato de salida debe ser exacto, sin espacios extra:
"archivo:linea:columna: error: mensaje"

Debes respetar estrictamente el tamaño del buffer 'cap': nunca escribas
más de cap-1 caracteres y asegúrate de que el resultado siempre termine
en '\0'. Si el buffer es nulo o la capacidad es <= 0, no hagas nada.
No uses main(). Añade los includes que necesites (por ejemplo <stdio.h>
para snprintf). Entrega código compilable con gcc -std=c11 -Wall -Wextra
-Werror, e indica la complejidad en tiempo y espacio.]
```

---

## 3. Código Devuelto por el Modelo
```c
// Código devuelto por el LLM
#include "api_sesion09.h"
#include <stdio.h>

void reportar(const char *archivo, int linea, int columna, const char *mensaje, char *salida, int cap) {
    if (salida == NULL || cap <= 0) return;
    snprintf(salida, (size_t)cap, "%s:%d:%d: error: %s", archivo, linea, columna, mensaje);
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Múltiples errores sin abortar parser

* ☑ Caso base y límites superados sin *Crash*.
* ☑ No existen accesos fuera de límites ni *Memory Leaks*.
* ☑ La complejidad teórica coincide con la implementada.

```text
=== AUDITORIA SESION 09 - Diagnostico con linea y columna ===
Caso                               esperado:              obtenido:              resultado
[ 1] main.c:14:5
     esperado: main.c:14:5: error: tipo incompatible
     obtenido: main.c:14:5: error: tipo incompatible
     -> OK
[ 2] a.c:1:1
     esperado: a.c:1:1: error: x
     obtenido: a.c:1:1: error: x
     -> OK
[ 3] linea 0
     esperado: x.c:0:0: error: y
     obtenido: x.c:0:0: error: y
     -> OK
[ 4] cap=10 trunca                      esperado: main.c:14    obtenido: main.c:14    OK

RESULTADO: 4/4 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion09.md
```

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> La IA omitió el cast (size_t) en cap. Su código original era snprintf(salida, cap, ...) sin el cast. Aunque cap es un entero positivo, el compilador con -Wall -Wextra -Werror se queja de una posible comparación de tipos con signo y sin signo, porque la función espera un size_t. Lo detecté al compilar su propuesta: el compilador falló con advertencias convertidas en error. Lo solucioné agregando (size_t)cap en la llamada.

---
**Firma digital:** LeslieTamara14
