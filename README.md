# buffcat
A hopefully helpful handbook for performing buffer overflows from the command line using radare2

🐅

There are already numerous guides for performing basic stack-based buffer overflows with popular crackme binaries, using debuggers like Immunity or ollydbg.

This guide hopes to empower you with the knowledge needed to perform these buffer overflows quickly using the commnand line, regardless of whether you're in Linux or Windows!

  I'm making one assumption, which is that you're already familiar with [cracking the brainpan.exe binary with Immunity Debugger](https://assume-breach.medium.com/oscp-prep-buffer-overflows-made-super-easy-with-the-brainpan-1-vm-e5ccaf7d3f0c). Learning this process caused me much feet-dragging, as I know it does for many security students. (I know, I know, just repeat it til you get it and it's easy.) Well, I found another way, using [radare2](https://github.com/radareorg/radare2)! I didn't find this process documented anywhere else, so here is how I eventually did it, streamlined into some hopefully easy steps:

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

Hopefully you'll see something like this in your Linux VM:

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

and you'll see something like this in your Windows VM:

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

