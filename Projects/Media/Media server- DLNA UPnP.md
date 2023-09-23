Thoughts:  
While DLNA is quite ubiquitous, it doesn't stack best with Fidd's philosophy, violating the principle of decrypting the content only at the consumption endpoint.  
Fidd's more naturally compatible with file/object transfer protocols, like FTPs, NFS, S3, etc.  
  
While playing local files is naturally a function of any player, modern players can stream from HTTP, FTPs and other protocols, for which Fidd client can provide local services to consume encrypted files.  
(see VLC 'Open Network Stream' )  
  
VLC also supports RTSP, but ideally Fidd just wants to stream the file, without thinking a whole lot about what it is.  
The way I see it now:  
While we can provide FTP endpoint for the client to navigate the file collection, it's much less involved and much more secure to expose a file or a group of files (potentially + a playlist) via HTTP, and let player stream it.  
  
**Thus, Exposing files on demand, on need-to-know basis, starting a player (or any external program) from Fidd client seems to be the easiest and securest way.**  
*Also, download file via browser*  
  
TBD: can a player open a playlist from HTTP, and then play media files from the same HTTP as referenced in the playlist?  
  
---  
  
~~VLC can be controlled via https://wiki.videolan.org/VLC_HTTP_requests/~~  
Ideally we don't need this HTTP interface, because we just supply the playlist in command line parameters.  
  
And VLC player's "open" function supports the following protocols:  
https://wiki.videolan.org/Media_resource_locator/  
  
---  
  
Abstract: At some point develop Local Media Server.  
Fidd-integrated.  
  
---  
  
Java lib:  
https://github.com/4thline/cling (not maintained)  
https://github.com/jupnp/jupnp (new fork)  
  
---  
  
A simple, zero-config DLNA media server, that you can just fire up and be done with it.  
https://nmaier.github.io/simpleDLNA/  
--> code  
https://github.com/nmaier/simpleDLNA  
  
---  
  
Linux Player:  
  
Recently VLC Version 2.0 was released. This Release features a fully functional UPNP / DLNA Player that works without bugs. I'm using it on Ubuntu and Archlinux and it works just fine with my minidlna player.  
  
To use VLC start the Playlist (`View`  ->  `Playlist`;  `Ctrl + L`) and then select  `Local Network`  ->  `Universal Plug 'n' Play`. However, be advised that VLCs implementation first downloads all the library information before you can select anything. Depending on the size of the library that actually might take several minutes and longer.  
  
VLC might require to have been launched with  `--miface=INTERFACE`  flag, because on some machines VLC uses the wrong interface by default. You can find the correct interface name from  `ifconfig`.  
  
---  
  
11 of the Best DLNA Streaming Apps for Android  
  
-   [1. VLC](https://play.google.com/store/apps/details?id=org.videolan.vlc&hl=en_US)  
-   [2. Plex](https://play.google.com/store/apps/details?id=com.plexapp.android&hl=en_GB)  
-   [3. Cast Videos: Castify](https://play.google.com/store/apps/details?id=castify.roku&hl=en_IN&gl=US)  
-   [4. LocalCast](https://play.google.com/store/apps/details?id=de.stefanpledl.localcast)  
-   [5. Kodi](https://play.google.com/store/apps/details?id=org.xbmc.kodi)  
-   [6. Hi-Fi Cast + DLNA](https://play.google.com/store/apps/details?id=com.findhdmusic.app.upnpcast)  
-   [7. XCast](https://play.google.com/store/apps/details?id=cast.video.screenmirroring.casttotv&hl=en_IN&gl=US)  
-   [8. MediaMonkey](https://play.google.com/store/apps/details?id=com.ventismedia.android.mediamonkey)  
-   [9. BubbleUPnP](https://play.google.com/store/apps/details?id=com.bubblesoft.android.bubbleupnp&hl=en)  
-   [10. iMediaShare Personal](https://play.google.com/store/apps/details?id=com.bianor.amspersonal&hl=en)  
-   [11. AllCast](https://play.google.com/store/apps/details?id=com.koushikdutta.cast)