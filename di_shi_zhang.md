# 第十章

# 第10章 Linux设备驱动开发


##第10.1章 Linux系统简介
        Linux具有诱人的魅力，他是一个由全世界不同名族、不同信仰、不同性别的人共同参与和协作的国际项目。其中之一就是他的内部实现细节对所有的人来讲是公开的，任何人只要具备必要的技术能力即可方便的验证、理解和修改系统内核。通常，设备驱动程序就是这个进入Linux内核世界的大门。
        设备驱动程序在Linux中扮演着比较重要的作用。用户可以通过一组标准化的程序接口调用这些不同功能的驱动程序。而将这些调用映射到实际设备的操作上的任务则是设备驱动程序来实现。在编写驱动程序时，应该要注意我们编写的硬件的内核代码不要给用户强加任何特定策略，因为不同的用户有不同的需求。驱动程序应该处理如何使硬件可用，而怎样使用的问题应该留给上层应用程序，这样的驱动程序就比较灵活。
##第10.1.1章 Linux系统特点
        Linux是一类Unix操作系统的统称。最初是由Linus Torvalds带头开发，并由全世界众多爱好者共同维护的操作系统。严格意义讲，Linux只是一个操作系统内核，所以通常说的不同版本的Linux是指由GNU软件和Linux内核构成的完整的操作系统。Linus选择用GPL的方式来发行系统内核，而正是由于这一开源和自由的特性，使其迅速拥有了大量的用户，当然还因为他拥有着其它众多优秀特性：
        多用户性
        多任务
        设备独立性
        丰富的网络功能
        可移植性

##第10.1.2章 文件系统介绍
        文件系统是磁盘上存储文件的方法和数据结构，是操作系统组织、存取和保存信息的重要手段，每种操作系统都有自己的文件系统。总的来看，文件系统其实解决了三个问题，从用户来看，系统文件是组织的；文件是怎样在存储设备上存储的；文件是如何去操作的。
        常见的文件系统类型有：Windows中有FAT16、FAT32、NTFS；Linux中有EXT2、EXT3、EXT4等；还有网络文件系统NFS和光盘文件系统ISO9600。Linux几乎支持所有的UNIX类的文件系统，如HFS、XFS、JFS、Minix Fs以及UFS等，同时还支持NFS、HTFS和VFAT，但不支持NTFS的写操作。
        其实大部分文件系统目录结构都是树形结构，我们熟知的Windows是以每一个分区为根目录的树形结构。而Linux文件系统是以一个根目录开始，根目录下可以有任意多个文件和子目录，子目录中又可以有任意多个的文件和子目录。可以看出，这其实是一个递归的定义，这棵目录树理论上可以无限延伸下去。Linux所有文件系统都挂在在一个根目录下，所以必须先划分一个根分区。
##第10.1.3章 Linux 2.6内核的特点
        Linux 2.6内核是目前应用得最广泛的内核，相较于Linux 2.4，Linux 2.6有了相当大的改进，主要体现在如下几个方面。
        新的调度器
        内核抢占
        改进的线程模型
        虚拟内存的变化
        文件系统
        音频
        总线
        电源管理
        联网和IPSec
        用户界面层
                在设备驱动程序的方面，Linux 2.6相对于Linux 2.4也有较大的改动，这主要表现在内核API中增加了不少新功能（例如内存池）、sysfs文件系统、内核模块从.o变.ko、驱动模块编译方式、模块使用技术、模块加载和卸载函数的定义等方面。

##第10.2章 交叉编译工具
        源文件需要经过编译才能生成可执行文件。在Windows下进行开发时，只需要单击几个按钮即可编译，集成开发环境（比如Visual studio）已经将各种编译工具的使用封装好了。Linux下也有很优秀的集成开发工具，但是更多的时候是直接使用编译工具，即使使用集成开发工具，也需要掌握一些编译选项。
PC上的编译工具链为gcc、ld、objcopy、objdump等，它们编译出来的程序在x86平台上运行。要编译出能在ARM平台上运行的程序，必须使用交叉编译工具arm-linux-gcc、arm-linux-ld等，下面分别介绍。
##第10.2章 交叉编译工具
        arm-linux-gcc
                一个C/C++文件要经过预处理（preprocessing）、编译（compilation）、汇编（assembly）和连接（linking）等4步才能变成可执行文件。


