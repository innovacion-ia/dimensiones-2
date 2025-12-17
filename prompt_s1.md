## Prompt para análisis del Segmento S1 (TRANSCRIPCIONES de llamadas de venta / no venta)

### Rol
Actúa como **Analista de Datos Senior** especializado en **Insights de Clientes y Estrategia de Ventas** para seguros de salud/oncológicos (Auna/Oncosalud).  
Tu salida debe ser **estructurada, consistente y directamente utilizable** para análisis y toma de decisiones comerciales.  
**Muy importante:** trabajarás siempre sobre **TRANSCRIPCIONES de llamadas telefónicas**, no sobre audio bruto.

---

### Input
Trabaja usando:

1. **`s1_final_para_categorias_sin_dni.csv`**  
   - Contiene **transcripciones de llamadas históricas** del segmento S1 (ventas y no ventas).  
   - Incluye: texto de la transcripción con roles (`client:`, `operator:`) y resultado de la llamada (venta/no venta).  
   - Todas las definiciones y categorizaciones deben basarse **exclusivamente** en esta data.

---

### 0. Definición y filtrado de “transcripciones válidas”

Antes de cualquier cálculo o clasificación:

1. Considera como **transcripción válida** únicamente aquellas donde:
   - Existen **al menos 2 turnos de `client:`** con contenido semántico (no solo “hola”, “aló”, “sí dígame”, “¿quién habla?”, etc.).  
   - El texto muestra una **mínima conversación** donde el cliente expresa alguna postura, duda, interés, objeción o comentario relacionado con el producto, la salud, la familia, el precio, la marca, etc.

2. Considera como **ruido de transcripción** y **excluye del análisis principal**:
   - Transcripciones con **0 o 1 turno de `client:`** con contenido útil.  
   - Transcripciones que reflejan llamadas cortadas de forma inmediata (cortes técnicos, silencio, texto vacío, error de grabación).  
   - Registros donde el cliente nunca llega a hablar (no aparece `client:`).

3. Reglas:
   - **No incluyas transcripciones no válidas** en ningún conteo de Comportamientos, Motivadores ni Barreras.  
   - Puedes reportar por separado el % de transcripciones S1 descartadas como “no válidas”, pero no las mezcles con el universo de análisis.

A partir de aquí, cuando se mencione “total de S1”, se entiende **total de TRANSCRIPCIONES válidas de llamadas S1 después del filtrado**.

---

### 1. Definición de dimensiones y categorías (sin solapamientos)

Usa SIEMPRE estas definiciones operativas, ya integradas al contexto del S1 (hombre casado, consolidado, con alta carga de responsabilidades y tiempo limitado):

- **Comportamientos (El Estilo de Interacción)**  
  Es la **personalidad y conducta observable del cliente durante la llamada**, vista en la transcripción.  
  - Describe **cómo** el S1 interactúa con el asesor: ritmo, cantidad de turnos, nivel de escucha, grado de cuestionamiento, validación de datos, interrupciones, cortes, etc.  
  - Debe capturar el hecho de que S1 es un perfil **con alta responsabilidad económica y familiar**, generalmente con **poco tiempo disponible y fuerte necesidad de control racional**, por lo que puede:
    - Cerrar rápido para proteger su tiempo.  
    - Mantener un diálogo cordial pero superficial.  
    - Entrar en modo altamente racional-financiero (pregunta, compara, pide números).  
  - Comportamiento **no es lo mismo** que interés ni motivador: un cliente puede tener un comportamiento muy analítico pero no comprar si las barreras predominan.  
  - Es la variable que dicta **cómo debe adaptarse el estilo de comunicación del vendedor** (ritmo, profundidad, foco en números vs historias, etc.).

- **Motivadores (El Detonante de Compra)**  
  Son los **impulsores específicos, racionales o emocionales**, que actúan como detonantes para que el cliente tome una decisión de compra en el **momento presente**, evitando la postergación.  
  - Se identifican por un **cambio cualitativo en la actitud del cliente**: pasa de defensivo/pasivo a interés activo.  
  - Se manifiestan cuando el cliente **reconoce un valor inmediato** y lo verbaliza o formula **preguntas de confirmación/validación** (ej. familia, riesgo, prevención, confianza en la marca).

- **Barreras (El Freno de Venta)**  
  Son los **obstáculos explícitos o implícitos** que detienen el progreso hacia el cierre.  
  - Incluyen **objeciones reales** (precio, redundancia, limitaciones de tiempo) y **excusas evasivas** (“más adelante”, “no estoy interesado”) usadas para terminar la interacción por falta de confianza o interés.  
  - Si no se gestionan, **resultan en la caída inmediata de la venta**.

Con estas definiciones:

1. Construye tres conjuntos de categorías, uno por dimensión:
   - **Comportamientos**: patrones de estilo de interacción (ej. interacción mínima, diálogo colaborativo, perfil racional-financiero, etc.).  
   - **Motivadores**: detonantes verbalizados (ej. riesgo/cáncer, protección familia/hijos, chequeo preventivo, confianza en la marca, otros).  
   - **Barreras**: frenos u objeciones (ej. redundancia con EPS, precio, falta de tiempo, postergación, reclamos, desinterés, otros).

2. Condición crítica de exclusividad:
   - Para cada **dimensión** (Comportamiento / Motivador / Barrera), **cada transcripción válida debe pertenecer a exactamente una categoría principal**.  
   - Si en una misma transcripción aparecen señales de varias categorías de la misma dimensión, define y aplica una **regla de prioridad clara** para elegir **solo una** (documenta la prioridad usada).  

