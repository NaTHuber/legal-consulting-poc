# Asesor Legal para Consulta de Historia de Demandas

Este documento presenta el avance de una prueba de concepto para modernizar el proceso de consulta de historias de demandas mediante Inteligencia Artificial Generativa.

**Estado actual:** Completada la fase de análisis, diseño de arquitectura, preparación de datos y generación de embeddings. 

---

## 1. Explicación del caso


**Nombre del proyecto:** _Asesor legal para consultar historia de demandas_

**Descripción:** El consultorio legal actualmente asesora a sus clientes revisando un archivo de Excel que contiene la historia de demandas y sus sentencias. Este proceso es manual y consume tiempo, ya que los abogados deben buscar fila por fila la información relevante para responder preguntas de los clientes.

La empresa busca modernizar este proceso mediante una prueba de concepto (PoC) que utilice Inteligencia Artificial Generativa para responder preguntas de forma automática, en un lenguaje claro y comprensible para personas sin conocimientos de derecho.

Para esta prueba se decidió concentrar el análisis en demandas relacionadas con redes sociales, y se definieron varias preguntas de ejemplo a resolver.


**Objetivos:** 

- Ofrecer un primer nivel de asesoría automática, dejando a los abogados únicamente los casos más complejos.
- Mejorar la precisión de las respuestas entregadas a los clientes.

**Categorías:** Herramientas de consultoría legal, Optimización de procesos, IA generativa.  

**Motivación:** Poner en práctica mis conocimientos para una prueba técnica.

**Preguntas ejemplo:** 
- ¿Cuáles son las sentencias de 3 demandas?
- ¿De qué se trataron esas demandas?
- ¿Cuál fue la sentencia del caso sobre acoso escolar?
- ¿Cuál es el detalle de esa demanda?
- ¿Existen casos que hablen sobre el PIAR, y cuáles fueron sus sentencias?

**Dataset:** 
- Filas: **329**
- Columnas: `#`, `Relevancia`, `Providencia`, `Tipo (todo vacío)`, `Fecha Sentencia`, `Tema - subtema`, `resuelve`, `sintesis`.

---

## 2. Supuestos 

- Cada fila del Excel corresponde a **un caso** con campos: `Providencia`, `Tema - subtema`, `resuelve`, `sintesis`, `Fecha Sentencia`, etc.  
- El sistema **no reemplaza** al abogado; provee respuestas iniciales y referencias a los casos.  
- El alcance de la PoC se limita a **redes sociales** y a responder en **lenguaje coloquial**.  
- En ausencia de evidencia en los datos, el sistema debe **decir que no encuentra información** (sin inventar).

## 3. Formas para resolver el caso y la opción tomada en esta prueba

Se evaluaron tres enfoques técnicos diferentes para resolver el problema de búsqueda y recuperación de información:

### Comparativa de Enfoques

| Enfoque | Descripción | Ventajas | Desventajas | Complejidad |
|---------|-------------|----------|-------------|-------------|
| **Búsqueda por palabras clave** | Filtrado mediante `str.contains()` en columnas relevantes | • Implementación inmediata<br>• Sin dependencias externas<br>• Rápido | • No entiende sinónimos<br>• Sensible a errores tipográficos<br>• Búsquedas muy literales | Muy baja |
| **TF-IDF / BM25** | Recuperación lexical con ponderación de términos relevantes | • Mejor que búsqueda básica<br>• Robusto ante ruido<br>• Fácil de explicar | • No captura significado semántico<br>• Requiere vectorizador e índice | Baja |
| **RAG (Embeddings + Vector Store + LLM)** | Búsqueda semántica mediante vectores y generación de respuestas en lenguaje natural | • Encuentra casos aunque varíe la redacción<br>• Respuestas en lenguaje natural<br>• Comprensión semántica profunda | • Requiere modelo de embeddings<br>• Necesita vector store<br>• Mayor complejidad técnica | Media |

### Decisión Técnica

**Se seleccionó el enfoque RAG** por las siguientes razones:

1. **Búsqueda semántica:** Permite encontrar casos relevantes aunque el usuario utilice palabras diferentes (ej. "bullying" vs "acoso escolar")
2. **Respuestas naturales:** Genera explicaciones en lenguaje coloquial, no solo referencias a números de caso
3. **Escalabilidad:** El sistema puede crecer a miles de casos sin degradación significativa de rendimiento
4. **Precisión contextual:** No inventa información; solo responde basándose en casos reales encontrados

---

### Flujo de Funcionamiento

```mermaid
flowchart TD
    subgraph FasePreparacion [FASE DE PREPARACIÓN]
        A[FASE DE PREPARACIÓN] --> B[Cargar Excel<br/>329 casos]
        B --> C[Limpieza y Preparación]
        C --> D[Crear campo 'documento']
        D --> E[Generar Embeddings]
        E --> F[Almacenar en ChromaDB]
    end

    subgraph FaseConsulta [FASE DE CONSULTA - cada pregunta del usuario ]
        G[FASE DE CONSULTA] --> H[Pregunta usuario]
        H --> I[Convertir a embedding]
        I --> J[Buscar casos similares<br/>en ChromaDB]
        J --> K[LLM genera respuesta<br/>con contexto]
        K --> L[Respuesta en<br/>lenguaje natural]
    end

    F --> G
```

### ¿Qué son los Embeddings?

Los embeddings son representaciones numéricas del texto que capturan su significado semántico. 

### Función de ChromaDB

ChromaDB actúa como una base de datos vectorial especializada que:
1. Almacena los embeddings de los 329 casos
2. Permite búsquedas por similitud matemática 
3. Recupera los casos más relevantes para una consulta en milisegundos

---

## 4. Preparación de Datos


El proceso de preparación siguió estos pasos:

1. **Carga y limpieza base**
   - Normalización de texto
   - Manejo de valores nulos (imputación con "No especificado")
   - Estandarización de formatos

2. **Creación de campos adicionales**
   - Banderas para identificar casos de redes sociales
   - Etiquetas para "acoso escolar" y "PIAR"
   - Campos auxiliares para filtrado

3. **Construcción del campo "documento"**
   - Combinación de columnas relevantes en un solo texto estructurado


4. **Filtrado para la PoC**
   - Selección de casos relacionados con redes sociales

5. **Generación de outputs**
   - `sentencias_limpio_full.csv`: Dataset completo limpio
   - `sentencias_rag_base.csv`: Subset optimizado para RAG

---

## 5. Generación de Embeddings

**Modelo utilizado:** `SentenceTransformer("all-MiniLM-L6-v2")`

**Características del modelo:**
- Optimizado para español e inglés
- Genera vectores de 384 dimensiones
- Balance óptimo entre calidad y velocidad
- Modelo ligero (23MB) apto para producción




## 6. Resultados

 **Completado:**
- Análisis del problema y definición de alcance
- Evaluación de alternativas técnicas
- Diseño de arquitectura RAG
- Limpieza y preparación de datos (329 casos)
- Generación de embeddings con SentenceTransformers

**Pendiente:**
- Indexación en ChromaDB
- Implementación de sistema de retrieval
- Integración con LLM para generación de respuestas
- Testing y validación


