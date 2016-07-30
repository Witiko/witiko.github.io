---
layout: post
title: Hugin Doesn't Abide the Orientation EXIF Tag
tags:
  - linux
  - snippet
  - photography
---

Lately, I have been stitching some of the panoramas I took during the vacation using [Hugin](http://hugin.sourceforge.net). Since I took quite a lot of these, I though I would leverage the shell and let Hugin create the panoramas automatically using the workflow described at <http://wiki.panotools.org/Panorama_scripting_in_a_nutshell>.

My camera is an aging Canon Powershot A590A loaded with a community [CHDK firmware](http://chdk.wikia.com/), which enables the camera to save digital negatives even though it was never meant to. For the majority of pictures, however, I just use the JPEG output directly. There is one peculiar thing about the output -- the camera detects its orientation using an internal gyroscope, but it doesn't rotate the output JPEG picture. Instead, it just sets the EXIF Orientation tag, which tells the viewer application to display the picture rotated.

The issue I encountered is this -- some of the tools in the Hugin toolchain ignore the Orientation EXIF tag, while some do not. The results are completely ill-stitched panoramas. A quick search yielded an `exifautotran` script readily available in the Debian GNU/Linux repositories (see the `libjpeg-turbo-progs` package), which hard-rotates the picture according to the EXIF tag and then clears the tag. The script, however, is unable to work with JFIF 1.01 (as documented at <http://how-to.wikia.com/wiki/How_to_auto-rotate_digital_photos_to_their_proper_orientation>). After a while, I came up with the following script that does what `exifautotran` was supposed to:

```bash
#!/bin/bash
# Hard-rotates the $@ EXIF-tagged images.
# Requires the `exiftool` and `libjpeg-turbo-progs` packages.
for FILE; do
  ORIENTATION=`exiftool -t -s -n -IFD{0,1}:Orientation "$FILE" |
    head -1 | awk '{ print $2 }'` || exit 1
  [[ -z $ORIENTATION || $ORIENTATION = 1 ]] && continue
  case $ORIENTATION in
    2) echo -flip horizontal ;;
    3) echo -rotate 180      ;;
    4) echo -flip vertical   ;;
    5) echo -rotate 90
       echo -flip horizontal ;;
    6) echo -rotate 90       ;;
    7) echo -flip horizontal
       echo -rotate 90       ;;
    8) echo -rotate 270      ;;
    *) exit 3                ;;
  esac | while read FLAGS; do
    jpegtran -copy all -perfect $FLAGS -outfile  "$FILE" "$FILE"
  done || exit $?
  exiftool -overwrite_original -n -IFD{0,1}:Orientation=1 "$FILE" || exit 4
done
```

Hopefully, you will also find this workaround useful.
