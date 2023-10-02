# Creating AV Resistant Malware — Part 3

[](https://medium.com/@bortiz_8281?source=post_page-----fdacdf071a5f--------------------------------)

![Brendan Ortiz](https://miro.medium.com/v2/resize:fill:88:88/0*5FWF64j5me7Cs788.jpg)

[](https://blog.securityevaluators.com/?source=post_page-----fdacdf071a5f--------------------------------)

![Independent Security Evaluators](https://miro.medium.com/v2/resize:fill:48:48/1*zlPb8cRt0y45p5s2efL2uw.png)

[Brendan Ortiz](https://medium.com/@bortiz_8281?source=post_page-----fdacdf071a5f--------------------------------)

·

[Follow](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fsubscribe%2Fuser%2F9bcd9c69fcf0&operation=register&redirect=https%3A%2F%2Fblog.securityevaluators.com%2Fcreating-av-resistant-malware-part-3-fdacdf071a5f&user=Brendan+Ortiz&userId=9bcd9c69fcf0&source=post_page-9bcd9c69fcf0----fdacdf071a5f---------------------post_header-----------)

Published in

[](https://blog.securityevaluators.com/?source=post_page-----fdacdf071a5f--------------------------------)

Independent Security Evaluators

·

7 min read

·

Dec 1, 2020

6

In part 3 of this series, we will be covering creating another malicious DLL that we will load into memory using the PowerShell and .NET’s  _System.Reflection_  technique. This new DLL will be the loader for our final Meterpreter shellcode. After we disable AMSI this 3rd stage payload will act as a utility to download and execute shellcode that will download encrypted shellcode from a remote web-server, decrypt it, and then inject it into a specified process.

Begin by starting a new project in Visual Studio using the DLL template specified in Part 2.

Once the new project is created, we start by importing some libraries we’re going to need, just like the DLL in Part 2, create a Win32 class that defines some Windows API calls the DLL needs. If you’re ever stuck with defining Windows API calls, reference the following website:

[](https://www.pinvoke.net/index.aspx?source=post_page-----fdacdf071a5f--------------------------------)

## pinvoke.net: the interop wiki!

### PInvoke.net is primarily a wiki, allowing developers to find, edit and add PInvoke* signatures, user-defined types, and…

www.pinvoke.net

Pinvoke is a compilation of Windows API’s and how to define them in your programs.

In the image below we define several Windows APIs we will be using.

![](https://miro.medium.com/v2/resize:fit:2000/1*pJz-Ir-cQdfeyP_FpIyBFA.png)

Beginning of Win32 Class

We end our Win32 Class with definitions of memory and process protection flags.

![](https://miro.medium.com/v2/resize:fit:1330/1*2kkxg5hMdJZBsOocYtE7Zw.png)

End of Win32 Class

Next, we’re going to create our remote process injection and execution class.

![](https://miro.medium.com/v2/resize:fit:2000/1*BFpdR9uangzXMNrEl_iuXA.png)

rp_sc_execute class

The  _rp_sc_execute_  class has two functions defined,  _GetPid_() and  _inject_sc_(). The  _GetPid_() function is a simple function that takes as a parameter the name of a process and returns the process identifier (PID) that Windows uses to reference that process. The execution works as follows:

1.  An array of type Process named  _procs_  is initialized with all of the current processes on the system.
2.  Next, for each of those processes in the array, the following tasks occur. First, the process name is checked against our specified process name, if the result is true, the loop breaks and the ID of the checked process is returned. If the result of the check is false, the loop restarts with the next Process from the  _procs_  array. If no match is found, then PID is returned with a value of zero.

Next,  _inject_sc()_  takes as arguments, decrypted shellcode, and the process ID of the process to inject shellcode into. The function works as follows:

1.  A handle to the specified process is initialized using the  _OpenProcess_  function defined in our Win32 Windows API class. The arguments passed to  _OpenProcess_  are also defined in our Win32 class. These arguments allow us to read and write to the memory of this process handle.
2.  Next, we allocate memory in our targets process using the VirtualAllocEx Windows API with read write permissions.
3.  Next we write our decrypted shellcode to our newly allocated memory in the target process using the WriteProcessMemory Windows API.
4.  Finally, we use the CreateRemoteThread function to execute our decrypted shellcode in the specified process! So, for example, if we specify notepad, our shellcode will then attempt to inject and run under the first notepad process it finds.

Next, we will create our cryptography function that will handle all the encrypting and decrypting of our shellcode. The encrypt function will take unencrypted shell code downloaded from a remote web-server, encrypt it and then return it to the calling function to be output to a file on disk. The encrypt function is optional and can separate to save program size. However, for brevity, my DLL will contain this function. To use the function, you’ll want to create your shellcode then use this DLL to encrypt it. Move the encrypted shellcode onto your attacking machine, and then use the stage 1 payload to download and execute all the final payloads. All of the aforementioned actions are covered in Part 4.

![](https://miro.medium.com/v2/resize:fit:1400/1*m1ZzJznYw3PhJs9ORybpng.png)

Crypto Class w/ Encryption Function

The first function defined in our encryption class will take the shellcode bytes we want to encrypt and a password we use to encrypt the shellcode as arguments. The encrypt function returns the encrypted bytes to the calling function.

![](https://miro.medium.com/v2/resize:fit:1400/1*mryHw2HX2d6mvzT1Mn7vJw.png)

Cryptography Decryption Function

Next, we define the decryption function. It’s almost the same as the encryption function. It takes as arguments the encrypted shellcode bytes and the password used for the encryption process. The result of this function is the final shellcode that is injected in the process specified by the user.  
Next, we define a class and function to download our shellcode from a remote server. This one is pretty simple.

![](https://miro.medium.com/v2/resize:fit:2000/1*pHpGRLrdMpE-8EqC7PFlhA.png)

Downloader Function and Class

This function takes as an argument the URL to our either encrypted or decrypted shellcode. The purpose is to return a byte array that will either be encrypted and saved to our attacking machine or decrypted and injected into a remote process.

![](https://miro.medium.com/v2/resize:fit:1400/1*tfhM8VSkUN5Eggv_OZwenw.png)

PrintShellCode Function

Next, we have a function to print our encrypted shellcode. The reason why this might be important is that we can use this to hardcode our encrypted shellcode into our final binary. However, the function is not necessary for OPSEC considerations, thus it should be removed. If the DLL contains hardcoded shellcode, then the downloader function is not necessary.

Next, we have the driver for our DLL or the ‘main’ function that will be our execution point for this program. My DLL contains a  _get_help_() function; however, this is also not necessary and should be omitted. I will leave mine out of this blog for brevity.

First, a function named  _execute_prog_() is defined. The function takes three required arguments and four optional arguments. The required arguments are a password string and two boolean values, Encrypt and Decrypt. The optional parameters are a URL, a local file path, an out file path, and a specified remote process. The URL and local file path retrieve the shellcode. The out file path saves the encrypted shellcode to the specified file, while the remote process injects a process specified with the decrypted shellcode.

Thus, if Decrypt is true, the remote process is required, and either a URL or a local file path is required. If Encrypt is true, the out file path is required, and either a URL or a local file path is required. Please refer to the following picture for further details.

![](https://miro.medium.com/v2/resize:fit:2000/1*2nZwPGQr07ymGwEQ4mifIg.png)

execute_prog function and tasks.

The function starts by checking the values of  _Encrypt_  or  _Decrypt,_  which are the main deciders of what paths the function takes. Thus, if  _Encrypt_  is true, the function defines a few variables, which are all byte arrays. Then the function checks if a URL is specified or a local file path is specified. Both path arguments have the same result the difference is where they retrieve the shellcode to encrypt from. The  _Encrypt_  path will load the shellcode then encrypt it using the hashed password specified from the  _execute_prog_  function call. Finally, it will write all the encrypted bytes to a file and print them out to the screen.

Next, we have our Decrypt path, which performs almost the same execution steps. The difference is the function decrypts the shellcode then injects it into the process specified by  _remoteProcess_  instead of writing it to a file.

![](https://miro.medium.com/v2/resize:fit:1400/1*G_AqYyXORK3AX-dYAbIXUw.png)

The next step is to repeat the same Dotfuscated process that was covered in Part 2 and then create a stage 3 final PowerShell payload. For information on creating the payload, refer to Part 2 of this blog.  
The final payload should look similar to the following:

![](https://miro.medium.com/v2/resize:fit:1400/1*7qI6x-UgxBU6nUsaLjheJA.png)

Stage 3 Final Payload

This concludes Part 3 of this blog series. For a short tutorial of how to tie it all together, please refer to Part 4.