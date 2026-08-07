# autodatamosh

![Vibecoded](https://img.shields.io/badge/vibecodeado-100%25-8A2BE2)

Genera animaciones con efecto **datamosh real** (a nivel de bitstream) a partir de una sola imagen. Incluye dos herramientas:

- **`simpledatamosh`** — la opción **más limpia y con menos requerimientos**: un solo efecto + un solo movimiento elegidos al azar. Solo necesita `bash`, `ffmpeg` y `ffprobe` (sin `python3`).
- **`datamosh`** — el motor completo: combina muchos efectos y variantes por sección, con más opciones de control (requiere `bash`, `ffmpeg`, `ffprobe` y `python3`).

![Ejemplo](ejemplo.gif)

## Cómo funciona

Un video normal se divide en "escenas", y cada escena arranca con un **I-frame** (imagen completa). El truco del datamosh es **quitarle el I-frame** a las escenas siguientes: sus P-frames (que solo guardan la diferencia) quedan apuntando a un frame que ya no existe, así que el decodificador usa el frame viejo y arrastra/sangra la imagen anterior sobre la nueva. Ese arrastre es el glitch.

Re-codificar con x264 **no** produce este efecto (x264 siempre predice bien y "arregla" el corte). El glitch requiere corrupción a nivel de bitstream: por eso estos scripts unen los streams comprimidos a mano.

## Requisitos

| Herramienta | `simpledatamosh` | `datamosh` |
|---|---|---|
| `bash` (4+) | ✓ | ✓ |
| `ffmpeg` + `ffprobe` | ✓ | ✓ |
| `python3` | — | ✓ |

`LC_ALL` se fuerza a `C` dentro de ambos scripts (necesario para que los decimales funcionen en los filtros de ffmpeg).

## simpledatamosh — la opción limpia

Elige **un** efecto visual al azar y **un** movimiento simple al azar, y genera un sangrado único (o ráfagas con `-n`). El I-frame se quita inline con `-bsf:v filter_units`, así que no necesita `python3` y el pipeline es mucho más corto.

```bash
./simpledatamosh IMAGEN -o salida.mp4      # todo random, solo input y output
./simpledatamosh IMAGEN -n 6 -e rotate -m zoom -o rafagas.mp4
```

Con **todos los parámetros opcionales**: si no se pasan, se eligen **al azar**. Si se especifican, se respetan.

| Opción | Descripción | Default |
|---|---|---|
| `-o, --output ARCHIVO` | Salida (`.mp4`, `.webm`, `.gif`) | `mosh.gif` |
| `-e, --effect NOMBRE` | Efecto único: `hflip`, `vflip`, `rotate`, `negate`, `hue`, `shift`, `eq` | random |
| `-m, --motion NOMBRE` | Movimiento simple: `zoom` (zoom-in lento) o `pan` (barrido) | random |
| `-n, --sections N` | N° de cortes/secciones. `1` = un solo sangrado largo; `6+` = glitch por ráfagas | random 1–8 |
| `--intro SEG` | Intro antes del primer glitch (imagen original) | `0.1` |
| `-d, --duration SEG` | Duración de cada sección | random 1–2.5 |
| `-i, --intensity N` | Intensidad del glitch (1–10, más alto = CRF más sucio) | random 3–9 |
| `-f, --fps N` | FPS de salida | `30` |
| `-s, --size WxH` | Resolución de salida | auto (1280px) |
| `-j, --jump N` | Stutter: muestra 1 de cada N frames | off |
| `--seed N` | Reproduce el mismo video (misma secuencia aleatoria) | sin seed |
| `-h, --help` | Ayuda | |

### Ejemplos

```bash
# Un solo sangrado largo, rotando hacia la derecha
./simpledatamosh imagen.png -n 1 -d 6 -e rotate -m zoom -o sangrado.mp4

# Glitch por ráfagas con desplazamiento
./simpledatamosh imagen.png -n 6 -d 1.3 -e shift -m pan -o cortes.mp4

# Reproducir un resultado que te gustó
./simpledatamosh imagen.png --seed 7 -o igual_que_antes.mp4
```

## datamosh — el motor completo

Combina muchos efectos: variantes rotadas/volteadas/desplazadas por sección + zoom pulsante in-out + barrido lateral, para que la imagen nunca quede quieta ni entera. La imagen original derecha **nunca** se muestra: la intro ya es una variante rotada.

```bash
./datamosh IMAGEN -o salida.mp4      # todo random, solo input y output
./datamosh IMAGEN -o salida.gif      # formato según extensión
```

Con **todos los parámetros opcionales**: si no se pasan, se eligen **al azar**. Si se especifican, se respetan.

| Opción | Descripción | Default |
|---|---|---|
| `-o, --output ARCHIVO` | Salida (`.mp4`, `.webm`, `.gif`) | `mosh.gif` |
| `-m, --mode MODO` | `bleed` (voltea/rota y sangra) o `melt` (derrite/desliza) | random |
| `-a, --start N` | Variante inicial de la intro (0–8): `0`=original, `1`=hflip, `2`=vflip, `3`=180°, `4`=hflip+vflip, `5`=90°cw, `6`=90°ccw, `7`=45°, `8`=-45° | random |
| `--rotate on\|off` | `off` = mostrar la imagen original derecha en todas las secciones (sin rotar/voltear) | `on` |
| `-n, --sections N` | N° de cortes/secciones que se van sangrando. `1` = un solo sangrado largo; `6+` = glitch por ráfagas | random 1–8 |
| `--intro SEG` | Intro antes del primer glitch (ya rotada) | `0.1` |
| `-d, --duration SEG` | Duración de cada sección | random 1–2.5 |
| `-z, --zoom FACTOR` | Amplitud del pulso de zoom in-out (`0` = solo paneo lateral) | random 0–0.3 |
| `-i, --intensity N` | Intensidad del glitch (1–10, más alto = CRF más sucio) | random 3–9 |
| `-f, --fps N` | FPS de salida | `30` |
| `-s, --size WxH` | Resolución de salida | auto (1280px) |
| `-j, --jump N` | Stutter: muestra 1 de cada N frames | off |
| `--seed N` | Reproduce el mismo video (misma secuencia aleatoria) | sin seed |
| `-h, --help` | Ayuda | |

### Ejemplos

```bash
# Un solo sangrado largo de 8s arrancando a 180°, intensidad alta
./datamosh imagen.png -n 1 -d 8 -a 3 -i 7 -o sangrado.mp4

# Muchos cortes rápidos (glitch por ráfagas)
./datamosh imagen.png -n 6 -d 1.3 -a 5 -i 7 -o cortes.mp4

# Glitch en gif, resolución chica
./datamosh imagen.png -s 640x348 -f 15 -o mosh.gif

# Reproducir un resultado que te gustó
./datamosh imagen.png --seed 7 -o igual_que_antes.mp4
```

## Vibecodeado

Este proyecto fue **vibecodeado**: desarrollado de punta a punta con un asistente de IA, iterando sobre prompts en lenguaje natural. El código fue generado con **opencode** usando el modelo **big-pickle** (`opencode/big-pickle`), con todo el proceso de diseño, debugging y verificación (PSNR por frame) hecho de forma conversacional.

## Notas

- La duración total aproximada es `intro + secciones × duración`, pero cada corte descarta ~10 frames (cascada de referencias rotas), así que el video queda un poco más corto de lo pedido.
- En `datamosh`, el modo `bleed` usa rotaciones/volteos y `melt` desplazamientos grandes (con algunos volteos); en `simpledatamosh` el efecto único se aplica con un parámetro que rampa por sección (los efectos binarios alternan con la paridad).
- En `simpledatamosh` la intro muestra la imagen original un instante; en `datamosh` nunca aparece derecha.
- Verificado por PSNR: la intro sale limpia (~37 dB vs. la referencia) y el tramo glitcheado se mantiene en ~6–7 dB en toda la duración (nunca se "asienta" a imagen limpia).
