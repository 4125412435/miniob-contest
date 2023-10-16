# how to use
# docker build -t miniob .
# make sure docker has been installed
# FROM rockylinux:8
FROM openanolis/anolisos:8.6

ARG HOME_DIR=/root
ARG GIT_SOURCE=github

ENV LANG=en_US.UTF-8

# install rpm
# note: gcc-c++ in rockylinux 8 and gcc-g++ in rockylinux 9. use `dnf groupinfo "Development Tools"` to list the tools
RUN dnf install -y make cmake git wget which flex gdb gcc gcc-c++ diffutils readline-devel texinfo
# rockylinux:9 RUN dnf --enablerepo=crb install -y texinfo
# rockylinux:8
# RUN dnf --enablerepo=powertools install -y texinfo

# prepare env
WORKDIR ${HOME_DIR}
RUN echo alias ls="ls --color=auto" >> .bashrc
RUN echo "export LD_LIBRARY_PATH=/usr/local/lib64:\$LD_LIBRARY_PATH" >> .bashrc

# clone deps and compile deps
RUN mkdir -p ${HOME_DIR}/deps
WORKDIR ${HOME_DIR}/deps

RUN git clone https://gitee.com/mirrors/libevent.git  -b release-2.1.12-stable  \
    && mkdir -p ${HOME_DIR}/deps/libevent/build  \
    && cmake -DEVENT__DISABLE_OPENSSL=ON -B ${HOME_DIR}/deps/libevent/build ${HOME_DIR}/deps/libevent \
    && make -C ${HOME_DIR}/deps/libevent/build -j install \
    && rm -rf ${HOME_DIR}/deps/*

RUN git clone https://gitee.com/mirrors/jsoncpp.git \
    && mkdir -p ${HOME_DIR}/deps/jsoncpp/build \
    && cmake -DJSONCPP_WITH_TESTS=OFF -DJSONCPP_WITH_POST_BUILD_UNITTEST=OFF -B ${HOME_DIR}/deps/jsoncpp/build ${HOME_DIR}/deps/jsoncpp/ \
    && make -C ${HOME_DIR}/deps/jsoncpp/build -j install \
    && rm -rf ${HOME_DIR}/deps/*

RUN git clone  https://gitee.com/mirrors/googletest.git \
    && mkdir -p ${HOME_DIR}/deps/googletest/build  \
    && cmake -B ${HOME_DIR}/deps/googletest/build ${HOME_DIR}/deps/googletest \
    && make -C ${HOME_DIR}/deps/googletest/build -j install \
    && rm -rf ${HOME_DIR}/deps/*

RUN wget http://ftp.gnu.org/gnu/bison/bison-3.7.tar.gz \
    && tar xzvf bison-3.7.tar.gz \
    && cd bison-3.7 \
    && ./configure --prefix=/usr/local \
    && make -j install \
    && rm -rf ${HOME_DIR}/deps/*

# install openssh
RUN yum -y install openssh openssh-clients openssh-server

RUN echo "root:root" | chpasswd #修改root密码
RUN mkdir /var/run/sshd
RUN ssh-keygen -A
RUN yum install -y zsh \
    && git clone https://gitee.com/mirrors/ohmyzsh.git ~/.oh-my-zsh \
    && cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc \
    && sed -i "s/robbyrussell/bira/" ~/.zshrc \
    && usermod --shell /bin/zsh root \
    && echo "export LD_LIBRARY_PATH=/usr/local/lib64:\$LD_LIBRARY_PATH" >> ~/.zshrc

# clone miniob code
RUN mkdir -p ${HOME_DIR}/source
# WORKDIR ${HOME_DIR}/source
# RUN git clone https://${GIT_SOURCE}.com/oceanbase/miniob
# RUN echo "mkdir -p build && cd build && cmake .. -DDEBUG=ON -DCMAKE_C_COMPILER=`which gcc` -DCMAKE_CXX_COMPILER=`which g++` && make -j4" > ${HOME_DIR}/source/miniob/build.sh && chmod +x ${HOME_DIR}/source/miniob/build.sh
# RUN mkdir -p ${HOME_DIR}/source/miniob/build
# WORKDIR ${HOME_DIR}/source/miniob/build
# RUN cmake -B ${HOME_DIR}/source/miniob/build -DDEBUG=ON -DCMAKE_C_COMPILER=`which gcc` -DCMAKE_CXX_COMPILER=`which g++` ${HOME_DIR}/source/miniob/ \
#     && make -j4 \
#     && rm -rf ${HOME_DIR}/source/miniob/build

WORKDIR ${HOME_DIR}

RUN echo -e "#!/bin/bash\n/usr/sbin/sshd -D\ntail -f /dev/null" > ${HOME_DIR}/docker-starter.sh && chmod +x ${HOME_DIR}/docker-starter.sh

ENTRYPOINT ./${HOME_DIR}/docker-starter.sh
# ENTRYPOINT ["/usr/sbin/sshd", "-D", "tail", "-f", "/dev/null"]