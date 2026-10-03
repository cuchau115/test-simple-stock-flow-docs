# ADR-009 — La documentación se mergea antes del primer commit de código

**Estado:** Aceptada · **Fecha:** 2026-10-03

## Contexto

El equipo son seis personas en seis repositorios. La constitución es explícita en el artículo XIII
sobre qué documentación lleva cada repositorio, y la regla que se repite es que un repositorio de
código **solo** lleva su README. La documentación del proyecto entero es de un repositorio, el de
documentación.

El riesgo no es escribir mal un README: es **empezar a codificar antes de que la estructura esté
acordada**. Si cada quien aplica las cuatro capas por su cuenta, las consecuencias aparecen tarde:
al unir los repositorios, un caso de uso importado de otro anillo, un nombre de carpeta distinto o
un puerto declarado donde otro no lo busca. Eso se deshace con trabajo, no con un mensaje.

Hay un segundo efecto, menos visible y más caro. Si el primer commit de cada repositorio es de
código, la revisión de la estructura se vuelve una **discusión de otra cosa**: nadie puede ver la
estructura propuesta en un solo lugar porque está repartida en seis Pull requests de código que
avanzan a la vez.

## Decisión

**El commit de documentación va primero, se mergea a `main` del fork, y solo después empieza el
código.**

**La revisión ocurre sobre el fork.** La entrega vive en el fork del equipo
(`cuchau115/test-simple-stock-flow-docs` y `cuchau115/test-simple-stock-flow-api`), y el instructor
revisa ese fork. Por eso `main` del fork es la fuente de verdad: cada fase se mergea ahí antes de
empujar la siguiente. El Pull request contra el repositorio del instructor no es requisito —no se
puede mergear sin permiso de escritura— y, si existe, es solo evidencia.

1. Este commit —`ARQUITECTURA-ONION.md`, los ADR-005 a ADR-010 y el índice del README— es el
   **primer** cambio en `test-simple-stock-flow-docs`. Se mergea a `main` del fork antes de empujar
   cualquier fase de código.
2. **Cada quien transcribe, no decide.** Los chats de `api`, `app` y `tool` leen
   `ARQUITECTURA-ONION.md` y lo aplican. Si al leerlo una persona necesita decidir algo que el
   documento no dice, **no decide**: lo escribe y lo manda al grupo. Esa respuesta es un cambio
   al documento, y el documento se actualiza antes de que ese código exista.
3. **Una decisión que no está escrita no está decidida.** Si dos personas implementan algo
   distinto de lo que dicen, gana la que está en el documento. La diferencia se corrige en el
   documento, nunca en el código.
4. **Los repositorios de código no reciben un commit de arquitectura.** No se crean carpetas
   vacías de las cuatro capas en un commit preliminar. Las carpetas aparecen cuando hay código
   dentro, en el commit que lo trae.

## Alternativas consideradas

| Alternativa | Por qué no |
|---|---|
| **Cada quien abre su Pull request de código a la vez** | Seis estructuras distintas creciendo a la vez. El conflicto aparece en la integración, cuando ya hay código escrito |
| **Un commit de esqueleto con las cuatro capas vacías** | Cuatro carpetas vacías en `infra`, `tool` y `page` no significan nada, y alguien las rellena «porque toca». Además contradice el artículo XIII sobre qué lleva cada repositorio |
| **Acordar la estructura por mensajes de Discord** | El acuerdo se pierde. Discord no es un documento: no se busca dentro de él, y nadie lo encuentra en la revisión |
| **Que cada quien escriba su propio documento** | Seis documentos que se contradicen en silencio. Es exactamente el problema que esta decisión previene |

## Consecuencias

**Positivas**

- **La revisión es real.** Quien evalúa abre un archivo, lee la estructura completa y comprueba el
  código contra ella. No tiene que reconstruirla desde seis repositorios.
- Los conflictos de nombres se descubren **antes** de que haya código que renombrar.
- El documento queda como segunda fuente de verdad **junto al spec**, no escondido en un chat: es
  lo que evita que cada quien interprete el spec a su manera.

**Negativas, declaradas**

- **Atrasa el primer commit de código.** El coste es de horas y se paga una vez.
- La decisión obliga a **escribir antes de tener la respuesta clara**. Hay preguntas abiertas —
  quién revisa, si la vista del reporte va en `page` o en `app`— que quedan escritas como
  pendientes en vez de resueltas por cada quien.
- Si el documento queda desactualizado, **toda decisión posterior queda mal**. Por eso el
  documento se actualiza en el mismo commit que el código que la aplica, no después.