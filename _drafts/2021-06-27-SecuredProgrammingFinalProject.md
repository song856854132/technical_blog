# Secure Programming Final Project
###### tags: `class notes`

## part 1. Buffer Overflow and Advanced Memory (Stack) Protection
### What Does Buffer Overflow Mean?
When executing a program, which may contain some variables, such as an array, a distributed space by function malloc, etc., it must generate buffers, which are used to store variables' values under normal circumstances, utilized by attackers to implant shellcodes for obtaining the control.

The causes of buffer overflow vary from over-long user input covering the return address to the user input exceeding the extreme value of the variable. Even if the vulnerabilities of the buffer overflow is triggered, it's not easy to implant shellcode to achieve the goal because there are some defense mechanism exsited into the system's memory.

### What Does Advanced Memory Protection Mean?
When executing a program, the system adopts some specific approaches to keep the program operating normally.
* **Structured Exception Handling (SEH)**: A protection mechanism to prevent buffer overflow with handling specific abnormal code situations.
    * The grammar is like "try...catch...finally".
    * Use a linked list as it contains a sequence of data records.
    * When an exception occurs, the OS will go through this list and check for the suitable exception function or hitting the last, as Figure 1.

![](https://i.imgur.com/oH6blOK.jpg)
--- Figure 1. The linked list containing the data records.

* **Stack Smashing Protection (SSP)**: Also known as "canary-based protection", which is used the canary value as detecting if the user input overflows to next layer in the stack, as Figure 2.

![](https://i.imgur.com/vfBoCzX.jpg)
--- Figure 2. The location of canary value in the stack.

* **Address Space Layout Randomization (ASLR)**: Increase the difficulty for the attacker to predict the destination address by randomizing the layouts of heaps and stacks, for preventing the attacker from directly targeting the location of the vulnerable code and achieving the purpose of resisting overflow attacks. There are two randomization approaches used in operating systems.
    * **Kernel Address Space Layout Randomization (KASLR)**: Load the same binaries but each time at a different and random location of memory.
    * **Kernel Address Randomized Link (KARL)**: Load a different kernel at a binary level each time and put it at the same place in memory. (Apply into OpenBSD System which is a kind of Unix-similar operating system.)



The other one has possibilities to bypass the defense technique of W XOR X (W^X), named **Return-oriented Programming (ROP)**.
* Extract the available instruction fragments, called ***gadgets***, ended by “ret”, as Figure 3.
* Procedure:
    * Operate the stack-related registers, e.g. esp, ebp, eip.
    * Control the flow of the program.
    * Execute the corresponding gadget.
    * Implement the attacker’s preset target.

![](https://i.imgur.com/DJQgX9P.jpg)
--- Figure 3. The return instructions (gadgets) in the stack.

Due to many assignments about dynamic analysis in the course, I choose some tools of static analysis to introduce.
* SCA (Fortify)
    * 17 support languages
* CxSuite (Checkmarx)
    * Web application
* CodeSecure (Armorize)
    * Combination of software and hardware

The comparison among the three tools could refer to the table below.

![](https://i.imgur.com/8ypM9MT.jpg)
--- Table 1. The table of the comparison among the three tools.

Because the three tools above need to charge, there is a free tool, named **Flawfinder**.
* Only suppot C/C++ and deploy on Linux systems.
* But identify quickly possible security vulnerabilities and generate simple reports according to risk level.

Just input two command lines to install:
* sudo apt-get update -y
* sudo apt-get install -y flawfinder

And enter the command line in the following format to execute the detection behavior:
* flawfinder _filename_: detect the specified file.
* flawfinder ./: detect all files in this directory.
* flawfinder --minlevel 4 ./: detect all files in this directory and just report the vulnerabilities whose risk levels are upper than 4.

I attach the process of implementaion by using Flawfinder to detect a file, named pwnshell.c, as Figure 4.

![](https://i.imgur.com/7rq79fZ.jpg)
--- Figure 4. The source code of pwnshell.c

The final results after execution is as Figure 5.

![](https://i.imgur.com/7NsdFnu.jpg)
--- Figure 5. The final results with vulnerabilities.

The analysis summary is as Figure 6.

![](https://i.imgur.com/hFxIrEP.jpg)
* Hits = 2: numbers of error found.
* Lines analyzed = 9: numbers of lines analyzed.
    * 724 lines/second: consumed time during analyzing.
* Physical Source Lines of Code (SLOC) = 9: numbers of lines of the source code
* Hits@level: numbers of error at each level from 0 to 5.
* Hits@level+: acculmulated Hits@level.
    * For instance, [0+] = 3, means the numbers of error whose risk level are upper than 0, so [1+] would be down to 2 (3 - 1 = 2).
* Hits/KSLOC@level+: (Hits@level+ * 1000) / SLOC, for detecting the error rate in a thousand of lines.
* Minimum risk level = 1: Report the error whose risk level is at least equal to 1.

The next part is the paper study. The reason I chose the paper is that I've searched the knowledges about ASLR first. So, I would like to understand more about the technique of ASLR how to apply into the kernel layer and the cloud environment.

### KASLR-MT: Kernel Address Space Layout Randomization for Multi-Tenant Cloud Systems, Journal of Parallel and Distributed Computing.

Cloud computing has rapidly developed and widely applied in practice in recent year. It has three properties such as flexibility, scalability and reliability, but only its security has always been worried by users, because the cloud computing paradigm introduces new scenarios where security protection techniques are weakened or disabled to obtain a better performance and resources utilization.

Kernel ASLR (KASLR) is a very effective technique that thwarts unknown attacks but unfortunately its randomness have a significant impact on memory deduplication savings. One of both techniques, KASLR-ON, which enables the kernel randomization, is with the high level of security, and the other one, KASLR-OFF, disabled the kernal randomization instead, is with better performance and resources utilization. 

Therefore, the authors propose KASLR-MT, a new Linux kernel randomization approach to achieve that attackers exploiting a kernel vulnerability from local, remote, inter-VM, intra-VM, inter-Tenant and intra-Tenant must face the full kernel randomization protection with almost no effect in the memory deduplication.

![](https://i.imgur.com/NpbpzWf.jpg)
--- Figure 1. The design of KASLR-MT.

There are some elements having to be introduced first.
* **Host**: The leader in the whole cloud environment.
    * Owns a table with one-to-one correspondence, linking Tenant ID and a unique random key. 
* **Guest**: The user in the whole cloud environment.
    * Each guest will be a tenant at the right part of the figure. A tenant must owns at least one virtual machine (VM).
    * Each virtual machine will be separated into some regions, whose addresses are determined by Address Producer computing by the corresponded key, e.g. (T1, K1), (T2, K2), for storing data just like text files.
    * If the addresses generated by the same key, e.g. K1, then the addresses of regions A in VM1 must be same with ones in VM2.
    * If the addresses generated by the different keys, e.g. K1 and K2, then the addresses of region A in T1 must be different with ones in T2.

The Multi-Tenancy KASLR (KASLR-MT) gives the host machine the ability to decide the locations of different kernel memory regions of guest virtual machines. There are two posibilities for each kernel memory region.
* **None/Low impact memory regions**: The memory region has low and negligible impact when all memory base address are randomized, so its base address doesn't need to be shared with any other kernel. It means the base address will be the different among virtual machines in both same and diffrent tenants.
* **Medium/High impact memory regions**: The memory region has medium and high impact when all memory base address are randomized, so its base address must need to be shared with other kernels belonging to the same tenant. It means the base address will be the same among virtual machines belonging to the same tenant and different among virtual machines of diffrent tenants.

With KASLR-MT, kernel memory base addresses can be randomized in two different way:
* **Per-Tenant**: Guests will have the same memory base address, of course by randomized, for a particular memory. It's used for memory regions where the impact is ***high*** or ***medium***.
* **Per-VM**: Every time a guest reboots, the region will have a different memory base address and it will not be shared across virtual machines belonging to same or other tenants. It's used for memory regions where the impact is ***low*** or ***none***.

The design of KASLR-MT can protect against external attackers including network applications interacting with the kernel and unknown tenants running virtual machines in the same host machine, even attacks from those virtual machines that share the same kernel address space layout are equivalent to local attacks, in other words, the userspace applications attack its own kernel. From these points of views, the reason for the failures of these attacks is that the exact address of the target data region couldn't be known, so ***the kernel memory layout is unpredictable for  attackers***. 

However, **the KASLR-MT mechanism** would like to combine the advantages of KASLR-ON and KASLR-OFF, which are **high security** and **high performance** respectively. From the above contents, we could see that high security has been met, and then would explain the analysis about high performance.

![](https://i.imgur.com/ddR6hsh.jpg)
--- Figure 2. Redundant memory curve for different number of simultaneous kernels.

The pecentage increases logarithmically as more kernel memories are added. This limit is approximately 70% when kernel randomization is not enabled, 67% in our solution, and 35% when kernel randomization is enabled.

![](https://i.imgur.com/B6jbdCt.jpg)
--- Figure 3. Percentage of redundant memory per kernel region, for 30 kernels.

We can confirm which are the kernel regions benefiting from our solution. The ***Linux code*** region gets exactly the same sharing when kernel randomization is disabled, while the ***Linux data*** region is just with only 2.7% of sharing loss. The ***modules code*** and ***modules data*** regions are also benefiting, similarly, ***vmemmap*** can share a 34.9% of its contents with our solution, an 8.3% less than when kernel randomization is disabled. With regard to ***vmalloc***, we can see that it has similar redundant memory, independently of kernel randomization. Consequently, the solution is close to the best case scenario, i.e., KASLR-OFF, in terms of redundant memory.

## part 2. Shellcode and Privilege Escalation Attacks

### PrivGuard: Protecting Sensitive Kernel Data From Privilege Escalation Attacks, IEEE Access 2018
Kernels of operating systems are written in low-level unsafe languages, which make them inevitably vulnerable to memory corruption attacks. Most existing kernel defense mechanisms focus on preventing **control-data attacks**, sudch as ~~Supervisor Mode Execution Protection~~ (**SMEP**), ~~Kernel Address Space Layout Randomization~~ (**KASLR**), ~~Kernel Control Flow Integrity~~ (**KCFI**), etc.. 
```
SMEP prevents the execution of the code located in a user-mode page while the operating system is running in a higher privilege level.
KASLR makes it much more difficult for attackers to find gadgets in the kernel address space. 
KCFI introduces a solid solution to prevent subverting the kernel’s control flow
```
Recently, attackers have turned the direction to **non-control-data attacks** by hijacking data flow, so as to bypass current defense mechanisms. Previous work has proved that **non-control-data attacks** are the critical threat to kernels. One of the important purposes of these attacks is to achieve privilege escalation by overwriting sensitive kernel data. The goal of our research is to develop a light weight protection mechanism to mitigate **non-control-data attacks** that compromise sensitive kernel data. 

Writer propose an approach that enforces data integrity of sensitive kernel data by preventing the illegal write to these data to mitigate privilege escalation attacks. Aside from previous researchers' solutions, rendering the non-control data either impossible to be manipulated, or harder to be located in the kernel, which suffer from limitaion of high performance overhead, the necessity for higher-privileged execution modes(e.g., hypervisors), dependency on the support of specific hardware features, or incompatibility with the kernel.

The practical method is to monitor the modification of sensitive data in kernels by hooking the system calls without changing the existing linux access control mechanisms, and leverage stack canary to protect the duplicated sensitive data. The main challenge of the proposed approach is to validatethe modification of sensitive kernel data at runtime. The validation routine must verify the legitimacy of the duplicated sensitive data and ensure the credibility of the verification. To address this challenge, we modify the system call entry point to monitor the change of the sensitive kernel data without any change to Linux access control mechanism. Then, we use stack canaries to protect the duplication of sensitive kernel data thatare used for integrity checking. In addition, we protect the integrity of sensitive kernel data by forbidding illegal updates to them.

They also demostrate **PrivGuard**, a framework that enforces data integrity of kernels’ sensitive data by preventing the illegal write to the data, and implement a prototype of **PrivGuard** in Linux kernel. As the result, the performance evaluation show that it incurs an average overhead of 9% on system call and nearly has no impact on I/O and computation. The performance overhead for applications is negligible. The security evaluation for the real-world non-control-data attack shows that **PrivGuard** can defend it effectively.


### Transforming malicious code to ROP gadgets for antivirusevasion, IET InformationSecurity 2019.
:::success
Return-oriented programming (ROP) is a computer security exploit technique that an attacker gains control of the call stack to hijack program control flow and then executes carefully chosen machine instruction sequences that are already present in the machine's memory, called "gadgets". Each gadget typically ends in a return instruction and is located in a subroutine within the existing program and/or shared library code. Chained together, these gadgets allow an attacker to perform arbitrary operations on a machine employing defenses that thwart simpler attacks. 
--- cited from Wikipedia
:::
The downside of current polymorphism techniques lies to the fact that they require a writeable code section, either marked as such in the corresponding Portable Executable (PE) section header, or by changing permissions during runtime. Both approaches are identified by AV software as alarming characteristics or behavior, since they are rarely found in benign PEs unless they are packed. In this paper we propose the use of Return-Oriented Programming (ROP) as a new way to achieve polymorphism and evade AV software. To this end, we have developed a tool named ROPInjector which, given any piece of shellcode and any non-packed Portable Executable (PE) file, it transforms the shellcode to its ROP equivalent and patches it into (i.e. infects) the PE file. After trying various combinations of evasion techniques, the results show that ROPInjector can evade nearly and completely all antivirus software employed in the online VirusTotal service. The main outcome of this research is the developed algorithms for: a) *analysis and manipulation of assembly code on the x86 instruction set*, and b) *the automatic chaining of gadgets by ROPInjector to form safe, and functional ROP code that is equivalent to a given shellcode*.
![](https://i.imgur.com/oO6zN9M.png)

### Detecting Successful Attacks from IDS Alerts Based On Emulation of Remote Shell-codes, 2019 IEEE 43rd Annual Computer Software and Applications Conference(COMPSAC)
Server administrators and security operation center analysts receive alerts from an intrusion detection system and check whether attacks have succeeded. However, it is difﬁcult to handle them quickly because a tremendous number of alerts is generated in a short period of time. We propose a method to identify important alerts that lead to security incidents automatically. The key idea is to determine the success or failure of an attack based on trafﬁc logs and the network behaviors observed during shellcode emulation. We evaluated the proposed method in terms of accuracy and performance and found that it can handle more than 60% of remote shellcodes and cope with practical attack cases.


## part 3. Reverse Engineering for Vulnerability Detection

### Automatically Patching Vulnerabilities of Binary Programs via Code Transfer FromCorrect Versions, IEEE Access 2019
