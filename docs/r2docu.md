# r2docu (WIP)

## Welcome
This documentation is aimed to be a quick and easy-to-follow first contact to radare2 (r2) so you can have it installed and start using it in few minutes.

It is not meant to replace the comprehensive guides and explanations from the r2book neither a full reference of every single feature, but a place where newcomers can feel and put in practice the r2 power from the very beginning.

### What is radare2
In a nutshell, r2 is a free and open source reverse engineering framework built from scratch in C without any third-party dependency that runs on very major (and many others) [platforms] and has native support for a lot of [file formats]. It can be used as is or you can built your own tools on top of r2 using your favourite scripting language.

### What can radare2 do
At a first glance, here you are a non-exhaustive list of things that r2 can be useful for:
- Disassemble binaries of several [architectures] and [operating systems]
- Analise code, data, references and structures.
- Binary manipulation, code injection, patching, bindiffing.
- Mount filesystems, detect partitions, carve for data.
- Extract information and metrics that can be used for binary classification.
- Kernel analysis and debugging.

## Installation
The recommended way to install radare2 is to get the last git version.
```
$ git clone https://github.com/radare/radare2
$ cd radare2
$ ./sys/install.sh # Run this to automatically pull and update
```
There also exist precompiled binaries for OS X and Windows.
Windows download: [here]
OS X download: [here]

You can also build it by yourself. Check [build instructions] for different platforms.


## Quick start
<!--- merge with below somehow? --->
You can follow step-by-step the following commands.

### Open a file
```
$ r2 /bin/ls
```
(You can prefix `-w` for opening in write mode or `-d` for opening in debug mode)

### Basic commands
Commands in r2 follow simple mnemonic rules.

`s` -> **s**eek [to (relative) address]
```
[0x00005850]> s 0x10
[0x00000010]> s 0
[0x00000000]> s 0x5850
[0x00005850]> s + 0x100
[0x00005950]>
```

`px` -> **p**rint he**x**dump [number of bytes]
```
[0x00005850]> px 64
- offset -   0 1  2 3  4 5  6 7  8 9  A B  C D  E F  0123456789ABCDEF
0x00005850  31ed 4989 d15e 4889 e248 83e4 f050 544c  1.I..^H..H...PTL
0x00005860  8d05 ca0a 0100 488d 0d53 0a01 0048 8d3d  ......H..S...H.=
0x00005870  1ce6 ffff ff15 5ea7 2100 f40f 1f44 0000  ......^.!....D..
0x00005880  488d 3de1 a921 0055 488d 05d9 a921 0048  H.=..!.UH....!.H
[0x00005850]>
```

`pd` -> **p**rint **d**isassembly [number of instructions]
```
[0x00005850]> pd 5
            0x00005850      31ed           xor ebp, ebp
            0x00005852      4989d1         mov r9, rdx
            0x00005855      5e             pop rsi
            0x00005856      4889e2         mov rdx, rsp
            0x00005859      4883e4f0       and rsp, 0xfffffffffffffff0
[0x00005850]>
```

`q` -> **q**uit
```
[0x00005850]> q
$
```

Now we will have to open the file with write access. As we don't want to mess with system binary, let's make a copy before.

```
$ cp /bin/ls /tmp/ls
$ r2 /tmp/ls
```

`wx` -> **w**rite he**x**pairs
```
[0x00005850]> px 16
- offset -   0 1  2 3  4 5  6 7  8 9  A B  C D  E F  0123456789ABCDEF
0x00005850  31ed 4989 d15e 4889 e248 83e4 f050 544c  1.I..^H..H...PTL
[0x00005850]> wx 1337
[0x00005850]> px 16
- offset -   0 1  2 3  4 5  6 7  8 9  A B  C D  E F  0123456789ABCDEF
0x00005850  1337 4989 d15e 4889 e248 83e4 f050 544c  .7I..^H..H...PTL
[0x00005850]>
```

`wa` -> **w**rite **a**ssembly
```
[0x00005850]> pd 5
            0x00005850      1337           adc esi, dword [rdi]
            0x00005852      4989d1         mov r9, rdx
            0x00005855      5e             pop rsi
            0x00005856      4889e2         mov rdx, rsp
            0x00005859      4883e4f0       and rsp, 0xfffffffffffffff0
[0x00005850]> s + 5
[0x00005855]> pd 1
            0x00005855      5e             pop rsi
[0x00005855]> wa nop
Written 1 byte(s) (nop) = wx 90
[0x00005855]> pd 1
            0x00005855      90             nop
[0x00005855]> s - 5
[0x00005850]> pd 5
            0x00005850      1337           adc esi, dword [rdi]
            0x00005852      4989d1         mov r9, rdx
            0x00005855      90             nop
            0x00005856      4889e2         mov rdx, rsp
            0x00005859      4883e4f0       and rsp, 0xfffffffffffffff0
[0x00005850]> 
```

