![banner](https://github.com/gnisrever/re-write-ups/assets/165166334/adb45311-d4d0-4fac-8c8c-f7ab3c94e7f2)


# Bbbbloat (Write-Up)

**Category:** Reverse Engineering \
**Points:** 300 \
**CTF Event:** picoCTF 2022 \
**Author:** gnisrever

---

## Challenge Description

Can you get the flag? Reverse engineer this [binary](https://artifacts.picoctf.net/c/46/bbbbloat)

---

## Approach
In this write-up, we will reverse engineer the provided binary to uncover the flag.

### Tools Used
*Note: All tools used in this write-up are preinstalled on REMnux (https://remnux.org/).*
| Tool       | Purpose                                                                                      |
|------------|----------------------------------------------------------------------------------------------|
| Ghidra     | Software Reverse Engineering (SRE) framework developed by the National Security Agency (NSA) |

### Step-by-Step Solution

#### 1. Setup
To prepare for the challenge, we first create a new directory named `bbbbloat` using the `mkdir` command. This directory will serve as our workspace for analysis.
```bash
remnux@remnux:~/ctf/pico$ mkdir bbbbloat
```
Next, we change to the newly created directory using the `cd` command.
```bash
remnux@remnux:~/ctf/pico$ cd bbbbloat
```
With our workspace ready, we download the binary file into this directory using the `wget` command. The binary can be obtained from the provided [link](https://artifacts.picoctf.net/c/46/bbbbloat).
```bash
remnux@remnux:~/ctf/pico/bbbbloat$ wget https://artifacts.picoctf.net/c/46/bbbbloat
```
```plaintext
--2024-06-02 20:56:58--  https://artifacts.picoctf.net/c/46/bbbbloat
Resolving artifacts.picoctf.net (artifacts.picoctf.net)... 18.155.216.44, 18.155.216.22, 18.155.216.49, ...
Connecting to artifacts.picoctf.net (artifacts.picoctf.net)|18.155.216.44|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 14472 (14K) [application/octet-stream]
Saving to: ‘bbbbloat’

bbbbloat                 100%[==================================>]  14.13K  --.-KB/s    in 0s      

2024-06-02 20:56:59 (209 MB/s) - ‘bbbbloat’ saved [14472/14472]
```
Now that we have obtained the binary, we can proceed with our initial analysis.
#### 2. Initial Analysis
The `file` command will be our first tool to examine the binary. It not only reveals the file type and architecture but also indicates whether the binary has been `stripped`.
<details>
<summary>What does "stripped" mean?</summary>
Stripping a binary removes debugging and symbol information, making it smaller and harder to analyze. This information, such as variable names and function names, is useful for developers but not necessary for the binary to run. Stripped binaries are more challenging to reverse engineer.

  
*Note*: Stripping is different from obfuscation. Obfuscation involves deliberately making the code more complex and harder to understand, often by renaming variables and functions to meaningless names or using convoluted logic. Stripping, on the other hand, simply removes helpful metadata without altering the actual code logic.
</details>

```bash
remnux@remnux:~/ctf/pico/bbbbloat$ file bbbbloat
```
```plaintext
bbbbloat: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=db1dc86836c3e0e4140eb30914db4af5bce7cb18, for GNU/Linux 3.2.0, stripped
```
The output indicates that our binary file has been stripped. This will complicate locating the main function and debugging the binary due to the absence of symbol and debugging information. However, it is important to note that stripping a file does not affect the strings within the binary. These strings can still be extracted and analyzed, providing valuable insights into the binary's functionality.

To gain a better understanding of the binary, we need to change its mode to make it executable. This can be accomplished using the `chmod` command with the `+x` flag.
```bash
remnux@remnux:~/ctf/pico/bbbbloat$ chmod -x bbbbloat
```
This allows us to directly execute the program by adding `./` in front of the executable, specifying that we want to run the file located in our current directory.
```bash
remnux@remnux:~/ctf/pico/bbbbloat$ ./bbbbloat
```
```plaintext
What's my favorite number?
```
The program prompts us with the question "What's my favorite number?". We can input a number, such as `1234`, and the program responds with ->
```plaintext
Sorry, that's not it!
```
This suggests that the program is likely expecting a specific number as an input.

Given that `bbbbloat` has been stripped, we can still attempt to uncover clues by using the `strings` command to analyze all the information provided.
```bash
remnux@remnux:~/ctf/pico/bbbbloat$ strings ./bbbbloat
```
```plaintext
/lib64/ld-linux-x86-64.so.2
libc.so.6
__isoc99_scanf
__stack_chk_fail
putchar
strdup
printf
strlen
stdout
fputs
__cxa_finalize
__libc_start_main
free
GLIBC_2.7
GLIBC_2.4
GLIBC_2.2.5
_ITM_deregisterTMCloneTable
__gmon_start__
_ITM_registerTMCloneTable
u+UH
< ~XH
A:4@r%uLH
4Ff0f9b0H
3=_cf0ehH
d_be6bN
VUUUH
VUUUH
VUUUH
VUUUH
VUUUH
VUUUH
VUUUH
VUUUH
dH3<%(
[]A\A]A^A_
What's my favorite number? 
Sorry, that's not it!
:*3$"
GCC: (Ubuntu 9.4.0-1ubuntu1~20.04.1) 9.4.0
.shstrtab
.interp
.note.gnu.property
.note.gnu.build-id
.note.ABI-tag
.gnu.hash
.dynsym
.dynstr
.gnu.version
.gnu.version_r
.rela.dyn
.rela.plt
.init
.plt.got
.plt.sec
.text
.fini
.rodata
.eh_frame_hdr
.eh_frame
.init_array
.fini_array
.dynamic
.data
.bss
.comment

```
The output of the `strings` command on the binary file `./bbbbloat` shows a mix of standard library function names, symbols related to the `GNU C Library (GLIBC)`, and some seemingly random strings. Here's what we can infer from this output:

- Standard Library Functions: The file is likely compiled with standard C library functions such as `scanf`, `putchar`, `strdup`, `printf`, `strlen`, `stdout`, `fputs`, `free`, etc. These functions are commonly used in C programs.

- GLIBC Versions: The presence of strings like `GLIBC_2.7`, `GLIBC_2.4`, `GLIBC_2.2.5` indicates the minimum GLIBC version required for the binary to run. This suggests that the binary was compiled on a system with GLIBC version 2.7 or later.

- Compiler Information: The string `GCC: (Ubuntu 9.4.0-1ubuntu1~20.04.1) 9.4.0` indicates that the binary was compiled using the GCC compiler version 9.4.0 on Ubuntu 20.04.1.

- Other Symbols: The other symbols such as `_ITM_deregisterTMCloneTable`, `__gmon_start__`, `_ITM_registerTMCloneTable`, etc., are related to compiler features or runtime behavior. For example, `_ITM_registerTMCloneTable` is related to thread-local storage in GCC.

- Random Strings: The random-looking strings like `u+UH`, `A:4@r%uLH`, `4Ff0f9b0H`, etc., are likely not meaningful and might be artifacts of the compilation process or data in the binary that happens to resemble strings.

However, the presence of the strings `"What's my favorite number?"` and `"Sorry, that's not it!"` within the binary strongly implies the existence of an if statement, which likely orchestrates a conditional question and answer scenario. Leveraging Ghidra's robust analysis capabilities, one can strategically search for these specific strings to pinpoint the associated code segment, thereby illuminating the underlying logic governing this interaction.
#### 3. Static Analysis
You can run `Ghidra` from the command line by simply typing `ghidra`.
```bash
remnux@remnux:~/ctf/pico/bbbbloat$ ghidra
```
##### Opening Ghidra:
Upon launching Ghidra, you are presented with a blank interface. To create a new project, click on `File` in the top-left corner of the window.

![file](https://github.com/gnisrever/re-write-ups/assets/165166334/56dd0063-e7ad-46b7-a8e7-a85e8ee030de)

##### Creating a New Project:
Select `New Project` from the dropdown menu or use the shortcut `Ctrl+N`.

![new project](https://github.com/gnisrever/re-write-ups/assets/165166334/eb6ac123-c673-4bdd-b117-31735651a22c)

##### Project Type:
Choose `Non-Shared Project` and click `Next`.

![non-shared](https://github.com/gnisrever/re-write-ups/assets/165166334/daebafa9-504e-43ca-8e02-01462074c33a)

##### Naming the Project:
Name your project, for example, `bbbbloat_ghidra`, and click `Next`.

![file name](https://github.com/gnisrever/re-write-ups/assets/165166334/3b76a2b0-d5de-42c4-9796-c456f90577fc)

##### Importing a File:
With the project created, click on `File` in the top-left corner and select `Import File` from the dropdown.

![import file](https://github.com/gnisrever/re-write-ups/assets/165166334/cd71d3de-5c82-4215-a08e-27f78073e0c4)

##### Selecting the File:
Navigate to the directory containing `bbbbloat`, select the file, and click `Select File to Import`.

![import file window](https://github.com/gnisrever/re-write-ups/assets/165166334/1cc0119c-b8d9-4360-b4a9-f482166de204)

##### Confirming File Details:
Confirm that the binary is an `Executable and Linking Format (ELF)` file in `x86:LE:64:default:gcc` language. Click `OK`.

![import pref](https://github.com/gnisrever/re-write-ups/assets/165166334/a3699d03-449b-48ca-a40c-7c29c6792392)

##### Viewing Import Results:
Review the import results, which include basic information and checksums (MD5, SHA256). Click `OK` to proceed.

![import results summary](https://github.com/gnisrever/re-write-ups/assets/165166334/8f9c1e15-1501-4aa0-af43-8529ac304128)

##### Starting Analysis:
To analyze the executable, click `Yes` when prompted to open the `Analysis Options` window.

![analyze](https://github.com/gnisrever/re-write-ups/assets/165166334/10d2198f-9b8d-4d5a-a697-3210f2e6520e)

##### Running the Analysis:
Ensure all necessary analyzers are selected. Click `Analyze` to begin the analysis.

![analysis option](https://github.com/gnisrever/re-write-ups/assets/165166334/156ae093-3909-41d4-98b2-fbe667f34697)

##### Accessing Defined Strings:
To locate specific strings, navigate to `Window` in the top-left corner and select `Defined Strings` from the dropdown.

![defined strings](https://github.com/gnisrever/re-write-ups/assets/165166334/2c90a4c9-123c-48c4-a834-65f314c068a3)

##### Finding a Specific String:
The `Defined Strings` window displays all strings in the executable. Scroll to find the string `"What's my favorite number?"`.

![strin in defined strings](https://github.com/gnisrever/re-write-ups/assets/165166334/58fb3b41-d66f-4380-b074-ccaac5f731b4)

##### Navigating to the Reference:
Double-click the string to navigate to its reference.

![xref](https://github.com/gnisrever/re-write-ups/assets/165166334/e11def0d-c252-43c9-9ee3-631b0863b79f)

##### Locating the String in Code:
Double-click the hyperlink reference (HREF) to jump to the specific part of the code where this string is referenced or used.

![xref 2](https://github.com/gnisrever/re-write-ups/assets/165166334/b15d990d-a26e-464e-9517-d780e3b65043)

##### Reviewing the String Location:
You have now located the string within the code.

![001013cb location](https://github.com/gnisrever/re-write-ups/assets/165166334/c00fe0a9-7342-4e21-aaf1-068062e726a2)

##### Using the Decompiler Window:
Utilize the Decompiler window to gain a better understanding of the function and environment where the string resides.

![decompile window](https://github.com/gnisrever/re-write-ups/assets/165166334/bdc5fb1f-1155-4368-87e8-60534849b25d)

##### Function Analysis
```c
undefined8 FUN_00101307(void)

{
  long in_FS_OFFSET;
  int local_48;
  undefined4 local_44;
  char *local_40;
  undefined8 local_38;
  undefined8 local_30;
  undefined8 local_28;
  undefined8 local_20;
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  local_38 = 0x4c75257240343a41;
  local_30 = 0x3062396630664634;
  local_28 = 0x68653066635f3d33;
  local_20 = 0x4e623665625f64;
  local_44 = 0xd2c49;
  printf("What\'s my favorite number? ");
  local_44 = 0xd2c49;
  __isoc99_scanf(&DAT_00102020,&local_48);
  local_44 = 0xd2c49;
  if (local_48 == 0x86187) {
    local_44 = 0xd2c49;
    local_40 = (char *)FUN_00101249(0,&local_38);
    fputs(local_40,stdout);
    putchar(10);
    free(local_40);
  }
  else {
    puts("Sorry, that\'s not it!");
  }
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return 0;
}
```
Upon reviewing the decompiled code, focus on the if statement:
```c
if (local_48 == 0x86187) {
    local_44 = 0xd2c49;
    local_40 = (char *)FUN_00101249(0,&local_38);
    fputs(local_40,stdout);
    putchar(10);
    free(local_40);
}
else {
    puts("Sorry, that\'s not it!");
}
```
This if statement compares two values:
```c
if (local_48 == 0x86187)
```
Here, `local_48` is the user input when prompted with, "What's my favorite number?". The value `0x86187` is a hexadecimal constant. If the input matches this constant, the if statement executes, printing the flag. Otherwise, the else statement executes, displaying "Sorry, that's not it!". Therefore, it is unnecessary to understand the if statement's body in detail, as it is irrelevant to our primary analysis. The critical part of our analysis is recognizing that the if statement checks if the input matches a specific value and responds accordingly. The detailed actions within the if statement are not essential for understanding how to trigger the correct response from the program. This logic can be simplified in pseudocode:
```c
if (input == 0x86187) {  // If the user input matches the hexadecimal value
  print flag           // Print the flag
} else {
  print "Sorry, that's not it!"  // Otherwise, indicate the input is incorrect
}
```
To convert `0x86187` to its decimal equivalent, right-click the hexadecimal value in the decompiler.

![binary if result](https://github.com/gnisrever/re-write-ups/assets/165166334/094adb25-4270-482b-9e66-fecbb6a30903)

The decimal value of `0x86187` is `549255`. 

To verify this, execute the binary from the command line with `549255` as the input:

```bash
remnux@remnux:~/ctf/pico/bbbbloat$ ./bbbbloat
```
```plaintext
What's my favorite number? 549255
```
```plaintext
picoCTF{cu7_7h3_bl047_695036e3}
```
We have successfully retrieved the flag!

## Conclusion
Through static analysis using tools like `file`, `strings`, and `Ghidra`, we reverse-engineered the `bbbbloat` binary and identified the specific input value required to reveal the flag. This write-up demonstrates a methodical approach to reverse engineering and the importance of understanding binary structures and logic.




