# ML y modelos abiertos

Modelos clásicos (tabulares, series, clasificación) y modelos abiertos que se sirven o se ajustan en infraestructura propia. Cada apartado da el valor por defecto, la señal medida para cambiarlo, cómo verificar y los riesgos que señalar siempre.

## Índice

1. Línea base trivial antes que el modelo
2. Fugas de datos y validación
3. Métrica, segmentos y varianza
4. Desplegar y vigilar (MLOps)
5. Servir modelos abiertos
6. Cuantizar con evaluación antes y después
7. Ajuste fino

Cifras y nombres de motores de este fichero: fuentes recogidas el 25/10/2026; comprobar en el momento.

---

## 1. Línea base trivial antes que el modelo

**Por defecto:** antes de valorar un modelo, calcula sobre el mismo conjunto de prueba y con la misma métrica:
1. Una línea base trivial: la clase mayoritaria, la media, el valor de ayer, la tasa base.
2. Una regla de negocio sencilla o una sola variable (la más obvia).
3. El modelo.

Pon las tres cifras juntas en el informe. En datos tabulares, la referencia fuerte es gradient boosting; las redes, solo con justificación.

Por qué: una cifra sola no dice nada. Un AUC de 0,75 es poco si una sola variable da 0,73.

**Cambiar si:** no aplica; el modelo tiene que superar las líneas base con claridad para justificar su coste.

**Verificar:** las tres cifras salen del mismo código de evaluación y del mismo conjunto.

**Señalar siempre:** si el modelo apenas supera la regla sencilla, recomendar la regla.

## 2. Fugas de datos y validación

**Por defecto:** un resultado sospechosamente bueno es una fuga hasta que se demuestre lo contrario. Revisa:
1. Que cada variable esté disponible en el momento de predecir (estados finales, fechas de cierre, campos que se rellenan después del hecho).
2. La partición: temporal si se predice el futuro; por grupo si hay varias filas por cliente.
3. Que el preprocesado (escalado, imputación, codificación) se ajuste solo con entrenamiento, dentro de un pipeline.
4. Que el conjunto de prueba quede intacto hasta el final.

**Cambiar si:** no aplica.

**Verificar:** quitar la variable sospechosa y reentrenar; si la métrica cae mucho, era fuga. Cita la línea donde entra (`fichero:línea`).

**Señalar siempre:** desaconsejar desplegar mientras la fuga no esté descartada.

## 3. Métrica, segmentos y varianza

**Por defecto:**
1. Métrica elegida según el problema (desequilibrio de clases, necesidad de probabilidades calibradas) y umbral de decisión según el coste de cada error.
2. Resultado por segmentos relevantes antes de dar el modelo por bueno.
3. Varianza: validación cruzada o varias semillas, y margen de error según el tamaño del conjunto de prueba.

**Cambiar si:** el conjunto de prueba es muy pequeño; dilo y da el intervalo, no solo la cifra.

**Verificar:** la tabla por segmentos y la dispersión entre ejecuciones están en el informe.

**Señalar siempre:** segmentos donde el modelo es peor que la línea base.

## 4. Desplegar y vigilar (MLOps)

**Por defecto:** infraestructura proporcional: entrenamiento manual o un pipeline sencillo (niveles 0 o 1) salvo que haya muchos modelos. Antes de desplegar, repasa como lista:
1. Mismo código para calcular variables al entrenar y al servir, o prueba de equivalencia.
2. Validación del modelo antes de servirlo.
3. Despliegue canario y vuelta atrás preparada.
4. Monitorización por capas: operativa, predicciones, calidad cuando llegan etiquetas.

**Cambiar si:** hay muchos modelos o reentrenamientos frecuentes medidos; entonces más automatización.

**Verificar:** diagnosticar antes de reentrenar; muchas derivas son fallos de datos.

**Señalar siempre:** bucles de realimentación cuando las predicciones influyen en los datos futuros (precios, fraude, recomendaciones).

## 5. Servir modelos abiertos

**Por defecto:**
1. Licencia y política de uso comprobadas para el modelo concreto, el sector y el uso previsto, antes de cualquier prueba técnica.
2. Pesos en safetensors, con revisión y hash fijados; nada de ejecutar código remoto del repositorio del modelo sin revisarlo.
3. Motor: llama.cpp u Ollama para desarrollar; vLLM para servir con concurrencia (comprobar en el momento).
4. Calcula la memoria antes de elegir modelo o GPU: memoria ≈ pesos + caché KV + margen del motor.
   - Pesos = parámetros × bytes por parámetro (FP16: 2; FP8 o INT8: 1; INT4: 0,5). Un modelo de 8.000 millones en FP16 ocupa unos 16 GB.
   - Caché KV = 2 × capas × cabezas KV × dimensión de cabeza × bytes × tokens de contexto × peticiones simultáneas. Los valores están en el `config.json` del modelo.
5. Contexto máximo ajustado al uso real, no al máximo del modelo: el contexto y la concurrencia se pagan en memoria.
6. Todo fijado por versión: pesos, motor, cuantización y parámetros de arranque. Cambiar cualquiera es un cambio de modelo.
7. API compatible con el estándar de facto de chat para que la aplicación no dependa del motor.

**Cambiar si:** solo hay CPU (llama.cpp) o cargas con mucho prefijo común (otro motor, medido).

**Verificar:** prueba de carga con datos reales: tiempo hasta el primer token y latencia p95 con la concurrencia esperada.

**Señalar siempre:** que la memoria es una estimación hasta que se mide en la GPU real.

## 6. Cuantizar con evaluación antes y después

**Por defecto:** FP8 si el hardware lo admite. Si la memoria no alcanza, INT8 o INT4 (solo pesos).

Pasos, siempre que se cuantice:
1. Fija el conjunto de evaluación propio y el umbral de pérdida aceptable antes de cuantizar.
2. Evalúa el modelo sin cuantizar.
3. Evalúa el modelo cuantizado con el mismo conjunto, los mismos parámetros y varias ejecuciones.
4. Compara las dos cifras y decide con el umbral fijado.

Por qué: las pérdidas generales publicadas son pequeñas, pero en tareas concretas (programación, por ejemplo) pueden ser mayores; solo la evaluación propia dice si vale para este uso.

**Cambiar si:** la pérdida supera el umbral: subir de precisión, otro método de cuantización o un modelo más pequeño sin cuantizar.

**Verificar:** la tabla antes y después está en el informe.

**Señalar siempre:** cualquier cuantización desplegada sin esa comparación.

## 7. Ajuste fino

**Por defecto:** no ajustar. Primero prompt y RAG, con un conjunto de evaluación y una línea base con prompt. Nunca para meter conocimiento nuevo: eso es RAG o contexto.

**Cambiar si:** hay un fallo de formato o de estilo que el prompt no consigue, medido. Entonces:
1. Licencia y datos personales revisados antes de preparar datos.
2. Pocos ejemplos buenos, revisados a mano, con el formato exacto de producción; sintéticos solo mezclados con reales.
3. LoRA en todas las capas por defecto, con la tasa de aprendizaje ajustada para LoRA.
4. Base, datos, configuración y adaptador versionados juntos.

**Verificar:** el ajuste supera la línea base con prompt en la evaluación propia; se comprueba también el olvido y las alucinaciones, no solo la mejora en la tarea.

**Señalar siempre:** datos personales en los datos de ajuste: no se pueden borrar después del modelo.
