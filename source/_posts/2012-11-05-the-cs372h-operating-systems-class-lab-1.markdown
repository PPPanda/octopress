---
layout: post
title: "The CS372H - Operating Systems class - Lab 1"
date: 2012-11-05 18:59
comments: true
categories: [OperatingSystems, JOS, CS372H]
---



## Lab 1 - boot loader 

### Part 2: The Boot Loader
 
 Q/A Section:

Q: _At exactly what point does the processor transition from executing 16-bit code to executing 32-bit code?_

A: Processor transitions from executing 16-bit code to executing 32-bit code on execution of following instruction:

{% codeblock lang:c %}
	mov cr0, eax
{% endcodeblock %}

Q: _What is the last instruction of the boot loader executed, and what is the first instruction of the kernel it just loaded?_

A: Last instruction of boot loader executed is call of routine in .text section of kernel:

{% codeblock lang:c %}
	call *%eax
{% endcodeblock %}

Q: _How does the boot loader decide how many sectors it must read in order to fetch the entire kernel from disk? Where does it find this information?_

A: Boot loader decides how many sectors to read using information stored in ELF header. The ELF file format is shown in following image:

{% img center /images/lab1_elf_file_header.png %}

At the start of main.c, first sector (512 bytes) is loaded from disk. This sector is now in memory starting on location of 0x10000 and contains [ELF File Header](http://www.ouah.org/RevEng/x430.htm). First check is whether this is a correct ELF file by checking if magic number (first four bytes) of ELF file are correct:

{% codeblock lang:c %} 
	if (ELFHDR->e_magic != ELF_MAGIC)
			goto bad;
{% endcodeblock %}
        
ELF Program Header address is loaded to `ph` variable:

{% codeblock lang:c %}
	ph = (struct Proghdr *) ((uint8_t *) ELFHDR + ELFHDR->e_phoff);
{% endcodeblock %}

where e_phoff is the offset of program header from the start of the file. Then we store the address of last program header to eph variable:

{% codeblock lang:c %}
	eph = ph + ELFHDR->e_phnum;
{% endcodeblock %}

where e\_phnum is the number of entries in program header. Now we simply iterate through program header table and read segments using function _readseg_ giving it 3 arguments:

{% codeblock lang:c %}
    ph->p_va (Segment virtual address) 
    ph->p_memsz (Segment size in memory)
    ph->p_offset (Segment file offset)
{% endcodeblock %}

It's obvious that read size is rounded to sector size boundary which means that we load more than we really need in most cases.

---

**Exercise 6.** Reset the machine (exit qemu and start it again). Examine the 8 words of memory at 0x00100000 at the point the BIOS enters the boot loader, and then again at the point the boot loader enters the kernel. Why are they different?

A: Entry point in kernel is 0xf0100000 which we pass as virtual address to `readseg` function which does 0xf0100000 & 0xFFFFFF which gives the address at which we load the program text (0x100000). First 8 words contain first 8 words of kernel program text.

---

**Exercise 8.** We have omitted a small fragment of code - the code
  necessary to print octal numbers using patterns of the form
  "%o". Find and fill in this code fragment.
  
Following fragment is added starting with line 209 in `printfmt.c`:

{% codeblock lang:c %}
    case 'o':
        num = getuint(&ap, lflag);
        base = 8;
        goto number;
{% endcodeblock %}
---

Q: _Explain the interface between printf.c and
console.c. Specifically, what function does console.c export? How is
this function used by printf.c?_

A: File `console.c` exposes `cputchar` function which is used in
`printf.c` file.

Q: _Explain the following from console.c:_

{% codeblock lang:c %}
    if (crt_pos >= CRT_SIZE) {
        int i;
        memmove(crt_buf, crt_buf + CRT_COLS, (CRT_SIZE - CRT_COLS) * sizeof(uint16_t));
        for (i = CRT_SIZE - CRT_COLS; i < CRT_SIZE; i++)
            crt_buf[i] = 0x0700 | ' ';
        crt_pos -= CRT_COLS;
    }
{% endcodeblock %}

A: This code does line scrolling if `crt_pos` is at last line of crt +
1

Q: _In the following code, what is going to be printed after 'y='?
(note: the answer is not a specific value.) Why does this happen?_

A: Random garbage will be printed since second variable is not
specified.

---

**Challenge** Enhance the console to allow text to be printed in
  different colors.
  
I decided to go with standard ANSI escape sequences to define text
foreground and background colors. Full specification can be found here
[ANSI escape sequence](http://ascii-table.com/ansi-escape-sequences.php). Defined
functions include various screen actions but for text
foreground/background coloring I decided to implement just a small
subset of whole specification called "Set Graphics Mode". Following
pattern is used to specify color:

	Esc[Value;...;Valuem

Following attributes are defined:

*Text attributes:*

	0 All attributes off
	1 Bold on
	4 Underscore (on monochrome display adapter only)
	5 Blink on
	7 Reverse video on
	8 Concealed on
    
(in my implementation - text attributes are ignored)

*Foreground colors:*

	30 Black
	31 Red
	32 Green
	33 Yellow
	34 Blue
	35 Magenta
	36 Cyan
	37 White

*Background colors:*

	40 Black
	41 Red
	42 Green
	43 Yellow
	44 Blue
	45 Magenta
	46 Cyan
	47 White
    
ANSI defined colors and CGA colors use different codes so translation table was needed. This table is defined in following array:

{% codeblock lang:c %}
static int ansi_to_cga_color[] = {
	  0, /* black */
	  4, /* red */
	  2, /* green */
	  7, /* yellow */
	  1, /* blue */
	  5, /* magenta */
	  3, /* cyan */
	  7, /* white */
};
{% endcodeblock %}

All changes are done in `lib/printfmt.c` file in `6e626f5e1fa1cb8d1daccccbd775c29faf7f836a` commit.

To test this feature I added ANSI coloring to one of of cprintf's:

{% codeblock lang:c %}
	cprintf("[32;45m395[40;31m decimal [37mis %o octal!\n", 395);
{% endcodeblock %}
	
And the result is:

{% img center /images/lab1_ansi_result.gif %}

---

**Exercise 11.** Implement the backtrace function as specified above. Use the same format as in the example, since otherwise the grading script will be confused. When you think you have it working right, run make grade to see if its output conforms to what our grading script expects (you should pass the Count and Args tests), and fix it if it doesn't. After you have handed in your Lab 1 code, you are welcome to change the output format of the backtrace function any way you like.

Calling stack for C programs has following layout:

{% img center /images/lab1_c_stack_layout.gif %}

I created the following C structure to read parameters from calling stack:

{% codeblock lang:c %}
	struct c_frame {
	    	void *prev_ebp;
  	    void *eip;
  	    unsigned int args[5];
	};
{% endcodeblock %}

I decided to limit it to 5 function arguments since not a single function in current JOS kernel uses more than 4 - so this should be enough. Now the only problem is how to know when to stop recursive traversing of stack and for that answer, following line is from `kern\entry.S`:

{% codeblock lang:c %}
	movl    $0x0,%ebp                       # nuke frame pointer
{% endcodeblock %}
	
So this way - when we reach ebp with value 0 - we know that we should stop traversing stack. This is partial code of traverse:

{% codeblock lang:c %}
    fp = (struct c_frame *) read_ebp();
    while (NULL != fp) {
	    /*
	      * Do something with current stack data
	      */
		fp = (struct c_frame *) fp->prev_ebp;
    }
{% endcodeblock %}

The rest of this exercise is simply printing arguments using:

{% codeblock lang:c %}
	fp->args[0] … fp->args[4]
{% endcodeblock %}

---

**Exercise 12.** Modify your stack backtrace function to display, for each eip, the function name, source file name, and line number corresponding to that eip.

Excellent tutorial on stabs debug format can be found [here](http://docs.freebsd.org/info/stabs/stabs.pdf).

Stabs structure and constants are defined in `inc\stab.h`. I decided to do both of those functions in one commit (`036672789b00140a951cc216f4ae8066b4c28a00`). Here is the result:

{% img center /images/lab1_backtrace_out.gif %}

and grade script for this lab is satisfied:

{% img center /images/lab1_grade_result.gif %}

---

**EXTRA point** This makes pretty useful backtrace function but I didn't want to guess how many arguments there are to the function (by default we print 5 of them no matter how many of them function uses) nor arguments names so I decided to use `N_PSYM` stab tag to calculate number of arguments and get argument names. Now the same output with this enhancement is a bit more useful:

{% img center /images/lab1_backtrace_out_en.gif %}

We can easily spot from this output that `monitor` function has one argument named `tf`, that `i386_init` has no arguments etc, etc. This will make troubleshooting a bit easier.

This enhancement is in commit (74818a8a7afc72da67f4c43a48a7e6bcd2f73960).

