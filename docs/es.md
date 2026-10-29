# Audio Engine — música, efectos de sonido y ambiente

Audio Engine da vida sonora al juego: la música sigue la escena actual, los efectos responden a las acciones y un bucle de ambiente pone el tono del lugar. Estas capas tienen controles de volumen separados, así que puedes bajar la banda sonora sin apagar los clics de la interfaz ni los sonidos del entorno.

Es un mod de motor compartido que usan el menú y los juegos. Normalmente no abres ni manejas Audio Engine directamente: el juego le aporta archivos de audio y le pide reproducirlos.

![Tres canales de reproducción: las listas de la escena alimentan Música; los sonidos de acciones, Sonidos; el bucle de ambiente de la escena, Ambiente. Cada canal tiene su volumen y su interruptor.](assets/audio-channels.png)

## Lo que oyes

- **Música:** una lista de la escena aporta las pistas. El reproductor elige otra pista cuando la actual termina y evita repetir la anterior si hay otra opción. Una lista de una sola pista la repite.
- **Sonidos:** efectos cortos como los clics de la interfaz o las acciones de combate. Los efectos de categorías distintas pueden sonar a la vez; un efecto nuevo de la misma categoría sustituye al anterior.
- **Ambiente:** el sonido de fondo de un lugar, como la atmósfera del bar de Town Pack. Solo suena un flujo de ambiente cada vez, y se repite hasta que se detiene o se sustituye.

Música y ambiente son cosas distintas aunque ambos suenen de fondo. Un lugar puede tener una banda sonora de exploración en el canal de Música y un bucle aparte de interior o exterior en Ambiente.

## Ajusta el sonido

Abre **Ajustes → Sonido** en el menú. Las tres filas son **Sonidos**, **Música** y **Ambiente**. Cada una tiene su interruptor y su deslizador de volumen. Los cambios se guardan al hacerlos.

![La pestaña de sonido de Ajustes: filas Sonidos, Música y Ambiente, cada una con su interruptor y su deslizador de volumen. Aquí la Música está bajada al 1 % mientras Sonidos y Ambiente se quedan al 100 %.](assets/sound-settings.png)

Por ejemplo, baja **Música** para oír mejor los efectos de los diálogos o de la interfaz sin tocar **Sonidos**. Apaga **Ambiente** si quieres la banda sonora sin el bucle del entorno.

El panel de ajustes lo aporta el mod Menu; Audio Engine aplica esas preferencias a la reproducción. No añade otra ventana de ajustes de sonido.

## Dónde se usa

- El frontend compartido carga Audio Engine para el menú principal y los ajustes, incluso sin ningún juego en marcha.
- Music Pack aporta el tema del menú y las listas compartidas de exploración y combate; Audio Engine las reproduce.
- En la plantilla de novela, la escena del bar pide `bar/bar_ambient`, aportado por Town Pack.
- Los sistemas de historia y de combate piden efectos o cambian el contexto musical según avanza el juego.

**Audio Engine y Music Pack sirven para cosas distintas.** Audio Engine da la reproducción; Music Pack, una banda sonora concreta y sus listas. Un autor puede usar Audio Engine con otro contenido de audio.

## Para autores de juegos: añadir una banda sonora

Añade `audio_base>=1` a la lista `depends` ya existente en el manifiesto de tu mod de contenido. Instala e incluye el mod en la colección del juego correspondiente: un mod de contenido instalado no forma parte automáticamente de todos los juegos.

```jsonc
// Añádelo a la lista de dependencias ya existente del manifiesto:
"depends": ["audio_base>=1"]
```

Coloca la música bajo `music/` y los efectos o bucles de ambiente bajo `sounds/`. Para ejemplos nuevos usa Ogg Vorbis (`.ogg`): es el formato de la banda sonora y del ambiente del bar que vienen de serie.

```filestree
my_audio_content/
├── manifest.json
├── music/
│   ├── evening_theme.ogg
│   └── playlists/
│       └── my_scene.json
└── sounds/
    └── my_room/
        └── ambience.ogg
```

Un ejemplo de `music/playlists/my_scene.json`:

```json
{
  "Scene": "my_scene",
  "Musics": [
    {
      "MusicType": "Ambient",
      "MusicNames": ["evening_theme"]
    }
  ]
}
```

`Scene` debe coincidir con el nombre lógico de la escena. Los nombres de pistas son relativos a `music/`, sin la extensión. Respeta la escritura, las mayúsculas y los espacios exactos del nombre del recurso.

Aquí, `MusicType: "Ambient"` significa **música** de exploración. No envía la pista al canal de Ambiente aparte.

## Para autores de juegos: dar ambiente a una escena

Pon `defaultAmbient` en la definición de tu escena con un recurso de sonido relativo a `sounds/`, sin su extensión:

```json
{
  "name": "my_scene",
  "defaultAmbient": "my_room/ambience"
}
```

Este fragmento ilustra los campos relacionados con el sonido; conserva los demás campos que tu escena necesite. Una escena con `defaultAmbient` vacío detiene el bucle de ambiente de la escena anterior.

Un ejemplo real es la escena `my_novel_bar` de la plantilla de novela: usa `defaultAmbient: "bar/bar_ambient"` con el audio aportado por Town Pack. Reutilizar ese recurso en tu propio mod también exige depender de su dueño, `town_pack`.

## Cambios de escena, combates y sustituciones

- Si una escena no tiene lista utilizable, Audio Engine busca la lista compartida `default`. Esto solo produce música cuando hay contenido de audio que coincida.
- Cuando existe una lista específica de la escena, no se rellena con los tipos de música que falten tomados de `default`. Incluye el bloque que necesite el contexto musical de esa escena.
- Al cambiar de escena, la pista actual puede continuar si sigue en la nueva lista y el tipo de música no cambia.
- Al entrar en combate se puede guardar la pista de exploración y su posición; al volver a la exploración se intenta restaurarlas. Es un comportamiento de reproducción en vivo, no la garantía de que cargar una partida restablezca el segundo musical exacto.
- Para la misma escena y tipo de música, un mod cargado más tarde sustituye la lista anterior. Las listas no se concatenan. Los recursos con el mismo nombre relativo también pueden ser sustituidos por un mod posterior.

Usa el bloque **Combat** para el contexto musical de combate. Añadir un nombre de bloque arbitrario no crea un contexto nuevo ni un deslizador nuevo en los ajustes.

## Si algo suena mudo

1. Revisa el interruptor y el volumen correspondientes en **Ajustes → Sonido**. Música, Sonidos y Ambiente son independientes.
2. Comprueba que Audio Engine y los mods de contenido necesarios están cargados para el juego elegido. Audio Engine no contiene las pistas de Music Pack ni el bucle del bar de Town Pack.
3. Para la música, revisa el nombre de la escena, el bloque `Ambient` o `Combat` necesario y cada nombre de pista. Una referencia en una lista no crea ni descarga un archivo de audio.
4. Para el ambiente, revisa `defaultAmbient` y el recurso bajo `sounds/`. Comprueba si hay un campo vacío en la nueva escena cuando un bucle anterior se detiene.
5. Revisa los mods posteriores que puedan sustituir un bloque de una lista o un recurso de audio. Mira el registro del juego por si faltan archivos de audio o hay errores al interpretar listas.

La guía usa la vía admitida de Ogg Vorbis. El motor también tiene una vía de carga WAV, pero el cargador actual no inspecciona los metadatos del contenedor WAV; no des por hecho que cualquier WAV exportado se comporte bien. Ante la duda, convierte el contenido nuevo a Ogg Vorbis. La fábrica de audio actual no admite MP3.
