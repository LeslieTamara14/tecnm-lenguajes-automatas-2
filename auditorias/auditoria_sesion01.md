# Bitácora de Requerimientos y Auditoría IA — Sesión 01
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Encuadre, Diagnóstico de Punteros y Metodología IA  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Canul Llanes Leslie Tamara 
**Matrícula:** 24531370 
**Track:** [ C | Rust ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Inversión de buffer in-place (char*)  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Al realizar la función manualmente, identifiqué que debía recorrer la cadena desde el inicio y desde el final para intercambiar los caracteres. La dificultad fue controlar los índices para no salir de los límites del buffer. También debía tener en cuenta el carácter nulo \0, que indica el final de una cadena en C, y comprobar que la función no fallara cuando recibiera un puntero NULL o una cadena vacía. Para resolverlo, utilicé dos índices y una variable temporal, sin crear otra cadena ni reservar memoria adicional.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Punteros, O(n) tiempo, O(1) memoria |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
 Estoy realizando una práctica de Lenguajes y Autómatas II sobre el manejo de punteros y memoria en C. Debo implementar una función que invierta una cadena en su mismo espacio de memoria.

Rol: Actúa como un ingeniero de software especializado en programación de sistemas y compiladores.

Especificación: Implementa la función void invertir(char *s); respetando exactamente esa firma. La función debe invertir los caracteres de una cadena modificable sin crear una copia, sin utilizar malloc() y sin emplear arreglos auxiliares. Utiliza dos índices, uno al inicio y otro al final, y una variable temporal de tipo char para intercambiar los caracteres. Comprueba si el puntero es NULL y maneja correctamente la cadena vacía. No debes modificar el carácter nulo que marca el final de la cadena. El código debe compilar con gcc -std=c11 -Wall -Wextra -Werror y no debe incluir una función main(). La complejidad esperada es O(n) en tiempo y O(1) en memoria adicional.

Output: Devuelve la implementación completa de la función, lista para guardarse en ia_sesion01_c.c, e incluye una explicación breve de cómo funciona y de su complejidad.
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion01.h"
#include <stddef.h>
#include <string.h>

void invertir(char *s) {
    if (s == NULL) {
        return;
    }

    size_t inicio = 0;
    size_t fin = strlen(s);

    if (fin == 0) {
        return;
    }

    fin--;

    while (inicio < fin) {
        char temporal = s[inicio];
        s[inicio] = s[fin];
        s[fin] = temporal;

        inicio++;
        fin--;
    }
}
Explicación del código

Primero se comprueba si el puntero es NULL. Si lo es, la función termina sin intentar acceder a la memoria. Después se obtiene la longitud de la cadena con strlen(). Si la cadena está vacía, también termina sin realizar cambios.

Se utilizan dos índices: inicio, que comienza en la primera posición, y fin, que apunta al último carácter. Mientras el índice inicial sea menor que el final, se intercambian los caracteres utilizando una variable temporal. Los índices avanzan hacia el centro hasta terminar la inversión.

La función modifica el buffer original, no utiliza malloc() ni crea arreglos auxiliares. Su complejidad temporal es O(n) y su memoria adicional es O(1).

```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Valores NULL, buffers vacios y longitudes impares

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 

---
**Firma digital:** [Usuario de GitHub]
