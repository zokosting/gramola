# Canciones (formato UltraStar)

Gramola lee el formato de texto **UltraStar**, abierto y muy extendido. Cada canción es una carpeta dentro de
`web/songs/` (o de la ruta indicada en `SONGS_DIR`) con un `.txt` y su audio. Para uso privado.

## Cabeceras

| Clave | Obligatoria | Uso |
|---|---|---|
| `#TITLE` / `#ARTIST` | sí | Título y artista. |
| `#BPM` | sí | Pulsos por minuto *de UltraStar*: un pulso dura `60000 / (BPM × 4)` ms. Admite decimales con coma. |
| `#GAP` | sí | Milisegundos desde el inicio del audio hasta el pulso 0. |
| `#AUDIO` o `#MP3` | sí | Fichero de audio (mp3, ogg, m4a…). |
| `#COVER` | no | Portada cuadrada. Sin portada se genera una con las iniciales. |
| `#BACKGROUND` | no | Imagen de fondo durante la canción. |
| `#VIDEO` / `#VIDEOGAP` | no | Vídeo de fondo. Gramola **no usa vídeos**: la preparación los quita. |
| `#START` | no | Segundo en el que empieza la canción. |
| `#RELATIVE:YES` | no | Formato antiguo con pulsos relativos a cada frase (soportado). |
| `#MEDLEYSTARTBEAT` / `#MEDLEYENDBEAT` | no | Tramo del estribillo para el Megamix. |

## Notas

```
: 0 3 4 Lu          normal: inicio, duración, tono, sílaba
* 96 7 9 no         dorada (vale doble)
F 120 4 0 ¡ey!      libre (no puntúa)
R 130 2 0 rap       rap
- 32                salto de frase
P1 / P2             partes de un dúo
E                   fin
```

- **Tono 0 = Do4** (MIDI 60); cada unidad es un semitono.
- **Los espacios cuentan**: `ces ` lleva el espacio al final para separar la palabra siguiente. No los quites al editar.
- Codificación UTF-8 o Windows-1252 (se detecta sola).
- **`~`** en una sílaba significa que la anterior se alarga en otra nota: la tele no lo muestra.

## Crear o ajustar canciones

- **Editores** libres de UltraStar (por ejemplo *Yass* o *UltraStar Creator*) permiten marcar sílabas y tonos sobre el audio.
- Si la letra va adelantada o retrasada respecto a la música, ajusta `#GAP` (en ms).
- `tools/demo-song.py` es un ejemplo completo de cómo generar una canción por código.

## Añadir canciones (paso a paso)

1. **Preparar** — `python3 tools/preparar-canciones.py ORIGEN DESTINO`
   - Nombres en Unicode NFC, `.txt` en UTF-8 (solo se reescriben las cabeceras; notas y letra intactas).
   - `#MP3`/`#COVER` apuntando a los ficheros que hay de verdad; elige la versión normal si viene también la `[MULTI]`.
   - Sin mp3: saca el audio del vídeo. Un «.mp3» que por dentro no es MP3 (MP2, WAV con etiqueta delante) se convierte a MP3.
   - Quita el vídeo y las referencias a fondos que no vienen.
   - Recorta el tramo del **Megamix** (`megamix.mp3` + `megamix.json`): del estribillo marcado o, si no lo hay, del bloque de
     frases repetido más largo (de 10 a 45 s; acierta en ~8 de cada 10 comparado con los marcados).
2. **Revisar duplicados** contra el servidor: por nombre de carpeta, por artista+título y por huella del audio.
3. **Igualar el volumen** — `python3 tools/normalizar.py DESTINO`: −12 LUFS (loudnorm en dos pasadas, picos a −1 dBTP) en las que
   se alejen más de 1,5 dB; guarda `volumen.json` y rehace el tramo del Megamix.
4. **Portadas** — `python3 tools/caratulas.py DESTINO SALIDA` busca en iTunes (600×600, prefiere el disco original al recopilatorio)
   y deja una página para revisarlas; `--aplicar … --excluir 3,17` las aplica como `cover-itunes.jpg` (la anterior queda en la carpeta
   y en `caratula.json`).
5. **Subir** — `tools/subir-canciones.sh DESTINO` (rsync a `web/songs/` del servidor). Las portadas llevan la fecha del fichero en su
   dirección, así que un cambio de portada se ve enseguida en la tele.

## En la tele

- Los **títulos** se muestran sin lo que va entre paréntesis o corchetes y con mayúscula al principio de cada palabra
  (salvo «a e i o u y» sueltas). Los ficheros no se tocan.
- El **avance** de la lista empieza en el minuto 1 de la canción (o hacia un tercio de la letra si es corta).
