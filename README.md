# Ciberseguridad sin drama para Devs · Lab de logs

Material de estudio del taller **"Ciberseguridad sin drama para Devs"** (GDG Pura Vida · DevFest 2026, sábado 21 de noviembre de 2026).
Aprenderás a leer **Cloud Audit Logs** como lo hace un analista de SOC, con consultas reales del Log Explorer de Google Cloud.

> **Todos los datos son sintéticos** (empresa, personas e IPs ficticias). No hay nada real en este repositorio.

## Empieza aquí (3 pasos)

1. Abre el visor: **https://lab.hidalgotechnologies.com**
2. Pega una consulta de [`CONSULTAS.md`](CONSULTAS.md) en el cuadro y pulsa **Ejecutar consulta** (o `Ctrl+Enter`).
3. Resuelve los **retos** del panel derecho (4 con respuesta y 2 de discusión).

## Cómo usar el visor

| Qué quieres hacer | Cómo |
|---|---|
| Ejecutar una consulta | Escríbela o usa un botón de ejemplo, y pulsa **Ejecutar consulta** / `Ctrl+Enter` |
| Ver el log completo | Haz clic en la fila: se abre el JSON del evento |
| Ver los campos anidados | Botón **Expandir campos anidados** dentro del JSON |
| Filtrar sin escribir | Haz clic en **cualquier valor** del JSON: se agrega como filtro a la consulta |
| Empezar de nuevo | Botón **Limpiar** |
| Ver cuántos eventos hay por hora | Gráfico sobre la tabla (gris = todo, azul = coincidencias) |

Cada fila muestra severidad, hora local, y un resumen con el método y quién lo hizo, igual que el Log Explorer.

## Sintaxis de consulta (la misma de GCP)

| Quiero | Escribo |
|---|---|
| Igual | `campo="valor"` |
| Distinto | `campo!="valor"` |
| Contiene | `campo:"texto"` |
| Expresión regular | `campo=~"regex"` |
| El campo existe | `campo:*` |
| Severidad mínima | `severity>=ERROR` |
| Rango de tiempo | `timestamp>="2026-11-12T21:00:00Z"` |
| Y / O / NO | `A AND B` · `A OR B` · `NOT A` o `-A` |

Reglas importantes:

- Un **espacio** o un salto de línea entre condiciones equivale a `AND`.
- Precedencia: `NOT` > `OR` > `AND`. Ante la duda, usa paréntesis.
- `=` **no distingue mayúsculas** (sí lo hacen las expresiones regulares y los operadores `AND`/`OR`/`NOT`, que van en mayúsculas).
- Cuidado con los arreglos: `bindingDeltas` contiene altas **y** bajas. Para ver solo cuando se hizo público usa `action="ADD"`.

### Diferencias entre el visor y GCP

El visor implementa un **subconjunto** del lenguaje, suficiente para el taller:

- No existen `SEARCH()` ni `ip_in_net()`.
- La regex del visor es de JavaScript y la de GCP es RE2 (con patrones simples dan lo mismo).
- El visor trae un solo proyecto sintético; en GCP verás **tu** proyecto.

## Probar en tu propio Google Cloud (opcional)

Las mismas consultas funcionan en **Logs Explorer** (Cloud Logging → Logs Explorer → pegar la consulta → *Run query*).
Verifica que el rango de tiempo cubra el evento (por ejemplo, últimas 24 horas).
Para generar actividad tú mismo: crea un bucket, cámbiale los permisos y revisa los eventos `storage.setIamPermissions`.
Hazlo en un proyecto de prueba, no en producción.

## Contenido del repositorio

```
README.md          Esta guía
CONSULTAS.md       Las consultas del taller, explicadas
consultas.json     Las mismas consultas en JSON
```

## Verificación

Cada consulta fue ejecutada en un proyecto real de Google Cloud y en el visor sobre los **mismos eventos**; devolvieron exactamente los mismos resultados.

## Aviso

Material educativo. Los logs son ficticios; los eventos de ejemplo no representan a ninguna empresa real.
