<!-- markdownlint-disable MD024 MD033 MD036 MD041 -->

<p align="center"><img src="./images/untref-logo.svg" alt="UNTreF" /></p>

# Trabajo Práctico Integrador: AskEDD

_Un chatbot que sabe de Estructuras de Datos_

- Modalidad: **Grupal (2 a 3 integrantes)**
- Lenguaje: **Python 3.10+**
- Entrega final: **Semana 14 del cuatrimestre**
- Presentación oral: **15 minutos por grupo, última semana**

## Introducción y motivación

A lo largo de la cursada estudiaremos estructuras de datos pensadas para almacenar, organizar y recuperar información de manera eficiente. Este trabajo práctico propone que apliquen esos conceptos construyendo algo concreto y útil: un chatbot capaz de responder preguntas sobre el propio apunte de la materia.

El proyecto se llama "AskEDD — Ask about Estructuras de Datos". La idea es simple: en lugar de buscar manualmente en el apunte, el usuario puede escribir una pregunta en lenguaje natural y el sistema devuelve el fragmento más relevante del texto junto con una respuesta estructurada.

> [!NOTE]
> **¿Por qué este TP?**
>
> Porque las estructuras que van a implementar (índice invertido, _heap_, grafo) no son ejercicios abstractos: son exactamente las que usan los motores de búsqueda, los asistentes de texto y los sistemas de recomendación reales. Van a entender desde adentro cómo funciona algo parecido a lo que usa Google o ChatGPT.
> El TP se organiza en tres etapas con entregas parciales. Cada etapa agrega una capa de funcionalidad al sistema, de modo que al final tendrán un programa completo e interactivo.

## Descripción general del sistema

AskEDD recibe como entrada un corpus de texto (los apuntes de la materia) y construye internamente una serie de estructuras de datos que permiten responder preguntas. El sistema expone una interfaz de chat en la terminal, donde el usuario escribe preguntas y recibe respuestas.

### Flujo general del sistema

```text
Corpus (texto plano)
        ↓
[Preprocesamiento]   →   tokenizar, normalizar, eliminar stopwords
        ↓
[Índice Invertido]   →   { término: [(doc_id, tf), ...] }
        ↓
[Motor de búsqueda]  →   recibe query → ranking por TF-IDF → top-K fragmentos
        ↓
[Interfaz de chat]   →   historial, templates de respuesta, sugerencias
```

El corpus que van a usar son los apuntes de la materia, publicados en: `https://untref-edd.github.io/apuntes/`

Cada página del apunte está disponible como archivo Markdown (`.md`) descargable. La primera tarea del proyecto es obtener ese corpus, parsearlo y definir qué es un "documento" para el sistema.

#### Definición de documento

En AskEDD, un documento es una sección del apunte delimitada por un título Markdown (líneas que empiezan con `#` o `##`). El texto bajo ese título (hasta el próximo título del mismo nivel o superior) forma el cuerpo del documento. Los tokens son las palabras de ese cuerpo, luego de la normalización.

## Etapas del proyecto

| Etapa | Nombre                          | Estructuras principales              | Puntos |
| ----: | :------------------------------ | :----------------------------------- | :----: |
|     1 | Chatbot ELIZA                   | lista, cola, diccionario de patrones | 30 pts |
|     2 | Corpus e Índice Invertido       | dict, list, set (posting lists)      | 30 pts |
|     3 | Búsqueda, Ranking e Integración | heap, TF-IDF, integración con chat   | 40 pts |

> [!NOTE]
> **Secuencia pedagógica**
>
> La Etapa 1 se entrega antes del primer parcial: trabaja con listas, diccionarios y control de flujo (estructuras que ya conocen). Las Etapas 2 y 3 se desarrollan en la segunda mitad de la cursada, después de estudiar índice invertido, TF-IDF y _heaps_ en clase.

## Etapa 1: Chatbot tipo ELIZA

Fecha de entrega: **semana 6 del cuatrimestre (antes del primer parcial)**

### ¿Qué hay que hacer?

En esta primera etapa construyen la interfaz conversacional del sistema: un chatbot en la terminal que responde preguntas sobre estructuras de datos usando patrones de texto predefinidos, sin todavía consultar el apunte. La inspiración es ELIZA, el chatbot de 1966 que simulaba conversación usando sustitución de patrones.

