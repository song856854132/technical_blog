---
title: Kernel Tracing Chapter 1 - Boot Up
date: 2024-02-21
categories: [Linux, Kernel]
tags: [KVM, Linux]
img_path: /assets/img/kernel/
---

## How Boot Operate

![Img](bootlinux1.png)

![Img](linuxboot.png)

## Breaking Point1 - Start_Kernel

```C
asmlinkage __visible void __init __no_sanitize_address start_kernel(void)
{
	char *command_line;
	char *after_dashes;

	set_task_stack_end_magic(&init_task);
	smp_setup_processor_id();
	debug_objects_early_init();
	cgroup_init_early();
	local_irq_disable();
	early_boot_irqs_disabled = true;
	boot_cpu_init();
	page_address_init();
	// Sooooo many init prepare workload.
	cpuset_init();
	cgroup_init();
	taskstats_init_early();
	delayacct_init();
	poking_init();
	check_bugs();
	
	arch_call_rest_init();  // -> function who call rest_init();
	prevent_tail_call_optimization();
}
```

Function `rest_init` is where kernel spawn the first thread to execute `init` program. So far we are still in kernel space, and we are about to step into next stage by following the function call to `kernel_init`.

```C
noinline void __ref rest_init(void)
{
	struct task_struct *tsk;
	int pid;
	rcu_scheduler_starting();
	/*
	 * We need to spawn init first so that it obtains pid 1, however
	 * the init task will end up wanting to create kthreads, which, if
	 * we schedule it before we create kthreadd, will OOPS.
	 */
	pid = kernel_thread(kernel_init, NULL, CLONE_FS);
	/*
	 * Pin init on the boot CPU. Task migration is not properly working
	 * until sched_init_smp() has been run. It will set the allowed
	 * CPUs for init to the non isolated CPUs.
	 */
	rcu_read_lock();
	tsk = find_task_by_pid_ns(pid, &init_pid_ns);
	set_cpus_allowed_ptr(tsk, cpumask_of(smp_processor_id()));
	rcu_read_unlock();
	numa_default_policy();
	pid = kernel_thread(kthreadd, NULL, CLONE_FS | CLONE_FILES);
	rcu_read_lock();
	kthreadd_task = find_task_by_pid_ns(pid, &init_pid_ns);
	rcu_read_unlock();
	/*
	 * Enable might_sleep() and smp_processor_id() checks.
	 * They cannot be enabled earlier because with CONFIG_PREEMPTION=y
	 * kernel_thread() would trigger might_sleep() splats. With
	 * CONFIG_PREEMPT_VOLUNTARY=y the init task might have scheduled
	 * already, but it's stuck on the kthreadd_done completion.
	 */
	system_state = SYSTEM_SCHEDULING;
	complete(&kthreadd_done);
	/*
	 * The boot idle thread must execute schedule()
	 * at least once to get things moving:
	 */
	schedule_preempt_disabled();
	/* Call into cpu_idle with preempt disabled */
	cpu_startup_entry(CPUHP_ONLINE);
}
```
![Img](rest_init_pid1.png)

![Img](rest_init_pid1.png)


In the if-else statement, `ramdisk_execute_command` or `/init` was mounted to initial proccess from the extracted CPIO archive, or `initramfs` that we'd been explained in last article.

```C
static int __ref kernel_init(void *unused)
{
	int ret;
	kernel_init_freeable();
	async_synchronize_full();
	kprobe_free_init_mem();
	ftrace_free_init_mem();
	free_initmem();
	mark_readonly();
	pti_finalize();
	system_state = SYSTEM_RUNNING;
	numa_default_policy();
	rcu_end_inkernel_boot();
	do_sysctl_args();
	if (ramdisk_execute_command) {
		ret = run_init_process(ramdisk_execute_command); // -> ramdisk_execute_command = '/init'
		if (!ret)
			return 0;
		pr_err("Failed to execute %s (error %d)\n",
		       ramdisk_execute_command, ret);
	}
	
	if (execute_command) {
		ret = run_init_process(execute_command);
		if (!ret)
			return 0;
		panic("Requested init %s failed (error %d).",
		      execute_command, ret);
	}
	if (CONFIG_DEFAULT_INIT[0] != '\0') {
		ret = run_init_process(CONFIG_DEFAULT_INIT);
		if (ret)
			pr_err("Default init %s failed (error %d)\n",
			       CONFIG_DEFAULT_INIT, ret);
		else
			return 0;
	}
	if (!try_to_run_init_process("/sbin/init") ||
	    !try_to_run_init_process("/etc/init") ||
	    !try_to_run_init_process("/bin/init") ||
	    !try_to_run_init_process("/bin/sh"))
		return 0;
		panic("No working init found.  Try passing init= option to kernel. "
	      "See Linux Documentation/admin-guide/init.rst for guidance.");
}
```

![Img](kernel_init_entry.png)

![Img](booting_screen.png)