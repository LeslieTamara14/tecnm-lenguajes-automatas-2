# Bitácora de Requerimientos y Auditoría IA — Sesión 03
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Google Colab, Mágicos y Flujo de Entrega  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Canul Lanes Leslie Tamara 
**Matrícula:** 24531370 
**Track:** [ C | Rust ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Manipulación de archivos y llamadas OS  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Al implementar las funciones identifiqué que es necesario abrir los archivos con fopen y comprobar que la apertura sea correcta antes de leer o escribir. También es importante cerrar cada archivo después de utilizarlo, incluso cuando ocurre un error.

> La dificultad principal consiste en contar correctamente las líneas. No basta con contar los caracteres \n, porque la última línea puede contener texto sin terminar con un salto de línea. Además, un archivo vacío debe devolver cero y una ruta inexistente debe devolver -1.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Salida estándar y códigos de retorno ABI |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Actúa como ingeniero de software especializado en C y compiladores. Implementa las funciones escribir_texto(const char *ruta, const char *texto) y contar_lineas(const char *ruta), respetando exactamente las firmas de api_sesion03.h.

La función escribir_texto debe abrir el archivo en modo escritura, guardar el texto, cerrar el archivo y devolver 0 si la operación termina correctamente o -1 si ocurre un error.

La función contar_lineas debe leer el archivo y devolver el número de líneas. Debe contar la última línea aunque no termine con \n, devolver 0 para un archivo vacío y -1 cuando la ruta no se pueda abrir o se detecte un error de lectura o cierre.

Comprueba los punteros nulos, verifica los resultados de las operaciones de archivo y garantiza que los archivos se cierren. No incluyas main() ni modifiques las firmas. Entrega código modular compatible con GCC y explica la complejidad temporal y espacial.

```

---

## 3. Código Devuelto por el Modelo
```c
Archivo: ia_sesion03_c.c

El código implementa escribir_texto mediante fopen, fputs y fclose. La función contar_lineas recorre el archivo con fgetc, cuenta los saltos de línea y contempla una última línea sin salto final.

La complejidad temporal de la escritura es O(m), donde m es la longitud del texto. El conteo de líneas tiene complejidad O(n), donde n es el número de caracteres del archivo. Ambas funciones utilizan O(1) de memoria auxiliar.

CÓDIGO UTILIZADO

#include "api_sesion03.h"
#include <stdio.h>
#include <limits.h>

int escribir_texto(const char *ruta, const char *texto) {
    FILE *archivo;
    int resultado_escritura;
    int resultado_cierre;

    if (ruta == NULL || texto == NULL) {
        return -1;
    }

    archivo = fopen(ruta, "w");
    if (archivo == NULL) {
        return -1;
    }

    resultado_escritura = fputs(texto, archivo);
    resultado_cierre = fclose(archivo);

    if (resultado_escritura == EOF || resultado_cierre == EOF) {
        return -1;
    }

    return 0;
}

int contar_lineas(const char *ruta) {
    FILE *archivo;
    int caracter;
    int ultimo = '\n';
    int hay_contenido = 0;
    int error = 0;
    long lineas = 0;

    if (ruta == NULL) {
        return -1;
    }

    archivo = fopen(ruta, "r");
    if (archivo == NULL) {
        return -1;
    }

    while ((caracter = fgetc(archivo)) != EOF) {
        hay_contenido = 1;
        ultimo = caracter;

        if (caracter == '\n') {
            lineas++;
        }

        if (lineas > INT_MAX) {
            error = 1;
            break;
        }
    }

    if (ferror(archivo)) {
        error = 1;
    }

    if (fclose(archivo) == EOF) {
        error = 1;
    }

    if (error) {
        return -1;
    }

    /* La última línea también cuenta si no termina en '\n'. */
    if (hay_contenido && ultimo != '\n') {
        lineas++;
    }

    if (lineas > INT_MAX) {
        return -1;
    }

    return (int)lineas;
}

```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Detección de errores de compilación GCC

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

Pruebas ejecutadas: seis casos del harness de la Sesión 03.

La escritura de texto devuelve 0.

El conteo de tres líneas devuelve 3.

El archivo sin salto de línea final devuelve 2.

El archivo vacío devuelve 0.

El archivo inexistente devuelve -1.

La ruta inválida devuelve -1.

=== AUDITORIA SESION 03 - Archivos: escribir y contar lineas ===
Caso                               esperado:              obtenido:              resultado
[ 1] escribir_texto(...)==0             esperado: 0            obtenido: 0            OK
[ 2] contar_lineas(3 lineas)            esperado: 3            obtenido: 3            OK
[ 3] contar_lineas(sin salto final)     esperado: 2            obtenido: 2            OK
[ 4] contar_lineas(archivo vacio)       esperado: 0            obtenido: 0            OK
[ 5] contar_lineas(inexistente)         esperado: -1           obtenido: -1           OK
[ 6] escribir_texto(ruta invalida)      esperado: -1           obtenido: -1           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> Un caso borde importante es el archivo cuyo último texto no termina con un salto de línea. Una implementación que solo cuente los caracteres \n lo contabilizaría incorrectamente. Para evitarlo, se comprueba si el archivo contiene datos y si el último carácter leído es distinto de \n; si ambas condiciones se cumplen, se suma una línea.

> También se comprueban los errores de apertura y las operaciones de lectura, escritura y cierre. El harness proporcionado no verifica todos los posibles errores de E/S, por lo que se necesitarían pruebas adicionales para validar esos escenarios.

---
**Firma digital:** LeslieTamara14
