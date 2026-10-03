# ADR-005 — Cuatro anillos, y `Bootstrap` no es el quinto

**Estado:** Aceptada · **Fecha:** 2026-10-03

## Contexto

El enunciado pide migrar la arquitectura **hexagonal** del spec (`spec-python/architecture.md`, «El
hexágono del servicio») a una arquitectura **Onion** de cuatro capas: `Domain`, `Application`,
`Infrastructure` y `Presentation`, con `Bootstrap` aparte como punto de ensamblaje.

Hay un desacuerdo entre compañeros sobre cómo se dibuja esa estructura. unos la describen como
`Domain → Application → Infrastructure/Presentation`; otros la describen como cuatro anillos
concéntricos. La segunda descripción es la correcta, y la diferencia no es cosmética: **la primera
se lee al revés**.

Además, el equipo trabaja en **seis repositorios** y la palabra «Infrastructure» aparece en dos
sentidos distintos: la **capa** `Infrastructure` del backend y el **repositorio**
`test-simple-stock-flow-infra`, que contiene contenedores, red, volúmenes y el motor de base de
datos. Confundirlos produce dos carpetas que se llaman igual y hacen cosas distintas.

## Decisión

**Son cuatro anillos concéntricos, y las dependencias apuntan hacia adentro:**

```
                    ┌───────────────────────────────┐
                    │  4 · Presentation             │  controladores, requests, rutas
                    │  3 · Infrastructure           │  Eloquent, repositorios, HTTP
                    │  2 · Application              │  casos de uso, puertos
                    │  1 · Domain                   │  entidades, reglas de negocio
                    └───────────────────────────────┘

    Bootstrap → conoce todos y los conecta. No está en la jerarquía.
```

1. **El número es el nivel de dependencia, no el orden de escritura.** El anillo 1 no depende de
   nadie. El anillo 4 no puede ser leído sin los tres anteriores.
2. **`Bootstrap` no es un anillo.** No contiene reglas de negocio, ni casos de uso, ni acceso a
   datos. Solo instancia implementaciones concretas y las inyecta donde se piden interfaces. Si
   alguien le encuentra lógica de negocio, es que la lógica está en el sitio equivocado.
3. **Solo dos repositorios llevan las cuatro capas:**

| Repositorio | ¿Anillos? | Por qué |
|---|---|---|
| `test-simple-stock-flow-api` | **Sí** | El backend es donde vive el dominio y la API |
| `test-simple-stock-flow-app` | **Sí** | El frontend replica la separación para no acoplar la UI al transporte |
| `test-simple-stock-flow-infra` | **No** | Es el motor vacío, la red y los volúmenes. **No es la capa `Infrastructure`** |
| `test-simple-stock-flow-tool` | **No** | Es el sembrador de datos de demostración |
| `test-simple-stock-flow-page` | **No** | Es un sitio público estático |
| `test-simple-stock-flow-docs` | **No** | Es el spec |

## Alternativas consideradas

| Alternativa | Por qué no |
|---|---|
| **Leer la cadena como `Domain → Application → Infrastructure`** | Invierte la arquitectura. `Domain` acabaría importando casos de uso, que es exactamente lo que el artículo II de la constitución prohíbe. Dos personas que lean la flecha en sentidos opuestos producen repositorios que no encajan |
| **`Bootstrap` como quinto anillo, el más externo** | Un anillo se evalúa por lo que depende, y `Bootstrap` depende de los cuatro: es un **ensamblador**, no un nivel. Puesto en la jerarquía, alguien lo trata como una capa más y le mete código |
| **Aplicar las cuatro capas a los seis repositorios** | `infra`, `tool` y `page` no tienen dominio: no hay reglas de negocio que proteger. Cuatro carpetas vacías que alguien rellena «porque toca» |
| **Una sola capa `Infrastructure` compartida entre `api` y `app`** | Obliga a los dos repos a desplegarse juntos y a versionarse al mismo ritmo. Cada quien trabaja en su repositorio |

## Consecuencias

**Positivas**

- La estructura es la misma en `api` y en `app`: quien lee un repositorio lee el otro.
- Los cinco repositorios sin anillos dicen «no aplican» en voz alta, y nadie tiene que adivinarlo.
- El nombre `infra` deja de confundirse con la capa `Infrastructure` porque ambos están
  documentados en el mismo lugar y con la diferencia explícita.

**Negativas, declaradas**

- El repositorio `infra` y la capa `Infrastructure` se distinguen por contexto, no por nombre.
  Renombrarlos está descartado por el enunciado, que fija los seis nombres de repositorio.
- `Bootstrap` no se puede probar con una prueba estructural de dependencia, porque por definición
  depende de todo. Se comprueba por revisión: si un archivo de `Bootstrap` contiene una regla de
  negocio, está mal.