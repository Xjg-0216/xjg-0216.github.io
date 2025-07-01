# 十二 多线程与线程池

多线程（Multithreading）是指在一个进程（Process）中同时执行多个线程（Thread），以实现并发处理任务，从而提高程序的执行效率，特别适合处理**IO密集型或部分CPU密集型**任务。


## 1. 线程 vs 进程

| 比较项  | 进程（Process）        | 线程（Thread）       |
| ---- | ------------------ | ---------------- |
| 定义   | 资源分配的基本单位          | CPU调度的基本单位       |
| 拥有资源 | 独立的内存空间等资源         | 与同一进程中的其他线程共享资源  |
| 通信开销 | 进程间通信（如管道、消息队列）较复杂 | 线程间通信简单，因共享内存    |
| 创建销毁 | 开销大                | 开销小              |
| 崩溃影响 | 一个进程崩溃不影响其他进程      | 一个线程崩溃可能导致整个进程崩溃 |



## 2. C++多线程

### 2.1 引入头文件

```cpp
#include <thread>
```

### 2.2 基本使用

```cpp
#include <iostream>
#include <thread>

void task() {
    std::cout << "线程ID: " << std::this_thread::get_id() << "\n";
}

int main() {
    std::thread t(task);  // 创建线程
    t.join();             // 等待线程执行完成
    return 0;
}
```

```cpp
#include <iostream>
#include <thread>
using namespace std;

void thread_1()
{
    cout<<"子线程1"<<endl;
}

void thread_2(int x)
{
    cout<<"x:"<<x<<endl;
    cout<<"子线程2"<<endl;
}

int main()
{
  thread first ( thread_1); // 开启线程，调用：thread_1()
  thread second (thread_2,100); // 开启线程，调用：thread_2(100)
  //thread third(thread_2,3);//开启第3个线程，共享thread_2函数。
  std::cout << "主线程\n";

  first.join(); //必须说明添加线程的方式            
  second.join(); 
  std::cout << "子线程结束.\n";//必须join完成
  return 0;
}
```

## 3. 线程常用方法

| 方法                              | 说明                |
| ------------------------------- | ----------------- |
| `join()`                        | 阻塞主线程，等待该线程完成     |
| `detach()`                      | 线程脱离，后台运行，主线程无需等待 |
| `std::this_thread::sleep_for()` | 当前线程睡眠一段时间        |
| `std::this_thread::get_id()`    | 获取当前线程ID          |

### 3.1 join与detach

当线程启动后，一定要在和线程相关联的thread销毁前，确定以何种方式等待线程执行结束。比如上例中的join。

- detach方式，启动的线程自主在后台运行，当前的代码继续往下执行，不等待新线程结束
- join方式，等待启动的线程完成，才会继续往下执行

可以使用joinable判断是join模式还是detach模式。

`if (myThread.joinable()) foo.join();`

#### 3.1.1 join举例

下面的代码，join后面的代码不会被执行，除非子线程结束
```cpp
#include <iostream>
#include <thread>
using namespace std;
void thread_1()
{
  while(1)
  {
  //cout<<"子线程1111"<<endl;
  }
}
void thread_2(int x)
{
  while(1)
  {
  //cout<<"子线程2222"<<endl;
  }
}
int main()
{
    thread first ( thread_1); // 开启线程，调用：thread_1()
    thread second (thread_2,100); // 开启线程，调用：thread_2(100)

    first.join(); // pauses until first finishes 这个操作完了之后才能destroyed
    second.join(); // pauses until second finishes//join完了之后，才能往下执行。
    while(1)
    {
      std::cout << "主线程\n";
    }
    return 0;
}
```
#### 3.1.2 detach举例

下列代码中，主线程不会等待子线程结束。如果主线程运行结束，程序则结束
```cpp
#include <iostream>
#include <thread>
using namespace std;

void thread_1()
{
  while(1)
  {
      cout<<"子线程1111"<<endl;
  }
}

void thread_2(int x)
{
    while(1)
    {
        cout<<"子线程2222"<<endl;
    }
}

int main()
{
    thread first ( thread_1);  // 开启线程，调用：thread_1()
    thread second (thread_2,100); // 开启线程，调用：thread_2(100)

    first.detach();                
    second.detach();            
    for(int i = 0; i < 10; i++)
    {
        std::cout << "主线程\n";
    }
    return 0;
}
```
#### 3.2 this_thread

`std::this_thread` 提供了对 当前线程的控制能力，比如让线程休眠、获取线程 ID、主动让出执行权等

| 函数                                | 作用                   |
| --------------------------------- | -------------------- |
| `std::this_thread::sleep_for()`   | 当前线程**睡一段时间**（持续时间）  |
| `std::this_thread::sleep_until()` | 当前线程**睡到某个时刻**       |
| `std::this_thread::yield()`       | 当前线程**主动让出 CPU 执行权** |
| `std::this_thread::get_id()`      | 获取当前线程的 **唯一 ID**    |

**sleep_for**
```cpp
#include <iostream>
#include <thread>
#include <chrono>

void task() {
    std::cout << "任务开始..." << std::endl;
    std::this_thread::sleep_for(std::chrono::seconds(2)); // 睡 2 秒
    std::cout << "任务结束！" << std::endl;
}

int main() {
    std::thread t(task);
    t.join();
    return 0;
}

```

**获取线程ID**
```cpp
#include <iostream>
#include <thread>

void show_thread_id() {
    std::cout << "当前线程 ID: " << std::this_thread::get_id() << std::endl;
}

int main() {
    std::thread t1(show_thread_id);
    std::thread t2(show_thread_id);
    t1.join();
    t2.join();
    return 0;
}

```

**yield() 主动让出 CPU 时间片**

