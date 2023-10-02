
# Creating AV Resistant Malware — Part 1

[](https://medium.com/@bortiz_8281?source=post_page-----7604b83ea0c0--------------------------------)

![Brendan Ortiz](https://miro.medium.com/v2/resize:fill:88:88/0*5FWF64j5me7Cs788.jpg)

[](https://blog.securityevaluators.com/?source=post_page-----7604b83ea0c0--------------------------------)

![Independent Security Evaluators](https://miro.medium.com/v2/resize:fill:48:48/1*zlPb8cRt0y45p5s2efL2uw.png)

[Brendan Ortiz](https://medium.com/@bortiz_8281?source=post_page-----7604b83ea0c0--------------------------------)

·

[Follow](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fsubscribe%2Fuser%2F9bcd9c69fcf0&operation=register&redirect=https%3A%2F%2Fblog.securityevaluators.com%2Fcreating-av-resistant-malware-part-1-7604b83ea0c0&user=Brendan+Ortiz&userId=9bcd9c69fcf0&source=post_page-9bcd9c69fcf0----7604b83ea0c0---------------------post_header-----------)

Published in

[](https://blog.securityevaluators.com/?source=post_page-----7604b83ea0c0--------------------------------)

Independent Security Evaluators

·

4 min read

·

Dec 1, 2020

17

1

Recently, I have been preparing for my eCPTXv2 certification from eLearnSecurity. The certification course covers a lot of advanced topics; including AV evasion. During preparation for my exam, I came across a scenario where I had code execution on a remote machine; however, the machine had an up-to-date anti-virus installation, thus I was unable to launch a Meterpreter shell without getting caught. To bypass the anti-virus on the target system, I needed to create a custom malware dropper that would evade the targets’ defense mechanisms. This blog is a technical tutorial on how to create a malware dropper using some of the advanced techniques I learned in my studies. As a disclaimer, it is important to remember that these techniques might help you bypass current Windows defense mechanisms; however, the best way to evade defenses is to create new malware! If it’s new and creative then it’s not likely to be caught by signature-based detection.

This blog will be broken up into 4 distinct parts. Each part contains the code and instructions necessary to build the malware pieces for that stage. Part 4 will include a short tutorial in executing the final pieces, as well as the normal building instructions.

# Part 1

We will be creating and executing this malware in an up-to-date version of Windows 10. To be precise, refer to the following image for the environment I will be working with.

![](https://miro.medium.com/v2/resize:fit:764/1*vQhxMgjG6JHwVZ_GdMdU8A.png)

Up-to-Date Version of Windows 10 Pro

We are also going to be working with the virus and threat protection settings depicted below. Note that we leave submission sampling off because we don’t want to submit samples to AV products before we even get to use them.

![](https://miro.medium.com/v2/resize:fit:1366/1*pPr87eMYM_clI_ggcfxoqw.png)

Windows Security Settings

Our malware is going to perform the following actions to bypass security mechanisms and remain undetected.

1.  We’re going to create a script to automate the downloading and executing of our malware. This is going to be stage 1.
2.  The first piece of malware loaded will be a custom-built DLL that will patch AMSI (Anti Malware Scan Interface) and ETW (Event Tracing for Windows). This disables Windows’ ability to detect our payload in memory. The payload from stage 1 will download the malicious DLL and use PowerShell reflection to load it into memory without the DLL ever touching the disk.
3.  The second piece of malware loaded will be another custom-built DLL that acts as a dropper for our final payload. The DLL will have classes and methods that will assist us in downloading an encrypted Meterpreter payload, decrypting, and executing in a process’ memory specified by us. This will be stage 3.
4.  The third piece of our malware will be an encrypted version of a Meterpreter payload. This is the end goal payload that gives us a shell onto the machine.

The first stage is the easiest. Using PowerShell’s IEX and WebClient functionality, we can automate this entire process. The script will look similar to the following:

![](https://miro.medium.com/v2/resize:fit:1400/1*4t3iVzwaetwByQhy4Es_vQ.png)

Stage 1 Payload

This script is simple, first, it defines a function called ‘load_dlls’. Within that function, it creates two ‘WebClient’ objects and uses the ‘downloadString’ method to download scripts from a web server we control. Then using the invoke-expression cmdlet, we can automatically execute the downloaded string. This all occurs without  _custom-script.ps1_ and  _load_remote_process_injector_dll.ps1_  without ever touching disk. They’re loaded straight into the PowerShell process memory and executed.

Once we get code execution on the remote machine, executing this payload is easy. We use the same technique we’re using in the ‘load_dlls’ function to download and execute the stage 1 payload.

In part 2 we will be talking about the technical details of the second stage of the malware. Please refer to the following article for more information: