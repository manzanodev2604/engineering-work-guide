# Sistemas con LLM

Chatbots, RAG, extracción, clasificación, agentes y cualquier sistema que llame a un modelo de lenguaje. Cada apartado da el valor por defecto, la señal medida para cambiarlo, cómo verificar y los riesgos que señalar siempre.

## Índice

1. Escalera de enfoques
2. ¿Cabe en el contexto? Estimarlo antes de elegir RAG
3. Evaluación: preguntas con respuesta esperada
4. RAG: troceado y recuperación
5. Salidas estructuradas
6. Agentes y herramientas
7. Elegir modelo
8. Coste y latencia
9. Fiabilidad en producción
10. Seguridad
11. Datos personales
12. Prompts versionados y regresiones

Cifras de este fichero: fuentes de 2024 a 2026 recogidas el 25/10/2026. Precios, ventanas de contexto y nombres de modelos: comprobar en el momento.

---

## 1. Escalera de enfoques

**Por defecto:** el peldaño más bajo que cumpla el objetivo medido:
1. Sin modelo, si reglas, SQL o una búsqueda lo resuelven.
2. Una llamada con buen prompt y ejemplos.
3. Más contexto recuperado (RAG), si el fallo es de conocimiento.
4. Flujo de varios pasos fijos, si la tarea se descompone de forma predecible.
5. Ajuste fino, si el fallo es de comportamiento constante o hace falta abaratar con un modelo pequeño, y hay 50 o más ejemplos buenos.
6. Agente, solo si el número de pasos no se puede prever; con límites, puntos de control y entorno aislado.

Antes de elegir, fija el umbral de "suficiente" con una cuenta del coste de los errores.

**Cambiar si:** la evaluación del peldaño actual muestra un fallo que el siguiente resuelve. Cada subida se justifica con esa medida.

**Verificar:** el informe dice en qué peldaño se queda y qué medida descartó el anterior.

**Señalar siempre:** cualquier framework propuesto, con lo que resuelve que la API directa no.

## 2. ¿Cabe en el contexto? Estimarlo antes de elegir RAG

**Por defecto:** antes de proponer RAG, haz la cuenta:
1. Tokens del material ≈ páginas × tokens por página. Para texto corrido, unos 400 tokens por página (supuesto; mídelo con el tokenizador del modelo sobre una muestra).
2. Compáralo con la ventana del modelo menos el espacio para instrucciones, pregunta y respuesta.
3. Si cabe con margen, todo en el contexto con caché de prompts, sin RAG.

Referencia fechada (Anthropic, 2024; comprobar en el momento): por debajo de unos 200.000 tokens (unas 500 páginas), meter todo en el prompt con caché; con caché, latencia más de dos veces menor y hasta un 90 % menos de coste.

Ejemplo: 200 páginas × 400 ≈ 80.000 tokens. Cabe en una ventana de 200.000 con margen; RAG no hace falta para empezar.

Por qué: RAG añade troceado, índice, permisos por fragmento y una fuente nueva de errores; si no hace falta, sobra.

**Cambiar si:** no cabe, el material crece por encima de la ventana, el coste por consulta con todo el contexto supera el presupuesto, o la evaluación muestra que el contexto largo degrada la precisión.

**Verificar:** la cuenta queda escrita en el informe, con el tamaño medido o supuesto.

**Señalar siempre:** si faltan documentos o no se sabe el volumen real, la cuenta es provisional.

## 3. Evaluación: preguntas con respuesta esperada

**Por defecto:** ningún sistema con LLM se da por terminado sin un conjunto de evaluación y su resultado a la vista, antes y después de cada cambio. Las pruebas unitarias del código no lo sustituyen.

Cómo montarlo:
1. Escribe preguntas reales con su respuesta esperada y, en RAG, el documento y la página donde está. Empieza con 20 a 50; ve hacia 100 o más con casos reales.
2. Cubre dimensiones, no solo casos fáciles: tipo de pregunta, dificultad, preguntas sin respuesta en el material, preguntas de alguien sin permiso para ese documento, intentos de inyección.
3. No pidas al modelo "dame preguntas de prueba": define las dimensiones, escribe unas 20 combinaciones a mano y genera más solo a partir de ellas.
4. Criterios binarios y específicos por fallo ("cita la fuente correcta", "no inventa una cifra"), no notas de 1 a 5.
5. Primero comprobaciones por código (contiene, esquema, cifra exacta); juez LLM solo donde no basten, y validado con una muestra revisada a mano.
6. Antes de proponer mejoras, lee trazas y clasifica los fallos con recuento.

