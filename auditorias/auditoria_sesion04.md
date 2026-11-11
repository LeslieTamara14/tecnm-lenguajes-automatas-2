# Bitácora de Requerimientos y Auditoría IA — Sesión 04
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Anatomía de un Árbol Sintáctico (AST)  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Canul Llanes Leslie Tamara
**Matrícula:** 24531370 
**Track:** [ C | Rust ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Estructura de nodo binario (ASTNode)  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Primero intenté hacer el código por mi cuenta. Cuando algo no me quedaba claro o no sabía cómo seguir, consultaba a la IA para resolver la duda y continuar. Con ese apoyo fui entendiendo cómo reservar memoria con malloc, guardar los datos de cada nodo y conectar sus hijos. Los números guardan su valor y las operaciones guardan el operador y las direcciones de sus dos hijos. Hay que respetar cuál es el izquierdo y cuál es el derecho, sobre todo en la resta. También hay que revisar si el nodo es NULL antes de usar sus datos.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Gramática E -> E + T | T con tipos de nodo |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Contexto: Estoy haciendo la sesión 04 de Lenguajes y Autómatas II
en Google Colab. El programa usa C y se prueba con GCC.

Rol: Ayúdame como tutor de programación. Usa instrucciones básicas
que pueda entender y explicar.

Especificación: Incluye api_sesion04.h y respeta estas firmas:
ASTNode *crear_numero(int v);
ASTNode *crear_binario(char op, ASTNode *izq, ASTNode *der);
int evaluar(ASTNode *n);

Reserva cada nodo con malloc. Usa NODO_NUM para números y NODO_BINOP
para operaciones. Guarda correctamente el hijo izquierdo y el derecho.
Evalúa ambos hijos antes de aplicar +, - o *.
evaluar(NULL) debe devolver -1.
Debe compilar con -std=c11 -Wall -Wextra -Werror.

Output: Entrega el código de ia_sesion04_c.c sin main(), usando
stdlib.h. Explica brevemente cuánto trabajo hace al evaluar el árbol.
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion04.h"
#include <stdlib.h>

ASTNode *crear_numero(int v) {
    ASTNode *nodo = (ASTNode *)malloc(sizeof(ASTNode));
    nodo->tipo = NODO_NUM;
    nodo->valor = v;
    nodo->op = 0;
    nodo->izq = NULL;
    nodo->der = NULL;
    return nodo;
}

ASTNode *crear_binario(char op, ASTNode *izq, ASTNode *der) {
    ASTNode *nodo = (ASTNode *)malloc(sizeof(ASTNode));
    nodo->tipo = NODO_BINOP;
    nodo->valor = 0;
    nodo->op = op;
    nodo->izq = izq;
    nodo->der = der;
    return nodo;
}

int evaluar(ASTNode *n) {
    if (n == NULL) return -1;
    if (n->tipo == NODO_NUM) return n->valor;

    int a = evaluar(n->izq);
    int b = evaluar(n->der);

    switch (n->op) {
        case '+': return a + b;
        case '-': return a - b;
        case '*': return a * b;
        default:  return -1;
    }
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Árbol vacío y evaluación de nodo raíz

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.


Pruebas ejecutadas: Evaluar 42, evaluar 3+5*2, revisar el hijo izquierdo, revisar el operador, evaluar NULL y evaluar 10-4.

Los resultados siguientes corresponden a las salidas de Colab compartidas. Ambas versiones obtuvieron 6/6 pruebas superadas.

Salida del harness — Artesano
=== AUDITORIA SESION 04 - Nodo binario de AST ===
Caso                               esperado:              obtenido:              resultado
[ 1] evaluar(nodo 42)                   esperado: 42           obtenido: 42           OK
[ 2] evaluar(3+5*2)                     esperado: 13           obtenido: 13           OK
[ 3] hijo izquierdo de (* 5 2)          esperado: 5            obtenido: 5            OK
[ 4] operador guardado                  esperado: *            obtenido: *            OK
[ 5] evaluar(NULL)                      esperado: -1           obtenido: -1           OK
[ 6] evaluar(10-4)                      esperado: 6            obtenido: 6            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. 

Salida del harness — Científico
=== AUDITORIA SESION 04 - Nodo binario de AST ===
Caso                               esperado:              obtenido:              resultado
[ 1] evaluar(nodo 42)                   esperado: 42           obtenido: 42           OK
[ 2] evaluar(3+5*2)                     esperado: 13           obtenido: 13           OK
[ 3] hijo izquierdo de (* 5 2)          esperado: 5            obtenido: 5            OK
[ 4] operador guardado                  esperado: *            obtenido: *            OK
[ 5] evaluar(NULL)                      esperado: -1           obtenido: -1           OK
[ 6] evaluar(10-4)                      esperado: 6            obtenido: 6            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE.

- [x] Los seis casos del harness pasaron sin que el programa se cerrara por un error.
- [ ] No existen accesos fuera de límites ni fugas de memoria: pendiente de comprobar con una herramienta de memoria. El harness no lo revisa.
- [x] La complejidad teórica coincide con el código: crear un nodo tarda O(1). Evaluar tarda O(n), porque visita cada nodo una vez. La recursión ocupa O(h), donde h es la altura del árbol.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> El código pasó las seis pruebas, pero al revisarlo encontré que no comprueba si malloc pudo reservar memoria. Si devuelve NULL, el programa intentaría guardar datos en una dirección que no es válida. Este problema también aparece en la versión Artesano y no lo detecta el harness, porque no provoca una falta de memoria.

La corrección propuesta es agregar esta revisión después de cada malloc, antes de usar nodo->:

```c
if (nodo == NULL) {
    return NULL;
}
```

> Esta corrección queda pendiente de aplicar y volver a ejecutar en Colab. Las salidas de arriba pertenecen al código mostrado, que todavía no incluye esa revisión.

> También observé que el harness no libera los nodos con free. Por eso, obtener 6/6 no demuestra que no haya fugas de memoria. Al terminar de usar un árbol, se deben liberar primero sus hijos y después el nodo. Esa comprobación queda pendiente.

---
**Firma digital:** LeslieTamara14
