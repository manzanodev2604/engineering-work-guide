# Verificación

Cómo comprobar que lo hecho funciona antes de decirlo. Cada apartado da el valor por defecto, la señal medida para cambiarlo, cómo verificar y los riesgos que señalar siempre.

## Índice

1. Pruebas desde la especificación, antes que el código
2. Oráculo independiente
3. No tocar las pruebas existentes
4. Prueba diferencial en migraciones y optimizaciones
5. Romper el código a propósito
6. Probar el artefacto como se ejecuta de verdad
7. Modelo como juez: solo para lo que no se ejecuta
8. Qué decir que no se comprobó

---

## 1. Pruebas desde la especificación, antes que el código

**Por defecto:** escribe las pruebas antes que el código siempre que la salida esperada se pueda escribir antes: transformaciones de datos, parseo, reglas de negocio, cálculo de métricas, validaciones, contratos de API.

Pasos:
1. Saca los casos de la especificación o de la petición, no del código: casos normales, límites (nulo, vacío, cero, duplicado, cambio de hora, formato ambiguo) y errores esperados.
2. Escribe cada caso como entrada y salida concreta, con el valor calculado a mano.
3. Ejecuta las pruebas y míralas fallar por la razón correcta (la función no existe o devuelve otra cosa), no por un error de sintaxis.
4. Escribe el código mínimo para que pasen.
5. Refactoriza solo en verde. Cambios de estructura y de comportamiento, en commits separados.

Por qué: una prueba escrita después de ver el código repite sus mismas suposiciones y no detecta sus errores.

**Cambiar si:** es exploración, prototipo, configuración o código de unión: pruebas después, si el código se queda. Si lo que se prueba es un modelo o un sistema con LLM, no hay TDD clásico: se fija antes un conjunto de evaluación y un umbral, y la "prueba en rojo" es la línea base medida.

**Verificar:** la salida de la ejecución en rojo existe en el registro de trabajo antes del código. Las aserciones comprueban valores concretos, no solo tipos o rangos.

**Señalar siempre:** si las pruebas se escribieron después del código, dilo en el informe.

## 2. Oráculo independiente

**Por defecto:** compara el resultado con algo que no haya salido del mismo paso que escribió el código:
- valores calculados a mano sobre una muestra pequeña;
- la versión anterior del código o del informe que ya usa negocio;
- otra implementación (una consulta SQL frente al cálculo en Python, una librería madura frente al parser propio);
- datos de referencia hechos a mano.

Por qué: si el mismo paso escribe el código, las pruebas y el valor esperado, todo comparte el mismo error.

**Cambiar si:** no hay ninguna salida esperada posible. Entonces comprueba relaciones entre ejecuciones (propiedades):
- reordenar las filas de entrada no cambia un agregado;
- ejecutar dos veces un proceso idempotente no cambia el resultado;
- filtrar por A y luego por B da lo mismo que en el orden inverso;
- añadir un documento irrelevante no cambia la respuesta.

**Verificar:**
1. Una prueba sobre datos reales compara cifras concretas, no solo que estén en un rango posible. Comprobar que un total es positivo no detecta casi ningún error; "la suma de la semana 3 es 1.250, calculada a mano con estas cinco filas" sí.
2. Mide en las dos direcciones: lo que sobra (falsos positivos) y lo que falta (falsos negativos), y lista las discrepancias concretas.
3. Si el oráculo no se puede ejecutar (falta una herramienta, faltan datos), cuenta como fallo, no como pase.

**Señalar siempre:** qué oráculo se usó y, si no hubo ninguno independiente, decirlo.

## 3. No tocar las pruebas existentes

**Por defecto:** nunca modifiques, desactives, saltes ni borres una prueba existente para que pase. No hagas casos especiales en el código para las entradas de las pruebas. Añadir pruebas nuevas sí está bien.

Por qué: es la forma más común de que un agente dé por bueno un cambio roto, y un revisor no siempre lo ve.

**Cambiar si:** la prueba es incorrecta según la especificación. Entonces para, explica por qué crees que está mal y pregunta antes de cambiarla.

**Verificar:** el diff de los ficheros de prueba solo añade casos; las pruebas originales siguen iguales.

**Señalar siempre:** cualquier prueba existente tocada, con el motivo.

## 4. Prueba diferencial en migraciones y optimizaciones

**Por defecto:** al migrar, reescribir u optimizar, ejecuta la versión anterior y la nueva con las mismas entradas y compara las salidas fila a fila. Cualquier diferencia es un posible error, aunque no sepas cuál es la respuesta correcta.

Casos típicos:
- pipeline antiguo frente al nuevo, tabla a tabla;
- consulta lenta y obvia frente a la rápida;
- métrica nueva frente al informe que ya usa negocio;
- prompt o modelo anterior frente al nuevo, sobre el mismo conjunto de evaluación.

**Cambiar si:** no existe versión anterior ejecutable. Entonces usa el apartado 2.

