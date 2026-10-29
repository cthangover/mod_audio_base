# Audio Engine — music, sound effects and ambience

Audio Engine makes the game sound alive: music follows the current scene, effects respond to actions, and an ambient loop can set the mood of a location. These layers have separate volume controls, so you can lower the soundtrack while keeping interface clicks and environmental sounds audible.

It is a shared engine mod used by the menu and games. You normally do not open or operate Audio Engine itself: the game supplies audio files and asks it to play them.

![Three playback channels: scene playlists feed Music; action sounds feed Sounds; a scene's ambient loop feeds Ambience. Each channel has its own volume and enabled setting.](assets/audio-channels.png)

## What you hear

- **Music:** a scene's playlist supplies tracks. The player chooses another track when one ends, avoiding the immediately previous track when another choice exists. A one-track list repeats that track.
- **Sounds:** short effects such as interface clicks or battle actions. Effects in different sound categories can overlap; a new effect in the same category replaces the previous one.
- **Ambience:** a location's background sound, such as the bar atmosphere from Town Pack. One ambient stream plays at a time and repeats until it is stopped or replaced.

Music and ambience are different even when both play in the background. A location can have an exploration soundtrack on the Music channel and a separate room or outdoor loop on Ambience.

## Adjust the sound

Open **Settings → Sound** in the menu. The three rows are **Sounds**, **Music** and **Ambience**. Each has an enabled toggle and its own volume slider. Changes are saved as you make them.

![The Sound tab of Settings: Sounds, Music and Ambience rows, each with an on/off toggle and a volume slider. Here Music is turned down to 1% while Sounds and Ambience stay at 100%.](assets/sound-settings.png)

For example, lower **Music** to hear dialogue-related or interface effects more clearly without lowering **Sounds**. Turn off **Ambience** if you want the soundtrack without the environmental loop.

The settings panel is supplied by the Menu mod; Audio Engine applies these preferences to playback. It does not add another sound-settings window.

## Where it is used

- The shared frontend loads Audio Engine for the main menu and settings, including when no game is running.
- Music Pack supplies the menu theme and shared exploration/combat playlists; Audio Engine plays them.
- In the novel template, the bar scene requests `bar/bar_ambient`, supplied by Town Pack.
- Story and battle systems request effects or change the active music context as the game progresses.

**Audio Engine and Music Pack serve different purposes.** Audio Engine provides playback. Music Pack provides a particular soundtrack and playlists. An author can use Audio Engine with other audio content instead.

## For game authors: add a soundtrack

Add `audio_base>=1` to your content mod's existing `depends` list. Install and include the mod in the intended game's collection as well: an installed content mod is not automatically part of every game.

```jsonc
// Add to the manifest's existing dependency list:
"depends": ["audio_base>=1"]
```

Place music under `music/` and sound effects or ambient loops under `sounds/`. Use Ogg Vorbis (`.ogg`) for new examples; it is the format used by the supplied soundtrack and bar ambience.

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

An example `music/playlists/my_scene.json`:

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

`Scene` must match the scene's logical name. Track names are relative to `music/`, without the file extension. Preserve the spelling, case and spaces of the actual resource name.

Here, `MusicType: "Ambient"` means exploration **music**. It does not send the track to the separate Ambience channel.

## For game authors: give a scene an ambient loop

Set `defaultAmbient` in your scene definition to a sound resource relative to `sounds/`, without its extension:

```json
{
  "name": "my_scene",
  "defaultAmbient": "my_room/ambience"
}
```

This fragment illustrates the sound-related fields; keep the other fields your scene needs. A scene with an empty `defaultAmbient` stops the previous scene's ambient loop.

A real example is the novel template's `my_novel_bar` scene: it uses `defaultAmbient: "bar/bar_ambient"` with the audio supplied by Town Pack. Reusing that asset in your own mod also requires a dependency on its owner, `town_pack`.

## Scene changes, battles and overrides

- If a scene has no usable playlist, Audio Engine looks for the shared `default` playlist. This only produces music when matching audio content is available.
- When a scene-specific playlist exists, it is not filled out with missing music types from `default`. Include the bucket needed by that scene's music context.
- On a scene change, the current track can continue if it is still in the new playlist and the music type is unchanged.
- Entering combat can save the exploration track and its position; returning to exploration attempts to restore them. This is live playback behaviour, not a guarantee that loading a saved game restores the exact musical timestamp.
- For the same scene and music type, a later-loaded mod replaces the earlier track list. The lists are not appended together. Resource files with the same relative name can also be overridden by a later-loaded mod.

Use the **Combat** bucket for the combat music context. Adding an arbitrary bucket name does not create a new context or a new settings slider.

## If something is silent

1. Check the corresponding enabled toggle and volume in **Settings → Sound**. Music, Sounds and Ambience are independent.
2. Check that Audio Engine and the required content mods are loaded for the selected game. Audio Engine does not contain Music Pack's tracks or Town Pack's bar loop.
3. For music, check the scene name, the required `Ambient` or `Combat` bucket, and every track name. A playlist reference does not create or download an audio file.
4. For ambience, check `defaultAmbient` and the resource under `sounds/`. Check for an empty field in the new scene if a previous loop stops.
5. Check later mods that may replace a playlist bucket or an audio resource. Check the game log for missing audio files or playlist parsing errors.

The guide uses the supported Ogg Vorbis path. The engine also has a WAV loading path, but the current loader does not inspect WAV container metadata; do not assume every exported WAV will behave correctly. Convert new content to Ogg Vorbis when in doubt. MP3 is not supported by the current audio factory.
