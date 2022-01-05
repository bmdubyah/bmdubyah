---
layout: post
title:  "Terminal Challenge - Yara Analysis"
date:   2021-12-31 01:34:39 -0800
categories: kringlecon2021 terminal
---

This is terminal challenge "Yara Analysis" in the 2021 SANS Holiday Hack Challenge (https://2021.kringlecon.com/). Objective:

>HELP!!!
>
>This critical application is supposed to tell us the sweetness levels of our candy
>manufacturing output (among other important things), but I can't get it to run.
>
>It keeps saying something something yara. Can you take a look and see if you
>can help get this application to bypass Sparkle Redberry's Yara scanner?
>
>If we can identify the rule that is triggering, we might be able change the program
>to bypass the scanner.
>
>We have some tools on the system that might help us get this application going:
>vim, emacs, nano, yara, and xxd

>The children will be very disappointed if their candy won't even cause a single cavity.

This challenge was all about modifying a binary to bypass Yara rules (https://github.com/VirusTotal/yara). By their on definition,

>YARA is a tool aimed at (but not limited to) helping malware researchers to identify and classify malware samples. With YARA you can create descriptions of >malware families (or whatever you want to describe) based on textual or binary patterns. Each description, a.k.a. rule, consists of a set of strings and a >boolean expression which determine its logic.

The challenge description names of simple tools like text editors and xxd to solve this challenge. This told me that there wasn't going to be anything to fancy other than byte manipulation to solve the challenge. The first step I took was to simply run the binary.

{% highlight shell %}
snowball2@8c75ea42d787:~$ ./the_critical_elf_app
yara_rule_135 ./the_critical_elf_app
{% endhighlight %}

This output suggests that Yara rule number 135 is being triggered. There was a file yara_rules/rules.yar which contained an ordered set of rules.

snowball2@8c75ea42d787:~$ vim yara_rules/rules.yar


{% highlight shell %}
rule yara_rule_135 {
   meta:
      description = "binaries - file Sugar_in_the_machinery"
      author = "Sparkle Redberry"
      reference = "North Pole Malware Research Lab"
      date = "1955-04-21"
      hash = "19ecaadb2159b566c39c999b0f860b4d8fc2824eb648e275f57a6dbceaf9b488"
   strings:
      $s = "candycane"
   condition:
      $s
}
{% endhighlight %}

This was triggering off of the word "candycane". So, I used vim and xxd to convert the binary into its corresponding hex representation using vim command ":%!xxd".

![Yara Hex](/assets/kringlecon2021/yara_analysis/yara_analysis_convert_to_hex.jpg)

![Yara Rule 135](/assets/kringlecon2021/yara_analysis/yara_analysis_rule135.jpg)


The hex representation for candycane is 63616e647963616e65. So, I changed 63 to 64 to make the string become "dandycane"

![Yara Candycane](/assets/kringlecon2021/yara_analysis/yara_analysis_candycane.jpg)

Revert back to binary with vim command ":%!xxd -r" and ":wq" to save.

![Yara Convert to Binary](/assets/kringlecon2021/yara_analysis/yara_analysis_convert_to_binary.jpg)

Re-run the application

{% highlight shell %}
snowball2@8c75ea42d787:~$ ./the_critical_elf_app
yara_rule_1056 ./the_critical_elf_app
{% endhighlight %}

Now we are snagging on rule 1056. Let's take a look at that:

{% highlight shell %}
rule yara_rule_1056 {
   meta:
        description = "binaries - file frosty.exe"
        author = "Sparkle Redberry"
        reference = "North Pole Malware Research Lab"
        date = "1955-04-21"
        hash = "b9b95f671e3d54318b3fd4db1ba3b813325fcef462070da163193d7acb5fcd03"
    strings:
        $s1 = {6c 6962 632e 736f 2e36}
        $hs2 = {726f 6772 616d 2121}
    condition:
        all of them
}
{% endhighlight %}

So we see that it is looking for 2 strings that are represented in hex - "6c6962632e736f2e36" and "726f6772616d2121". Let's open the executable back up in vim and convert to the hexadecimal representation using ":%!xxd", and do a search for those values

![Yara program string](/assets/kringlecon2021/yara_analysis/yara_analysis_program.jpg)

![Yara libc string](/assets/kringlecon2021/yara_analysis/yara_analysis_libc.jpg)

There are two strings. $s1 (6c6962632e736f2e36) is the hex representation for "libc.so.6". This is a shared library object that the application depends on for execution so we cannot alter that. $hs2 is the hex representation for "rogram!!". This we should be able to safely change and not alter program behavior. The important note about yara rule 1056 is that the condition is "all of them". This means that both $s1 and $hs2 must be true. So, as long as we can make one of those conditions false we can bypass the rule.

![Yara modify string](/assets/kringlecon2021/yara_analysis/yara_analysis_modifyprogram.jpg)

After making the change, converting back to binary (:%!xxd -r), saving, and re-running the program we get.

{% highlight shell %}
snowball2@d6f4f313594d:~$ ./the_critical_elf_app
yara_rule_1732 ./the_critical_elf_app
{% endhighlight %}

Rule 1732 now. Let's take a look

{% highlight shell %}
rule yara_rule_1732 {
   meta:
      description = "binaries - alwayz_winter.exe"
      author = "Santa"
      reference = "North Pole Malware Research Lab"
      date = "1955-04-22"
      hash = "c1e31a539898aab18f483d9e7b3c698ea45799e78bddc919a7dbebb1b40193a8"
   strings:
      $s1 = "This is critical for the execution of this program!!" fullword ascii
      $s2 = "__frame_dummy_init_array_entry" fullword ascii
      $s3 = ".note.gnu.property" fullword ascii
      $s4 = ".eh_frame_hdr" fullword ascii
      $s5 = "__FRAME_END__" fullword ascii
      $s6 = "__GNU_EH_FRAME_HDR" fullword ascii
      $s7 = "frame_dummy" fullword ascii
      $s8 = ".note.gnu.build-id" fullword ascii
      $s9 = "completed.8060" fullword ascii
      $s10 = "_IO_stdin_used" fullword ascii
      $s11 = ".note.ABI-tag" fullword ascii
      $s12 = "naughty string" fullword ascii  
      $s13 = "dastardly string" fullword ascii  
      $s14 = "__do_global_dtors_aux_fini_array_entry" fullword ascii
      $s15 = "__libc_start_main@@GLIBC_2.2.5" fullword ascii
      $s16 = "GLIBC_2.2.5" fullword ascii
      $s17 = "its_a_holly_jolly_variable" fullword ascii
      $s18 = "__cxa_finalize" fullword ascii
      $s19 = "HolidayHackChallenge{NotReallyAFlag}" fullword ascii
      $s20 = "__libc_csu_init" fullword ascii
   condition:
      uint32(1) == 0x02464c45 and filesize < 50KB and
      10 of them
}
{% endhighlight %}

There is a lot going on here. But if we look at the condition:

{% highlight shell %}
uint32(1) == 0x02464c45 and filesize < 50KB and
10 of them
{% endhighlight %}

We see that 3 things need to be true, and therefore only need to defeat 1 of them to bypass the rule. The easiest of those three was the filesize check. We can easily pad the binary with null bytes to make it larger than 50kb

{% highlight shell %}
snowball2@d6f4f313594d:~$ dd if=/dev/zero bs=1 count=50000 >> the_critical_elf_app
50000+0 records in
50000+0 records out
50000 bytes (50 kB, 49 KiB) copied, 0.16452 s, 304 kB/s
{% endhighlight %}

Re-run the app, and we have it!

{% highlight shell %}
snowball2@d6f4f313594d:~$ ./the_critical_elf_app
Machine Running..
Toy Levels: Very Merry, Terry
Naughty/Nice Blockchain Assessment: Untampered
Candy Sweetness Gauge: Exceedingly Sugarlicious
Elf Jolliness Quotient: 4a6f6c6c7920456e6f7567682c204f76657274696d6520417070726f766564
{% endhighlight %}
