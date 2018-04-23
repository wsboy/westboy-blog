# Tomcat中使用到的设计模式
* 模板方法模式



```java
public abstract class LifecycleBase implements Lifecycle {
    
    protected abstract void initInternal() throws LifecycleException;
    
    @Override
    public final synchronized void init() throws LifecycleException {
        if (!state.equals(LifecycleState.NEW)) {
            invalidTransition(Lifecycle.BEFORE_INIT_EVENT);
        }

        try {
            setStateInternal(LifecycleState.INITIALIZING, null, false);
            //使用模板方法具体执行初始化
            initInternal();
            setStateInternal(LifecycleState.INITIALIZED, null, false);
        } catch (Throwable t) {
            handleSubClassException(t, "lifecycleBase.initFail", toString());
        }
    }
}
```

# Tomcat的顶层结构及启动过程
# Tomcat的生命周期管理
# Container分析
# Pipeline-Value管道
# Connector分析