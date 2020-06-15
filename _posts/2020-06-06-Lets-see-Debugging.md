---
title: Lets see debugging
layout: default
---

<h1>Ways of Debugging on a embedded platform</h1>
<ul>
<li>Getting the core dump on a crash</li>
<li>Connecting GDB on a running system</li>
</ul>

<h2>Getting the core dump on a crash</h2>
<p>What is required for identifying a crash?<br>
<ul>
<li>PC address and other register contents</li>
<li>Stack contents for the call trace</li>
<li>System status information</li>
</ul>
</p>
<h3>PC address and other register contents</h3>
<p>We know that when the crash occured in the system (segmentation fault, floating point exception ... etc) the exception handler routine is called. The exception handler routine is the one which
which prints the panic message and all other stuffs. We put our prints to print the required contents(address caused the exception, stack pointer and other useful registers) for debugging. So in our bootloader
{% for posts in site.posts %}
	{% if posts.title contains "BeagleBoneBlack" %}
		<li><a href="{{ posts.url }}">{{ posts.title }}</a></li>
	{% endif %}
{% endfor %}
<br>
we just add the prints in the exception handler and print the register contents(LR contains the return address which is the cause for the exception, r13 stack pointer)
So with gdb and the elf file we can identify the line caused the crash.
</p>
<h3>Stack contents for the call trace</h3>
<p>We got the stack pointer from the LR register so printing the stack contents is just incrementing the address and printing. So now with the stack contents we just compare all the address with the elf symbol table and can get the call trace</p>

<h3>System status information</h3>
<p>Printing thread struct information, process related information, memory related information and other important contents specific to that system can be printed</p>

<h2>We will go through the code in mBootloader</h2>
<p><a href="https://raw.githubusercontent.com/slpp95prashanth/Beaglebone-mBootloader/shell/arch/arm/cpu/armv7/vectors.S">vestor.S</a> Here is this file we register the vector table.
In ARMv7 there are seven modes of execution in that data_abort is one of the mode the cpu switches when there is a segmentation fault, floating point exception ... etc. Under the data_abort routine we setup the
stack and save the previous context then call <a href="https://raw.githubusercontent.com/slpp95prashanth/Beaglebone-mBootloader/shell/arch/arm/cpu/armv7/show_regs.c">show_regs</a> function whch prints all the
register contents.</p>

<p>Note:<br>
Here the contents are with respect to the BeagleBoneBlack soc.
</p>
