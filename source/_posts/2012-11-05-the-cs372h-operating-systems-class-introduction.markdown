---
layout: post
title: "The CS372H - Operating Systems class - Introduction"
date: 2012-11-05 18:52
comments: true
categories: [OperatingSystems, JOS, CS372H]
---
## FAQ

Q: _Didn't you attend Operating Systems class in school?_

A: Yes, there was one. It was good for introductory suff like using syscalls from user-land, getting to know what process is, what thread is, how to do mutual exclusion using semaphores, mutexes etc., how to use signals.... It was good but it was all user-land based. There was no hacking kernel space at all.

Q: _Why this class? Why not Coursera or something else?_

A: I love Coursera. I took three courses there "Machine Learning", "Algorithms: Design and Analysis, Part 1" and "Compilers". All three are great courses. Unfortunately there is no operating systems class. I found classes from few universities (Brown, MIT) but not all lab software is available to "outsiders". This course is used by University of Texas as CS372H (it's actually a MIT class) and all materials are publicly available.

Q: _Why learning operating systems? Are you going for an operating system devel job?_

A: Well, I am interested in operating systems. I can't explain it - I just love low level stuff. I don't expect to land operating systems dev job since there is no such thing as operating systems dev where I live. (There isn't much IT wise where I live at all, but that is another story). 

## System setup

### Linux setup 

I will be doing these labs on Linux Mint 13 64-bit which is publicly available for download. There is no particular reason for this exact version - I just had it at home on DVD.

Procedure described at [Tools](http://www.cs.utexas.edu/~mwalfish/classes/s11-cs372h/tools.html) page work as described except one little thing - I found that gcc "make" should not be started inside gcc source directory. I created build directory and started configure, make and make install from it.

### Labs

Labs are available from following [GIT](http://www.cs.utexas.edu/~mwalfish/classes/s11-cs372h/jos.git) repository. All the implementations I do as a part of this labs are available at [github](https://github.com/iostres/CS372H). Now we are done with setup and can actually start working on labs assignments. 

