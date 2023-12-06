Determining CasparCG Media Type
========

When Media Scanner app starts up it scans the CasparCG server store, probing each file in the main media folder and any sub-folders to determine the CasparCG media file type - AUDIO, STILL or MOVIE. It also monitors the template folder for the html and flash templates. After the initial scan completes it monitors the media store and template store for any changes to the media files, running a scan operation on added files and removing deleted files from the media and template database.

Version 1.0 SVT scanner processing assumed a movie file has any video in stream 0, but this is frequently not the case. If the file has audio in stream 0 and video in stream 1 the file was allocated to be type AUDIO, and this was placed in the wrong library section of the SVT standard control client.

This version of the scanner, based on the code base for SVT scanner version 1.2.0, tests every stream in the file looking for the presence of video, audio and still image element streams. It uses the combination of stream types found to allocate the media type category.

**ffprobe** returns a JSON formatted property set for the media file. One JSON property is an array of streams where each stream has a property called `codec_type`. The two property values of interest are "audio" and "video".  

There are three further properties of interest for video streams - `duration`, `codec_name`, `disposition.attached_pic`. The duration property is the primary mechanism to detect when a video stream is a still image. The duration value is the replay time for the stream, expressed as a floating point number (in a string format). For several still image file formats the duration property is "undefined", but .jpg/.jpeg and Targa (.tga) files return a duration of 40 milliseconds. CasparCG displays GIF files as a still, using the first image in the file. The duration property of the GIF file varies, but GIF media can be identified by the codec_name property. 

It is possible to have an audio file that includes a still image - such as an album cover. Tests on sveral such files show the duration property is commonly the same as the audio stream, but the `disposition.attached_pic` property has the value 1 to indicate the poster frame nature of the stream.

Some stills may be extraced from movie clip using a video editing package. Some edit packages do not have simple still export, so it is neccessary to create a minimal length media file that can be used as a still by CasparCG server. Most edit packages allow the user to reduce clip length by dragging an end of the clip using the mouse, but this may only reduce to two frames (for exaple using Final Cut Pro or Blackmagic Design DaVinci). The lowest frame rate is for movies at 24 * (1000/1001) giving a 2 frame duration of just over 0.0834 seconds. This version of scanner therefore declares video clips that have a duration less than 0.1 second to be stills.  

It is possible that audio is exported as part of the media file, so there is a potential to flag the file as audio with poster frame. This possibility is detected and the media type forced to 'STILL'

The simple still detection algorithm is:

```
const stillLimit = 0.1

if (stream.codec_type === 'video')  {
   if ((Number(stream.duration ?? '0') < stillLimit) || 
       (stream.disposition.attached_pic === 1) || 
       (stream.codec_name === 'gif')) 
       {
         // process stream as a still image.
       }
}
```

