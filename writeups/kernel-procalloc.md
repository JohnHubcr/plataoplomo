# Bug in XNU process allocation
A vulnerability in XNU of iOS 12.1 and below exists providing local attackers the ability to cause a denial of service in the system of any iOS device through a maliciously crafted app.


## How I found the bug
Bug hunting is painful work because you need to put in a lot of time and sometimes are left with no results.

Luckily this time I got a result when reading through the source code of XNU and stumbled accross a serious design flaw.

XNU like any kernel can spawn processes and therefore needs to allocate them before they can be used.

Some systemcalls such as execve can be used anywhere in the system to invoke the allocation of a new process.

Execve for example is a common UNIX systemcall for spawning processes with arguments and environment variables.

Ofcourse due to the limitations of the sandbox this is not possible. At least that is what I thought.

I looked at the code for spawning processes and saw that a process gets an address space identifier upon allocation, a unique number identifying a process it's address space.

I saw that there are a limited number of address space identifiers and that eventough it still are a lot, these could run out in an unhappy flow scenario.

Therefore I wanted to see if I could race the kernel to make it run out of them.

The systemcall execve should ofcourse call the process allocation but I had high doubts it would reach that point because of the sandbox restrictions, however my hypothesis was incorrect and the following happened:

- XNU accepts the systemcall, even when NULL arguments are provided
- XNU calls the code for allocating a process
- The systemcall is rejected because of sandbox restrictions and returns -1

So that is pretty odd, you can make a call and the mac policies and sandbox restrictions don't apply for the allocation of processes only for the creation of them.

This left me with the goal to write a PoC to make the kernel run out of adress space identifiers (asid).

## The Exploit
I figured that systemcalls from different threads get higher priorities in the queue than the main thread sending them repeatedly therefore thought of calling execve() is as much possible threads.

This approach seemed to be correct and with 10.000 systemcalls per 10.000 thread spawns I was able to make the kernel run out of address space identifiers proving that my race works.

The race condition seems to be quite on my side, as I can win the race in milliseconds every time with this approaches.

A panic occurs and will mention alloc_asid(): Out of ASID number

And that proves the existance of the vulnerability.

I am still investigating new scenarios where this bug is used in race conditions that aim to gain control over allocated address space that is never used for processes.

## The patch
I found this bug in iOS 9.2.1 and immediately reported it to Apple Inc.

After getting a default response they still did not patch this vulnerability after months so I wrote them again.

They replied that they are researching it and that it will be patched soon.

With the release of iOS 10 I started to wonder if Apple Inc. even cares about customers and security, but looking at the patch logs the seem to do yet the vulnerability was not patched yet and I emailed again got the standard response and they said they would look at it again.

With the release of iOS 11 it was still not patched, I e-mailed Apple for the last time and decided to do full disclosure if they did not patched it in 11.1, they said it is investigated again and it would be patched which again, did not happen.

At this point I disclosed my PoC publicly in projects on Github but not many people noticed, Apple neither seems to have because now iOS 12.1 is in beta the bug is still not patched.

## Conclusion
Apple Inc. does not care about lowlevel vulnerabilities unless they are exploited for privilege escallation.

Apple Inc. sometimes responds in an unprofessional way when reporting vulnerabilities and pointing them on how they can improve their software.

The bug is a design flaw and therefore it takes effort to patch the vulnerability, with some revision on the lowest parts of the kernel.

The work of a security researcher is quite lonely and responsible disclosure sometimes is not worth it.
