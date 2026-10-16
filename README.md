# Laboratorio 2 — Especificaciones de la App

## Objetivo de la app

**Ayudar a profesionales junior e intermedios de tecnología desplazados o sin primer empleo por la automatización con IA a reconvertir sus habilidades hacia roles de baja exposición y conectarse con vacantes no automatizables, de modo que al menos el 70% complete su micro-ruta de 8 semanas, consiga tres entrevistas en 60 días y reciba una oferta en menos de 90 días.**

Este objetivo une las tres opciones del Laboratorio 1. La ruta de reskilling se acortó de 6 meses a 8 semanas para que sea compatible con la meta de oferta en 90 días.

El flujo de valor para el usuario tiene cuatro pasos: diagnóstico de exposición a IA, micro-ruta de reskilling con proyectos, portafolio verificable y postulación a vacantes de baja exposición con seguimiento de metas.

---

## Historias de usuario

Seis historias cubren el objetivo; las cinco primeras son del usuario principal y la sexta cierra el ciclo con la medición de metas.

| ID | Historia | Parte del objetivo |
| --- | --- | --- |
| HU-01 | Como profesional junior desplazado, quiero registrarme y crear mi perfil con experiencia y habilidades, para que la app conozca mi punto de partida. | Base de todo el flujo |
| HU-02 | Como usuario, quiero un diagnóstico de qué tan expuesto a la IA está mi rol actual, para saber hacia qué roles de baja exposición puedo moverme. | Reskilling |
| HU-03 | Como usuario, quiero seguir una micro-ruta de 8 semanas con proyectos prácticos, para reconvertirme rápido hacia el rol elegido. | Reskilling (meta 70%) |
| HU-04 | Como usuario, quiero un portafolio generado a partir de los proyectos que completé, para demostrar mis habilidades a los reclutadores. | Empleabilidad rápida |
| HU-05 | Como usuario, quiero ver vacantes de roles de baja exposición compatibles con mi perfil y postularme desde la app, para concentrar mi esfuerzo en empleos con futuro. | Conexión con vacantes |
| HU-06 | Como usuario, quiero un tablero con mis postulaciones y mi avance frente a las metas de 60 y 90 días, para saber si voy en camino. | Medición (3 entrevistas / oferta) |

---

## Mapa de Componentes v1

La app se organiza en 6 módulos y 18 componentes, que siguen el recorrido del usuario: registro, diagnóstico, reskilling, portafolio, vacantes y seguimiento.

```
[M1 Cuenta y perfil] → [M2 Diagnóstico] → [M3 Reskilling]
                                                 ↓
[M6 Seguimiento]   ←   [M5 Vacantes]   ←   [M4 Portafolio]
```

**Módulos y componentes** (en negrita, los componentes especificados en este documento)

| Módulo | Componentes |
| --- | --- |
| M1 Cuenta y perfil | **C1 Formulario de registro y perfil (UI)** · C2 Servicio de autenticación (Servicio) · C3 BD de usuarios y perfiles (Datos) |
| M2 Diagnóstico | C4 Cuestionario de diagnóstico (UI) · **C5 Motor de diagnóstico de exposición (Servicio)** · C6 Catálogo de roles y exposición (Datos) |
| M3 Reskilling | C7 Vista de micro-ruta (UI) · **C8 Servicio de rutas de reskilling (Servicio)** · C9 Catálogo de módulos y proyectos (Datos) |
| M4 Portafolio | **C10 Generador de portafolio (Servicio)** · C11 Página pública de portafolio (UI) · C12 Integración con GitHub (Integración) |
| M5 Vacantes y postulaciones | C13 Buscador de vacantes (UI) · **C14 Servicio de matching de vacantes (Servicio)** · C15 Integración con portales de empleo (Integración) · C16 BD de vacantes y postulaciones (Datos) |
| M6 Seguimiento | **C17 Tablero de seguimiento de metas (UI)** · C18 Servicio de métricas y alertas (Servicio) |

**Dependencias principales**

- C1 → C2 → C3: el formulario crea la cuenta y guarda el perfil.
- C4 → C5 → C6: el cuestionario envía respuestas al motor, que consulta el catálogo de roles.
- C7 → C8 → C9: la vista de ruta pide la ruta al servicio, que toma módulos del catálogo.
- C8 → C10 → C11: los proyectos aprobados alimentan el portafolio y su página.
- C15 → C16 ← C14 → C6: la integración carga vacantes; el matching las lee y consulta la exposición de cada rol.
- C14 → C10: cada postulación adjunta el resumen del portafolio.
- C8 y C16 → C17 → C18: el tablero lee avance y postulaciones y envía eventos para las alertas.

