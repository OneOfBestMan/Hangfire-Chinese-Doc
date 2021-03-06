在后台中调用方法
=============================

Fire-and-forget 的调用方法极其简单。正如您从 :doc:`快速开始<../quick-start>` 一节中了解到，您只需要传递一个具有相应方法和参数的lambda表达式：

.. code-block:: c#

  BackgroundJob.Enqueue(() => Console.WriteLine("Hello, world!"));

``Enqueue`` 方法不会立即调用目标方法，而是运行以下步骤：

1. 序列化目标方法及其所有参数。
2. 根据序列化的信息创建一个新的后台任务。
3. 将后台任务保存到持久化存储。
4. 将后台任务入队。

执行这些步骤后， ``BackgroundJob.Enqueue`` 方法立即返回结果。轮到另一个Hangfire组件，:doc:`Hangfire Server <../background-processing/processing-background-jobs>` 将会从持久化存储中检查到队列中有后台任务后如期执行。

队列任务由专门的工作线程处理。每个worker将如下述流程执行任务:

1. 获取一个任务，并对其他worker隐藏该任务。
2. 执行任务及其所有的扩展过滤器。
3. 从队列中删除该任务。

因此，只有处理成功后才能删除该任务。即使一个进程在执行期间被终止，Hangfire将执行补偿逻辑来保证每个任务都被处理。

每种持久存储各有各自的步骤和补偿逻辑机制：

* **SQL Server** 使用常规SQL事务，因此在进程终止的情况下，后台作业ID立即放回队列。
* **MSMQ** 使用事务队列，因此不需要定期检查。入队后几乎立即获取作业。
* **Redis** 实现使用阻塞的 ``BRPOPLPUSH`` 命令，因此与MSMQ一样立即获取作业。但是在进程终止的情况下，只有在超时到期后（默认为30分钟）才重新排队。
