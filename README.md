# Dirty Cow CVE-2016-5195 Presentation Notes

## For ECE 9069: Introduction to Hacking (Cybersecurity) course, during master's degree in Software Engg. at Western University, ON, Canada

* It is a linux based vulnerability which existed since 2007 and got fully patched in 2017.
* It is a vulnerability since kernel version 2.6.22 until patched.
* It escalates privileges of the user by using race condition and copy-on-write mechanism.
* So essentially a normal user can gain root access and can read, edit, delete protected content and perform functions only designated by the root user. 
* It was discovered by Phil Oester in 2016.


____
## CVE Details

* The CVSS score is 7.2 making it qualitatively highly rated.
* Complete confidentiality is compromised as we get root access.
* Integrity is also fully compromised as there is a total loss of system protection.
* Availability of resource is complete and the attacker can make the system unavailable to the user.
* It has low complexity and even a neophyte can exploit this vulnerability.
---
| CVSS Score | 7.2 |
| -------- | -------------- |
| Confidentiality Impact | Complete (There is total information disclosure, resulting in all system files being revealed.) |
| Integrity Impact | Complete (There is a total compromise of system integrity. There is a complete loss of system protection, resulting in the entire system being compromised.) |
| Availability Impact | Complete (There is a total shutdown of the affected resource. The attacker can render the resource completely unavailable.) |
| Access Complexity | Low (Specialized access conditions or extenuating circumstances do not exist. Very little knowledge or skill is required to exploit. ) |
| Authentication | Not required (Authentication is not required to exploit the vulnerability.) |
| Gained Access | None |
| Vulnerability Type(s) | Gain privileges | 
| CWE ID | 362 |

___

## Critical concepts
### Privilege Escalation
* DirtyCow escalated the privileges of the local user. This means the user now has elevated access to resources which were otherwise protected from them.
* This is done by exploitation of bug, design flaw, or a configuration oversight.
* Two forms:
    * Vertical: Lower privilege user access access higher privilege  content or functions
    * Horizontal: access other users content

