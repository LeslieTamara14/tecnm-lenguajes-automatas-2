# Bitácora de Requerimientos y Auditoría IA — Sesión 07
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Evaluación de Expresiones Aritméticas  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Leslie Tamara Canul Llanes
**Matrícula:** 24531370
**Track:** [ C | Rust ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Evaluador recursivo de operaciones  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Desreferencia de puntero nulo (memoria). El primer borrador leía n->tipo antes de comprobar n == NULL. En el caso 4 del harness (evaluar(NULL, &e)) eso habría provocado SIGSEGV. Lo corregí moviendo if (n == NULL) al inicio absoluto de la función, antes de cualquier acceso a n->tipo o n->valor.

> División/módulo entre cero (sintaxis + UB). En C, izq / der con der == 0 no devuelve un valor "infinito": aborta con SIGFPE, y para % el estándar C11 declara comportamiento indefinido. Por eso la verificación if (der == 0) debe ejecutarse antes de operar, no después. Ese es el corazón de la sesión: detectar el error estáticamente antes de que el proceso se caiga.

> Propagación del error sin recalcular el padre. Si un hijo marca *error = 1 o *error = 2, el padre no debe operar con el 0 devuelto por el hijo. La solución fue if (*error != 0) return 0; inmediatamente después de cada llamada recursiva, antes del segundo hijo y antes del operador.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Manejo estricto de división entre cero |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
[Pega aquí el prompt formal que redactaste para la IA]
```

---

## 3. Código Devuelto por el Modelo
```c
// Código devuelto por el LLM
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Expresiones complejas y división por 0

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 

---
**Firma digital:** [Usuario de GitHub]
