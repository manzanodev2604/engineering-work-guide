# Entornos restringidos

Trabajo sin conexión, con datos sensibles o en sectores regulados (defensa, sector público, infraestructuras críticas). Cada apartado da el valor por defecto, la señal medida para cambiarlo, cómo verificar y los riesgos que señalar siempre.

## Índice

1. Reconocer el entorno
2. Nada sale del perímetro
3. Trabajar sin internet
4. Dependencias
5. Modelos y pesos
6. Imágenes y artefactos
7. Normativa: preguntar y señalar
8. Procedencia y auditoría

Fuente de las prácticas: investigación del 25/10/2026. Nombres de normas y de herramientas: comprobar en el momento.

---

## 1. Reconocer el entorno

**Por defecto:** antes de cualquier acción hacia fuera, busca señales de entorno restringido en el `README`, las normas del repositorio y la petición:
- "sin conexión", "aislado", "sin internet", red con proxy;
- defensa, doble uso, sector público, infraestructura crítica, información clasificada;
- datos personales, de salud o de clientes.

Si aparece alguna, todo el trabajo sigue este fichero.

**Cambiar si:** la persona confirma por escrito qué accesos están permitidos y para qué.

**Verificar:** el informe dice qué señal se encontró y dónde (`fichero:línea`).

**Señalar siempre:** las señales contradictorias (por ejemplo, un `README` que dice "sin conexión" y un script que descarga algo).

## 2. Nada sale del perímetro

**Por defecto:** en un entorno restringido no se intenta ninguna salida sin permiso explícito:
1. Ni búsquedas web ni consultas a páginas, aunque sean de documentación.
2. Ni descargas de paquetes, modelos o imágenes desde registros públicos.
3. Ni envío de código, datos, trazas o telemetría a servicios externos.
4. Ni pedir permiso "para consultar un momento" como primer paso: primero agota lo que hay en local.

Si algo de verdad necesita salir, para y presenta qué saldría, a dónde, por qué hace falta y la alternativa local.

Por qué: el intento ya puede sacar datos o metadatos, y en estos entornos una salida no autorizada es un incidente aunque no pase nada.

**Cambiar si:** hay una vía aprobada (espejo interno, registro interno de modelos); entonces se usa esa y solo esa.

**Verificar:** la transcripción no contiene accesos a red no autorizados; el informe lista lo que se habría necesitado de fuera.

**Señalar siempre:** que el propio agente, si usa un modelo en la nube, ya envía fuera lo que lee. Dilo si el repositorio contiene datos sensibles.

## 3. Trabajar sin internet

**Por defecto:**
1. Comprueba cada API contra el código instalado antes de usarla: el código fuente de la librería en el entorno, su ayuda integrada o la inspección de sus funciones.
2. Manda la versión instalada, no la que recuerda el modelo.
3. No propongas dependencias que no estén en el espejo interno sin decirlo: añadir una es un trámite, no una instalación.
4. Desarrolla con datos simulados generados desde el esquema; los datos reales, solo en el entorno autorizado.

Por qué: sin documentación en línea el modelo tiende a inventar funciones o a usar las de otra versión.

**Cambiar si:** no aplica; sin conexión, la fuente es siempre lo instalado.

**Verificar:** cada función externa usada existe en la versión instalada (comprobado con un comando, no de memoria).

**Señalar siempre:** las APIs que no se pudieron comprobar.

## 4. Dependencias

**Por defecto:**
1. Fichero de bloqueo con versiones exactas y hashes de todo el árbol; la instalación falla si un hash no coincide.
2. Instalación reproducible desde el espejo interno o desde un directorio de paquetes transferido.
3. Cada dependencia nueva, justificada y revisada: existe, es la que se pretendía (no un nombre parecido), está mantenida, su licencia es compatible.
4. Menos dependencias mejor: cada una es algo más que revisar, transferir y mantener.

