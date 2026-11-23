# The Collective

Sistema de deliberación con cuatro agentes de IA para decisiones difíciles. Funciona como proyecto de Claude Code: se abre esta carpeta y se usa `/collective` (decisiones personales) o `/collective-group` (decisiones que afectan a un grupo). Para decisiones más ligeras, ambas skills aceptan `--lite`.

Cada sesión sigue el mismo proceso:

1. El **Regente** arma un dossier con la decisión, las opciones, los hechos y las citas literales.
2. **Ronda 1:** los cuatro miembros proponen por separado y en paralelo, sin ver a los demás.
3. **Ronda 2:** cada miembro critica las propuestas de los otros (se omite con `--lite`).
4. El Regente guarda el veredicto completo en `verdict.md` y muestra solo `answer.md`: respuesta directa, costo por opción o por facción y un primer paso. El debate íntegro queda en `round1.md` y `round2.md`.

## Estructura

```
.claude/
  agents/                      Personalidades completas (una por archivo)
    husserl.md                 El Alba: hechos vs. supuestos. Busca con WebSearch y abre con WebFetch.
    derrida.md                 El Ocaso: el marco de la pregunta y la opción descartada.
    lacan.md                   La Noche: el deseo detrás de la decisión. Lee el historial.
    berkeley.md                El Día: las miradas de los demás. Busca testimonios con WebSearch y WebFetch.
  skills/
    collective/SKILL.md        Orquestador individual. Incluye "The members of the
                               Collective", el resumen de lentes y sesgos que usa el Regente.
    collective-group/SKILL.md  Orquestador de grupo: modos líder/asamblea, archivo por
                               grupo, seudonimización, veredicto compartible.
  collective/                  Archivo de sesiones (local, fuera de git)
    sessions/                  Sesiones de /collective
    groups/<grupo>/            Sesiones de /collective-group y legend.md de facciones
    groups/_rehearsals/        Ensayos, separados de la historia real
```

`collective-group` no repite a los miembros: remite a la sección de miembros de `collective/SKILL.md` y a la sección "Group mode" de cada agente. Para que funcione hacen falta los seis archivos. Si falta cualquiera de los cuatro agentes, la skill se detiene y nombra el archivo ausente en lugar de improvisar al miembro.

## Instalación

1. Clona este repo dentro de tu proyecto: `git clone https://github.com/jeffersonRBatistaV/the-collective.git`, o copia la carpeta completa.
2. Comprueba que estén los seis archivos: los cuatro de `.claude/agents/` y los dos `SKILL.md` de `.claude/skills/`. Sin ellos la sesión no arranca.
3. Abre Claude Code en la raíz del proyecto y prueba con `/collective` o `/collective-group`.

## Antes de usarla

- **Costo:** una sesión completa lanza 8 subagentes (4 por ronda) más una llamada de síntesis, con hasta 300 palabras por agente y ronda. `/collective --lite` y `/collective-group --lite` hacen solo la ronda 1: 4 subagentes, sin debate, y la respuesta lo indica.
- **Idioma:** los prompts de agentes y skills están en inglés; las respuestas, los dossiers y los veredictos salen en el idioma de quien consulta.
- **Uso fuera de Claude Code:** la lógica es markdown portable (ver abajo), pero aquí las skills dependen de herramientas de Claude Code: `Agent`, `AskUserQuestion`, `Read`, `Glob`, `Grep`, `WebSearch` y `WebFetch`. En otra plataforma hay que proporcionar equivalentes.

## Privacidad

`.claude/collective/` está en `.gitignore`. Ahí se guardan dossiers, transcripciones con citas, lecturas de Lacan sobre personas y facciones, veredictos completos y documentos internos. No se sube a ningún remoto.

En las sesiones de grupo hay tres salidas: `verdict.md` es el veredicto completo e interno, `verdict_shareable.md` va sin las lecturas sobre personas para que pueda circular, y `answer.md` es la respuesta corta que se muestra, construida con la misma regla de privacidad que `verdict_shareable.md`.

## Portarlo a otra plataforma

La lógica está en markdown, así que se puede llevar a una app que use la API de Claude:

| En Claude Code | En la plataforma |
|---|---|
| Cuerpo de cada `agents/*.md` (sin frontmatter) | *System prompt* de la llamada de ese miembro |
| Fases de `collective-group/SKILL.md` | Orquestador en código: 4 llamadas en paralelo (ronda 1), 4 en paralelo (ronda 2, solo modo completo), 1 de síntesis |
| Plantilla del veredicto y reglas del Regente | *System prompt* de la llamada de síntesis |
| `Read`/`Glob`/`Grep` de los agentes | Dossier y rondas inyectados en el mensaje; biblioteca e historial por RAG |
| `WebSearch` + `WebFetch` (Husserl, Berkeley) | Herramientas de búsqueda y de lectura de páginas de la API |
| Carpetas de `.claude/collective/` | Tablas en la BD, con control de acceso por rol |

Antes de cargar datos reales hay que definir la retención de datos con el proveedor, el cifrado y el cumplimiento de la ley de protección de datos que aplique (en República Dominicana, la Ley 172-13).
