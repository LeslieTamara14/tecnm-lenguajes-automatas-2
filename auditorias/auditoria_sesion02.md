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
Estoy trabajando en un laboratorio de Lenguajes y Autómatas II sobre listas enlazadas en C.
Existe un archivo llamado api_sesion02.h que define una estructura NodoToken y las funciones agregar_token, contar_tokens y liberar_lista. El código será evaluado mediante un harness automático compilado con gcc usando -Wall -Wextra -Werror.

Rol:
Actúa como un ingeniero de software especializado en programación de sistemas, compiladores y gestión de memoria en C.

Especificación:
Implementa exactamente las siguientes funciones:

NodoToken *agregar_token(NodoToken *cabeza, const char *lexema);
int contar_tokens(NodoToken *cabeza);
void liberar_lista(NodoToken *cabeza);

Requisitos:
- Reservar cada nodo usando malloc.
- Copiar el lexema respetando el límite de 16 caracteres del arreglo.
- El último nodo debe tener siguiente == NULL.
- Si la lista está vacía, agregar_token debe crear el primer nodo y devolver la nueva cabeza.
- Mantener el orden de inserción.
- contar_tokens(NULL) debe devolver 0.
- liberar_lista(NULL) no debe provocar errores.
- Utilizar free para liberar toda la memoria reservada.
- No agregar función main().
- No modificar ninguna firma declarada en api_sesion02.h.

Output:
Devuelve únicamente código C compilable, modular y compatible con el contrato definido en api_sesion02.h.

```

---

## 3. Código Devuelto por el Modelo
```c

#include "api_sesion02.h"
#include <stdlib.h>
#include <string.h>

NodoToken *agregar_token(NodoToken *cabeza, const char *lexema)
{
    NodoToken *nuevo = malloc(sizeof(NodoToken));

    if (nuevo == NULL) {
        return cabeza;
    }

    strncpy(nuevo->lexema, lexema, sizeof(nuevo->lexema) - 1);
    nuevo->lexema[sizeof(nuevo->lexema) - 1] = '\0';
    nuevo->siguiente = NULL;

    if (cabeza == NULL) {
        return nuevo;
    }

    NodoToken *actual = cabeza;

    while (actual->siguiente != NULL) {
        actual = actual->siguiente;
    }

    actual->siguiente = nuevo;

    return cabeza;
}

int contar_tokens(NodoToken *cabeza)
{
    int contador = 0;

    while (cabeza != NULL) {
        contador++;
        cabeza = cabeza->siguiente;
    }

    return contador;
}

void liberar_lista(NodoToken *cabeza)
{
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

 Caso base y límites superados sin Crash.
 No existen accesos fuera de límites en los casos evaluados.
 La complejidad teórica coincide con la implementada.

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
> El harness verifica seis casos funcionales, pero no comprueba explícitamente escenarios como un lexema más largo que el tamaño del arreglo, un puntero lexema con valor NULL o una falla de malloc.

> Durante la auditoría se detectó que la primera versión generada por la IA no respetó el contrato definido en api_sesion02.h. La función agregar_token fue implementada con la firma:
> 
> void agregar_token(NodoToken **cabeza, const char *lexema);
> cuando el contrato exigía:
> NodoToken *agregar_token(NodoToken *cabeza, const char *lexema);

> El problema se identificó al compilar con GCC, ya que el compilador emitió el error:

>conflicting types for 'agregar_token'

> La solución consistió en corregir la función para que devolviera la cabeza de la lista y respetara exactamente la firma especificada en api_sesion02.h. Una vez realizada la corrección, la implementación superó las seis pruebas del harness con resultado 6/6.

---
**Firma digital:** LeslieTamara14
