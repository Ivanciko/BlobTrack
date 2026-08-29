# BlobTrack

**BlobTrack** es una aplicación web que detecta y sigue "blobs" (manchas en movimiento) en un vídeo o en la cámara en directo, al estilo del efecto de *blob tracking* que se usa en TouchDesigner para VJing, instalaciones interactivas o proyección mapeada. Todo el análisis ocurre en el propio navegador, sin servidor: el vídeo nunca sale de tu dispositivo.

## Qué hace

Sobre cada fotograma, la app dibuja:
- Un **cuadrado** alrededor de cada zona en movimiento detectada.
- Un **identificador numérico** por cada blob, que se mantiene mientras el blob sigue en pantalla (tracking, no solo detección puntual).
- Las **coordenadas** (x, y) normalizadas del centro de cada blob.
- Opcionalmente, una **red de líneas** conectando los distintos puntos detectados, y efectos de **glitch** o **blanco y negro**.

Funciona con dos fuentes de entrada:
- Un **vídeo subido** (MP4, MOV, WebM), que se puede reproducir, pausar y exportar ya procesado.
- La **cámara del dispositivo en directo** (frontal o trasera), con opción de grabar en tiempo real exactamente lo que se ve en pantalla.

## Cómo funciona por dentro

El método es el clásico de visión por computador para detección de movimiento por **diferencia de fondo**, simplificado para correr fluido en un navegador:

1. **Reducción de resolución**: cada fotograma se reescala a una imagen muy pequeña internamente (200 px de ancho) antes de analizarlo. Esto es lo que permite que el análisis vaya a 30+ fps incluso en un móvil — la imagen que ves en pantalla sigue siendo nítida, la versión reducida es solo para calcular dónde hay movimiento.
2. **Escala de grises**: se convierte esa imagen reducida a blanco y negro.
3. **Modelo de fondo adaptativo**: la app mantiene una imagen de referencia ("cómo se ve la escena sin movimiento") que se va actualizando lentamente fotograma a fotograma, para adaptarse a cambios de luz poco a poco.
4. **Diferencia**: se compara el fotograma actual contra ese fondo de referencia, píxel a píxel. Donde la diferencia supera un umbral (el control de "sensibilidad"), se marca como "en movimiento".
5. **Limpieza de ruido**: se descartan píxeles sueltos aislados que no tienen vecinos también marcados, para evitar falsos positivos.
6. **Componentes conexos**: los píxeles en movimiento que están pegados entre sí se agrupan en "manchas" (blobs), descartando las demasiado pequeñas (control de "tamaño mínimo").
7. **Seguimiento (tracking)**: cada blob detectado se empareja con el blob más cercano del fotograma anterior (dentro de una distancia máxima configurable), para que mantenga el mismo identificador mientras se mueve, en vez de generar un ID nuevo en cada fotograma.

Este mismo pipeline se usa tanto para la vista previa en directo como para la exportación de vídeo, así que lo que ves mientras grabas es exactamente lo que obtienes en el archivo final.

## Controles de la barra lateral

| Control | Qué hace |
|---|---|
| Sensibilidad | Umbral de diferencia para considerar un píxel "en movimiento". Más alto = detecta cambios más sutiles (y más ruido). |
| Tamaño mínimo | Filtra blobs por debajo de cierto tamaño, para ignorar detecciones diminutas. |
| Estabilidad | Distancia máxima a la que un blob puede "saltar" entre fotogramas y seguir considerándose el mismo. |
| Color / tamaño del cuadrado | Estilo del marcador que rodea cada blob. |
| Coordenadas | Muestra u oculta el texto con la posición (x, y) de cada blob, y su color. |
| Red de líneas | Dibuja líneas conectando los blobs detectados entre sí. |
| Glitch | Efecto de distorsión visual sincronizado con el movimiento detectado. |
| Blanco y negro | Convierte la imagen base a monocromo antes de superponer el tracking. |

## Cámara en directo

Al usar la cámara (en vez de un vídeo subido):
- Se puede alternar entre cámara frontal y trasera.
- La exportación fotograma-a-fotograma no aplica (no hay un archivo que recorrer), así que se sustituye por una **grabación en tiempo real** del propio lienzo ya procesado — con todos los efectos activos — usando `MediaRecorder`. El formato de salida se elige automáticamente según lo que soporte el navegador (MP4 en Safari/iOS, WebM en Chrome/Firefox/Android).
- Requiere HTTPS (o `localhost`): es una restricción de seguridad de los navegadores para el acceso a la cámara, no de la app en sí.

## Instalable como app (PWA)

La aplicación puede instalarse en el móvil como si fuera una app nativa (icono propio, pantalla completa, funciona offline tras la primera carga), gracias al `manifest.json` y al `sw.js` incluidos. En Android, Chrome ofrece "Instalar app" directamente; en iOS, desde Safari hay que usar el botón compartir → "Añadir a pantalla de inicio".

## Privacidad

Ningún vídeo, imagen o fotograma se envía a ningún servidor. Todo el procesamiento (detección, tracking, renderizado y grabación) ocurre localmente en el navegador del dispositivo.
