# Race Condition
	条件竞争是指一个系统的运行结果依赖于不受控制的事件的先后顺序。
	由于目前的系统中大量采用并发编程，经常对资源进行共享，往往会产生条件竞争漏洞。
	当一个软件的运行结果依赖于进程或者线程的顺序时，就可能会出现条件竞争。简单考虑一下，可以知道条件竞争需要如下的条件：
	1. 并发，即至少存在两个并发执行流。这里的执行流包括线程，进程，任务等级别的执行流。
	2. 共享对象，即多个并发流会访问同一对象。常见的共享对象有共享内存，文件系统，信号。一般来说，这些共享对象是用来使得多个程序执行流相互交流。此外，我们称访问共享对象的代码为临界区。在正常写代码时，这部分应该加锁。
	3. 改变对象，即至少有一个控制流会改变竞争对象的状态。因为如果程序只是对对象进行读操作，那么并不会产生条件竞争。
## 形式
### CWE-367: TOCTOU Race Condition
	TOCTOU (Time-of-check Time-of-use) 指的是程序在使用资源（变量，内存，文件）前会对进行检查，但是在程序使用对应的资源前，该资源却被修改了。
### CWE-365: Race Condition in Switch
	当程序正在执行 switch 语句时，如果 switch 变量的值被改变，那么就可能造成不可预知的行为。尤其在 case 语句后不写 break 语句的代码，一旦 switch 变量发生改变，很有可能会改变程序原有的逻辑。
### CWE-363: Race Condition Enabling Link Following
	Linux 中提供了两种对于文件的命名方式:
	* 文件路径名
	* 文件描述符
	正是由于间接性，产生了上面我们所说的时间窗口。

	介绍一个例子，程序在访问某个文件之前，会检查是否存在，之后会打开文件然后执行操作。但是如果在检查之后，真正使用文件之前，攻击者将文件修改为某个符号链接，那么程序将访问错误的文件。 这种条件竞争出现的问题的根源在于文件系统中的名字对象绑定的问题。而下面的函数都会使用文件名作为参数：access(), open(), creat(), mkdir(), unlink(), rmdir(), chown(), symlink(), link(), rename(), chroot(),…
	我们可以使用 fstat 函数来读取文件的信息并把它存入到 stat 结构体中，然后我们可以将该信息与我们已知的信息进行比较来判断我们是否读入了正确的信息。其中，stat 结构体中的 st_ino 和 st_dev 变量可以唯一表示文件。
### CWE-364: Signal Handler Race Condition
	条件竞争经常会发生在信号处理程序中，这是因为信号处理程序支持异步操作。尤其是当信号处理程序是不可重入的或者状态敏感的时候，攻击者可能通过利用信号处理程序中的条件竞争，可能可以达到拒绝服务攻击和代码执行的效果。比如说，如果在信号处理程序中执行了 free 操作，此时又来了一个信号，然后信号处理程序就会再次执行 free 操作，这时候就会出现 double free 的情况，再稍微操作一下，就可能可以达到任意地址写的效果了。
	一般来说，与信号处理程序有关的常见的条件竞争情况有：
	1. 信号处理程序和普通的代码段共享全局变量和数据段。
	2. 在不同的信号处理程序中共享状态。
	3. 信号处理程序本身使用不可重入的函数，比如 malloc 和 free 。
	4. 一个信号处理函数处理多个信号，这可能会进而导致 use after free 和 double free 漏洞。
	5. 使用 setjmp 或者 longjmp 等机制来使得信号处理程序不能够返回原来的程序执行流。
#### 线程安全与可重入
	线程安全：即该函数可以被多个线程调用，而不会出现任何问题。

	可重用： 
	1. 一个函数可以被多个实例可以同时运行在相同的地址空间。
	2. 可重入函数可以被中断，并且其它代码在进入该函数时，不会丢失数据的完整性。所以可重入函数一定是线程安全的。
	3. 可重入强调的是单个线程执行时，重新进入同一个子程序仍然是安全的。
	4. 可重入强调的是单个线程执行时，重新进入同一个子程序仍然是安全的。
	5. 不满足条件：
		1. 函数体内使用了静态数据结构，并且不是常量
		2. 函数体内使用了 malloc 或者 free 函数
		3. 函数使用了标准 IO 函数。
		4. 调用的函数不是可重入的。
## mitigation
	如果想要消除条件竞争，那么首要的目标是找到竞争窗口（race windows）。
	所谓竞争窗口，就是访问竞争对象的代码段，这给攻击者相应的机会来修改相应的竞争对象。
	一般来说，如果我们可以使得冲突的竞争窗口相互排斥，那么就可以消除竞争条件。
	