**Notas**

- **Incluye en MVP:** los 6 módulos y 17 componentes. C15 arranca con una sola fuente de vacantes, o con carga manual por un administrador, hasta confirmar qué portal ofrece acceso por API.
- **Futuro:** C12 Integración con GitHub, varias fuentes de vacantes y un perfil para empresas que publiquen vacantes directamente.

---

## Especificación C1: Formulario de registro y perfil

**Tipo:** UI · **Módulo:** M1 Cuenta y perfil · **Historias:** HU-01

**Propósito:** crear la cuenta del usuario y capturar su punto de partida (rol, experiencia, habilidades y situación laboral) para alimentar el diagnóstico.

**Entradas**

- Nombre, correo y contraseña.
- Rol actual o último rol, años de experiencia (0 a 5 para el segmento objetivo).
- Habilidades (selección de una lista más campo libre).
- Situación laboral: desplazado, buscando primer empleo o empleado en riesgo.
- Ciudad, país y modalidad deseada (presencial, híbrida, remota); enlace a GitHub opcional.

**Salidas**

- Cuenta creada en el Servicio de autenticación (C2) y perfil guardado en la BD de usuarios (C3).
- Redirección al Cuestionario de diagnóstico (C4).

**Reglas de negocio**

- El correo es único por cuenta.
- La contraseña tiene mínimo 8 caracteres e incluye al menos una letra y un número.
- Aceptar la política de tratamiento de datos es obligatorio para registrarse.
- Se requieren al menos 3 habilidades para habilitar el diagnóstico.
- Si el usuario declara más de 5 años de experiencia, se registra igual, pero se le informa que la app está pensada para perfiles junior e intermedios.

**Flujo principal**

1. El usuario abre el registro e ingresa nombre, correo y contraseña.
2. Acepta la política de datos y crea la cuenta.
3. Completa rol, experiencia, habilidades, situación laboral y ubicación.
4. El sistema valida los campos y guarda el perfil.
5. El sistema lo lleva al diagnóstico.

**Requisitos no funcionales**

- **Rendimiento:** la validación de cada campo responde en menos de 200 ms; el guardado del perfil tarda menos de 2 s en el percentil 95.
- **Seguridad:** contraseñas almacenadas con hash (bcrypt o Argon2), nunca en texto plano; tráfico solo por HTTPS (TLS 1.2 o superior); consentimiento registrado conforme a la Ley 1581 de 2012 si se opera en Colombia.
- **Accesibilidad:** cumple WCAG 2.1 nivel AA: cada campo tiene etiqueta, el formulario se completa solo con teclado y los errores se anuncian al lector de pantalla.

**Criterios de aceptación**

- Dado un correo no registrado y datos válidos, cuando el usuario envía el formulario, entonces se crea la cuenta y se le muestra el diagnóstico en menos de 2 s.
- Dado un correo ya registrado, cuando el usuario envía el formulario, entonces se muestra "Este correo ya tiene una cuenta" y no se crea un duplicado.
- Dado un perfil con menos de 3 habilidades, cuando el usuario intenta continuar, entonces el botón de diagnóstico sigue deshabilitado y se indica cuántas faltan.
- Dado que el usuario no aceptó la política de datos, cuando intenta crear la cuenta, entonces el registro no se completa y se resalta la casilla pendiente.

---

## Especificación C5: Motor de diagnóstico de exposición a IA

**Tipo:** Servicio · **Módulo:** M2 Diagnóstico · **Historias:** HU-02

**Propósito:** calcular qué tan expuesto a la automatización con IA está el rol actual del usuario y proponer roles destino de menor exposición con su brecha de habilidades.

**Entradas**

- Perfil del usuario desde C3: rol, experiencia y habilidades.
- Respuestas del Cuestionario de diagnóstico (C4): tareas que realiza y porcentaje de tiempo en cada una.
- Catálogo de roles y exposición (C6).

**Salidas**

- Nivel de exposición del rol actual: Alta, Media o Baja, con una explicación de 2 a 3 líneas.
- Hasta 3 roles destino sugeridos, cada uno con porcentaje de coincidencia de habilidades y lista de habilidades faltantes.
- Registro del diagnóstico en el historial del usuario.

**Reglas de negocio**

- El nivel de exposición de cada rol se toma del catálogo C6, y cada valor del catálogo debe citar su fuente (estudio o informe).
- Si el rol del usuario no está en el catálogo, se usa el rol más parecido y el resultado se marca como "estimado".
- Solo se sugieren roles con exposición menor a la del rol actual.
- Coincidencia = habilidades del usuario que exige el rol ÷ habilidades totales que exige el rol.
- Cada vez que el usuario actualiza su perfil, puede generar un diagnóstico nuevo; los anteriores se conservan en el historial.

