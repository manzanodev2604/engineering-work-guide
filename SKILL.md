---
name: engineering-work-guide
description: Use when working on any task inside a code repository, before reading or editing files: implementing or refactoring code, fixing bugs or failing tests, improving performance, changing schemas or migrating tables, building data pipelines, SQL or analytics, evaluating prompts, models or LLM/RAG systems, reviewing diffs, or deleting files and data. Not for general knowledge questions, translations, jokes or explaining a single concept.
---

# Guía de trabajo de ingeniería

Guía de método para trabajar en un repositorio con código, datos, analítica, ML o sistemas con LLM. No explica tecnologías: dice qué hacer, en qué orden y dónde parar.

**Las normas de la empresa y del repositorio mandan sobre esta guía.** Si una norma contradice algo de aquí, sigue la norma y di el choque en el informe final.

## El ciclo

Sigue estos cinco pasos en orden, también en cambios pequeños.

1. **Entender y estimar.**
   1. Lee las normas del repositorio (parada 1).
   2. Haz el inventario de las herramientas del agente: skills, plugins, hooks y servidores MCP activos. Sácalo de lo que ya tienes: la lista de herramientas y skills de la sesión y los ficheros de configuración que ya hayas visto al leer las normas. No gastes comandos solo en buscarlos salvo que la tarea trate de ellos. No instales, actives ni desactives nada.
   3. Anota qué herramientas chocan con las normas del repositorio, cuáles se solapan entre sí y cuáles mandan datos fuera. Ejemplo de choque: un hook que hace commit automático frente a una norma que exige revisión antes de cada commit. Ante un choque, sigue las normas. Por qué: una instrucción de una herramienta puede contradecir al repositorio y nadie lo verá si no se dice.
   4. En tareas de diseño, cifras antes de arquitectura: volumen, lo que lee cada ejecución, latencia y frescura necesarias, coste mensual y coste de un error (plantilla en `assets/estimacion.md`). Marca cada cifra como medida o supuesta.
   5. Escribe las suposiciones. Si la petición admite dos lecturas que cambian el comportamiento, aplica la parada 3 antes de seguir.
2. **Planificar.**
   1. Si el cambio cabe en una frase, no escribas plan; las pruebas primero y las cinco paradas siguen valiendo.
   2. Si no, escribe el plan con `assets/plan.md`: objetivo, criterios Dado/Cuando/Entonces, fuera de alcance, unidades y riesgos.
   3. Las decisiones difíciles de deshacer (modelo de datos, proveedor, formato, esquema) van en un ADR con `assets/adr.md`.
3. **Hacer en unidades pequeñas.**
   1. Escribe las pruebas desde la especificación **antes** que el código y míralas fallar. Por qué: una prueba escrita después de ver el código repite sus mismos errores.
   2. Una unidad cada vez, terminada y en verde antes de la siguiente; un commit por unidad si las normas lo permiten.
   3. Cambio incremental sobre lo que existe; no reescribas sin una razón y sin permiso.
4. **Verificar con evidencia.**
   1. Ejecuta las comprobaciones ahora y lee la salida; lo que pasó antes no cuenta.
   2. Compara con algo que no salga del mismo paso: valores calculados a mano, la versión anterior u otra implementación.
   3. Ejecuta el resultado como se ejecutará de verdad: proceso real, datos reales pequeños, entrada real.
5. **Entregar con informe** (formato más abajo).

## Cinco paradas obligatorias

Detente en cada una aunque nadie lo pida.

1. **Normas primero.** Antes de tocar nada, lee `CLAUDE.md`, `AGENTS.md`, `CONTRIBUTING.md`, el `README` y cualquier guía de contribución del repositorio, también si el cambio es de una línea. Por qué: las normas se saltan sobre todo en los cambios pequeños, y mandan sobre esta guía.
2. **Localizar antes de editar código compartido.** Busca la definición, cada uso y las pruebas que lo cubren, y cítalos como `fichero:línea` (por ejemplo, `src/precios.py:42`), no solo el fichero. Di qué no pudiste ver: usos dinámicos, configuración, SQL dentro de cadenas, otros repositorios. Por qué: con la línea, cualquiera comprueba la afirmación en segundos.
3. **Reglas de desviación.**
   1. Corrige por tu cuenta lo que bloquea la tarea o es inseguro, y anótalo.
   2. Consulta antes de decidir: cambios de alcance, de arquitectura, de interfaz, de datos o de dependencias.
   3. Consulta también cualquier ambigüedad que cambie el comportamiento, como un peso que puede venir en kilos o en libras, o una duración en segundos o en milisegundos. Plantéala con las opciones y tu recomendación **antes** de escribir el código, no después.
   4. Si no puedes esperar la respuesta, escribe primero la pregunta, sigue con la opción que conserva el comportamiento actual y márcala como suposición.

   Por qué: una decisión de comportamiento tomada en silencio llega a producción sin que nadie la haya elegido.
