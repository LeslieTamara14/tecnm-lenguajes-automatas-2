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
> Al codificar manualmente, fue fundamental recordar el uso de malloc para reservar memoria dinámica en el heap para cada nodo, asegurando que los punteros a los hijos izquierdo y derecho (izq, der) se inicializaran correctamente para evitar accesos a memoria indefinida o fallos de segmentación. Asimismo, se debió considerar el caso base de la función recursiva evaluar cuando el nodo recibido es NULL.

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
Desarrollo de un compilador en C99 bajo estándares estrictos (-Wall -Wextra -Werror).

Rol: Ingeniero experto en compiladores y estructuras de datos en C.
Especificación: Escribe las funciones crear_numero(int v), crear_binario(char op, ASTNode *izq, ASTNode *der), y evaluar(ASTNode *n) basadas en la estructura ASTNode provista en api_sesion04.h. La función evaluar debe manejar nodos numéricos y operaciones binarias (+, -, *), evaluando recursivamente ambos hijos y retornando -1 si el nodo de entrada es NULL sin provocar fallos de segmentación.

Output: Únicamente el código en C necesario para cumplir la interfaz, sin incluir main().
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion04.h"
#include <stdlib.h>

ASTNode *crear_numero(int v) {
    ASTNode *n = malloc(sizeof(ASTNode));
    if (!n) return NULL;
    n->tipo = NODO_NUM;
    n->valor = v;
    n->op = '\0';
    n->izq = NULL;
    n->der = NULL;
    return n;
}

ASTNode *crear_binario(char op, ASTNode *izq, ASTNode *der) {
    ASTNode *n = malloc(sizeof(ASTNode));
    if (!n) return NULL;
    n->tipo = NODO_BINOP;
    n->valor = 0;
    n->op = op;
    n->izq = izq;
    n->der = der;
    return n;
}

int evaluar(ASTNode *n) {
    if (!n) return -1;
    if (n->tipo == NODO_NUM) return n->valor;
    
    int izq = evaluar(n->izq);
    int der = evaluar(n->der);
    
    switch (n->op) {
        case '+': return izq + der;
        case '-': return izq - der;
        case '*': return izq * der;
        default: return -1;
    }
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Árbol vacío y evaluación de nodo raíz

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

Salida del Harness esperado
=== AUDITORIA SESION 04 - Nodo binario de AST ===
Caso                                   esperado:        obtenido:        resultado
[ 1] evaluar(nodo 42)                  esperado: 42     obtenido: 42     OK
[ 2] evaluar(3+5*2)                    esperado: 13     obtenido: 13     OK
[ 3] hijo izquierdo de (* 5 2)         esperado: 5      obtenido: 5      OK
[ 4] operador guardado                 esperado: *      obtenido: *      OK
[ 5] evaluar(NULL)                     esperado: -1     obtenido: -1     OK
[ 6] evaluar(10-4)                     esperado: 6      obtenido: 6      OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> La versión generada inicialmente por la IA omitió validar de forma explícita el manejo seguro ante posibles fallos en la asignación dinámica de memoria (malloc), lo cual podía causar desreferenciación de punteros nulos si el heap se agotaba. Se incorporaron verificaciones defensivas (if (!n) return NULL;) para garantizar la robustez del código ante limitaciones del sistema operativo.

---
**Firma digital:** LeslieTamara14
