PHOTOGRAPHY SLOTS
=================
The site references these filenames only. Drop a new image over the old one
keeping the SAME filename and the site picks it up. No HTML editing.

  hero-1.jpg       Rotating homepage hero, frame 1. Cross-fades every 6s, with
  hero-2.jpg       manual arrows either side and dots beneath. Landscape; the
  hero-3.jpg       headline sits lower-left, so keep that area uncluttered.
                   Each one carries its own inline object-position in
                   index.html, because the right crop depends on where the
                   truck sits in that particular frame -- if you swap a file,
                   re-check that number.
  summit.jpg       Summit tier card. The card is 16:10, NOT 16:9.
  apex.jpg         Apex tier card. Same.
  about-trio.jpg   About section. 1.62:1 to match the box exactly.
  film.mp4         The film band above About.
  film.jpg         Its poster: shown before the video loads, and what a phone
                   with data saver on shows instead.

The gallery page uses none of these. It builds itself at load from
media/gallery/manifest.json plus one WebP per photo in media/gallery/view/.
To change the gallery, regenerate those; gallery.html never needs editing.

Retired: hero.jpg, facility.jpg and film-poster.jpg. Nothing references them.

VIDEO ENCODING
==============
Two limits apply, and the tighter one wins:
  - Claude artifact hosting: 15 MB per binary file.
  - Cloudflare Pages:        25 MiB per file.

The source ad (advertisement_Ford F450_v2.mp4) is 1920x1080 and 98.9 MB, so
it has to be re-encoded. Two-pass at a target bitrate is used rather than CRF
because it lands on a predictable size -- CRF guesses, and the 15 MB ceiling
is not a number worth guessing at:

  ffmpeg -i SRC -c:v libx264 -b:v 1900k -pass 1 -an -vf scale=1280:-2 \
    -preset slow -profile:v high -f mp4 /dev/null
  ffmpeg -i SRC -c:v libx264 -b:v 1900k -pass 2 -vf scale=1280:-2 \
    -preset slow -profile:v high -c:a aac -b:a 96k -ac 2 \
    -movflags +faststart film.mp4

That is 720p, 51s, and lands at 12.40 MB -- 2.6 MB under the artifact limit.
+faststart puts the index at the front so playback starts before the file has
finished downloading; without it the band sits dead until the whole file is in.

For a longer cut, keep the SIZE fixed and let the bitrate fall out of it:
  b:v (kbps) = (12 * 8192) / duration_seconds - 96

The poster is pulled from the cut itself so the band is never empty:
  ffmpeg -ss 28.5 -i film.mp4 -frames:v 1 -q:v 2 film.jpg

A note on picking that frame: do not choose it by sharpness score. Tried it,
and it picked a frame full of roadside brush over the one with the truck in
it -- foliage scores higher than sheet metal. Look at a contact sheet instead.

WATERMARK
=========
The ad carries CARTOMEDIA's watermark in the top-left. It is left alone in the
video. about-trio.jpg is a frame from the same ad, cropped to start below it
(the mark clears at y=130 of 1080).

STATUS
======
The gallery, the tier cards, the heroes, the About photo and the film are all
MAV's own trucks. The stock placeholder truck is gone from the site.
