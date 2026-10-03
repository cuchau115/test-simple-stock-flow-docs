# ADR-010 — Una rama corta por fase, Pull request a `main`

**Estado:** Aceptada · **Fecha:** 2026-10-03

## Contexto

El equipo trabaja en paralelo y cada quien tiene su repositorio. Eso reduce los conflictos de
escritura, pero deja dos problemas que hay que resolver antes de que aparezcan.

El primero es la **revisión**. Si cada quien trabaja directamente sobre `main` y hace commits
grandes, la revisión se vuelve imposible: nadie sabe qué cambió respecto a la semana anterior, y
la persona que revisa no tiene un punto de comparación. En un equipo de seis eso significa que
nadie revisa de verdad.

El segundo es la **trazabilidad**. Cuando algo falla tres semanas después, hay que poder responder
qué cambió y por qué. `git blame` ayuda, pero solo si los commits son pequeños y dicen una cosa.

También hay una decisión pendiente de confirmar con el instructor: si la revisión ocurre sobre
`main` o sobre Pull requests. **Esta decisión no depende de esa respuesta**: funciona en los dos
casos. Si el instructor revisa por Pull request, el flujo ya es ese. Si revisa sobre `main`, se
 fusiona la rama a `main` después de que alguien mire el diff, y el resultado es el mismo.

## Decisión

**Una rama corta y con nombre de fase por cada bloque de trabajo. Todo llega a `main` por Pull
request.**

El patrón de nombre es `phase/<n>-<descripcion-corta>`, y el número de fase es el del plan, de modo
que el nombre de la rama dice en qué punto del plan está el trabajo:

| Fase | Rama | Qué entra |
|---|---|---|
| Documentación | `docs/onion-decision` | `ARQUITECTURA-ONION.md`, ADR-005…010, README |
| 1 | `phase/1-domain` | Entidades y reglas de negocio del `api` |
| 2 | `phase/2-ports` | Interfaces de `Application\Ports` |
| 3 | `phase/3-usecases` | Casos de uso |
| 4 | `phase/4-infrastructure` | Eloquent, repositorios, `UnitOfWork` |
| 5 | `phase/5-presentation` | Controladores, requests, respuestas |
| 6 | `phase/6-bootstrap` | `PortBindingsServiceProvider` |
| 7 | `phase/7-delivery` | Docker, `page`, `tool`, pruebas |

Las cuatro reglas que acompañan a la decisión:

1. **El nombre de la rama va en inglés**, igual que el mensaje del commit. Es lo que lee quien
   mantiene, no quien usa el sistema (artículo XI).
2. **El mensaje del commit va en inglés** y dice **una sola cosa**. Si necesita dos líneas de
   asunto, son dos commits.
3. **Una rama, una fase.** No se mezclan dos fases en la misma rama: si el Pull request mezcla
   dominio y presentación, no se puede revisar por capas.
4. **Cada Pull request declara contra qué documento se puede comprobar.** Si una regla del
   `ARQUITECTURA-ONION.md` no se cumple, el Pull request está mal —no es cuestión de gusto— y se
   puede señalar citando el bloque.

## Alternativas consideradas

| Alternativa | Por qué no |
|---|---|
| **Trabajar directo sobre `main`** | Nada delimita un cambio: la revisión se vuelve difusa y `git blame` deja de decir qué cambió. Es la opción que más rápido produce un repositorio que nadie entiende |
| **Una rama larga por persona** | Se vuelve `main` con otro nombre, y cada merge baja veinte cosas a la vez. El diff deja de ser legible |
| **Nombres de rama en español** | El artículo XI fija en inglés lo que lee quien mantiene: identificadores, comentarios, mensajes de log, commits y ramas |
| **Una rama por repositorio sin fases** | El nombre no dice nada. Seis personas abren seis ramas y no hay forma de saber si un Pull request va antes o después de otro |

## Consecuencias

**Positivas**

- Cada Pull request se puede **leer entero**. El diff de una fase cabe en una pantalla.
- Se puede saber **en qué orden** se construyó el sistema, y por qué algo está donde está.
- Si una fase está mal, se revierte **esa** rama y no el trabajo de los otros seis.

**Negativas, declaradas**

- **Siete ramas por persona son muchas.** La ceremonia es real, y en un proyecto de este tamaño
  puede pesar más que el beneficio. Se acepta porque el costo aparece una sola vez, mientras que
  un repositorio ilegible se paga cada semana.
- Mergear una fase **puede romper la fase siguiente** si esta exigía una interfaz que aún no existe.
  Se resuelve con PR encadenados: se abre el Pull request de la fase siguiente **antes** de
  terminar el de la actual, con el trabajo sin integrar.
- Se depende de que **alguien revise**. Una rama que espera revisión bloquea la siguiente. Con seis
  personas hay que acordar quién revisa qué, y eso todavía no está escrito.