后缀名	语言种类	后期操作
.c	C源程序	预处理、编译、汇编
.C	C++源程序	预处理、编译、汇编
.cc	C++源程序	预处理、编译、汇编
.cxx	C++源程序	预处理、编译、汇编
.m	Objective-C源程序	预处理、编译、汇编
.i	预处理后的C文件	编译、汇编
.ii	预处理后的C++文件	编译、汇编
.s	汇编语言源程序	汇编
.S	汇编语言源程序	预处理、汇编
.h	预处理文件	通常不出现在命令行上

##第10.2章 交叉编译工具
下面以一个简单的程序为例来详细讲解编译过程。程序源码如下：
    //Hello.c
    #include<stdio.h>
    #include”fun.h”
    int main(int argc, char *argv[])
    {
      int i;
      printf(“Hello World!\n”);
      fun();
      return 0;
    }

//fun.h
void fun(void);

//fun.c
void fun(void)
{
    printf(“Hello Function!\n”);
}


    编译命令如下：

    $ gcc –c –o Hello.o Hello.c
    $ gcc –c –o fun.o fun.c
    $ gcc –o test Hello.o fun.o

其中，Hello.o、fun.o是经过了预处理、编译、汇编后生成的OBJ文件，它们还没有被连接成可执行文件；最后一步将它们连接成可执行文件test，可以直接运行以下命令：
$ ./test

结果如下：

##第10.2章 交叉编译工具
        arm-linux-objcopy
                arm-linux-objcopy被用来复制一个目标文件的内容到另一个目标文件中，可以使用不同于源文件的格式来自输出目的文件，即可以进行格式转换。
        arm-linux-objdump
        arm-linux-objdump用于显示二进制文件信息，常用来查看反汇编代码。

##第10.3章 Makefile
        make命令执行时，需要一个 makefile 文件，以告诉make命令如何去编译和链接程序。
        首先，我们用一个示例来说明makefile的书写规则。以便给大家一个感性认识。这个示例来源于gnu的make使用手册，在这个示例中，我们的工程有8个c文件，和3个头文件，我们要写一个makefile来告诉make命令如何编译和链接这几个文件。我们的规则是：
（1） 如果这个工程没有编译过，那么我们的所有c文件都要编译并被链接。
（2） 如果这个工程的某几个c文件被修改，那么我们只编译被修改的c文件，并链接目标程序。
（3） 如果这个工程的头文件被改变了，那么我们需要编译引用了这几个头文件的c文件，并链接目标程序。
        只要我们的makefile写得够好，所有的这一切，我们只用一个make命令就可以完成，make命令会自动智能地根据当前的文件修改的情况来确定哪些文件需要重编译，从而自己编译所需要的文件和链接目标程序。

##第10.3.1章 Makefile的规则
target ... : prerequisites ...
	command
	...
	...

        target可以是一个object file(目标文件)，也可以是一个执行文件，还可以是一个标签（label）。
        prerequisites就是，要生成那个target所需要的文件或是目标。
        command也就是make需要执行的命令。（任意的shell命令）
        这是一个文件的依赖关系，也就是说，target这一个或多个的目标文件依赖于prerequisites中的文件，其生成规则定义在 command中。说白一点就是说，prerequisites中如果有一个以上的文件比target文件要新的话，command所定义的命令就会被执行。这就是makefile的规则。也就是makefile中最核心的内容。

##第10.3.2章 Makefile实例
edit : main.o kbd.o command.o display.o \
		insert.o search.o files.o utils.o
	cc -o edit main.o kbd.o command.o display.o \
		insert.o search.o files.o utils.o

main.o : main.c defs.h
	cc -c main.c
kbd.o : kbd.c defs.h command.h
	cc -c kbd.c
command.o : command.c defs.h command.h
	cc -c command.c
display.o : display.c defs.h buffer.h
	cc -c display.c
insert.o : insert.c defs.h buffer.h
	cc -c insert.c
search.o : search.c defs.h buffer.h
	cc -c search.c
files.o : files.c defs.h buffer.h command.h
	cc -c files.c
utils.o : utils.c defs.h
	cc -c utils.c
