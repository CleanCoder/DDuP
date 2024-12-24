# 使用 CentOS 8.2 作为基础镜像
FROM centos:8.2.2004

# 替换为国内的阿里云镜像源
RUN sed -i 's|^mirrorlist=|#mirrorlist=|g' /etc/yum.repos.d/CentOS-*.repo && \
    sed -i 's|^#baseurl=http://mirror.centos.org|baseurl=http://mirrors.aliyun.com|g' /etc/yum.repos.d/CentOS-*.repo && \
    yum clean all && \
    yum makecache


# 设置默认工作目录
WORKDIR /root

# 设置默认启动命令
CMD ["/bin/bash"]
