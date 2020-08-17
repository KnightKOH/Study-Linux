# 破解 root 密码

原理：之前的密码经过加密处理，无法进行破解（不能找回原密码），只能重新设置一个新密码

以 root 的身份执行 passwd 命令

### 进入shell

1. 重启系统
2. 将光标移动到要启动的内核
3. 编译当前条目
4. 光标移至内核命令行
5. 末尾添加 rb.break
6. 继续启动

### 重置密码

1. 以读写方式挂载 /sysroot

   > switch_root:/# mount -o remount, rw /sysroot

2. 切换至真正操作系统的根 /sysroot

   > switch_root:/# chroot /sysroot

3. 重置密码

   > sh-5.6# echo redhat | passwd --stdin root

4. 打标签

   > sh-5.6# touch /.autorelabel

5. exit 退出两次



以读写方式挂载根时，另一种方法是修改内核命令行中 ro 为 rw