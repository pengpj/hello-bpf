FROM ubuntu:20.04

# 错误的用法是在环境变量中全局生效，会导致 BASH 等命令操作异常 ?? 
ENV DEBIAN_FRONTEND=nointeractive

# 安装依赖项
# DEBIAN_FRONTEND=noninteractive, 非交互式地进行操作 ? 好像没有用，不加上面的环境变量，还是会交互输入地域信息
RUN DEBIAN_FRONTEND=noninteractive \
    && sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list \
    && echo "Asia/Shanghai" > /etc/timezone \
    && apt update \
    && apt install -y bison flex build-essential git cmake make libelf-dev clang llvm strace tar libfl-dev libssl-dev libedit-dev zlib1g-dev python  python3-distutils glibc-source libncurses5-dev binutils-dev bc telnet net-tools lsof tcpdump iotop sysstat vim wget

# 把包提前下载，如果网络好也可以在构建时下载
# 如果有下载好的源码，放到当前路径
# COPY linux-4.19.113.tar.gz /root/
RUN mkdir -p /root \
    && wget http://ftp.sjtu.edu.cn/sites/ftp.kernel.org/pub/linux/kernel/v4.x/linux-4.19.113.tar.gz -O /root/linux-4.19.113.tar.gz

# 学习包
# COPY linux-observability-with-bpf.tar.gz /root/

RUN cp /root/linux-4.19.113.tar.gz /usr/src/ \
    && cd /usr/src/ \
    && tar -xvf linux-4.19.113.tar.gz \
    && cd linux-4.19.113

RUN cd /usr/src/linux-4.19.113 \
# 配置源码
    && make defconfig \
# 修复modpost的报错，修复模块    
    && make modules_prepare \
# 关联内核头文件    
    && make headers_install \
# 进入到 kernal source 目录下，并编译libbpf
# Ubuntu没有生成文件所期望的库路径，所以我们现在需要把我们的库移到它的库路径    
    && cd /usr/src/linux-4.19.113/tools/lib/bpf \
    && make && make install prefix=/usr/local \
    && mv /usr/local/lib64/libbpf.* /lib/x86_64-linux-gnu/

# 由于是测试相关的代码，所以可以skip掉。修改完的文件在这里
# 参考作者的实现，但还没有搞懂变更的内容
# 作者文章：https://cloud.tencent.com/developer/article/1644458
# 变更后文件：https://github.com/nevermosby/linux-bpf-learning/edit/master/bpf/perf-sys.h
COPY perf-sys.h /usr/src/linux-4.19.113/tools/perf/perf-sys.h

# 安装相关的工具类
RUN apt install -y iputils-ping

# 编译示例代码
RUN cd /usr/src/linux-4.19.113 \
    && make M=samples/bpf