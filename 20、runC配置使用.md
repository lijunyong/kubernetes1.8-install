# runC
RunC 是一个轻量级的工具，它是用来运行容器的，只用来做这一件事，并且这一件事要做好。我们可以认为它就是个命令行小工具，可以不用通过 docker 引擎，直接运行容器。事实上，runC 是标准化的产物，它根据 OCI 标准来创建和运行容器。而 OCI(Open Container Initiative)组织，旨在围绕容器格式和运行时制定一个开放的工业化标准

```
# create a 'github.com/opencontainers' in your GOPATH/src
cd github.com/opencontainers
git clone https://github.com/opencontainers/runc
cd runc

[root@node2 runc]# make
go build -buildmode=pie  -ldflags "-X main.gitCommit="4fc53a81fb7c994640722ac585fa9ca548971871" -X main.version=1.0.0-rc5 " -tags "seccomp" -o runc .
# pkg-config --cflags libseccomp libseccomp
Package libseccomp was not found in the pkg-config search path.
Perhaps you should add the directory containing `libseccomp.pc'
to the PKG_CONFIG_PATH environment variable
No package 'libseccomp' found
Package libseccomp was not found in the pkg-config search path.
Perhaps you should add the directory containing `libseccomp.pc'
to the PKG_CONFIG_PATH environment variable
No package 'libseccomp' found
pkg-config: exit status 1
make: *** [runc] Error 2

[root@node2 runc]# yum install libseccomp
[root@node2 runc]# make BUILDTAGS=""
go build -buildmode=pie  -ldflags "-X main.gitCommit="4fc53a81fb7c994640722ac585fa9ca548971871" -X main.version=1.0.0-rc5 " -tags "" -o runc .
[root@node2 runc]# make install
install -D -m0755 runc /usr/local/sbin/runc
[root@node2 runc]#
```
