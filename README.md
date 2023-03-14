## kmus - Kyle's Music Manager

`kmus` syncs a list of music files from my music archive to my portable music player, much like how iTunes could have done it with an iPod, but this time, you're in control of everything.

### Restrictions

kmus applies some restrictions to the music that it syncs.

- Source file must have replay gain tags
- Source album must have album art image (either `album.jpg` or `cover.jpg`)
- Album art image must be less than or equal to 750 KB.

### Features

- Strictly adheres to a `Genre` - `Artist` - `Album` folder hierarchy
- Configurable encoding settings (format and options)
- Uses `ffmpeg` so it can read and write a plethora of codecs
- Keeps music tags but discards embedded album art. Avoid duplication of data when you can save it as one single album art image file.
- Will not encode and copy a file if the origin file has not changed since last encoding (also applies to album art)
- The database is a simple JSON file so you can edit it easily. No proprietary data storage here.

### Usage

```bash
kmus \[db\]
```

* **db** - the JSON database file

### Example `db` file

```json
{
  "config": {
    "iroot": "/media/K_MUSIC/PCM",
    "oroot": "/home/kylxbn/Music/Portable",
    "codec": "-c:a libopus -b:a 190k -apply_phase_inv 0",
    "extension": "opus"
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

The `iroot` is the base path of your whole music collection. Likewise, the `oroot` is the base path of your portable music folder.

The `iprefix` key is the *input path* component. For example, the source song file in the example above is ultimately located in the path `/media/K_MUSIC/PCM/Pop/Rick Astley/Whenever You Need Somebody/01 Never Gonna Give You Up.wv`. By default, this will be transcoded to `/home/kylxbn/Music/Portable/Pop/Rick Astley/Whenever You Need Somebody/01 Never Gonna Give You Up.opus`. However, if you do need it, there is an option to specify an `oprefix` as well, when you want to customize the output path of the file. For example, if you add `"oprefix": "Meme King",` after `"iprefix": "Rick Astley",`, then the resulting path of the transcoded file will be `/home/kylxbn/Music/Portable/Pop/Meme King/Whenever You Need Somebody/01 Never Gonna Give You Up.wv`. You get the point by now, I hope.

### License

GPL-3.
