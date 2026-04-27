# Practica-Sistemas-Operativos

# Ejecutor de lotes

Proyecto de Sistemas Operativos para el diseño de un sistema de ejecución de procesos por lotes.

## Primera entrega

Esta primera entrega consiste en el diseño de la API del sistema. No se incluye implementación de código fuente.

El documento principal de la entrega se encuentra en:

```text
docs/Diseño.md
```

## Descripción 

El sistema simula un ejecutor de procesos por lotes. Primero se registran programas y ficheros, y luego se ejecutan tareas usando esos recursos previamente registrados.

## Componentes principales

- `cliente`: envía peticiones al sistema.
- `ctrllt`: recibe las peticiones y las redirige al servicio correspondiente.
- `gesprog`: gestiona los programas registrados.
- `gesfich`: gestiona los ficheros registrados.
- `ejecutor`: ejecuta procesos de lote.
- `aralmac`: representa el área de almacenamiento.

## Comunicación

La comunicación entre los procesos se diseña mediante tuberías nombradas y mensajes en formato JSON.

## Sistemas operativos considerados

El diseño contempla una futura implementación en:

- Linux
- Windows 11
