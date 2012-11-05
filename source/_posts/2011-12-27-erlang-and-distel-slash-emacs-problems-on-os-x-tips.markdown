---
layout: post
title: "Erlang and Distel/Emacs problems on OS X – tips"
date: 2011-12-27 11:40
comments: true
categories: [Erlang,Emacs] 
---

I recently compiled  R14B04 version of Erlang OTP suite and tried to make it work with Distel (current trunk version). Pretty good how-to is described on Clemensont’s blog but still had two major problems with Erlang/Distel/Emacs24 combo:

	* Connect to node (C-c C-d n) and node-ping (C-c C-d g) 
	  would always result in message “nodedown”
	* (after first one is fixed): Sometimes when adding breakpoint 
	  I would get a message “Module is not interpreted, can’t set breakpoints”
	
Solutions are as follows:

<br />
**"Nodedown" problem**

This message indicated that Emacs is unable to connect to inferior Erlang process. This is a well known problem of Distel/Erlang for versions of Erlang starting from R14. I downloaded the patch from Ralfs post and applied it to Distel epmd.el file. Now I can correctly connect to Erlang node.

<br />
**"Module is not interpreted, can’t set breakpoints" problem**

This is also Distel issue (so it seems). Fortunately, there is a simple workaround – when this message appears do the following:

	* C-c C-d m (opens the monitor buffer)
	* k (kills the monitor buffer)
	* C-c C-d i (start interpreting) in the .erl buffer will now work.
 
That’s it.