Por qué: sin preguntas con respuesta esperada no se puede saber si un cambio mejora o empeora el sistema.

**Cambiar si:** no aplica; lo que cambia es el tamaño del conjunto, que crece con cada fallo de producción.

**Verificar:** tasa de aprobados antes y después, sobre el mismo conjunto, varias ejecuciones en los casos críticos. Un 100 % suele indicar que el conjunto no aprieta.

**Señalar siempre:** quién debe validar las respuestas esperadas (alguien que conozca el material) y cuántas están validadas.

## 4. RAG: troceado y recuperación

**Por defecto:**
1. Troceado recursivo o por estructura, unos 200 a 400 tokens, sin solape (estudio de Chroma, julio de 2024; comprobar en el momento).
2. Línea base con BM25 antes de montar embeddings.
3. Búsqueda híbrida BM25 y vectores con fusión por rango; reordenado solo si la medida lo justifica.
4. Metadatos y permisos desde el principio, aplicados dentro de la consulta de recuperación, no después.
5. Un solo almacén mientras se pueda (extensión vectorial de la base de datos que ya existe).
6. El modelo de embeddings es una decisión de un solo sentido: ADR y coste de reindexar previsto.

**Cambiar si:** el recall medido no llega al umbral; cada cambio (troceado, embeddings, híbrido, reordenado, top-k) se acepta solo si mejora la métrica sin empeorar la latencia más de lo aceptable.

**Verificar:**
1. Conjunto de recuperación etiquetado y recall@k medido antes de tocar el prompt de generación.
2. Orden de depuración: recuperación, después fidelidad, después relevancia.
3. Fragmentos reales inspeccionados antes de dar el índice por bueno.

**Señalar siempre:** respuestas con consecuencias (legales, de salud, de dinero) sin cita a la fuente que el usuario pueda comprobar.

## 5. Salidas estructuradas

**Por defecto:** toda salida que consume código va con esquema, decodificación restringida si el proveedor la ofrece y validación propia. Una salida que no valida es un error explícito: reintento limitado y, si falla, registro y camino alternativo. Nunca un valor por defecto silencioso. Prevé el "no está" con nulos o "desconocido".

**Cambiar si:** la tarea es compleja y la evaluación muestra errores; razona antes de estructurar (campo previo o dos pasos).

**Verificar:** evalúa el contenido, no solo el formato: un JSON válido puede ser incorrecto.

**Señalar siempre:** campos que el sistema rellena aunque el dato no exista en la entrada.

## 6. Agentes y herramientas

**Por defecto:** flujo fijo. Agente único solo si los pasos no se pueden prever. Varios agentes solo para lectura o investigación en paralelo, nunca escribiendo en paralelo sobre lo mismo, y con su coste calculado antes.

Todo agente lleva:
1. Límites de pasos, coste y tiempo.
2. Pocas herramientas, completas y bien descritas.
3. Confirmación humana para acciones irreversibles o externas.
4. Primero un bucle simple con la API, sin framework.

**Cambiar si:** la evaluación con tareas reales muestra que el peldaño actual no llega.

**Verificar:** llamadas a herramientas, tokens y errores por tarea, sobre tareas reales.

**Señalar siempre:** la tríada letal (apartado 10) si el agente tiene herramientas.

## 7. Elegir modelo

**Por defecto:** el modelo más barato que supere el umbral de calidad fijado antes, con versión fijada (nunca un alias que cambia solo) y detrás de una interfaz propia para poder cambiarlo.

Método:
1. Conjunto de evaluación propio.
2. Un candidato por gama (pequeño, medio, grande).
3. Cada caso repetido varias veces; una pasada no basta.
4. Tabla con calidad, latencia p95 y coste por tarea (no por token).
5. Elegir y dejarlo en un ADR.

