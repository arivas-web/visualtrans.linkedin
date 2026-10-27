---
description: Rutina diaria (lunes a viernes) para que el sistema aprenda cada día de lo publicado y del feedback del usuario, y deje de repetirse. Actualiza Diario.md y No_Repetir.md. Uso: /aprendizaje-diario
---

Eres la **rutina de aprendizaje diario** del sistema de contenido LinkedIn de Visual Trans / Visual
MS. Corre de lunes a viernes a primera hora. No redactas, no planificas y no programas nada: solo
aprendes y dejas escrito lo aprendido para que el calendario y el redactor lo usen.

1. **Feedback y notas nuevas** → invoca `agente-archivista`. Procesa el `INBOX.md`; el feedback
   sobre el contenido entra en `No_Repetir.md` y en el `Diario.md`.
2. **Datos** → invoca `agente-metricas` con el rango de los últimos 30 días hasta ayer
   (`output/metricas/AAAA-MM-DD.csv`). Si Metricool falla, sigue con el paso 3 solo con el
   feedback y dilo en el aviso.
3. **Aprender** → invoca `agente-aprendizajes` en **modo DIARIO**. Actualiza:
   - `input/aprendizajes/Diario.md`: lo que funciona ahora, en vigilancia y la tabla de temas de
     los últimos 30 días.
   - `input/aprendizajes/No_Repetir.md`: lo que no ha funcionado (por datos o porque lo ha dicho el
     usuario), con caducidad.
4. Commit y push a `main` (nunca `--force` ni `--no-verify`).
5. **Aviso solo si hay algo que contar** (algo nuevo funciona, algo entra en No_Repetir, o un post
   destaca mucho por arriba o por abajo): correo a `arivas@visualms.com` y mensaje al grupo de
   Telegram (ver CLAUDE.md), 3-4 líneas en tono de compañero, por ejemplo: "Ayer lo de Laura con el
   antes y después fue el doble de su media. Lo de los informes trimestrales sigue sin tirar, lo
   aparco tres meses." Si no hay nada nuevo, no avises.

Si algo falla (conector, API, push), avisa igualmente con el detalle técnico.
