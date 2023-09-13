https://seedsigner.com/  
https://github.com/SeedSigner/seedsigner  
  
# Build an offline, airgapped Bitcoin signing device for less than $50!  
  
[![Image of SeedSigners in Open Pill Enclosures](https://github.com/SeedSigner/seedsigner/raw/dev/docs/img/Open_Pill_Star.JPG)](https://github.com/SeedSigner/seedsigner/blob/dev/docs/img/Open_Pill_Star.JPG)[![Image of SeedSigner in an Orange Pill enclosure](https://github.com/SeedSigner/seedsigner/raw/dev/docs/img/Orange_Pill.JPG)](https://github.com/SeedSigner/seedsigner/blob/dev/docs/img/Orange_Pill.JPG)  
  
----------  
  
-   [Project Summary](https://github.com/SeedSigner/seedsigner/blob/dev/README.md#project-summary)  
-   [Shopping List](https://github.com/SeedSigner/seedsigner/blob/dev/README.md#shopping-list)  
-   [Software Installation](https://github.com/SeedSigner/seedsigner/blob/dev/README.md#software-installation)  
    -   [Verifying the Software](https://github.com/SeedSigner/seedsigner/blob/dev/README.md#verifying-the-software)  
-   [Enclosure Designs](https://github.com/SeedSigner/seedsigner/blob/dev/README.md#enclosure-designs)  
-   [SeedQR Printable Templates](https://github.com/SeedSigner/seedsigner/blob/dev/README.md#seedqr-printable-templates)  
-   [Manual Installation Instructions](https://github.com/SeedSigner/seedsigner/blob/dev/README.md#manual-installation-instructions)  
  
----------  
  
# [](https://github.com/SeedSigner/seedsigner/blob/dev/README.md#project-summary)Project Summary  
  
The goal of SeedSigner is to lower the cost and complexity of Bitcoin multi-signature wallet use. To accomplish this goal, SeedSigner offers anyone the opportunity to build a verifiably air-gapped, stateless Bitcoin signing device using inexpensive, publicly available hardware components (usually < $50). SeedSigner helps users save with Bitcoin by assisting with trustless private key generation and multisignature (aka "multisig") wallet setup, and helps users transact with Bitcoin via a secure, air-gapped QR-exchange signing model.  
  
Additional information about the project can be found at  [SeedSigner.com](https://seedsigner.com/).  
  
You can follow  [@SeedSigner](https://twitter.com/SeedSigner)  on Twitter for the latest project news and developments.  
  
If you have specific questions about the project, our  [Telegram Group](https://t.me/joinchat/GHNuc_nhNQjLPWsS)  is a great place to ask them.  
  
### [](https://github.com/SeedSigner/seedsigner/blob/dev/README.md#feature-highlights)Feature Highlights:  
  
-   Calculate the final word (aka checksum) of a 12- or 24-word BIP39 seed phrase  
-   Create a 24-word BIP39 seed phrase with 99 dice rolls or a 12-word with 50 rolls  [(Verifying dice seed generation)](https://github.com/SeedSigner/seedsigner/blob/dev/docs/dice_verification.md)  
-   Create a 12- or 24-word BIP39 seed phrase via image entropy from the onboard camera  
-   Temporarily stores seeds in memory while the device is powered; all memory is wiped when power is removed  
-   SD card removable after boot to ensure no secret data can be written to it  
-   Guided interface to manually transcribe a seed to the SeedQR format for instant seed loading  [(demo video here)](https://youtu.be/c1-PqTNx1vc)  
-   BIP39 passphrase (aka "word 25") support  
-   Native Segwit Multisig XPUB generation  
-   PSBT-compliant; scan and parse transaction data from animated QR codes  
-   Sign transactions & transfer XPUB data using animated QR codes  [(demo video here)](https://youtu.be/LPqvdQ2gSzs)  
-   Live preview during image entropy seed generation and QR scanning UX  
-   Optimized seed word entry interface  
-   Support for Bitcoin Mainnet & Testnet  
-   Support for custom user-defined derivation paths  
-   On-demand receive address verification  
-   Address Explorer for single sig and multisig wallets  
-   User-configurable QR code display density  
-   Responsive, event-driven user interface  
  
### [](https://github.com/SeedSigner/seedsigner/blob/dev/README.md#considerations)Considerations:  
  
-   Built for compatibility with Specter Desktop, Sparrow, and BlueWallet Vaults  
-   Device takes up to 60 seconds to boot before menu appears (be patient!)  
-   Always test your setup before transferring larger amounts of bitcoin (try Testnet first!)  
-   Taproot not quite yet supported  
-   Slightly rotating the screen clockwise or counter-clockwise should resolve lighting/glare issues  
-   If you think SeedSigner adds value to the Bitcoin ecosystem, please help us spread the word! (tweets, pics, videos, etc.)  
  
### [](https://github.com/SeedSigner/seedsigner/blob/dev/README.md#planned-upcoming-improvements--functionality)Planned Upcoming Improvements / Functionality:  
  
-   Multi-language support  
-   Significantly faster boot time  
-   Reproducible builds  
-   Port to MicroPython to broaden the range of compatible hardware to include low-cost microcontrollers  
-   Other optimizations based on user feedback!  
  
----------  
  
# [](https://github.com/SeedSigner/seedsigner/blob/dev/README.md#shopping-list)Shopping List  
  
To build a SeedSigner, you will need:  
  
-   Raspberry Pi Zero (preferably version 1.3 with no WiFi/Bluetooth capability, but any Raspberry Pi 2/3/4 or Zero model will work, Raspberry Pi 1 devices will require a hardware modification to the Waveshare LCD Hat, as per the  [instructions here](https://github.com/SeedSigner/seedsigner/blob/dev/docs/legacy_hardware.md))  
-   Waveshare 1.3" 240x240 pxl LCD (correct pixel count is important, more info at  [https://www.waveshare.com/wiki/1.3inch_LCD_HAT](https://www.waveshare.com/wiki/1.3inch_LCD_HAT))  
-   Pi Zero-compatible camera (tested to work with the Aokin / AuviPal 5MP 1080p with OV5647 Sensor)  
  
Notes:  
  
-   You will need to solder the 40 GPIO pins (20 pins per row) to the Raspberry Pi Zero board. If you don't want to solder, purchase "GPIO Hammer Headers" for a solderless experience.  
-   Other cameras with the above sensor module should work, but may not fit in the Orange Pill enclosure  
-   Choose the Waveshare screen carefully; make sure to purchase the model that has a resolution of 240x240 pixels  
  
----------  
  
# [](https://github.com/SeedSigner/seedsigner/blob/dev/README.md#software-installation)Software Installation  
  
## [](https://github.com/SeedSigner/seedsigner/blob/dev/README.md#a-special-note-on-minimizing-trust)A Special Note On Minimizing Trust  
  
As is the nature of pre-packaged software downloads, downloading and using the prepared SeedSigner release images means implicitly placing trust in the individual preparing those images; in our project the release images are prepared and signed by the eponymous creator of the project, SeedSigner "the person". That individual is additionally the only person in possession of the PGP keys that are used to sign the release images.  
  
However, one of the many advantages of the open source software model is that the need for this kind of trust can be negated by our users' ability to (1) review the project's source code and (2) assemble the operating image necessary to use the software themselves. From our project's inception, instructions to build a SeedSigner operating image (using precisely the same process that is used to create the prepared release images) have been made availabile. We have put a lot of thought and work into making these instructions easy to understand and follow, even for less technical users. These instructions can be found  [here](https://github.com/SeedSigner/seedsigner/blob/dev/docs/manual_installation.md).  
  
## [](https://github.com/SeedSigner/seedsigner/blob/dev/README.md#downloading-the-software)Downloading the Software  
  
Download the current Version (0.6.0) software image that is compatible with your Raspberry Pi Hardware. The Pi Zero 1.3 is the most common and recommended board.  
  
Board  
  
Download Image Link/Name  
  
**[Raspberry Pi Zero 1.3](https://www.raspberrypi.com/products/raspberry-pi-zero/)**  
  
[`seedsigner_os.0.6.0.pi0.img`](https://github.com/SeedSigner/seedsigner/releases/download/0.6.0/seedsigner_os.0.6.0.pi0.img)  
  
[Raspberry Pi Zero W](https://www.raspberrypi.com/products/raspberry-pi-zero-w/)  
  
[`seedsigner_os.0.6.0.pi0.img`](https://github.com/SeedSigner/seedsigner/releases/download/0.6.0/seedsigner_os.0.6.0.pi0.img)  
  
[Raspberry Pi Zero 2 W](https://www.raspberrypi.com/products/raspberry-pi-zero-2-w/)  
  
[`seedsigner_os.0.6.0.pi02w.img`](https://github.com/SeedSigner/seedsigner/releases/download/0.6.0/seedsigner_os.0.6.0.pi02w.img)  
  
[Raspberry Pi 2 Model B](https://www.raspberrypi.com/products/raspberry-pi-2-model-b/)  
  
[`seedsigner_os.0.6.0.pi2.img`](https://github.com/SeedSigner/seedsigner/releases/download/0.6.0/seedsigner_os.0.6.0.pi2.img)  
  
[Raspberry Pi 3 Model B](https://www.raspberrypi.com/products/raspberry-pi-3-model-b/)  
  
[`seedsigner_os.0.6.0.pi02w.img`](https://github.com/SeedSigner/seedsigner/releases/download/0.6.0/seedsigner_os.0.6.0.pi02w.img)  
  
[Raspberry Pi 4 Model B](https://www.raspberrypi.com/products/raspberry-pi-4-model-b/)  
  
[`seedsigner_os.0.6.0.pi4.img`](https://github.com/SeedSigner/seedsigner/releases/download/0.6.0/seedsigner_os.0.6.0.pi4.img)  
  
[Raspberry Pi 400](https://www.raspberrypi.com/products/raspberry-pi-400-unit/)  
  
[`seedsigner_os.0.6.0.pi4.img`](https://github.com/SeedSigner/seedsigner/releases/download/0.6.0/seedsigner_os.0.6.0.pi4.img)  
  
Note: If you have physically removed the WiFi component from your board, you will still use the image file of the original(un-modified) hardware. (Our files are compiled/based on the  _processor_  architecture). Although it is better to spend a few minutes upfront to determine which specific Pi hardware/model you have, if you are still unsure which hardware you have, you can try using the pi0.img file. Making an incorrect choice here will not ruin your board, because this is software, not firmware.  
  
**also download**  these 2 signature verification files to the same folder    
[The Plaintext manifest file](https://github.com/SeedSigner/seedsigner/releases/download/0.6.0/seedsigner.0.6.0.sha256)    
[The Signature of the manifest file](https://github.com/SeedSigner/seedsigner/releases/download/0.6.0/seedsigner.0.6.0.sha256.sig)  
  
Users familiar with older versions of the SeedSigner software might be surprised with how fast their software downloads now are, because since version 0.6.0 the software image files are now 100x smaller! Each image file is now under 42 Megabytes so your downloads and verifications will be very quick now (and might even seem  _too_  quick)!  
  
Once the files have all finished downloading, follow the steps below to verify the download before continuing on to write the software onto a MicroSD card. Next, insert the MicroSD into your assembled hardware and connect the USB power. Allow about 45 seconds for our logo to appear, and then you can begin using your SeedSigner!  
  
[Our previous software versions are available here](https://github.com/SeedSigner/seedsigner/releases). Choose a specific version and then expand the  _Assets_  sub-heading to display the .img file binary and also the 2 associated signature files.  **Note:**  The prior version files will have lower numbers than the scripts and examples provided in this document, but the naming format will be the same, so you can edit them as required for signature verification etc.  
  
## [](https://github.com/SeedSigner/seedsigner/blob/dev/README.md#verifying-that-the-downloaded-files-are-authentic-optional-but-highly-recommended)Verifying that the downloaded files are authentic (optional but highly recommended!)  
  
You can quickly verify that the software you just downloaded is both authentic and unaltered, by following these instructions. We assume you are running the commands from a computer where both  [GPG](https://gnupg.org/download/index.html)  and  [shasum](https://command-not-found.com/shasum)  are already installed, and that you also know  [how to navigate on a terminal](https://terminalcheatsheet.com/guides/navigate-terminal).  
  
### [](https://github.com/SeedSigner/seedsigner/blob/dev/README.md#step-1-verify-that-the-signature-sig-file-is-genuine)Step 1. Verify that the signature (.sig) file is genuine:  
  
Run GPG's  _fetch-keys_  command to import the SeedSigner projects public key from the popular online keyserver called  _Keybase.io_, into your computers  _keychain_.  
  
```  
gpg --fetch-keys https://keybase.io/seedsigner/pgp_keys.asc  
  
```  
  
The result should confirm that 1 key was  _either_  imported or updated.  _Ignore_  any key ID's or email addresses shown.  
  
[![SS - Fetchkeys-Keybase PubKey import with Fingerprint shown (New import or update of the key)v3-100pct](https://user-images.githubusercontent.com/91296549/221334414-adc3616c-462e-490e-8492-3dfee367d13a.jpg)](https://user-images.githubusercontent.com/91296549/221334414-adc3616c-462e-490e-8492-3dfee367d13a.jpg)  
  
Next, you will run the  _verify_  command on the signature (.sig) file. (_Verify_  must be run from inside the same folder that you downloaded the files into earlier. The  `*`'s in this command will auto-fill the version from your current folder, so it should be copied and pasted as-is.)  
  
```  
gpg --verify seedsigner.0.6.*.sha256.sig  
  
```  
  
When the verify command completes successfully, it should display output like this:    
[![SS - Verify Command - GPG on Linux - Masked_v4-100pct](https://user-images.githubusercontent.com/91296549/221334135-8ad1f1af-26d2-429a-91ce-ad41703ed38c.jpg)](https://user-images.githubusercontent.com/91296549/221334135-8ad1f1af-26d2-429a-91ce-ad41703ed38c.jpg)    
The result must display "**Good signature**". Ignore any email addresses -  _only_  matching Key fingerprints count here. Stop immediately if it displays "_Bad signature_"!    
  
On the  _last_  output line, look at your  _rightmost_  16 characters (the 4 blocks of 4).    
**Crucially, we must now check WHO that Primary key fingerprint /ID belongs to.**  We will start by looking at Keybase.io to see if it is the  _SeedSigner project_  's public key or not.  
  
About the warning message:    
More about how the verify command works:    
  
Now to determine  _**who**_  the Public key ID belongs to: Goto  [Keybase.io/SeedSigner](https://keybase.io/seedsigner)    
    
[![SS - Keybase Website PubKey visual matching1_Cropped-80pct](https://user-images.githubusercontent.com/91296549/215326193-97c84e35-5570-4e52-bf3f-e86d367c8908.jpg)](https://user-images.githubusercontent.com/91296549/215326193-97c84e35-5570-4e52-bf3f-e86d367c8908.jpg)  
  
**You must now  _manually_  compare: The 16 character fingerprint ID (as circled in red above) to, those  _rightmost_  16 characters from your  _verify_  command.**  
  
**If they match exactly, then you have successfully confirmed that your .sig file is authentically from the SeedSigner Project!**    
  
Learn more about how keybase.io helps you check that someone (online) is who they say they are:    
  
If the two ID's do  _not_  match, then you must stop here immediately. Do not continue. Contact us for assistance in the Telegram group address above.  
  
    
  
### [](https://github.com/SeedSigner/seedsigner/blob/dev/README.md#step-2-verifying-that-the-software-imagesbinaries-are-genuine)Step 2. Verifying that the  _software images/binaries_  are genuine  
  
Now that you have confirmed that you do have the real SeedSigner Project's Public Key (ie the 16 characters match) - you can return to your terminal window. Running the  _shasum_  command, is the final verification step and will confirm (via file hashing) that the software code/image files, were also not altered since publication, or even during your download process.    
(Prior to version 0.6.0 , your verify command will check the .zip file which contains the binary files.)  
  
**On Linux or OSX:**  Run this command  
  
```  
shasum -a 256 --ignore-missing --check seedsigner.0.6.*.sha256    
  
```  
  
**On Windows (inside Powershell):**  Run this command  
  
```  
CertUtil -hashfile  seedsigner_os.0.6.0.Insert_Your_Pi_Models_binary_here_For_Example_pi02w.img SHA256   
  
```  
  
On Windows, you must then manually compare the resulting file hash value to the corresponding hash value shown inside the .SHA256 cleartext file.    
  
Wait up to 30 seconds for the command to complete, and it should display:  
  
```  
seedsigner_os.0.6.x.[Your_Pi_Model_For_Example:pi02w].img: OK  
  
```  
  
**If you receive the "OK" message**  for your  **seedsigner_os.0.6.x.[Your_Pi_Model_For_Example:pi02w].img file**, as shown above, then your verification is fully complete!    
**All of your downloaded files have now been confirmed as both authentic and unaltered!**  You can proceed to create/write your MicroSD card😄😄 !!  
  
If your file result shows "FAILED", then you must stop here immediately. Do not continue. Contact us for assistance at the Telegram group address above.  
  
    
  
Please recognize that this process can only validate the software to the extent that the entity that first published the key is an honest actor, and their private key is not compromised or somehow being used by a malicious actor.    
    
  
## [](https://github.com/SeedSigner/seedsigner/blob/dev/README.md#writing-the-software-onto-your-microsd-card)Writing the software onto your MicroSD card  
  
To write the SeedSigner software onto your MicroSD card, there are a few options available:  
  
Application  
  
Description  
  
Platform and official Source  
  
Balena Etcher  
  
The application is called Etcher, and the company that wrote it is called Balena. Hence  _Etcher by Balena_  or  _Balena Etcher_  
  
[Available for Windows, Mac and Linux](https://www.balena.io/etcher#download-etcher)  
  
Raspberry Pi Imager  
  
Produced by the Raspberry Pi organization.  
  
[Available for Windows, Mac and Linux](https://www.raspberrypi.com/software/)  
  
DD Command Line Utility  
  
Built-in to Linux and MacOS, the DD (Data Duplicator) is a tool for advanced users. If not used carefully it can accidentally format the incorrect disk!  
  
Built-in to Linux and MacOS  
  
Be sure to download the software from the genuine publisher.    
Either of the Etcher or Pi Imager software is recommended. Some SeedSigner users have reported a better experience with one or the other. So, if the one application doesn’t work well for your particular machine, then please try the other one.    
  
### [](https://github.com/SeedSigner/seedsigner/blob/dev/README.md#general-considerations)**General Considerations:**  
  
The writing and verify steps are very quick from version 0.6.0 upwards, so please pay close attention to your screen. Make sure to set any write-protection physical slider on the MicroSD Card Adapter to UN-locked.    
You also don’t need to pre-format the MicroSD beforehand. You  _dont_  need to unzip any .zip file beforehand. Current Etcher and Pi Imager software will perform a verify action (by default) to make sure the card was written successfully! Watching for that verify step to complete successfully, can save you a lot of headaches if you later need to troubleshoot issues where your SeedSigner device doesn’t boot up at power on.    
Writing the MicroSd card is also known as flashing.    
It will overwrite everything on the MicroSD card.    
If the one application fails for you, then please try again using our other recommended application.    
Advanced users may want to try the Linux/MacOS  _DD_  command instead of using Etcher or Pi Imager, however, a reminder is given that DD can overwrite the wrong disk if you are not careful !  
  
#### [](https://github.com/SeedSigner/seedsigner/blob/dev/README.md#specific-considerations-for-windows-users)**Specific considerations for Windows users:**  
  
Use the Pi imager software as your first choice on Windows. Windows can sometimes flag the writing of a MicroSD as risky behaviour and hence it may prevent this activity. If this happens, your writing/flashing will fail, hang or wont even begin, in which case you should to try to run the Etcher/Pi-Imager app "As administrator", (right-click and choose that option). It can also be blocked by windows security in some cases, so If you have the (non-default)  _Controlled Folder Access_  option set to active, try turning that  _off_  temporarily.  
  
----------  
  
# [](https://github.com/SeedSigner/seedsigner/blob/dev/README.md#enclosure-designs)Enclosure Designs  
  
### [](https://github.com/SeedSigner/seedsigner/blob/dev/README.md#open-pill)Open Pill  
  
The Open Pill enclosure design is all about quick, simple and inexpensive depoloyment of a SeedSigner device. The design does not require any additional hardware and can be printed using a standard FDM 3D printer in about 2 hours, no supports necessary. A video demonstrating the assembly process can be found  [here](https://youtu.be/gXPFJygZobEa). To access the design file and printable model, click  [here](https://github.com/SeedSigner/seedsigner/tree/main/enclosures/open_pill).  
  
### [](https://github.com/SeedSigner/seedsigner/blob/dev/README.md#orange-pill)Orange Pill  
  
The Orange Pill enclosure design offers a more finished look that includes button covers and a joystick topper. You'll also need the following additional hardware to assemble it:  
  
-   4 x F-F M2.5 spacers, 10mm length  
-   4 x M2.5 pan head screws, 6mm length  
-   4 x M2.5 pan head screws, 12mm length  
  
The upper and lower portions of the enclosure can be printed using a standard FDM 3D printer, no supports necessary. The buttons and joystick nub should ideally be produced with a SLA/resin printer. An overview of the entire assembly process can be found  [here](https://youtu.be/aIIc2DiZYcI). To access the design files and printable models, click  [here](https://github.com/SeedSigner/seedsigner/tree/main/enclosures/orange_pill).  
  
### [](https://github.com/SeedSigner/seedsigner/blob/dev/README.md#community-designs)Community Designs  
  
-   [Lil Pill](https://cults3d.com/en/3d-model/gadget/lil-pill-seedsigner-case)  by @_CyberNomad  
-   [OrangeSurf Case](https://github.com/orangesurf/orangesurf-seedsigner-case)  by @OrangeSurfBTC  
-   [PS4 SeedSigner](https://www.thingiverse.com/thing:5363525)  by @Silexperience  
-   [OpenPill Faceplate](https://www.printables.com/en/model/179924-seedsigner-open-pill-cover-plates-digital-cross-jo)  by @Revetuzo  
-   [Waveshare CoverPlate](https://cults3d.com/en/3d-model/various/seedsigner-coverplate-for-waveshare-1-3-inch-lcd-hat-with-240x240-pixel-display)  by @Adathome1  
  
----------  
  
# [](https://github.com/SeedSigner/seedsigner/blob/dev/README.md#seedqr-printable-templates)SeedQR Printable Templates  
  
You can use SeedSigner to export your seed to a hand-transcribed SeedQR format that enables you to instantly load your seed back into SeedSigner.  
  
[More information about SeedQRs](https://github.com/SeedSigner/seedsigner/blob/dev/docs/seed_qr/README.md)  
  
[![](https://github.com/SeedSigner/seedsigner/raw/dev/docs/seed_qr/img/handmade_qr.jpg)](https://github.com/SeedSigner/seedsigner/blob/dev/docs/seed_qr/img/handmade_qr.jpg)  
  
Standard SeedQR templates:  
  
-   [12-word SeedQR template dots (25x25)](https://github.com/SeedSigner/seedsigner/blob/dev/docs/seed_qr/printable_templates/dots_25x25.pdf)  
-   [24-word SeedQR template dots (29x29)](https://github.com/SeedSigner/seedsigner/blob/dev/docs/seed_qr/printable_templates/dots_29x29.pdf)  
-   [12-word SeedQR template grid (25x25)](https://github.com/SeedSigner/seedsigner/blob/dev/docs/seed_qr/printable_templates/grid_25x25.pdf)  
-   [24-word SeedQR template grid (29x29)](https://github.com/SeedSigner/seedsigner/blob/dev/docs/seed_qr/printable_templates/grid_29x29.pdf)  
-   [Baseball card template: 24-word SeedQR (29x29)](https://github.com/SeedSigner/seedsigner/blob/dev/docs/seed_qr/printable_templates/trading_card_29x29_w24words.pdf)  
  
CompactSeedQR templates:  
  
-   [12-word CompactSeedQR template dots (21x21)](https://github.com/SeedSigner/seedsigner/blob/dev/docs/seed_qr/printable_templates/dots_21x21.pdf)  
-   [24-word CompactSeedQR template dots (25x25)](https://github.com/SeedSigner/seedsigner/blob/dev/docs/seed_qr/printable_templates/dots_25x25.pdf)  
-   [12-word CompactSeedQR template grid (21x21)](https://github.com/SeedSigner/seedsigner/blob/dev/docs/seed_qr/printable_templates/grid_21x21.pdf)  
-   [24-word CompactSeedQR template grid (25x25)](https://github.com/SeedSigner/seedsigner/blob/dev/docs/seed_qr/printable_templates/grid_25x25.pdf)  
-   [Baseball card template: 12-word Compact SeedQR (21x21)](https://github.com/SeedSigner/seedsigner/blob/dev/docs/seed_qr/printable_templates/trading_card_21x21_w12words.pdf)  
-   [Baseball card template: 24-word Compact SeedQR (25x25)](https://github.com/SeedSigner/seedsigner/blob/dev/docs/seed_qr/printable_templates/trading_card_25x25_w24words.pdf)  
  
2-sided SeedQR templates - 8 per sheet Printing settings - (2-sided)("flip on long edge")("Actual Size") If printing on cardstock, adjust your printer settings via its control panel  
  
A4 templates(210mm * 297mm):  
  
-   [21x21 - stores 12-word seeds ONLY in CompactSeedQR format ONLY](https://github.com/SeedSigner/seedsigner/blob/dev/docs/seed_qr/printable_templates/21x21_A4_trading_card_2sided.pdf)  
-   [25x25 - stores 12-word or 24 word seeds depending on SeedQR format](https://github.com/SeedSigner/seedsigner/blob/dev/docs/seed_qr/printable_templates/25x25_A4_trading_card_2sided.pdf)  
-   [29x29 - stores 24-word seeds ONLY as plaintext SeedQR format ONLY](https://github.com/SeedSigner/seedsigner/blob/dev/docs/seed_qr/printable_templates/29x29_A4_trading_card_2sided.pdf)  
  
Letter templates(8.5in * 11in):  
  
-   [21x21 - stores 12-word seeds ONLY in CompactSeedQR format ONLY](https://github.com/SeedSigner/seedsigner/blob/dev/docs/seed_qr/printable_templates/21x21_letter_trading_card_2sided.pdf)  
-   [25x25 - stores 12-word or 24 word seeds depending on format](https://github.com/SeedSigner/seedsigner/blob/dev/docs/seed_qr/printable_templates/25x25_letter_trading_card_2sided.pdf)  
-   [29x29 - stores 24-word seeds ONLY as plaintext SeedQR format ONLY](https://github.com/SeedSigner/seedsigner/blob/dev/docs/seed_qr/printable_templates/29x29_letter_trading_card_2sided.pdf)  
  
----------  
  
# [](https://github.com/SeedSigner/seedsigner/blob/dev/README.md#manual-installation-instructions)Manual Installation Instructions  
  
see the docs:  [Manual Installation Instructions](https://github.com/SeedSigner/seedsigner/blob/dev/docs/manual_installation.md)