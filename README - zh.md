### logger 源码阅读



#### 前言

设计模式，设计原则 ，处理的很好 ，真的大佬 ，佩服了。



### 设计模式 

1 创建Builder 的设计模式 使用 

2Adapter设计模式   一个log 多种处理 ，使用容器装载起来



### 细节处理

1静态的工具类 ，构造 private ， class final 修饰 



### 其他

1  ThreadLocal 的使用  

2  HandlerThread 的使用  



### 整体结构

```java
 interface Printer {
   void addAdapter(@NonNull LogAdapter adapter);

   Printer t(@Nullable String tag);

   void d(@NonNull String message, @Nullable Object... args);
  
   void log(int priority, @Nullable String tag, @Nullable String message, @Nullable Throwable throwable);

   void clearLogAdapters();
 }
```



```java

class LoggerPrinter implements Printer {

  /**
   * Provides one-time used tag for the log message
   */
  private final ThreadLocal<String> localTag = new ThreadLocal<>();

  private final List<LogAdapter> logAdapters = new ArrayList<>();

  @Override public Printer t(String tag) {
    if (tag != null) {
      localTag.set(tag);
    }
    return this;
  }

  @Override public void d(@NonNull String message, @Nullable Object... args) {
    log(DEBUG, null, message, args);
  }

  @Override public synchronized void log(int priority,
                                         @Nullable String tag,
                                         @Nullable String message,
                                         @Nullable Throwable throwable) {

    for (LogAdapter adapter : logAdapters) {
      if (adapter.isLoggable(priority, tag)) {
        adapter.log(priority, tag, message);
      }
    }
  }

  @Override public void clearLogAdapters() {
    logAdapters.clear();
  }

  @Override public void addAdapter(@NonNull LogAdapter adapter) {
    logAdapters.add(checkNotNull(adapter));
  }


}

```



1  Logger 调用  Printer 接口  

2 Printer 的实现类  LoggerPrinter 获取  priority, tag,  message, throwable 



```
public interface LogAdapter {

  boolean isLoggable(int priority, @Nullable String tag);

  void log(int priority, @Nullable String tag, @NonNull String message);
}
```

//  LogAdapter 调用  FormatStrategy .log 

```
public class AndroidLogAdapter implements LogAdapter {

  @NonNull private final FormatStrategy formatStrategy;

  public AndroidLogAdapter() {
    this.formatStrategy = PrettyFormatStrategy.newBuilder().build();
  }

  public AndroidLogAdapter(@NonNull FormatStrategy formatStrategy) {
    this.formatStrategy = checkNotNull(formatStrategy);
  }

  @Override public boolean isLoggable(int priority, @Nullable String tag) {
    return true;
  }

  @Override public void log(int priority, @Nullable String tag, @NonNull String message) {
    formatStrategy.log(priority, tag, message);
  }

}
```







