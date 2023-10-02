
# Creating AV Resistant Malware — Part 2

[](https://medium.com/@bortiz_8281?source=post_page-----1ba1784064bc--------------------------------)

![Brendan Ortiz](https://miro.medium.com/v2/resize:fill:88:88/0*5FWF64j5me7Cs788.jpg)

[](https://blog.securityevaluators.com/?source=post_page-----1ba1784064bc--------------------------------)

![Independent Security Evaluators](https://miro.medium.com/v2/resize:fill:48:48/1*zlPb8cRt0y45p5s2efL2uw.png)

[Brendan Ortiz](https://medium.com/@bortiz_8281?source=post_page-----1ba1784064bc--------------------------------)

·

[Follow](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fsubscribe%2Fuser%2F9bcd9c69fcf0&operation=register&redirect=https%3A%2F%2Fblog.securityevaluators.com%2Fcreating-av-resistant-malware-part-2-1ba1784064bc&user=Brendan+Ortiz&userId=9bcd9c69fcf0&source=post_page-9bcd9c69fcf0----1ba1784064bc---------------------post_header-----------)

Published in

[](https://blog.securityevaluators.com/?source=post_page-----1ba1784064bc--------------------------------)

Independent Security Evaluators

·

8 min read

·

Dec 1, 2020

2

In part two of this series, we’re going to be creating the AV destroyer part of our malware! The DLL we will be building is based on Rasta-mouse’s ASBBypass DLL. However, we will be modifying this DLL to increase our level of stealth. The reason for that is most public offensive tooling on the internet will already be added to an AV database.

[](https://github.com/rasta-mouse/AmsiScanBufferBypass?source=post_page-----1ba1784064bc--------------------------------)

## rasta-mouse/AmsiScanBufferBypass

### ASBBypass.ps1 works for both x86 and x64 PowerShell. Before use, I recommend obfuscating strings such as Win32 to…

github.com

In this stage, we create a malicious DLL that will perform the following tasks:

1.  Disable AMSI.
2.  Tamper with Event Tracing for Windows.
3.  Evade detection while doing it.
4.  Create a script that loads our DLL into memory and then executes it.

If you’ve never made a DLL before, download the GitHub repo I linked above or open visual studio and select the C# Windows Class Library options. It’s important to note that AMSI has visibility into .NET starting from version 4.8, so it’s important to choose a .NET framework lower than that if you’re concerned about it.

![](https://miro.medium.com/v2/resize:fit:1400/1*7351_eJnm9EkdN-bsrkVoQ.png)

The malicious DLL Looks like the following:

![](https://miro.medium.com/v2/resize:fit:1400/1*zHwaVb5qqgHU5avVHd7KbQ.png)

Global Variables for our DLL.

First, we create a class named whatever you’d like; however, know that this will be how you invoke your DLL. The class defines a few global variables. The byte arrays are the bytes required to patch the x64 or x86 versions of ETW and the AMSI  _AmsiScanBuffer_  function. The AMSI patch works by inserting the value of AMSI_RESULT_CLEAN and then a return statement at the beginning of the  _AmsiScanBuffer_  function. This action means that the AmsiScanBuffer function has no opportunity to scan our code before it returns a CLEAN result. After the patch, our PowerShell process will be free from AMSI influence.

Disabling Event Tracing for Windows is also imperative because it will temporarily or permanently cease the flow of logging in the currently running process. It works by patching the  _EtwEventWrite_  function from ‘_ntdll.dll_’ that’s loaded in the PowerShell process memory. This patch is very similar to the AMSI patch.

The next functions are simple support functions. The first converts strings from base64 format to their ASCII representatives. This action is an attempt to hide trigger words from AV. The second function determines the current architecture the system is using. If the system is a 64-bit version of Windows it will return true, otherwise false.

![](https://miro.medium.com/v2/resize:fit:1400/1*w0ppJG-xWigOm6k-Wg81HA.png)

Supporting Functions

Our next two functions are the meat and bones of our malicious DLL. The first  _Bypass_() is our execution point that we will be calling when we want to bypass AMSI in PowerShell. The second  _PatchMem_  is the function we’ll be using to do the AMSI and ETW tampering.

![](https://miro.medium.com/v2/resize:fit:1400/1*NiD43uS55iW2VcMtZNK2GA.png)

Bypass and PatchMem functions

The bypass function simply calls our supporting  _is64bit_() function and if it returns true continues with our 64bit versions of our patches. If it’s not 64bit then it continues with the x86 version of our patches.

_PatchMem_  takes as arguments the patch bytes we’re using to set the  _AmsiScanBuffer_  function to return AMSI_RESULT_CLEAN the second it enters. Next, it will take the name of a library we want to open, in our case we want to open ‘_amsi.dll_’, which contains our  _AmsiScanBuffer_  function, and ‘_ntdll.dll_’ which contains the  _EtwEventWrite_  function. The next argument is the function we want to load in memory, our targets will be the functions just mentioned. Finally, an offset can be used to once again increase our level of stealth. Instead of calling the  _AmsiScanBuffer_  function directly, which will most definitely be picked up by AV, we can use relative addressing to call a function that won’t be picked up by AV and then increase our pointers’ value until we’re at our target functions. This also bypasses an ALSR problem. Instead of calling the  _AmsiScanBuffers_’ address directly, which will change every reboot, we can use an offset relative to the target modules base address. Hence the use of the offset variables in the ‘global variables’ screenshot above.

Finding the reliable relative addressing is above the scope of this blog. However, if interested DLL Export Viewer was the tool used for this action.

[](https://www.nirsoft.net/utils/dll_export_viewer.html?source=post_page-----1ba1784064bc--------------------------------)

## DLL Export Viewer - view exported functions list in Windows DLL

### See Also Description This utility displays the list of all exported functions and their virtual memory addresses for…

www.nirsoft.net

_PatchMem_  works by performing the following tasks:

1.  Gets a handle to  _amsi.dll_  by calling the LoadLibrary Windows API.
2.  Using the handle from LoadLibrary we then call the  _GetProcaddress_  Windows API with the name of the function we want to get a pointer to  _AmsiScanBuffer_.
3.  To patch bytes relative to the function pointer returned by  _GetProcAddress_  in step 2, we add the offset value to the function pointer.
4.  Code segments in memory are typically read only, therefore we need to change the permissions so we can edit the targeted region. This is done using the  _VirtualProtect_  Windows API call. The 0x40 stands for EXECUTE_READWRITE. Meaning we’re changing the area of memory to be executable, readable, and writable.
5.  Next, we use Windows  _Marshal’s_  _copy_  method to copy our patch into the region of memory we just changed permissions for.
6.  Finally, we revert the area of memory to its original memory protection state which should be READ_EXECUTE.

Now that we’ve defined the  _Bypass()_ and _PatchMem()_ functions we will create our supporting functions. The supportin functions include  _Win32_  which defines the Windows API calls we used in  _PatchMem_.

![](https://miro.medium.com/v2/resize:fit:1400/1*Sh5KEdauhdBPjqs4vgikTA.png)

PatchAmsi and Win32 Class

_PatchAmsi_  function takes as arguments the chosen patch which is determined in  _Bypass()_  and the offset we’re patching to avoid detection. To help avoid detection by AV we also obfuscate the target dll ‘_amsi.dll_’ by base64 encoding the name and decoding it upon use (more advanced methods may use variations of pattern scanning memory or enumerating loaded dlls and hashing their name, then using a hash value as a comparison — eliminating the possibility of AVs detecting us using suspicious strings).

PatchETW works in the same way, except we use a bit of a different detection avoidance approach. Instead of base64 encoding the string ‘_ntdll.dll_’ we split the string and have the program concatenate it during run time. Both functions are passed their respective target functions and offsets.

Finally, we define the Win32 class, this class contains our Windows API calls used in the Patch Memory Function. All that is required at this point is to compile on release!

Next, I want to introduce you to another defense evasion technique that is incredibly simple and yet super effective. VisualStudio supports a tool named DotFuscated which you can download using their Get Tools and Features option. It works by taking .NET assemblies and obfuscating the names of Types, Methods, and Fields. It’s simple, effective, and best of all free!

![](https://miro.medium.com/v2/resize:fit:574/1*S_zhaiKm7pBStyoqUVOXDg.png)

DotFuscated Results.

To use it, select the target assembly and press build! Once the assembly’s recompiled with the obfuscated names, it is that much less likely to trigger signature-based anti-virus solutions.

The next step is building the  _custom-script.ps1_  PowerShell script, which I introduced in Part 1. The final result of this script will be our entire stage 2 payload. The goal is to load the malicious DLL just created without it ever touching the disk. Therefore, the assembly needs to be in a format that can be read over HTTP and simultaneously loaded into memory. Thus, the following utility script to convert the assembly bytes into a base64 format is used.

![](https://miro.medium.com/v2/resize:fit:1018/1*cODcybihE4u8_UZmJCbxOA.png)

Base64 conversion utility script

This script takes as parameters an Infile, which is our malicious DotFuscated DLL, grabs the contents of it with a Byte set as the encoding, and converts the contents to base64. If OutFile is specified then it will put the base64 string to a file, otherwise, it will print it to the screen.

Once we have our base64 encoded DLL we can use PowerShell to call .NET’s  _System.Reflection_  method to load our assembly without it ever touching disk! If you remember our payload is going to be downloaded and invoked over HTTP. The final payload for our second stage looks like the following:

![](https://miro.medium.com/v2/resize:fit:1400/1*ygC_oHlKzbBCliqZtDTt-Q.png)

Final Stage 2 Payload

(After publishing I found that PowerView has functionality to compress and base64 encode your DLL and will create a script to load it automatically. If interested please refer to the following URL:  [https://powersploit.readthedocs.io/en/latest/ScriptModification/Out-CompressedDll/](https://powersploit.readthedocs.io/en/latest/ScriptModification/Out-CompressedDll/))

This payload is incredibly simple, the DLL was the complex and heavy lifter. Once the DLL is loaded into memory using the reflection technique, we can call our  _Amsi_  class we defined in our DLL and use the  _Bypass_() function. If you recall the  _Bypass_() function is our execution point for our DLL.

Here is an example of how this works, I will be executing the string “amsiscanbuffer” because this is a known malicious string for AMSI. Once we execute our script our PowerShell process will no longer trigger AV protection mechanisms.

![](https://miro.medium.com/v2/resize:fit:1300/1*M5QYwPRtFq42RmCibYX_3g.png)

Example of the Bypass Payload Working.

That concludes Part 2 I hope you enjoyed it. Part 3’s objective will be creating the malicious DLL that acts as a dropper for our final encrypted shell code.