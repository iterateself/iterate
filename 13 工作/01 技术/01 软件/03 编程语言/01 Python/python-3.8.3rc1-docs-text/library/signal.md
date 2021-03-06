"signal" --- 设置异步事件处理程序
*********************************

======================================================================

该模块提供了在 Python 中使用信号处理程序的机制。


一般规则
========

"signal.signal()" 函数允许定义在接收到信号时执行的自定义处理程序。少量
的默认处理程序已经设置： "SIGPIPE" 被忽略（因此管道和套接字上的写入错
误可以报告为普通的 Python 异常）以及如果父进程没有更改 "SIGINT" ，则其
会被翻译成 "KeyboardInterrupt" 异常。

一旦设置，特定信号的处理程序将保持安装，直到它被显式重置（ Python 模拟
BSD 样式接口而不管底层实现），但 "SIGCHLD" 的处理程序除外，它遵循底层
实现。


执行 Python 信号处理程序
------------------------

Python 信号处理程序不会在低级（ C ）信号处理程序中执行。相反，低级信号
处理程序设置一个标志，告诉 *virtual machine* 稍后执行相应的 Python 信
号处理程序（例如在下一个 *bytecode* 指令）。这会导致：

* 捕获同步错误是没有意义的，例如 "SIGFPE" 或 "SIGSEGV" ，它们是由 C 代
  码中的无效操作引起的。Python 将从信号处理程序返回到 C 代码，这可能会
  再次引发相同的信号，导致 Python 显然的挂起。 从Python 3.3开始，你可
  以使用 "faulthandler" 模块来报告同步错误。

* 纯 C 中实现的长时间运行的计算（例如在大量文本上的正则表达式匹配）可
  以在任意时间内不间断地运行，而不管接收到任何信号。计算完成后将调用
  Python 信号处理程序。


信号与线程
----------

Python 信号处理程序总是在主 Python 线程中执行，即使信号是在另一个线程
中接收的。这意味着信号不能用作线程间通信的手段。 你可以使用
"threading" 模块中的同步原函数。

此外，只允许主线程设置新的信号处理程序。


模块内容
========

在 3.5 版更改: 信号（ SIG* ），处理程序（ "SIG_DFL" ， "SIG_IGN"）和
sigmask（ "SIG_BLOCK" ， "SIG_UNBLOCK" ， "SIG_SETMASK" ）下面列出的相
关常量变成了 "enums" 。 "getsignal()" ， "pthread_sigmask()" ，
"sigpending()" 和 "sigwait()" 函数返回人类可读的 "enums" 。

在 "signal" 模块中定义的变量是：

signal.SIG_DFL

   这是两种标准信号处理选项之一；它只会执行信号的默认函数。 例如，在大
   多数系统上，对于 "SIGQUIT" 的默认操作是转储核心并退出，而对于
   "SIGCHLD" 的默认操作是简单地忽略它。

signal.SIG_IGN

   这是另一个标准信号处理程序，它将简单地忽略给定的信号。

signal.SIGABRT

   Abort signal from *abort(3)*.

signal.SIGALRM

   Timer signal from *alarm(2)*.

   Availability: Unix.

signal.SIGBREAK

   Interrupt from keyboard (CTRL + BREAK).

   可用性: Windows。

signal.SIGBUS

   Bus error (bad memory access).

   Availability: Unix.

signal.SIGCHLD

   Child process stopped or terminated.

   可用性: Windows。

signal.SIGCLD

   Alias to "SIGCHLD".

signal.SIGCONT

   Continue the process if it is currently stopped

   Availability: Unix.

signal.SIGFPE

   Floating-point exception. For example, division by zero.

   参见:

     "ZeroDivisionError" is raised when the second argument of a
     division or modulo operation is zero.

signal.SIGHUP

   Hangup detected on controlling terminal or death of controlling
   process.

   Availability: Unix.

signal.SIGILL

   Illegal instruction.

