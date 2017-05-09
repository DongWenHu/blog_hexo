---
title: Linux 共享内存实现原理
date: 2017-04-24 11:22:48
categories: 面试基础必备
---
------
## **Linux 共享内存实现原理**
1、共享内存的实现采用了内存映射技术
2、共享内存主要集中3个内核函数，*do_shmat*、*sys_shmat*、*sys_shmdt*
3、*sys_shmat*调用*do_shmat*实现内存共享的attach
4、*sys_shmdt*实现内存共享的detach和destroy，首先查找相应的vma，如果找到执行ummap操作
<!--more-->
***do_shmat原理***
当系统提出attach或创建共享内存时，内核会为每一个共享内存段提供一个*shmid_kernel*的数据结构:
```c
struct shmid_kernel /* private to the kernel */
{  
    struct kern_ipc_perm    shm_perm; // 访问权限的信息
    struct file *       shm_file; // 指向虚拟文件系统的指针
    unsigned long       shm_nattch; // 有多少个进程attach上了这个共享内存段
    unsigned long       shm_segsz; // 共享内存段大小
    // 以下是一些访问时间的相关信息
    time_t          shm_atim;
    time_t          shm_dtim;
    time_t          shm_ctim；
   
    pid_t           shm_cprid;
    pid_t           shm_lprid;
    struct user_struct  *mlock_user;
};
```
struct file定义参考如下:
```c
struct file {
        /*
         * fu_list becomes invalid after file_free is called and queued via
         * fu_rcuhead for RCU freeing
         */
        union {
                struct list_head        fu_list;
                struct rcu_head         fu_rcuhead;
        } f_u;
        struct path             f_path;
#define f_dentry        f_path.dentry
#define f_vfsmnt        f_path.mnt
        const struct file_operations    *f_op;
        atomic_t                f_count;
        unsigned int            f_flags;
        mode_t                  f_mode;
        loff_t                  f_pos;
        struct fown_struct      f_owner;
        unsigned int            f_uid, f_gid;
        struct file_ra_state    f_ra;
        unsigned long           f_version;
#ifdef CONFIG_SECURITY
        void                    *f_security;
#endif
        /* needed for tty driver, and maybe others */
        void                    *private_data;
#ifdef CONFIG_EPOLL
        /* Used by fs/eventpoll.c to link all the hooks to this file */
        struct list_head        f_ep_links;
        spinlock_t              f_ep_lock;
#endif /* #ifdef CONFIG_EPOLL */
        struct address_space    *f_mapping;
};
```
1、该数据结构中最重要的字段就是shm_file，它指向了共享内存对应的虚拟文件
2、shm_file中的f_mapping指向了该内存段使用的页面(物理内存)
3、当进程创建或attach共享内存时，在用户态先向虚拟内存系统申请各自的vma_struct, 内部成员vm_file指向shm_file
4、这样就完成了虚拟内存, 共享内存(文件系统)和物理内存的连接
