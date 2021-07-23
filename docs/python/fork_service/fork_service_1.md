## 概念

守护进程(Daemon)也称为精灵进程是一种生存期较长的一种进程。它们独立于控制终端并且周期性的执行某种任务或等待处理某些发生的事件。他们常常在系统引导装入时启动，在系统关闭时终止。unix系统有很多守护进程，大多数服务器都是用守护进程实现的，例如inetd守护进程。

## 需要了解的相关概念

- 进程 (process)
- 进程组 (process group)
- 会话 (session)

**可参考以下博文**

- [inux进程基础](https://www.cnblogs.com/vamei/archive/2012/09/20/2694466.html)
- [Linux进程关系](http://www.cnblogs.com/vamei/archive/2012/10/07/2713023.html)

## 实现原理

参考 [APUE关于守护进程的章节](https://yq.aliyun.com/articles/41477?spm=5176.100240.searchblog.39.HyGU9I)

**大致流程如下：**

1. 后台运行

    `首次fork，创建父-子进程，使父进程退出`

2. 脱离控制终端，登录会话和进程组

    `通过setsid使子进程成为process group leader、session leader`

3. 禁止进程重新打开控制终端

    `二次fork，创建子-孙进程，使sid不等pid`

4. 关闭打开的文件描述符

    `通常就关闭STDIN、STDOUT和STDERR`

5. 改变当前工作目录

    `防止占用别的路径的working dir的fd，导致一些block不能unmount`

6. 重设umask

    `防止后续子进程继承非默认umask造成奇怪的行为`

7. 处理SIGCHLD信号
    
    `非必需`

8. 日志
    
    `输出重定向后，需要有机制放映内部情况`


## 关于两次fork

第二个fork不是必须的，只是为了防止进程打开控制终端。

打开一个控制终端的条件是该进程必须是session leader。第一次fork，setsid之后，子进程成为session leader，进程可以打开终端；第二次fork产生的进程，不再是session leader，进程则无法打开终端。

也就是说，只要程序实现得好，控制程序不主动打开终端，无第二次fork亦可。

## 代码实现

```python
# coding: utf-8

import os
import sys
import time
import atexit
import signal


class Daemon:
    def __init__(self, pidfile='/tmp/daemon.pid', stdin='/dev/null', stdout='/dev/null', stderr='/dev/null'):
        self.stdin = stdin
        self.stdout = stdout
        self.stderr = stderr
        self.pidfile = pidfile

    def daemonize(self):
        if os.path.exists(self.pidfile):
            raise RuntimeError('Already running.')

        # First fork (detaches from parent)
        try:
            if os.fork() > 0:
                raise SystemExit(0)
        except OSError as e:
            raise RuntimeError('fork #1 faild: {0} ({1})\n'.format(e.errno, e.strerror))

        os.chdir('/')
        os.setsid()
        os.umask(0o22)

        # Second fork (relinquish session leadership)
        try:
            if os.fork() > 0:
                raise SystemExit(0)
        except OSError as e:
            raise RuntimeError('fork #2 faild: {0} ({1})\n'.format(e.errno, e.strerror))

        # Flush I/O buffers
        sys.stdout.flush()
        sys.stderr.flush()

        # Replace file descriptors for stdin, stdout, and stderr
        with open(self.stdin, 'rb', 0) as f:
            os.dup2(f.fileno(), sys.stdin.fileno())
        with open(self.stdout, 'ab', 0) as f:
            os.dup2(f.fileno(), sys.stdout.fileno())
        with open(self.stderr, 'ab', 0) as f:
            os.dup2(f.fileno(), sys.stderr.fileno())

        # Write the PID file
        with open(self.pidfile, 'w') as f:
            print(os.getpid(), file=f)

        # Arrange to have the PID file removed on exit/signal
        atexit.register(lambda: os.remove(self.pidfile))

        signal.signal(signal.SIGTERM, self.__sigterm_handler)

    # Signal handler for termination (required)
    @staticmethod
    def __sigterm_handler(signo, frame):
        raise SystemExit(1)

    def start(self):
        try:
            self.daemonize()
        except RuntimeError as e:
            print(e, file=sys.stderr)
            raise SystemExit(1)

        self.run()

    def stop(self):
        try:
            if os.path.exists(self.pidfile):
                with open(self.pidfile) as f:
                    os.kill(int(f.read()), signal.SIGTERM)
            else:
                print('Not running.', file=sys.stderr)
                raise SystemExit(1)
        except OSError as e:
            if 'No such process' in str(e) and os.path.exists(self.pidfile): 
                os.remove(self.pidfile)

    def restart(self):
        self.stop()
        self.start()

    def run(self):
        pass
```

## 使用测试

```python
import os
import sys
import time

from daemon import Daemon

class MyTestDaemon(Daemon):
    def run(self):
        sys.stdout.write('Daemon started with pid {}\n'.format(os.getpid()))
        while True:
            sys.stdout.write('Daemon Alive! {}\n'.format(time.ctime()))
            sys.stdout.flush()

            time.sleep(5)

if __name__ == '__main__':
    PIDFILE = '/tmp/daemon-example.pid'
    LOG = '/tmp/daemon-example.log'
    daemon = MyTestDaemon(pidfile=PIDFILE, stdout=LOG, stderr=LOG)

    if len(sys.argv) != 2:
        print('Usage: {} [start|stop]'.format(sys.argv[0]), file=sys.stderr)
        raise SystemExit(1)

    if 'start' == sys.argv[1]:
        daemon.start()
    elif 'stop' == sys.argv[1]:
        daemon.stop()
    elif 'restart' == sys.argv[1]:
        daemon.restart()
    else:
        print('Unknown command {!r}'.format(sys.argv[1]), file=sys.stderr)
        raise SystemExit(1)
```

```shell
[daemon] python test.py start                                         23:45:42
[daemon] cat /tmp/daemon-example.pid                                  23:45:49
8532
[daemon] ps -ef|grep 8532 | grep -v grep                              23:46:07
  502  8532     1   0 11:45下午 ??         0:00.00 python test.py start
[daemon] tail -f /tmp/daemon-example.log                              23:46:20
Daemon started with pid 8532
Daemon Alive! Fri Dec  2 23:45:49 2016
Daemon Alive! Fri Dec  2 23:45:54 2016
Daemon Alive! Fri Dec  2 23:45:59 2016
Daemon Alive! Fri Dec  2 23:46:04 2016
Daemon Alive! Fri Dec  2 23:46:09 2016
Daemon Alive! Fri Dec  2 23:46:14 2016
Daemon Alive! Fri Dec  2 23:46:19 2016
Daemon Alive! Fri Dec  2 23:46:24 2016
Daemon Alive! Fri Dec  2 23:46:29 2016
Daemon Alive! Fri Dec  2 23:46:34 2016

[daemon] python test.py stop                                          23:46:36
[daemon] ps -ef|grep 8532 | grep -v grep                              23:46:43
```

> 也可以使用 Supervisor 管理进程，具体可看 [Supervisor安装与配置](https://blog.csdn.net/xyang81/article/details/51555473)

## 参考

- [tzuryby/daemon.py](https://gist.github.com/tzuryby/961228) python2实现的通用的python daemon类
- [12.14 在Unix系统上面启动守护进程](http://python3-cookbook.readthedocs.io/zh_CN/latest/c12/p14_launching_daemon_process_on_unix.html#id3) python3实现的daemon