# iBooks
iBooks up to iOS 12.1 is vulnerable to a type confusion vulnerability.

Luckly this vulnerability is only exploitable via USB  through a paired computer but it can still be quite nasty.

The minor result that the PoC shows is that an attacker could compromise iBooks and make it permanently crash, persistantly accross reboots.

Chances are that it's possible that an attacker can gain control over memory as well as the type of Objective-C objects ofcourse differs in size and way that they are read.

The bug could be abused in malware but can not be used for privilege escallation as the highest user it applies to is mobile, though it can be used to escape the sandbox.

## How I found the bug
I read online about the Apple File Conduit protocol, a protocol designed by Apple for iTunes to communicate with the iPhone over USB.

On iOS AFC can be used for a lot of maintanance operations, but in my case I used it for uploading and viewing files, with the sandbox restrictions applied.

I found out that in the Media directory of the iPhone contained few directories for Metadata of media such as photos and ebooks and those directories and their subdirectories.

The Books directory contained a file named Purchases.plist, which is in fact just a propertylist file containing info about the books that the user has in it's bookshelf of iBooks.

The propertylist format is a standard built on top of XML and it resembles a dictionary with key-value based objects.

Keys are strings but the values can be of different types supported by the standards.

Some types values can have are: integer, string, data, real, date, true, false, dict and array.

The data type is read as base64 encoded data.

The string resembles an NSMutableString, the integer an NSInteger and real an NSNumber.

Arrays and Dictionaries are references to NSMutableArray and NSMutableDictionary.

This made me think of a scenario where the serializer does not check the type correctly thuswhen changing the type fails to read more data and type confusion would of occur.

## Proof of Concept

I noticed that the code for reading the Purchases propertylist did not check this correctly so that when a type was expected a bug would occur.

I changed the path value in the Purchases propertylist  to number 4141414141 and changed the type to integer.

Secondly I found that instead of caching the books into an sqlite database, iBooks parses the propertylist each launch thus the crash would occur each time.

I checked the crashlogs and saw that indeed type confusion occured but that it is selector based thus harder to exploit than I thought, therefore no registers were overwritten but a denial of Service still occurs ofcourse.

In theory it is still possible to use this vulnerability for exploitation but that requires some further studies that I currently not am interested to do because of the scope of this vulnerability.

Perhaps in the future I will demonstrate a memory corruption exploit as well based on this type confusion.

## The Patch

The issue is that apple did not sandbox the Purchases file thus making it controllable by attackers and at the same time does not check the type of the property list values but rather expects one when serializing the propertylist.

I reported the vulnerability to Apple Inc. and got response the same day however eventhough they promised to patch it soon, this bug is not patched in iOS 12.1 yet.

A second e-mail confirmed that they are still investigating it but I am skeptical about that, I think this issue is not going to be patched anytime soon.

Therefore I give the advice not to pair your phone with devices in public spaces such as a library because the pairing record could be stolen by local attackers and used in malware.
