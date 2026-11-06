# Reglas de formato Excel (Modelo relacional)

## 1. Distribución en la hoja

- **Independencia de bloques:** Cada tabla se diseña de forma independiente y se organiza en su propio bloque de columnas.
- **Flujo visual:** Bloques ordenados de izquierda a derecha, siguiendo la posición de las clases en el diagrama (no el orden de creación del SQL).
- **Separación:** Exactamente 1 columna vacía entre bloques adyacentes.
- **Nomenclatura transversal:** Nombres de tablas y atributos en MAYÚSCULAS, sin espacios ni guiones bajos (`_`), y nombres de tabla en singular.

## 2. Estructura de filas

### Fila 1: Nombre de la tabla

- Nombre de la tabla en MAYÚSCULAS y en singular.
- Estrictamente sin espacios ni guiones bajos (`_`).
- Sin formato especial (sin negrita ni subrayado).
- **Sin bordes**.

### Fila 2: Indicadores de clave (Rol)

Define el rol del atributo inferior inmediato. **Sin bordes**.

- `PK`: Clave primaria propia.
- `FK`: Clave foránea (aplica a asociaciones simples, agregaciones, tablas intermedias y composiciones).
  - *Regla de composición:* La PK de la tabla de la parte es compuesta: la PK del todo (heredada) más un atributo propio (discriminador). Todos los atributos de esa PK van en Negrita + Subrayado. En la fila 2 solo se marca `FK` sobre el atributo heredado (aunque forme parte de la PK compuesta); el atributo propio (discriminador) queda con la celda vacía, porque su Negrita + Subrayado ya indica que es PK.
- *Celda vacía:* Si el atributo no es clave.

### Fila 3: Formato tipográfico de los atributos

Los nombres de los atributos varían su estilo según su rol relacional. **Con bordes**.

- **Nomenclatura general:** En MAYÚSCULAS, palabras compuestas unidas sin espacios ni guiones bajos (`_`). Se omiten tipos de datos (definidos exclusivamente en la fase SQL).
- **Nomenclatura de claves foráneas (FK):** Pasan con la estructura `LLAVE + CLASE` (nombre de la clave primaria origen + nombre o sufijo abreviado de la clase origen). Aplica a toda foránea, incluidas las heredadas en tablas de composición e intermedias.
  - *Ejemplo:* Clase `PRODUCTO`, atributo clave `CODIGO` -> FK: `CODIGOPROD` o `CODIGOP`.
  - *Una sola forma por clase:* Para una misma clase origen se usa siempre el mismo sufijo en todo el modelo (completo o abreviado).
  - *FK a una subclase:* El sufijo es la subclase del diagrama aunque la tabla física sea la de la superclase (`CICLIENTE` y `CIENCARGADO` apuntan a `PERSONA.CI`).
  - *Mismo nombre en el SQL:* El nombre de la FK en el Excel es el nombre exacto de la columna en el SQL.
- **Negrita + Subrayado:** PK propia, parte propia de una PK compuesta o clave heredada por composición (al ser parte funcional de la PK compuesta).
- **Negrita + Cursiva (sin subrayado):** FK estándar o claves heredadas en tablas intermedias.
- **Texto normal:** Atributos no clave.

### Fila 4 en adelante: Registros de datos

Filas con los datos propios de la tabla. **Con bordes**.

- Si no hay datos disponibles, dejar **3 filas vacías con bordes** para mantener la estructura visual del modelo.
- **Tipo de valor en celda:** Números como número y fechas como fecha (nunca texto como `'599.75'`). Sin hipervínculos automáticos en correos, para que el subrayado no se confunda con el de las claves.
- **Datos coherentes:** Cada FK con valor debe existir en la PK referenciada, y cada fila respeta la multiplicidad mínima del diagrama.

## 3. Matriz de estilos y roles de atributos

| Rol del Atributo | Fila 2 (Indicador) | Fila 3 (Estilo de Texto) | Bordes de Celda |
| --- | --- | --- | --- |
| **PK Propia** | `PK` | Negrita + Subrayado | Sí (desde Fila 3) |
| **Parte propia de PK compuesta (discriminador)** | *(Vacío)* | Negrita + Subrayado | Sí (desde Fila 3) |
| **Clave heredada por Composición** | `FK` | Negrita + Subrayado | Sí (desde Fila 3) |
| **FK Estándar / Tabla Intermedia** | `FK` | Negrita + Cursiva (sin subrayado) | Sí (desde Fila 3) |
| **Atributo No Clave** | *(Vacío)* | Texto Normal | Sí (desde Fila 3) |

## 4. Bordes y cuadrícula

- **Con bordes:** Aplicar bordes de celda exclusivamente desde la **Fila 3** hasta la última fila de datos (o las 3 filas vacías si no existen registros).
- **Sin bordes:** Las **Filas 1 y 2** nunca llevan bordes.

## 5. Generalización

- **Tabla única:** La superclase y sus subclases forman un solo bloque con el nombre de la superclase. Las subclases no generan bloque propio.
- **Atributos de subclase:** Texto normal, sin `PK` ni `FK`, en el mismo bloque.
- **Solapamiento (completo o incompleto):** Un flag por subclase, llamado `TIPO` + inicial de la subclase (`TIPOE` encargado, `TIPOC` cliente), con valores `V` / `F`. Si dos subclases comparten inicial se usa un sufijo más largo.
- **Disjunto (completo o incompleto):** Un único atributo `TIPO` con la inicial de la subclase como valor.
- **Datos:** Los flags y atributos de subclase de cada fila reflejan el tipo; los atributos de otra subclase quedan vacíos.
