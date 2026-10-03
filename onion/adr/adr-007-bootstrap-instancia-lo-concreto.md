# ADR-007 — Solo `Bootstrap` instancia lo concreto

**Estado:** Aceptada · **Fecha:** 2026-10-03

## Contexto

Un equipo de seis personas, seis repositorios y cuatro capas. El error de integración más probable
es que cada quien resuelva el **ensamblado** por su cuenta: uno registra los bindings en un
`AppServiceProvider`, otro en `config/services.php`, otro dentro de un controlador, otro en el
propio caso de uso. El sistema funciona en cada máquina y falla cuando se unen los repositorios,
porque no hay un solo lugar donde viva la composición.

En Laravel esto tiene una consecuencia técnica concreta: **cualquier clase concreta se puede
construir en cualquier parte**. El contenedor resuelve dependencias automáticamente, así que un
caso de uso puede recibir un `EloquentProductRepository` sin que nadie lo haya decidido. El
framework no impide esa dependencia; solo la hace invisible.

Lo mismo pasa en React con los proveedores de contexto: si cada componente crea su propio cliente
HTTP, las URLs base y los encabezados dejan de tener un solo origen.

## Decisión

**Un solo archivo por repositorio instancia las implementaciones concretas.**

En el backend, Laravel:

```
app/
└── Bootstrap/
    └── PortBindingsServiceProvider.php     ← el único lugar
bootstrap/
└── providers.php                            ← el único registro
```

`PortBindingsServiceProvider` es el **único** archivo autorizado a escribir
`$this->app->bind(...)` o `$this->app->singleton(...)` hacia una interfaz de
`Application\Ports`. Se registra una vez en `bootstrap/providers.php` y no se vuelve a registrar.

En el frontend, React:

```
src/
└── infrastructure/
    └── providers.ts                          ← el único lugar
```

`providers.ts` es el único archivo que devuelve las implementaciones concretas de `application/ports`.

Las reglas que acompañan a la decisión:

1. **`Presentation` no importa `Infrastructure`.** Un controlador pide un caso de uso; nunca pide
   un repositorio. Si lo hace, la regla estructural lo detecta y falla la construcción.
2. **Un caso de uso no menciona ninguna clase concreta.** Recibe interfaces por el constructor y
   eso es todo lo que sabe de ellas.
3. **Nada se resuelve «en el momento de usarlo».** Si un controlador construye un repositorio
   con `new`, la dependencia se ha saltado el ensamblado.

## Alternativas consideradas

| Alternativa | Por qué no |
|---|---|
| **Bindings repartidos en varios providers** | Cada quien resuelve su parte y no hay un lugar donde leer el ensamblado completo. El fallo aparece al integrar, no al escribir |
| **Que el contenedor resuelva todo por convención** | Funciona hasta el día en que hay dos implementaciones de una interfaz. Para entonces la elección ya está implícita en cinco archivos |
| **Un contenedor propio escrito a mano** | Duplica lo que Laravel y React ya resuelven, y añade una capa que hay que mantener sin ganar nada |
| **Inyectar la implementación concreta en el caso de uso** | El caso de uso deja de ser probable sin base de datos y la dependencia queda escondida en el constructor |

## Consecuencias

**Positivas**

- **Se puede leer el ensamblado entero en un archivo.** Quien llega nuevo sabe qué implementación
  se usa sin recorrer el código buscando `bind`.
- Cambiar Eloquent por otra cosa toca **un** archivo.
- Los casos de uso se prueban con dobles sin tocar nada.

**Negativas, declaradas**

- **Un archivo que concentra todo el ensamblado es un punto de conflicto.** Seis personas
  trabajando a la vez en el mismo repositorio se estorban; como cada quien tiene su repositorio,
  en la práctica no colisionan.
- No es posible tener dos assemblages distintos —por ejemplo, uno para pruebas y otro para
  producción— sin cambiar a mano el binding. Se acepta: las pruebas usan implementaciones en
  memoria por construcción de los casos de uso, no un contenedor distinto.
- `PortBindingsServiceProvider` no se puede probar con una prueba de estructura de dependencias,
  porque depende de todas las capas a propósito. Se revisa a mano.