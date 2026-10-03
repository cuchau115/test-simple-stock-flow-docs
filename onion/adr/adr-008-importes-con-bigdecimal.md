# ADR-008 — Los importes usan `Brick\Math\BigDecimal`

**Estado:** Aceptada · **Fecha:** 2026-10-03

## Contexto

El spec es explícito con los importes y exige que sean **exactos**:

> **D-05.** Los importes son valores decimales exactos de dos posiciones. No se usa punto flotante
> en ningún punto del camino de un importe.

En el motor, `sale_item.unit_price` es `DECIMAL(12,2)` y los totales del reporte salen de
`SUM(quantity * unit_price)`. El dato nunca se degrada si el lenguaje lo respeta. El riesgo está
enteramente en PHP, y es un riesgo conocido: `float` es binario, así que `0.1` no existe y
`19.99 * 3` puede dar `59.970000000000006`.

Un `float` que llega a la respuesta JSON como `59.970000000000006` no es un defecto cosmético: es
una venta cuyo importe no cuadra con la base de datos, y quien lo detecta es el cliente comparando
dos números que deberían ser iguales.

PHP no trae un tipo decimal nativo. `ext-bcmath` existe pero es una extensión opcional y su API es
orientada a cadenas, no a objetos. Las alternativas razonables son: usar `float` y aceptar el
error, usar cadenas y propagar el problema a cada comparación, o usar una biblioteca de
aritmética decimal.

## Decisión

**Los importes se representan con `Brick\Math\BigDecimal`, en el dominio, con dos posiciones
decimales y redondeo `RoundingMode::HALF_UP`.**

1. **El tipo vive en `Domain`.** `BigDecimal` implementa `Stringable` y `JsonSerializable`, así que
   `Presentation` lo serializa sin convertirlo a `float` por el camino.
2. **Nada entra al dominio como `float`.** El `Request` de Laravel recibe la cadena y la convierte
   en el mapper de `Presentation`. Un `float` que llega a un caso de uso es un defecto.
3. **Más de dos decimales es un error de entrada, no una precisión que se redondea en silencio.**
   Si la petición llega con `10.999`, la respuesta es `422`. Redondear en silencio haría que el
   cliente crea que pagó `11.00` cuando él pidió otra cosa, y nadie se enteraría hasta comparar
   el reporte.
4. **`HALF_UP` es el redondeo de la moneda.** Un `HALF_EVEN` es más elegante para mathematicians y
   no es lo que espera quien calcula un total a mano.

```php
use Brick\Math\BigDecimal;
use Brick\Math\RoundingMode;

final class Money
{
    public const int SCALE = 2;

    public static function of(string $amount): self
    {
        $value = BigDecimal::of($amount);

        if ($value->scale() > self::SCALE) {
            throw new InvalidAmount($amount);
        }

        return new self($value->toScale(self::SCALE, RoundingMode::HALF_UP));
    }
}
```

En el frontend el mismo valor viaja como **cadena** (`"59.97"`), nunca como número de JavaScript:
`JSON.parse` convierte los números en `double` y se pierde la garantía justo en el último salto.

## Alternativas consideradas

| Alternativa | Por qué no |
|---|---|
| **`float` de PHP** | No representa `0.1`. El error aparece en el importe total, que es el número que el cliente compara |
| **Cadenas formateadas** (`"59.97"`) | La comparación es lexicográfica: `"9.99" > "10.00"` es verdad. Obliga a una función de comparación en cada operación aritmética |
| **`ext-bcmath`** | Es una extensión opcional: si el contenedor no la instala, el código no arranca. Su API trabaja sobre cadenas, no sobre objetos |
| **`int` en centavos** | Funciona, pero no cubre el `DECIMAL(12,2)` cuando hay otras restricciones de rango, y esconde la escala dentro del tipo |
| **Redondear en silencio los decimales de más** | El cliente cree que compró por `11.00` y el sistema cobró por `10.999`. El desajuste aparece meses después en el reporte |

## Consecuencias

**Positivas**

- El importe que sale por la respuesta es **exactamente** el que está en `DECIMAL(12,2)`.
- La suma del reporte se puede hacer en PHP con la misma garantía que hace MySQL, y comparar los
  dos resultados es una prueba real.
- Un `422` con dos decimales de más es un mensaje claro: el usuario sabe qué corregir.

**Negativas, declaradas**

- **Añade una dependencia de Composer:** `brick/math`. Hay que declararla en `composer.json` y no
  está en el esqueleto de Laravel.
- `BigDecimal` es **inmutable**, así que cada operación devuelve un objeto nuevo. Una cadena de
  operaciones de suma es legible, pero hay que recordar leer cada valor en una variable nueva.
- `json_encode` de un `BigDecimal` **no** produce un número JSON: produce la cadena `"59.97"`. Es lo
  correcto, pero es una diferencia visible contra el contrato del spec, que declara `number`. El
  contrato hay que leerlo como *importe exacto en JSON*, y el cliente formatea para mostrar.
- Hay que escribir los tests de amounts con cadenas exactas: `"0.1" + "0.2" === "0.3"` en
  `BigDecimal`, y **`false`** si el test se hace con `float`. Un test escrito con `float` pasa en
  local y falla en el motor, o al revés.