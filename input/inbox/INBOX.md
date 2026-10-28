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