4. **Antes de una acción irreversible o hacia fuera**, para. Son acciones de este tipo: borrar, sobrescribir, publicar, desplegar, gastar y enviar datos fuera. Presenta qué harías, las opciones, tu recomendación y una pregunta. Si el `README`, las normas o la petición indican un entorno sin conexión o sensible, cualquier consulta a internet o a servicios externos (búsquedas, descargas, registros de paquetes o de modelos) también es una acción hacia fuera: no la intentes; trabaja con lo que hay en local y anota en «Pendiente» qué haría falta de fuera y por qué. Por qué: estas acciones no se deshacen, y el intento ya puede sacar datos del perímetro.
5. **Antes de decir "hecho".** Comprueba que:
   1. cada criterio está comprobado con evidencia de ahora;
   2. la suite completa está en verde;
   3. lo entregado se ha ejecutado como se ejecutará;
   4. ninguna prueba existente se ha modificado, desactivado ni borrado para que pase (añadir pruebas nuevas sí está bien);
   5. el informe dice qué no se comprobó y por qué.

   Por qué: el informe del agente no es una prueba; lo que no se comprobó y no se dice parece hecho.

## Informe final

Escríbelo directamente con estas secciones, en este orden (no hace falta abrir `assets/informe-final.md`):

1. **Hecho**: qué cambió, con `fichero:línea`.
2. **Comprobado**: cada comprobación con su comando y su resultado.
3. **No comprobado y por qué**: obligatoria siempre. Si comprobaste todo, di cómo lo sabes. Ejemplos: no probado contra el servicio real, sin datos reales, sin ejecutar en otra plataforma, ficheros que no leíste, rendimiento a escala.
4. **Suposiciones**: cada una con lo que cambiaría si fuera falsa.
5. **Pendiente**: lo que queda y quién debe hacerlo.
6. **Desviaciones**: lo que hiciste distinto del plan o de la petición.
7. **Herramientas del agente**: el inventario del paso 1, los choques con las normas, los solapes y las carencias que mostró la tarea. Sin instalar nada.

## Cuándo leer cada referencia

Lee una referencia solo cuando la tarea la toque, y solo la que toque; no las recorras todas. Cada una da el valor por defecto, la señal medida para cambiarlo, cómo verificar y los riesgos que señalar siempre.

| Si la tarea... | Lee |
|---|---|
| Escribe o cambia código, pruebas o cálculos; o hay que comprobar un resultado | `references/verificacion.md` |
| Depende de cómo trabaja el agente: inventario de herramientas, choques, plan entre sesiones, contexto, subagentes | `references/trabajo-con-agentes.md` |
| Ocurre sin conexión, en defensa, en el sector público o con datos sensibles; o añade dependencias, modelos o imágenes | `references/entornos-restringidos.md` |
| Usa un LLM: chatbot, RAG, extracción, agente, evaluación, coste de llamadas | `references/ia.md` |
| Entrena, valida, despliega, cuantiza o ajusta un modelo, clásico o abierto | `references/ml-y-modelos-abiertos.md` |

## Explica el porqué

Explica el porqué de cada decisión en una o dos frases, también en lo que parece obvio. Ejemplo: "Una tabla nueva y no una columna JSON, porque cada consulta filtra por ese campo."

## Lista de comprobación

Repásala antes del informe final; no hace falta copiarla.

```
[ ] Normas del repositorio leídas (CLAUDE.md, AGENTS.md, CONTRIBUTING.md, README)
[ ] Inventario de skills, plugins, hooks y MCP; choques con las normas anotados
[ ] Suposiciones escritas; ambigüedades de comportamiento planteadas antes de decidir
[ ] Usos del código compartido citados con fichero:línea
[ ] Pruebas escritas desde la especificación y vistas en rojo antes del código
[ ] Ninguna acción irreversible o hacia fuera sin permiso
[ ] Suite completa en verde, ejecutada ahora
[ ] Ejecutado como se ejecutará de verdad
[ ] Informe con "No comprobado y por qué" y "Herramientas del agente"
```