La idea es simple: el usuario escribe algo, el sistema detecta qué tipo de pregunta es (por las palabras clave que contiene) y responde con una frase predefinida. No hay búsqueda en ningún corpus todavía, todo el conocimiento está hardcodeado en los patrones.

> [!NOTE]
> **¿Por qué ELIZA primero?**
>
> Porque el _loop_ de chat, el historial y el sistema de patrones son la "carcasa" del sistema final. En las etapas siguientes van a reemplazar las respuestas hardcodeadas por resultados reales del índice invertido. Es más fácil construir las piezas de a una.

### Tareas

1. Implementar el _loop_ principal de chat: leer input del usuario, procesar, imprimir respuesta, repetir.
2. Definir un conjunto de patrones de reconocimiento. Cada patrón es una lista de palabras clave que, si aparecen en el input del usuario, activan una respuesta determinada. Deben cubrir al menos estas categorías:
   - Preguntas de definición: palabras como `"qué es"`, `"definí"`, `"explicá"`
   - Preguntas de comparación: palabras como `"diferencia"`, `"versus`", `"mejor"`
   - Preguntas de uso: palabras como `"cuándo usar"`, `"para qué sirve"`, `"ejemplo"`
   - Saludos y despedidas
   - Preguntas que el sistema no entiende (caso por defecto)
3. Implementar comandos especiales que el usuario puede escribir en cualquier momento:
   - `!ayuda` &mdash; muestra las categorías de preguntas que el sistema entiende
   - `!historial` &mdash; muestra las últimas N consultas
   - `!salir` &mdash; termina el programa
4. Mantener el historial de consultas usando una lista. Cada entrada del historial debe guardar el texto de la pregunta y la categoría detectada.
5. El sistema debe tener al menos 3 respuestas posibles por categoría y elegir una al azar, para que no siempre diga exactamente lo mismo.

### Ejemplo de sesión

```output
╔══════════════════════════════════╗
║         AskEDD  v0.1             ║
║  Tu asistente de EDD — UNTREF    ║
╚══════════════════════════════════╝

Escribí tu pregunta (o !ayuda para ver comandos)

> qué es un árbol binario

Un árbol binario es una estructura de datos donde cada nodo
tiene a lo sumo dos hijos. Es muy usado para búsqueda eficiente.
(Próximamente voy a poder buscarte esto en el apunte)

> diferencia entre lista y arreglo

Buena pregunta. Las listas y los arreglos difieren principalmente
en cómo usan la memoria y en la eficiencia de sus operaciones.
(Próximamente voy a poder buscarte esto en el apunte)

> cuándo usar un heap

Usá un heap cuando necesitás acceder siempre al mínimo o máximo
de un conjunto de forma eficiente, como en una cola de prioridad.

> !historial

-> Últimas consultas:
   1. [definición]   qué es un árbol binario
   2. [comparación]  diferencia entre lista y arreglo
   3. [uso]          cuándo usar un heap

> !salir
¡Hasta la próxima!
```

### Estructura de datos sugerida para los patrones

Una forma limpia de organizar los patrones es como una lista de diccionarios, donde cada diccionario representa una categoría:

```python
patrones = [
    {
        "categoria": "definicion",
        "palabras_clave": ["qué es", "definí", "explicá", "qué son"],
        "respuestas": [
            "Una {tema} es una estructura que...",
            "El concepto de {tema} se refiere a...",
            "{tema} es una de las estructuras más usadas porque...",
        ]
    },
    {
        "categoria": "comparacion",
        "palabras_clave": ["diferencia", "versus", "vs", "mejor", "comparar"],
        "respuestas": [ ... ]
    },
    ...
]
```

