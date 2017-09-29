## 1.LinkedList源码分析
LinkedList的是基于链表实现的java集合类，通过index插入到指定位置的时候使用LinkedList效率要比ArrayList高，以下源码分析是基于JDK1.8.
### 1.1 类的继承结构
LinkedList类的继承结构如如下所示：
![]()
从以上继承结构图中可以看到，最顶层接口为Iterable接口，实现此接口的类支持通过迭代器遍历集合中的元素。其他相关接口如Collection、List、Queue、Deque分别为列表，队列功能的相关接口，即此LinkedList支持列表、队列的相关操作。Serializable接口为序列化标志接口，即所有需要序列化的类都需要实现此接口。

### 1.2 LinkedList数据结构说明
首先我们来看下LinkedList中保存数据的内部类定义源码如下：

    private static class Node<E> {
            E item;
            Node<E> next;
            Node<E> prev;

            Node(Node<E> prev, E element, Node<E> next) {
                this.item = element;
                this.next = next;
                this.prev = prev;
            }
        }

通过以上定义可以看出来每个节点中除了保存元素的值之外还保存了前后节点的引用。即使用了链表的数据接口保存数据元素。数据结构如下图所示：
![11]()
在LinkedList类中，只是定义了两个Node类型的属性，定义如下：

     transient Node<E> first;
     transient Node<E> last;
这两个属性分别表示头节点和尾节点，始终指向链表的第一个节点和最后一个节点。

### 1.3 关键方法源码分析
当不指定index往LinkedList中添加元素的时候默认是在链表的尾部添加数据，源代码如下

      void linkLast(E e) {
          //保存链表插入之前的最后一个节点
          final Node<E> l = last;
          //将新节点的preNode指向链表最后一个节点
          final Node<E> newNode = new Node<>(l, e, null);
          last指向插入后的尾节点
          last = newNode;
          if (l == null)
              //当第一次插入数据的时候，头节点和尾节点指向同一个节点
              first = newNode;
          else
              //插入之前的尾节点的nextNode指向新插入的节点
              l.next = newNode;
          size++;
          modCount++;
      }
插入节点的详细操作如上面代码注释。在次操作中其实last即尾节点是共享资源，当多个线程同时执行此方法的时候，其实会出现线程安全问题。具体的线程安全问题后面分析。

当指定index插入数据的时候，源代码如下所示：

    public void add(int index, E element) {
           checkPositionIndex(index);

           if (index == size)
               linkLast(element);
           else
               linkBefore(element, node(index));
       }
在指定index插入元素的时候会先查询出index位置的元素，然后调用linkBefore将此元素插入到查询到的元素的前一个位置。其中linkBefore源码如下所示：

    void linkBefore(E e, Node<E> succ) {
           // assert succ != null;
           final Node<E> pred = succ.prev;
           //新建节点，并将preNode指向idnex-1节点
           final Node<E> newNode = new Node<>(pred, e, succ);
           //插入前的index节点的prev指向新节点
           succ.prev = newNode;
           if (pred == null)
               first = newNode;
           else
              //index-1节点的nextNode指向新节点
               pred.next = newNode;
           size++;
           modCount++;
       }
在指定节点插入的方式如上注释所示。此操作中共享资源是插入之前index节点。同样会出现并发安全问题，下面对此问题进行分析。
## 2.LinkedList并发插入时节点覆盖的问题
在指定index插入或者addLast的时候都是在链表的尾部插入数据，当并发插入的时候如果出现以下执行顺序的时候，会出现插入的数据被覆盖的问题。
>当多个线程同时获取到相同的尾节点的时候，然后多个线程同时在此尾节点后面插入数据的时候会出现数据覆盖的问题。