**Flujo principal**

1. C4 envía las respuestas del cuestionario junto con el identificador del usuario.
2. El servicio obtiene el perfil y busca el rol en el catálogo.
3. Calcula el nivel de exposición del rol actual.
4. Calcula la coincidencia con los roles de menor exposición y ordena los 3 mejores.
5. Guarda el resultado y lo devuelve a la interfaz.

**Requisitos no funcionales**

- **Rendimiento:** responde en menos de 3 s en el percentil 95.
- **Seguridad:** el diagnóstico solo lo ve el usuario; no se comparte con empresas sin su consentimiento explícito; el servicio solo acepta llamadas autenticadas.
- **Confiabilidad:** el cálculo es determinista: la misma entrada siempre produce el mismo resultado.
- **Accesibilidad:** el nivel se muestra con texto (Alta, Media, Baja) y no solo con color.

**Criterios de aceptación**

- Dado un usuario con rol de soporte técnico que existe en el catálogo, cuando completa el cuestionario, entonces recibe su nivel de exposición y hasta 3 roles sugeridos con habilidades faltantes en menos de 3 s.
- Dado un rol que no existe en el catálogo, cuando se genera el diagnóstico, entonces el resultado se marca como "estimado" e indica el rol usado como referencia.
- Dado un rol actual con exposición Baja, cuando se generan sugerencias, entonces no se propone ningún rol con exposición Media o Alta.
- Dado el mismo perfil y las mismas respuestas, cuando se repite el diagnóstico, entonces el resultado es idéntico.

---

## Especificación C8: Servicio de rutas de reskilling

**Tipo:** Servicio · **Módulo:** M3 Reskilling · **Historias:** HU-03

**Propósito:** generar una micro-ruta de 8 semanas hacia el rol destino elegido y registrar el avance del usuario, que alimenta la meta de 70% de rutas completadas.

**Entradas**

- Rol destino elegido y habilidades faltantes, ambos desde el diagnóstico (C5).
- Horas disponibles por semana que declara el usuario.
- Catálogo de módulos y proyectos (C9).
- Eventos de avance: módulo iniciado, proyecto entregado, proyecto aprobado.

**Salidas**

- Ruta de 8 semanas con módulos por semana y al menos 2 proyectos prácticos.
- Porcentaje de avance y estado de la ruta: activa, pausada, completada o abandonada.
- Lista de proyectos aprobados, que consume el Generador de portafolio (C10).

**Reglas de negocio**

- Solo se incluyen módulos que cubren al menos una habilidad faltante.
- La carga de cada semana no supera las horas declaradas por el usuario.
- Un módulo cuenta como completado solo cuando su proyecto o evaluación queda aprobado.
- La ruta pasa a "abandonada" tras 14 días sin actividad; el usuario puede pausarla una vez por hasta 14 días sin perder el avance.
- La ruta se considera completada cuando todos sus proyectos están aprobados; esto suma a la métrica del 70%.

**Flujo principal**

1. El usuario elige un rol destino y declara sus horas por semana.
2. El servicio selecciona módulos del catálogo según las habilidades faltantes.
3. Reparte los módulos en 8 semanas respetando las horas declaradas.
4. El usuario avanza y entrega proyectos; el servicio actualiza avance y estado.
5. Al aprobar el último proyecto, marca la ruta como completada y notifica al portafolio.

**Requisitos no funcionales**

- **Rendimiento:** la ruta se genera en menos de 5 s; cada cambio de avance se guarda en menos de 1 s.
- **Disponibilidad:** el avance no se pierde si el usuario cierra la app a mitad de un módulo.
- **Seguridad:** solo el usuario dueño de la ruta puede modificar su avance.
- **Accesibilidad:** todo video del catálogo tiene subtítulos o transcripción.

**Criterios de aceptación**

- Dado un usuario con rol destino elegido y 10 horas semanales, cuando solicita su ruta, entonces recibe 8 semanas con al menos 2 proyectos y ninguna semana supera 10 horas.
- Dado un módulo con proyecto entregado pero no aprobado, cuando se consulta el avance, entonces ese módulo no cuenta como completado.
- Dada una ruta activa sin actividad durante 14 días, cuando corre la revisión diaria, entonces su estado cambia a "abandonada" y se notifica al usuario.
- Dado que el usuario aprueba el último proyecto, cuando se registra la aprobación, entonces la ruta queda "completada" y sus proyectos aparecen disponibles para el portafolio.

---

## Especificación C10: Generador de portafolio

