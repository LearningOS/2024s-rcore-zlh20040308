# lab1 report  


## 实现的功能  

1. 在`TaskManagerInner`加入了一个数组`start_time_of_tasks`用于记录每个任务第一次被调用时的时刻  
2. 在`TaskControlBlock`加入了一个数组`syscall_times`用于记录每个任务的各个系统调用的被调用次数  
3. 在`os/src/task/mod.rs`实现了三个方法：
   - `pub fn increase_current_task_syscall_times(syscall_id:usize)`：为当前运行的用户程序所调用的系统调用次数加一  
   - `pub fn get_current_task_syscall_times() -> [u32; MAX_SYSCALL_NUM]`：获取当前运行的用户程序的`syscall_times`数组  
   - `pub fn get_current_task_start_time() -> usize`：获取当前运行的用户程序第一次被调度的时刻  
4. 实现了系统调用`pub fn sys_task_info(_ti: *mut TaskInfo) -> isize`  

## 问答题  

1. 正确进入 U 态后，程序的特征还应有：使用 S 态特权指令，访问 S 态寄存器后会报错。请同学们可以自行测试这些内容描述程序出错行为，同时注意注明你使用的 sbi 及其版本。  

> 答：程序报了PageFault和bad instruction这两个错，具体内容如下：  
> ```
> [kernel] PageFault in application, bad addr = 0x0, bad instruction = 0x804003ac, kernel > killed it.
> [kernel] IllegalInstruction in application, kernel killed it.
> [kernel] IllegalInstruction in application, kernel killed it.
> [kernel] Panicked at src/trap/mod.rs:72 Unsupported trap Exception(LoadFault), stval = > 0x18!  
> ```  
> sbi及其版本号：[rustsbi] Implementation     : RustSBI-QEMU Version 0.2.0-alpha.2







2. 深入理解 `trap.S`中两个函数 ``__alltraps`` 和 ``__restore`` 的作用，并回答如下问题:  

   1. L40：刚进入 ``__restore`` 时，``a0`` 代表了什么值。请指出 ``__restore`` 的两种使用情景。  
    > 答：  
    > 1. 从用内核态转换成用户态的时候，`__restore`可以用来还原用户态上下文  
    > 2. 可以通过复用`__restore`来达到在批处理操作系统初始化完成之后的特权级切换等一系列工作


   2. L43-L48：这几行汇编代码特殊处理了哪些寄存器？这些寄存器的的值对于进入用户态有何意义？请分别解释。  

    ```riscv  
    ld t0, 32*8(sp)
    ld t1, 33*8(sp)
    ld t2, 2*8(sp)
    csrw sstatus, t0
    csrw sepc, t1
    csrw sscratch, t2
    ```  

    > 答：特殊处理了`sstatus`、`sepc`和`sscratch`这三个寄存器：  
    > - `sstatus`：其SPP 等字段给出 Trap 发生之前 CPU 处在哪个特权级（S/U）等信息，在切换成用户态之前要恢复`SPP`字段  
    > - `sepc`：里面存放着从内核态切换成用户态之后的第一条指令的地址  
    > - `sscratch`：里面存放着指向用户栈栈顶的`sp`

   3. L50-L56：为何跳过了 ``x2`` 和 ``x4``？  

    ```riscv  
    ld x1, 1*8(sp)
    ld x3, 3*8(sp)
    .set n, 5
    .rept 27
       LOAD_GP %n
       .set n, n+1
    .endr
    ```  

    > 答：跳过了`sp(x2)`和`tp(x4)`的原因：    
    > - `sp(x2)`：用户栈的栈顶指针已经被放在`sscratch`里面并在之后会被保存，此时的`sp`指向内核栈，所以不需要被保存  
    > - `tp(x4)`：应用程序不会使用这个寄存器，所以不需要保存  

   4. L60：该指令之后，``sp`` 和 ``sscratch`` 中的值分别有什么意义？  

    ```riscv  
    csrrw sp, sscratch, sp
    ```  

    > 答：
    > - `sp`：指向内核栈栈顶  
    > - `sscratch`：指向用户栈栈顶  

   5. ``__restore``：中发生状态切换在哪一条指令？为何该指令执行之后会进入用户态？  

    > 答：`sret`指令。因为这条指令的作用是使得`pc`指向`sepc`，也就是回到 U 特权级继续运行应用程序控制流。

   6. L13：该指令之后，``sp`` 和 ``sscratch`` 中的值分别有什么意义？  


    ```riscv  
         csrrw sp, sscratch, sp
    ```  

    > 答：
    > - `sp`：指向用户栈栈顶  
    > - `sscratch`：指向内核栈栈顶  

   7. 从 U 态进入 S 态是哪一条指令发生的？  

    > 答：`ecall`指令  

## 荣誉准则  

1. 在完成本次实验的过程（含此前学习的过程）中，我曾分别与 **以下各位** 就（与本次实验相关的）以下方面做过交流，还在代码中对应的位置以注释形式记录了具体的交流对象及内容：

    *无*

2. 此外，我也参考了 **以下资料** ，还在代码中对应的位置以注释形式记录了具体的参考来源及内容：

    *无*

3. 我独立完成了本次实验除以上方面之外的所有工作，包括代码与文档。
我清楚地知道，从以上方面获得的信息在一定程度上降低了实验难度，可能会影响起评分。

4. 我从未使用过他人的代码，不管是原封不动地复制，还是经过了某些等价转换。
我未曾也不会向他人（含此后各届同学）复制或公开我的实验代码，我有义务妥善保管好它们。
我提交至本实验的评测系统的代码，均无意于破坏或妨碍任何计算机系统的正常运转。
我清楚地知道，以上情况均为本课程纪律所禁止，若违反，对应的实验成绩将按“-100”分计。  

## 对本次实验设计及难度/工作量的看法  

我觉得从ch2到ch3增加了太多的代码，有些在当前分支用不到的代码就可以先不要加进来，有时看了半天原来不是本章要理解的代码，就很浪费时间  