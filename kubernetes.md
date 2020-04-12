https://kubernetes.io/docs/tasks/configure-pod-container/assign-memory-resource/#exceed-a-containers-memory-limit

"The output includes a record of the Container being killed because of an out-of-memory condition:
Warning OOMKilling Memory cgroup out of memory: Kill process 4481 (stress) score 1994 or sacrifice child"【文档中的这段话说明kubernetes可能会通过cgroup进行memory的限制】


查看cgroup limit的方法： cat /sys/fs/cgroup/memory/kubepods.slice/kubepods-podaace5b66_c7d0_11e9_ba2a_dcf401d01e81.slice/memory.limit_in_bytes
https://github.com/kubernetes/kubernetes/issues/82230


linux kernel cgroup文档：
https://www.kernel.org/doc/Documentation/cgroup-v1/memory.txt
https://www.kernel.org/doc/Documentation/cgroup-v2.txt
https://www.kernel.org/doc/html/latest/admin-guide/cgroup-v2.html



cgroup v2 文档

“memory.max
A read-write single value file which exists on non-root cgroups. The default is “max”.
Memory usage hard limit. This is the final protection mechanism. If a cgroup’s memory usage reaches this limit and can’t be reduced, the OOM killer is invoked in the cgroup. Under certain circumstances, the usage may go over the limit temporarily.
This is the ultimate protection mechanism. As long as the high limit is used and monitored properly, this limit’s utility is limited to providing the final safety net.”






https://kubernetes.io/docs/tasks/administer-cluster/out-of-resource/#node-oom-behavior
当Node 出现OOM 的时候 kubernetes文档连接到了这篇关于oom-killer的文档：
https://lwn.net/Articles/391222/


关于存活检查，就绪检查
Configure Liveness, Readiness and Startup Probes


关于CPU资源： "1 Hyperthread on a bare-metal Intel processor with Hyperthreading"  https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/


关于计算资源：https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/


问题点：
在一台多核心CPU的Node上，设置CPU require 和 CPU limit 为 1。和单核机器上执行的区别是什么？是否会出现两个线程同事被两个CPU执行的情况？通过多线程编程，由多个CPU分担计算任务的性能优化在这种环境中还是否有效呢？

问题点：
在kubernetes中，Pod使用的资源，是否等于这个Pod当中所有container使用资源的总和？除containers使用资源外，Pod是否会消耗额外的资源。
kubernetes文档： “When you run a Pod on a Node, the Pod itself takes an amount of system resources. These resources are additional to the resources needed to run the container(s) inside the Pod. Pod Overhead is a feature for accounting for the resources consumed by the pod infrastructure on top of the container requests & limits.”

“When Pod Overhead is enabled, the overhead is considered in addition to the sum of container resource requests when scheduling a pod. Similarly, Kubelet will include the pod overhead when sizing the pod cgroup, and when carrying out pod eviction ranking.” 【kubernetes居然会给Pod设置cgroup? 所以说不仅每个容器会有一个cgroup, 每个Pod也会有cgroup? 】https://kubernetes.io/docs/concepts/configuration/pod-overhead/


问题点：

调度一个pod时，总可用内存减去所有运行中pod require内存，还是总可用内存减去 每个pod require 内存和实际已使用内存的最大值之和？




kubernetes 文档说： “Docker is the most common container runtime used in a Kubernetes Pod, but Pods support other container runtimes as well.”【kubernetes理论上不一定基于docker容器，也可以基于其它容器技术】 https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/



问题点： 衡量进程的内存占用有很多个值， kubernetes limit 按哪个为准呢？



“The memory resource is measured in bytes. You can express memory as a plain integer or a fixed-point integer with one of these suffixes: E, P, T, G, M, K, Ei, Pi, Ti, Gi, Mi, Ki. For example, the following represent approximately the same value:
128974848, 129e6, 129M , 123Mi”
https://kubernetes.io/docs/tasks/configure-pod-container/assign-memory-resource/#memory-units
【kubernetes文档中关于内存单位的说明】


**问题**： linux top命令输出的buff/cache是什么意思？

free 在 [linux 手册中的条目](http://man7.org/linux/man-pages/man1/free.1.html)
page cache的[维基百科条目](https://en.wikipedia.org/wiki/Page_cache)

**问题**： kubernetes和cgropu对程序的内存限制，只包含resident memory吗？是否包含buff/cache?

互联网上的一篇[文章](https://srvaroa.github.io/jvm/kubernetes/memory/docker/oomkiller/2019/05/29/k8s-and-java.html)写到，文章作者在阅读了linux[内核文档](https://www.kernel.org/doc/Documentation/cgroup-v1/memory.txt)和docker文档之后，发现内核在统计容器是否达到内存限制（也就是cgroup限制）的时候，page cache 也会被计入。还提到："The page cache is used by the OS to speed up access to data in disk by keeping the most accessed pages in memory"。这个cache的含义与top, free命令输出的cache含义一致。

上面这篇文章引用了docker文档的一段内容，说多个cgroup都对某个文件进行读取，对应文件cache产生的内存开销会被这些cgroups分摊。

上面的内容说明 linux cgroup 会统计 cache 使用量，但是，cgroup使用的cache是否会被限制，是和RSS一起被限制的呢？还是独立限制的呢？
