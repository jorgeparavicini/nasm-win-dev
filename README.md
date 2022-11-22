# NASM Windows Dev

This document explains how to setup a Windows development environment for NASM.
The reason I create this doc, is because I have not found any good up-to-date documentation
that explains how to start writing assembly on Windows from A to Z including a debugger and syntax highlighting.

## Features

After installing the environment will contain following features:

* Visual Studio 2022 - used to develop and debug
* NASM compiler
* Debugger
* Syntax Highlighting
* Intellisense
* Automatic debug symbols in debug mode

## Installation

1) Download [Visual Studio](https://visualstudio.microsoft.com/vs/)
2) In the installer make sure to include `Desktop development with C++`

![Installer example](img/Screenshot%202022-11-22%20184809.png)

3) Download the latest version of NASM from their [website](https://www.nasm.us/).
   You can download either the Installer or the executable only.
4) Extract the downloaded file to a location. In my case this will be `A:\nasm` 
5) Now copy the extracted path and create a new environment variable called `NASMPATH` with the copied value. 
Make sure to add a trailing `\` to the path. In my case the final value is `A:\nasm\`.
6) Download [VSNASM](https://github.com/ShiftMediaProject/VSNASM). This project adds some build customisations 
for Visual Studio. It says it's only available up to VS2019 but VS2022 is also supported.
7) Execute the `install_script.bat` with elevated privileges or follow their manual installation.
8) Download and install the `ASMDude` Visual Studio extension.
   1) At the time of writing, the extension is not ready for VS2022 yet.
   However, a preview can be downloaded from [here](https://github.com/HJLebbink/asm-dude/files/7822110/AsmDude-vs2022.zip).
   Follow [this github issue](https://github.com/HJLebbink/asm-dude/issues/128) to find out more about VS 2022 support.

## Setup

Now we have installed everything we need to get started with a new project.

1) Create a new empty C++ project
   
![Empty Project](img/Screenshot%202022-11-22%20190319.png)

2) Give the project a name and create it.

3) I recommend changing the solution explorer view to show all files by clicking the button `Show all files`.

![show all files button](img/Screenshot%202022-11-22%20190550.png)

4) Now update the build sequence by right clicking the project > `Build Dependencies` > `Build Customizations...`

![build customizations](img/Screenshot%202022-11-22%20190643.png)

5) Select the `nasm(.targets, .props)` value
   
![nasm](img/Screenshot%202022-11-22%20191017.png)

6) Next, create a new `C++ File (.cpp)` called `main.asm`. It's important that the extension is named `.asm`.

![new file](img/Screenshot%202022-11-22%20191130.png)

7) Right click on the new file > Properties. Under `General` set the `Item Type` to `Netwide Assembler`. In my case it is called `NASM - Netwide Assembler` because I renamed it.
   
![item type](img/Screenshot%202022-11-22%20193154.png)

> If the `Netwide Assembler` is missing. Make sure the vcxproj file contains the necessary dependencies.
> The `ExtensionTargets` needs: `$(VCTargetsPath)\BuildCustomizations\nasm.targets`
> The `ExtensionSettings` needs: `$(VCTargetsPath)\BuildCustomizations\nasm.props`

8) If you run it now you the build will succeed. 
However, the linking stage will probably run into an error if you use some basic symbols.

The following basic code requires `printf` and `ExitProcess` to be available, but they are not linked yet.

```assembly
bits 64
default rel

segment .data
	msg db "Hello world!", 0xd, 0xa, 0

segment .text
global main
extern ExitProcess

extern printf

main:
	push	rbp
	mov		rbp, rsp
	sub		rsp, 32

	lea		rcx, [msg]
	call	printf

	xor		rax, rax
	call	ExitProcess
```

To include the required libraries, right click the project and go to `Properties` > `Configuration Properties` >
`Linker` > `Input` and add `legacy_stdio_definitions.lib;msvcrt.lib;` to the `Additional Dependencies`.

![Additional Dependencies](img/Screenshot%202022-11-22%20194954.png)

Click `Ok` and `Apply`

9) If you run it now, the code should work.
10) To get the debugger to work you can either manually add the debug flag to each `.asm` file by 
right clicking each file and going to `Properties` > `Netwide Assembler` > `Assembler Options` 
and setting `Generate Debug Information` to `Yes (-g)`.
However, this can get tiresome to do for each file. To do this automatically for every `.asm` file close 
Visual Studio and open the `.vcxproj` file in a text editor. 
Now create an `<ItemGroup>` right under the root `<Project>` tag and set following:

```xml
<ItemGroup>
  <NASM Include="*.asm">
    <FileType>Document</FileType>
    <GenerateDebugInformation Condition="'$(Configuration)|$(Platform)'=='Debug|x64'">true</GenerateDebugInformation>
    <GenerateDebugInformation Condition="'$(Configuration)|$(Platform)'=='Release|x64'">false</GenerateDebugInformation>
  </NASM>
</ItemGroup>
```

This will add the debug flag for every `.asm` file if the project is built with the `debug` configuration.
If the `release` configuration is used the files will be compiled without debug symbols.

## Further reading

To get started with NASM for windows I can recommended following tutorial: https://sonictk.github.io/asm_tutorial/