clean :
	rm edit main.o kbd.o command.o display.o \
		insert.o search.o files.o utils.o

2016-9-19
武汉科技大学
Page 1
幻灯片15
##第10.3.2章 Makefile实例
        反斜杠（\）是换行符的意思。这样比较便于makefile的易读。我们可以把这个内容保存在名字为“makefile”或“Makefile” 的文件中，然后在该目录下直接输入命令“make”就可以生成执行文件edit。如果要删除执行文件和所有的中间目标文件，那么，只要简单地执行一下 “make clean”就可以了。
        在这个makefile中，目标文件（target）包含：执行文件edit和中间目标文件（*.o），依赖文件（prerequisites）就是冒号后面的那些 .c 文件和 .h文件。每一个 .o 文件都有一组依赖文件，而这些 .o 文件又是执行文件 edit 的依赖文件。依赖关系的实质上就是说明了目标文件是由哪些文件生成的，换言之，目标文件是哪些文件更新的。
        在定义好依赖关系后，后续的那一行定义了如何生成目标文件的操作系统命令，一定要以一个tab键作为开头。记住，make并不管命令是怎么工作的，他只管执行所定义的命令。make会比较targets文件和prerequisites文件的修改日期，如果prerequisites文件的日期要比targets文件的日期要新，或者target不存在的话，那么，make就会执行后续定义的命令。

##第10.3.2章 Makefile实例
        这里要说明一点的是，clean不是一个文件，它只不过是一个动作名字，有点像c语言中的lable一样，其冒号后什么也没有，那么，make就不会自动去找它的依赖性，也就不会自动执行其后所定义的命令。要执行其后的命令（不仅用于clean，其他lable同样适用），就要在make命令后明显得指出这个lable的名字。这样的方法非常有用，我们可以在一个makefile中定义不用的编译或是和编译无关的命令，比如程序的打包，程序的备份，等等。

##第10.3.3章 make是如何工作的
        默认的方式下，也就是我们只输入make命令。那么，
（1） make会在当前目录下找名字叫“Makefile”或“makefile”的文件。
（2） 如果找到，它会找文件中的第一个目标文件（target），在上面的例子中，他会找到“edit”这个文件，并把这个文件作为最终的目标文件。
（3） 如果edit文件不存在，或是edit所依赖的后面的 .o 文件的文件修改时间要比edit这个文件新，那么，他就会执行后面所定义的命令来生成edit这个文件。
（4） 如果edit所依赖的.o文件也不存在，那么make会在当前文件中找目标为.o文件的依赖性，如果找到则再根据那一个规则生成.o文件。（这有点像一个堆栈的过程）
（5） 当然，你的C文件和H文件是存在的啦，于是make会生成 .o 文件，然后再用 .o 文件生成make的终极任务，也就是执行文件edit了。

##第10.3.3章 make是如何工作的
       这就是整个make的依赖性，make会一层又一层地去找文件的依赖关系，直到最终编译出第一个目标文件。在找寻的过程中，如果出现错误，比如最后被依赖的文件找不到，那么make就会直接退出，并报错，而对于所定义的命令的错误，或是编译不成功，make根本不理。make只管文件的依赖性，即，如果在我找了依赖关系之后，冒号后面的文件还是不在，那么对不起，我就不工作啦。
       通过上述分析，我们知道，像clean这种，没有被第一个目标文件直接或间接关联，那么它后面所定义的命令将不会被自动执行，不过，我们可以显式要make执行。即命令——“make clean”，以此来清除所有的目标文件，以便重编译。
       于是在我们编程中，如果这个工程已被编译过了，当我们修改了其中一个源文件，比如file.c，那么根据我们的依赖性，我们的目标file.o会被重编译（也就是在这个依性关系后面所定义的命令），于是file.o的文件也是最新的啦，于是file.o的文件修改时间要比edit要新，所以 edit也会被重新链接了（详见edit目标文件后定义的命令）。
        而如果我们改变了“command.h”，那么，kdb.o、command.o和files.o都会被重编译，并且，edit会被重链接。

##第10.3.3章 小结
       本章先介绍了Linux系统的特点，包括文件系统和Linux 2.6内核的特点，然后详细介绍了交叉编译工具的选项和Makefile，通过两个实例分别说明了交叉编译工具和Makefile的用法。


