# Bitácora de Requerimientos y Auditoría IA — Sesión 22
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Generación de TAC para Expresiones  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Canul Llanes Leslie Tamara
**Matrícula:** 24531370
**Track:** [ C | Rust ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Traductor de nodos AST a TAC  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> El mayor desafío conceptual fue entender cómo una función recursiva puede comunicar sus resultados hacia arriba. En un inicio intenté usar variables globales (lo cual está prohibido por el contrato) o imprimir directamente. La solución correcta fue hacer que la función recursiva devuelva un `const char *` con el nombre del temporal que acaba de crear (o el valor de la hoja).
 
> Al principio cometí el error de no devolver este string y el código fallaba porque el nodo padre no sabía qué temporales usar para sus hijos. Además, tuve que asegurarme de que el contador de temporales fuera el mismo durante toda la recursión, pasándolo por referencia (con punteros), para que los nombres generados fueran únicos (`_t0`, `_t1`, `_t2`...).

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Conservación de precedencia de operadores |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
CONTEXTO: Sesión 22 de Lenguajes y Autómatas II. Entorno C11 (GCC) con -Wall -Wextra -Werror. Uso de api_sesion22.h. Sin main().

ROL: Ingeniero experto en compiladores y generación de código.

ESPECIFICACIÓN: 
- Implementar int generar_tac(const Nodo *raiz, char lineas[][64], int max).
- Recorrer el árbol en postorden.
- Para cada nodo de tipo 'B' (operador binario), generar una línea TAC con el formato "_tX = izq op der".
- Usar nuevo_temporal para obtener el nombre de _tX.
- Respetar el límite max de líneas.
- Devolver el número total de líneas generadas.

OUTPUT: Código en C modular, sin main().
```

---

## 3. Código Devuelto por el Modelo
```c
/* ia_sesion22_c.c - Fase 2: El Cientifico */
#include "api_sesion22.h"
#include <stdio.h>

typedef struct {
    char nombre[64][16];
    int usados;
} Pool;

const char *nuevo_temporal(Pool *p, int *contador) {
    if (p->usados >= 64) return "_t?";
    snprintf(p->nombre[p->usados], sizeof p->nombre[0], "_t%d", (*contador)++);
    return p->nombre[p->usados++];
}

static const char *tac_rec(const Nodo *n, char l[][64], int max, int *cnt, Pool *p, int *c) {
    if (!n) return "";
    if (n->tipo == 'H') return n->valor;

    const char *izq = tac_rec(n->izq, l, max, cnt, p, c);
    const char *der = tac_rec(n->der, l, max, cnt, p, c);

    const char *t = nuevo_temporal(p, c);
    if (*cnt < max) {
        snprintf(l[*cnt], 64, "%s = %s %c %s", t, izq, n->op, der);
        (*cnt)++;
    }
    return t;
}

int generar_tac(const Nodo *raiz, char lineas[][64], int max) {
    if (!raiz || raiz->tipo == 'H') return 0;
    Pool p = {0};
    int cont = 0, n = 0;
    tac_rec(raiz, lineas, max, &n, &p, &cont);
    return n;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Expresiones anidadas con paréntesis

* [✓] Caso base y límites superados sin *Crash*.
* [✓] No existen accesos fuera de límites ni *Memory Leaks*.
* [✓] La complejidad teórica coincide con la implementada.

=== AUDITORIA SESION 22 - Salida del Harness ===
Caso                               esperado:              obtenido:              resultado
[ 1] a + b * c                          esperado: _t0 = b * c  obtenido: _t0 = b * c  OK
[ 2] a + b * c (2a linea)
     esperado: _t1 = a + _t0
     obtenido: _t1 = a + _t0
     -> OK
[ 3] (a + b) * c
     esperado: _t1 = _t0 * c
     obtenido: _t1 = _t0 * c
     -> OK
[ 4] x * (y - z) + w
     esperado: _t2 = _t1 + w
     obtenido: _t2 = _t1 + w
     -> OK
[ 5] hoja sin operaciones               esperado: 0            obtenido: 0            OK
[ 6] (a + b) * (c - d)
     esperado: _t2 = _t0 * _t1
     obtenido: _t2 = _t0 * _t1
     -> OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion22.md

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> El problema: En la primera iteración, mi código del Artesano falló porque intenté recuperar los nombres de los temporales de forma estática (usando _t?), lo que provocaba que los casos 2, 3, 4 y 6 fallaran. Esto me demostró que la IA tenía razón al plantear la función recursiva devolviendo un string.

> Al auditar el código de la IA, noté que omitió una validación defensiva: si el árbol tiene un nodo mal formado (por ejemplo, un operador con un solo hijo), la IA intentará desreferenciar un puntero nulo. Aunque el harness no prueba esto, agregué una validación temprana en mi Artesano (if (nodo == NULL) return "";).

> La solución (Auditoría): Aprendí que la recursión en compiladores requiere devolver el resultado parcial. Al final, ambas implementaciones (la mía corregida y la de la IA) pasaron el 6/6, demostrando que la mejor manera de generar TAC es mediante una función que retorne el nombre del temporal para que el nodo superior lo use en su propia operación.

---
**Firma digital:** LeslieTamara14
