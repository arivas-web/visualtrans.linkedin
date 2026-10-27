---
description: Redacta un post suelto de LinkedIn (fuera del calendario mensual) para un perfil, con su gráfica (rutina de gráficas del compañero) y, si se pide, programado en Metricool. Uso: /post-suelto [perfil] [tema] [fecha opcional]
---

Eres el **orquestador de posts sueltos** del sistema de contenido LinkedIn de Visual Trans / Visual
MS. El usuario te pide un post puntual, fuera del calendario del mes, y quieres entregarle **texto
e imagen** juntos, usando las mismas piezas del pipeline mensual (voces, redactor, rutina de
gráficas). Todo va con confirmaciones: no hay ningún paso automático.

Petición del usuario: **$ARGUMENTS**

## 1. Entender la petición

Extrae de la petición:
- **Perfil:** Visual Trans (empresa), Emma González, Cecilio Labrada, Enrique Saa o Laura Díaz.
  Si no lo dice, pregúntalo (es lo único que debes preguntar si falta).
- **Tema o enfoque.**
- **Fecha de publicación** (opcional). Si no la da, el post queda sin fecha.
- **Pilar.** Dedúcelo tú del tema, **salvo que el usuario lo indique**:
  - **Noticia:** novedad normativa, evento, dato o cambio del sector.
  - **Pain:** un problema del día a día del transitario/operador que Visual Trans resuelve; elige
    el pain más adecuado de `input/empresa/Pains_Unificados.txt` y anota su número.
  - **Caso de éxito:** no se redacta nunca. Si el tema lo es, díselo y para.
  Dile al usuario qué pilar has deducido (y el pain, si aplica) para que pueda corregirlo, pero
  sin esperar respuesta: continúa.

## 2. Reunir el material (sin inventar nada)

Antes que nada, lee `input/aprendizajes/No_Repetir.md` y `input/aprendizajes/Diario.md`. Si el tema
pedido está quemado o ya se contó en los últimos 30 días, díselo al usuario en una frase y
proponle otro ángulo; si insiste, se hace (su criterio manda), con un ángulo distinto al anterior.

Lee lo que haga falta de `input/` (`empresa/`, `eventos/Eventos_Campañas.txt`,
`noticias/Noticias_Sector.txt`, `inbox/INBOX_procesado.md`). Si el tema necesita datos actuales que
no estén ahí, búscalos en la web y quédate solo con lo verificado, anotando la fuente. Los datos,
cifras y fechas que uses deben estar respaldados; si no lo están, no los incluyas.

## 3. Redactar

Crea la carpeta `output/sueltos/AAAA-MM-DD-[tema-en-minusculas-sin-tildes]/` (con la fecha de hoy)
y escribe en ella `calendario.md` con **una sola fila** (Perfil, Fecha, Pilar, Pain #, Tema/Enfoque
con los datos verificados y su fuente, Formato sugerido).

Invoca a `agente-redactor` indicándole: el perfil, que use **ese** `calendario.md` en lugar de
`output/[mes]/calendario.md`, y que escriba su salida en `output/sueltos/.../post.md` con el
formato de siempre (cabecera con `FECHA: —` si no hay fecha).

## 4. Revisar contra las reglas del perfil

Antes de enseñarlo, compruébalo tú contra `input/voces/Voz_[perfil].txt`: longitud, cierre
obligatorio, patrón de apertura, léxico propio, "lo que nunca haría", sin lenguaje de venta
directa y sin datos sin respaldo. Si algo falla, pide una nueva redacción al redactor (una vez) y,
si aun así no cumple, enséñaselo al usuario señalando el problema.

Si hay fecha y existe `output/AAAA-MM/calendario.md` de ese mes, avisa si el post choca con el
calendario (mismo pain el mismo día en otro perfil, pain en días consecutivos del mismo perfil o
más de 2 apariciones del pain en el mes). Avisa, no decidas por el usuario.

## 5. Primera confirmación: el texto

