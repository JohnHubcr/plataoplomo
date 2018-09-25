# AFC Path Traversal Infoleak
This vulnerability is not big issue to security but rather goes against the security design guidelines

## How I found the bug
AFC is the protocol Apple invented for making applications able to talk to the iphone and perform maintanance operations on the device over USB, it is meant for iTunes primarily.

However, Apple File Conduit has been reverse engineered by multiple security researchers and third party clients are public such as AFCClient for accessing the protocol as an API over a command line.

I found out that invoking the directory listing AFC call is not evaluated properly making an attacker able to view directories outside the sandbox.

I found that out by invoking `afcclient ls ../../../../../../../../usr/libexec/` and it worked, the reason that worked is because the daemon serving the protocol is in that directory and thus by design has rights to list that directory.

../ simply stands for the upper directory and the issue here is that the service com.apple.afc allows traversing the directories causing this infoleak vulnerability.

Other directories that are viewable with this bug are the root (/) and (/System/Library).

afcd runs as root thus in theory that means traversal would make private system directories listable but sadly sandbox restrictions still apply mostly and you can see that back in the system logs.

Apple is not notified about this vulnerability simply because I do not see how this can be abused, but this bug or rather, feature is against their security guidelines.