**Cambiar si:** cambia la evaluación o el coste; antes de cambiar de modelo para arreglar un problema, comprueba con el análisis de errores que el modelo es la causa. Ajustar el esfuerzo de razonamiento suele ser mejor palanca que cambiar de modelo.

**Verificar:** la evaluación se repite periódicamente; los proveedores actualizan modelos.

**Señalar siempre:** condiciones del proveedor (retención, uso para entrenamiento, región) cuando hay datos personales o confidenciales, antes de cualquier prueba técnica.

## 8. Coste y latencia

**Por defecto:** coste mensual estimado en toda propuesta: peticiones × tokens × precio, con caché y lotes si aplican (`assets/estimacion.md`). Límites de gasto por tarea, usuario y día, con alerta. Caché de prompts por diseño (parte estática primero). Lotes para todo lo no interactivo, incluidas las evaluaciones.

**Cambiar si:** el coste por interacción supera el objetivo; entonces es un problema de coste, no de capacidad.

**Verificar:** cada optimización se comprueba con la evaluación de calidad; ahorrar con respuestas peores no es ahorrar.

**Señalar siempre:** bucles sin tope de gasto y caché semántica sin validar falsos positivos ni aislar por cliente.

## 9. Fiabilidad en producción

**Por defecto:**
1. Timeout en toda llamada al modelo; en respuestas largas, streaming y timeout hasta el primer token.
2. Reintentos solo de errores transitorios, con espera exponencial y aleatoria, respetando la cabecera de espera, en una sola capa (el SDK puede reintentar ya) y con tope.
3. Un error de tope de gasto no se reintenta: se alerta.
4. Degradación decidida de antemano y documentada; nunca respuestas inventadas.
5. Herramientas que escriben, idempotentes o con comprobación previa.
6. Registrar el identificador de petición, los intentos y las alternativas usadas.

**Cambiar si:** se añade un proveedor alternativo; solo si está evaluado con el mismo conjunto y aprobado para esos datos.

**Verificar:** pruebas con fallos simulados en la frontera del proveedor.

**Señalar siempre:** reintentos en varias capas a la vez.

## 10. Seguridad

**Por defecto:**
1. Revisa la tríada letal: datos privados, contenido no fiable y un canal para sacar datos. Si están las tres, quita una.
2. Permisos, autorización y validación en código, no en el prompt.
3. La salida del modelo es entrada no fiable: SQL parametrizado, HTML escapado, nunca ejecutada directamente.
4. Herramientas y permisos mínimos, con las credenciales del usuario.
5. Ningún secreto en los prompts.
6. Dependencias sugeridas por el modelo, verificadas antes de instalarlas.

**Cambiar si:** no aplica.

**Verificar:** casos de inyección directa e indirecta dentro del conjunto de evaluación.

**Señalar siempre:** documentos que el usuario no debería ver y que el sistema puede recuperar.

## 11. Datos personales

**Por defecto:**
1. Pregunta qué datos personales toca el sistema y si hay categorías especiales (salud, menores). Si las hay, avisa de que hay que implicar a quien lleve la protección de datos.
2. Sin proveedor revisado (contrato, retención, entrenamiento, región), no se envían datos personales.
3. Minimiza o seudonimiza antes de llamar al modelo.
4. Retención definida para trazas, memoria, caché e índices.
5. Borrado completo diseñado desde el principio: índice reconstruible, sin datos personales en modelos ajustados.

**Cambiar si:** no aplica.

**Verificar:** busca datos personales en una muestra de documentos y trazas antes de indexar.

**Señalar siempre:** los datos personales encontrados, con ejemplos del tipo (no del valor), y la posible obligación normativa, sin dar dictamen jurídico.

## 12. Prompts versionados y regresiones

**Por defecto:** prompts, esquemas, descripciones de herramientas y configuración del modelo versionados como código. Versión del prompt y del modelo registrada en cada traza desde la primera versión.

**Cambiar si:** no aplica.

**Verificar:** conjunto de regresión antes del primer cambio, alimentado con incidentes reales; puerta de CI por calidad, coste y latencia que también salta al cambiar de modelo.

**Señalar siempre:** cambios de prompt o de modelo sin vuelta atrás preparada.
