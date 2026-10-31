# Suavizado del perfil de tracks GPS mediante la media móvil gaussiana de Arnaud Legoux (ALMA)

_Javier Urrutia — [Mendikat](https://www.mendikat.net) — octubre de 2026_

![ALMA: un perfil más claro para disfrutar la montaña. A la izquierda, un perfil GPS con ruido; a la derecha, el mismo perfil suavizado con ALMA](mendikat-alma-elevation-analysis-image.png)

White paper sobre el método de análisis de elevaciones de Mendikat.

**[⬇️ Descargar el white paper (PDF)](https://github.com/mendikat/mendikat-alma-elevation-analysis-white-paper/raw/main/mendikat-alma-elevation-analysis-white-paper.pdf)**

- [Ver el PDF en GitHub](mendikat-alma-elevation-analysis-white-paper.pdf)
- [Fuente LaTeX](mendikat-alma-elevation-analysis-white-paper.tex)
- [Release v1.0](https://github.com/mendikat/mendikat-alma-elevation-analysis-white-paper/releases/tag/v1.0)

Para regenerar el PDF (dos pasadas, para el índice y las referencias):

```bash
pdflatex mendikat-alma-elevation-analysis-white-paper.tex
pdflatex mendikat-alma-elevation-analysis-white-paper.tex
```

## Resumen

El documento describe el método de análisis de elevaciones que Mendikat usa **desde la versión 18.6** para calcular el desnivel positivo y negativo, las altitudes extremas y si un track es circular.

### El problema

La altitud es la coordenada que peor resuelve un receptor GNSS (error vertical entre 1,5 y 2 veces el horizontal). El ruido se acumula punto a punto al sumar subidas y bajadas, y puede multiplicar el desnivel real. Como cada track procede de un aparato y de unas condiciones distintas, el método tiene que funcionar sin ajustes manuales.

### Métodos anteriores

- **Umbral fijo**: se ignoran las variaciones menores que _u_ metros (histéresis). Ningún umbral sirve para todos: si es pequeño deja pasar ruido y si es grande borra desniveles reales.
- **Histéresis con umbral adaptativo** (hasta la v18.6): se prueban umbrales de 2 a 36 m hasta que el desnivel se estabiliza (diferencia < 3 m). Se adapta a cada track, pero la regla de parada no tiene por qué coincidir con el fin del ruido y el resultado es discontinuo: un pequeño cambio en el track puede mover el desnivel decenas de metros.

### Método actual: media móvil gaussiana (ALMA)

Tras repasar las medias móviles del análisis técnico (SMA, WMA, EMA), se adopta la ALMA, cuyos pesos siguen una curva de Gauss. En Mendikat se aplica en cinco pasos:

1. **Serie de altitudes** del segmento, descartando los puntos sin altitud.
2. **Ventana en metros**: W = 100 m de recorrido, convertida a puntos con la separación media entre puntos en movimiento (tramos > 1 m). La ventana queda entre 1 y 100 puntos.
3. **Prolongación de los extremos** repitiendo el primer y el último valor, para no perder el desnivel del principio y del final.
4. **Suavizado gaussiano centrado** (`offset = 0,5`, `σ = 6`): filtro simétrico, sin desfase, que no desplaza cimas ni collados. La desviación típica equivale a unos 16,7 m de recorrido, sea cual sea la densidad de puntos.
5. **Resultados**: el desnivel se calcula sobre el perfil suavizado; las altitudes máxima y mínima, sobre el original. Si el final está a menos de 100 m del inicio, el track se considera circular y el desnivel negativo se iguala al positivo.

### Por qué ALMA y por qué en metros

- Filtro **continuo**: sin umbrales ni reglas de parada.
- Filtro **paso bajo** gaussiano: atenúa el ruido de alta frecuencia, que es el origen del error, sin lóbulos secundarios.
- **Sin desfase**, al ser centrado (posible porque el perfil se conoce entero).
- Coste **O(n·k)** con k ≤ 100 y un único parámetro con significado físico: la anchura de la ventana.
- Una ventana en número de puntos aplica un suavizado muy distinto según el aparato: en tracks con menos de 20 puntos/km, una ventana de 9 puntos elimina un 20 % del desnivel real.

Como contrapartida, el suavizado redondea ligeramente cimas y collados agudos.

### Resultados (octubre de 2026)

Evaluado sobre los 1000 últimos tracks publicados y otros 1000 al azar con más de 500 puntos, frente a la histéresis adaptativa:

- Con **100 m** de ventana, diferencia mediana de **+0,1 %** y el **88 %** de los tracks dentro de **±10 %**, sin depender de la densidad de puntos.
- En tracks limpios obtiene algo menos de desnivel (**−1,5 %**), porque redondea cimas; en tracks ruidosos, algo más (**+4 %**), porque la histéresis usaba umbrales grandes que borraban desniveles reales.

### Apéndices

Implementaciones de referencia del algoritmo en **Python, Java, PHP y Rust**.
