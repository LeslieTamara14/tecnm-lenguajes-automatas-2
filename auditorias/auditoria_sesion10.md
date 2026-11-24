# Bitácora de Requerimientos y Auditoría IA — Sesión 10
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Hito 1: Mini-Evaluador de Expresiones  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Leslie Tamara Canul Llanes
**Matrícula:** 24531370
**Track:** [ C | Rust ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Integración AST y Tipado Estático  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Al codificar el evaluador a mano, el mayor reto fue estructurar las funciones para respetar la precedencia matemática (multiplicación antes que suma) sin complicar el código. Entendí que debía crear tres niveles: `primaria` para leer números y resolver paréntesis (recursión), `termino` para agrupar multiplicaciones/divisiones, y `expr` para sumas/restas.

> Además, tuve que tener cuidado con el manejo de punteros para avanzar en la cadena de texto (`*p->s`) y asegurarme de no quedar en bucles infinitos al consumir los operadores. También fue crucial implementar la validación de división por cero (`rhs == 0`) para evitar un *crash* en tiempo de ejecución (excepción de hardware).
---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Contrato formal de Frontend Semántico |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
CONTEXTO: Estoy completando un mini-evaluador de expresiones en C para la asignatura "Lenguajes y Autómatas II" (Sesión 10, Hito 1). El harness oficial compila mi archivo con -std=c11 -Wall -Wextra -Werror y llama a la función `evaluar_expresion` declarada en api_sesion10.h. Solo se admiten enteros no negativos literales y los operadores + - * / con paréntesis y espacios.

ROL: Actúa como ingeniero de compiladores experto en análisis sintáctico descendente recursivo en C11.

ESPECIFICACIÓN: 
- Firma EXACTA: `long evaluar_expresion(const char *fuente, int *error);`
- Respeta precedencia: * y / antes que + y -, y paréntesis.
- `*error = 1` si hay división entre cero.
- `*error = 2` si hay error de sintaxis (desbalance, operador sin operando, texto sobrante).
- Si hay error, devuelve 0 en el valor. Sin variables ni menos unario.
- Debe compilar sin warnings con -Wall -Wextra -Werror y sin `main()`.

OUTPUT: Devuelve únicamente el contenido completo del archivo `ia_sesion10_c.c` incluyendo los #include necesarios, las estructuras, `saltar`, `primaria`, `termino`, `expr` y `evaluar_expresion`.

```

---

## 3. Código Devuelto por el Modelo
```c
/* ia_sesion10_c.c - Fase 2: El Cientifico */
#include "api_sesion10.h"
#include <ctype.h>
#include <stdio.h> /* Corrección manual: agregado tras fallo de compilación */

typedef struct {
    const char *cursor;
    int error;
} Parser;

static void skip_ws(Parser *p) {
    while (*p->cursor == ' ' || *p->cursor == '\t') p->cursor++;
}

static long parse_expr(Parser *p);

static long parse_primary(Parser *p) {
    skip_ws(p);
    if (*p->cursor == '(') {
        p->cursor++;
        long v = parse_expr(p);
        skip_ws(p);
        if (*p->cursor != ')') { p->error = 2; return 0; }
        p->cursor++;
        return v;
    }
    if (!isdigit((unsigned char)*p->cursor)) {
        p->error = 2;
        return 0;
    }
    long v = 0;
    while (isdigit((unsigned char)*p->cursor)) {
        v = v * 10 + (*p->cursor - '0');
        p->cursor++;
    }
    return v;
}

static long parse_term(Parser *p) {
    long acc = parse_primary(p);
    for (;;) {
        skip_ws(p);
        char op = *p->cursor;
        if (op != '*' && op != '/') break;
        p->cursor++;
        long rhs = parse_primary(p);
        if (p->error) return 0;
        if (op == '*') {
            acc *= rhs;
        } else {
            if (rhs == 0) { p->error = 1; return 0; }
            acc /= rhs;
        }
    }
    return acc;
}

static long parse_expr(Parser *p) {
    long acc = parse_term(p);
    for (;;) {
        skip_ws(p);
        char op = *p->cursor;
        if (op != '+' && op != '-') break;
        p->cursor++;
        long rhs = parse_term(p);
        if (p->error) return 0;
        acc = (op == '+') ? acc + rhs : acc - rhs;
    }
    return acc;
}

long evaluar_expresion(const char *fuente, int *error) {
    static int dummy;
    if (error == NULL) error = &dummy;
    *error = 0;
    if (fuente == NULL) { *error = 2; return 0; }
    Parser p = { fuente, 0 };
    long v = parse_expr(&p);
    skip_ws(&p);
    if (p.error == 0 && *p.cursor != '\0') p.error = 2;
    *error = p.error;
    return p.error ? 0 : v;
}

```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Batería de 10 expresiones mixtas complejas

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

Pruebas ejecutadas: Batería de 10 expresiones mixtas complejas (incluyendo los 6 casos base del harness oficial y 4 casos extra de estrés).

☑ Caso base y límites superados sin Crash.
☑ No existen accesos fuera de límites ni Memory Leaks (no se usó memoria dinámica, todo fue en el stack).
☑ La complejidad teórica coincide con la implementada (O(n) en tiempo, O(1) en espacio).
1. Salida del Harness(Artesano)
   === AUDITORIA SESION 10 - Mini-evaluador de expresiones (Hito 1) ===
Caso                               esperado:              obtenido:              resultado
[ 1] "2+3*4"                            esperado: 14           obtenido: 14           OK
[ 2] "(2+3)*4"                          esperado: 20           obtenido: 20           OK
[ 3] "10/0" -> error 1                  esperado: 1            obtenido: 1            OK
[ 4] "2+" -> error 2                    esperado: 2            obtenido: 2            OK
[ 5] "  7 "                             esperado: 7            obtenido: 7            OK
[ 6] "5-10"                             esperado: -5           obtenido: -5           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion10.md
2. Salida del Harness(Código de Ia/Científico)
=== AUDITORIA SESION 10 - Mini-evaluador de expresiones (Hito 1) ===
Caso                               esperado:              obtenido:              resultado
[ 1] "2+3*4"                            esperado: 14           obtenido: 14           OK
[ 2] "(2+3)*4"                          esperado: 20           obtenido: 20           OK
[ 3] "10/0" -> error 1                  esperado: 1            obtenido: 1            OK
[ 4] "2+" -> error 2                    esperado: 2            obtenido: 2            OK
[ 5] "  7 "                             esperado: 7            obtenido: 7            OK
[ 6] "5-10"                             esperado: -5           obtenido: -5           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion10.md

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*

El problema: La IA omitió las librerías estándar necesarias (como <stdio.h>) para definir la macro NULL. Al compilar el código de la IA con las banderas estrictas -Werror, el proceso falló de inmediato arrojando el error: 'NULL' undeclared (first use in this function).

La solución: Lo detecté al ejecutar la terminal en Colab. Agregué el #include <stdio.h> manualmente. Esto comprueba que, aunque la IA tenga la lógica algorítmica perfecta, desconoce el entorno estricto de compilación y el programador debe actuar como auditor.

---
**Firma digital:** LeslieTamara14
