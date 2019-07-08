RIOT 线程
==========

## 结构体
默认最大线程数为32
```
    // core/include/kernel_types.h
    /**
    * @def MAXTHREADS
    * @brief The maximum number of threads to be scheduled
    */
    #ifndef MAXTHREADS
    #define MAXTHREADS 32
    #endif

    /**
    * Canonical identifier for an invalid PID.
    */
    #define KERNEL_PID_UNDEF 0
```

线程控制块大小与使用的模块相关。
```
    // core/include/thread.h
    struct _thread {
        char *sp;                       /**< thread's stack pointer         */
        uint8_t status;                 /**< thread's status                */
        uint8_t priority;               /**< thread's priority              */

        kernel_pid_t pid;               /**< thread's process id            */

    #if defined(MODULE_CORE_THREAD_FLAGS) || defined(DOXYGEN)
        thread_flags_t flags;           /**< currently set flags            */
    #endif

        clist_node_t rq_entry;          /**< run queue entry                */

    #if defined(MODULE_CORE_MSG) || defined(MODULE_CORE_THREAD_FLAGS) \
        || defined(MODULE_CORE_MBOX) || defined(DOXYGEN)
        void *wait_data;                /**< used by msg, mbox and thread
                                            flags                          */
    #endif
    #if defined(MODULE_CORE_MSG) || defined(DOXYGEN)
        list_node_t msg_waiters;        /**< threads waiting for their message
                                            to be delivered to this thread
                                            (i.e. all blocked sends)       */
        cib_t msg_queue;                /**< index of this [thread's message queue]
                                            (thread_t::msg_array), if any  */
        msg_t *msg_array;               /**< memory holding messages sent
                                            to this thread's message queue */
    #endif
    #if defined(DEVELHELP) || defined(SCHED_TEST_STACK) \
        || defined(MODULE_MPU_STACK_GUARD) || defined(DOXYGEN)
        char *stack_start;              /**< thread's stack start address   */
    #endif
    #if defined(DEVELHELP) || defined(DOXYGEN)
        const char *name;               /**< thread's name                  */
        int stack_size;                 /**< thread's stack size            */
    #endif
    #ifdef HAVE_THREAD_ARCH_T
        thread_arch_t arch;             /**< architecture dependent part    */
    #endif
    };
```

## 线程数组
使用指针数组存储线程控制块地址
```
// core/sched.c
volatile thread_t *sched_threads[KERNEL_PID_LAST + 1];
volatile thread_t *sched_active_thread;
```
## 线程创建
堆栈顶记录着线程的控制模块,创建线程时初始化线程sp。
```
//core/thread.c 
kernel_pid_t thread_create(char *stack, int stacksize, char priority, int flags, thread_task_func_t function, void *arg, const char *name)
{
    ...
    /* allocate our thread control block at the top of our stackspace */
    thread_t *cb = (thread_t *) (stack + stacksize);
    ...
    /* allocate our thread control block at the top of our stackspace */
    thread_t *cb = (thread_t *) (stack + stacksize);
    ...
    cb->pid = pid;
    cb->sp = thread_stack_init(function, arg, stack, stacksize);

}
```