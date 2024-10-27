## ffmpeg timestamp in segment name 

Appears you need to add the "-f segment" parameter. Here's an example with strftime as well:

ffmpeg -i your_input -f segment -strftime 1 -segment_time 60 -segment_format mp4 out%Y-%m-%d_%H-%M-%S.mp4
segment_time 60 means 60 seconds, strftime 1 means "enable strftime names"

For me this created files with names like this:

out2015-03-05_10-27-43.mp4

