Download Link: https://assignmentchef.com/product/solved-ece357-computer-operating-systems-set-5
<br>



<h1>Problem 1 — Pag e Tables</h1>

We envision a semi-hypothetical computer architecture (it is just an idealization of the actual X86-32 bit) with the following description of its page table structure (this is essentially identical to what is presented in lecture notes):

<ul>

 <li>32 bit virtual and physical addresses with 4K pages, 2-level page table structure split 10/10 bits.</li>

 <li>The full physical address of the Page Global Directory (PGD) is contained in register CR3</li>

 <li>All Page Directories and Page Tables are stored in whole pages at page-aligned physical addresses</li>

 <li>Both Page Directory Entries and Page Table Entries have the following format (this is a distortion of the truth forX86-32 architecture):</li>

</ul>

Bit 31: Present

Bit 30: Supervisor(=1)/User(=0)

Bit 29: Read permission (n/a for PDE)

Bit 28: Write permission (n/a for PDE) Bit 27: eXecute permission (n/a for PDE)

Bit 26: Dirty (n/a for PDE)

Bit 25: Accessed (n/a for PDE)

Bits 24-20: Not used, always 0

Bits 19-0: Page Frame Number

<ul>

 <li>For Page Directory entries, the RWXD and A bits are not applicable and can be assumed as 0.</li>

</ul>

Now, we are going to trace out the creation of page tables and the on-demand paging-in and allocation of page frames by the kernel using some simple, idealized examples. We make the following assumptions:

<ol>

 <li>The kernel has a contiguous pool of free page frames. At the moment that our example process comes to life, the lowest-numbered free page frame is at physical address 0X00100000</li>

 <li>As the kernel requires page frames, it allocates them sequentially from this pool. The pool is contiguous and wewon’t run out of pages. <strong>Memory is plentiful and the PFRA is not active.</strong></li>

 <li>At process creation time, the kernel allocates a single page frame to serve as the PGD for that process’s addressspace. All other pages, including pages required to store Page Tables, are demand-paged. I.e. the page frames are not pre-allocated, but are allocated and “plumbed in” as page faults occur.</li>

 <li>The kernel will lay out a process virtual address space starting at virtual address 0x08040000 for the text region.The data and bss regions will be <strong>contiguous </strong>with the text region (no guard zones between memory regions); each region of course must be page-aligned. The stack will be allocated at the highest virtual address allowed for user processes (0xBFFFF000 – 0xBFFFFFFF) and grows down as needed on demand.</li>

 <li>We will assume that for our example case, the <strong>stack does not need to grow beyond a single page</strong>. We’ll also simplify things and ignore the existence of the C library, instead assuming that the main() function is the first thing executed when the program is exec’d, and that the <strong>text region fits within a single page</strong>. We’ll assume that the compiler is set for no optimization, so operations occur as written in the C source.</li>

</ol>

You will be making a diagram of the page table structure for a given process running a small program. Here is a sample of what the diagram would look like:

this sample diagram depicts a PGD at PA 0x00300000, and there is a single virtual page at VA 0x40000000 which is mapped to PFN 0x00302. PFN 0x00301 is being used to hold one page table (which maps VA 0x40000000 0x403FFFFF) The virtual page in question has RW permissions, is a user-mode page, and has not been accessed or dirtied. <strong>Note: </strong>the example above is meant to show how the diagram should be constructed. <strong>It is not the correct beginning </strong>for the solution of the actual problem below.

<strong>Now, trace out the program below</strong>. Provide the state of the page table structure at the indicated point in the code, including the value of register CR3, the contents of the PGD, and the contents of any and all intermediate Page Tables, in a form similar to the example above:

int b[40]; char d[]=”ABC123″;

main() {

f1(); /* CONSIDER THE STATE OF THE PAGE TABLES AT THIS POINT */ }

f1() {

b[0]=d[3];

}

<em>Some hints: </em>Make a sketch of the VA space of the process. Trace out each operation of the program in order and determine what virtual memory regions are accessed and for what (r/w/x). Then trace out how the kernel handled the demand-paging.

<h1>Problem 2 — Thoughts about Memory</h1>

2A) One way to think about paging-out is that physical memory is a scarce, faster resource which caches virtual memory. As with any cache where cache capacity is scarce, we’d want the items that are in the cache to be the ones that are most likely to be used again soon. Conversely, we’d want to evict items from the cache based on a Least Recently Used (LRU) policy.

Question to ponder: Does the scheme described in the lecture notes, with the use of the hardware PTE “A” bits and the PFRA algorithm, achieve true LRU behavior? If not, does it come close and how?

2B) In your own words, what does the radix-64 tree of the address_space data structure do? Under what circumstances is this data structure used during paging-in operations?

2C) When handling a page fault, where the faulting virtual address is below the user/kernel line, which data structure(s) is consulted to determine if a SIGSEGV should be delivered? Which fields of that struct in particular are examined?

<h1>Problem 3 — Smear program</h1>

We’re going to take advantage of the mmap system call to do something which is kind of awkward with the traditional read/write file I/O interface. I’ll call the program smear and it will be invoked like this:

smear TARGET REPLACEMENT file1 {file 2….}

For each of the named files, search for every instance of the string given by the first argument and replace it with the replacement string given by the second argument. <strong>Note: </strong>to simplify the problem, both target and replacement strings must be the same number of characters.

Do this with mmap. Again, to simplify the problem, you aren’t required to handle cases where the file is so large that it can’t be mapped in its entirety into virtual address space (but still remember to check for and properly report any applicable system call errors)


