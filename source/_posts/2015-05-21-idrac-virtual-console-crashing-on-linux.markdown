---
layout: post
title: "iDrac virtual console crashing on linux"
date: 2015-05-21 02:13
comments: true
categories: note-to-self linux idrac java-webstart
---

## iDrac

[iDrac][1] (or any other [IPMI][2]) is great and can really save your time and your life, but
it is really frustrating to find out that your out-of-band virtual console crashes on you at
the moment you need it the most.

Given that this particular problem has struck me a few times already I decided to write a
post to my future self and save me some trouble.

## Scenario

After receiving an alert and failing to connect to the host, you log into the iDrac's web
interface and click "Launch" on the Virtual Console option.  
It downloads a file called `viewer.jnlp`.  
You start it with `javaws viewer.jnlp` and, after a few confirmation dialogs, it explodes!

```
carlos@multi ~/Downloads> javaws viewer.jnlp
This application does not specify a Codebase in its manifest. Please verify with the applet's vendor. Continuing. See: http://docs.oracle.com/javase/7/docs/technotes/guides/jweb/security/no_redeploy.html for details.
This application does not specify a Codebase in its manifest. Please verify with the applet's vendor. Continuing. See: http://docs.oracle.com/javase/7/docs/technotes/guides/jweb/security/no_redeploy.html for details.
Application title was not found in manifest. Check with application vendor
KVM/VM Client Version: 5.01.01 (Build 22)
ProtocolAPCP.receieveSessionSetup : reconType = 101
capabilities..4
#
# A fatal error has been detected by the Java Runtime Environment:
#
#  SIGSEGV (0xb) at pc=0x00007f1268a72550, pid=14296, tid=139717301577472
#
# JRE version: OpenJDK Runtime Environment (8.0_45-b14) (build 1.8.0_45-b14)
# Java VM: OpenJDK 64-Bit Server VM (25.45-b02 mixed mode linux-amd64 compressed oops)
# Problematic frame:
# C  0x00007f1268a72550
#
# Failed to write core dump. Core dumps have been disabled. To enable core dumping, try "ulimit -c unlimited" before starting Java again
#
# An error report file with more information is saved as:
# /home/carlos/Downloads/hs_err_pid14296.log
#
# If you would like to submit a bug report, please visit:
#   http://bugreport.java.com/bugreport/crash.jsp
#
fish: Job 1, “javaws viewer.jnlp ” terminated by signal SIGABRT (Abort)
```

Looking into the log file gives no better clue.

```
---------------  T H R E A D  ---------------

Current thread (0x00007f1289ae0000):  JavaThread "iDRAC7 Virtual Console Client" [_thread_in_native, id=14329, stack(0x00007f1278112000,0x00007f1278213000)]

siginfo: si_signo: 11 (SIGSEGV), si_code: 1 (SEGV_MAPERR), si_addr: 0x00007f1268a72550

Registers:
RAX=0x00007f1268a72550, RBX=0x0000000000000378, RCX=0x0000000000000000, RDX=0x00007f12782102cf
RSP=0x00007f12782102e8, RBP=0x00007f1278210550, RSI=0x00007f12698cf477, RDI=0x00007f12698cf590
R8 =0x00007f1298984908, R9 =0x0000000000000000, R10=0x00007f1268d5e380, R11=0x0000000000000202
R12=0x0000000000000000, R13=0x00007f1269e15eb0, R14=0x00007f12782107b8, R15=0x00007f1289ae0000
RIP=0x00007f1268a72550, EFLAGS=0x0000000000010206, CSGSFS=0x0000000000000033, ERR=0x0000000000000014
  TRAPNO=0x000000000000000e

Top of Stack: (sp=0x00007f12782102e8)
0x00007f12782102e8:   00007f1269800440 0000000000000000
(...)
```

There is a lot of information there but going thru java stacktraces of your virtual console tool while
your server is down is undesired at best.

## Solution

Fortunately the solution is simple.

Open the `viewer.jnlp` file.  
There will be a bunch of OS and architecture-specific lines like this:

```
 <resources os="Windows" arch="x85">
   <nativelib href="https://server.example.com:443/software/avctKVMIOWin32.jar" download="eager"/>
   <nativelib href="https://server.example.com:443/software/avctVMAPI_DLLWin32.jar" download="eager"/>
 </resources>
 <resources os="Windows" arch="amd64">
   <nativelib href="https://server.example.com:443/software/avctKVMIOWin64.jar" download="eager"/>
   <nativelib href="https://server.example.com:443/software/avctVMAPI_DLLWin64.jar" download="eager"/>
 </resources>
 (...)
```

Delete all of them, except for the relevant one. In my case:

```
  <resources os="Linux" arch="x86_64">
    <nativelib href="https://server.example.com:443/software/avctKVMIOLinux64.jar" download="eager"/>
   <nativelib href="https://server.example.com:443/software/avctVMAPI_DLLLinux64.jar" download="eager"/>
  </resources>
```

That's it. Running `javaws viewer.jnlp` should work fine now.

I believe this is not the "correct" solution but it gets you going.

  [1]: https://en.wikipedia.org/wiki/Dell_DRAC
  [2]: https://en.wikipedia.org/wiki/Intelligent_Platform_Management_Interface