![Privilege Escalation](https://upload.wikimedia.org/wikipedia/commons/thumb/c/cc/Privilege_Escalation_Diagram.svg/440px-Privilege_Escalation_Diagram.svg.png)

### copy-on-write(COW) mechanism
* It is a resource management technique where
creation of a resource is deferred until modification
* Old resource is shared till the first write is donw.
* It is also known as implicit sharing or shadowing.
* For example, process P and Q shares the same data in Page 1, Page 2 and Page 3. Process P then tries to modify the data, triggering COW mechanism.
* The figures below shows what happens when Process P tries to modify page 3.  COW is triggered and the modification would be done on the copy of the page3. This way Process Q would be unaffected.


![ProcessP before modification](https://media.geeksforgeeks.org/wp-content/uploads/20200512180436/11150.png)
![ProcessP after modification](https://media.geeksforgeeks.org/wp-content/uploads/20200512181458/12127.png)

### race condition
* A race condition arises when multiple threads read and write the same variable.
* Same data is accessed and tried to be changed by the threads at the same time.
* Threads are “racing” against each other to change the data

## Dirty Cow vulnerability and exploitation
 Let’s see how the exploit works.
* Linux like many other OS does not give direct access to physical memory. 
* It has something called Virtual Memory which is dedicated to a program. 
* The memory is divided into a basic unit called Page. 

![Virtual Memory and Physical Memory](https://i.imgur.com/k1GV4yi.png)

For our example we have a read only file called ‘root_file’.
We have an exploit program to make use of the Dirty Cow vulnerability.We start by making a private mapping of the file in the virtual memory.
![Virtual Memory and Physical Memory](https://imgur.com/VRgMSVk.png)
![Virtual Memory and Physical Memory](https://imgur.com/6XXnB5a.png)
We then run 2 threads in parallel.

The 1st thread has a write function which consists of multiple processes such as page fault management etc. Thus, Write can then be divided into two non-atomic actions:
Broadly divided into
1)	Finding actual physical memory
2)	Writing into the physical memory


![Virtual Memory and Physical Memory](https://imgur.com/IWOJqR8.png)

The 2nd thread which is run in parallel consist of the madvise() system call which tells the kernel to discard the private memory 
mapping. We try run this between the two set of Write actions.

![Virtual Memory and Physical Memory](https://imgur.com/gxa9HkR.png)


The write then triggers a COW again, the madvise would delete again.

![Virtual Memory and Physical Memory](https://imgur.com/Po99xls.png)

We do this millions of times and try to race the two threads such that we eventually trick the kernel (which can write into any files) into writing into the original read-only file.

![Virtual Memory and Physical Memory](https://imgur.com/SL3mZFk.png)

By this way,we can write into files that can change root password and give us access to root.

---
## Impact of Dirty COW
1. Android phones(upto version 7 Android Nougat) could be rooted using DirtyCow
1. Backdoor entry to phones could be created
1. Tools used this vulnerabilty to root phones for custom rom and run root permisiion apps
1. Linux holds a major share in server market. So wide number of machines were vulnerable.("Steaks are High!" :P)

___
## Try a Demo
* Download a compromised Ubuntu version from [download link](http://old-releases.ubuntu.com/releases/14.04.0/) and install it on a VM.
* In case you want to check the vulnerability try any of the proof of concepts of Dirty cow exploits from [Github](https://github.com/dirtycow/dirtycow.github.io/wiki/PoCs).
* Follow the instuctions in the PoC and test it on the Ubuntu VM.
* Once example is [Dirty Cow Tutorial](https://www.youtube.com/watch?v=YoeuGnF_2Qk). Warning!: The audio in the begining isn't the best. Use this tutorial to try this [exploit](https://gist.github.com/joshuaskorich/86c90e12436c873e4a06bd64b461cc43)



___
## References
Note: Note: References posted at different sections of this notes is also listed below.

1. https://en.wikipedia.org/wiki/Dirty_COW
1. https://en.wikipedia.org/wiki/Privilege_escalation
1. https://www.cvedetails.com/cve/CVE-2016-5195/?q=CVE-2016-5195
1. https://www.geeksforgeeks.org/copy-on-write
1. https://media.geeksforgeeks.org/wp-content/uploads/20201228232441/gfgdiagram.png
1. https://en.wikipedia.org/wiki/Race_condition
1. https://www.geeksforgeeks.org/race-condition-vulnerability/
1. https://ideas.ted.com/wp-content/uploads/sites/3/2018/09/cow_farts_istock.jpg?resize=750,450
1. https://www.cs.toronto.edu/~arnold/427/18s/427_18S/indepth/dirty-cow/index.html
1. https://www.cs.toronto.edu/~arnold/427/18s/427_18S/indepth/dirty-cow/demo.html
1. https://www.youtube.com/watch?v=kEsshExn7aE
1. https://www.youtube.com/watch?v=CQcgz43MEZg&t=307s
1. https://allenhan.ca/dirtycow/
1. https://arstechnica.com/information-technology/2017/09/in-a-first-android-apps-abuse-serious-dirty-cow-bug-to-backdoor-phones/
1. https://www.bleepingcomputer.com/news/security/first-android-malware-discovered-using-dirty-cow-exploit/
1. https://www.redhat.com/cms/managed-files/styles/wysiwyg_full_width/s3/IDCGraphic2.png?itok=seNQSvXR
1. https://dirtycow.ninja/
1. https://gist.github.com/joshuaskorich/86c90e12436c873e4a06bd64b461cc43
1. https://imgresizer.eurosport.com/unsafe/1200x0/filters:format(webp):focal(1208x573:1210x571)/origin-imgresizer.eurosport.com/2020/06/23/2837892-58522868-2560-1440.jpg