> [!TIP
> **Sobre la detección de patrones**
>
> No hace falta nada sofisticado: alcanza con recorrer la lista de patrones y verificar si alguna de las palabras clave aparece en el input del usuario (convertido a minúsculas). El primer patrón que matchee es el que se usa. Si ninguno matchea, se activa el caso por defecto.

### Preguntas para la entrega

- ¿Qué complejidad tiene la detección de patrones en función de la cantidad de patrones y palabras clave?
- ¿Qué limitaciones tiene este enfoque? ¿Qué tipos de preguntas no puede responder bien?
- ¿Qué estructura de datos sería más eficiente para el matching si hubiera miles de patrones?

## Etapa 2: Corpus e Índice Invertido

Fecha de entrega: **semana 10 del cuatrimestre**

### ¿Qué hay que hacer?

En esta etapa construyen el motor de búsqueda real: descargan el corpus, lo parsean y construyen el índice invertido. Esta estructura permite saber, dado un término, en qué documentos del apunte aparece y con qué frecuencia.

#### Paso 0: Obtener el corpus

El código fuente del apunte está disponible en GitHub como repositorio público. Los archivos `.md` están en la carpeta `contenidos/` y pueden descargarse directamente desde ahí: `https://github.com/untref-edd/apuntes`

```shell
# Opción A: clonar el repositorio completo
git clone https://github.com/untref-edd/apuntes.git
# Los .md están en apuntes/contenidos/

# Opción B: descargar el ZIP desde GitHub
# Code → Download ZIP → descomprimir → usar la carpeta contenidos/
```

> [!TIP]
> **Una vez descargados**
>
> Copien los archivos `.md` que quieran usar a la carpeta `corpus/` de su proyecto. No es necesario usar todos, pueden empezar con un subconjunto para probar y agregar más después.

### Paso 1: Parsear los documentos

Un documento en AskEDD es una sección del apunte delimitada por un título Markdown. La regla: cada línea que empiece con `#` o `##` marca el inicio de un nuevo documento. El cuerpo del documento son todas las palabras entre ese título y el próximo.

Ejemplo: archivo `arboles.md`

```md
## Árboles binarios

Un árbol binario es una estructura donde cada nodo tiene como máximo dos hijos: el hijo izquierdo y el hijo derecho.

## Recorridos

Los recorridos de un árbol pueden ser inorden, preorden o postorden según el orden en que se visitan los nodos.
```

Resultado del parseo:

```python
doc_0 = { "titulo": "Árboles binarios",
          "texto":  "Un árbol binario es una estructura...",
          "archivo": "arboles.md" }
doc_1 = { "titulo": "Recorridos",
          "texto":  "Los recorridos de un árbol...",
          "archivo": "arboles.md" }
```

### Paso 2: Tokenizar y normalizar

1. Convertir todo a minúsculas.
2. Eliminar signos de puntuación.
3. Separar por espacios.
4. Eliminar stopwords (lista provista por la cátedra).

```python
texto = "Un árbol binario es una estructura donde cada nodo tiene"
```

Después de normalizar y eliminar stopwords:

```python
["árbol", "binario", "estructura", "nodo", "tiene"]
```

### Paso 3: Construir el índice invertido

```python
indice = {
    "arbol": [(0, 3), (4, 1), (7, 2)],
    "recorrido": [(1, 5), (4, 2)],
    "hash": [(2, 1), (3, 4), (8, 1)],
    ...
}

# (doc_id, term_frequency)
```

### Paso 4: Búsqueda booleana básica

1. `busqueda_AND(terminos)` &mdash; documentos que contienen TODOS los términos.
2. `busqueda_OR(terminos)` &mdash; documentos que contienen AL MENOS UNO.

> [!TIP]
> **Pista para la búsqueda `AND`**
>
> Conviertan la posting list de cada término a un set de `doc_ids` y usen el operador `&` de Python. Es una sola línea de código, pero entiendan por qué funciona.

### Preguntas para la entrega

- ¿Cuántos documentos tiene el corpus? ¿Cuántos términos únicos tiene el índice?
- ¿Qué complejidad temporal tiene la construcción del índice?
- ¿Qué ventaja tiene guardar las posting lists ordenadas por doc_id?
- ¿Hay títulos que generan documentos muy cortos? ¿Cómo lo manejan?

## Etapa 3: Búsqueda, Ranking e Integración

Fecha de entrega: **semana 14 del cuatrimestre (entrega final)**

### ¿Qué hay que hacer?

La etapa final tiene dos partes: implementar el ranking de resultados con TF-IDF y _heap_, e integrar el motor de búsqueda al chatbot de la Etapa 1. El resultado es AskEDD completo: el chatbot ya no responde con frases hardcodeadas sino con fragmentos reales del apunte.

### Parte A: Ranking con TF-IDF

> [!IMPORTANT]
> **Esto hay que investigarlo**
>
> TF-IDF es un concepto que no vamos a ver en clase antes de esta entrega. Parte del trabajo es que lo investiguen ustedes: busquen qué significa, para qué sirve y cómo se calcula. La fórmula de abajo es una referencia mínima para que tengan un punto de partida, pero se espera que en la presentación puedan explicarlo con sus propias palabras.

TF-IDF (_Term Frequency–Inverse Document Frequency_) es una métrica que mide qué tan relevante es un término para un documento dentro de un corpus. La idea intuitiva: un término es relevante si aparece mucho en un documento específico (TF alto) pero poco en el resto del corpus (IDF alto).

Para un término `t` en un documento `d` del corpus `C`:

```text
TF(t, d)  = frecuencia del término t en el documento d
            (ya lo tienen del índice de la Etapa 2)

IDF(t, C) = log( N / df(t) )
            N     = cantidad total de documentos
            df(t) = cantidad de documentos que contienen t

TF-IDF(t, d) = TF(t, d) × IDF(t, C)

Score(query, d) = suma de TF-IDF(t, d) para cada t en query
```

1. Calcular el IDF de cada término una sola vez al cargar el índice y guardarlo en un diccionario.
2. Implementar `score(query, doc_id)` que devuelva el puntaje TF-IDF de un documento para una consulta.
3. Implementar `top_k(query, k=3)` usando `heapq` para devolver los `k` documentos más relevantes.

> [!IMPORTANT]
> **¿Por qué un _heap_ y no `sorted()`?**
>
> Si el corpus tuviera millones de documentos, ordenar toda la lista sería $O(N log N)$. Con un heap de tamaño `k` obtienen el `top-k` en $O(N log k)$. En este TP el corpus es chico, pero el razonamiento sobre eficiencia es parte de la evaluación.

### Parte B: Integración con el chatbot

Ahora conectan el motor de búsqueda con la interfaz conversacional de la Etapa 1. El cambio principal es en cómo se generan las respuestas: en lugar de usar una frase del diccionario de patrones, el sistema busca en el índice y devuelve el fragmento más relevante del apunte.

Antes (Etapa 1): respuesta hardcodeada

```output
> qué es un árbol binario
"Un árbol binario es una estructura de datos donde cada nodo..."
  ↑ texto fijo, siempre el mismo
```

Ahora (Etapa 3): respuesta desde el apunte

```output
> qué es un árbol binario
📚 Del apunte (score: 3.84) — Árboles binarios [arboles.md]:
   "Un árbol binario de búsqueda es una estructura donde...
    cada nodo tiene a lo sumo dos hijos, y para todo nodo n,
    los valores del sub00e1rbol izquierdo son menores que n...
```

1. Modificar el _loop_ de chat para que, al detectar una categoría (definición, comparación, uso), llame a `top_k()` con las palabras clave extraídas del input.
2. Si `top_k()` devuelve resultados, mostrar el título del documento, el archivo de origen y un fragmento del texto.
3. Si no hay resultados relevantes (todos los scores son 0), responder con el template genérico de la Etapa 1.
4. Los comandos `!historial`, `!ayuda` y `!salir` deben seguir funcionando igual que antes.

### Extensión opcional: grafo de co-ocurrencia

Como bonus, pueden construir un grafo donde los nodos son términos y existe una arista entre dos términos si co-ocurren en el mismo documento. Al mostrar un resultado, el sistema puede sugerir términos relacionados.

```output
> qué es un árbol binario

📚 Del apunte — Árboles binarios:
   Un árbol binario es una estructura donde cada nodo tiene...

🔗 Temas relacionados: recorrido, altura, BST, heap
```

### Ejemplo de sesión completa (sistema integrado)

```output
╔══════════════════════════════════╗
║         AskEDD  v1.0             ║
║  Tu asistente de EDD — UNTREF    ║
╚══════════════════════════════════╝

Escribí tu pregunta (o !ayuda para ver comandos)

> qué es un índice invertido

📚 Del apunte (score: 4.72) — Índice invertido [indice-invertido.md]:
   "Un índice invertido es una estructura que mapea términos a los
    documentos en los que aparecen, junto con la frecuencia de
    aparición. Es la base de los motores de búsqueda modernos."

> diferencia entre lista y arreglo

📚 Del apunte (score: 3.10) — Listas enlazadas [listas.md]:
   "A diferencia de los arreglos, las listas enlazadas no requieren
    memoria contigua y permiten inserciones en O(1)..."

> !historial

📋 Últimas consultas:
   1. [definición]  qué es un índice invertido
   2. [comparación] diferencia entre lista y arreglo

> !salir
¡Hasta la próxima!
```

### Preguntas para la entrega

- ¿Por qué el IDF penaliza los términos que aparecen en muchos documentos?
- ¿Qué complejidad tiene `top_k` con `heapq` versus con `sorted()`?
- ¿Cómo cambió la calidad de las respuestas respecto a la Etapa 1? ¿En qué casos el sistema hardcodeado era mejor?

## Estructura del proyecto

El repositorio del proyecto debe tener la siguiente estructura mínima de archivos:

```output
askedd/
├── corpus/
│   ├── introduccion.md      ← archivos descargados del apunte (Etapa 2)
│   ├── arboles.md
│   ├── indice-invertido.md
│   └── ...                  ← un .md por página del apunte
├── src/
│   ├── chat.py              ← loop de chat, historial, patrones ELIZA (Etapa 1)
│   ├── corpus.py            ← descarga y parseo de .md → lista de documentos (Etapa 2)
│   ├── preprocesamiento.py  ← tokenización, normalización, stopwords (Etapa 2)
│   ├── indice.py            ← construcción del índice invertido (Etapa 2)
│   ├── buscador.py          ← TF-IDF, ranking, top-k con heap (Etapa 3)
│   └── main.py              ← punto de entrada
├── tests/
│   └── test_indice.py       ← tests unitarios (mínimo 5 tests)
└── README.md                ← instrucciones de uso y cómo obtener el corpus
```

> [!IMPORTANT]
> **Importante**
>
> Cada archivo debe tener una responsabilidad clara. No pongan todo en un único script. Parte de la evaluación es la organización del código.
>
> Se sugiere que el sistema esté diseñado siguiendo los principios de la **Programación Orientada a Objetos (POO)**. Cada componente mencionado (Buscador, Índice, ChatBot) puede estar representado por clases con responsabilidades bien definidas, así se evita usar funciones sueltas y/o variables globales.

## Criterios de evaluación

| Criterio                    | Insuficiente (0–4)                 | Regular (5–6)                            | Bien (7–8)                             | Excelente (9–10)                                |
| :-------------------------- | :--------------------------------- | :--------------------------------------- | :------------------------------------- | :---------------------------------------------- |
| **Chatbot ELIZA (E1)**      | No funciona o no reconoce patrones | Reconoce algunos patrones, sin historial | Patrones variados, historial funcional | Múltiples categorías, comandos, bien organizado |
| **Índice invertido (E2)**   | No construido o con errores graves | Construido pero incompleto               | Funcional, tokenización razonable      | Correcto, con normalización y stopwords         |
| **Búsqueda y ranking (E3)** | No implementado                    | Búsqueda básica sin ranking              | Ranking con TF-IDF, heap correcto      | TF-IDF bien explicado, integrado al chat        |
| **Calidad del código**      | Difícil de leer, sin estructura    | Parcialmente organizado                  | Funciones bien separadas               | Claro, documentado, con tests                   |
| **Presentación oral**       | Sin explicación de decisiones      | Explica qué hace                         | Explica cómo y por qué                 | Conecta con la teoría de la materia             |

La nota final del TP es el promedio ponderado de las tres etapas más la presentación oral. La Etapa 1 recibe feedback y puede reentregarse una vez antes del primer parcial. Las Etapas 2 y 3 reciben feedback y pueden reentregarse una vez antes de la entrega final.

## Restricciones técnicas

> [!CAUTION]
> **Librerías no permitidas**
>
> No pueden usar librerías de NLP como `spaCy`, `NLTK`, `scikit-learn`, ni ninguna que construya el índice o calcule TF-IDF automáticamente. El objetivo es que implementen las estructuras ustedes mismos.

Librerías permitidas:

- Módulos de la biblioteca estándar de Python (`math`, `re`, `json`, `collections`, `heapq`, `os`, etc.)
- `rich` o `colorama` para darle formato a la terminal (opcional, para embellecer la interfaz)
- `pytest` o `unittest` para los tests unitarios, manteniendo consistencia con el uso de una sola librería
- Cualquier otra librería debe ser consultada con la cátedra antes de usarla

## Formato de entrega

### Repositorio en GitHub

El proyecto debe entregarse como repositorio público en GitHub. El link, al commit entregado, se sube en Google Classroom antes de cada fecha de entrega. Cada commit debe tener un mensaje descriptivo, los commits vacíos o con mensajes genéricos como 'asdf' o 'cambios' se consideran mala práctica y se reflejan en la evaluación.

### Informe escrito (entrega final)

Junto con el código deben entregar un informe en formato Markdown que incluya:

1. Descripción de las estructuras de datos elegidas y por qué.
2. Análisis de complejidad de las operaciones principales.
3. Dificultades encontradas y cómo las resolvieron.
4. Reflexión: ¿qué mejorarían si tuvieran más tiempo?

### Presentación oral

La presentación dura 15 minutos por grupo y consiste en:

- Demo en vivo del chatbot (5 minutos)
- Explicación de las estructuras implementadas (5 minutos)
- Preguntas de la cátedra (5 minutos)

> [!WARNING]
> **Sobre las preguntas orales**
>
> Durante la presentación, la cátedra puede preguntarle a cualquier integrante del grupo sobre cualquier parte del código. Todos deben poder explicar el proyecto completo. No alcanza con 'esa parte la hizo mi compañero/a'.

## Preguntas frecuentes

**¿Qué pasa si dos títulos tienen exactamente el mismo texto?**

Es posible que el apunte tenga títulos repetidos en distintas páginas (por ejemplo, dos secciones llamadas 'Introducción'). En ese caso traten cada instancia como un documento separado: el `doc_id` los diferencia. Pueden incluir el nombre del archivo `.md` como metadato del documento para distinguirlos en la respuesta.

**¿Tienen que descargar los `.md` manualmente o con código?**

Pueden hacerlo de cualquiera de las dos formas. Lo que sí es obligatorio es que el proyecto incluya, como mínimo, la carpeta `corpus/` con todos los archivos descargados, y un archivo `README.md` que explique cómo obtenerlos. Si hacen un script de descarga, súbanlo también al repositorio.

**¿El chatbot tiene que usar IA generativa (como GPT)?**

No. El sistema que van a construir es un motor de recuperación de información clásico, no un modelo de lenguaje generativo. No usen APIs de OpenAI, Anthropic ni similares. El objetivo es entender las estructuras de datos subyacentes.

**¿Pueden trabajar de a dos en lugar de tres?**

Sí. Los grupos pueden ser de 2 o 3 personas. Los grupos de 2 tienen las mismas exigencias que los de 3; no se reduce la funcionalidad requerida.

**¿Qué pasa si el corpus tiene palabras en inglés?**

El apunte mezcla términos en español e inglés (como _heap_, _hash_, _merge_). Traten ambos idiomas por igual: si el usuario busca _heap_, debe encontrar fragmentos que contengan _heap_. No necesitan traducción automática.

**¿Hay que persistir el índice en disco?**

No es obligatorio, pero si el tiempo de construcción del índice supera los 10 segundos (improbable con este corpus), es recomendable guardarlo en un archivo JSON para no reconstruirlo cada vez. Pueden hacerlo como mejora opcional.

## Recursos y referencias

Los siguientes capítulos del apunte son especialmente relevantes para este TP:

- Recuperación de la Información &mdash; fundamentos del índice invertido
- Representación y Adquisición de Datos &mdash; cómo estructurar el corpus
- Árboles y heaps &mdash; para el ranking con heapq
- Grafos &mdash; para la extensión opcional de co-ocurrencia

Material complementario recomendado:

- Introduction to Information Retrieval &mdash; Manning, Raghavan, Schütze (disponible online, CC)
- Documentación oficial de Python: módulos heapq, collections.Counter, re
- Wikipedia: TF-IDF, Inverted Index, ELIZA (el chatbot original de los años 60)
