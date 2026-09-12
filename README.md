# Lienzo Vivo — demo

Espejo público de una sola página, para poder medirla en un teléfono real.

**[Abrir la demo](https://rodolfo710314-art.github.io/lienzo-vivo-demo/)**

Toma los 33 puntos del esqueleto que entrega MediaPipe y los rasteriza como
una malla de glifos. No toca un solo píxel del vídeo, solo la geometría, así
que el coste depende del tamaño de la silueta en pantalla y no de la
resolución de la cámara.

## Por qué existe este repo

La página necesita una URL HTTPS sin restricciones de origen. Medido en un
Galaxy A34 5G, el sandbox donde se probó primero permite importar el módulo de
MediaPipe desde jsDelivr pero **bloquea los `fetch` que ese módulo hace
después** para su WASM y su modelo, y además no cede la cámara al iframe.
Ninguna de las tres cosas se puede suponer: hubo que medirlas.

Este espejo contiene únicamente `index.html`. El desarrollo, las pruebas y el
historial viven en otra parte.

## Qué hace y qué no

**El vídeo no sale del teléfono.** Se procesa entero en la página: los 33
puntos se calculan en el aparato y de ahí no pasan. No se graba, no se sube,
no se guarda.

**El motor y el modelo sí se descargan de un CDN** — jsDelivr y
`storage.googleapis.com`, unos 8.9 MB la primera vez. Eso revela tu IP a esos
dos terceros, y sin cobertura la página no arranca. Es la razón por la que
esto es una demo y no algo listo para producción: resolverlo significa alojar
el fileset WASM y el `.task` en el mismo origen.

Y ojo con el «primera vez»: el modelo se sirve con `cache-control:
max-age=3600`. Una hora, no para siempre.

## Controles

`?delegate=CPU` en la URL fuerza el delegado CPU. El defecto es **GPU**, y esta
vez por medición hecha aquí: en un Galaxy A34 5G, **49.0 ms con GPU contra 73.9
con CPU**, un 34 % más rápido. Antes el defecto era CPU, justificado solo con
que «es el defecto de la librería». Los límites de esa medición: una toma por
delegado, sin A-B-A, sin control térmico, un aparato. `?delegate=CPU` está para
repetirla.

La línea inferior muestra en vivo: delegado, tamaño de celda, DPR, `techo→real`
en Hz (el techo lo fija el gobernador térmico; el real, el movimiento de la
persona), milisegundos de red y de pintado, velocidad de la pose en torsos por
segundo, ocupación (fracción del tiempo de reloj calculando), glifos dibujados y
escalón de degradación.

**El botón «Medir 20 s»** toma una ventana y entrega un informe copiable con
medianas y p95. Es la razón de ser de este espejo: pegar ese texto es la única
forma de que las cifras del teléfono lleguen a algún sitio.

## Lo que no está medido

Ya hay una vuelta medida en un Galaxy A34 5G: red 49.0 ms (GPU), pintar 0.2 ms,
ocupación mediana 39-44 %, escalón final 5/5, el teléfono frío. De ahí salieron
dos conclusiones incómodas: que el delegado GPU gana, y que optimizar el pintado
—que se aceleró entre 56 y 197 veces— atacaba el 0.4 % del coste.

Lo que sigue **sin medir** es el control de ritmo por movimiento de la pose con
una persona de verdad delante. Su aritmética está cubierta por 25 tests y 21
mutaciones, y la página entera arranca y corre en Chromium, pero una cámara
sintética no produce pose: el camino de la velocidad solo se ejercita con
alguien moviéndose. Los umbrales `QUIETO_TPS = 0.08` y `MOVIDO_TPS = 0.50`, los
del gobernador (0.65 / 0.40) y el borde de cápsula (0.45) son valores de partida
elegidos a ojo. Las cifras del rasterizador son de V8 en un contenedor Linux.
