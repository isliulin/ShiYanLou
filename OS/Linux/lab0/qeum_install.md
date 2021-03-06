# Qemu 架构 硬件模拟器
    Qemu 是纯软件实现的虚拟化模拟器，
    几乎可以模拟任何硬件设备，
    我们最熟悉的就是能够模拟一台能够独立运行操作系统的虚拟机，
    虚拟机认为自己和硬件打交道，
    但其实是和 Qemu 模拟出来的硬件打交道，
    Qemu 将这些指令转译给真正的硬件。
![](https://images2017.cnblogs.com/blog/431521/201711/431521-20171118213522843-322721697.png)
    
    从本质上看，虚拟出的每个虚拟机对应 host 上的一个 Qemu 进程，
    而虚拟机的执行线程（如 CPU 线程、I/O 线程等）对应 Qemu 进程的一个线程。

# 1. 源码下载

    centos：
    sudo apt-get install qemu
    ubuntu：
    sudo yum install qemu -y
    安装包：
    
    $wget http://wiki.qemu-project.org/download/qemu-2.0.0.tar.bz2
    $tar xjvf qemu-2.0.0.tar.bz2
    
    wget https://download.qemu.org/qemu-2.12.0.tar.x
    xz -d qemu-2.12.0.tar.xz
    tar xvf qemu-2.12.0.tar qemu-2.12.0/
    
    Git渠道：
    $git clone git://git.qemu-project.org/qemu.git
# 2. 配置编译及安装
        $cd qemu-2.12.0 //如果使用的是git下载的源码，执行cd qemu
    配置:
        $./configure --enable-kvm --enable-debug --enable-vnc --enable-werror  --target-list="x86_64-softmmu"  x86 64位格式
        $./configure --enable-kvm --enable-debug --enable-vnc --enable-werror  --target-list="i386-softmmu"　　32位格式
    
        configure 脚本用于生成 Makefile，其选项可以用 ./configure --help 查看。
        
        这里使用到的选项含义如下：
            --enable-kvm：编译 KVM 模块，使 Qemu 可以利用 KVM 来访问硬件提供的虚拟化服务。
            --enable-vnc：启用 VNC。
            --enalbe-werror：编译时，将所有的警告当作错误处理。
            --target-list：选择目标机器的架构。默认是将所有的架构都编译，但为了更快的完成编译，指定需要的架构即可。

    安装好之后，会生成如下应用程序：
        1. ivshmem-client/server：这是一个 guest 和 host 共享内存的应用程序，遵循 C/S 的架构。
        2. qemu-ga：这是一个不利用网络实现 guest 和 host 之间交互的应用程序（使用 virtio-serial），运行在 guest 中。
        3. qemu-io：这是一个执行 Qemu I/O 操作的命令行工具。
        4. qemu-system-x86_64：Qemu 的核心应用程序，虚拟机就由它创建的。
        5. qemu-img：创建虚拟机镜像文件的工具，下面有例子说明。
        6. qemu-nbd：磁盘挂载工具。

    编译:
        $make -j4　
    安装:
        $sudo make install
    符号链接:
        cd /usr/local/bin
        sudo ln -s qemu-system-i386 qemu
        
# 3. 创建虚拟机
## a. 使用qemu-img创建虚拟机镜像
    虚拟机镜像用来模拟虚拟机的硬盘，在启动虚拟机之前需要创建镜像文件。
    qemu-img create -f qcow2 test-vm-1.qcow2 10G
    
    -f 选项用于指定镜像的格式，
    qcow2 格式是 Qemu 最常用的镜像格式，
    采用来写时复制技术来优化性能。
    test-vm-1.qcow2 是镜像文件的名字，
    10G是镜像文件大小。
    镜像文件创建完成后，可使用 qemu-system-x86 来启动x86 架构的虚拟机.
## b. 使用 qemu-system-x86 来启动 x86 架构的虚拟机
    qemu-system-x86_64 test-vm-1.qcow2
    因为 test-vm-1.qcow2 中并未给虚拟机安装操作系统，
    所以会提示 “No bootable device”，无可启动设备。

## c. 启动 VM 安装操作系统镜像
    qemu-system-x86_64 -m 2048 -enable-kvm test-vm-1.qcow2 -cdrom ./Centos-Desktop-x86_64-20-1.iso
    -m 指定虚拟机内存大小，默认单位是 MB， 
    -enable-kvm 使用 KVM 进行加速，
    -cdrom 添加 fedora 的安装镜像。
    可在弹出的窗口中操作虚拟机，
    安装操作系统，安装完成后重起虚拟机便会从硬盘 ( test-vm-1.qcow2 ) 启动。

    之后再启动虚拟机只需要执行：

        qemu-system-x86_64 -m 2048 -enable-kvm test-vm-1.qcow2

    qemu-img 支持非常多种的文件格式，
    可以通过 qemu-img -h 查看.
    其中 raw 和 qcow2 是比较常用的两种，
    raw 是 qemu-img 命令默认的，
    qcow2 是 qemu 目前推荐的镜像格式，是功能最多的格式。


## 运行参数
    sudo ln -s qemu-system-i386 qemu
    如果 qemu 使用的是默认 /usr/local/bin 安装路径，
    则在命令行中可以直接使用 qemu 命令运行程序。
    
    qemu 运行可以有多参数，格式如：
    
    qemu [options] [disk_image]
    其中 disk_image 即硬盘镜像文件。
    
    部分参数说明：

    1. `-hda file' / `-hdb file' /`-hdc file' /`-hdd file'
            : 使用 file  作为 硬盘 0、1、2、3镜像。
    2. `-fda file' / `-fdb file'
            : 使用 file  作为软盘镜像，可以使用 /dev/fd0 作为 file 来使用主机软盘。
    3. `-cdrom file'
            : 使用 file  作为光盘镜像，可以使用 /dev/cdrom 作为 file 来使用主机 cd-rom。
    4. `-boot [a|c|d]'
            : 从软盘(a)、光盘(c)、硬盘启动(d)，默认硬盘启动。
    5. `-snapshot'
            : 写入临时文件而不写回磁盘镜像，可以使用 C-a s 来强制写回。
    6. `-m megs'
            : 设置虚拟内存为 msg M字节，默认为 128M 字节。
    7. `-smp n'
            : 设置为有 n 个 CPU 的 SMP 系统。以 PC 为目标机，最多支持 255 个 CPU。
    8. `-nographic'
            : 禁止使用图形输出。
    9. 其他：
          A. 可用的主机设备 dev 例如：
            a. vc              虚拟终端。
            b. null            空设备
            c. /dev/XXX        使用主机的 tty设备
            d. file: filename  将输出写入到文件 filename 中。
            e. stdio           标准输入/输出。
            f. pipe：pipename  命令管道 pipename。
            等。
          B. 使用 dev 设备的命令如：
            a. `-serial dev'      重定向虚拟串口到主机设备 dev 中。
            b. `-parallel dev'    重定向虚拟并口到主机设备 dev 中。
            c. `-monitor dev'     重定向 monitor 到主机设备 dev 中。
          C. 其他参数：
            a. `-s'               等待 gdb 连接到端口 1234。
            b. `-p port'          改变 gdb 连接端口到 port。
            c. `-S'               在启动时不启动 CPU， 需要在 monitor 中输入 'c'，才能让qemu继续模拟工作。
            d. `-d'               输出日志到 qemu.log 文件。

# 在实验中，例如 lab1，可能用到的命令如：
    qemu -hda ucore.img -parallel stdio         # 让ucore在qemu模拟的x86-64/32硬件环境中执行
    或
    qemu -S -s -hda ucore.img -monitor stdio    # 用于与gdb配合进行源码调试


# 常用调试命令
    qemu中monitor的常用命令： 

        1. help         查看 qemu 帮助，显示所有支持的命令。
        2. q|quit|exit  退出 qemu。
        3. stop         停止 qemu。
        4. c|cont|continue  继续执行。
        5. x /fmt addr
        xp /fmt addr 
                    显示内存内容，其中 'x' 为虚地址，'xp' 为实地址。
                    参数 /fmt i 表示反汇编，缺省参数为前一次参数。   
        6. p|print' 　　计算表达式值并显示，例如 $reg 表示寄存器结果。
        7. memsave addr size file
        pmemsave addr size file 
                         将内存保存到文件，memsave 为虚地址，pmemsave 为实地址。
        8. breakpoint 相关：
                         设置、查看以及删除 breakpoint，pc执行到 breakpoint，qemu 停止。（暂时没有此功能）
        9. watchpoint 相关：  设置、查看以及删除 watchpoint, 当 watchpoint 地址内容被修改，停止。（暂时没有此功能）
        a. s|step            单步一条指令，能够跳过断点执行。
        b. r|registers       显示全部寄存器内容。
        c. info 相关操作      查询 qemu 支持的关于系统状态信息的操作。
