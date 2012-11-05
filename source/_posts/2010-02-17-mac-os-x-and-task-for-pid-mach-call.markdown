---
layout: post
title: "Mac OS X and task_for_pid() mach call"
date: 2010-02-17 22:07
comments: true
categories: [OSX]
---

If you never heard of mach system calls and specifically task_for_pid() call on Mac OS X, you can consider yourself lucky. If you want to stay that way – stop reading now! Still here? In that case let’s start with disclaimer – author of this text is not and can not be in any way responsible for damage produced or influenced by this article.

Prior to the Mac OS X 10.4.X (Tiger), it was completely legal for one process to control another for the purpose of influencing its execution (single stepping, resuming, stopping etc) and inspecting or modifying its memory and registers. In one of the patches for Tiger, this policy was changed so that only a process owned by root or with a “primary effective group of procmod or procview” has this privilege. In Leopard (Mac OS X 10.5), this policy was changed again (that much about consistent security policy – nice work Apple) such that an inspector process now depends on the security framework to authorize use of the task_for_pid system service which gives a process the capability to control another process.

To build a utility that will use task_for_pid(), you need to do the following:

	1. Create Info.plist file which will be embedded to your executable 
	   file and will enable code signing 
	2. Create self-signed code signing certificate using Keychain access 
	3. Write your program that uses security framework to obtain rights 
	   to execute task\_for_pid() 
	4. Compile your program and code-sign it.
	
So let’s get started.

###Step 1 – Create Info.plist

I used one of the standard Info.plist files I could find in Xcode and changed some particular parts as can be seen in following example:

{% include_code task_for_pid_info_plist.xml %}

The important part is key “SecTaskAccess” with value “allowed”.

###Step 2 – Create self-signed code signing certificate

Open your Keychain Access and do the following:

{% img center /images/task_for_pid_key_1m.png %}

{% img center /images/task_for_pid_key_2m.png %}

{% img center /images/task_for_pid_key_3m.png %}

{% img center /images/task_for_pid_key_4m.png %}

{% img center /images/task_for_pid_key_5m.png %}

{% img center /images/task_for_pid_key_6m.png %}

{% img center /images/task_for_pid_key_7m.png %}

When created – this certificate will be untrusted by default – change “When using this certificate” to “Always Trust” and you should be OK and ready to go for the next step.

###Step 3 – Write your program

I wrote a very simple program that takes PID of a process you want to investigate (ran by your UID), connects to it and writes current register values for it. Code is pretty self-explaining so I won’t go into nifty details:

{% include_code task_for_pid_prog.c %}

###Step 4 – Compile and sign

To compile the program I used following command line:

{% codeblock %}
gcc tfpexample.c -sectcreate __TEXT __info_plist ./Info.plist -o tfpexample -framework Security - framework CoreFoundation
{% endcodeblock %}

To sign the code with certificate we prepared before – do this:

{% codeblock %}
codesign -s tfpexample ./tfpexample
{% endcodeblock %}

We can check if everything went OK:


{% codeblock %}
[ivan]~/hoby/debuger/blog-tfp > codesign -dvvv ./tfpexample

Executable=/Users/ivan/hoby/debuger/blog-tfp/tfpexample 
Identifier=net.os-tres.tfpexample 
Format=Mach-O thin (i386)
CodeDirectory v=20001 size=187 flags=0!0(none) hashes=4+2 location=embedded 
CDHash=30c98f962fc9a2b1a2f73cff55d4584bd053aa3a 
Signature size=1454 
Authority=tfpexample
Signed Time=Feb 17, 2010 8:40:04 PM 
Info.plist entries=6 
Sealed Resources=none 
Internal requirements count=0 size=12
{% endcodeblock %}

This looks good – let’s test it.

###Step 5 – Test program

{% codeblock %}
[ivan]~/hoby/debuger/blog-tfp > 
[ivan]~/hoby/debuger/blog-tfp > ps 

PID TTY TIME CMD 
2511 ttys000 0:00.02 -zsh
2104 ttys001 0:00.08 -zsh 

[ivan]~/hoby/debuger/blog-tfp > 
[ivan]~/hoby/debuger/blog-tfp > ./tfpexample 

Enter pid: 2104 

Thread 2104 has 1 threads. Thread 0 state: 
EIP: 953d1562 
EAX: 4006f 
EBX: 2ad74 
ECX: bffff9cc 
EDX: 953d1562 SS: 1f 

[ivan]~/hoby/debuger/blog-tfp >
{% endcodeblock %}

It works.
