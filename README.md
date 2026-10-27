# visualtrans-linkedin

Datos del **agente de contenido LinkedIn de Visual Trans.

El agente genera cada mes el calendario editorial de LinkedIn (~65-70 posts en 5 perfiles) con un único punto de validación humana. Objetivo de negocio: **100.000 impresiones/mes**.

Este repo es **autosuficiente**: contiene los datos (lo que el agente lee y lo que produce) y la definición de los agentes (`.claude/agents/`) y del comando (`.claude/commands/pipeline-mensual.md`). Todas las rutas son relativas a la raíz de este repo.

---

## Dos formas de ejecutarse

El sistema funciona de forma **automática** (acciones programadas) y de forma **manual** (hablando con Claude Code).

### Forma automática

Dos acciones programadas, sin intervención humana salvo la validación del calendario:

| Acción | Cuándo | Qué hace |
|---|---|---|
| **Vigilancia de noticias** | Periódica (frecuencia por definir; propuesta: semanal) | El `agente-investigador` busca noticias relevantes del sector (normativa, aduanas, logística, eventos, competencia) y las **acumula** en `input/noticias/Noticias_Sector.txt`, con fecha, fuente y resumen. Así, cuando llega el momento de planificar, el material ya está reunido. |
| **Calendario del mes siguiente** | **El día 15 de cada mes** | Lanza el pipeline para el mes siguiente. Parte de todo lo acumulado en `input/`: el `INBOX`, los eventos y campañas, las noticias del sector, los pains y las voces. Genera el calendario y lo envía para validación. |

El día 15 el pipeline se detiene en la **validación humana del calendario** (Fase 2, ver más abajo). Hasta que se valida no se redacta ningún post. Tras la validación continúa solo hasta la aprobación final de los posts.

Se ejecutan como rutinas de Claude Code en la nube: vigilancia los **viernes a las 8:00** y calendario el **día 15 a las 9:00** (hora de Madrid).

> **Estado:** creadas. El calendario es la rutina "Calendario LinkedIn mensual — día 15" (abre una conversación nueva cada mes, avisa a `arivas@visualms.com` y espera las validaciones en esa conversación). Necesita los conectores de Gmail, Google Drive y Metricool, que se le añaden desde claude.ai.

### Forma manual

Se trabaja directamente con Claude Code, de dos maneras:

1. **Pedir contenido concreto.** Se abre Claude Code y se pide lo que se necesite: un post suelto, un calendario, una tanda para un perfil, o el pipeline completo con `/pipeline-mensual octubre 2026`. Sigue las mismas reglas y voces que la forma automática.
2. **Meter información.** Se le cuenta a Claude Code lo que se quiera incorporar: una noticia, un evento, una campaña, un cambio de tono, un pain nuevo. Claude Code lo escribe en `input/inbox/INBOX.md`, y el archivista lo clasifica en la siguiente ejecución. También se puede editar el `INBOX.md` directamente (ver más abajo).

---

## Cómo funciona

El pipeline mensual completo se lanza con un solo comando, a mano o desde la acción automática del día 15:

```
/pipeline-mensual octubre 2026
```

Una orquestación de subagentes, cada uno con una responsabilidad única, recorre estas fases:

| Fase | Agente | Qué hace |
|---|---|---|
| 0 | `agente-archivista` | Lee `input/inbox/INBOX.md`, clasifica cada línea y la vuelca al archivo de `input/` que le corresponde. Las entradas procesadas pasan a `INBOX_procesado.md`. |
| 1 | `agente-investigador` | Investiga tendencias, normativas y ferias del mes, apoyándose en `input/noticias/Noticias_Sector.txt`, y decide el **pain prioritario**. Escribe `briefing.md`. Es el mismo agente que alimenta la vigilancia automática de noticias. |
| 1.5 | `agente-metricas` + `agente-aprendizajes` (RETRO) | Mira los últimos 30 días publicados y escribe `retro.md`: qué funcionó, qué no y qué cambia. Lo que no funcionó entra en `No_Repetir.md`. |
| 2 | `agente-calendario` | Construye el calendario aplicando el retro, `No_Repetir.md`, los pilares y las reglas de no-solapamiento. Cada fila dice por qué. Lo entrega en slides. |
| 2 | **Validación humana** | **Única pausa del pipeline.** Se valida el calendario en el chat. Hasta entonces no se redacta nada. |
| 3 | `agente-redactor` | Redacta los posts. Se invoca 5 veces en paralelo, una por perfil, cada una con su documento de voz. |
| 3.5 | `agente-presentacion` | Genera las slides de los posts con `scripts/slides.py` (formato único de entrega). |
| 4 | `agente-validador` | Comprueba el resultado final contra las restricciones absolutas. Escribe `validacion.md`. |
| 4.5 | **Aprobación humana** | Se aprueban los posts ya validados. Al aprobar se crea `output/AAAA-MM/APROBADO.md`. |