```cpp
#include <iostream>
#include <thread>

void busy_task() {
    for (int i = 0; i < 5; ++i) {
        std::cout << "任务正在执行：" << i << std::endl;
        std::this_thread::yield(); // 让出CPU，让其他线程执行
    }
}

int main() {
    std::thread t(busy_task);
    t.join();
    return 0;
}

```
`sleep_for` 和 `sleep_until`依赖系统的计时精度。

`yield()` 是一种优化建议，系统可能让当前线程暂停，也可能忽略（不是强制让出）


## 4. 线程安全与同步

多线程之间**共享资源**可能导致数据竞争（data race）和**不确定行为**，为此需引入**同步机制**：


| 工具                        | 作用                 |
| ------------------------- | ------------------ |
| `std::mutex`              | 互斥锁，防止多个线程同时访问同一资源 |
| `std::lock_guard`         | 自动加锁解锁，RAII风格      |
| `std::unique_lock`        | 更灵活的锁管理            |
| `std::condition_variable` | 条件变量，用于线程等待和唤醒     |

### 4.1 mutex

`mutex`（互斥锁，全称 mutual exclusion）是多线程编程中用来 保护共享资源，避免多个线程同时访问而造成冲突或数据不一致的一种机制

- 互斥锁：在某段代码执行前加锁（lock），执行完毕后解锁（unlock）。
- 加锁成功的线程可以继续执行，其他线程会阻塞，直到该锁被释放。

```cpp
#include <iostream>
#include <thread>
#include <mutex>

std::mutex mtx;  // 全局互斥锁
int counter = 0;

void increase() {
    for (int i = 0; i < 10000; ++i) {
        std::lock_guard<std::mutex> lock(mtx); // 自动加锁和解锁
        ++counter;
    }
}

int main() {
    std::thread t1(increase);
    std::thread t2(increase);

    t1.join();
    t2.join();

    std::cout << "Final counter: " << counter << std::endl;
    return 0;
}
```

`std::mutex mtx`：声明一个互斥锁。

`std::lock_guard<std::mutex>`：RAII方式自动加锁和解锁，防止因异常等导致忘记解锁。


| 用法                 | 说明             |
| ------------------ | -------------- |
| `mutex.lock()`     | 手动加锁（不推荐）      |
| `mutex.unlock()`   | 手动解锁（不推荐）      |
| `std::lock_guard`  | 推荐方式，自动加解锁     |
| `std::unique_lock` | 更灵活，可延迟加锁、提前解锁 |
| `std::try_lock`    | 非阻塞加锁，锁失败立即返回  |

### 4.2 condition_variable

`std::condition_variable`允许一个线程在某个条件满足之前进入等待状态，而由另一个线程在条件满足时通知它继续执行。
常用于 生产者-消费者模型、任务调度 等场景。

- **`std::condition_variable`**：条件变量对象
    
- **`std::mutex`**：互斥锁，用于保护共享数据
    
- **`wait()`**：阻塞当前线程，直到条件满足
    
- **`notify_one()` / `notify_all()`**：唤醒一个或所有等待线程

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>

std::mutex mtx;
std::condition_variable cv;
bool ready = false;

void worker() {
    std::unique_lock<std::mutex> lock(mtx);  // 一定要用 unique_lock
    cv.wait(lock, []{ return ready; });      // 条件不满足就一直阻塞
    std::cout << "Worker is running!\n";
}

int main() {
    std::thread t(worker);

    {
        std::lock_guard<std::mutex> lock(mtx);
        ready = true;           // 设置条件
    }

    cv.notify_one();            // 通知等待的线程
    t.join();
    return 0;
}

```

`wait`的三种形式

```cpp
cv.wait(lock);  // 等待 notify，被唤醒后要手动检查条件是否满足（可能虚假唤醒）

cv.wait(lock, [] { return ready; });  // 推荐写法，自动循环判断条件

cv.wait_for(lock, timeout);  // 等待一段时间（超时自动返回）

```

**示例：生产者消费者模型**
```cpp
std::queue<int> data_queue;
std::mutex data_mutex;
std::condition_variable data_cv;

void producer() {
    for (int i = 0; i < 5; ++i) {
        {
            std::lock_guard<std::mutex> lock(data_mutex);
            data_queue.push(i);
        }
        data_cv.notify_one();  // 通知消费者
        std::this_thread::sleep_for(std::chrono::milliseconds(100));
    }
}

void consumer() {
    while (true) {
        std::unique_lock<std::mutex> lock(data_mutex);
        data_cv.wait(lock, []{ return !data_queue.empty(); }); // 等待非空
        int val = data_queue.front();
        data_queue.pop();
        lock.unlock();
        std::cout << "Consumed: " << val << std::endl;
        if (val == 4) break;
    }
}
```

| 注意点                                | 说明                                        |
| ---------------------------------- | ----------------------------------------- |
| `wait()` 要配合 `std::unique_lock` 使用 | 因为它需要释放锁并在唤醒时重新加锁                         |
| 要防止 **虚假唤醒**                       | `wait(lock, predicate)` 会自动循环判断，推荐使用      |
| 条件变量本身不保存状态                        | `notify_one()` 不会“记住”曾经调用过，如果此时没人等待，通知会丢失 |
| 避免死锁                               | 修改条件时应先加锁，通知时应解锁或在锁内快速通知                  |


## 5. 线程池

如果你频繁地创建和销毁线程，会带来性能开销，因此可以使用**线程池**（Thread Pool）：
一般线程池都会有以下几个部分构成：

线程池管理器（ThreadPoolManager）:用于创建并管理线程池，也就是线程池类
工作线程（WorkThread）: 线程池中线程
任务队列task: 用于存放没有处理的任务。提供一种缓冲机制。
append：用于添加任务的接口




