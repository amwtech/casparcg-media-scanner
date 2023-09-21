Determining CasparCG Media Type
========

When the Media Scanner process starts it scans the CasparCG server store probing each file in the main media folder and any sub-folders to determine the CasparCG media file type - AUDIO, STILL or MOVIE. It also monitors the template folder for the html and flash templates. After the initial scan completes it monitors the media store and template store for any changes to the media files running a scan operation on added files and removing deleted files from the media and template database.

Version 1.0 SVT scanner processing assumes a movie file has any video in stream 0, but this is frequently not the case. If the file has audio in stream 0 and video in stream 1 the file was allocated to be type AUDIO, and this was placed in the wrong library section of the SVT standard control client.

This version of the scanner, based on the code base for SVT scanner version 1.2.0, tests every stream in the file looking for the presence of video, audio and still image element streams. It uses the combination of stream types found to allocate the media type category.

ffprobe returns a JSON formatted property set for the file. One JSON property is an array of streams where each stream has a property called `codec_type`. The two property values of interest are "audio" and "video". There is also a property called `codec_time_base` which returns a value "0/1" for any stream that is a still image. Note that GIF files can contain multiple images these are defined as movies not stills.

This version of the scanner defines a media file as a still for files that return the property combination:

```
((streams[n].codec_type === "video") && (codec_time_base === "0/1"))
```
The media type allocation is managed by `function generateCinf(doc, json)` in source file scanner.js which has been re-coded to test all streams, using the first instance of a stream as the source of the primary properties. An object has three flag fields, one per media stream of interest. The flags are initialised to 0, and a new value set when a matching stream type is found. The audio stream flag is set to value 4, the video stream flag to value 2, and the stills stream set to value 1. This gives eight possible combinations which area allocted to media type as listed below.

|Audio|Video|Still|Total|CasparCG Type|  
|---|---|---|---|---|
| 0 | 0 | 0 | 0 |AUDIO|
| 0 | 0 | 1 | 1 |STILL|  
| 0 | 2 | 0 | 2 |MOVIE| 
| 0 | 2 | 1 | 3 |MOVIE| 
| 4 | 0 | 0 | 4 |AUDIO| 
| 4 | 0 | 1 | 5 |AUDIO| 
| 4 | 2 | 0 | 6 |MOVIE| 
| 4 | 2 | 1 | 7 |MOVIE|  

The combination in the first line is not relevant and the file is declared non-media by an earlier test ahead of function generateCinf(). The line with a toal of 5 is an audio file with cover artwork, and hence should be defined as audio.