设置POSIX capabilities
==========================

`POSIX capabilities`_ 允许为进程细化权限。除了标准的UNIX权限方案之外，它们为系统资源定义了新的一组权限。要启用capabilities支持 (仅Linux)，构建uWSGI之前，你必须安装 ``libcap`` 头文件 (在基于Debian的发行版上，则是 ``libcap-dev`` )。像往常一样，在 ``setuid`` 调用之后，你的进程将会失去几乎所有的capabilities。uWSGI的 ``cap`` 选项允许你定义一个通过调用保持的capabilities列表。

例如，要允许你的非特权应用绑定到特权端口，并且设置系统时钟，你将使用以下选项。

.. code-block:: sh

  uwsgi --socket :1000 --uid 5000 --gid 5000 --cap net_bind_service,sys_time

uWSGI生成的所有进程稍后将继承此行为。如果你的系统支持在uWSGI列表中不可用的capabilities，那么你可以简单指定常数数：

.. code-block:: sh

  uwsgi --socket :1000 --uid 5000 --gid 5000 --cap net_bind_service,sys_time,42

除了 ``net_bind_service`` 和 ``sys_time`` 之外，添加了一个数字为“42”的新capability。

.. _POSIX capabilities: http://en.wikipedia.org/wiki/Capability-based_security

可用capabilities
----------------------


这是可用capabilities列表。

==================  ======================
audit_control       CAP_AUDIT_CONTROL
audit_write         CAP_AUDIT_WRITE
chown               CAP_CHOWN
dac_override        CAP_DAC_OVERRIDE
dac_read_search     CAP_DAC_READ_SEARCH
fowner              CAP_FOWNER
fsetid              CAP_FSETID
ipc_lock            CAP_IPC_LOCK
ipc_owner           CAP_IPC_OWNER
kill                CAP_KILL
lease               CAP_LEASE
linux_immutable     CAP_LINUX_IMMUTABLE
mac_admin           CAP_MAC_ADMIN
mac_override        CAP_MAC_OVERRIDE
mknod               CAP_MKNOD
net_admin           CAP_NET_ADMIN
net_bind_service    CAP_NET_BIND_SERVICE
net_broadcast       CAP_NET_BROADCAST
net_raw             CAP_NET_RAW
setfcap             CAP_SETFCAP
setgid              CAP_SETGID
setpcap             CAP_SETPCAP
setuid              CAP_SETUID
sys_admin           CAP_SYS_ADMIN
sys_boot            CAP_SYS_BOOT
sys_chroot          CAP_SYS_CHROOT
sys_module          CAP_SYS_MODULE
sys_nice            CAP_SYS_NICE
sys_pacct           CAP_SYS_PACCT
sys_ptrace          CAP_SYS_PTRACE
sys_rawio           CAP_SYS_RAWIO
sys_resource        CAP_SYS_RESOURCE
sys_time            CAP_SYS_TIME
sys_tty_config      CAP_SYS_TTY_CONFIG
syslog              CAP_SYSLOG
wake_alarm          CAP_WAKE_ALARM
==================  ======================