signal.SIGINT

   Interrupt from keyboard (CTRL + C).

   Default action is to raise "KeyboardInterrupt".

signal.SIGKILL

   Kill signal.

   It cannot be caught, blocked, or ignored.

   Availability: Unix.

signal.SIGPIPE

   Broken pipe: write to pipe with no readers.

   Default action is to ignore the signal.

   Availability: Unix.

signal.SIGSEGV

   Segmentation fault: invalid memory reference.

signal.SIGTERM

   Termination signal.

signal.SIGUSR1

   User-defined signal 1.

   Availability: Unix.

signal.SIGUSR2

   User-defined signal 2.

   Availability: Unix.

signal.SIGWINCH

   Window resize signal.

   Availability: Unix.

SIG*

   All the signal numbers are defined symbolically.  For example, the
   hangup signal is defined as "signal.SIGHUP"; the variable names are
   identical to the names used in C programs, as found in
   "<signal.h>".  The Unix man page for '"signal()"' lists the
   existing signals (on some systems this is *signal(2)*, on others
   the list is in *signal(7)*). Note that not all systems define the
   same set of signal names; only those names defined by the system
   are defined by this module.

signal.CTRL_C_EVENT

   对应于 "Ctrl+C" 击键事件的信号。此信号只能用于 "os.kill()" 。

   可用性: Windows。

   3.2 新版功能.

signal.CTRL_BREAK_EVENT

   对应于 "Ctrl+Break" 击键事件的信号。此信号只能用于 "os.kill()" 。

   可用性: Windows。

   3.2 新版功能.

signal.NSIG

   比最高信号数多一。

signal.ITIMER_REAL

   实时递减间隔计时器，并在到期时发送 "SIGALRM" 。

signal.ITIMER_VIRTUAL

   仅在进程执行时递减间隔计时器，并在到期时发送 SIGVTALRM 。

signal.ITIMER_PROF

   Decrements interval timer both when the process executes and when
   the system is executing on behalf of the process. Coupled with
   ITIMER_VIRTUAL, this timer is usually used to profile the time
   spent by the application in user and kernel space. SIGPROF is
   delivered upon expiration.

signal.SIG_BLOCK

   A possible value for the *how* parameter to "pthread_sigmask()"
   indicating that signals are to be blocked.

   3.3 新版功能.

signal.SIG_UNBLOCK

   A possible value for the *how* parameter to "pthread_sigmask()"
   indicating that signals are to be unblocked.

   3.3 新版功能.

signal.SIG_SETMASK

   A possible value for the *how* parameter to "pthread_sigmask()"
   indicating that the signal mask is to be replaced.

   3.3 新版功能.

The "signal" module defines one exception:

exception signal.ItimerError

   Raised to signal an error from the underlying "setitimer()" or
   "getitimer()" implementation. Expect this error if an invalid
   interval timer or a negative time is passed to "setitimer()". This
   error is a subtype of "OSError".

   3.3 新版功能: This error used to be a subtype of "IOError", which
   is now an alias of "OSError".

The "signal" module defines the following functions:

signal.alarm(time)

   If *time* is non-zero, this function requests that a "SIGALRM"
   signal be sent to the process in *time* seconds. Any previously
   scheduled alarm is canceled (only one alarm can be scheduled at any
   time).  The returned value is then the number of seconds before any
   previously set alarm was to have been delivered. If *time* is zero,
   no alarm is scheduled, and any scheduled alarm is canceled.  If the
   return value is zero, no alarm is currently scheduled.

   Availability: Unix.  See the man page *alarm(2)* for further
   information.

signal.getsignal(signalnum)

   Return the current signal handler for the signal *signalnum*. The
   returned value may be a callable Python object, or one of the
   special values "signal.SIG_IGN", "signal.SIG_DFL" or "None".  Here,
   "signal.SIG_IGN" means that the signal was previously ignored,
   "signal.SIG_DFL" means that the default way of handling the signal
   was previously in use, and "None" means that the previous signal
   handler was not installed from Python.

