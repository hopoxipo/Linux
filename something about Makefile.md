# Makefile进阶
## 关于通配符
* **~**
类似于HOME，如`~/test`表示当前用户的`$HOME`目录下的`test`目录；同理，`~currentusr/test`代表`currentusr`下的`test`目录;
* **\***
 表示一类文件，如`*.c`表示所有以c文件后缀的文件;
 **注意**：通配符可以用在变量中，但不会展开：`objs = *.o`，该语句表示表示`objs`值就是`*.o`，它不会在变量中展开；如果要让其表示[.o]文件名集合，可以：`objs := $(wildcard *.o)`;

## VPATH vpath
* **VPATH**变量
定义这个变量后，make会在当前目录找不到依赖文件和目标文件的情况下到指定的目录中搜索：  
`VPATH = src : ../headers`  
这里，定义了两个目录（`src` `../headers`），make会按照这个顺序搜索，其中，目录由 **:** 隔开，当前目录为最先搜索目录；  
* **vpath**关键字
可指定不同文件在不同搜索目录：
 * `vpath <pattern> <directories>` **为符合模式 \<pattern> 的文件指定搜索目录 \<directories>**；
 * `vpath <pattern>`  **清除符合模式\<pattern>的文件的搜索目录**；  
 * `vpath`  **清除所有已被设置好的文件搜索目录**；

 其中，\<pattern>需要包含"%"符（意为匹配零或若干字符）：`%.h`表示所有以`.h`结尾的文件
 ```
 vpath %.c test
 vpath % beta
 vpath %.c exam
```
以上表示`.c`文件先在**test**目录，然后在`beta`，最后在`exam`目录搜索；
```
vpath %.c test: exam
vpath % beta
```
以上表示，`.c`文件以`test`,`exam`,`beta`的顺序目录查找；

## 伪目标
 **.PHONY : clean** 伪目标总是被执行

## 静态模式    
 主要用于多目标编译：   
 ```
 <targets...> : <targes-pattern> : <prereq-pattens...>
     <commands>
 ```
其中，
* *targets*为目标文件，可包含通配符，可理解为目标文件的一个集合；
* *target-pattern*为*target*的模式；
* *prereq-patterns*是目标的依赖模式，相当于对*target-pattern*的递归依赖；
* *command*好说，就是编译命令。
 for example：    
 ```
 objs = a.o b.o c.o
 all : $(objs)

 $(objs) : %.o : %.c      
     gcc -c $< -o $@    

 .PHONY : celan
 clean :
     rm -rf *.o

 ```
 这表示 **$(objs)** 里的目标以 **.o** 的模式获取，依赖于相应的 **.c** 文件，执行后面的 **commands**，等价于如下语句：    
 ```
 a.o : a.c
     gcc -c $< -o $@    
 b.o : b.c    
     gcc -c $< -o $@
 c.o : c.c
     gcc -c $< -o $@
 ```

## 命令相关
* **终端显示命令**
 当需要在终端显示语句时，可以用*echo*来实现：
 ```
 echo excting...
 ```
 则会在make时在终端显示该条命令及内容：
 ```
 ehco excting...
 excting...
 ```
 如果不希望显示命令而只显示内容，则可在*echo* 前加 *@*：
 ```
 @echo excting...
 ```
 则其执行结果为：
 ```
 excting...
 ```
 如果make执行中想只显示命令而不执行命令，则 *make* 执行时可在其后加上 *-n* 或 *--just-print* ，一般用于调试观察执行顺序。

* **执行命令**    
 当makefile中上一条语句的结果需要作用于下一条命令时，需要将两条命令卸载一行上，并用 **;** 隔开（当然，三条四条更多条有相关性的命令也是这样，你也可以用 *\\* 来实现伪分行的效果）：
 ```
 exec :
    cd /usr/beta
    pwd
```
上述例子 *make exec* 时，相当于上一句进入文件夹操作无用，*pwd* 照样会打印 *makefile* 的当前目录；而：
```
exec:
    cd /usr/beta; pwd
```
则会先 *cd* 进 */usr/beta* 文件夹，然后 *pwd* 打印该目录，也即输出 **/usr/beta** 。

* **强制取消出错**    
*make* 执行时，只要检测到一个错误就会立即停止编译，终止所有规则语句的执行。这时就可以在该出错命令行前加 **-** 来标记该命令，让 *make* 认为它是对的。比如如果 *makefile* 中，有创建目录操作，如果该目录先前不存在，则能顺利执行，如果已经存在，就会报错并终止之后命令执行。而这不是我们希望看到的，那么我们可以在这条命令行前加 **-**， 也即 `-mkdir hello` 。
