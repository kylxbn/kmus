## kmus - Kyle's Music Manager

`kmus` syncs a list of music files from your music archive to your portable music player, much like how iTunes could have done it with an iPod, but this time, you're in control of everything. Yes, *everything*.

### Restrictions

`kmus` applies some restrictions to the music that it syncs.

- Source files must have ReplayGain tags
- Can be configured to require source albums to have album art image (such as `album.jpg` or `cover.jpg`)
    - Recognized filenames are configurable via the `db` file config
- Strictly adheres to a `Genre` - `Artist` - `Album` folder hierarchy
    - Maybe you can override this somehow by writing full paths in `iprefix` keys but it'd be awkward and definitely a hassle

### Features

- Multiple output configurations so you can have different settings so you can set a different codec and bitrate on your 3DS copy compared to your phone copy (just an example—use it however you want). Or you can just have one single `default` configuration.
    - Lets you copy the ReplayGain tags as is, or apply ReplayGain itself to the audio data if you want (will automatically remove the ReplayGain tags on the output in that case)
    - Runs on FFMPEG so you can read and write to a plethora of codecs. Maybe you want your master music collection as AMR files but need them to be GSM full rate on your portable device? You can definitely do that.
- Keeps music tags but discards embedded album art. Avoid duplication of data when you can save it as one single album art image file.
- Album art image is automatically rescaled when needed, and copied as a lower quality image.
    - Image format, resolution, and quality configurable via the `db` file
- Will not encode and copy a file if the origin file has not changed since last encoding (also applies to album art)
- The database is a simple JSON file so you can edit it easily. No proprietary data storage here.
- Will automatically remove files and folders in the destination folder that is not in the JSON database.
    - This will of course not remove album art (although if the album itself doesn't exist on the database, the entire album folder will be deleted so that will also delete its album art)
    - This will ignore folders in `oroot` that are in the `ignore_root` config setting because sometimes you just have to have those.
- Will accept both video and audio files.
    - Will convert the video file to audio only in case you do feed it a video file.

### Usage

```bash
kmus db [--alt config_name]
```

* **db** - the JSON database file. Doesn't have to be named `db`, you can call it whatever you want.
* **--alt** - in case you want to use a different output configuration from the `default` one, use this
  * **config name** - enter the config name you want to use

### Example `db` file

```json
{
  "config": {
    "iroot": "/media/K_MUSIC/PCM",
    "output": {
      "default": {
        "oroot": "/home/kylxbn/Music/Portable",
        "codec": "-c:a libopus -b:a 190k -apply_phase_inv 0",
        "extension": "opus",
        "copy_art": true
      },
      "fiio": {
        "oroot": "/media/FiiO M7/Music",
        "codec": "-c:a libvorbis -aq 5",
        "extension": "ogg",
        "copy_art": true,
        "rg_preamp": 5.0,
        "apply_rg": "track"
      },
      "3ds": {
        "oroot": "/media/3DS/Music",
        "codec": "-c:a libopus -b:a 32k -apply_phase_inv 1 -ac 2 -ar 48000",
        "extension": "opus",
        "copy_art": false,
        "rg_preamp": 5.0,
        "apply_rg": "track"
      }
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

* The `iroot` is the base path of your master music collection.
* You have at least one configuration in the `output` named `default`. If you don't specify `--alt`, then this configuration will be used. However, you can have as many configurations as you want and then use which you like depending on the situation.
    * The `oroot` is the base path of your portable music folder.
    * The `codec` is the FFMPEG setting related to the output codec. Change this as needed.
    * The `extension` is of course, the proper file extension of the codec. For example, `ogg` when encoding to Ogg Vorbis.
    * `copy_art` configures whether the album art requirement is enforced and album arts are copied to your portable music folder.
    * `apply_rg` configures whether ReplayGain is applied to the audio data instead of just being copied as tag metadata. Regardless of this setting, ReplayGain tags are still required on the source files. This just configures whether the ReplayGain is applied to the audio data and removed from the tags, or just copied as is as tags. This can be `track` to apply the track ReplayGain, or `album` to apply the album ReplayGain. Don't add `apply_rg` if you don't want to apply ReplayGain.
        * If you are applying ReplayGain to audio data, and you want to adjust the loudness and add a preamp to the ReplayGain value, then you can add `rg_preamp`. If the original ReplayGain is -6dB and you set this to `3.5`, then the actual ReplayGain applied to the audio will be 2.5dB.
        * The `rg_preamp` sets the preamp of ReplayGain values but this does not modify the tag value. Instead, this sets the preamp when you apply the ReplayGain to the audio itself. (see the explanation for the `codec` config below)
* The `ignore_root` array contains the folders that the autoremove feature should ignore when scanning the folders in your `iroot`.
* The `iprefix` key is the *input path* component. For example, the source song file in the example above is ultimately located in the path `/media/K_MUSIC/PCM/Pop/Rick Astley/Whenever You Need Somebody/01 Never Gonna Give You Up.wv`. By default, this will be transcoded to `/home/kylxbn/Music/Portable/Pop/Rick Astley/Whenever You Need Somebody/01 Never Gonna Give You Up.opus`. However, if you do need it, there is an option to specify an `oprefix` as well, when you want to customize the output path of the file. For example, if you add `"oprefix": "Meme King",` after `"iprefix": "Rick Astley",`, then the resulting path of the transcoded file will be `/home/kylxbn/Music/Portable/Pop/Meme King/Whenever You Need Somebody/01 Never Gonna Give You Up.wv`.

See? That wasn't too difficult, was it?

#### Why not just use `-af volume=replaygain=track`?

If you peek inside the source code, you'll notice that I manually set the volume filter instead of just using FFMPEG's built in ReplayGain mechanism when applyin ReplayGain to the audio data. For some reason, ffmpeg won't recognize the ReplayGain tags on my file even though ffprobe shows them, so I had to do a workaround.

### Known Issues

- Foobar2000 seems to ignore replaygain tags on Opus files. For Opus files, just use the `apply_rg` option to work around this while I investigate.

### License

kmus - Kyle's Music Library Manager
Copyright (C) 2023  Kyle Alexander Buan

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program.  If not, see <https://www.gnu.org/licenses/>.
