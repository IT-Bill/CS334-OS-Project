### 1.

| Time                  | HRRN | FIFO/FCFS | RR   | SJF  | Priority |
| --------------------- | ---- | --------- | ---- | ---- | -------- |
| 1                     | A    | A         | A    | A    | A        |
| 2                     | A    | A         | A    | A    | B        |
| 3                     | A    | A         | B    | A    | A        |
| 4                     | A    | A         | A    | A    | D        |
| 5                     | B    | B         | D    | B    | D        |
| 6                     | D    | D         | A    | D    | C        |
| 7                     | D    | D         | C    | D    | C        |
| 8                     | C    | C         | D    | C    | C        |
| 9                     | C    | C         | C    | C    | A        |
| 10                    | C    | C         | C    | C    | A        |
| Avg. Turn-around Time | 4.5  | 4.5       | 4.75 | 4.5  | 4.25     |



### 2.

#### Design idea

- Add a syscall interface `sys_setgood` in **user/libs/syscall.c**. The interface will use inline asm to call `ecall` to switch to **kernel mode**.
- Also add a syscall `sys_setgood` in **kern/syscall**, this syscall will call **do_setgood** in proc.c. 
- In **do_setgood**, change the `labschedule_good` and `need_resched`.
- Change the `pick_next`  in `default_sched.c`. When the scheduler needs to switch another process, it selects the process with max labschedule_good.



#### Modified code

![image-20230410211700407](P:\CS334-OS\ass04\code1)

![image-20230410211746615](P:\CS334-OS\ass04\code2)

![image-20230410211803792](P:\CS334-OS\ass04\code3)

![image-20230410211831556](P:\CS334-OS\ass04\code4)

![image-20230410211852064](P:\CS334-OS\ass04\code5)

![image-20230410211912649](P:\CS334-OS\ass04\code6)

![image-20230410211957680](P:\CS334-OS\ass04\code7)



#### Running sequence

```
1 2
3 4 5 6 7
6
2 5
2 3
2 7
2 4
2
1
```





#### Running result

![image-20230410212521192](P:\CS334-OS\ass04\code8)