# Bitácora de Requerimientos y Auditoría IA — Sesión 06
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Sistema de Tipos Primitivos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Canul Llanes Leslie Tamara
**Matrícula:** 24531370  
**Track:** [ C | Rust ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Chequeo de compatibilidad de tipos  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> En mi primera ejecución, compatible siempre devolvía 0 y nombre_tipo siempre devolvía "desconocido". Pasaron solo los dos casos que esperaban rechazar una asignación. Para corregirlo con ayuda de IA, comprobé que ambos tipos fueran válidos, respeté el orden compatible(destino, origen) y devolví el nombre correspondiente a cada tipo.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Matriz de compatibilidad estática |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Actúa como programador de un verificador semántico en C. Usa api_sesion06.h
e implementa compatible(Tipo destino, Tipo origen) y nombre_tipo(Tipo t).
Acepta si ambos tipos son válidos y el destino tiene un rango igual o mayor
que el origen, según char < int < float. Rechaza TIPO_ERROR en ambos argumentos.
Devuelve los nombres char, int, float o error en minúsculas. Entrega código
sin main(), compilable con GCC usando -std=c11 -Wall -Wextra -Werror,
e indica la complejidad en tiempo y espacio.
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion06.h"

int compatible(Tipo destino, Tipo origen) {
    return origen >= TIPO_CHAR && destino <= TIPO_FLOAT && destino >= origen;
}

const char *nombre_tipo(Tipo t) {
    switch (t) {
        case TIPO_CHAR: return "char";
        case TIPO_INT: return "int";
        case TIPO_FLOAT: return "float";
        default: return "error";
    }
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Rechazo de asignaciones incompatibles

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

| Caso del harness | Esperado | Artesano inicial | Científico |
|---|---:|---|---|
| `compatible(INT, INT)` | `1` | `0` · Falla | `1` · OK |
| `compatible(FLOAT, INT)` | `1` | `0` · Falla | `1` · OK |
| `compatible(INT, FLOAT)` | `0` | `0` · OK | `0` · OK |
| `compatible(INT, ERROR)` | `0` | `0` · OK | `0` · OK |
| `nombre_tipo(INT)` | `int` | `desconocido` · Falla | `int` · OK |
| `nombre_tipo(ERROR)` | `error` | `desconocido` · Falla | `error` · OK |

Artesano inicial en Colab: 2/6 pruebas superadas.

Científico en Colab: 6/6 pruebas superadas.

Artesano con corrección asistida: 6/6 pruebas superadas en Colab con ./test_artesano. Los seis valores obtenidos coincidieron con los esperados en la tabla.

[x] Artesano con corrección asistida y Científico completaron los seis casos del harness sin interrupciones.

[] Verificación con herramientas de memoria pendiente. La implementación usa comparaciones y cadenas literales, sin reservar memoria dinámica.

[x] Ambas funciones tienen complejidad de tiempo y espacio auxiliar O(1).

Salida del harness después de implementar mi código en el artesano: 

| Caso | Esperado | Obtenido | Resultado |
|---|---:|---|---|
| `compatible(INT, INT)`   | `1` | `1` |  OK |
| `compatible(FLOAT, INT)` | `1` | `1` |  OK |
| `compatible(INT, FLOAT)` | `0` | `0` |  OK |
| `compatible(INT, ERROR)` | `0` | `0` |  OK |
| `nombre_tipo(INT)` | `int` | `int`   |  OK |
| `nombre_tipo(ERROR)` | `error` | `error`| OK |

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. 

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> El harness prueba TIPO_ERROR como origen. También revisé el caso con TIPO_ERROR como destino. El código de Científico lo rechaza mediante la comparación de rangos, y en la corrección asistida de Artesano dejé explícita la validación de ambos argumentos.

---
**Firma digital:** LeslieTamara14
