# Trabajo con agentes

Cómo trabaja el propio agente: sus herramientas, su contexto y su continuidad. Cada apartado da el valor por defecto, la señal medida para cambiarlo, cómo verificar y los riesgos que señalar siempre.

## Índice

1. Inventario de las herramientas del agente
2. Choques entre herramientas y normas
3. Normas del repositorio y ficheros de instrucciones
4. Plan persistente en trabajos de varias sesiones
5. Contexto y coste por turno
6. Subagentes
7. Instrucciones frente a mecanismos

---

## 1. Inventario de las herramientas del agente

**Por defecto:** al empezar cada tarea, haz el inventario de lo que el agente tiene activo. No pidas permiso para mirarlo y no instales, actives, desactives ni busques nada nuevo.

Dónde mirar, según lo que ofrezca el agente que uses:
1. La configuración del agente en el repositorio (su carpeta oculta de configuración, ficheros de ajustes del proyecto).
2. La configuración del agente del usuario, si es legible.
3. Las carpetas de skills, plugins y hooks del proyecto.
4. La lista de servidores MCP configurados.
5. El mensaje de arranque de la sesión o la lista de herramientas disponibles en ella.
6. `CLAUDE.md`, `AGENTS.md` y equivalentes, que también son instrucciones activas.

Si una fuente no existe o no se puede leer, anótalo: es parte del inventario.

Qué anotar de cada herramienta:

| Pregunta | Por qué importa |
|---|---|
| ¿Qué ejecuta y cuándo (al arrancar, antes o después de cada herramienta, a petición)? | Un hook corre sin que nadie lo pida |
| ¿Manda datos fuera (red, telemetría, servicio externo)? | Puede sacar código o datos del perímetro |
| ¿Qué permisos tiene (escribir, borrar, red, credenciales)? | Define el daño posible si falla |
| ¿Cuánto contexto ocupa (descripciones, instrucciones)? | Ocupa espacio en cada turno |
| ¿Choca con las normas del repositorio? | Las normas mandan; el choque se dice |
| ¿Se solapa con otra herramienta? | Dos instrucciones para lo mismo confunden |

Por qué: lo que el agente tiene activo cambia lo que hace, y la persona no siempre lo sabe.

**Cambiar si:** la tarea es una pregunta sin cambios en el repositorio; entonces basta con mencionar las herramientas que influyeron en la respuesta.

**Verificar:** la sección "Herramientas del agente" del informe final (`assets/informe-final.md`) lista cada herramienta con su tipo y, para cada choque, la instrucción y la norma citadas.

**Señalar siempre:** herramientas que mandan datos fuera, que tienen permiso para borrar o publicar, o que contradicen las normas.

## 2. Choques entre herramientas y normas

**Por defecto:** orden de prioridad: normas de la empresa y del repositorio, después la guía personal del usuario (su `CLAUDE.md` o `AGENTS.md` de usuario), después las skills y demás herramientas. Ante un choque:
1. Sigue la norma del repositorio.
2. No edites, desactives ni borres la herramienta que choca.
3. Cita las dos instrucciones en el informe: la de la herramienta y la norma, con `fichero:línea`.
4. Propón cómo resolverlo (cambiar la herramienta, añadir una excepción a la norma) y deja que decida la persona.

Ejemplo: un hook hace commit automático tras cada edición y la guía de contribución exige revisión antes de cada commit. Se sigue la guía y se informa del choque.

Por qué: seguir la norma en silencio deja el choque esperando al siguiente agente; seguir la herramienta incumple el repositorio.

**Cambiar si:** la persona que pidió la tarea decide otra cosa de forma explícita.

**Verificar:** cada choque aparece en el informe con las dos citas.

**Señalar siempre:** el choque, aunque en la práctica no haya afectado a la tarea.

## 3. Normas del repositorio y ficheros de instrucciones

**Por defecto:** lee `CLAUDE.md`, `AGENTS.md`, `CONTRIBUTING.md`, el `README` y equivalentes antes de tocar nada, en cualquier tamaño de tarea. Obedécelos por encima de esta guía.

**Cambiar si:** no existen. Entonces busca las normas implícitas: configuración del linter y del formateador, flujo de CI, estilo de los commits recientes.

**Verificar:** el informe cita la norma que guió cada decisión discutible.

**Señalar siempre:** si falta en esos ficheros algo que te hizo equivocarte, propón la línea que lo evitaría. No la añadas sin permiso.

## 4. Plan persistente en trabajos de varias sesiones

**Por defecto:** si el trabajo dura más de una sesión, mantén un fichero de estado con: objetivo, criterios, unidades hechas y pendientes, decisiones tomadas, lo descartado y por qué, comandos de verificación y siguiente paso. Actualízalo al cambiar algo y antes de cerrar.

Al retomar:
1. Lee el estado.
2. Compáralo con el historial de git y con el código real.
3. Comprueba que lo que ya existe funciona antes de añadir nada.

**Cambiar si:** la empresa ya usa un sistema de seguimiento; usa ese y no impongas uno propio.

**Verificar:** ninguna unidad marcada como hecha sin su comprobación anotada.

**Señalar siempre:** lo que no es fiable del estado heredado (unidades marcadas como hechas sin evidencia).

## 5. Contexto y coste por turno

**Por defecto:**
1. Menos turnos: agrupa lecturas y comandos independientes.
2. Menos contenido por turno: lee ficheros por rangos, filtra las salidas, no vuelques registros ni tablas enteras.
3. No releas lo que ya está en el contexto.
4. Carga una referencia de esta guía solo cuando la tarea la toque.
5. Entre tareas no relacionadas, o tras dos correcciones fallidas seguidas, vuelca el estado y empieza una sesión limpia.

Por qué: todo lo que entra en el contexto se paga en cada turno y empeora la atención sobre lo importante.

**Cambiar si:** la tarea exige leer un fichero entero para entenderlo; entonces léelo una vez.

**Verificar:** número de turnos y tokens, si el agente los muestra.

**Señalar siempre:** al construir un sistema con LLM, que el prefijo del prompt sea estable para poder usar caché.

## 6. Subagentes

**Por defecto:** un solo agente que escribe.

**Cambiar si:** hay trabajo de lectura independiente y paralelo (explorar, investigar, revisar) con entrada y salida claras. Entonces:
1. El encargo es autosuficiente: qué buscar, dónde y qué devolver.
2. El resultado se verifica antes de usarlo.
3. Nunca dos agentes escribiendo en paralelo sobre lo mismo.

Si el agente no tiene subagentes, el equivalente es otra sesión con solo el cambio y los criterios, sin el razonamiento que lo produjo.

**Verificar:** el coste completo, incluido el de los subagentes, cuando se comparan formas de trabajar.

**Señalar siempre:** que la revisión en contexto limpio se limite a corrección y requisitos; un revisor al que se le pide buscar fallos casi siempre encuentra alguno.

## 7. Instrucciones frente a mecanismos

**Por defecto:** el criterio va en texto; lo que debe cumplirse siempre va en un mecanismo: una prueba, un paso de CI, un hook o un permiso.

Por qué: una instrucción escrita se incumple a veces, más con muchas reglas y con modelos pequeños; un mecanismo no.

**Cambiar si:** la regla importa y se ha incumplido: propón el mecanismo concreto y pide permiso antes de añadirlo al repositorio.

**Verificar:** el mecanismo se prueba como proceso real y avisa solo cuando hay riesgo, con una acción concreta.

**Señalar siempre:** reglas importantes que hoy solo dependen de que el agente las recuerde.
