# buffcat
A hopefully helpful handbook for performing buffer overflows from the command line using radare2

🐅

There are already numerous guides for performing basic stack-based buffer overflows with popular crackme binaries, using debuggers like Immunity or ollydbg.

This guide hopes to empower you with the knowledge needed to perform these buffer overflows quickly using the commnand line, regardless of whether you're in Linux or Windows!

  This guide compliments Brainpan/Immunity Debugger walkthroughs [like this one](https://assume-breach.medium.com/oscp-prep-buffer-overflows-made-super-easy-with-the-brainpan-1-vm-e5ccaf7d3f0c). Learning this process caused me much feet-dragging, as I know it does for many security students. (I know, I know, just repeat it til you get it and it's easy.) Well, I found another way, using [radare2](https://github.com/radareorg/radare2)! I didn't find this process documented anywhere else, so here is how I eventually did it, streamlined into some hopefully easy steps:

## The setup
#### 1) VMs
You'll need a Windows VM and a Linux VM running together, and able to ping each other.
#### 2) Brainpan 
On the Windows VM, download and launch brainpan.exe Copies of this are all over the web, just make sure you get it from somewhere reputable and don't run it outside of a VM.
#### 3) Radare2
Also on the Windows VM, download radare2 for Windows. This part is minorly tricky because, to debug 32-bit binaries, you have to get the 32-bit version of the debugger, which you have to find on radare2's github actions page, or you can just use [the version I used in this example](https://github.com/radareorg/radare2/files/6617618/radare2-install.zip). If, once we open radare2 in the next step, you notice that your registers start with 'e' instead of 'r' ('eip' instead of 'rip'), you'll know that you got the right version to debug brainpan.

32-bit radare2 (the one you want for 32 bit binaries, and for this example):

![eregs](https://user-images.githubusercontent.com/5056836/121236365-1a0f8e80-c853-11eb-89f1-aa1cb7ae32fd.PNG)

64-bit radare2 (the one you want for 64-bit binaries, which I don't know how to do yet, so I won't cover here!):

![rregs](https://user-images.githubusercontent.com/5056836/121236629-678bfb80-c853-11eb-94ed-72fcfe7f3814.PNG)

#### 4) PID
In your Windows VM, press ctrl + shift + esc to bring up the Task Manager. Go to the Details tab to find brainpan's PID. PIDs are basically random so don't worry if yours is a different number than mine:

![tm](https://user-images.githubusercontent.com/5056836/121237591-732bf200-c854-11eb-99be-9b8cd28552c3.PNG)

Keep the Task Manager available. Anytime we close and re-open brainpan during debugging, we'll need to look up this PID since it will change.

### The good part

#### 5) Debugging
In your Windows VM, open up a second cmd terminal aside from the one brainpan is running in. In your `\radare2-install\bin\` directory, run `radare2.exe -dA <YOUR_BRAINPAN_PID_HERE>`, using your brainpan.exe PID as the last arg. `-d` is for debug mode, and `-A` is to auto analyze the binary.

![rad1](https://user-images.githubusercontent.com/5056836/121239756-e33b7780-c856-11eb-932d-f76177bdfc54.PNG)

You'll get some analysis output, followed by a command prompt, which is the bottom line in yellow. The hex address shown is the current address in memory. From this prompt, run `4dc` to un-pause the debugger 4 times (radare2 appears to pause brainpan's execution whenever a new thread is launched, and this command un-pauses it so that we can debug in real time). 

It will say "Finished thread", at which point you can hit enter again and get back to the yellow radare2 prompt.

```
[0x772e29dc]> 4dc
(6680) loading library at 0x77270000 (C:\Windows\SysWOW64\ntdll.dll) ntdll.dll
(6680) Created thread 3244 (start @ 772A58E0) (teb @ 00277000)
(6680) Created thread 4300 (start @ 772A58E0) (teb @ 0027A000)
... 
(6680) Created thread 6940 (start @ 7731DC70) (teb @ 0027D000)![err](https://user-images.githubusercontent.com/5056836/121292239-dc3c5580-c8a6-11eb-9db8-fe49f7663519.PNG)

(6680) Finished thread 6940 Exit code 0
(6680) Finished thread 3244 Exit code 0
[0x772e29dc]>
```

#### 6) Fuzzing
This is where we'll try bombing the the binary with lots  input to find out how many bytes it takes to crash the app. For other binaries, you'll have to write your own fuzzer. For this exercise, there's one that works pretty well already, and we can use it for later steps of the guide too. 

Over in the Linux VM, clone [this repo full of pre-made brainpan scripts](https://github.com/jessekurrus/brainpan) (you'll need to have python2 installed to use them I'm afraid).

Navigate into the repo and run `python2 brainfuzzer.py <YOUR_WINDOWS_VM_IP> 9999`

You should see something like this in your Linux VM:

```
python2 brainfuzzer.py <YOUR_WINDOWS_VM_IP> 9999
[+] Attempting to crash brainpan.exe at 1 bytes
[+] Attempting to crash brainpan.exe at 100 bytes
[+] Attempting to crash brainpan.exe at 200 bytes
[+] Attempting to crash brainpan.exe at 300 bytes
[+] Attempting to crash brainpan.exe at 400 bytes
[+] Attempting to crash brainpan.exe at 500 bytes
[+] Attempting to crash brainpan.exe at 600 bytes
[+] Attempting to crash brainpan.exe at 700 bytes
[+] Attempting to crash brainpan.exe at 800 bytes
[+] Attempting to crash brainpan.exe at 900 bytes
[+] Attempting to crash brainpan.exe at 1000 bytes
[+] Connection failed. Make sure IP/port are correct, or check debugger for brainpan.exe crash.
```

and something like this in your Windows VM:

```
brainpan.exe
[+] initializing winsock...done.
[+] server socket created.
[+] bind done on port 9999
[+] waiting for connections.
[+] received connection.
[get_reply] s = [A
]
[get_reply] copied 3 bytes to buffer
[+] check is -1
[get_reply] s = [A
]
[get_reply] copied 3 bytes to buffer
[+] received connection.
[get_reply] s = [AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
]
[get_reply] copied 102 bytes to buffer
[+] check is -1
[get_reply] s = [AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
]
...
[get_reply] copied 502 bytes to buffer
[+] check is -1
[get_reply] s = [AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
]
[get_reply] copied 502 bytes to buffer
[+] received connection.
[get_reply] s = [AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
]
[get_reply] copied 602 bytes to buffer
```

The fuzzer will probably run a little past what the server actually responds to, this is fine. We see on the Windows box that brainpan crashed after 602 bytes.

This is how we know roughly how long our offset will be, but it's only a very rough number. To find the exact offset, we need to go to the next step. We'll send another long string with no repeating patterns.

First, notice that the memory address of our crashed program is `0x41414141`. 0x41 is the hexadecimal translation of the ASCII character 'A', which tells us that our EIP register was filled with As when the program crashed.

Restart brainpan and reattach radare2, making sure to run `4dc` to skip past all the thread startup steps. 

You may notice by now that the process of restarting the radare2 debugger isn't as time-consuming as with point-click debuggers!

In your Linux VM, execute:

` $ python2 brainpan1.py <YOUR_WINDOWS_VM_IP> 9999`

In your Windows VM, observe:

![bp1-2](https://user-images.githubusercontent.com/5056836/121798525-23e31a00-cbe4-11eb-93d3-78c1ad1e277c.PNG)
Brainpan receives the pattern from the attack server and crashes

![bp1](https://user-images.githubusercontent.com/5056836/121798522-1d54a280-cbe4-11eb-934a-29db40437c77.PNG)

The attached radare2 process shows that when the application crashed, we were at memory address `0x35724134`

You may notice that this is the same address we landed on in the Immunity tutorial I mentioned earlier when we reached this step! So we're still on track.

In our Linux VM, with `metasploit-framework` installed, we can check:

```
metasploit-framework/tools/exploit/pattern_offset.rb -q 35724134

[*] Exact match at offset 524
```

This tells us that our offset should be exactly 524 bytes off from our starting point. `brainpan2.py` performs a proof of concept, to make sure we can control EIP. First restart brainpan and radare2, then go to your Linux VM:

` $ python2 brainpan2.py <YOUR_WINDOWS_VM_IP> 9999`

In your Windows VM, observe:

![bp2-2](https://user-images.githubusercontent.com/5056836/121798886-0e6eef80-cbe6-11eb-8af0-69c8d58b41c6.PNG)

We send a long string with a bunch of As, followed by 4 Bs, followed by a bunch of Cs.

![bp2](https://user-images.githubusercontent.com/5056836/121798843-e4b5c880-cbe5-11eb-97a3-f0e4c60c2644.PNG)

EIP is filled with the only 4 Bs (0x42 in ASCII) in our input string, proving that we control precisely the 4 bytes that will control EIP.

### 7) Identify a usable JMP ESP instruction
Great! Now we need to find a JMP ESP instruction in our binary, somewhere that we can exploit, so that we can redirect the execution flow of our program.

Restart brainpan and reattach radare2, then run `is` to list all the symbols.

![is](https://user-images.githubusercontent.com/5056836/121800858-64956000-cbf1-11eb-990a-02a19aa35400.PNG)

For small, simple binaries like this, we'll commonly find references to functions here that cannot be reached by our normal execution path, but which sometimes contain a JMP ESP instruction that we can exploit. In this case, the `_winkwink` function catches our eye. Let's see what it does by looking up the "vaddr" virtual address value for our `_winkwink` function: `0x311712f0`, and then using radare2's "seek" command: `> s 0x311712f0`.

![s](https://user-images.githubusercontent.com/5056836/121801001-2f3d4200-cbf2-11eb-83b1-aedb2219bf41.PNG)

We'll see that our current position in memory has changed. But with only that piece of information, we can't do much.

Type `Shift + V`, then press `Enter` to enter visual mode:

![v](https://user-images.githubusercontent.com/5056836/121801108-d6ba7480-cbf2-11eb-9d28-5bd8367965f3.PNG)

If your screen changes when you press `Enter` but it doesn't look like this screenshot, don't worry. There are multiple perspectives in visual mode. Press the `p` key to cycle forward through perspectives, and press `Shift + p` to cycle backwards. Cycle through visual perspectives until you reach the debug view, which shows the CPU registers at the top:

![vd](https://user-images.githubusercontent.com/5056836/121801250-a6270a80-cbf3-11eb-888b-890003769b5b.PNG)

In the screenshot above, I've pointed out the columns of one instruction in particular. The right column shows the assembly instruction, `jmp esp`. This is the assembly instruction we've been searching for an exploitable intance of. The middle column shows the hexidecimal translation of that assembly instruction, `ffe4`. And the left column is, of course, the memory address where the instruction resides, `0x311712f3`.

(At some point, I hope to come back to this section with a little more detail regarding how to choose this JMP ESP. This is the step mona.py usually handles.)

Since our binary is 32-bit, it will use little-endian notation. So, we'll write each hex character in our new JMP ESP address in reverse, so that the instruction gets interpreted properly: `\xf3\x12\x17\x31`. We'll need this address shortly.

### 8) Bad Characters
