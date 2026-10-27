# INBOX

Bandeja de entrada de texto libre. Escribe aquí cualquier cosa suelta que se te
ocurra: una noticia, un evento nuevo, un cambio de tono para algún perfil, un pain
nuevo, una campaña que se activa, un caso de éxito disponible, etc.

No hay formato obligatorio. Basta con una línea con fecha delante, así:

```
- 2026-09-22: [texto libre]
```

En cada ejecución del pipeline (`/pipeline-mensual`), el **agente-archivista** lee
este archivo completo, clasifica cada entrada, la incorpora al archivo estructurado
que le corresponde (`input/empresa/Contexto_Visual_Trans.txt`, `input/empresa/Pains_Unificados.txt`,
`input/voces/Voz_[perfil].txt` o `input/eventos/Eventos_Campañas.txt`) y la mueve a `INBOX_procesado.md` con la
fecha de proceso. Las entradas ambiguas o que contradicen una regla existente no se
descartan: quedan anotadas como "requiere revisión" en el log de decisiones del mes,
pero el pipeline continúa igualmente.

Tu único trabajo manual es añadir líneas aquí abajo cuando se te ocurra algo. No hace
falta tocar ningún otro archivo ni relanzar nada aparte del pipeline habitual.

---

<!-- Añade tus líneas nuevas debajo de esta marca. El archivista las retira de aquí
cuando las procesa. -->
- 2026-10-20 (Adrián, feedback sobre el calendario 21-31 oct): no más posts sobre DeCA ni eCMR; no hablar de factura electrónica ni Verifactu. Apoyar los siguientes calendarios en lo que más ha funcionado en los últimos meses. Avisarle del enfoque antes de generar un calendario nuevo.
- 2026-10-27 (Adrián): hemos abusado de Tariff Code; está quemado y ya no genera likes ni impresiones. Dejar de basar posts en Tariff Code (producto, "Los demás", frutos secos...) en los próximos calendarios; priorizar lo que funciona (personas/equipo, normativa con dato práctico, eventos).
- 2026-10-27 (Adrián): los posts salen todos con el mismo molde y el mismo formato en todos los perfiles; se nota que venden. Variar la forma de contar cada pain y cada noticia (anécdota, storytelling con escena, diálogo, antes/después, pregunta incómoda, mito vs realidad, cuenta de la vieja, confesión) y que cada perfil use formatos distintos. Mezclar pains con noticias de normativa y de tecnología, sin repetir los mismos temas de un post a otro.
