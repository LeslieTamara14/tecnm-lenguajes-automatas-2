# Bitácora de Requerimientos y Auditoría IA — Sesión 20
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Hito 2: Analizador Semántico y Scopes  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Canul Llanes Leslie Tamara 
**Matrícula:** 24531370  
**Track:** [ C | Rust ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Integración de Tabla de Símbolos y AST  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> El reto principal fue la función `contar_visibles`. El problema del *shadowing* (ocultamiento) es que un mismo nombre puede aparecer en el ámbito actual y en un ámbito padre, pero el compilador solo debe contarlo una vez. Mi solución manual fue crear un arreglo temporal (`vistos`) para guardar los nombres a medida que los encontraba y, antes de sumar al contador, revisaba con `strcmp` si ya lo había visto.

> En `profundidad_de` tuve que tener cuidado con el retorno -2; el contrato especifica que si falta algún argumento (como el nombre o el ámbito), se debe retornar -2 en lugar de -1, para diferenciarlo de "nombre no encontrado".

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Contrato de Validación Semántica Integral |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
CONTEXTO: Sesión 20 (Hito 2) de Lenguajes y Autómatas II. Entorno C11 (GCC) con -Wall -Wextra -Werror. Uso de api_sesion20.h. Sin main().

ROL: Ingeniero experto en compiladores y análisis semántico.

ESPECIFICACIÓN: 
- Implementar int profundidad_de(const Ambito *actual, const char *nombre): Si actual o nombre son NULL, retorna -2. Busca el nombre subiendo por los padres. Si lo encuentra, retorna la profundidad (0 para el actual, 1 para el padre, etc.). Si no existe en ningún padre, retorna -1.
- Implementar int contar_visibles(const Ambito *actual): Cuenta todos los nombres únicos visibles desde el ámbito actual hacia arriba. Si el mismo nombre está en varios ámbitos (shadowing), solo cuenta una vez.

OUTPUT: Código en C modular, sin main().

```

---

## 3. Código Devuelto por el Modelo
```c
/* ia_sesion20_c.c - Fase 2: El Cientifico */
#include "api_sesion20.h"
#include <string.h>

int profundidad_de(const Ambito *actual, const char *nombre) {
    if (!actual || !nombre) return -2;
    
    int p = 0;
    for (const Ambito *a = actual; a != NULL; a = a->padre) {
        for (int i = 0; i < a->total; i++) {
            if (strcmp(a->simbolos[i], nombre) == 0) return p;
        }
        p++;
    }
    return -1;
}

int contar_visibles(const Ambito *actual) {
    if (!actual) return 0;
    
    const char *vistos[256];
    int count = 0;
    
    for (const Ambito *a = actual; a != NULL; a = a->padre) {
        for (int i = 0; i < a->total; i++) {
            int dup = 0;
            for (int j = 0; j < count; j++) {
                if (strcmp(a->simbolos[i], vistos[j]) == 0) {
                    dup = 1;
                    break;
                }
            }
            if (!dup) {
                vistos[count++] = a->simbolos[i];
            }
        }
    }
    return count;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Casos complejos de shadowing y ámbito

* [✓] Caso base y límites superados sin *Crash*.
* [✓] No existen accesos fuera de límites ni *Memory Leaks*.
* [✓] La complejidad teórica coincide con la implementada.

=== AUDITORIA SESION 20 - Salida del Harness ===
Caso                               esperado:              obtenido:              resultado
[ 1] nombre en el ambito actual         esperado: 0            obtenido: 0            OK
[ 2] nombre dos niveles arriba          esperado: 2            obtenido: 2            OK
[ 3] shadowing gana el interno          esperado: 0            obtenido: 0            OK
[ 4] nombre no visible                  esperado: -1           obtenido: -1           OK
[ 5] visibles sin repetir               esperado: 3            obtenido: 3            OK
[ 6] ambito NULL                        esperado: -2           obtenido: -2           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion20.md

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> El problema: La IA usó un arreglo estático de 256 posiciones (const char *vistos[256];) para guardar los nombres ya contados. Si bien funciona para el harness y para programas pequeños, este es un caso borde crítico en memoria: si un programa tiene más de 256 variables visibles, la IA provocará un Buffer Overflow al desbordar el arreglo estático, corrompiendo la memoria.

> La solución (Auditoría): Aunque el Artesano también usó un límite (128), lo hice de forma consciente y documentada por las limitaciones del laboratorio. La IA simplemente puso un número sin justificarlo. En un compilador de producción, esta función requeriría usar memoria dinámica (malloc) o una estructura de datos como un Árbol Binario de Búsqueda (BST) para almacenar los nombres vistos sin límites arbitrarios. Lo detecté al revisar las variables declaradas en el código de la IA.

---
**Firma digital:** LeslieTamara14
