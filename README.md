# Shoggoth

<img src="https://github.com/frkngksl/Shoggoth/blob/main/img/logo.png?raw=true">

<div align="center">
 <a href="https://twitter.com/R0h1rr1m">
    <img src="https://img.shields.io/badge/License-MIT-green">
 </a>
 <a href="https://github.com/frkngksl/Shoggoth/issues">
    <img src="https://img.shields.io/github/issues/frkngksl/Shoggoth">
 </a>
 <a href="https://github.com/frkngksl/Shoggoth">
    <img src="https://img.shields.io/github/stars/frkngksl/Shoggoth?color=green&style=flat-square">
 </a>
<a href="https://twitter.com/R0h1rr1m">
    <img src="https://img.shields.io/twitter/follow/R0h1rr1m?style=social">
 </a>
</div>

**Presented at**
- [BlackHat Europe 2022 Arsenal](https://www.blackhat.com/eu-22/arsenal/schedule/index.html#shoggoth-asmjit-based-polymorphic-encryptor-29588)

# Introduction

## Description

Shoggoth is an open-source project based on C++ and asmjit library used to encrypt given shellcode, PE and COFF files polymorphically. 

Shoggoth will generate an output file that stores the payload and its corresponding loader in an obfuscated form. Since the content of output is position-independent, it can be executed directly as a shellcode. While the payload is executing, it decrypts itself at the runtime. In addition to encryption routine, Shoggoth also adds garbage instructions, that change nothing, between routines.

I started to develop this project to study different dynamic instruction generation approaches, assembly practices, and signature detections. I am planning to regularly update the repository with my new learnings.

# Features
## Execution Flow

General execution flow of Shoggoth for an input file can be seen in the image below. You can observe this flow with the default configurations. 

<img src="https://github.com/frkngksl/Shoggoth/blob/main/img/ShoggothExecutionFlow.png">

Basically, Shoggoth first merges the precompiled loader shellcode according to the chosen mode (COFF or PE file) and the input file. It then adds multiple garbage instructions it generates to this merged paylaod. The stub containing the loader, garbage instruction, and the payload is encrypted first with RC4 encryption and then with randomly generated block encryption by combining corresponding decryptors. Finally, it adds a garbage instruction to the resulting block.

## Machine Code Generation

While Shoggoth randomly generating instructions for garbage stubs or encryption routines, it uses [AsmJit](https://asmjit.com/) library. 

AsmJit is a lightweight library for machine code generation written in C++ language. It can generate machine code for X86, X86_64, and AArch64 architectures and supports baseline instructions and all recent extensions. AsmJit allows specifying operation codes, registers, immediate operands, call labels and embedding arbitrary values to any offset inside the code. While generating some assembly instructions by using AsmJit, it is enough to call the API function that corresponds to the required assembly operation with assembly operand values from the Assembler class. For each API call, AsmJit holds code and relocation information in its internal CodeHolder structure. After calling API functions of all assembly commands to be generated, its JitRuntime class can be used to copy the code from CodeHolder into memory with executable permission and relocate it.

While I was searching for a code generation library, I encountered with AsmJit, and I saw that it is widely used by many popular projects. That's why I decided to use it for my needs. I don't know whether Shoggoth is the first project that uses it in the red team context, but I believe that it can be a reference for future implementations.

## COFF and PE Loaders

Shoggoth can be used to encrypt given PE and COFF files so that both of them can be executed as a shellcode thanks to precompiled position-independent loaders. I simply used *C to Shellcode* method to obtain PIC version of well known PE and COFF loaders I modified for my old projects. For compilation, I used the Makefile from [HandleKatz](https://github.com/codewhitesec/HandleKatz) project which is a LSASS dumper in PIC form.

Basically, in order to obtain shellcode with the C to Shellcode technique, I removed all the global variables in the loader source code, made all the strings stored in the stack, and resolved the Windows API functions' addresses by loading and parsing the necessary DLLs at runtime. Afterward, I determined the entry point with a linker script and compiled the code by using MinGW with various compilation flags. I extracted the .text section of the generated executable file and obtained the loader shellcode. Since the executable file obtained after editing the code as above does not contain any sections other than the .text section, the code in this section can be used as position-independent.

## RC4 Cipher

## Random Block Cipher

## Garbage Instruction Generation

# Usage

## Command Line Usage

```

  ______ _                                  _
 / _____) |                             _  | |
( (____ | |__   ___   ____  ____  ___ _| |_| |__
 \____ \|  _ \ / _ \ / _  |/ _  |/ _ (_   _)  _ \
 _____) ) | | | |_| ( (_| ( (_| | |_| || |_| | | |
(______/|_| |_|\___/ \___ |\___ |\___/  \__)_| |_|
                    (_____(_____|

                     by @R0h1rr1m

                "Tekeli-li! Tekeli-li!"

Usage of Shoggoth.exe:

    -h | --help                             Show the help message.
    -v | --verbose                          Enable a more verbose output.
    -i | --input <Input Path>               Input path of payload to be encrypted. (Mandatory)
    -o | --output <Output Path>             Output path for encrypted input. (Mandatory)
    -s | --seed <Value>                     Set seed value for randomization.
    -m | --mode <Mode Value>                Set payload encryption mode. Available mods are: (Mandatory)
                                                [*] raw - Shoggoth doesn't append a loader stub. (Default mode)
                                                [*] pe - Shoggoth appends a PE loader stub. The input should be valid x64 PE.
                                                [*] coff - Shoggoth appends a COFF loader stub. The input should be valid x64 COFF.
    --coff-arg <Argument>                   Set argument for COFF loader. Only used in COFF loader mode.
    -k | --key <Encryption Key>             Set first encryption key instead of random key.
    --dont-do-first-encryption              Don't do the first (stream cipher) encryption.
    --dont-do-second-encryption             Don't do the second (block cipher) encryption.
    --encrypt-only-decryptor                Encrypt only decryptor stub in the second encryption.

```

# What does Shoggoth mean?
<p align="center">
<img width="800" height="500" src="https://github.com/frkngksl/Shoggoth/blob/main/img/shoggoth.jpg?raw=true">
</p>

<br>

> "It was a terrible, indescribable thing vaster than any subway train—a shapeless congeries of protoplasmic bubbles, faintly self-luminous, and with myriads of temporary eyes forming and un-forming as pustules of greenish light all over the tunnel-filling front that bore down upon us, crushing the frantic penguins and slithering over the glistening floor that it and its kind had swept so evilly free of all litter."
> ~ H. P. Lovecraft, At the Mountains of Madness

<br>

A Shoggoth is a fictional monster in the Cthulhu Mythos. The beings were mentioned in passing in H. P. Lovecraft's sonnet cycle Fungi from Yuggoth (1929–30) and later described in detail in his novella At the Mountains of Madness (1931). They are capable of forming whatever organs or appendages they require for the task at hand, although their usual state is a writhing mass of eyes, mouths and wriggling tentacles.

Since these creatures are like a sentient blob of self-shaping, gelatinous flesh and have no fixed shape in Lovecraft's descriptions, I want to give that name to a Polymorphic Encryptor tool. :slightly_smiling_face: 

# References

- https://github.com/EgeBalci/sgn
- https://github.com/asmjit/asmjit
- https://www.pelock.com/articles/polymorphic-encryption-algorithms
- https://github.com/codewhitesec/HandleKatz
- https://github.com/vxunderground/MalwareSourceCode
- https://www.nayuki.io/page/rc4-cipher-in-x86-assembly
- https://www.deviantart.com/halycon450/art/Shoggoth-914584713

# Disclaimer

I shared this tool only for showing the code snippets of well known TTPs. I'm not responsible for the use of this tool for malicious activities.
