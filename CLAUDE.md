# visualtrans-linkedin

Sistema de contenido LinkedIn de Visual Trans / Visual MS. Lee `README.md` primero: explica
la estructura (`input/` lo que lee la IA, `output/AAAA-MM/` lo que genera) y las formas de
ejecución (automática y manual).

- Forma de hablar con el usuario: como un compañero de trabajo en un chat, natural y variado
  (no siempre las mismas frases). Nunca cuentes lo que haces por dentro ni nombres herramientas,
  commits, push, ramas, archivos, workflows o conectores ("voy a pushear", "compruebo el repo",
  "lanzo el workflow"...): eso no le interesa. Ejemplo de post suelto:
  - Pide un post → "Vale, me pongo con ello". Si falta algo (p. ej. el perfil), pregúntalo sin más:
    "¿Para qué perfil es?".
  - Le pasas el post → "¿Qué te parece? ¿Cambiamos algo?".
  - Le gusta → "¿Te preparo la gráfica y te lo programo?".
  - Responde → "Ok, me pongo con ello", y luego le pasas el resultado (post y enlace a la imagen).
  Solo si hay un error o un problema se explica, y entonces sí con todo el detalle técnico necesario.
- Comando del pipeline mensual: `/pipeline-mensual [mes] [año]` (`.claude/commands/pipeline-mensual.md`).
- Subagentes: `.claude/agents/` (archivista, investigador, calendario, redactor, presentacion, validador, metricas, aprendizajes). Rutina el día 1 de cada mes: `agente-metricas` recoge el rendimiento del mes anterior (`output/AAAA-MM/rendimiento.csv`) y `agente-aprendizajes` actualiza `input/aprendizajes/Aprendizajes_Calendario.md` (lo lee el calendario) y `Aprendizajes_Redaccion.md` (lo lee el redactor). Son sugerencias acumulativas, nunca por encima de las reglas fijas ni de las voces.
- Rutas siempre relativas a la raíz de este repo (`input/voces/Voz_Ceci.txt`, `output/2026-10/...`).
- Información nueva del usuario: se escribe en `input/inbox/INBOX.md` con fecha delante; el
  archivista la clasifica en la siguiente ejecución.
- Git: commit y push a `main` tras cada fase, sin `--force` ni `--no-verify`.
- Gráficas: `/generar-graficas` (`.claude/commands/generar-graficas.md`) con los subagentes `subagente-noticias` y `subagente-pains`; prompts de estilo en `input/graficas/`; detalle en `docs/graficas.md`. Nunca inventes enlaces de imagen ni marques `OK` sin imagen real.
  **Las gráficas de un mes aprobado se generan con la rutina `agente-graficas` (cuenta de Claude de un compañero, con el conector de Magnific y sus créditos), disparada por `.github/workflows/lanzar-graficas.yml`, no con los créditos de esta sesión.** Si `account_balance` de esta sesión da 0, no es un bloqueo: usa el workflow (`workflow_dispatch` con `mes`, o push a `main` de `output/AAAA-MM/posts-para-graficas.txt`). El workflow solo lee ese nombre exacto de archivo y escribe `graficas.csv`; para posts extra, añádelos a ese TXT con IDs nuevos, nunca en un archivo aparte. Antes de disparar, confirma con el usuario (el push a `main` y los créditos son de otra persona).
- Metricool: `/programar-metricool AAAA-MM` (`.claude/commands/programar-metricool.md`) programa los posts aprobados con su imagen, solo tras un "ok" explícito y siempre en Metricool, nunca en HubSpot.
- Posts sueltos: `/post-suelto [perfil] [tema]` (`.claude/commands/post-suelto.md`): se redacta el texto y se pega en el chat; cuando el usuario lo valida, se le pregunta si quiere la gráfica y que se programe en Metricool (puede decir que no a una o a las dos). La gráfica siempre con la rutina del compañero (bloque nuevo en `posts-para-graficas.txt` del mes + push a `main`), nunca con el Magnific de esta sesión. Salida en `output/sueltos/`.
- Calendario mensual: se genera solo en la fecha programada o cuando el usuario lo pide (si pide algo concreto, se hace eso).
- **Cada vez que el usuario pida un calendario** (mensual, semanal o de otro rango), además de pasarlo en el chat se crea como Google Sheets en Drive y se le envía por correo a `arivas@visualms.com`, antes de que lo valide. Si el correo no se puede enviar (conector sin sesión), se deja el Sheets creado, se le da el enlace en el chat y se avisa de que el correo no salió.
- Entrega siempre en el chat, sea un post suelto o el calendario: el texto de los posts y el enlace a cada imagen, no solo rutas de archivo.
- **Entrega en artifact o PPT cuando son más de 10 posts** (calendario con posts redactados, un lote, un rango de fechas...): se devuelve el resultado como artifact (por fecha, con filtro por perfil y botón para copiar cada texto; es privado, solo lo abre el usuario con su cuenta) o como .pptx (una diapositiva por post, en orden por fecha, con portada, como hace `agente-presentacion`, guardado en `output/` con enlace `https://github.com/arivas-web/visualtrans-linkedin-test/raw/main/<ruta>`, tras commit y push a `main`); se elige el que mejor encaje. Con 10 posts o menos, texto plano directamente en el chat.
- **Avisos de las rutinas automáticas (calendario del día 15, métricas y aprendizajes del día 1, investigación...)**: cuando terminen o encuentren algo relevante, avisan por las dos vías: correo a `arivas@visualms.com` y mensaje al grupo de Telegram "Visual Trans" (chat_id `-1003759003410`) con el webhook de Make "Claude → Telegram (respuestas LinkedIn)" (mismo POST `{"chat_id", "text"}` que usa Peter para contestar). Mensaje corto, tono de compañero, con el enlace al Sheets/resultado.
- **Vigilancia de noticias (investigación)**: además del correo y Telegram, al terminar deja un resumen corto en el chat de Marketing OS (lo más urgente primero, con el enlace si lo hay). Pedido por el usuario el 2026-10-30. Si la ejecución no trae datos de entrega al chat, avisa por correo y Telegram y dilo en el último mensaje.
