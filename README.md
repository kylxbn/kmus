## kmus - Kyle's Music Manager

`kmus` syncs a list of music files from my music archive to my portable music player, much like how iTunes could have done it with an iPod, but this time, you're in control of everything.

### Restrictions

kmus applies some restrictions to the music that it syncs.

- Source files must have replay gain tags
- Source albums must have album art image (such as `album.jpg` or `cover.jpg`)
    - Recognized filenames are configurable via the `db` file config
- Strictly adheres to a `Genre` - `Artist` - `Album` folder hierarchy
    - Maybe you can override this somehow by writing full paths in `iprefix` keys but it'd be awkward and definitely a hassle

### Features

- Configurable encoding settings (format and options)
    - There is an "alternate" encoding setting so
    - Will copy replaygain values without modifying the audio but can also apply replaygain values when you want it to
    - Uses `ffmpeg` so it can read and write a plethora of codecs
- Keeps music tags but discards embedded album art. Avoid duplication of data when you can save it as one single album art image file.
- Album art image is automatically scaled to 1024x1024 when needed, and copied as Q90 JPEG.
    - Image format, resolution, and quality configurable via the `db` file
- Will not encode and copy a file if the origin file has not changed since last encoding (also applies to album art)
- The database is a simple JSON file so you can edit it easily. No proprietary data storage here.
- Will automatically remove files and folders in the destination folder that is not in the JSON database.
    - This will of course not remove album art
    - This will ignore folders in `oroot` that are in the `ignore_root` config setting because sometimes you just have to have those.
- Will accept both video and audio files.
    - Will convert the video file to audio only in case you do feed it a vidoe file.

### Usage

```bash
kmus [db] [--alt]
```

* **db** - the JSON database file. Doesn't have to be named `db`, you can call it whatever you want.
* **--alt** - in case you want to use an alternate `oroot`, `codec`, `rg_preamp`, and `extension` config, use this

### Example `db` file

```json
{
  "config": {
    "iroot": "/media/K_MUSIC/PCM",
    "oroot": "/home/kylxbn/Music/Portable",
    "codec": "-c:a libopus -b:a 190k -apply_phase_inv 0",
    "rg_preamp": 0.0,
    "extension": "opus",
    "alt": {
      "oroot": "/media/FiiO M7/Music",
      "codec": "-c:a libvorbis -aq 5 -af volume=volume=|RG_TRACK|dB",
      "rg_preamp": 5.0,
      "extension": "ogg"
    },
    "art": {
      "codec": "JPEG",
      "extension": "jpeg",
      "quality": 90,
      "size": 1024,
      "accepted": [
        "album.jpg",
        "cover.jpg",
        "Cover.jpg",
      ]
    },
    "ignore_root": [
      ".stfolder"
    ]
  },
  "genres": [
    {
      "iprefix": "Pop",
      "artists": [
        {
          "iprefix": "Rick Astley",
          "albums": [
            {
              "iprefix": "Whenever You Need Somebody",
              "tracks": [
                "01 Never Gonna Give You Up.wv"
              ]
            }
          ]
        }
      ]
    }
  ]
}
```

It is pretty much self-explanatory.

* The `iroot` is the base path of your whole music collection. Likewise, the `oroot` is the base path of your portable music folder.
* The `ignore_root` array contains the folders that the autoremove feature should ignore when scanning the folders in your `iroot`.
* The `rg_preamp` sets the preamp of ReplayGain values but this does not modify the tag value. Instead, this sets the preamp when you apply the ReplayGain to the audio itself. (see the explanation for the `codec` config below)
* The `codec` key is special because you can use the keywords `|RG_TRACK|` and `|RG_ALBUM|` which will get replaced by the actual ReplayGain track or album gain during encode. Pass this to ffmpeg's `volume` audio filter to actually modify the audio data instead of just copying the ReplayGain tag.
* The `alt` setting replaces the `oroot`, `codec`, `rg_preamp`, and `extension` settings when the `--alt` flag is passed to the program.
* The `iprefix` key is the *input path* component. For example, the source song file in the example above is ultimately located in the path `/media/K_MUSIC/PCM/Pop/Rick Astley/Whenever You Need Somebody/01 Never Gonna Give You Up.wv`. By default, this will be transcoded to `/home/kylxbn/Music/Portable/Pop/Rick Astley/Whenever You Need Somebody/01 Never Gonna Give You Up.opus`. However, if you do need it, there is an option to specify an `oprefix` as well, when you want to customize the output path of the file. For example, if you add `"oprefix": "Meme King",` after `"iprefix": "Rick Astley",`, then the resulting path of the transcoded file will be `/home/kylxbn/Music/Portable/Pop/Meme King/Whenever You Need Somebody/01 Never Gonna Give You Up.wv`.

See? That wasn't too difficult, was it?

#### Why not just use `-af volume=replaygain=track`?

For some reason, ffmpeg won't recognize the ReplayGain tags on my file even though ffprobe shows them. I had to do a workaround.

### License

GPL-3.
