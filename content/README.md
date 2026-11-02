# SD card layout

The firmware manages the card through the web page. You do not need to hand build it. For backup and restore, the card holds:

```
/library.json      tag UID to track list bindings, plus settings such as poseRequise and volume
/<uploaded files>  MP3 and M4A as uploaded from the web page
```

Back up by copying the whole card. Restore by copying it back. No media is committed to this repo.
