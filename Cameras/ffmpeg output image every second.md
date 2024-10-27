It's very simple with ffmpeg, and it can output one frame every N seconds without extra scripting. To export as an image sequence just use myimage_%04d.png or similar as the output. The %0xd bit is converted to a zero-padded integer x digits long - the example I gave gets output as

myimage_0000.png,

myimage_0001.png,

myimage_0002.png

etc..

You can use lots of still image formats, `png`, `jpeg`, `tga`, whatever (see `ffmpeg -formats` for a full list).

Ok so now we know how to export the movie as a sequence of images, but say we don't want to export every single frame?

The trick is to simply change the frame rate of the output to whatever we want using the `-r n` option where `n` is the number of frames per second. 1 frame per second would be `-r 1`, one frame every four seconds would be `-r 0.25`, one frame every ten seconds would be `-r 0.1` and so on.

So to put it all together, this is how it would look to save one frame of `input.mov` every four seconds to `output_0000.png`, `output_0001.png` etc.:

`ffmpeg -i input.mov -r 0.25 output_%04d.png`

Note that the `-r 0.25` option goes after the `-i input.mov` part, because it's controlling the frame rate of the output. If you put it before the input it would treat the input file as if it had the specified frame rate.

Change the `%xd` to however many digits you need, e.g. if the command would create more than 10,000 frames change the `%04d` to `%05d`. This also works for input files that are image sequence. Read more here.

Windows users: On the command line use %

example: `ffmpeg -i inputFile.mp4 -r 1 outputFile_%02d.png`

In CMD and BAT Scripts use %%

example: `ffmpeg -i inputFile.mp4 -r 1 outputFile %%02d.png`

So double %% in scripts, and single % on the interactive command line. Getting it wrong in either situation will generate an error.

Edit Worth checking out the discussion of file types in the comments below. There are advantages and disadvantages with each:

jpeg much smaller file sizes, lossy, only 8-bit colour, no alpha.
png lossless, large files, slow. 8-bit colour supports alpha.
tiff lossless, even larger files, fast, supports up to 16 colour and alpha.
To specify the file format, just use the required extension in the output file, e.g. `myimage_%04d.jpg`, `myimage_%04d.png` or `myimage_%04d.tiff`.
