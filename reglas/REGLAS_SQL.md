# Reglas de formato SQL

## 1. Estructura general del script

Orden fijo, de arriba hacia abajo:

1. `CREATE DATABASE` y `USE` (siempre primeras dos líneas).
2. Tablas, en orden de dependencia: primero las que no tienen FK, después las que sí. Así nunca se referencia una tabla que aún no existe.
3. Datos (`INSERT`).
4. `SELECT * FROM` de verificación, justo después de los `INSERT` de cada tabla principal. Solo sirve para revisar los datos; su ubicación es estética y no afecta el funcionamiento.
5. Actualizaciones y borrados (`UPDATE`, `DELETE`), si los hay.
6. Consultas, cada una con su comentario en español.

```sql
CREATE DATABASE NOMBREBD;
USE NOMBREBD;
 
CREATE TABLE TABLA1 (...);
CREATE TABLE TABLA2 (...);
 
INSERT INTO TABLA1 VALUES (...);
 
-- MOSTRAR LOS REGISTROS DE TABLA1
SELECT * FROM TABLA1;
```

## 2. Nombres

- Base de datos, tablas y columnas en MAYÚSCULAS, sin guiones bajos ni espacios (regla transversal compartida con Draw.io y Excel).
- Columnas compuestas por palabras juntas: `NOMBRECOMPLETO`, `FECHANAC`, `IDPADRE`, `CIABOGADO`.
- Nombres de tabla en singular: `PERSONA`, `ALUMNO`, `ASUNTO`.
- Tablas y columnas llevan exactamente los mismos nombres que el diagrama y el Excel.
- Los valores de texto que se insertan (nombres de personas, motivos) van en mayúscula inicial o en mayúsculas, pero siempre consistentes dentro del mismo script. Los scripts mezclan ambos casos y eso es un error que hay que evitar.

## 3. Definición de tablas

Paréntesis en línea propia, una columna por línea, indentación de 4 espacios.

```sql
CREATE TABLE PERSONA
(
    ID SMALLINT NOT NULL PRIMARY KEY,
    NOMBRE VARCHAR(50) NOT NULL,
    SEXO CHAR NOT NULL,
    IDPADRE SMALLINT,
    IDMADRE SMALLINT,
    FOREIGN KEY (IDPADRE) REFERENCES PERSONA(ID)
        ON UPDATE CASCADE
        ON DELETE NO ACTION,
    FOREIGN KEY (IDMADRE) REFERENCES PERSONA(ID)
        ON UPDATE CASCADE
        ON DELETE NO ACTION
);
```

- `PRIMARY KEY` inline cuando es una sola columna; al final cuando es compuesta.
- Las FK siempre con `ON UPDATE CASCADE` y un `ON DELETE` explícito.
- **ON DELETE:** Es preferencia del usuario en cada ejecución. Al presentar el modelo lógico, el asistente sugiere `CASCADE` en todas las FK (lo habitual) y pregunta si alguna relación debe ser `NO ACTION`. La sugerencia advierte que `CASCADE` borra historiales y, en autorreferencias, todo el subárbol. Si el usuario no indica nada, se usa `CASCADE`. El ejemplo de arriba usa `NO ACTION` como ilustración de esa preferencia.
- **Nulabilidad desde la multiplicidad:** Se lee el mínimo de la etiqueta junto a la clase referenciada. Mínimo `1` → FK `NOT NULL`. Mínimo `0` → FK sin `NOT NULL`. Las FK que forman parte de una PK compuesta (composición) son siempre `NOT NULL`.
- **Tipo de la FK:** Idéntico al de la PK que referencia.
- **Mínimos del lado "muchos":** Un `1..*` en la tabla referenciante (cada marca con al menos un producto) no se puede imponer con DDL; queda documentado solo en el diagrama.
- Sin nombres explícitos para constraints.

## 4. Tipos de datos

A diferencia del diagrama (Draw.io) y del modelo relacional en Excel (donde se omiten los tipos), el script SQL exige la definición estricta de tipos de datos para cada columna.