Por qué: lo que se ejecuta es el paquete publicado, no el código revisado, y pueden no coincidir.

**Cambiar si:** el repositorio ya tiene otro mecanismo de bloqueo; se usa el suyo.

**Verificar:** instalar en limpio desde el fichero de bloqueo y sin red.

**Señalar siempre:** dependencias sin fijar o sugeridas por el modelo sin revisar.

## 5. Modelos y pesos

**Por defecto:**
1. Pesos solo en formatos que no ejecutan código al cargarse (safetensors; GGUF para motores que lo usan).
2. Cualquier fichero serializado con pickle (`.bin`, `.pt`, `.pkl`, joblib) se trata como código no fiable.
3. No activar la ejecución de código remoto del repositorio del modelo sin revisar ese código.
4. Descargar una vez por la vía aprobada, anotar la revisión exacta y los hashes, guardarlo en un registro interno y desplegar siempre desde ahí.
5. Licencia y política de uso comprobadas para el sector y el uso previsto antes de cualquier prueba técnica.

**Cambiar si:** solo existe el formato pickle; entonces carga en modo de solo pesos y en un entorno aislado, y dilo.

**Verificar:** los hashes de los ficheros desplegados coinciden con los anotados.

**Señalar siempre:** modelos sin licencia clara o con restricciones de uso que toquen el sector.

## 6. Imágenes y artefactos

**Por defecto:**
1. Lista de componentes (SBOM) por imagen.
2. Imagen firmada; sin conexión, con clave propia o con verificación que no consulte servicios públicos.
3. Análisis de vulnerabilidades con la base de datos del analizador actualizada a mano y con fecha.
4. CI con permisos mínimos, sin interpolar en comandos datos que controlan terceros (títulos, nombres de rama), y acciones fijadas por hash.

Por qué: una firma demuestra quién publicó la imagen, no que esté libre de vulnerabilidades; y un análisis con la base de datos antigua da falsa tranquilidad.

**Cambiar si:** la empresa tiene su propia cadena de publicación; se sigue la suya.

**Verificar:** fecha de la base de datos de vulnerabilidades usada en el último análisis.

**Señalar siempre:** artefactos sin SBOM, sin firma o analizados con una base antigua.

## 7. Normativa: preguntar y señalar

**Por defecto:**
1. Pregunta el uso previsto (exclusivamente defensa o doble uso) y el tipo de cliente (sector público, infraestructura crítica) cuando el sistema toque esos ámbitos.
2. No des por hecha ninguna exclusión, tampoco la militar, en la normativa de IA o de ciberseguridad.
3. Señala la posible obligación (protección de datos, esquema de seguridad del sector público, normativa de IA, de ciberseguridad de productos o de información clasificada) y deriva a quien corresponda.

Por qué: el uso previsto decide qué normas aplican, y un error aquí no se arregla con código.

**Cambiar si:** la empresa ya tiene el análisis hecho; se sigue y se cita.

**Verificar:** el informe dice qué se preguntó y qué respondieron.

**Señalar siempre:** que la guía no da dictámenes jurídicos.

## 8. Procedencia y auditoría

**Por defecto:**
1. Cada resultado que apoya una decisión lleva su procedencia: entradas, código, modelo, prompt, ejecución y petición, identificados por versión o hash.
2. Se explica con evidencias (fuentes, consultas, código, pruebas), nunca con la autoexplicación del modelo.
3. Si el sistema apoya decisiones con consecuencias, una persona decide, con la incertidumbre visible y la aprobación registrada.
4. Registro de auditoría a prueba de manipulación: cadena de hashes con la raíz firmada y copiada fuera del sistema. Nunca criptografía propia.

**Cambiar si:** hay mucho volumen o terceros que verifican el registro: árbol de Merkle con componentes ya probados.

**Verificar:** se puede reconstruir un resultado concreto desde su procedencia.

**Señalar siempre:** que el registro de auditoría hereda la sensibilidad de lo que registra.