signal.strsignal(signalnum)

   Return the system description of the signal *signalnum*, such as
   "Interrupt", "Segmentation fault", etc. Returns "None" if the
   signal is not recognized.

   3.8 新版功能.

signal.valid_signals()

   Return the set of valid signal numbers on this platform.  This can
   be less than "range(1, NSIG)" if some signals are reserved by the
   system for internal use.

   3.8 新版功能.

signal.pause()

   Cause the process to sleep until a signal is received; the
   appropriate handler will then be called.  Returns nothing.

   Availability: Unix.  See the man page *signal(2)* for further
   information.

   See also "sigwait()", "sigwaitinfo()", "sigtimedwait()" and
   "sigpending()".

signal.raise_signal(signum)

   Sends a signal to the calling process. Returns nothing.

   3.8 新版功能.

signal.pthread_kill(thread_id, signalnum)

   Send the signal *signalnum* to the thread *thread_id*, another
   thread in the same process as the caller.  The target thread can be
   executing any code (Python or not).  However, if the target thread
   is executing the Python interpreter, the Python signal handlers
   will be executed by the main thread.  Therefore, the only point of
   sending a signal to a particular Python thread would be to force a
   running system call to fail with "InterruptedError".

   Use "threading.get_ident()" or the "ident" attribute of
   "threading.Thread" objects to get a suitable value for *thread_id*.

   If *signalnum* is 0, then no signal is sent, but error checking is
   still performed; this can be used to check if the target thread is
   still running.

   Raises an auditing event "signal.pthread_kill" with arguments
   "thread_id", "signalnum".

   Availability: Unix.  See the man page *pthread_kill(3)* for further
   information.

   See also "os.kill()".

   3.3 新版功能.

signal.pthread_sigmask(how, mask)

   Fetch and/or change the signal mask of the calling thread.  The
   signal mask is the set of signals whose delivery is currently
   blocked for the caller. Return the old signal mask as a set of
   signals.

   The behavior of the call is dependent on the value of *how*, as
   follows.

   * "SIG_BLOCK": The set of blocked signals is the union of the
     current set and the *mask* argument.

   * "SIG_UNBLOCK": The signals in *mask* are removed from the current
     set of blocked signals.  It is permissible to attempt to unblock
     a signal which is not blocked.

   * "SIG_SETMASK": The set of blocked signals is set to the *mask*
     argument.

   *mask* is a set of signal numbers (e.g. {"signal.SIGINT",
   "signal.SIGTERM"}). Use "valid_signals()" for a full mask including
   all signals.

   For example, "signal.pthread_sigmask(signal.SIG_BLOCK, [])" reads
   the signal mask of the calling thread.

   "SIGKILL" and "SIGSTOP" cannot be blocked.

   Availability: Unix.  See the man page *sigprocmask(3)* and
   *pthread_sigmask(3)* for further information.

   See also "pause()", "sigpending()" and "sigwait()".

   3.3 新版功能.

signal.setitimer(which, seconds, interval=0.0)

   Sets given interval timer (one of "signal.ITIMER_REAL",
   "signal.ITIMER_VIRTUAL" or "signal.ITIMER_PROF") specified by
   *which* to fire after *seconds* (float is accepted, different from
   "alarm()") and after that every *interval* seconds (if *interval*
   is non-zero). The interval timer specified by *which* can be
   cleared by setting *seconds* to zero.

   When an interval timer fires, a signal is sent to the process. The
   signal sent is dependent on the timer being used;
   "signal.ITIMER_REAL" will deliver "SIGALRM",
   "signal.ITIMER_VIRTUAL" sends "SIGVTALRM", and "signal.ITIMER_PROF"
   will deliver "SIGPROF".

   The old values are returned as a tuple: (delay, interval).

   Attempting to pass an invalid interval timer will cause an
   "ItimerError".

   Availability: Unix.

