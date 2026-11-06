# Reglas de formato Draw.io (Diagrama de clases - Estilo StarUML)

## 1. Lienzo y tipografía

- **Lienzo:** Fondo blanco, cuadrícula visible, sin colores de relleno ni sombras. Esquema estricto en blanco y negro.
- **Tipografía (Arial / Helvetica):**
  - **Nombre de clase:** Tamaño 11–12 pt, color negro.
  - **Atributos y etiquetas:** Tamaño 10–11 pt, color negro.

## 2. Clases

- **Estructura:** Rectángulo de dos compartimentos (`Nombre` | `Atributos`) divididos por un separador horizontal.
- **Nombre:** MAYÚSCULAS, en negrita, centrado y en singular. Estrictamente sin espacios ni guiones bajos (`_`).
- **Compartimentos:**
  - El compartimento de atributos permanece visible aunque esté vacío.
  - Sin compartimento de métodos.

## 3. Atributos

- **Formato:** Uno por línea, alineados a la izquierda y en MAYÚSCULAS. Palabras compuestas unidas sin espacios ni guiones bajos (`_`).
- **Prefijo:** Visibilidad pública indicada con `+`.
- **Tipo de dato:** Sin tipos de datos (el diagrama omite tipos; se definen exclusivamente en la fase SQL).
- **Claves foráneas:** No se dibujan como atributos. Cada FK se representa mediante su asociación, composición o agregación; la columna FK aparece recién en el Excel y en el SQL.

## 4. Asociación

- **Trazo:** Línea sólida fina, recta u ortogonal, sin flechas ni rombos. Excepciones: rombos de composición y agregación (§6) y triángulo hueco de generalización (§9).
- **Etiqueta:** Nombre con prefijo `+` centrado sobre la línea, sin guiones bajos (`_`). Si el trazo es vertical, se ubica al costado y centrado verticalmente.

## 5. Multiplicidad

Ubicación: Una etiqueta por extremo, adyacente a su clase correspondiente.

| Criterio | Regla |
| --- | --- |
| **Semántica** | La etiqueta junto a una clase indica cuántas instancias de ella corresponden por cada instancia de la clase opuesta; es la lectura de la asociación que llega a dicha clase (ej. "de uno a muchos" = `1..*`). |
| **Escritura** | Se escribe siempre con los dos valores `mín..máx`. En trazos horizontales el máximo debe quedar del lado de la clase y el mínimo del lado de la línea: si la clase queda a la derecha de la etiqueta se lee `mín..máx` (ej. `1..*`); si queda a la izquierda se lee `máx..mín` (ej. `*..1`). En trazos verticales se escribe siempre `mín..máx`. |
| **Sintaxis** | Enteros (`1`), rangos cerrados (`n..m`), o asterisco (`*`) para máximo abierto. El mínimo nunca puede ser mayor que el máximo. |

## 6. Composición y agregación

Ambas son asociaciones todo–parte. El rombo va siempre en el extremo del **todo** (clase contenedora), pegado a su borde. Las multiplicidades son fijas: no se eligen, las determina el tipo de relación, y **se dibujan siempre**, aunque se den por supuestas.

| Tipo | Rombo (extremo del todo) | Junto a la parte | Junto al todo |
| --- | --- | --- | --- |
| Composición | ◆ relleno | `1..*` | `1..1` |
| Agregación | ◇ vacío | `0..*` | `1..1` |

- **Lectura (según §5):** Cada todo tiene `1..*` partes (composición) o `0..*` partes (agregación); cada parte pertenece a exactamente un todo.
- **Orden de escritura:** Los valores de la tabla son lógicos; el orden mín/máx de cada etiqueta sigue §5 según la orientación del trazo. Ej. en trazo horizontal con la parte a la izquierda y el todo a la derecha: `*..1` junto a la parte y `1..1` junto al todo (`[PARTE] *..1 ──◆ 1..1 [TODO]`).
- **Línea y nombre:** Igual que una asociación (§4): línea sólida fina, recta u ortogonal, sin flechas, con nombre `+nombre` centrado. Todo vínculo todo–parte lleva nombre.
- **Agregación:** Si la parte puede existir sin un todo, no es una agregación: se modela como asociación simple con mínimo `0` en el lado del todo.
- **Efecto en el modelo relacional:** En la composición la parte no existe sin el todo: su PK incluye la PK del todo (FK + parte de la PK compuesta). En la agregación la parte conserva PK propia y recibe la PK del todo como FK estándar, `NOT NULL` por el `1..1`.

## 7. Clase de asociación

- **Estructura:** Bloque con el mismo formato estándar de clase.
- **Conexión:** Unido al punto medio de la línea de asociación mediante una línea punteada sin flechas.

## 8. Asociación reflexiva

- **Estructura:** Bucle rectangular que sale y reentra en la misma clase.
- **Anotaciones:** Contiene nombre de relación y multiplicidades en ambos extremos.
- **Escritura de etiquetas:** Cada etiqueta del bucle sigue §5 según el tramo donde se ubica (vertical: `mín..máx`; horizontal: según el lado de la clase). Un rango como `1..0` es inválido; se escribe `0..1`.

## 9. Generalización (herencia)

- **Trazo:** Línea sólida fina desde cada subclase hacia la superclase, terminada en triángulo hueco (sin relleno) pegado a la superclase. Sin nombre ni multiplicidad.
- **Restricción:** Texto plano junto a los triángulos, con coma y espacio: `Solapamiento, completo`, `Solapamiento, incompleto`, `Disjunto, completo` o `Disjunto, incompleto`.
- **Subclases:** Siguen las reglas de clase (§2): MAYÚSCULAS, negrita, singular. Contienen solo sus atributos propios.

## 10. Distribución y trazado

- **Flujo:** Bloques organizados de izquierda a derecha con separación uniforme.
- **Espaciado:** Espacio suficiente para que las multiplicidades se lean con claridad sin solaparse.
- **Trazos:** Rectos u ortogonales, evitando cruces innecesarios entre líneas y clases.
