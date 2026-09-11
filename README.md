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

`?delegate=GPU` en la URL fuerza el delegado GPU de MediaPipe. El defecto es
CPU, porque es el defecto de la librería y no hay ninguna cifra publicada que
respalde cambiarlo — **no** porque haya evidencia de que la GPU sea peor. Esa
comparación es justo para lo que sirve este espejo.

La línea inferior muestra en vivo: delegado, tamaño de celda, DPR, frecuencia
de inferencia, milisegundos de red y de pintado, ocupación (fracción del
tiempo de reloj calculando), glifos dibujados y escalón de degradación.

## Lo que no está medido

Todas las cifras de rendimiento del código son de V8 en un contenedor Linux.
Los umbrales de ocupación del gobernador (0.65 / 0.40) y el ancho del borde de
cápsula (0.45) son valores de partida elegidos a ojo. Medir eso en un aparato
real es el propósito de esta demo.
