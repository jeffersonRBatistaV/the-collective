# The Collective

Sistema de deliberación con cuatro agentes de IA para decisiones difíciles. Funciona como proyecto de Claude Code: se abre esta carpeta y se usa `/collective` (decisiones personales) o `/collective-group` (decisiones que afectan a un grupo).

Cada sesión sigue el mismo proceso:

1. El **Regente** arma un dossier con la decisión, las opciones, los hechos y las citas literales.
2. **Ronda 1:** los cuatro miembros proponen por separado y en paralelo, sin ver a los demás.
3. **Ronda 2:** cada miembro critica las propuestas de los otros.
4. El Regente escribe el veredicto: respuesta directa, costo por opción o por facción, el mejor argumento de quien pierde, el disenso que queda y un primer paso.

## Estructura

```
.claude/
  agents/                      Personalidades completas (una por archivo)
    husserl.md                 El Alba: hechos vs. supuestos. Verifica con WebSearch.
    derrida.md                 El Ocaso: el marco de la pregunta y la opción descartada.
    lacan.md                   La Noche: el deseo detrás de la decisión. Lee el historial.
    berkeley.md                El Día: las miradas de los demás. Busca testimonios con WebSearch.
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

`collective-group` no repite a los miembros: remite a la sección de miembros de `collective/SKILL.md` y a la sección "Group mode" de cada agente. Para que funcione hacen falta los seis archivos.

## Privacidad

`.claude/collective/` está en `.gitignore`. Ahí se guardan dossiers, transcripciones con citas, lecturas de Lacan sobre personas y facciones, y documentos internos. No se sube a ningún remoto.

En las sesiones de grupo hay dos veredictos: `verdict.md` es interno y `verdict_shareable.md` va sin las lecturas sobre personas, para que pueda circular.

## Portarlo a otra plataforma

La lógica está en markdown, así que se puede llevar a una app que use la API de Claude:

| En Claude Code | En la plataforma |
|---|---|
| Cuerpo de cada `agents/*.md` (sin frontmatter) | *System prompt* de la llamada de ese miembro |
| Fases de `collective-group/SKILL.md` | Orquestador en código: 4 llamadas en paralelo (ronda 1), 4 en paralelo (ronda 2), 1 de síntesis |
| Plantilla del veredicto y reglas del Regente | *System prompt* de la llamada de síntesis |
| `Read`/`Glob`/`Grep` de los agentes | Dossier y rondas inyectados en el mensaje; biblioteca e historial por RAG |
| `WebSearch` (Husserl, Berkeley) | Herramienta de búsqueda web de la API |
| Carpetas de `.claude/collective/` | Tablas en la BD, con control de acceso por rol |

Antes de cargar datos reales hay que definir la retención de datos con el proveedor, el cifrado y el cumplimiento de la ley de protección de datos que aplique (en República Dominicana, la Ley 172-13).
