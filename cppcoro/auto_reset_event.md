# class auto_reset_event(win version)
用windows事件的原子性避免冲突
## auto_reset_event(bool)
对于windows在构造函数中会通过windows内部函数 ::CreateEventW在windows中创建自动重置时间
如果创建失败就会抛出异常 (句柄为NULL的情况)
## ~auto_reset_event()
不做处理，通过safe_handle(对于指针的包装，特定于句柄)类实现对句柄的释放，对于事件释放句柄就意味这释放资源
## set()
调用windows系统函数通过句柄将对应的事件设置为Set, 如果失败了从系统中得到错误类型并抛出（windows对于异常的支持相较于linux系统更强）
## wait()
该线程会等待该事件发生，传入的参数会等待无穷时间，如果出现了等待成功之外情况就会抛出异常

# class auto_reset_event(none win version)
由于不使用windows原生事件 会复杂很多，需要使用临界区，情况变量。
通过condition_variable来阻塞和恢复对应的线程，通常的解锁步骤为首先获取一个对应的锁，在拥有锁时修改共享变量（可以是普通变量或者是原子变量）,调用notify_one 或者 notify_all通知阻塞线程恢复（可以在锁范围外进行）
对于等待的流程也是类似，先获取锁，循环等待设置变量是否被设置，没有就调用condition_variable的wait，在退出时将变量修改为非设置
