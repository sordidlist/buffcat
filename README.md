# buffcat
A hopefully helpful handbook for performing buffer overflows from the command line using radare2

🐅

There are already numerous guides for performing basic stack-based buffer overflows with popular crackme binaries, using debuggers like Immunity or ollydbg.

This guide hopes to empower you with the knowledge needed to perform these buffer overflows quickly using the commnand line, regardless of whether you're in Linux or Windows!

  I'm making one assumption, which is that you're already familiar with [cracking the brainpan.exe binary with Immunity Debugger](https://assume-breach.medium.com/oscp-prep-buffer-overflows-made-super-easy-with-the-brainpan-1-vm-e5ccaf7d3f0c). Learning this process caused me much feet-dragging, as I know it does for many security students. (I know, I know, just repeat it til you get it and it's easy.) Well, I found another way, using [radare2](https://github.com/radareorg/radare2)! I didn't find this process documented anywhere else, so here is how I eventually did it, streamlined into some hopefully easy steps:

### The setup
1) You'll need a Windows VM and a Linux VM running together, and able to ping each other.
2) On the Windows VM, download and launch brainpan.exe Copies of this are all over the web, just make sure you get it from somewhere reputable and don't run it outside of a VM.
3) Also on the Windows VM, download radare2 for Windows. This part is minorly tricky because, to debug 32-bit binaries, you have to get the 32-bit version of the debugger, which you have to find on radare2's github actions page, or you can just use [the version I used in this example](https://github.com/radareorg/radare2/files/6617618/radare2-install.zip). If, once we open radare2 in the next step, you notice that your registers start with 'e' instead of 'r' ('eip' instead of 'rip'), you'll know that you got the right version to debug brainpan.

32-bit radare2 (the one you want for 32 bit binaries, and for this example):

![eregs](https://user-images.githubusercontent.com/5056836/121236365-1a0f8e80-c853-11eb-89f1-aa1cb7ae32fd.PNG)

64-bit radare2 (the one you want for 64-bit binaries, which I don't know how to do yet, so I won't cover here!):

![rregs](https://user-images.githubusercontent.com/5056836/121236629-678bfb80-c853-11eb-94ed-72fcfe7f3814.PNG)

4) In your Windows VM, press ctrl + shift + esc to bring up the Task Manager. Go to the Details tab to find brainpan's PID. PIDs are basically random so don't worry if yours is a different number than mine:

![tm](https://user-images.githubusercontent.com/5056836/121237591-732bf200-c854-11eb-99be-9b8cd28552c3.PNG)

Keep the Task Manager available. Anytime we close and re-open brainpan during debugging, we'll need to look up this PID since it will change.

### The good part

5) In your Windows VM, open up a second cmd terminal aside from the one brainpan is running in. In your `\radare2-install\bin\` directory, run `radare2.exe -dA <YOUR_BRAINPAN_PID_HERE>`, using your brainpan.exe PID as the last arg. `-d` is for debug mode, and `-A` is to auto analyze the binary.

![rad1](https://user-images.githubusercontent.com/5056836/121239756-e33b7780-c856-11eb-932d-f76177bdfc54.PNG)

You'll get some analysis output, followed by a command prompt, which is the bottom line in yellow. The hex address shown is the current address in memory. 

6) This is where we'd fuzz the binary to find out how many bytes it takes to crash the app. Over in the Linux VM, run:

`python3 -c "print ('A' * 1500)"`

to get a long string of A characters printed to the console. If we were really trying to approach this binary blind, we might start with smaller values and work our way up to 1500 incrementally. In this case, my goal is to illustrate a parallel process to a known process, so I've skipped the relatively trivial stuff.

1500 As is a large enough input to crash our vulnerable app, so in your Linux VM, save the following as a python file:

```


```