| Uso | Tipo |
| --- | --- |
| Identificadores pequeños | `SMALLINT` |
| Identificadores grandes o documentos (CI) | `INT` |
| Números de documento y cantidades (`NRO`, `NIT`, `CANTIDAD`) | `INT` |
| Plazos | `SMALLINT` |
| Correlativo dentro de una composición (ítem, nro de cuota) | `TINYINT`, el mismo tipo en todas las composiciones |
| Montos (precio, total, monto) | `DECIMAL(10,2)`; nunca `DECIMAL` sin precisión |
| Nombres | `VARCHAR(50)` |
| Nombre de producto, descripción | `VARCHAR(100)` |
| Direcciones | `VARCHAR(100)` |
| Correo | `VARCHAR(50)` |
| Cargo | `VARCHAR(30)` |
| Catálogos cortos (tipo de pago) | `VARCHAR(20)` |
| Código con formato fijo (`P001`) | `VARCHAR(n)` con `n` = largo del formato |
| Siglas | `VARCHAR(6)` a `VARCHAR(10)` |
| Sexo, estado civil, flags | `CHAR` (flags de subclase con `'V'` / `'F'`) |
| Semestres, días | `TINYINT` |
| Fechas | `DATE` (`'YYYY-MM-DD'`) |
| Horas | `TIME` (`'HH:MM'`) |
| Teléfonos | `INT` |

Teléfonos como `INT`, no como `VARCHAR`, porque así lo hacen los scripts.

## 5. Inserción de datos

Un `INSERT` por línea, con `VALUES` sin lista de columnas, como en todos los scripts actuales. Solo cuando la tabla tiene muchos registros se usa INSERT múltiple.

```sql
INSERT INTO PERSONA VALUES (1,'JOAQUIN CHUMACERO COLQUE','M',NULL,NULL);
INSERT INTO PERSONA VALUES (2,'MARIA ARANCIBIA','F',NULL,NULL);
```

- Números y `NULL` sin comillas; cadenas, fechas y horas entre comillas simples.
- Cuando una tabla tiene más de unos 10 registros, usa INSERT múltiple con `VALUES` seguidos de tuplas separadas por comas:

```sql
INSERT INTO CLIENTE (CI, NOMBRE, DIRECCION, TELEFONO) VALUES
(111, 'JOAQUIN CHUMACERO', 'CALLE JUNIN 200', 7102030),
(222, 'SATURNINO MAMANI', 'CALLE LANDIVAR 40', 7301011);
```

## 6. Actualizaciones y borrados

```sql
-- Cambiar el nombre del registro 10
UPDATE PERSONA
SET NOMBRE='PEDRO CHUMACERO CESPEDES'
WHERE ID=10;
 
-- Eliminar el registro con REGISTRO 444
DELETE FROM ALUMNO
WHERE REGISTRO=444;
```

- `SET` y `WHERE` en líneas separadas.
- `WHERE` siempre presente en `UPDATE` y `DELETE`. Sin WHERE se borran o cambian todas las filas.
- `DROP DATABASE` no va en el script principal. Solo se usa como comentario, con `/* ... */`, para no borrar la base por accidente.

## 7. Consultas SELECT

Cada consulta lleva un comentario en español en MAYÚSCULAS antes de ella, en el formato `-- MOSTRAR ...`.

```sql
-- MOSTRAR EL PADRE DE 'JOAQUIN CHUMACERO AGUILERA'
SELECT PADRE.ID, PADRE.NOMBRE
FROM PERSONA HIJO, PERSONA PADRE
WHERE HIJO.IDPADRE=PADRE.ID AND HIJO.NOMBRE='JOAQUIN CHUMACERO AGUILERA';
```

- JOIN implícito (`FROM` con varias tablas y condiciones en `WHERE`). Nada de `INNER JOIN`.
- Alias descriptivos en mayúsculas según el rol de la tabla: `HIJO`, `PADRE`, `MADRE`, `ABUELO`, `CLIENTE`, `ABOGADO`.
- Condiciones en el `WHERE` en una sola línea si son pocas; si son muchas, una por línea con `AND` al inicio.
- `DISTINCT` cuando el resultado pueda repetir filas, por ejemplo al llegar a madres de varios hijos.
- Subconsultas con `IN` para búsquedas por nivel, con el mismo estilo de indentación de los scripts:

```sql
-- ANIDADO
SELECT ID, NOMBRE
FROM PERSONA
WHERE ID IN (
            SELECT IDPADRE
            FROM PERSONA
            WHERE NOMBRE='JOAQUIN CHUMACERO AGUILERA'
            );
```

- Para conteos y agrupamientos: `COUNT(*)` o `COUNT(columna)`, con `GROUP BY` al final.

```sql
-- MOSTRAR LA CANTIDAD DE ALUMNOS POR SEXO
SELECT SEXO, COUNT(*)
FROM ALUMNO
GROUP BY SEXO;
```

## 8. Comentarios

- `--` para comentarios de una línea, en español, antes de cada consulta o bloque.
- `/* ... */` para bloques largos o código desactivado temporalmente, como las consultas de ABOGADOS que quedaron comentadas.

## 9. Errores que hay que evitar (encontrados en los scripts)

Estos errores aparecen en los scripts actuales y conviene no repetirlos:

