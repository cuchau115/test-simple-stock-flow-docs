# ADR-006 — Los puertos viven en `Application`, no en `Domain`

**Estado:** Aceptada · **Fecha:** 2026-10-03

## Contexto

Hay dos lecturas entre compañeros sobre dónde se declara un puerto. La lectura clásica de Onion
pone las **interfaces en el anillo interior** —el dominio—, porque «el dominio define qué necesita».
La constitución de este proyecto dice otra cosa:

> **Artículo II.** Una capa **no importa** de la capa exterior. Solo el anillo interior define qué
> necesita, y esa necesidad se expresa en el anillo donde se usa.

En Onion, la diferencia no es dónde se *declara* la interfaz, sino que **el anillo interior nunca
importe a la exterior**. El caso de uso vive en `Application` y es quien recibe el puerto, así que
la interfaz se declara a su lado.

Este desacuerdo importa más de lo que parece: si el puerto queda en `Domain`, entonces cualquier
capa que quiera implementarlo tiene que **importar el dominio**, y con él arrastrar el negocio
completo. La flecha se invierte y la arquitectura deja de ser Onion.

## Decisión

**Los puertos se declaran en `Application`, en la subcarpeta `Ports/`.**

```
app/
├── Domain/                 ← cero imports hacia afuera
├── Application/
│   ├── Ports/              ← interfaces: ProductRepository, UnitOfWork, …
│   └── UseCases/           ← los casos de uso, que reciben Ports
├── Infrastructure/         ← implementa Application\Ports
└── Presentation/           ← usa Application\UseCases
```

En el frontend el criterio es el mismo, con otra ruta:

```
src/
├── domain/
├── application/
│   ├── ports/              ← interfaces (ProductApi, SessionStore)
│   └── usecases/
├── infrastructure/         ← implementa los puertos con fetch y localStorage
└── features/               ← componentes; usan useCases
```

Las tres reglas que lo sostienen:

1. **Una interfaz que solo va a usar `Infrastructure` no es un puerto.** Es una classe concreta
   con nombre de interfaz. El caso de uso no la recibe por interfaz si nadie la consume por
   interfaz.
2. **Un puerto que usa el caso de uso se declara junto al caso de uso.** No se declara «por si
   acaso» en un archivo común de puertos.
3. **`Domain` no declara puertos.** Si el dominio necesita algo del exterior, lo recibe como
   argumento de un método suyo —un `BigDecimal`, un `DateTimeImmutable`—, nunca como servicio
   inyectado.

## Alternativas consideradas

| Alternativa | Por qué no |
|---|---|
| **Puertos en `Domain`** | Invierte la dependencia: todo lo que implementa el puerto acaba importando el dominio entero. Además contradice el artículo II de la constitución, que sitúa la necesidad «donde se usa» |
| **Puertos en `Infrastructure`** | El anillo exterior define lo que el interior necesita: la flecha vuelve a apuntar hacia afuera y la prueba estructural deja de poder detectar nada |
| **Un archivo único con todos los puertos** | Mezcla puertos que usan cosas distintas. Se convierte en un cajón donde todo cabe y nada se explica |
| **Inyectar Eloquent directamente en los casos de uso** | Los casos de uso dejan de ser probables sin base de datos, y no hay forma de saber qué necesita cada uno |

## Consecuencias

**Positivas**

- **Se puede probar sin base de datos.** Un caso de uso recibe dobles de prueba de `Ports/` y se
  ejecuta en memoria.
- La prueba estructural (`deptrac`) puede exigir que `Application` **no** importe
  `Infrastructure`, y que el resultado sea una lista vacía de errores.
- Añadir un caso de uso no obliga a tocar el dominio. El dominio cambia solo cuando cambia el
  negocio, que es lo correcto.

**Negativas, declaradas**

- El número de archivos en `Application` sube: cada puerto es un archivo propio, y con cinco o seis
  casos de uso hay seis carpetas de una interfaz cada una.
- **Cambia una convención heredada del spec hexagonal.** En `spec-python/architecture.md` los
  puertos se dibujan junto al dominio. Quien siga leyendo solo el spec encontrará la diferencia y
  tendrá que venir aquí a descubrir por qué.
- No hay una prueba que demuestre que `Ports/` está completo. Un puerto que sobra no se detecta;
  uno que falta, sí — falla el caso de uso que lo necesita.