# Diseño de la API
# Maria Clara Medina y Franchesca Garcia Tabares

## 1. Propósito del proyecto

Este documento presenta el diseño de la API para la primera entrega del proyecto.

En esta entrega no se implementa código. El objetivo es definir los servicios del sistema, sus operaciones, los argumentos que reciben y las respuestas que deben retornar en formato JSON.

El sistema permite registrar programas y ficheros para luego ejecutar procesos de lote. Un proceso de lote usa un fichero de entrada, ejecuta uno o varios programas en orden y guarda el resultado en un fichero de salida.

La comunicación entre procesos se realizará mediante tuberías nombradas. Como nuestro trabajo es en parejas, el diseño contempla una futura implementación en Linux y Windows 11.

---

## 2. Componentes propuestos

El sistema se divide en los siguientes componentes:

| Componente | Responsabilidad |
|---|---|
| `cliente` | Envía solicitudes al sistema. No se implementa en esta entrega. |
| `ctrllt` | Recibe peticiones y las redirige al servicio correcto. |
| `gesprog` | Administra programas registrados. |
| `gesfich` | Administra ficheros registrados. |
| `ejecutor` | Ejecuta y controla procesos de lote. |
| `aralmac` | Área donde se almacenan programas, ficheros e información del sistema. |

Flujo general:

```text
cliente -> ctrllt -> gesprog
                 -> gesfich
                 -> ejecutor
```

`ctrllt` actúa como pasarela. Recibe una petición JSON, revisa el servicio solicitado y la envía al componente correspondiente.

---

## 3. Comunicación y sistemas operativos

Los procesos se comunicarán mediante tuberías nombradas. Cada mensaje enviado por una tubería será un JSON.

El diseño contempla dos casos:

- `full-duplex`: una tubería permite enviar y recibir.
- `half-duplex`: se usan dos tuberías, una para petición y otra para respuesta.

Ejemplo:

```text
cliente ---- petición ----> ctrllt
cliente <--- respuesta ---- ctrllt
```

La API será la misma en Linux y Windows 11. Lo que cambia es la implementación de las tuberías.

En Linux se podrían usar nombres como:

```text
/tmp/lotes_ctrllt
/tmp/lotes_gesprog
/tmp/lotes_gesfich
/tmp/lotes_ejecutor
```

En Windows 11 se podrían usar nombres como:

```text
\\.\pipe\lotes_ctrllt
\\.\pipe\lotes_gesprog
\\.\pipe\lotes_gesfich
\\.\pipe\lotes_ejecutor
```

---

## 4. Estructura base de los mensajes

Todas las peticiones tendrán esta estructura:

```json
{
    "id_peticion": "req-0001",
    "servicio": "nombre_servicio",
    "operacion": "nombre_operacion",
    "datos": {}
}
```

Respuesta exitosa:

```json
{
    "id_peticion": "req-0001",
    "estado": "ok",
    "mensaje": "Operación realizada correctamente",
    "datos": {}
}
```

Respuesta con error:

```json
{
    "id_peticion": "req-0001",
    "estado": "error",
    "mensaje": "Descripción del error",
    "codigo": "CODIGO_ERROR"
}
```

---

## 5. Servicios de la API

En esta sección se definen los servicios principales y las operaciones que acepta cada uno. Para no repetir demasiado, primero se muestran las operaciones en tablas y luego algunos ejemplos JSON de las peticiones más importantes.

---

### 5.1 gesprog

`gesprog` administra los programas registrados en `aralmac`. Cada programa se identifica con el formato:

```text
p-XXXX
```

Ejemplo:

```text
p-0001
```

Un programa almacena ejecutable, argumentos, variables de ambiente, descripción y estado.

| Operación | `datos` esperados | Respuesta principal |
|---|---|---|
| `registrar_programa` | `{ "ejecutable": "/usr/bin/wc", "argumentos": ["-l"], "ambiente": { "LANG": "es_CO.UTF-8" }, "descripcion": "Cuenta líneas" }` | Retorna `id_programa`. |
| `leer_programa` | `{ "id_programa": "p-0001" }` | Retorna la información del programa. |
| `listar_programas` | `{}` | Lista los programas registrados. |
| `actualizar_programa` | `{ "id_programa": "p-0001", "ejecutable": "/usr/bin/wc", "argumentos": ["-w"], "ambiente": {}, "descripcion": "Cuenta palabras" }` | Actualiza el programa. |
| `borrar_programa` | `{ "id_programa": "p-0001" }` | Borra el programa. |
| `suspender_servicio` | `{}` | Cambia el servicio a `suspendido`. |
| `reasumir_servicio` | `{}` | Cambia el servicio a `corriendo`. |
| `terminar_servicio` | `{}` | Cambia el servicio a `terminado`. |

Ejemplo para registrar programa:

