About BIOS bricked GPD, Corona and happy end  
  
**April 2018**  - After buying, I've decided to update bios and make my GDP Pocket(first gen) bricked. Be very careful with this, read all system messages, be sure that your update was successful... But if you reading this, perhaps, your bios update have already unsuccessful. After some googling time I've found potential solution - CH341a programmer & clips to connect to bios chip and flash it,  [thanks to](https://www.reddit.com/r/GPDPocket/comments/6x6az9/bricked_bios_assistance/)  
  
**May 5th year 2018**  - I've tried everything, bought several clips and ect. And after several times I've stopped my tries to fix BIOS on my GDP Pocket(1st gen).  
  
**Year 2020**  - Corona-virus Era! My GDP pocket still bricked, hands with me, I have enough time...  
  
**November 13 year 2020 - I've finally fixed it! That's how i did it:**  
  
-1. Find out what bios chip you've met. 1st gen GDP Pocket produced at leas with two different - Winbond 25Q64FWS1G or Gigadevice 25LQ64CVIG. You can find this info on chip.  
  
-2. Read datasheet for this chip!!! Define your connection scheme and voltage - 1st pin marked by circle.  [For 25LQ64CVIG voltage is 1,8.](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&ved=2ahUKEwi3hMef0aPtAhUFkMMKHf6nA40QFjAAegQIAxAC&url=https%3A%2F%2Fwww.gigadevice.com%2Fdatasheet%2Fgd25lq64c%2F&usg=AOvVaw2RY4Fk1v-AZ5wJJooD_V99)  
  
-3. To flash you will need the programmer and software:  
  
-   I've used CH3141A with 1.8V adapter SPI Flash SOP8 DIP8 (links below)  
      
-   As software I'm recommend this -  [NeoProgrammer](https://4pda.ru/forum/index.php?showtopic=884713&st=3840#entry96411343)  it is more stable than native Chinese.  
      
  
-4. Be sure that you have good contact to your chip!!! My first tries have failed due this mistake. I've spent about a week for different experiments before admit my defeat and back to this issue in COVID circumstances after 2 years. Clips quality was very bad, it was hard to understand that it really connected.  [So thank this guy for the good idea with sponge](https://techtablets.com/forum/topic/cube-i9-how-to-flash-and-recover-a-failed-bios-flash-or-bricked-bios/)  In my case I've used 1 sponge in half, one paperclip in half, and two dumbbells.  
  
-5. After 30 minutes for connection, I've finally got it! Programmer should show you your model of chip. If not for 99% you have the problems with connection.  
  
-6. Erase and flash new bios - it takes about 2 minutes.  
  
**What I've used:**  
  
1.  CH3141A 24 25 Series EEPROM Flash USB Programmer with 1.8V adapter SPI Flash SOP8 DIP8l  
      
2.  [How to tier down](https://www.youtube.com/watch?v=iTymxvORuZA&t=181s)  
      
3.  This soft for programming -  [NeoProgrammer](https://4pda.ru/forum/index.php?showtopic=884713&st=3840#entry96411343)  
      
4.  [official site with firmware](https://gpd.hk/gpdpocketfirmware)  
      
  
**What links was helpful:**  
  
-   [https://4pda.ru/forum/index.php?showtopic=884713&st=40](https://4pda.ru/forum/index.php?showtopic=884713&st=40)  
      
-   [https://www.reddit.com/r/GPDPocket/comments/78hovy/unbrick_gpd_pocket_flash_bios_eeprom_by_usb/](https://www.reddit.com/r/GPDPocket/comments/78hovy/unbrick_gpd_pocket_flash_bios_eeprom_by_usb/)  
      
-   [https://kb.isn.cz/doku.php/gpd-pocket#unbrick-gpd-pocket-flash-eeprom-bios-by-usb-programmer](https://kb.isn.cz/doku.php/gpd-pocket#unbrick-gpd-pocket-flash-eeprom-bios-by-usb-programmer)  
      
-   [https://forum.xda-developers.com/showpost.php?p=66188047&postcount=14](https://forum.xda-developers.com/showpost.php?p=66188047&postcount=14)  
      
-   [https://techtablets.com/forum/topic/cube-i9-how-to-flash-and-recover-a-failed-bios-flash-or-bricked-bios/](https://techtablets.com/forum/topic/cube-i9-how-to-flash-and-recover-a-failed-bios-flash-or-bricked-bios/)  
      
-   [https://www.reddit.com/r/GPDPocket/comments/6x6az9/bricked_bios_assistance/](https://www.reddit.com/r/GPDPocket/comments/6x6az9/bricked_bios_assistance/)