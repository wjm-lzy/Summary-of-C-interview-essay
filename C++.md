# 美团1
### 内存static和dynamic的区别
static（静态）​​ 和 ​​dynamic（动态）<br>
static:内存分配在编译的时候确定，大小和生命周期固定，无需运行时分配开销<br>
dynamic:内存分配在运行时动态申请和释放（new），适应不确定的需求<br>

### const 修饰的变量，可以被修改吗
可以通过引用修改<br>
实例代码如下<br>
const int x = 100;<br>
int *p = (int*)&x;//得到x的地址，通过定义指针接收<br>
*p = 200;<br>
上述代码就可以绕过const的限制<br>

### 智能指针的介绍
使用智能指针式为了自动释放资源，避免内存泄漏（忘记delete）和程序崩溃（多次delete）<br>
C++11引入了如下智能指针：包含在头文件<memory>中<br>
1.std::unique_ptr(独占所有权)<br>
  同一时间只能有一个unique_ptr拥有资源，不可复制，但支持移动语义（std::move）(补充：相信有的朋友不知道什么是移动语义，这道题结束介绍)<br>
  管理动态分配的单个对象或者数组<br>
  代码实例：<br>
    std::unique_ptr<int> p1 = std::make_unique<int>(10);<br>
    std::unique_ptr<int[]> p2 = std::make_unique<int>(5);<br>
    //转移所有权<br>
    std::unique_ptr<int> p3 = std::move(p1);<br>
2.std::shared_ptr(共享所有权)<br>
  多个shared_ptr共享一个资源，通过引用计数的方式跟踪资源所有者数量，计数为0的时候自动释放资源<br>
  // 创建 shared_ptr<br>
  std::shared_ptr<int> p1 = std::make_shared<int>(20);<br>
  std::shared_ptr<int> p2 = p1; // 引用计数 +1<br>
  需要避免两个shared_ptr互相引用，导致引用计数器无法归零，可以将其中一个指针替换为weak_ptr<br>
  // 创建两个 shared_ptr 并互相引用
    std::shared_ptr<A> a = std::make_shared<A>();<br>
    std::shared_ptr<B> b = std::make_shared<B>();<br>
    a->b_ptr = b;  // a 引用 b<br>
    b->a_ptr = a;  // b 引用 a<br>
  
 （如果你学过操作系统文件管理中的共享文件打开文件表的话，理解应该不难，本题结束补充）<br>
3.std::weak_ptr(弱引用)<br>
  不增加引用计数，不拥有资源所有权，用于解决shared_ptr的循环引用问题，经常用于转换this指针<br>
4.auto_str(已经不使用了)<br>
  所有权转移时会导致原指针变为 nullptr，容易引发错误，不需要记忆<br>
  
note:智能指针是可以手动释放的，使用reset()//释放资源并将指针置为nullptr<br>

### 这里补充一下，面试官看到这里觉得你小子很懂了？？？，那么继续向下追问：你是否能写一下智能指针的实现

1.std::unique_str<br>
  template<typename T><br>
  class UniquePtr {<br>
  private:
      T* ptr;<br>
  public:<br>
      explicit UniquePtr(T* p = nullptr) : ptr(p) {}//构造函数，为p复制<br>
      ~UniquePtr() { delete ptr; }<br>

    // 禁用复制<br>
      UniquePtr(const UniquePtr&) = delete;<br>
      UniquePtr& operator=(const UniquePtr&) = delete;<br>

    // 支持移动语义<br>
      UniquePtr(UniquePtr&& other) : ptr(other.ptr) { other.ptr = nullptr; }<br>
      UniquePtr& operator=(UniquePtr&& other) {<br>
          if (this != &other) {<br>
              delete ptr;<br>
              ptr = other.ptr;<br>
              other.ptr = nullptr;<br>
            }<br>
        return *this;<br>
      }<br>
  };<br>

2.std::shared_ptr
  这个实现比较复杂，读者可以自行查阅资料，涉及到的函数较多，但是不实现的话思想还是简单的<br>
  这里简单介绍一下：count计数私有化，随后构造函数中需要在参数列表中对其初始化，再有就是=运算符重载，实现所有权的共享，但是要注意的是这里需要实现对原有资源的释放，不然会造成内存泄露的（-_-）。<br>

### 本题花了好长时间啊，这里是之前承诺的补充

1.移动语义move<br>
  为C++11引入的核心特性，旨在转移资源所有权来提高程序性能，其实就是原指针置空，现指针指向当前资源<br>
2.操作系统的打开文件表<br>
  在操作系统中存在一个系统打开文件表，通过引用计数来跟踪共享文件的用户数量，这是实现多线程和多进程共享文件的核心机制之一<br>
  首选每个进程维护一个进程打开文件表，这个里面记录了进程打开的文件，同时表中的每个条目（文件描述符），注意打开文件最初是使用文件名去找寻文件对应的inode，找到后就可以使用文件描述符访问文件了<br>
  随后系统打开文件表中包含了所有被打开的文件，这里使用引用计数来标识有多少进程正在使用这个文件<br>


  