```json
{
    "id_peticion": "req-0001",
    "servicio": "gesprog",
    "operacion": "registrar_programa",
    "datos": {
        "ejecutable": "/usr/bin/wc",
        "argumentos": ["-l"],
        "ambiente": {
            "LANG": "es_CO.UTF-8"
        },
        "descripcion": "Cuenta la cantidad de líneas del fichero de entrada"
    }
}
```

Respuesta:

```json
{
    "id_peticion": "req-0001",
    "estado": "ok",
    "mensaje": "Programa registrado correctamente",
    "datos": {
        "id_programa": "p-0001"
    }
}
```

Ejemplo para leer programa:

```json
{
    "id_peticion": "req-0002",
    "servicio": "gesprog",
    "operacion": "leer_programa",
    "datos": {
        "id_programa": "p-0001"
    }
}
```

---

### 5.2 gesfich

`gesfich` administra los ficheros registrados en `aralmac`. Cada fichero se identifica con el formato:

```text
f-XXXX
```

Ejemplo:

```text
f-0001
```

| Operación | `datos` esperados | Respuesta principal |
|---|---|---|
| `crear_fichero` | `{ "contenido": "texto inicial" }` o `{ "contenido": "" }` | Retorna `id_fichero`. |
| `leer_fichero` | `{ "id_fichero": "f-0001" }` | Retorna el contenido del fichero. |
| `leer_fichero` | `{}` | Lista los ficheros registrados. |
| `listar_ficheros` | `{}` | Lista los ficheros registrados. |
| `actualizar_fichero` | `{ "id_fichero": "f-0001", "ruta_fichero": "./datos/nuevo.txt" }` | Reemplaza el contenido en `aralmac`. |
| `borrar_fichero` | `{ "id_fichero": "f-0001" }` | Borra el fichero. |
| `suspender_servicio` | `{}` | Cambia el servicio a `suspendido`. |
| `reasumir_servicio` | `{}` | Cambia el servicio a `corriendo`. |
| `terminar_servicio` | `{}` | Cambia el servicio a `terminado`. |

Ejemplo para crear fichero:

```json
{
    "id_peticion": "req-0003",
    "servicio": "gesfich",
    "operacion": "crear_fichero",
    "datos": {
        "contenido": "3\n1\n2\n"
    }
}
```

Respuesta:

```json
{
    "id_peticion": "req-0003",
    "estado": "ok",
    "mensaje": "Fichero creado correctamente",
    "datos": {
        "id_fichero": "f-0001"
    }
}
```

Ejemplo para leer un fichero:

```json
{
    "id_peticion": "req-0004",
    "servicio": "gesfich",
    "operacion": "leer_fichero",
    "datos": {
        "id_fichero": "f-0001"
    }
}
```

Respuesta:

```json
{
    "id_peticion": "req-0004",
    "estado": "ok",
    "mensaje": "Fichero encontrado",
    "datos": {
        "id_fichero": "f-0001",
        "contenido": "3\n1\n2\n"
    }
}
```

Si `leer_fichero` no recibe `id_fichero`, se interpreta como una consulta general de los ficheros registrados.

Ejemplo para actualizar:

```json
{
    "id_peticion": "req-0005",
    "servicio": "gesfich",
    "operacion": "actualizar_fichero",
    "datos": {
        "id_fichero": "f-0001",
        "ruta_fichero": "./datos/entrada_actualizada.txt"
    }
}
```

En esta operación, actualizar significa reemplazar el contenido almacenado en `aralmac` por el contenido del fichero indicado en `ruta_fichero`.

Ejemplo para borrar:

```json
{
    "id_peticion": "req-0006",
    "servicio": "gesfich",
    "operacion": "borrar_fichero",
    "datos": {
        "id_fichero": "f-0001"
    }
}
```

---

### 5.3 ejecutor

`ejecutor` administra los procesos de lote. Cada lote se identifica con el formato:

```text
l-XXXX
```

Ejemplo:

```text
l-0001
```

Un lote recibe un fichero de entrada, un fichero de salida y una lista ordenada de programas.

Ejemplo conceptual:

```text
f-0001 -> p-0001 -> p-0002 -> f-0002
```

| Operación | `datos` esperados | Respuesta principal |
|---|---|---|
| `ejecutar_lote` | `{ "entrada": "f-0001", "salida": "f-0002", "programas": ["p-0001", "p-0002"] }` | Retorna `id_lote`. |
| `estado_lote` | `{ "id_lote": "l-0001" }` | Retorna el estado del lote. |
| `estado_lote` | `{}` | Lista el estado de todos los lotes. |
| `listar_lotes` | `{}` | Lista los procesos de lote. |
| `matar_lote` | `{ "id_lote": "l-0001" }` | Cambia el lote a `matado`. |
| `suspender_servicio` | `{}` | Cambia el ejecutor a `suspendido`. |
| `reasumir_servicio` | `{}` | Cambia el ejecutor a `corriendo`. |
| `parar_ejecutor` | `{}` | Detiene el ejecutor. |

