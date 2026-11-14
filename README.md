# El diario de Eliot

Página pública de **Eliot Want To Live**, un juego de supervivencia zombi en pixel art con vista cenital, hecho en Godot por Equipo Rojo (Sistemas de Información II, UDO).

La página está escrita como el cuaderno de Eliot, el protagonista. Sirve para que alguien que todavía no ha jugado entienda de qué va el juego: qué infectados hay, cómo se comportan y cómo es el recorrido del prólogo.

## Qué contiene

La página tiene una sola vista y se recorre bajando. Las secciones van en el mismo orden que el menú:

1. **Portada.** Título, presentación y la horda animada.
2. **Nota del diario.** Introducción narrativa escrita por Eliot.
3. **Bestiario.** Fichas de los 13 tipos de infectado, con barras de vida, velocidad y daño, cualidades, debilidades, un consejo y desde cuándo aparece cada uno. Se pueden filtrar por categoría (comunes, especiales y gigantes). Al pasar el cursor por una ficha, o al tocarla en el móvil, el zombi camina y ataca con la animación del juego.
4. **Cómo funcionan.** Las seis reglas que comparten todos los infectados: detección, ruido, rodeos, fuego, aturdimiento y Rex.
5. **Tamaños.** Todos los infectados a escala real junto a Eliot.
6. **Guía de juego.** El prólogo paso a paso, con capturas del juego:
   - **El hospital:** despertar en la habitación 214, registrar cajas, conseguir el bate, el primer combate y la oleada de recepción.
   - **La ciudad y las armas:** la pistola y el mapa del policía caído, las dos zonas marcadas en el mapa, la escopeta y el subfusil del campamento militar y el AR-15 del furgón SWAT.
   - **La llave de la estación:** la puerta trabada, las patrullas calcinadas, la oleada fuerte y el final del prólogo.
   - **Cómo seguir vivo:** golpear, esquivar, lanzar la molotov, curarse, registrar y la mochila.
   - **Controles:** teclado y ratón, mando y móvil.

   Al tocar una captura se abre a mayor tamaño.
7. **Sobrevivir.** Tres reglas de supervivencia escritas como notas arrancadas del cuaderno.

## Estructura

```
el_diario_de_eliot/
├── index.html      La página completa: HTML, estilos y scripts en un solo archivo
├── img/
│   ├── *.png       Sprites de los infectados y de Eliot (de frente), logos del juego y del equipo
│   ├── anim/       Tiras con los cuadros de caminar y atacar de cada infectado
│   ├── iconos/     Iconos de "Cómo funcionan" y de las armas de la guía
│   └── guia/       Capturas de la guía de juego (WebP sin pérdida)
└── README.md
```

La página no usa frameworks ni pasos de compilación. Lo único externo son tres tipografías de Google Fonts: Special Elite, Caveat e Inter.

## Verla en local

Basta con abrir `index.html` en el navegador. Funciona también con doble clic (`file://`), porque no carga datos con `fetch`: los datos de las animaciones están escritos dentro del propio HTML.

Para probarla como quedará publicada, sírvela con cualquier servidor estático. Por ejemplo:

```
npx serve .
```

o

```
python -m http.server 8000
```

## Publicarla en Vercel

1. Sube este repositorio a GitHub.
2. En Vercel, importa el repositorio con estos ajustes:
   - **Framework Preset:** Other
   - **Build Command:** vacío
   - **Output Directory:** vacío (la raíz)
3. Cada cambio en la rama principal se publica solo.

También se puede publicar sin GitHub ejecutando `vercel` desde esta carpeta.

## Diseño y comportamiento

- **Adaptable.** La página funciona desde 360 px de ancho. Por debajo de 860 px el menú se convierte en un botón desplegable, y los pasos de la guía ponen la captura encima del texto.
- **Pixel art nítido.** Los sprites y las animaciones se dibujan con `image-rendering: pixelated` para que los píxeles no se emborronen al ampliarse.
- **Animaciones.** Las fichas del bestiario usan un único bucle de `requestAnimationFrame` para todos los canvas, y solo animan la ficha activa.
- **Accesibilidad.**
  - Las fichas y las capturas se pueden usar con el teclado (Tab, Enter, Espacio y Esc).
  - Las imágenes tienen texto alternativo descriptivo.
  - Si el sistema pide reducir el movimiento (`prefers-reduced-motion`), se desactivan las animaciones decorativas: el grano de película, la niebla, las gotas de sangre y la aparición al hacer scroll.
- **Carga diferida.** Las capturas de la guía (unos 4,4 MB en total) se cargan con `loading="lazy"`, solo cuando el visitante se acerca a ellas.

## De dónde salen los datos

Todo el contenido sale de la **versión 1.0.3** del juego:

- **Estadísticas del bestiario.** La vida, la velocidad y el daño se copiaron de las escenas de cada zombi. Están en el array `Z` del script de `index.html`. Las barras se calculan en proporción al valor más alto de cada columna.
- **Animaciones.** Las tiras de `img/anim/` se recortaron de las hojas de sprites del juego. El tamaño de cuadro, el número de cuadros y la velocidad de cada animación están en el objeto `ANIM` de `index.html`.
- **Guía de juego.** Las capturas se tomaron con el juego en marcha, recorriendo el prólogo. Los textos describen el comportamiento de la v1.0.3: dónde está cada arma, cuánto cura cada objeto y qué teclas usa cada acción.

Si el juego cambia en una versión posterior (balance, orden del prólogo, controles), hay que actualizar a mano los datos de `index.html` y, si hace falta, las imágenes. Al pie de la página se indica de qué versión salen los datos.

### Herramientas de generación

Las imágenes se generan con scripts que viven fuera de este repositorio, porque dependen de la carpeta del juego. La carpeta `_herramientas/` está excluida en `.gitignore`.

- **`_herramientas/hacer_animaciones.py`** (Python con Pillow).
  - Lee las hojas de sprites y los archivos `_frames.tres` del juego.
  - Recorta las animaciones de caminar y atacar de cada infectado a una caja común y las guarda como tiras en `img/anim/`.
  - Prepara los iconos de `img/iconos/`.
  - Si se regeneran las tiras, hay que copiar los valores nuevos al objeto `ANIM` de `index.html`.
- **`tools/captura_guia.tscn`**, dentro del proyecto del juego (v1.0.3).
  - Carga cada escena del prólogo, coloca a Eliot y guarda las capturas.
  - Necesita ventana, no funciona sin interfaz.
  - Se ejecuta por partes con `--solo=hospital`, `ciudad`, `estacion` o `mecanicas`.
  - Las capturas se convierten después a WebP y se recortan para `img/guia/`.

## Créditos

**Eliot Want To Live** es un juego de **Equipo Rojo** para la asignatura Sistemas de Información II de la UDO. El arte de los personajes, los infectados y los escenarios es del propio juego.