**Verificar:** número de filas o casos comparados y número de diferencias, con las diferencias explicadas una a una.

**Señalar siempre:** diferencias que se aceptan a propósito y por qué.

## 5. Romper el código a propósito

**Por defecto:** una prueba que nunca se ha visto fallar no está probada. Si no la viste en rojo antes del código, rompe a mano la función que acabas de escribir (devolver un valor fijo, invertir una condición, quitar un filtro, cambiar un `JOIN` interno por externo, quitar un `DISTINCT`) y comprueba que alguna prueba falla. Restaura el código después y no dejes la mutación en el commit.

Por qué: la cobertura mide qué líneas se ejecutan, no si las aserciones detectan un error; las pruebas generadas por modelos suelen cubrir mucho y comprobar poco.

**Cambiar si:** el repositorio ya tiene una herramienta de pruebas de mutación; úsala solo sobre lo que cambió, porque es lenta en suites grandes.

**Verificar:** la prueba que falla con la mutación y la que vuelve a pasar al restaurar.

**Señalar siempre:** lógica crítica (cálculos, reglas, métricas) sin ninguna mutación probada.

## 6. Probar el artefacto como se ejecuta de verdad

**Por defecto:** antes de decir "hecho", ejecuta lo entregado como se ejecutará, no solo sus pruebas unitarias.

| Artefacto | Comprobación |
|---|---|
| Paquete | Construirlo, instalarlo en un entorno limpio, importarlo y ejecutar el punto de entrada |
| CLI | Ejecutarla como proceso con argumentos reales; mirar salida estándar, salida de error y código de salida |
| Hook o plugin de agente | Lanzarlo como lo lanza el cliente, con la entrada real |
| Servidor o API | Arrancarlo en frío y hacer una petición real |
| Imagen de contenedor | Construirla, arrancarla y hacer una prueba de humo |
| Pipeline de datos | De punta a punta con datos reales pequeños, en el orquestador |
| Notebook o informe | Ejecutarlo de arriba abajo en un núcleo limpio |
| Modelo servido | Petición real al servidor con el formato de entrada de producción |

Diferencias de plataforma que revisar: codificación de texto (en Windows, forzar UTF-8), rutas con espacios, procesos hijos que esperan entrada estándar, zona horaria y configuración regional (separador decimal, formato de fecha), sin red o con proxy, permisos de solo lectura.

Por qué: las pruebas corren dentro del proceso con el código local; el artefacto real corre con otra configuración, otra codificación y otros permisos.

**Cambiar si:** no tienes acceso al entorno, a los datos o al servicio. Entonces dilo y deja el comando exacto para que lo ejecute la persona.

**Verificar:** el comando ejecutado y su salida.

**Señalar siempre:** dependencias o imágenes sin fijar, porque lo probado puede no ser lo que se despliega.

## 7. Modelo como juez: solo para lo que no se ejecuta

**Por defecto:** para el código, el juez es ejecutarlo (pruebas, oráculos). Un modelo que lee código no certifica que funcione.

**Cambiar si:** lo que se juzga no se puede ejecutar (claridad de una explicación, si una revisión cubre los requisitos). Entonces el juez:
1. responde preguntas binarias y específicas, escritas antes de ver resultados;
2. no sabe qué sistema produjo cada respuesta;
3. es un modelo distinto del que hizo el trabajo, o se advierte que no lo es;
4. se valida contra una muestra revisada a mano antes de fiarse.

**Verificar:** tasa de acuerdo del juez con la muestra revisada a mano.

**Señalar siempre:** cualquier conclusión que dependa de un juez sin validar.

## 8. Qué decir que no se comprobó

**Por defecto:** el informe final lleva siempre la sección "No comprobado y por qué", también cuando el resultado parece correcto. Es cuando más se olvida.

Repasa esta lista y escribe cada punto que aplique:
1. ¿Se ejecutó contra el servicio, la API o la base de datos real, o solo contra datos simulados?
2. ¿Se probó con datos reales, con el volumen real?
3. ¿Se ejecutó como se ejecutará (proceso, instalación, plataforma, sin red)?
4. ¿Se leyeron todos los ficheros y usos relevantes, o solo los encontrados con una búsqueda?
5. ¿Hay usos que la búsqueda no ve (llamadas dinámicas, configuración, SQL en cadenas, otros repositorios)?
6. ¿Se midió el rendimiento, la memoria o el coste, o se estimó?
7. ¿Las cifras del informe son medidas, del fabricante o supuestas?
8. ¿Algún comando no se pudo ejecutar por permisos o por el entorno?

Por qué: lo que no se comprobó y no se dice parece hecho, y quien lo recibe no tiene forma de saberlo.

**Cambiar si:** no aplica.

**Verificar:** cada punto "no comprobado" lleva el motivo y el comando o paso para comprobarlo.

**Señalar siempre:** si todo se comprobó, cómo se sabe; "nada pendiente" sin evidencia no vale.