signal.getitimer(which)

   Returns current value of a given interval timer specified by
   *which*.

   Availability: Unix.

signal.set_wakeup_fd(fd, *, warn_on_full_buffer=True)

   Set the wakeup file descriptor to *fd*.  When a signal is received,
   the signal number is written as a single byte into the fd.  This
   can be used by a library to wakeup a poll or select call, allowing
   the signal to be fully processed.

   The old wakeup fd is returned (or -1 if file descriptor wakeup was
   not enabled).  If *fd* is -1, file descriptor wakeup is disabled.
   If not -1, *fd* must be non-blocking.  It is up to the library to
   remove any bytes from *fd* before calling poll or select again.

   When threads are enabled, this function can only be called from the
   main thread; attempting to call it from other threads will cause a
   "ValueError" exception to be raised.

   There are two common ways to use this function. In both approaches,
   you use the fd to wake up when a signal arrives, but then they
   differ in how they determine *which* signal or signals have
   arrived.

   In the first approach, we read the data out of the fd's buffer, and
   the byte values give you the signal numbers. This is simple, but in
   rare cases it can run into a problem: generally the fd will have a
   limited amount of buffer space, and if too many signals arrive too
   quickly, then the buffer may become full, and some signals may be
   lost. If you use this approach, then you should set
   "warn_on_full_buffer=True", which will at least cause a warning to
   be printed to stderr when signals are lost.

   In the second approach, we use the wakeup fd *only* for wakeups,
   and ignore the actual byte values. In this case, all we care about
   is whether the fd's buffer is empty or non-empty; a full buffer
   doesn't indicate a problem at all. If you use this approach, then
   you should set "warn_on_full_buffer=False", so that your users are
   not confused by spurious warning messages.

   在 3.5 版更改: On Windows, the function now also supports socket
   handles.

   在 3.7 版更改: Added "warn_on_full_buffer" parameter.

signal.siginterrupt(signalnum, flag)

   Change system call restart behaviour: if *flag* is "False", system
   calls will be restarted when interrupted by signal *signalnum*,
   otherwise system calls will be interrupted.  Returns nothing.

   Availability: Unix.  See the man page *siginterrupt(3)* for further
   information.

   Note that installing a signal handler with "signal()" will reset
   the restart behaviour to interruptible by implicitly calling
   "siginterrupt()" with a true *flag* value for the given signal.

signal.signal(signalnum, handler)

   Set the handler for signal *signalnum* to the function *handler*.
   *handler* can be a callable Python object taking two arguments (see
   below), or one of the special values "signal.SIG_IGN" or
   "signal.SIG_DFL".  The previous signal handler will be returned
   (see the description of "getsignal()" above).  (See the Unix man
   page *signal(2)* for further information.)

   When threads are enabled, this function can only be called from the
   main thread; attempting to call it from other threads will cause a
   "ValueError" exception to be raised.

   The *handler* is called with two arguments: the signal number and
   the current stack frame ("None" or a frame object; for a
   description of frame objects, see the description in the type
   hierarchy or see the attribute descriptions in the "inspect"
   module).

   On Windows, "signal()" can only be called with "SIGABRT", "SIGFPE",
   "SIGILL", "SIGINT", "SIGSEGV", "SIGTERM", or "SIGBREAK". A
   "ValueError" will be raised in any other case. Note that not all
   systems define the same set of signal names; an "AttributeError"
   will be raised if a signal name is not defined as "SIG*" module
   level constant.

signal.sigpending()

   Examine the set of signals that are pending for delivery to the
   calling thread (i.e., the signals which have been raised while
   blocked).  Return the set of the pending signals.

   Availability: Unix.  See the man page *sigpending(2)* for further
   information.

   See also "pause()", "pthread_sigmask()" and "sigwait()".

   3.3 新版功能.