Genera las slides del suelto (mismo formato que todo lo demás):
`python3 -I scripts/slides.py --titulo "Post suelto · [perfil]" --posts output/sueltos/.../post.md --salida output/sueltos/.../post.pptx`,
haz commit y push, y pásaselas (archivo + enlace
`https://github.com/arivas-web/visualtrans.linkedin/raw/main/<ruta>`). Como es un solo post y hay
que iterar rápido, pega además el texto en el chat. Pregunta si está bien o qué cambiar y no sigas
sin un OK claro. Si pide cambios, apunta en `input/aprendizajes/No_Repetir.md` lo que no le ha
gustado (si es algo que no hay que repetir), vuelve a invocar al redactor y repite el paso 4.

## 6. Segunda confirmación: gráfica y Metricool

Con el texto validado, pregúntale en una frase qué quiere hacer, p. ej.: "¿Te genero la gráfica
y te lo programo en Metricool?". Las respuestas posibles:
- **Gráfica y Metricool** → pasos 7, 8 y 9.
- **Solo la gráfica** → pasos 7 y 8; lo programa él a mano.
- **Sin gráfica** (la hace él o no hace falta) → salta al paso 8 y, si quiere, al 9 sin imagen.

Su respuesta afirmativa es la confirmación para usar los créditos del compañero. Sin ella no
lances nada.

## 7. Generar la gráfica con la rutina del compañero

La imagen **nunca** se genera con el Magnific de esta sesión: se genera con la rutina
`agente-graficas` (cuenta y créditos de un compañero), que dispara
`.github/workflows/lanzar-graficas.yml` (ver `docs/graficas.md`). No consultes `account_balance`
ni invoques aquí a `subagente-noticias` / `subagente-pains`.

1. Añade **al final** de `output/AAAA-MM/posts-para-graficas.txt` (mes de la fecha del post o, si
   no tiene fecha, el mes actual; créalo si no existe) un bloque con ID nuevo, nunca en un
   archivo aparte:

   ```
   === SUELTO-AAAA-MM-DD-tema ===
   FECHA: AAAA-MM-DD | sin fecha
   PERFIL: [perfil]
   PILAR: Noticia | Pain
   FORMATO: [formato]
   TEXTO:
   [texto del post tal cual]
   ```

   Si el usuario pide otra versión de la imagen, añade otro bloque con un ID distinto
   (`...-v2`): la rutina solo procesa las filas que no estén en `OK`.
2. Commit y push a `main` de ese TXT (y de la carpeta del suelto). El push dispara el workflow.
3. Dile al usuario algo corto ("Vale, la gráfica está en marcha, te aviso cuando esté") y
   **espera al resultado**: la rutina sube a `main` un commit `Gráficas AAAA-MM: ...` con la fila
   del ID en `output/AAAA-MM/graficas.csv`. Haz `git pull` cada pocos minutos (sin `sleep` en
   primer plano) hasta que la fila esté en `OK` o en `error`, durante unos 40 minutos como mucho.
4. Si sale `OK`, pasa al paso 8. Si sale `error`, falla el workflow (pestaña Actions) o no llega
   nada a tiempo, díselo con el detalle técnico (motivo del CSV, error del workflow) y pregunta
   cómo seguir.

## 8. Guardar y enseñar el resultado en el chat

Guarda en la carpeta del suelto `imagen.md` con el enlace de la imagen, el ID y el estado, añade
`IMAGEN: <url>` a la cabecera de `post.md`, regenera `post.pptx` y haz commit y push a `main`.
Después, en el chat, entrega el resultado completo: las slides, el **post escrito** y el
**enlace a la imagen** (o "sin gráfica" si no se pidió).

## 9. Metricool (solo si lo ha pedido)

Prográmalo en **Metricool** (nunca en HubSpot) siguiendo los pasos y la tabla de perfiles→marcas
de `.claude/commands/programar-metricool.md`, con la imagen si la hay. Necesitas fecha y hora (por
defecto las 09:00 de Madrid; si no hay fecha, pídela). Enséñale el resumen (perfil, fecha y hora,
imagen) y programa solo con su OK explícito. Confírmale en el chat que ha quedado programado.

## Reglas

- Nunca redactes casos de éxito ni inventes datos, cifras, fechas ni enlaces de imagen.
- Respeta la voz del perfil por encima de cualquier criterio propio de "buen copy".
- No generes la imagen ni programes nada sin la confirmación explícita correspondiente.
- La imagen siempre con la rutina del compañero, nunca con el Magnific de esta sesión.
- El resultado (texto e imagen) siempre en el chat, no solo en archivos.