3. Temas que deben estar contemplados **explícitamente** en las categorías, si aparecen en el texto del cliente:

   - **Motivadores (ejemplos esperados en la transcripción)**:
     - Riesgo/cáncer (propio o de familiares).  
     - Protección de familia e hijos.  
     - Chequeo preventivo / exámenes de despistaje.  
     - Reputación / confianza en Auna–Oncosalud.  
     - Otros motivadores que el cliente verbalice claramente.

   - **Barreras (ejemplos esperados en la transcripción)**:
     - Redundancia: ya tiene EPS/seguro y percibe la oferta como duplicada.  
     - Precio / percepción de que es caro o fuera de presupuesto.  
     - Falta de tiempo / interrupción de su rutina.  
     - Postergación (“más adelante”, “lo voy a pensar”).  
     - Reclamos / mala experiencia previa con Oncosalud–Auna.  
     - Desinterés / baja prioridad.  
     - Otras barreras explícitas que verbalice.

4. Categoría de “sin motivador explícito”:
   - Si la necesitas, nómbrala como **“motivador no verbalizado / no observado”**.  
   - Úsala solo cuando, en el texto de `client:`, **no aparezca ninguna razón concreta de interés** (familia, riesgo, prevención, marca, etc.) y no haya evidencia suficiente para asignar un motivador específico.  
   - Deja claro que un % alto en esta categoría indica **problema de guion o codificación**, no ausencia real de motivadores.

---

### 2. Tabla base por dimensión

Para cada dimensión (Comportamiento, Motivador, Barrera), genera una tabla con las columnas:

- `dimension`  
  - Valores posibles: `"Comportamiento"`, `"Motivador"`, `"Barrera"`.

- `categoria`  
  - Nombre corto, único y entendible para negocio.

- `descripcion`  
  - 1–2 oraciones claras que describan **la lógica de asignación** de esa categoría según el texto de la transcripción.

- `recuento_llamadas`  
  - Número de **transcripciones válidas S1** que pertenecen a esa categoría.

- `porcentaje_llamadas`  
  - `recuento_llamadas / total_transcripciones_validas_S1 * 100`.

**Condiciones de consistencia**:

- La suma de `recuento_llamadas` por dimensión debe ser exactamente igual al total de transcripciones válidas S1.  
- La suma de `porcentaje_llamadas` por dimensión debe ser ~100% (solo tolerar pequeñas diferencias por redondeo).  
- Ninguna transcripción válida puede estar en más de una categoría de la misma dimensión.

---

### 3. Venta vs no venta por categoría

Sobre las mismas categorías, agrega estas columnas:

- `porc_ventas_categoria`  
  - `% de todas las ventas S1 válidas (según transcripción) que pertenecen a esta categoría`.  
  - Denominador: total de transcripciones válidas S1 con resultado **venta**.

- `porc_no_ventas_categoria`  
  - `% de todas las no ventas S1 válidas (según transcripción) que pertenecen a esta categoría`.  
  - Denominador: total de transcripciones válidas S1 con resultado **no venta**.

- `conversion_categoria`  
  - `ventas_en_categoria / total_transcripciones_en_categoria * 100`.

**Objetivo**: comparar fácilmente qué categorías se asocian fuertemente a **venta** y cuáles a **no venta**, usando solo evidencia de las transcripciones.

---

### 4. Ejemplos de transcripción (evidencia cualitativa)

Agrega una columna:

- `ejemplos`  
  - Debe contener **2–3 mini extractos textuales** de `client:` representativos de esa categoría.  
  - Requisitos:
    - Frases **cortas y anónimas** (sin datos personales, DNIs, direcciones, etc.).  
    - Deben expresar claramente el patrón de la categoría tal como aparece en la transcripción.  
    - Separa los ejemplos con ` | ` en la misma celda.

Ejemplo de formato:
- `"No señorita, ya tengo seguro, gracias. | Más que nada quiero algo para mis hijos."`

---

### 5. Recomendaciones tácticas

Agrega una columna:

- `recomendacion`  
  - 1 oración (máximo 2) que indique **qué acción concreta de guion o gestión comercial** se recomienda cuando se detecta esa categoría, basada en lo que dice el cliente en la transcripción.  
  - Debe estar directamente conectada con el patrón observado (por ejemplo, cómo aprovechar un motivador o cómo gestionar una barrera).

### Reglas obligatorias:
- Máximo 2 oraciones cortas.
- **No usar comas (,)**.
- Usar conectores simples como `y`, `porque`, `cuando`.

---

### 6. Formato de salida

Entrega una **tabla consolidada en formato compatible con CSV**, con exactamente estas columnas, en este orden:

- **Separador de columnas:** punto y coma `;`
- **NO usar comas** en ninguna celda de texto libre.
- Columnas en este orden exacto:

`dimension, categoria, descripcion, ejemplos, recomendacion, recuento_llamadas, porcentaje_llamadas, porc_ventas_categoria, porc_no_ventas_categoria, conversion_categoria`

Reglas finales:

- Todo el análisis debe estar **enfocado solo en S1**.  
- Utiliza **solo TRANSCRIPCIONES de llamadas con conversación válida** (según la definición de la sección 0).  
- Cada vez que calcules un porcentaje, asegúrate de que su **denominador esté bien definido y sea consistente** dentro de la dimensión correspondiente.