- Nombres de columna mal escritos: `REGISTTRO`, `MOSRAR`, `LTETRA`, `ABOGADOS` que no coincide con la tabla `ABOGADO`.
- Nombre de tabla o columna distinto al del Excel o el diagrama (por ejemplo `TIPODEPAGO` frente a `TIPOPAGO`, o `IDCATEGORIA` frente a `IDCATE`).
- Tabla referenciada antes de crearse: en `GRUPO` se usa `REFERENCES MATERIA` antes de que exista `MATERIA`.
- `FROM MATERIAS` cuando la tabla es `MATERIA`.
- Consultas incompletas, como `WHERE` sin condición al final de un archivo.
- `ON DELETE CASCADE` sobre historiales (como `ASUNTO`): es el valor por defecto, pero la sugerencia al usuario debe advertir que se pierde historial.
- `DECIMAL` sin precisión: trunca los centavos (en MySQL equivale a `DECIMAL(10,0)`).
- FK `NOT NULL` hacia una asociación con mínimo `0` en el diagrama: impide insertar filas que el modelo permite (por ejemplo una cuota aún sin factura).
- Mezcla de mayúsculas y minúsculas en los valores insertados y en las condiciones `WHERE`. Un `WHERE NOMBRE='Fabiola Mendez'` no encuentra `'FABIOLA MENDEZ'` si el collation es sensible; mantener un solo criterio.

## 10. Snippets de referencia

Tabla con FK autorreferencial (ver punto 3; el ejemplo usa `NO ACTION` por preferencia del usuario).

Tabla intermedia con PK compuesta:

```sql
CREATE TABLE LLEVA
(
    NROASUNTO INT NOT NULL,
    CIABOGADO INT NOT NULL,
    FECHAASIG DATE NOT NULL,
    PRIMARY KEY (NROASUNTO, CIABOGADO),
    FOREIGN KEY (NROASUNTO) REFERENCES ASUNTO(NRO)
        ON UPDATE CASCADE
        ON DELETE CASCADE,
    FOREIGN KEY (CIABOGADO) REFERENCES ABOGADO(CI)
        ON UPDATE CASCADE
        ON DELETE CASCADE
);
```

Tabla de composición con FK opcional (mínimo `0`, sin `NOT NULL`):

```sql
CREATE TABLE CUOTA
(
    NROV INT NOT NULL,
    NROCUOTA TINYINT NOT NULL,
    FECHA DATE NOT NULL,
    MONTO DECIMAL(10,2) NOT NULL,
    NROFACT INT,
    PRIMARY KEY (NROV, NROCUOTA),
    FOREIGN KEY (NROV) REFERENCES NOTAVENTA(NRO)
        ON UPDATE CASCADE
        ON DELETE CASCADE,
    FOREIGN KEY (NROFACT) REFERENCES FACTURA(NRO)
        ON UPDATE CASCADE
        ON DELETE CASCADE
);
```

Consulta con agrupamiento y nombre:

```sql
-- MOSTRAR LA CANTIDAD DE ASUNTOS QUE TIENE CADA CLIENTE
SELECT C.CI, C.NOMBRE, COUNT(*) AS CANTIDAD
FROM CLIENTE C, ASUNTO A
WHERE C.CI=A.CICLIENTE
GROUP BY C.CI, C.NOMBRE;
```

## 11. Generalización

- **Tabla única:** La superclase y sus subclases forman una sola tabla con el nombre de la superclase. Los atributos propios de cada subclase van sin `NOT NULL`.
- **Solapamiento (completo o incompleto):** Un flag `CHAR NOT NULL` por subclase, llamado `TIPO` + inicial de la subclase (`TIPOE`, `TIPOC`), con valores `'V'` / `'F'`.
- **Disjunto (completo o incompleto):** Un único `TIPO CHAR` con la inicial de la subclase. `NOT NULL` si es completo; sin `NOT NULL` si es incompleto.
- **Solapamiento completo:** Recomendado, si el motor aplica `CHECK` (MySQL 8.0.16 o superior), añadir `CHECK (TIPOE='V' OR TIPOC='V')` para que toda fila sea al menos una subclase.
- **Solapamiento incompleto:** Se permite que todos los flags sean `'F'`.
- **FK hacia una subclase:** Referencia la tabla de la superclase y se nombra con la subclase (`CICLIENTE`, `CIENCARGADO`).

```sql
CREATE TABLE PERSONA
(
    CI INT NOT NULL PRIMARY KEY,
    NOMBRE VARCHAR(50) NOT NULL,
    TELEFONO INT NOT NULL,
    CARGO VARCHAR(30),
    DIRECCION VARCHAR(100),
    CORREO VARCHAR(50),
    TIPOE CHAR NOT NULL,
    TIPOC CHAR NOT NULL
);
```