**Tipo:** Servicio · **Módulo:** M4 Portafolio · **Historias:** HU-04

**Propósito:** convertir los proyectos aprobados de la ruta en un portafolio que el usuario pueda compartir con reclutadores como evidencia de sus habilidades.

**Entradas**

- Proyectos aprobados desde el Servicio de rutas (C8), con descripción y habilidades que demuestran.
- Datos de perfil desde C3: nombre, rol destino, habilidades.
- Repositorios de GitHub, si el usuario conectó su cuenta mediante la Integración con GitHub (C12).
- Proyectos externos que el usuario agregue a mano.

**Salidas**

- Página de portafolio (C11) con URL propia.
- Versión descargable en PDF.
- Resumen del portafolio que el Servicio de matching (C14) adjunta a cada postulación.

**Reglas de negocio**

- Los proyectos aprobados en la app llevan la etiqueta "verificado"; los agregados a mano llevan la etiqueta "declarado".
- El portafolio es privado por defecto; el usuario decide cuándo hacerlo público.
- La versión pública no muestra correo ni teléfono; el contacto se hace a través de la app.
- Cuando se aprueba un proyecto nuevo, el portafolio se actualiza sin intervención del usuario.

**Flujo principal**

1. C8 notifica la aprobación de un proyecto.
2. El servicio reúne proyectos, habilidades y, si existen, repositorios conectados.
3. Arma la página con proyectos verificados primero y luego los declarados.
4. El usuario revisa el resultado y elige visibilidad privada o pública.
5. El servicio publica la página y genera el PDF.

**Requisitos no funcionales**

- **Rendimiento:** la página pública carga en menos de 2 s; el PDF se genera en menos de 10 s.
- **Seguridad:** solo el dueño puede editar el portafolio o cambiar su visibilidad; la conexión con GitHub usa OAuth con permisos de solo lectura.
- **Accesibilidad:** la página usa HTML semántico, las imágenes tienen texto alternativo y cumple WCAG 2.1 nivel AA.

**Criterios de aceptación**

- Dado un usuario con un proyecto recién aprobado, cuando abre su portafolio, entonces el proyecto aparece con la etiqueta "verificado" sin que él lo agregue.
- Dado un proyecto agregado a mano, cuando se muestra en el portafolio, entonces lleva la etiqueta "declarado".
- Dado un portafolio privado, cuando un visitante sin sesión abre su URL, entonces ve un mensaje de acceso no disponible y ningún dato del usuario.
- Dado un portafolio público, cuando un visitante lo abre, entonces no ve el correo ni el teléfono del usuario.

---

## Especificación C14: Servicio de matching de vacantes

**Tipo:** Servicio · **Módulo:** M5 Vacantes y postulaciones · **Historias:** HU-05

**Propósito:** mostrar al usuario solo vacantes de roles con baja o media exposición a la IA que encajen con su perfil, y registrar sus postulaciones.

**Entradas**

- Perfil y habilidades actuales del usuario (C3), incluidas las ganadas en la ruta (C8).
- Rol destino, ubicación y modalidad deseada.
- Vacantes almacenadas en la BD de vacantes (C16), cargadas por la Integración con portales de empleo (C15).
- Nivel de exposición de cada rol desde el catálogo (C6).

**Salidas**

- Lista de vacantes ordenada por compatibilidad, cada una con su porcentaje de compatibilidad y nivel de exposición.
- Postulación registrada en C16 con estado "enviada" y el resumen del portafolio (C10) adjunto.

**Reglas de negocio**

- Solo se muestran vacantes de roles con exposición Baja o Media según el catálogo.
- Solo se muestran vacantes de nivel junior o intermedio.
- Una vacante aparece como "recomendada" si su compatibilidad es de 60% o más.
- Las vacantes con más de 30 días de publicadas o marcadas como cerradas por la fuente no se muestran.
- Un usuario no puede postularse dos veces a la misma vacante.
- Los datos del usuario se envían a la empresa solo cuando él confirma la postulación.

**Flujo principal**

1. El usuario abre el Buscador de vacantes (C13) o ajusta sus filtros.
2. El servicio filtra vacantes por exposición, nivel, ubicación y vigencia.
3. Calcula la compatibilidad de cada vacante con las habilidades del usuario y ordena la lista.
4. El usuario elige una vacante y confirma la postulación.
5. El servicio registra la postulación, adjunta el portafolio y actualiza el tablero (C17).

**Requisitos no funcionales**