`aa` -> **a**nalyse **a**ll
```
[0x00005850]> aa
[x] Analyze all flags starting with sym. and entry0 (aa)
[0x00005850]> 
```


### Basic modifiers
- Append `?` to show subcommands and get inline help
```
[0x00005850]> wx?
|Usage: wx[f] [arg]
| wx 9090     write two intel nops
| wxf -|file  write contents of hexpairs file here
| wxs 9090    write hexpairs and seek at the end
[0x00005850]> pd?
|Usage: p[dD][ajbrfils] [sz] [arch] [bits] # Print Disassembly
| NOTE: len  parameter can be negative
| NOTE:      Pressing ENTER on empty command will repeat last print command in next page
| pd N       disassemble N instructions
| pd -N      disassemble N instructions backward
| pD N       disassemble N bytes
| pda        disassemble all possible opcodes (byte per byte)
| pdb        disassemble basic block
| pdc        pseudo disassembler output in C-like syntax
| pdC        show comments found in N instructions
| pdf        disassemble function
| pdi        like 'pi', with offset and bytes
| pdj        disassemble to json
| pdJ        formatted disassembly like pd as json
| pdk        disassemble all methods of a class
| pdl        show instruction sizes
| pdp        disassemble by following pointers to read ropchains
| pdr        recursive disassemble across the function graph
| pdr.       recursive disassemble across the function graph (from current basic block)
| pdR        recursive disassemble block size bytes without analyzing functions
| pds[?]     disassemble summary (strings, calls, jumps, refs) (see pdsf and pdfs)
| pdt        disassemble the debugger traces (see atd)
[0x00005850]> 
```

(we have a subcommand of `pd` called `pdf` to **p**rint **d**isassembly of **f**unction that we are going to use just right now)

- Use `@` for temporary seek.
As we already ran `aa` to analyse all, we have flags for identified functions' offsets.
```
[0x00005850]> pdf @ main
            ;-- section..text:
┌ (fcn) main 686
│   main (int argc, char **argv, char **envp);
│           ; var int local_8h @ rsp+0x8
│           ; var int local_10h @ rsp+0x10
│           ; var int local_28h @ rsp+0x28
│           ; var int local_30h @ rsp+0x30
│           ; var int local_32h @ rsp+0x32
│           ; var int local_38h @ rsp+0x38
│           ; var int local_45h @ rsp+0x45
│           ; var int local_46h @ rsp+0x46
│           ; var int local_47h @ rsp+0x47
│           ; var int local_48h @ rsp+0x48
│           ; arg int argc @ rdi
│           ; arg char **argv @ rsi
│           ; DATA XREF from entry0 (0x586d)
│           0x00003e90      4157           push r15                    ; [14] -r-x section size 74969 named .text
│           0x00003e92      4156           push r14
│           0x00003e94      4155           push r13
│           0x00003e96      4154           push r12
│           0x00003e98      55             push rbp
│           0x00003e99      53             push rbx
│           0x00003e9a      89fd           mov ebp, edi                ; argc
│           0x00003e9c      4889f3         mov rbx, rsi                ; argv
│           0x00003e9f      4883ec58       sub rsp, 0x58               ; 'X'
│           0x00003ea3      488b3e         mov rdi, qword [rsi]        ; argv
│           0x00003ea6      64488b042528.  mov rax, qword fs:[0x28]    ; [0x28:8]=0x203a0 ; '('
│           0x00003eaf      4889442448     mov qword [local_48h], rax
│           0x00003eb4      31c0           xor eax, eax
│           0x00003eb6      e815e10000     call fcn.00011fd0
[...]
[0x00005850]>
```
As we can see, we disassembled the function seeking temporarily to it's starting offset but after the disassembly we are still placed are previous offset because of the `@` modifier.


### Handy tricks
- Many commands support appending `j` (`j~{}`) for json (indented) output
- Append `q` for quit output.
- Pipe with shell commands like in a normal shell with `|`
- Prefix with `!` to run shell commands.
- Internal grep with `~`

## To know more
### Books
- radare2 book (main reference): https://radare.gitbooks.io/radare2book
- radare2 explorations: https://monosource.gitbooks.io/radare2-explorations

### Presentations
- [talk]
- [talk]
- [...]

### Blogposts
-  http://radare.today
-  https://www.megabeets.net
-  [blog]
-  [blog]

## Support
- IRC #radare at irc.freenode.net
- Telegram: https://t.me/radare
