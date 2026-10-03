# test-simple-stock-flow-docs

> **Prueba técnica · Ficha ADSO 3413974**
> Horario: de **9:00 a. m. a 3:00 p. m.** (15:00)

Este repositorio contiene el **spec** de *Simple Stock Flow*. Es el único con contenido: los otros cinco empiezan vacíos.

## Instrucciones

Cada aprendiz debe **crear el fork** de los seis repositorios del proyecto y **resolver el proyecto
con el spec planteado**.

1. Hacer fork, a su cuenta de GitHub, de cada repositorio de la tabla del final.
2. Leer el spec en [`test-simple-stock-flow-docs`](https://github.com/code-sena/test-simple-stock-flow-docs).
   Se entrega en dos versiones: `spec-python/` y `spec-.net/`.
3. Desarrollar en los forks.

## El reto se desarrolla con React y PHP (Laravel)

El spec está escrito para Python y para .NET, pero el reto **no** se hace en esos lenguajes:

| Capa | Tecnología del reto |
|---|---|
| Frontend | React |
| Backend | PHP con Laravel |

Lo que el spec define sobre el negocio —historias, criterios de aceptación, reglas, contrato de la
API, modelo de datos— se respeta. Lo que define sobre la tecnología se traduce a React y Laravel.

## La prueba no consiste en escribir el código

El propósito principal es ver la **capacidad de desempeño con SDD** (*Spec-Driven Development*,
desarrollo guiado por especificación): cómo se lee, se interpreta y se aplica una especificación
para llevarla a un stack distinto. El código es el medio, no el fin.

## Qué contiene este repositorio

| Carpeta | Contenido |
|---|---|
| [`spec-python/`](spec-python/) | El spec de Simple Stock Flow escrito para un backend Python |
| [`spec-.net/`](spec-.net/) | El mismo sistema especificado para un backend .NET |
| [`ARQUITECTURA-ONION.md`](ARQUITECTURA-ONION.md) | Nuestra traducción del spec a arquitectura **Onion** con React y Laravel |
| [`onion/adr/`](onion/adr/) | Las decisiones que esa traducción añadió: ADR-005 a ADR-010 |

Las dos versiones describen el **mismo producto** (las mismas historias, reglas de negocio y
endpoints); cambian las decisiones de tecnología. Ninguna de las dos es el stack del reto.

## Cómo se lee el spec

En cualquiera de las dos carpetas, en este orden:

1. `constitution.md` — principios innegociables.
2. `spec.md` — qué debe hacer el sistema: actores, historias, criterios de aceptación, reglas de negocio.
3. `plan.md`, `architecture.md` y `arquitectura-panoramica.md` — cómo se construye.
4. `data-model.md` y `api-contract.md` — el modelo de datos y el contrato de la API.
5. `tasks.md` — qué hay que hacer y con qué evidencia se da por hecho.
6. `adr/` — las decisiones de arquitectura, con sus alternativas y consecuencias.

## Nuestra traducción: arquitectura Onion

Las dos carpetas anteriores describen una arquitectura **hexagonal**. Eso es lo que dice
[`spec-python/architecture.md`](spec-python/architecture.md), que se titula «El hexágono del
servicio» y define `adapters/inbound/api` y `adapters/outbound/{persistence,storage,security}`.
[`spec-python/arquitectura-panoramica.md`](spec-python/arquitectura-panoramica.md) lo dice más
claro todavía en su Figura 6: **«ESTA ES EL DIAGRAMA QUE HAY QUE INVERTIR»**.

Nuestro equipo lo traduce a **Onion**, con cuatro capas —`Domain`, `Application`,
`Infrastructure`, `Presentation`— y `Bootstrap` aparte como punto de ensamblaje. Esa traducción
está en **[`ARQUITECTURA-ONION.md`](ARQUITECTURA-ONION.md)**.

Lo que cambia es la forma; lo que no cambia es el negocio:

| | El spec | Nuestra traducción |
|---|---|---|
| Forma | Hexagonal, con adaptadores | Onion, con anillos |
| Entrada por HTTP | `adapters/inbound/api` | `app/Presentation` |
| Definición de puertos | Junto al dominio | En `Application/Ports` — [ADR-006](onion/adr/adr-006-puertos-en-application.md) |
| Ensamblaje | Un composition root | `app/Bootstrap` — [ADR-005](onion/adr/adr-005-cuatro-anillos-y-bootstrap.md) |
| Historias, reglas, contrato | — | **Sin cambios.** Se respetan tal cual |

Las decisiones que la traducción añadió están en [`onion/adr/`](onion/adr/):

| ADR | Qué decide |
|---|---|
| [ADR-005](onion/adr/adr-005-cuatro-anillos-y-bootstrap.md) | Cuatro anillos, y `Bootstrap` no es un quinto anillo |
| [ADR-006](onion/adr/adr-006-puertos-en-application.md) | Los puertos viven en `Application`, no en `Domain` |
| [ADR-007](onion/adr/adr-007-bootstrap-instancia-lo-concreto.md) | Solo `Bootstrap` instancia las implementaciones concretas |
| [ADR-008](onion/adr/adr-008-importes-con-bigdecimal.md) | Los importes usan `Brick\Math\BigDecimal`, no `float` |
| [ADR-009](onion/adr/adr-009-documentacion-antes-que-codigo.md) | La documentación se mergea antes del primer commit de código |
| [ADR-010](onion/adr/adr-010-rama-corta-por-fase.md) | Una rama corta por fase, Pull request a `main` |

**Un aviso sobre la dirección de las dependencias.** Como los cuatro anillos son concéntricos, las
dependencias apuntan **hacia adentro**:

```
Infrastructure ─┐
Presentation  ─┼──► Application ──► Domain
Bootstrap     ─┘        (solo conoce interfaces)
```

Se lee al revés de como se escribiría una cadena de construcción, y ahí es donde se comete el error
más caro: si `Domain` importa de `Application`, la arquitectura queda invertida y nada encaja al
unir los repositorios. `Domain` no importa nada. `Application` solo importa `Domain`.
`Presentation` nunca importa `Infrastructure`.

## Los seis repositorios

| Repositorio | Qué va ahí |
|---|---|
| [`test-simple-stock-flow-docs`](https://github.com/code-sena/test-simple-stock-flow-docs) | El spec: `spec-python/` y `spec-.net/` |
| [`test-simple-stock-flow-api`](https://github.com/code-sena/test-simple-stock-flow-api) | Backend en PHP (Laravel) |
| [`test-simple-stock-flow-app`](https://github.com/code-sena/test-simple-stock-flow-app) | Frontend en React |
| [`test-simple-stock-flow-page`](https://github.com/code-sena/test-simple-stock-flow-page) | Sitio público estático de presentación |
| [`test-simple-stock-flow-infra`](https://github.com/code-sena/test-simple-stock-flow-infra) | Contenedores, red, volúmenes y motor de base de datos vacío |
| [`test-simple-stock-flow-tool`](https://github.com/code-sena/test-simple-stock-flow-tool) | Utilidades: sembrador de datos de demostración |