- **Rendimiento:** la lista se devuelve en menos de 2 s en el percentil 95.
- **Actualización:** las vacantes se sincronizan con las fuentes al menos cada 24 horas.
- **Seguridad:** el perfil se comparte con la empresa solo tras confirmación del usuario, y cada envío queda registrado con fecha y hora.
- **Accesibilidad:** el porcentaje y el nivel de exposición se muestran como texto, no solo con barras o colores.

**Criterios de aceptación**

- Dada una vacante de un rol con exposición Alta, cuando el usuario busca vacantes, entonces esa vacante no aparece en los resultados.
- Dada una vacante con 65% de compatibilidad, cuando se muestra en la lista, entonces lleva la marca "recomendada".
- Dado un usuario que ya se postuló a una vacante, cuando intenta postularse otra vez, entonces el botón aparece deshabilitado con el texto "Ya te postulaste".
- Dada una vacante publicada hace 31 días, cuando corre la sincronización diaria, entonces deja de mostrarse.

---

## Especificación C17: Tablero de seguimiento de metas

**Tipo:** UI · **Módulo:** M6 Seguimiento · **Historias:** HU-06

**Propósito:** mostrar al usuario su avance frente a las tres metas del objetivo (ruta completada, 3 entrevistas en 60 días, oferta en 90 días) y permitirle actualizar el estado de sus postulaciones.

**Entradas**

- Fecha de registro del usuario, que es el día 0.
- Avance y estado de la ruta desde C8.
- Postulaciones desde C16 y los cambios de estado que el usuario registra: enviada, entrevista, oferta, rechazada.

**Salidas**

- Tres indicadores: porcentaje de ruta completada; entrevistas logradas sobre 3 y días restantes hasta el día 60; oferta sí o no y días restantes hasta el día 90.
- Lista de postulaciones con su estado.
- Alertas generadas por el Servicio de métricas y alertas (C18).

**Reglas de negocio**

- Los estados de una postulación solo avanzan en este orden: enviada, entrevista, oferta; "rechazada" puede aplicarse desde cualquier estado.
- Una entrevista cuenta para la meta solo si se registra antes del día 60.
- Si al día 45 el usuario lleva menos de 3 entrevistas, se le sugiere ampliar filtros de búsqueda o revisar su portafolio.
- Cuando el usuario registra una oferta, el tablero marca la meta de 90 días como cumplida y le pide confirmar si la aceptó.

**Flujo principal**

1. El usuario abre el tablero.
2. El sistema calcula los días transcurridos desde el registro y los tres indicadores.
3. El usuario actualiza el estado de una postulación.
4. El sistema recalcula los indicadores y envía el evento a C18.
5. Si aplica una regla de alerta, la muestra en el tablero.

**Requisitos no funcionales**

- **Rendimiento:** el tablero carga en menos de 2 s y los indicadores se actualizan en menos de 1 s tras un cambio.
- **Seguridad:** el tablero solo es visible para su dueño; las métricas agregadas del producto se calculan sin datos que identifiquen al usuario.
- **Accesibilidad:** cada indicador muestra texto y número, no solo color; el diseño funciona desde pantallas de 360 px de ancho.

**Criterios de aceptación**

- Dado un usuario en el día 30 con 1 entrevista, cuando abre el tablero, entonces ve "1 de 3 entrevistas" y "30 días restantes".
- Dada una postulación en estado "enviada", cuando el usuario intenta marcarla como "oferta" sin pasar por "entrevista", entonces el sistema no lo permite e indica el paso faltante.
- Dado un usuario en el día 45 con 2 entrevistas, cuando abre el tablero, entonces ve la alerta con sugerencias para ampliar su búsqueda.
- Dado que el usuario registra una oferta el día 80, cuando se recalculan los indicadores, entonces la meta de 90 días aparece como cumplida.

---

## Trazabilidad

Las 6 historias quedan cubiertas por al menos un componente especificado, y cada una tiene 4 criterios de aceptación vinculados.

| Historia | Componente especificado | Componentes de apoyo | Criterios | Meta del objetivo que mide |
| --- | --- | --- | --- | --- |
| HU-01 | C1 Formulario de registro y perfil | C2, C3 | 4 | Habilita el flujo |
| HU-02 | C5 Motor de diagnóstico | C4, C6 | 4 | Elección de rol de baja exposición |
| HU-03 | C8 Servicio de rutas de reskilling | C7, C9 | 4 | 70% de rutas completadas |
| HU-04 | C10 Generador de portafolio | C11, C12 | 4 | Evidencia para la oferta |
| HU-05 | C14 Servicio de matching de vacantes | C13, C15, C16 | 4 | 3 entrevistas en 60 días |
| HU-06 | C17 Tablero de seguimiento de metas | C18 | 4 | Medición de las tres metas |
