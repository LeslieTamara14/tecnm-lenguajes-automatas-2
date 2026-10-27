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
Estoy realizando una práctica de Lenguajes y Autómatas II sobre el manejo de punteros y memoria en C.
Debo implementar una función que invierta una cadena en su mismo espacio de memoria.

ROL (R):
Actúa como ingeniero de software especializado en programación de
sistemas, manejo de memoria, punteros y auditoría de código C.

ESPECIFICACIÓN (E):
Implementa void invertir(char *s) para invertir una cadena mutable
sobre el mismo buffer, sin reservar memoria adicional.
- Conserva exactamente la firma void invertir(char *s).
- Si s es NULL, regresa sin llamar a strlen ni provocar un fallo.
- Maneja correctamente una cadena vacía y una de un solo carácter.
- Usa dos índices y una variable char temporal para intercambiar.
- No uses malloc, arreglos auxiliares ni main().
- No alteres el terminador '\0' ni accedas fuera de los límites de
  una cadena válida, mutable y terminada en '\0'.
- La complejidad esperada es O(n) en tiempo y O(1) en memoria auxiliar.

OUTPUT (O):
Entrega la implementación modular en C y un análisis breve de sus
complejidades temporal y espacial.

```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion01.h"
#include <string.h>

void invertir(char *s) {
    size_t izquierda = 0;
    size_t derecha = strlen(s);

    if (derecha == 0) {
        return;
    }

    derecha--;

    while (izquierda < derecha) {
        char temporal = s[izquierda];
        s[izquierda] = s[derecha];
        s[derecha] = temporal;

        izquierda++;
        derecha--;
    }
}

Corrección aplicada para la prueba final:

Se añadió una comprobación inicial de NULL antes de calcular la longitud. También se retorna inmediatamente si la cadena tiene cero o un carácter.
#include "api_sesion01.h"
#include <stddef.h>
#include <string.h>

void invertir(char *s) {
    if (s == NULL) {
        return;
    }

    size_t izquierda = 0;
    size_t derecha = strlen(s);

    if (derecha < 2) {
        return;
    }

    derecha--;

    while (izquierda < derecha) {
        char temporal = s[izquierda];
        s[izquierda] = s[derecha];
        s[derecha] = temporal;

        izquierda++;
        derecha--;
    }

Complejidad: strlen recorre la cadena una vez y el ciclo de intercambio recorre aproximadamente la mitad de ella. El tiempo total es O(n). Solo se usan un número constante de variables, así que la memoria auxiliar es O(1). La función supone que cualquier puntero distinto de NULL apunta a un buffer válido, mutable y terminado en \0.

```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Valores NULL, buffers vacios y longitudes impares

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

Resultado de las pruebas

Se compiló cada implementación con gcc -std=c11 -Wall -Wextra -Werror y se ejecutó el harness de seis casos. 
Además, se repitió la ejecución con AddressSanitizer y UndefinedBehaviorSanitizer habilitados; no se reportaron errores para los casos del harness.

- [x] Los casos base y límite del harness terminaron sin crash.
- [x] En los seis casos comprobados no se detectaron accesos fuera de límites; la implementación no reserva memoria dinámica ni usa malloc.
- [x] La complejidad coincide con la implementación: O(n) tiempo y O(1) memoria auxiliar.
      
**Salida del harness:** 

=== AUDITORIA SESION 01 - Inversion de buffer in-place ===
Caso                                    esperado:              obtenido:              resultado
[ 1] invertir("HOLA")                   esperado: ALOH         obtenido: ALOH         OK
[ 2] invertir("abc")                    esperado: cba          obtenido: cba          OK
[ 3] invertir("")                       esperado:              obtenido:              OK
[ 4] invertir("AB")                     esperado: BA           obtenido: BA           OK
[ 5] invertir("a")                      esperado: a            obtenido: a            OK
[ 6] invertir(NULL)                     esperado: sin crash    obtenido: sin crash    OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. 

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> El caso omitido en la propuesta inicial fue el puntero NULL. La función llamaba a strlen(s) sin verificar primero que s fuera válido, lo cual podía provocar un fallo de ejecución en el sexto caso. Se corrigió agregando if (s == NULL) { return; } al principio de la función, antes de calcular la longitud. Después de la corrección, las seis pruebas del harness pasaron. La prueba cubre los casos especificados por el docente; no demuestra que un puntero distinto de NULL sea válido, por lo que la función requiere una cadena mutable y terminada correctamente en \0.

---
**Firma digital:** LeslieTamara14
    
