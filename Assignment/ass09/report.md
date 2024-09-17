### 1.

Initially, a TLB miss occurs, prompting the access of the page table. Subsequently, the operating system locates the Page Table Entry (PTE) associated with the page and determines that it is not present in physical memory (indicated by the present bit being 0). The operating system proceeds to search for an available physical page frame to map the virtual page to. If no such page frame exists, the replacement algorithm is invoked. Following this, the operating system identifies a page to be swapped out based on a specific rule and releases the corresponding page frame. Lastly, the operating system swaps the desired page into the found physical frame and retries the instruction.



### 2.

![image-20230523094452951](C:\Users\itbil\AppData\Roaming\Typora\typora-user-images\image-20230523094452951.png)







![image-20230523094430432](C:\Users\itbil\AppData\Roaming\Typora\typora-user-images\image-20230523094430432.png)