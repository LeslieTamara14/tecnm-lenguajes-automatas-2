# Bitácora de Requerimientos y Auditoría IA — Sesión 08
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Inferencia y Promoción de Tipos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Leslie Tamara Canul Llanes 
**Matrícula:** 24531370
**Track:** [ C | Rust ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Reglas de casteo implícito  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> En mi primer intento promover devolvía siempre TIPO_ERROR y nombre_tipo devolvía "desconocido", porque solo tenía el esqueleto con (void)a; (void)b; return TIPO_ERROR;. Para corregirlo con ayuda de IA comprobé primero que ninguno de los dos operandos fuera TIPO_ERROR (si alguno lo es, se devuelve el centinela), y después apliqué la jerarquía CHAR < INT < FLOAT devolviendo el de mayor rango. En hay_perdida descarté los casos con ERROR y comparé con destino < origen, porque el enum ya está ordenado numéricamente (CHAR=1, INT=2, FLOAT=3).

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Promoción entera y ensanchamiento |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Actúa como programador de un verificador de tipos en C. Usa api_sesion08.h
e implementa:

  Tipo promover(Tipo a, Tipo b);
  int hay_perdida(Tipo destino, Tipo origen);
  const char *nombre_tipo(Tipo t);

Jerarquía: CHAR < INT < FLOAT. El enum ya está ordenado como
TIPO_ERROR=0, TIPO_CHAR=1, TIPO_INT=2, TIPO_FLOAT=3.

promover: si a o b es TIPO_ERROR, devolver TIPO_ERROR; si no, el de
mayor rango. hay_perdida: 1 si el destino es más angosto que el origen,
0 en caso contrario; 0 si alguno es TIPO_ERROR. nombre_tipo: "char",
"int", "float" o "error". El esqueleto solo incluye "api_sesion08.h";
no añadas otros includes ni uses NULL. Entrega código sin main(),
compilable con gcc -std=c11 -Wall -Wextra -Werror, e indica la
complejidad en tiempo y espacio.
```

---

## 3. Código Devuelto por el Modelo
```c
// Código devuelto por el LLM
#include "api_sesion08.h"

Tipo promover(Tipo a, Tipo b) {
    if (a == TIPO_ERROR || b == TIPO_ERROR) {
        return TIPO_ERROR;
    }
    return (a > b) ? a : b;
}

int hay_perdida(Tipo destino, Tipo origen) {
    if (destino == TIPO_ERROR || origen == TIPO_ERROR) {
        return 0;
    }
    return destino < origen;
}

const char *nombre_tipo(Tipo t) {
    switch (t) {
        case TIPO_CHAR:  return "char";
        case TIPO_INT:   return "int";
        case TIPO_FLOAT: return "float";
        default:         return "error";
    }
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Mezcla de tipos enteros y flotantes

* ☑ Caso base y límites superados sin *Crash*.
* ☑ No existen accesos fuera de límites ni *Memory Leaks*.
* ☑ La complejidad teórica coincide con la implementada.

=== AUDITORIA SESION 08 - Reglas de promocion y perdida ===
Caso                               esperado:              obtenido:              resultado
[ 1] promover(CHAR,INT)                 esperado: int          obtenido: int          OK
[ 2] promover(INT,FLOAT)                esperado: float        obtenido: float        OK
[ 3] promover(FLOAT,FLOAT)              esperado: float        obtenido: float        OK
[ 4] promover(ERROR,INT)                esperado: error        obtenido: error        OK
[ 5] hay_perdida(CHAR,INT)              esperado: 1            obtenido: 1            OK
[ 6] hay_perdida(INT,CHAR)              esperado: 0            obtenido: 0            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion08.md
### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> La IA omitió documentar el caso promover(ERROR, ERROR) y, en general, no dejó explícito que el centinela se descarta en ambos argumentos. Su código sí lo resuelve porque el primer if atrapa TIPO_ERROR en cualquiera de los dos operandos, pero no lo menciona. Lo detecté revisando la tabla completa 4×4 de pares de tipos y comprobando que el resultado coincide con la jerarquía. En el Artesano lo dejé con dos if separados para que el centinela quedara visible; en el Científico lo compacté con || y lo comenté.

---
**Firma digital:** [Usuario de GitHub]
