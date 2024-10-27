> Is there any suggestion on how to prevent FFmpeg from stalling when
> opening an RTSP source has failed?

There are two timeout type of options for RTSP:

‘-timeout’

    Set maximum timeout (in seconds) to wait for incoming connections.

    A value of -1 mean infinite (default). This option implies the
    ‘rtsp_flags’ set to ‘listen’. ‘reorder_queue_size’

    Set number of packets to buffer for handling of reordered packets.

‘-stimeout’

    Set socket TCP I/O timeout in micro seconds.

See the RTSP protocol documentation for more info:
http://ffmpeg.org/ffmpeg-protocols.html#rtsp