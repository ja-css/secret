
# Creating AV Resistant Malware — Part 4

[](https://medium.com/@bortiz_8281?source=post_page-----6cb2d215a50f--------------------------------)

![Brendan Ortiz](https://miro.medium.com/v2/resize:fill:88:88/0*5FWF64j5me7Cs788.jpg)

[](https://blog.securityevaluators.com/?source=post_page-----6cb2d215a50f--------------------------------)

![Independent Security Evaluators](https://miro.medium.com/v2/resize:fill:48:48/1*zlPb8cRt0y45p5s2efL2uw.png)

[Brendan Ortiz](https://medium.com/@bortiz_8281?source=post_page-----6cb2d215a50f--------------------------------)

·

[Follow](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fsubscribe%2Fuser%2F9bcd9c69fcf0&operation=register&redirect=https%3A%2F%2Fblog.securityevaluators.com%2Fcreating-av-resistant-malware-part-4-6cb2d215a50f&user=Brendan+Ortiz&userId=9bcd9c69fcf0&source=post_page-9bcd9c69fcf0----6cb2d215a50f---------------------post_header-----------)

Published in

[](https://blog.securityevaluators.com/?source=post_page-----6cb2d215a50f--------------------------------)

Independent Security Evaluators

·

3 min read

·

Dec 1, 2020

2

The final part of this blog series will be a short tutorial of how to use your new tools and how to tie it all together to get a reverse shell on an updated version of Windows 10 Pro with AV enabled!

First, you want to start with creating a reverse shell using M_sfvenom_.

![](https://miro.medium.com/v2/resize:fit:1400/1*Z5JtSvwx8xiQcviCjof_pA.png)

Creating a Reverse Shell Using Msfvenom

Next, host this shellcode on a web server to be downloaded and encrypted by your DLL.

![](https://miro.medium.com/v2/resize:fit:2000/1*L065esVdA8Z8z9n1Em-vHQ.png)

After the DLL is loaded into memory using the final payload created in Part 3, invoke  _execute_prog_  function with arguments to download and encrypt the shellcode created using  _Msfvenom_. Moving the shellcode onto a Windows machine before encrypting is detrimental because anti-virus will detect and block it. Msfvenom creates payloads with common signatures that are picked up by almost all anti-virus solutions. However, the encrypted version does not have the same problem. Thus, when  _execute_prog_  outputs the encrypted version onto a Windows machine with anti-virus, it is not removed.

The next step is to move the final payloads created in Parts 1, 2, and 3 to your machine hosting your web server. Once that is complete, move onto the victim machine that has updated anti-virus running. The machine used for this blog series is specified in Part 1.

Afterward, start a listener that will catch the Meterpreter shell call back once it’s been injected and executed.

![](https://miro.medium.com/v2/resize:fit:1400/1*Y0HPNE40uXMr0B7VuOYJBA.png)

Meterpreter Shell Listener

Now comes the fun part, to execute everything automatically, use PowerShell’s web client, and invoke expression to download and execute the payload from Part 1.

![](https://miro.medium.com/v2/resize:fit:1400/1*VEYsC3I1ZGIAVmwyM8eNRA.png)

Executing Final Payloads

The web server will be hit four separate times, for stages 1–4. All actions are performed automatically without any of the payloads created touching the disk. Additionally, the PowerShell Process that executes the stage 1 payload has AMSI and ETW patched.

![](https://miro.medium.com/v2/resize:fit:1400/1*DS9uxMxHkcrzXiRpItLx8w.png)

Web Server Receiving four Connections

Finally, when the remote shell code is injected into the notepad process and executed you will receive a call back on the listener you set up.

![](https://miro.medium.com/v2/resize:fit:1400/1*swNROfErYZISNBwGEpnraQ.png)

Getting Call Back

That’s it! You’ve just bypassed modern anti-virus to receive a reverse shell in an incredibly stealthy fashion. Thanks for reading!