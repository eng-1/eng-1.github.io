---
layout: post
title: Extracting Decrypted DLLs from Memory Dumps
categories: [Programming, Reverse, Engineering]
tags: [C++, memory, dump, dll, file]
---

 Many popular Android applications are made with the Unity 3D engine. Written in C#, the game's code, physics, and programming are packed into an assortment of assemblies (DLL files), often either encrypted or obfuscated. 


```
$ xxd ~/not_encrypted/Assembly-CSharp.dll | head -n 8
00000000: 4d5a 9000 0300 0000 0400 0000 ffff 0000  MZ..............
00000010: b800 0000 0000 0000 4000 0000 0000 0000  ........@.......
00000020: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000030: 0000 0000 0000 0000 0000 0000 8000 0000  ................
00000040: 0e1f ba0e 00b4 09cd 21b8 014c cd21 5468  ........!..L.!Th
00000050: 6973 2070 726f 6772 616d 2063 616e 6e6f  is program canno
00000060: 7420 6265 2072 756e 2069 6e20 444f 5320  t be run in DOS 
00000070: 6d6f 6465 2e0d 0d0a 2400 0000 0000 0000  mode....$.......
```

```
$ xxd ~/encrypted/Assembly-CSharp.dll | head -n 8
00000000: 6c44 f7c3 86f8 29c7 5a1b df82 18b7 cffc  lD....).Z.......
00000010: 9364 eed4 e5b7 0186 61f6 8a40 5396 b955  .d......a..@S..U
00000020: 4292 58a9 e7b3 a3bd db0c 85f1 4676 65e1  B.X.........Fve.
00000030: 61ac 9f3b f92f 59f1 f57c 9a03 4186 fee6  a..;./Y..|..A...
00000040: d579 9a87 fc20 03ca 4002 3da8 3f6c 964e  .y... ..@.=.?l.N
00000050: 2020 a325 b18d 485b 162f 1b5e d0dc 4845  Z..%..H[./.^....
00000060: 4c50 204d 4520 4920 414d 2053 5455 434b  ?..=.....SV.....
00000070: 2049 4e20 4120 5349 4d55 4c41 5449 4f4e  .Q9O.a....By....

```

 Seen above, the second hex dump shows a heavily encrypted DLL file. Lacking the notable `MZ` header, means we cannot decompile it unlike the first file. While it is certainly possible to dissasemble the rest of the game in order to search for the encryption algorithm, there is in fact a much easier, straightforward way to obtain a fully readable version of the `Assembly-CSharp.dll`.

 To do that, we will abuse a requirement of the Mono library which executes the code within those DLLs. Specifically, the fact that the game will only run if it is given a readable DLL file. This means that, during runtime, our encrypted DLL will be loaded, decrypted, and stored somewhere in RAM. After dumping the device's memory into a file, we are free to look over it, and search for the now fully-visible `MZ` signature. 

 However, since simply searching for `MZ` will surely bring up countless false positives, the search has to be intelligent enough to check if whether or not the `4D 5A` that it found really is the header of a valid DLL. This includes verifying the consitency of the MZ header, as well as checking for the existance of the mandatory [`PE` header](https://en.wikipedia.org/wiki/Portable_Executable). This of course, all requires *automation*. After all--engineering is 90% of reverse-engineering.

 Here is a C++ program that I wrote for this task:

[https://github.com/eng-1/DLLFileExtractor](https://github.com/eng-1/DLLFileExtractor/tree/master)

```bash
$ DLLFileExtractor <Memory_dump_path> <Output_directory> <Size_of_largest_DLL>
```

 To run this tool, simply enter into the command line the path to your memory dump file, the directory in which you want to output the separated DLL files, and file size to have each of them be. For now, this version of the tool isn't smart enough to figure out the size of the DLL automatically. As such, you need to manually supply a rough estimate of the largest possible size any DLL will be, which you can guess by checking the size of the encrypted files. Since the validity of the DLL format isn't affected by trailing grabage, you do not have to worry about over-estimating the size of it.


Having capturing your device's memory and processed it using the code provided above, you are now free to look at the many individual files that comrprise it in detail, and acquire the decrypted, readable version of our Assembly-CSharp.dll.
