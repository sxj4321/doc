最近做一个项目需要用到java事件模型，期间遇到了一些问题，这里写出来和大家分享一下。

#### 需求

公司需要做一个SDK供各个系统调用，其中有一个需求，基础系统需要在特定的时间通知各个系统去进行某些业务操作，类似定时任务。我的目标是写一个抽象方法，各个业务系统集成SDK内的抽象类，实现抽象方法，在抽象方法内写自己的业务实现。因为无法知道有多少子类，所以想到了事件模型。

#### 实现

因为网上有很多这种例子，我就简单的说明一下直接上代码了。

首先我们需要实现一个事件状态对象，对象里有事件源对象属性。
```java
public class TaskEvent {

    private String taskId;

    private String taskResultId;

    //事件源对象
    private Object source;

    /**
     * 构造方法，必须要传入一个source事件源.
     */
    public TaskEvent(Object source,String taskId,String taskResultId) {
        this.source = source;
        this.taskId = taskId;
        this.taskResultId = taskResultId;
    }

    public String getTaskId() {
        return taskId;
    }

    public void setTaskId(String taskId) {
        this.taskId = taskId;
    }

    public String getTaskResultId() {
        return taskResultId;
    }

    public void setTaskResultId(String taskResultId) {
        this.taskResultId = taskResultId;
    }
}
```

再定义一个事件源
```java
public class TaskSource {

    private final Set<AbstractTaskListener> listeners = new HashSet<AbstractTaskListener>();

    //注册监听者
    public void addTaskListener(AbstractTaskListener listener) {
        listeners.add(listener);
    }

    //移除监听者
    public void removeTaskListener(AbstractTaskListener listener) {
        listeners.remove(listener);
    }
    
    //事件的具体操作
    public void doTask(String taskId,String taskResultId) {
        if (listeners == null)
            return;
        TaskEvent event = new TaskEvent(this, taskId,taskResultId);
        notifyListeners(event);
    }

    //通知注册列表里的监听者
    private void notifyListeners(TaskEvent event) {
        for (AbstractTaskListener taskListener:listeners){
            if (taskListener.getTaskId().equals(event.getTaskId())){
                taskListener.excute(event);
            }

        }
    }
```

再定义一个监听接口
```java
public interface TaskListener {
    public void excute(TaskEvent event);
}
```

最后定义一个抽象类来实现监听接口
```java
public abstract class AbstractTaskListener implements TaskListener {

    private String taskId;

    public String getTaskId() {
        return taskId;
    }

    public void setTaskId(String taskId) {
        this.taskId = taskId;
    }
 
    //实现接口方法
    public void excute(TaskEvent event) {
        doTask(event);
    }

    //抽象方法，由子类来实现
    public abstract void doTask(TaskEvent event);
}
```

我们写两个测试类
```java
public class Test1 extends AbstractTaskListener {
    public void doTask(TaskEvent event) {
        System.out.println("test1 get");
    }
}

public class Test2 extends AbstractTaskListener {
    public void doTask(TaskEvent event) {
        System.out.println("test2 get");
    }
}
```

执行main方法测试一下
```java
public class TestRun {
    public static void main(String[] args) {
        TaskSource taskSource = new TaskSource();
        
        //设置ID
        Test1 test1 = new Test1();
        test1.setTaskId("1");
        //设置ID
        Test2 test2 = new Test2();
        test2.setTaskId("2");
        //注册监听者
        taskSource.addTaskListener(test2);
        taskSource.addTaskListener(test1);
        
        //修改事件源
        taskSource.doTask("1","1");
    }
}
```

执行main方法，控制台输出：test1 get，测试成功！

#### 问题

但是大家有没有发现有个问题，如果执行taskSource.doTask("1","1");的同时有一个类在修改或者删除监听者列表，那我通过for循环去读取HashSet的值的时候就会报ConcurrentModificationException。

解决这个问题其实有个办法，每次我需要for之前，我先把HashSet里的值复制到另一个Set里，类似一个副本，修改代码：
```java
//通知注册列表里的监听者
    private void notifyListeners(TaskEvent event) {
        Set<AbstractTaskListener> copyBak = new HashSet<AbstractTaskListener>(listeners);
        for (AbstractTaskListener taskListener:copyBak){
            if (taskListener.getTaskId().equals(event.getTaskId())){
                taskListener.excute(event);
            }

        }
    }
```

问题又来了，如果有多个线程来操作notifyListeners方法，我的copyBak对象是不是会有问题？我们常规的思维，加一个synchronized就行了啊，修改代码：
```java
//通知注册列表里的监听者
    private synchronized void notifyListeners(TaskEvent event) {
        Set<AbstractTaskListener> copyBak = new HashSet<AbstractTaskListener>(listeners);
        for (AbstractTaskListener taskListener:copyBak){
            if (taskListener.getTaskId().equals(event.getTaskId())){
                taskListener.excute(event);
            }

        }
    }
```

我们试想一下这种场景：线程A改变了TaskEvent的taskId值，在向各个监听者通知这个值的时候，线程B访问taskId，然后被阻塞。如果线程B持有了一个对象的同步锁，这个对象又是关于taskId的，并且本来是要通知给众多监听者当中的某一个的，这种情况下我们就会遇到一个死锁。 

那我们就缩小同步状态来解决这个问题：
```java
//通知注册列表里的监听者
    private synchronized void notifyListeners(TaskEvent event) {
        Set<AbstractTaskListener> copyBak；
        synchronized( listeners ) {  
            copyBak = new HashSet<AbstractTaskListener>(listeners);
        }
        for (AbstractTaskListener taskListener:copyBak){
            if (taskListener.getTaskId().equals(event.getTaskId())){
                taskListener.excute(event);
            }

        }
    }
```

这样的代码是不是太不优雅了，我们程序员追求的就是优雅的完成代码，我们可以使用concurrent包里的CopyOnWriteArraySet来实现我们上面的功能。因为CopyOnWriteArraySet是线程安全的，并且CopyOnWriteArraySet和CopyOnWriteArrayList是被设计专门用来在这种场景中使用的，CopyOnWriteArraySet和CopyOnWriteArrayList内置了一个查询副本。
```java
public class TaskSource {

    private final Set<AbstractTaskListener> listeners = new CopyOnWriteArraySet<AbstractTaskListener>();

    //注册监听者
    public void addTaskListener(AbstractTaskListener listener) {
        listeners.add(listener);
    }

    //移除监听者
    public void removeTaskListener(AbstractTaskListener listener) {
        listeners.remove(listener);
    }
    
    //事件的具体操作
    public void doTask(String taskId,String taskResultId) {
        if (listeners == null)
            return;
        TaskEvent event = new TaskEvent(this, taskId,taskResultId);
        notifyListeners(event);
    }

    //通知注册列表里的监听者
    private void notifyListeners(TaskEvent event) {
        for (AbstractTaskListener taskListener:listeners){
            if (taskListener.getTaskId().equals(event.getTaskId())){
                taskListener.excute(event);
            }

        }
    }
}
```
