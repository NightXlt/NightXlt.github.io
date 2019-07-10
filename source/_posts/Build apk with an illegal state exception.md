title: android studio build成功，build app时报illegalStateException
date: 2019-1-3
tags: [Android]
categories: bug
description: build apk时报illegal state exception
---

### Error
```java
Error:beta<build>|]
Error:java.lang.RuntimeException: com.android.build.api.transform.TransformException: java.lang.IllegalStateException: java.lang.IllegalStateException
Error:java.lang.IllegalStateException: java.lang.IllegalStateException
Error:com.android.build.api.transform.TransformException: java.lang.IllegalStateException: java.lang.IllegalStateException
Error:java.lang.IllegalStateException
```
### 日志信息

```gradle
org.gradle.api.tasks.TaskExecutionException: Execution failed for task ':AnkiDroid:transformClassesWithInstantRunForDebug'.
	at org.gradle.api.internal.tasks.execution.ExecuteActionsTaskExecuter.executeActions(ExecuteActionsTaskExecuter.java:110)
	at org.gradle.api.internal.tasks.execution.ExecuteActionsTaskExecuter.execute(ExecuteActionsTaskExecuter.java:77)
	at org.gradle.api.internal.tasks.execution.OutputDirectoryCreatingTaskExecuter.execute(OutputDirectoryCreatingTaskExecuter.java:51)
	at org.gradle.api.internal.tasks.execution.SkipUpToDateTaskExecuter.execute(SkipUpToDateTaskExecuter.java:59)
	at org.gradle.api.internal.tasks.execution.ResolveTaskOutputCachingStateExecuter.execute(ResolveTaskOutputCachingStateExecuter.java:54)
	at org.gradle.api.internal.tasks.execution.ValidatingTaskExecuter.execute(ValidatingTaskExecuter.java:59)
	at org.gradle.api.internal.tasks.execution.SkipEmptySourceFilesTaskExecuter.execute(SkipEmptySourceFilesTaskExecuter.java:101)
	at org.gradle.api.internal.tasks.execution.FinalizeInputFilePropertiesTaskExecuter.execute(FinalizeInputFilePropertiesTaskExecuter.java:44)
	at org.gradle.api.internal.tasks.execution.CleanupStaleOutputsExecuter.execute(CleanupStaleOutputsExecuter.java:91)
	at org.gradle.api.internal.tasks.execution.ResolveTaskArtifactStateTaskExecuter.execute(ResolveTaskArtifactStateTaskExecuter.java:62)
	at org.gradle.api.internal.tasks.execution.SkipTaskWithNoActionsExecuter.execute(SkipTaskWithNoActionsExecuter.java:59)
	at org.gradle.api.internal.tasks.execution.SkipOnlyIfTaskExecuter.execute(SkipOnlyIfTaskExecuter.java:54)
	at org.gradle.api.internal.tasks.execution.ExecuteAtMostOnceTaskExecuter.execute(ExecuteAtMostOnceTaskExecuter.java:43)
	at org.gradle.api.internal.tasks.execution.CatchExceptionTaskExecuter.execute(CatchExceptionTaskExecuter.java:34)
	at org.gradle.api.internal.tasks.execution.EventFiringTaskExecuter$1.run(EventFiringTaskExecuter.java:51)
	at org.gradle.internal.operations.DefaultBuildOperationExecutor$RunnableBuildOperationWorker.execute(DefaultBuildOperationExecutor.java:300)
	at org.gradle.internal.operations.DefaultBuildOperationExecutor$RunnableBuildOperationWorker.execute(DefaultBuildOperationExecutor.java:292)
	at org.gradle.internal.operations.DefaultBuildOperationExecutor.execute(DefaultBuildOperationExecutor.java:174)
	at org.gradle.internal.operations.DefaultBuildOperationExecutor.run(DefaultBuildOperationExecutor.java:90)
	at org.gradle.internal.operations.DelegatingBuildOperationExecutor.run(DelegatingBuildOperationExecutor.java:31)
	at org.gradle.api.internal.tasks.execution.EventFiringTaskExecuter.execute(EventFiringTaskExecuter.java:46)
	at org.gradle.execution.taskgraph.LocalTaskInfoExecutor.execute(LocalTaskInfoExecutor.java:42)
	at org.gradle.execution.taskgraph.DefaultTaskExecutionGraph$BuildOperationAwareWorkItemExecutor.execute(DefaultTaskExecutionGraph.java:277)
	at org.gradle.execution.taskgraph.DefaultTaskExecutionGraph$BuildOperationAwareWorkItemExecutor.execute(DefaultTaskExecutionGraph.java:262)
	at org.gradle.execution.taskgraph.DefaultTaskPlanExecutor$ExecutorWorker$1.execute(DefaultTaskPlanExecutor.java:135)
	at org.gradle.execution.taskgraph.DefaultTaskPlanExecutor$ExecutorWorker$1.execute(DefaultTaskPlanExecutor.java:130)
	at org.gradle.execution.taskgraph.DefaultTaskPlanExecutor$ExecutorWorker.execute(DefaultTaskPlanExecutor.java:200)
	at org.gradle.execution.taskgraph.DefaultTaskPlanExecutor$ExecutorWorker.executeWithWork(DefaultTaskPlanExecutor.java:191)
	at org.gradle.execution.taskgraph.DefaultTaskPlanExecutor$ExecutorWorker.run(DefaultTaskPlanExecutor.java:130)
	at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
	at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
	at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
	at java.lang.Thread.run(Thread.java:745)
Caused by: java.lang.RuntimeException: com.android.build.api.transform.TransformException: java.lang.IllegalStateException: java.lang.IllegalStateException
	at com.android.builder.profile.Recorder$Block.handleException(Recorder.java:55)
	at com.android.builder.profile.ThreadRecorder.record(ThreadRecorder.java:104)
	at com.android.build.gradle.internal.pipeline.TransformTask.transform(TransformTask.java:230)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:497)
	at org.gradle.internal.reflect.JavaMethod.invoke(JavaMethod.java:73)
	at org.gradle.api.internal.project.taskfactory.IncrementalTaskAction.doExecute(IncrementalTaskAction.java:50)
	at org.gradle.api.internal.project.taskfactory.StandardTaskAction.execute(StandardTaskAction.java:39)
	at org.gradle.api.internal.project.taskfactory.StandardTaskAction.execute(StandardTaskAction.java:26)
	at org.gradle.api.internal.tasks.execution.ExecuteActionsTaskExecuter$1.run(ExecuteActionsTaskExecuter.java:131)
	at org.gradle.internal.operations.DefaultBuildOperationExecutor$RunnableBuildOperationWorker.execute(DefaultBuildOperationExecutor.java:300)
	at org.gradle.internal.operations.DefaultBuildOperationExecutor$RunnableBuildOperationWorker.execute(DefaultBuildOperationExecutor.java:292)
	at org.gradle.internal.operations.DefaultBuildOperationExecutor.execute(DefaultBuildOperationExecutor.java:174)
	at org.gradle.internal.operations.DefaultBuildOperationExecutor.run(DefaultBuildOperationExecutor.java:90)
	at org.gradle.internal.operations.DelegatingBuildOperationExecutor.run(DelegatingBuildOperationExecutor.java:31)
	at org.gradle.api.internal.tasks.execution.ExecuteActionsTaskExecuter.executeAction(ExecuteActionsTaskExecuter.java:120)
	at org.gradle.api.internal.tasks.execution.ExecuteActionsTaskExecuter.executeActions(ExecuteActionsTaskExecuter.java:99)
	... 34 more
Caused by: com.android.build.api.transform.TransformException: java.lang.IllegalStateException: java.lang.IllegalStateException
	at com.android.build.gradle.internal.transforms.InstantRunTransform.doTransform(InstantRunTransform.java:320)
	at com.android.build.gradle.internal.transforms.InstantRunTransform.transform(InstantRunTransform.java:186)
	at com.android.build.gradle.internal.pipeline.TransformTask$2.call(TransformTask.java:239)
	at com.android.build.gradle.internal.pipeline.TransformTask$2.call(TransformTask.java:235)
	at com.android.builder.profile.ThreadRecorder.record(ThreadRecorder.java:102)
	... 51 more
Caused by: java.lang.IllegalStateException: java.lang.IllegalStateException
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:422)
	at java.util.concurrent.ForkJoinTask.getThrowableException(ForkJoinTask.java:593)
	at java.util.concurrent.ForkJoinTask.reportException(ForkJoinTask.java:677)
	at java.util.concurrent.ForkJoinTask.join(ForkJoinTask.java:720)
	at com.android.ide.common.internal.WaitableExecutor.waitForTasksWithQuickFail(WaitableExecutor.java:146)
	at com.android.build.gradle.internal.transforms.InstantRunTransform.doTransform(InstantRunTransform.java:315)
	... 55 more
Caused by: java.lang.IllegalStateException
	at org.objectweb.asm.tree.analysis.BasicInterpreter.<init>(BasicInterpreter.java:66)
	at com.android.build.gradle.internal.incremental.ConstructorBuilder$1.<init>(ConstructorBuilder.java:127)
	at com.android.build.gradle.internal.incremental.ConstructorBuilder.build(ConstructorBuilder.java:127)
	at com.android.build.gradle.internal.incremental.IncrementalSupportVisitor.visitMethod(IncrementalSupportVisitor.java:223)
	at org.objectweb.asm.ClassVisitor.visitMethod(ClassVisitor.java:327)
	at org.objectweb.asm.commons.SerialVersionUIDAdder.visitMethod(SerialVersionUIDAdder.java:236)
	at org.objectweb.asm.tree.MethodNode.accept(MethodNode.java:686)
	at org.objectweb.asm.tree.ClassNode.accept(ClassNode.java:436)
	at com.android.build.gradle.internal.incremental.IncrementalVisitor.instrumentClass(IncrementalVisitor.java:365)
	at com.android.build.gradle.internal.transforms.InstantRunTransform.transformToClasses2Format(InstantRunTransform.java:414)
	at com.android.build.gradle.internal.transforms.InstantRunTransform.lambda$doTransform$4(InstantRunTransform.java:276)
	at com.android.build.gradle.internal.transforms.InstantRunTransform.lambda$null$5(InstantRunTransform.java:305)
	at java.util.concurrent.ForkJoinTask$AdaptedCallable.exec(ForkJoinTask.java:1424)
	at java.util.concurrent.ForkJoinTask.doExec(ForkJoinTask.java:289)
	at java.util.concurrent.ForkJoinPool$WorkQueue.runTask(ForkJoinPool.java:1056)
	at java.util.concurrent.ForkJoinPool.runWorker(ForkJoinPool.java:1692)
	at java.util.concurrent.ForkJoinWorkerThread.run(ForkJoinWorkerThread.java:157)
```
### 解决方案
　　关闭Instant run(Settings->Build->Instant Run).再重新安装即可成功。