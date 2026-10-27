# Bitácora de Requerimientos y Auditoría IA — Sesión 02
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Gestión en GitHub Web y Nodos Dinámicos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Canul Llanes Leslie Tamara
**Matrícula:** 24531370
**Track:** [ C | Rust ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Lista enlazada de tokens en Heap  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Al implementar la lista enlazada identifiqué que cada nodo debía reservarse dinámicamente con malloc y enlazarse mediante el puntero siguiente. También fue necesario evitar perder la referencia al siguiente nodo al liberar memoria.

> Otro aspecto importante fue respetar el tamaño fijo de lexema, que es de 16 caracteres incluyendo el terminador nulo. Para evitar accesos fuera de límites, la copia del texto debe limitarse al espacio disponible.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Gestión de memoria dinámica C99/Rust |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Actúa como ingeniero de software especializado en C y compiladores. Implementa las funciones crear_nodo, agregar_token, contar_tokens y liberar_lista, respetando exactamente las declaraciones de api_sesion02.h.

Utiliza memoria dinámica mediante malloc y libera cada nodo con free. Comprueba los punteros antes de utilizarlos, conserva el orden de inserción, garantiza que el último nodo tenga siguiente == NULL y permite contar y liberar una lista vacía sin provocar errores.

El campo lexema tiene capacidad para 16 caracteres, incluido el terminador nulo. Evita desbordamientos al copiar cadenas y contempla la falla de asignación de memoria. No incluyas main() ni cambies las firmas de las funciones. Entrega únicamente el código modular y un análisis de complejidad temporal y espacial.
```

---

## 3. Código Devuelto por el Modelo
```c
El código propuesto implementa las cuatro funciones solicitadas. La creación de nodos utiliza malloc, la copia del lexema limita los caracteres al espacio disponible, la inserción conserva el orden y la liberación recorre la lista antes de terminar.

Archivo: ia_sesion02_c.c

#include "api_sesion02.h"
#include <stdlib.h>
#include <string.h>

NodoToken *crear_nodo(const char *lexema) {
    NodoToken *nuevo = malloc(sizeof(NodoToken));

    if (nuevo == NULL) {
        return NULL;
    }

    if (lexema == NULL) {
        nuevo->lexema[0] = '\0';
    } else {
        strncpy(nuevo->lexema, lexema, sizeof(nuevo->lexema) - 1);
        nuevo->lexema[sizeof(nuevo->lexema) - 1] = '\0';
    }

    nuevo->siguiente = NULL;
    return nuevo;
}

void agregar_token(NodoToken **cabeza, const char *lexema) {
    if (cabeza == NULL) {
        return;
    }

    NodoToken *nuevo = crear_nodo(lexema);
    if (nuevo == NULL) {
        return;
    }

    if (*cabeza == NULL) {
        *cabeza = nuevo;
        return;
    }

    NodoToken *actual = *cabeza;

    while (actual->siguiente != NULL) {
        actual = actual->siguiente;
    }

    actual->siguiente = nuevo;
}

int contar_tokens(NodoToken *cabeza) {
    int contador = 0;

    while (cabeza != NULL) {
        contador++;
        cabeza = cabeza->siguiente;
    }

    return contador;
}

void liberar_lista(NodoToken *cabeza) {
    while (cabeza != NULL) {
        NodoToken *siguiente = cabeza->siguiente;
        free(cabeza);
        cabeza = siguiente;
    }
}

```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Prevención de Use-After-Free y cadenas nulas

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

Pruebas ejecutadas: creación de nodos, conteo, orden de inserción, terminación de la lista, lista vacía y liberación sin errores.

=== AUDITORIA SESION 02 - Lista enlazada de tokens en heap ===
Caso                               esperado:              obtenido:              resultado
[ 1] crear_nodo("if")->lexema           esperado: if           obtenido: if           OK
[ 2] contar_tokens(3 tokens)            esperado: 3            obtenido: 3            OK
[ 3] orden: segundo token               esperado: dos          obtenido: dos          OK
[ 4] ultimo->siguiente == NULL          esperado: 1            obtenido: 1            OK
[ 5] contar_tokens(NULL)                esperado: 0            obtenido: 0            OK
[ 6] liberar_lista(NULL)                esperado: sin crash    obtenido: sin crash    OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. 

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> El harness verifica seis casos, pero no comprueba explícitamente todos los escenarios posibles. Por ejemplo, no prueba directamente la recepción de un lexema nulo, un lexema que exceda la capacidad del arreglo ni una falla de malloc.

> La implementación contempla el lexema nulo y limita la copia a 15 caracteres más el terminador nulo. También comprueba si malloc devuelve NULL. Estas medidas mejoran la seguridad, aunque para confirmar su comportamiento se requieren pruebas adicionales específicas.

> No se afirma que no existan fugas de memoria en todos los escenarios únicamente por superar el harness; esa conclusión requiere pruebas adicionales de memoria (La AI no omitió ningún error).

---
**Firma digital:** LeslieTamara14