signal.sigwait(sigset)

   Suspend execution of the calling thread until the delivery of one
   of the signals specified in the signal set *sigset*.  The function
   accepts the signal (removes it from the pending list of signals),
   and returns the signal number.

   Availability: Unix.  See the man page *sigwait(3)* for further
   information.

   See also "pause()", "pthread_sigmask()", "sigpending()",
   "sigwaitinfo()" and "sigtimedwait()".

   3.3 新版功能.

signal.sigwaitinfo(sigset)

   Suspend execution of the calling thread until the delivery of one
   of the signals specified in the signal set *sigset*.  The function
   accepts the signal and removes it from the pending list of signals.
   If one of the signals in *sigset* is already pending for the
   calling thread, the function will return immediately with
   information about that signal. The signal handler is not called for
   the delivered signal. The function raises an "InterruptedError" if
   it is interrupted by a signal that is not in *sigset*.

   The return value is an object representing the data contained in
   the "siginfo_t" structure, namely: "si_signo", "si_code",
   "si_errno", "si_pid", "si_uid", "si_status", "si_band".

   Availability: Unix.  See the man page *sigwaitinfo(2)* for further
   information.

   See also "pause()", "sigwait()" and "sigtimedwait()".

   3.3 新版功能.

   在 3.5 版更改: The function is now retried if interrupted by a
   signal not in *sigset* and the signal handler does not raise an
   exception (see **PEP 475** for the rationale).

signal.sigtimedwait(sigset, timeout)

   Like "sigwaitinfo()", but takes an additional *timeout* argument
   specifying a timeout. If *timeout* is specified as "0", a poll is
   performed. Returns "None" if a timeout occurs.

   Availability: Unix.  See the man page *sigtimedwait(2)* for further
   information.

   See also "pause()", "sigwait()" and "sigwaitinfo()".

   3.3 新版功能.

   在 3.5 版更改: The function is now retried with the recomputed
   *timeout* if interrupted by a signal not in *sigset* and the signal
   handler does not raise an exception (see **PEP 475** for the
   rationale).


示例
====

Here is a minimal example program. It uses the "alarm()" function to
limit the time spent waiting to open a file; this is useful if the
file is for a serial device that may not be turned on, which would
normally cause the "os.open()" to hang indefinitely.  The solution is
to set a 5-second alarm before opening the file; if the operation
takes too long, the alarm signal will be sent, and the handler raises
an exception.

   import signal, os

   def handler(signum, frame):
       print('Signal handler called with signal', signum)
       raise OSError("Couldn't open device!")

   # Set the signal handler and a 5-second alarm
   signal.signal(signal.SIGALRM, handler)
   signal.alarm(5)

   # This open() may hang indefinitely
   fd = os.open('/dev/ttyS0', os.O_RDWR)

   signal.alarm(0)          # Disable the alarm


Note on SIGPIPE
===============

Piping output of your program to tools like *head(1)* will cause a
"SIGPIPE" signal to be sent to your process when the receiver of its
standard output closes early.  This results in an exception like
"BrokenPipeError: [Errno 32] Broken pipe".  To handle this case, wrap
your entry point to catch this exception as follows:

   import os
   import sys

   def main():
       try:
           # simulate large output (your code replaces this loop)
           for x in range(10000):
               print("y")
           # flush output here to force SIGPIPE to be triggered
           # while inside this try block.
           sys.stdout.flush()
       except BrokenPipeError:
           # Python flushes standard streams on exit; redirect remaining output
           # to devnull to avoid another BrokenPipeError at shutdown
           devnull = os.open(os.devnull, os.O_WRONLY)
           os.dup2(devnull, sys.stdout.fileno())
           sys.exit(1)  # Python exits with error code 1 on EPIPE

   if __name__ == '__main__':
       main()

Do not set "SIGPIPE"'s disposition to "SIG_DFL" in order to avoid
"BrokenPipeError".  Doing that would cause your program to exit
unexpectedly also whenever any socket connection is interrupted while
your program is still writing to it.
