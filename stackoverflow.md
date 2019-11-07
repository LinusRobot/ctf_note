ROP:
	填充空间方法： 1> ida ; 2> gdb.  canary found是，触发check_failed(),ROP失效。

栈溢出利用：
     函数内局部变量利用，可覆盖这函数返回地址(返回地址后4/8个字节要清零) 或者  可以修改该函数内局部变量值。
     注意：read 读取需要注意读取长度，需覆盖目标字段。fgets gets
     32bit : ebp + 8h 第一个参数； ebp + 0ch 第二个参数 以此类推 ebp的位置是固定的
     可以使用plt的函数地址。
     函数栈结构有时候不满足一般条件，需要通过gdb调试知道返回地址位置，main函数的返回地址看retn指令。
     通过距离算偏移用ebp
     /bin/bash -> +7 就是sh

system() 是一个单参数的函数，汇编过程： push 参数地址(说明调用system之前，参数在栈顶);    call _system;  原因如下：
函数地址调用函数，参数入栈，[返回地址入栈，ip ->函数地址](call function),相当于ic语言调函数入栈的情况,参数入栈,call function入栈. 
call function指令调用函数，将函数的返回地址入栈，ip指向函数开始处(函数地址). 开始进入函数体中，ebp入栈， esp = ebp, esp -**h (在call function之前参数已经入栈).
main函数调 function过程:
     para n , para n-1, ... , para 1 入栈
     返回地址 入栈
     e/rbp 入栈
     开辟栈空间
##构建bin/sh
栈溢出：ret -> gets_addr --.bss--> binsh_addr --ret-> system_addr    


libc函数利用：
	1. 通过栈溢出泄露write,puts的运行地址got。
	2. 利用libc里write,puts中的地址， 函数偏移地址不变，得到system的运行地址。