linux下 嵌入式代码 用户态 + 内核模块 共仓

构建：
阶段一 使用makefile作为构建代码组织构建
阶段二 使用cmake(生成器 makefile)作为构建代码组织构建
阶段三 使用cmake(生成器 ninja + makefile)作为构建代码组织构建，用户态和内核模块代码解耦，分开编译

早期嵌入式构建现场
早期嵌入式代码(C + 部分汇编)主要是使用makefile来组织构建，配合make命令来组织编译构建的。
优点
makefile书写各种依赖关系、生成构建规则、生成目标，自动化构建工具make调用配置文件(makefile)来调度编译构建所有目标。一键式构建全部代码
缺点
makefile各种规则，各种函数和操作比较晦涩，特别是在大型的构建系统上，入门和维护的成本都比较高(当然这个是makefile写得很稀烂导致的)，linux kernel源码仓的makefile就挺难看懂的。

cmake的出现
工具总在不断迭代，后续出现了cmake。
cmake对于并发的处理更好(据说)，将目标、依赖关系等用更易于理解的命令表现出来，cmake命令给人的感觉更像编程语言一样，cmake的脚本语言其实从另一方面来说也是一门编程语言了。
cmake相对makefile的晦涩来说，对于长期写代码的程序员来说更易读了，我觉得这个是cmake的构建脚本的最大有点和进步了。
cmake原生命令是支持用户态代码的构建的，但是内核态代码的编译受限kbuild构建系统，cmake并不支持。
用户态 cmake原生命令  +  内核态  add_custom_target等命令调用make走kbuild构建内核模块。

优点
如果使用makefile作为生成器，cmake生成makefile代码使用make调度，内核态和用户态的整套代码全部使用make来调度编译构建，可以共享make的jobserver令牌池，统一调度系统资源，构建效率更高。
缺点
make对cmake生成的层层嵌套makefile进行目标解析是单线程的，编译的目标过多会导致实际编译前的依赖解析超级耗时；
且cmake的高版本是不推荐继续使用makefile继续作为生成器的(据说)，cmake 3.31.1往后的版本会对各个target添加codegen的预留目标，更加拉长了make的目标解析耗时。

其实从make切cmake的就应该考虑做用户态和内核态的分离的

用户态 内核态代码解耦，目录分离
cmake的高版本更推荐使用ninja构建用户态代码，cmake生成的ninja脚本是个超级大的文件，而不是像生成的makefile那样层层嵌套。
原理上是牺牲空间换取时间，ninja只需要读一份文件，而不是像make那样一层层读makefile，这样子编译前的依赖解析就非常快了。(对于大型构建系统，可能1min -> 10s这样子)

用户态这一块是比较简单的；主要是内核态这块，如果还是使用add_custom_target这一套就比较挫了，还是会存在高版本cmake的一堆重型冗余，影响编译效率。
构建分离
用户态  cmake(ninja)
内核态  cmake(makefile模块使用configure生成内核态的makefile)

可以写一个kbuild_kernel_generator.cmake来处理内核态cmake脚本，仅用于生成内核态的makefile。
抽象出标准一系列makefile模板，kbuild_kernel_generator.cmake封装一系列接口，源文件软连接到build目录  +  configure命令转化makefile模板到build目录中。
内核态代码用cmake脚本来维护，易于维护阅读；使用cmake命令生成内核态需要makefile，最终使用make来构建。

很完美！！！
