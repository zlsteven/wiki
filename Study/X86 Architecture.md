CPU的IP(EIP)中存放的是虚地址,把一个虚拟地址转换位物理地址,模式不同,转换方式不同.
实模式：
	虚地址到实地址转换，段寄存器左移4位与偏移相加，得到物理地址，寻址
	空间1M.
保护模式：
	虚地址到实地址转换经过MMU，也就是分段和分页机制，寻址空间4G.
8086:
<Linux 操作系统原理与应用>
	8086地址总线是20位的，但是CPU中ALU的宽度，即数据总线只有16位，也就是可直
	接加以运算的指针长度是16位的.为了实现16位的地址映射到20位的地址空间:分段
	为了支持分段，8086设置了四个段寄存器:CS DS SS和ES，分别用于代码段,数据段
	堆栈段及其他段。每个段寄存器都是16位，对应于地址总线中的高16位。每条访存
	指令中内部地址也都是16位的，但是在送上地址总线之前，CPU内部自动把它与某
	个段寄存器中的内存相加。因为段寄存器中内容对应于20位地址总线中的高16位，
	也就是把段寄存器左移4位，所以相加时就是内存总线中的高12位与段寄存的的16
	位相加，而低4位保留不变。
80386:
ALU的数据总线是32位的，地址总线与数据总线宽度一致，也是32位，寻址能力4GB.现在对于
内存寻址来说是足够了，而且宽度一致cpu结构应该简洁明了。作为X86系列，80386必须维持
那些寄存器的存在，还必须支持实模式，同时又要能支持保护模式。

为了保持兼容，出现了实模式，利用实模式，有出现了保护模式(段基址+加偏移地址)，保护
模式又能实现权限分级保护功能。

实模式:
	又叫实地址模式,CPU完全按照8086的实际寻址方法访问从00000h--FFFFFh(1MB大小)
	的地址范围的内存,在这种模式下,CPU只能做单任务运行.
	寻址公式:物理地址 = 左移4位的段地址 + 偏移地址。
保护模式:
	又叫内存保护模式，寻址采用32位段和偏移量，最大寻址空间是4GB，在这种模式下
	系统运行于多任务。好处：增加了寻址空间，增加了对多任务的支持，增加了段页
	式寻址机制的内存管理(分段机制使得段具有访问权限和特权级，各应用程序和操作
	系统的代码和核心是被保护的，这也是多任务支持实现的关键和保护这个名字的由
	来).
	寻址过程：物理地址 = 由段地址查询全局描述符号表中给出的段基址 + 偏移地址
	即：物理地址由 影像寄存器中的基址加上16位或者32位的偏移组成。
1.实模式
	是CPU启动的时候的模式
	这时候就相当于一个速度超快的8086
	不能使用多线程
	不能实现权限分级
	还不能访问20位以上地址线，也就是说只能访问1M内存
2.保护模式
	操作系统接管CPU后
	会使CPU进入保护模式
	这时候可以发挥80X86的所有威力
	包括权限分级，内存分页等等 

实模式是将整个物理内存看成分段的区域，程序代码和数据位于不同区域，系统程序和用户
程序没有区别对待，而且每一个指针都是指向实在的物理地址。这样以来，用户程序的一个
指针如果指向了系统程序区域或其他程序区域，并改变了值，那么对于这个被修改的系统程
序或用户程序，其后果就很可能是灾难性的。

CPU启动环境为16位实模式，之后可以切换到保护模式。但从保护模式无法切换会实模式。
