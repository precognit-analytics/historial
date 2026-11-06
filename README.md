[README.md](https://github.com/user-attachments/files/34460736/README_historial.md)
[CAMBIOS_MODELO.md](https://github.com/user-attachments/files/34460807/CAMBIOS_MODELO.md)# Versiones del modelo

Cada predicción de `registro_predicciones.json` se hizo con una versión
concreta del modelo. Este archivo dice cuál, y desde cuándo.

**Por qué importa:** si el modelo cambia y no queda registrado, el
porcentaje acumulado mezcla predicciones de modelos distintos y deja de
significar nada. Para comparar manzanas con manzanas, filtra por el campo
`generada_utc` según los rangos de abajo.

---

## v2 — desde el 6 de noviembre de 2026

Aplica a las predicciones con `generada_utc` posterior a
`2026-11-06T00:00:00Z`.

| Parámetro | Valor |
|---|---|
| `k` (pendiente de la sigmoide de reparto) | 2.80 |
| `rho` (corrección Dixon-Coles) | 0.00 |
| `lambda_base` (goles esperados por equipo) | dinámico, recalculado cada jornada |
| Peso de `forma` en la matriz de fuerza | 0.00 |

Rendimiento en validación cruzada entre ligas, 989 partidos:

| Métrica | Valor |
|---|---|
| Acierto 1X2 | 49.7% |
| Estrategia trivial | 43.3% |
| Ventaja | +6.5 pp |
| McNemar | p < 0.0001 |
| Brier Score | 0.6107 |

### Qué cambió respecto a v1 y por qué

- **`k` de 3.50 a 2.80.** El 3.50 se había calibrado sobre Serie A sola y
  era sobreajuste a esa liga: fuera de muestra daba +2.3 pp con McNemar
  p=0.344, es decir, no distinguible de la estrategia trivial.
- **`forma` de 0.20 a 0.00.** Medido sobre 989 partidos, quitarla mejora el
  modelo (Log Loss −0.0119), y el efecto se repite en las tres ligas por
  separado. Duplicaba la variable `rendimiento` añadiendo ruido.
- **`lambda_base` fijo a dinámico.** Un valor fijo no sigue a la liga: Serie
  A pasó de 2.44 goles por partido en 2025 a 3.18 en 2026, y el modelo la
  subestimaba un 41%.

### Qué se dejó de publicar en v2

- **Recomendación de Over/Under.** Acertaba el 50.0% exacto sobre 989
  partidos. Se mantiene solo la probabilidad, como referencia.
- **Recomendación de Ambos anotan.** Emitía el mismo valor en 39 de 40
  casos: no contenía información.
- **El marcador exacto como pronóstico.** Acierta 11.2% frente al 12.4% de
  pronosticar siempre 1-1. Se publica como escenario, no como pick.

---

## v1 — del 11 de septiembre al 5 de noviembre de 2026

Aplica a las predicciones con `generada_utc` anterior a
`2026-11-06T00:00:00Z`. Son **60 predicciones verificadas**.

| Parámetro | Valor |
|---|---|
| `k` | 3.50 |
| `rho` | 0.00 |
| `lambda_base` | 1.221 fijo, igual para todas las ligas |
| Peso de `forma` | 0.20 |

Resultado del historial publicado bajo v1:

| Métrica | Valor |
|---|---|
| Predicciones con resultado | 60 |
| Acierto 1X2 | 63.3% (38/60) |
| Marcador exacto | 5.0% |
| Brier Score | 0.5137 |

Ese 63.3% corresponde a los 5 partidos de mayor confianza por liga y
jornada, no a todos los que el modelo predijo. El filtro aproximadamente
duplica la ventaja aparente respecto al universo completo.

---

## Regla

Cuando cambie cualquier parámetro del modelo:

1. Se añade una versión nueva a este archivo, con la fecha y el porqué
2. Se vuelve a correr la validación cruzada y se actualiza la cifra pública
3. Se commitea **antes** de registrar la primera predicción con el modelo
   nuevo

No se cambia el modelo a mitad de jornada ni se recalibra retroactivamente.
Las predicciones ya registradas no se tocan nunca, ni siquiera cuando el
modelo que las produjo deja de usarse.

---

*Precognit | Sports Analytics*