Fuera del ciclo mensual hay una rutina que corre **el día 1 de cada mes**: `agente-metricas` recoge el rendimiento del mes anterior (`rendimiento.csv`) y `agente-aprendizajes` (analista estadístico: `scripts/analizar_rendimiento.py` + lectura del texto de los mejores y peores posts + experimentos para el mes siguiente) actualiza `input/aprendizajes/Aprendizajes_Calendario.md` (lo lee `agente-calendario`) y `input/aprendizajes/Aprendizajes_Redaccion.md` (lo lee `agente-redactor`). Son sugerencias acumulativas con nivel de confianza; nunca mandan sobre las reglas fijas ni sobre las voces.

Todo lo demás es autónomo. Las decisiones que antes requerían confirmación quedan registradas en `log-decisiones.md`, para auditar *después*, sin bloquear la ejecución.

---

## Cómo aprende (nuevo, 27-oct-2026)

El flujo completo, en diagramas, está en `docs/flujo-agente.pptx`. En resumen:

- **Cada día laborable** (`/aprendizaje-diario`): procesa el INBOX y el feedback, mira las métricas
  de los últimos 30 días y actualiza `input/aprendizajes/Diario.md` (lo que funciona ahora y los
  temas ya contados) y `input/aprendizajes/No_Repetir.md` (lo que no se vuelve a hacer).
- **Antes de cada calendario** (de cualquier rango): retro obligatorio (`retro.md`). Lo que no
  funcionó no se reutiliza; cada fila dice de qué aprendizaje sale.
- **Cuando el usuario dice que algo no le gusta**: entra en `No_Repetir.md` en ese momento.
- **El día 1**: análisis estadístico mensual (`Aprendizajes_Calendario.md` y `Aprendizajes_Redaccion.md`).
- **Entrega**: siempre slides con `scripts/slides.py`, con la diapositiva "Qué aprendimos y qué cambia".

## Estructura del repo

```
input/                              ← lo que LEE la IA
├── inbox/
│   ├── INBOX.md                    ← único archivo que se edita a mano
│   └── INBOX_procesado.md          ← histórico de entradas ya incorporadas (nunca se borra)
├── empresa/
│   ├── Contexto_Visual_Trans.txt   ← productos, propuesta de valor, argumentario
│   └── Pains_Unificados.txt        ← pains numerados con descripción completa
├── eventos/
│   └── Eventos_Campañas.txt        ← ferias, webinars, lanzamientos, normativas con fecha, campañas
├── aprendizajes/
│   ├── No_Repetir.md               ← prohibiciones: temas quemados, moldes y frases de relleno
│   ├── Diario.md                   ← lo que funciona ahora y temas de los últimos 30 días
│   ├── Aprendizajes_Calendario.md  ← lo aprendido del rendimiento, para el calendario
│   └── Aprendizajes_Redaccion.md   ← lo aprendido del rendimiento, para la redacción
├── historico/
│   ├── rendimiento_historico.csv   ← histórico de rendimiento (LinkedIn + Metricool), base del agente-aprendizajes
│   └── analisis_2026-10-06.md      ← primer análisis completo
├── graficas/
│   ├── prompt-noticias.txt         ← estilo de las imágenes del pilar Noticias
│   └── prompt-pains.txt            ← estilo de las imágenes del pilar Pains
├── noticias/
│   └── Noticias_Sector.txt         ← noticias del sector acumuladas por el agente-investigador
└── voces/
    ├── Voz_VT.txt                  ← empresa (página de Visual Trans)
    ├── Voz_Ceci.txt                ← Cecilio Labrada
    ├── Voz_Emma.txt                ← Emma González
    ├── Voz_Enrique.txt             ← Enrique Saa
    └── Voz_Laura.txt               ← Laura Díaz

output/                             ← lo que GENERA la IA, un directorio por mes
└── AAAA-MM/
    ├── retro.md                    ← qué funcionó, qué no y qué cambia (antes del calendario)
    ├── briefing.md                 ← investigación del mes y pain prioritario
    ├── calendario.md               ← calendario completo
    ├── posts/[perfil].md           ← posts redactados, uno por perfil
    ├── validacion.md               ← informe del agente-validador
    ├── rendimiento.csv             ← métricas reales de los posts publicados (agente-metricas)
    ├── log-decisiones.md           ← auditoría de decisiones autónomas
    └── APROBADO.md                 ← solo existe tras la aprobación humana
```

**Regla de oro:** `input/` es la fuente de verdad que lee el agente; `output/` es siempre regenerable. Nunca se escribe a mano en `output/`.

---

## Cómo meter información: `input/inbox/INBOX.md`

Es la única entrada manual de información. Hay dos formas de usarla: decirle a Claude Code lo que quieres añadir y que él lo escriba en el INBOX, o editar el archivo directamente. Puedes añadir, cuando quieras, cualquier cosa suelta: una noticia, un evento nuevo, un cambio de tono para un perfil, un pain nuevo, una campaña que se activa, un caso de éxito disponible. No hay formato obligatorio, solo la fecha delante:

```
- 2026-09-22: [texto libre]
```

