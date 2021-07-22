# LittleCorporal
LittleCorporal: A C# Automated Maldoc Generator
```
C:\LittleCorporal\bin\Debug>LittleCorporal.exe C:\Users\ANON\Desktop\beacon.bin explorer.exe
.____    .__  __    __  .__         _________                                         .__
|    |   |__|/  |__/  |_|  |   ____ \_   ___ \  _________________   ________________  |  |
|    |   |  \   __\   __\  | _/ __ \/    \  \/ /  _ \_  __ \____ \ /  _ \_  __ \__  \ |  |
|    |___|  ||  |  |  | |  |_\  ___/\     \___(  <_> )  | \/  |_> >  <_> )  | \// __ \|  |__
| _______ \__||__|  |__| |____/\___  >\______  /\____/|__|  |   __/ \____/|__|  (____  /____/
        \/                        \/        \/             |__|                     \/


             ________
            /        \
         __ /       (o)\__
        /     ______\   \                   I am the strongest debater in the whole Conseil.
        | ____/__  __\____|                 I let myself be attacked, because I know how to defend myself.
           [  --~~--  ]                                                              - Napoleon Bonaparte
            | (  L   )|
      ___----\  __  /----___
     /   |  < \____/ >   |   \
    /    |   < \--/ >    |    \
    ||||||    \ \/ /     ||||||
    |          \  /   o       |
    |     |     \/   === |    |
    |     |      |o  ||| |    |
    |     \______|   +#* |    |
    |            |o      |    |
     \           |      /     /
     |\__________|o    /     /
     |           |    /     /

[+] Parsed Arguments:
   [>] Shellcode Path: C:\Users\ANON\Desktop\beacon.bin
   [>] Target Process: explorer.exe

[+] Embedded shellcode in Loader.cs!
[+] Generated C# Loader artifact!
[+] Ran the .NET assembly through Donut!
[+] Donut artifact is located at: C:\Users\ANON\Desktop\LittleCorporal\Artifacts\payload.bin
[+] Path to Word document: C:\Users\ANON\Desktop\LittleCorporal\Artifacts\Form.doc
```
## How It Works?
LittleCorporal accepts a user-supplied argument for a process to inject into on a _remote_ machine, in which you plan to execute the malicious Word document on, and also accepts a path to a local shellcode file stored in `.bin` format - such as a Beacon Stageless shellcode blob on the machine you are running LittleCorporal from. So, if you would like to use the maldoc generated from this project, you will need to specify an already running process on the machine you intend to run the maldoc on (be it the local machine or a different machine. `explorer.exe` is always going to have one instance, so use this if you do not care about which process you inject into).

LittleCorporal embeds the shellcode and the target process into `Loader.cs` and then utilizes [thread hijacking](https://connormcgarr.github.io/thread-hijacking/) to perform remote process injection. LittleCorporal then compiles the updates `Loader.cs` into a .NET `.exe` artifact and then sends the artifact through [Donut](https://github.com/TheWover/donut) to generate position independent shellcode, which will execute the `.exe`. The shellcode generated by Donut is then base64 encoded, a Word document is generated, and the final Donut blob is stored in an [InlineShape.AlternativeText](https://docs.microsoft.com/en-us/office/vba/api/word.inlineshape.alternativetext) Word property, which is able to hold the entire payload. This is done by inserting an image (currently a blank image, giving the document a "blank" look) property into the Word document, as alternative text on the image, which contains the payload. LittleCorporal then leverages a VBA "template", contained in this project as a text file, and injects this Macro into the newly generated Word document. The Macro is named `autoopen`, so it opens upon the document opening, and then is configured to extract the value of the alternative text of the previously generated image, which contains the final payload, base64 decodes it, and finally uses Windows API calls, in VBA, to perform _local_ injection into Word. In essence, this project uses a simple "loader" in VBA to perform local injection into Word, which is a bit less scrutinized than remote process injection, and then uses execution from the simple local injection injection to execute the Donut shellcode, which is _another_ loader that performs thread hijacking for the final remote process injection of the user-supplied shellcode into the user supecified process. This is all done in an automated fashion, including generation of the Word document.

## Usage
1. __YOU MUST FETCH THE ENTIRE PROJECT IN ORDER TO USE__! LittleCorporal uses relative paths for additional resources, such as Donut.
2. Once obtaining the entire project, change your working directory to the `bin\Debug` directory (`cd C:\Path\to\LittleCorporal\bin\Debug`)
3. The thread hijacking code first performs a check if the machine executing the Word document is domain joined. If running on a non-domain joined machine, please edit [this line of code](https://github.com/connormcgarr/LittleCorporal/blob/main/LittleCorporal/LittleCorporal.Loader/Loader.cs#L266) to `bool FUNC1 = true;` before running `LittleCorporal.exe` to generate a Word document
4. Specify the path to your shellcode file on the machine you are executing `LittleCorporal.exe` from and the already running process on the machine which you would like to execute the Word document on. (`LittleCorporal.exe C:\Path\To\Shellcode.bin explorer.exe`)
5. LittleCorporal will then output the path to the final Word document
