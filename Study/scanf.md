你首先要明白，从键盘读入键盘缓冲区(buffer)的数据都是以ASCII码存储的（包括回车）。
程序1
```
    #include "stdio.h"
    void main()
    {
    	char a;
    	char b;
    	scanf("%d",&a);
    	scanf("%d",&b);
    	printf("%d %d",a,b);
    }
```
键盘输入

97<回车> /* 9(ascii:39H), 7(ascii:37H) */

第一次回车后，buffer中的ASCII:39h,37h,0AH（0A是换行的ASCII）， scanf会根据格式字符串中的第一个%d对buffer按字节顺序读取，当读取到0A时，认为％d型的数据结束，此时把已经读取到的39h,37h依据％d转为整型数据97存储在字符型变量a中。（这里是除去了扫描截止点0AH）
此时buffer中已经无任何数据了。

96<回车>
第二次回车后，按同样的流程，scanf会根据格式字符串中的第二个%d对buffer按字节顺序读取。最终b得到96.
此时buffer中已经无任何数据了。
输出
97 96

程序2
#include "stdio.h"

void main()
{
 char a;
 char b;
 scanf("%c",&a);
 scanf("%c",&b);
 printf("%d %d",a,b);
}

键盘输入
9<回车>buffer:39H，0AH
因为scanf会按照第一个％c格式扫描buffer（只扫描一个字节就结束），然后把扫描到的39H直接送到变量a（当以%d格式读出来时，39H就是57）
此时，buffer中只有：0AH。
然后，scanft又遇到第二个%c，继续扫描buffer，得到0aH并送入变量b.
此时buffer中已经无任何数据了

输出
57 10



程序3
#include "stdio.h"

void main()
{
 char a[100];
 char b[100];
 scanf("%s",a);
 scanf("%s",b);
 printf("%s %s",a,b);
}

键盘输入
abc<回车>
第一次回车后，buffer:61H，62H，63H，0AH。
scanf会按照％s的格式对buffer按字节顺序扫描，当扫描到0AH时，结束扫描（按照%s的要求，空格20H也是扫描结束点）。
然后把扫描到的（除去最后一个判断扫描截至的字节0AH）数据直接送入以a为起始地址的字符串。
此时，buffer无任何数据了。

def<回车>
第二次回车后，buffer:65H,66H,67H,0AH.扫描的流程与上面的完全一致。
输出
abc def



程序4
#include <stdio.h>
void main()
{
 int i;
 char j;
 for(i=0;i<2;i++)
  scanf("%c",&j);/*注意这里%前没有空格*/
 printf("%d",j);
}

键盘输入
1<回车>，
这里scanf执行了两次（i==0时，与i==1时），而且每次都是想对j赋值。
第一次scanf,按％c的要求，只扫描buffer中的一个字节,但是buffer中并不数据，于是要求键盘输入数据到buffer，此时的1<回车>代表向buffer中输入了：31H,0AH。
然后按％c的要求，只扫描buffer中的一个字节:31h,并将它直接送入变量j.
此时，buffer中还留下：0AH。

第二次scanf要求键盘输入数据，按％c的要求，只扫描buffer中的一个字节:0Ah,并将它直接送入变量j.
此时，buffer无数据了。

最后，你用%d格式输出j的值（0AH换成整型就是10）

输出
10


程序5
#include <stdio.h>
void main()
{
 int i;
 char j;
 for(i=0;i<2;i++)
  scanf(" %c",&j);/*注意这里%前有一个空格*/
 printf("%d",j);
}
1<回车>2<enter>的情况：
scanf会按照格式控制字符串的要求，顺序扫描buffer.
但是你其中有一个空格，这个很特殊，我也是第一次发现这个问题（一般我都不会在scanf中加入任何常量字符）

我测试了一下：我发现这个空格有吸收回车（0AH）和空格（20H）的“神奇功效”，吸收之后再要求buffer给一个字节，直到这个字节不是0AH或者 20H，此时把这个字节交给下一个格式字串。

第一次循环时遇到格式字串空格，就扫描buffer中的一个字节，但是buffer中无数据，要求从键盘输入数据：1〈回车〉，buffer中有数据了——31H，0AH。再读取到字节31H，scanf发现这个并不是0AH/20H，就把这个字节31H交给格式字符%c处理。
循环结束，此时buffer里面还有：0AH.

第二次循环时遇到格式字串空格，就扫描buffer中的一个字节——0AH，发现是0AH/20H，于是就要求buffer再来一个字节。此时buffer里面已经没有数据了，要求键盘输入：2<enter>.
buffer中有数据了——32H，0AH。于是再读一个字节31H，scanf发现这个并不是0AH/20H，就把这个字节32H交给格式字符%c处理（j最终得到32H）。
循环结束，此时buffer里面还有：0AH.

这里有一篇关于Printf的帖子：http://blog.csdn.net/arong1234/archive/2008/05/18/2456455.aspx


程序6
#include "stdio.h"

void main()
{
 int a;
 int b;
 scanf("%c",&a);
 scanf("%c",&b);
 printf("%d %d",a,b);
}

键盘输入
1<回车>

问题5：

你的编译器VC认为%d数据应该是4个字节，但是你采用的是％c读数据，
 scanf("%c",&a);此句读到的是1的ascii码：31h.然后把31H直接送入地址&a(而并没有改写a的三个高字节地址)。
 scanf("%c",&b);同理。
你可以用printf("a=%x,b=%x\n",a,b);来验证我说的。它们的最低字节肯定是31H，0AH。

PS1：
当你把 int a;int b;放在main()外进行定义时，a,b的初值就是0。此时你会得到正确的结果。
当你把 int a;int b;放在main()内进行定义时，a,b不会被初始化（它们的三个三个高字节地址的内容是不确定的），你就会得到上面错误的结果。（定义的动态变量都不会被初始化，静态变量会被初始化为0）

PS2:以下也是不正确的用法。
char c;
scanf("%d",&c);/当你用％d给c赋值时，会对从＆c开始的连续4个字节进行赋值。当从buffer得到的值是在一个字节范围内（－128～127），下面是可以正常输出的。但是不管怎样，这样做是很危险的——越界。
printf("%d",c);
=================请你测试下这个程序========================
#include "stdio.h"
void main()
{
char c[4],i=4;
scanf("%d",c);/*请输入258<回车>*/

while(i-->0)
printf("%02x ",c[i]);
printf("\n");
}/*如果得到的结果是00 00 00 01 02就说明我的结论是正确的（258的转为16进制数就是00 00 01 02H，然后scanf会把这个数放入以c为起始地址的） 

================以下程序也是======================
#include "stdio.h"
void main()
{
char c,i=4;
char *p=&c;
scanf("%d",&c);/*请输入258<回车>*/

while(i-->0)
printf("%02x ",p[i]);
printf("\n");
}
=============== fflush只清洗标准输入 ==========
吸收任意不等于换行的符号:
```
    scanf("%[^\n]", &t);
```

可以使用scanf清洗输入,当清洗到换行时候，就结束。
```
    *************************
    do {
    	scanf("%c",&t); 
    } while(t != '\n');
```
scanf好像在%c时候,在前面加上空格，可以跳过缓冲前面的空白符.