En la siguiente ejecución, el archivista la clasifica y la incorpora a `empresa/`, `eventos/` o `voces/`. Las entradas ambiguas o que contradicen una regla existente **no se descartan**: quedan como "requiere revisión" en el `log-decisiones.md` del mes y el pipeline continúa.

---

## Reglas de negocio

- **Pilares:** 60 % Noticias · 30 % Pains · 10 % Casos de éxito.
- **Casos de éxito:** nunca se redactan. Solo se reserva el hueco `CASO DE ÉXITO — pendiente Adrián`.
- **No-solapamiento de pains:**
  - nunca el mismo pain el mismo día en dos perfiles;
  - nunca en días consecutivos en el mismo perfil;
  - máximo 2 apariciones por pain al mes.
- **Restricciones absolutas:**
  - nunca lenguaje de venta directa;
  - solo días laborables;
  - cierres obligatorios por perfil;
  - la voz nunca se improvisa ni se mezcla entre perfiles. Cada post sigue su `Voz_*.txt`.

---

## Después de la aprobación: las gráficas

Al aprobar los posts, el agente crea `output/AAAA-MM/APROBADO.md` y genera `output/AAAA-MM/posts-para-graficas.txt`: un bloque por post (`ID`, `FECHA`, `PERFIL`, `PILAR`, `FORMATO`, `TEXTO`), sin los casos de éxito.

Subir ese archivo hace que GitHub lance el workflow `.github/workflows/lanzar-graficas.yml`, que avisa a una **rutina de Claude** (la que tiene el conector de Magnific). La rutina trabaja en **este mismo repo**:

1. Lee el archivo de posts y separa cada post por su pilar (Noticia o Pain).
2. Pasa cada post al subagente de su pilar (`subagente-noticias`, `subagente-pains`), que genera una imagen en Magnific con el prompt de `input/graficas/`.
3. Escribe `output/AAAA-MM/graficas.csv` con el ID de cada post y el enlace a su imagen.

Para que funcione, lo único que hay que configurar son **dos secretos** de este repo (Settings → Secrets and variables → Actions), que salen del disparo API de la rutina:

- `GRAFICAS_ROUTINE_ID`: el identificador de la rutina (`trig_…`).
- `GRAFICAS_ROUTINE_TOKEN`: el token de ese disparo.

Para cambiar el estilo de las imágenes se editan `input/graficas/prompt-noticias.txt` y `prompt-pains.txt`. También se puede lanzar a mano desde Actions (*Lanzar agente de gráficas* → Run workflow, indicando el mes), o desde Claude Code con `/generar-graficas 2026-10 output/2026-10/posts-para-graficas.txt`. El formato exacto de entrada y salida está en `docs/graficas.md`.

---

## Último paso: programar en Metricool

Cuando `graficas.csv` tiene todas las imágenes, se revisan (los enlaces están en el CSV). Si están bien, se le dice a Claude Code que las dé por buenas y se ejecuta:

```
/programar-metricool 2026-10
```

El comando comprueba que cada post tiene su imagen, enseña un resumen (post, fecha y hora, perfil, imagen) y **no programa nada hasta que confirmas con un "ok"**. Entonces crea cada post en Metricool, en la marca de su perfil (Visual Trans, Emma, Cecilio, Enrique, Laura), a las 09:00 de Madrid salvo que indiques otra hora, y deja `output/AAAA-MM/metricool.csv` con el resultado. Se puede relanzar sin duplicar posts. Este paso lo lanzas tú a mano desde Claude Code: nunca corre solo.

Hoy los posts en formato carrusel o infografía llevan una sola imagen.

---

## Posts sueltos (a mano)

Además del flujo automático del mes, se pueden pedir posts puntuales con su gráfica:

```
/post-suelto Emma, sobre el DeCA y lo que implica para el sector
```

El agente deduce el pilar (Noticia o Pain) del tema, a no ser que lo indiques, redacta el post con la voz del perfil y lo revisa contra sus reglas. Te enseña el texto y espera tu OK. Antes de generar la imagen te dice cuántos créditos de Magnific hay y **te pregunta**; solo la genera con tu sí. Todo queda en `output/sueltos/AAAA-MM-DD-tema/` (`post.md` e `imagen.md`) y no se mezcla con el calendario del mes. Por defecto lo programas tú a mano; si le pides que lo programe, lo hace en Metricool, con tu confirmación.

---

## Notas para el agente

- Los agentes leen los archivos con **rutas relativas desde la raíz del repo**, p. ej. `Read input/voces/Voz_Ceci.txt`.
- Cada subagente lee lo que necesita en tiempo de ejecución. No hay copias duplicadas de las reglas: si cambia una regla, se cambia en su archivo de `input/`.
- Nunca usar `--force` ni `--no-verify`, ni reescribir historia ajena. El pipeline hace commit y push a `main` tras cada fase sin pedir confirmación. Las únicas pausas son las dos de contenido (calendario y posts).
- Los textos históricos dentro de `output/2026-10/` citan las rutas antiguas (`Voz_Laura.txt`, `Pains_Unificados.txt`…), anteriores a la reorganización en `input/`.
