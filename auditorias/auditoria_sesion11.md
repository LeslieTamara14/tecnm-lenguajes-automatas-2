# Bitácora de Requerimientos y Auditoría IA — Sesión 11
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Arquitectura de la Tabla de Símbolos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Canul Llanes Leslie Tamara 
**Matrícula:** 24531370
**Track:** [ C | Rust ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Tabla Hash con encadenamiento  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Al programar a mano, el reto principal fue manejar correctamente las colisiones. Entendí que si dos claves caen en el mismo índice (`hash % cubetas`), no debo sobrescribir el dato, sino crear una lista enlazada (encadenamiento) apuntando al nodo anterior. Además, tuve que ser cuidadoso con la validación de la cadena: el contrato pide que `nombre` sea de 16 caracteres, así que usé `strncpy` para evitar desbordamientos de memoria (buffer overflow). Finalmente, el manejo de punteros en `insertar_simbolo` al inicio de la lista requería actualizar el puntero de la tabla al nuevo nodo para no perder la referencia anterior.
---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Función de dispersión y colisiones O(1) |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
CONTEXTO: Estoy completando la Sesión 11 sobre Tablas Hash en C. El harness oficial compila con -std=c11 -Wall -Wextra -Werror. Solo se admiten enteros, operadores y el uso de las firmas exactas.
ROL: Actúa como ingeniero experto en estructuras de datos y compiladores.

ESPECIFICACIÓN: 
- Implementar unsigned int hash_djb2(const char *clave) (iniciando en 5381 y multiplicando por 33).
- Implementar int insertar_simbolo(Simbolo *tabla[], unsigned int cubetas, const char *nombre, int tipo, int offset).
- Devuelve -1 si tabla, nombre son NULL o cubetas es 0.
- Si el nombre ya existe (usar strcmp), devuelve 0.
- Si no existe, crear nodo, copiar nombre a salvo de desbordamientos, y encadenar al inicio. Devuelve 1.
- Debe compilar sin warnings con -Wall -Wextra -Werror.

OUTPUT: Devuelve únicamente el contenido completo de ia_sesion11_c.c.
```

---

## 3. Código Devuelto por el Modelo
```c
/* ia_sesion11_c.c - Fase 2: El Cientifico */
#include "api_sesion11.h"
#include <string.h>
#include <stdlib.h>
#include <stdio.h> /* Corrección manual: agregado tras fallo de compilación */

unsigned int hash_djb2(const char *clave) {
    unsigned int hash = 5381;
    int c;
    if (!clave) return hash;
    while ((c = *clave++)) {
        hash = ((hash << 5) + hash) + c; 
    }
    return hash;
}

int insertar_simbolo(Simbolo *tabla[], unsigned int cubetas, const char *nombre, int tipo, int offset) {
    if (!tabla || !nombre || cubetas == 0) return -1;

    unsigned int idx = hash_djb2(nombre) % cubetas;
    Simbolo *curr = tabla[idx];

    while (curr) {
        if (strcmp(curr->nombre, nombre) == 0) return 0;
        curr = curr->sig;
    }

    Simbolo *nuevo = (Simbolo *)malloc(sizeof(Simbolo));
    if (!nuevo) return -1;

    snprintf(nuevo->nombre, sizeof(nuevo->nombre), "%s", nombre);
    nuevo->tipo = tipo;
    nuevo->offset = offset;
    nuevo->sig = tabla[idx];
    tabla[idx] = nuevo;

    return 1;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Manejo de claves duplicadas y colisiones

* [✔] Caso base y límites superados sin *Crash*.
* [✔] No existen accesos fuera de límites ni *Memory Leaks*.
* [✔] La complejidad teórica coincide con la implementada.

=== AUDITORIA SESION 11 - Tabla hash con encadenamiento ===
Caso                               esperado:              obtenido:              resultado
[ 1] hash_djb2("")                      esperado: 5381         obtenido: 5381         OK
[ 2] hash_djb2("abc")                   esperado: 193485963    obtenido: 193485963    OK
[ 3] insertar x dos veces               esperado: 0            obtenido: 0            OK
[ 4] dos claves, una cubeta             esperado: 2            obtenido: 2            OK
[ 5] offset de beta en cadena           esperado: 4            obtenido: 4            OK
[ 6] insertar en tabla NULL             esperado: -1           obtenido: -1           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion11.md

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> El problema: La IA omitió incluir la librería <stdio.h>, lo que provocó un error de compilación al intentar usar snprintf para copiar el nombre del símbolo en el arreglo. El flag -Werror del harness detectó la declaración implícita de la función y detuvo el proceso arrojando implicit declaration of function 'snprintf'.

> La solución: Lo detecté al ejecutar la terminal de Colab con el código de la IA. Agregué el #include <stdio.h> manualmente y el código compiló.

> Esto comprueba que la IA tiende a olvidar las dependencias de las funciones que usa, y el programador debe auditar no solo la lógica del algoritmo (que en esta ocasión fue correcta), sino también los entornos de compilación y las librerías necesarias.

---
**Firma digital:** LeslieTamara14