Ejemplo para ejecutar:

```json
{
    "id_peticion": "req-0007",
    "servicio": "ejecutor",
    "operacion": "ejecutar_lote",
    "datos": {
        "entrada": "f-0001",
        "salida": "f-0002",
        "programas": ["p-0001", "p-0002"]
    }
}
```

Respuesta:

```json
{
    "id_peticion": "req-0007",
    "estado": "ok",
    "mensaje": "Proceso de lote iniciado correctamente",
    "datos": {
        "id_lote": "l-0001",
        "estado_lote": "corriendo"
    }
}
```

Ejemplo para consultar estado:

```json
{
    "id_peticion": "req-0008",
    "servicio": "ejecutor",
    "operacion": "estado_lote",
    "datos": {
        "id_lote": "l-0001"
    }
}
```

Si `estado_lote` no recibe `id_lote`, se interpreta como una consulta general equivalente a listar los procesos de lote.

Ejemplo para matar:

```json
{
    "id_peticion": "req-0009",
    "servicio": "ejecutor",
    "operacion": "matar_lote",
    "datos": {
        "id_lote": "l-0001"
    }
}
```

Ejemplo para listar:

```json
{
    "id_peticion": "req-0010",
    "servicio": "ejecutor",
    "operacion": "listar_lotes",
    "datos": {}
}
```

Respuesta:

```json
{
    "id_peticion": "req-0010",
    "estado": "ok",
    "mensaje": "Listado de procesos de lote",
    "datos": {
        "procesos": [
            {
                "id_lote": "l-0001",
                "estado_lote": "corriendo"
            },
            {
                "id_lote": "l-0002",
                "estado_lote": "matado"
            }
        ]
    }
}
```

---

## 6. Estados, control y errores

Estados de servicios:

```text
iniciado, corriendo, suspendido, terminado
```

Estados de programas y ficheros:

```text
activo, inactivo, borrado
```

Estados de lotes:

```text
pendiente, corriendo, suspendido, terminado, fallido, matado
```

Operaciones de control:

| Operación | Descripción |
|---|---|
| `suspender_servicio` | Suspende temporalmente un servicio. |
| `reasumir_servicio` | Reactiva un servicio suspendido. |
| `terminar_servicio` | Termina un servicio. |
| `parar_ejecutor` | Detiene el ejecutor. |

Errores principales:

| Código | Significado |
|---|---|
| `SERVICIO_INVALIDO` | El servicio no existe. |
| `OPERACION_INVALIDA` | La operación no existe para ese servicio. |
| `JSON_INVALIDO` | El mensaje no tiene formato JSON válido. |
| `DATOS_INCOMPLETOS` | Faltan campos obligatorios. |
| `PROGRAMA_NO_EXISTE` | El programa solicitado no existe. |
| `FICHERO_NO_EXISTE` | El fichero solicitado no existe. |
| `LOTE_NO_EXISTE` | El lote solicitado no existe. |
| `EJECUTABLE_INVALIDO` | El ejecutable no existe o no es válido. |
| `REFERENCIA_INVALIDA` | Algún identificador enviado no existe. |
| `LISTA_PROGRAMAS_VACIA` | No se enviaron programas para ejecutar. |
| `ERROR_INTERNO` | Error inesperado del servicio. |

---

## 7. Decisiones de diseño

Para esta entrega se toman estas decisiones:

1. Todos los mensajes serán JSON.
2. Las peticiones tendrán un `id_peticion`.
3. Las respuestas conservarán el mismo `id_peticion`.
4. Los programas usarán identificadores `p-XXXX`.
5. Los ficheros usarán identificadores `f-XXXX`.
6. Los lotes usarán identificadores `l-XXXX`.
7. `ctrllt` funcionará como pasarela.
8. `aralmac` no se fija todavía; después puede implementarse con archivos, base de datos o memoria.
9. La API será igual para Linux y Windows 11.
10. `ejecutar_lote` recibirá identificadores registrados, no rutas directas.

---

## 8. Primera entrega

Esta primera entrega incluye únicamente el diseño de la API. No incluye implementación de código.

El documento define:

- Componentes del sistema.
- Comunicación por tuberías nombradas.
- Diseño para Linux y Windows 11.
- Formato de mensajes JSON.
- Operaciones de `gesprog`, `gesfich` y `ejecutor`.
- Estados, errores y operaciones de control.

Con esto dejamos definido el contrato de comunicación entre los procesos. La implementación futura podrá hacerse en Linux y Windows 11 manteniendo la misma API y cambiando solo la forma de manejar las tuberías nombradas.
