# Note
## MySQL
### 索引

什么是聚簇索引？
找到了索引就找到了需要的数据，那么这个索引就是聚簇索引，所以主键就是聚簇索引，修改聚簇索引其实就是修改主键。
什么是非聚簇索引？
索引的存储和数据的存储是分离的，也就是说找到了索引但没找到数据，需要根据索引上的值(主键)再次回表查询,非聚簇索引也叫做辅助索引。

### 事务隔离级别是怎么实现的？
1. 事务的特性 ACID
A: Atomicity 一个事务中的所有操作，要么全部完成，要么全部不完成(通过回滚日志保证)
C:Consistency 事务操作前后满足数据的完整性约束 （通过其他三者保证）
I:Isolation 多个事务并发执行 不会相互干扰 （MVCC或者锁机制保证）
D:Durability 事务执行结束，对事务的修改是永久的（通过重做日志保证）

2. 并行事务会引发什么问题？
在同时处理多个事务的时候，就可能出现脏读（dirty read）、不可重复读（non-repeatable read）、幻读（phantom read）的问题。

脏读（dirty read）：一个事务「读到」了另一个「未提交事务修改过的数据」

不可重复读（non-repeatable read）：在一个事务内多次读取同一个数据，如果出现前后两次读到的数据不一样的情况，就意味着发生了「不可重复读」现象。

假设有 A 和 B 这两个事务同时在处理，事务 A 先开始从数据库中读取小林的余额数据，然后继续执行代码逻辑处理，在这过程中如果事务 B 更新了这条数据，并提交了事务，那么当事务 A 再次读取该数据时，就会发现前后两次读到的数据是不一致的，这种现象就被称为不可重复读。

幻读（phantom read）：在一个事务内多次查询某个符合查询条件的「记录数量」，==如果出现前后两次查询到的记录数量不一样的情况==，就意味着发生了「幻读」现象。

SQL 标准提出了四种隔离级别来规避以上三种现象，隔离级别越高，性能效率就越低，这四个隔离级别如下：

- 读未提交（read uncommitted），指一个事务还没提交时，它做的变更就能被其他事务看到；
- 读提交（read committed），指一个事务提交之后，它做的变更才能被其他事务看到；
- 可重复读（repeatable read），指一个事务执行过程中看到的数据，**一直跟这个事务启动时看到的数据是一致的**，==MySQL InnoDB 引擎的默认隔离级别==；
- 串行化（serializable ）；会对记录加上读写锁，在多个事务对这条记录进行读写操作时，如果发生了读写冲突的时候，后访问的事务必须等前一个事务执行完成，才能继续执行；

这四种隔离级别具体是如何实现的呢？

对于「读未提交」隔离级别的事务来说，因为可以读到未提交事务修改的数据，所以直接读取最新的数据就好了；
对于「串行化」隔离级别的事务来说，通过加读写锁的方式来避免并行访问；
对于「读提交」和「可重复读」隔离级别的事务来说，它们是通过 Read View 来实现的，它们的区别在于创建 Read View 的时机不同，==Read View 可以理解成一个数据快照==，就像相机拍照那样，定格某一时刻的风景。「读提交」隔离级别是在「每个语句执行前」都会重新生成一个 Read View，而「可重复读」隔离级别是「启动事务时」生成一个 Read View，然后整个事务期间都在用这个 Read View。

### Read View 在 MVCC 里如何工作的？

MVCC（多版本并发控制）：通过「版本链」来控制并发事务访问同一个记录时的行为；用于「读提交」和「可重复读」隔离级别的实现


### MySQL 有哪些锁？

https://www.cnblogs.com/y2ek/p/12630007.html

https://blog.csdn.net/weixin_43093878/article/details/105858568

https://blog.csdn.net/hongchh/article/details/52914507?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-1-52914507-blog-102891495.235%5Ev38%5Epc_relevant_sort&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-1-52914507-blog-102891495.235%5Ev38%5Epc_relevant_sort&utm_relevant_index=1


https://blog.csdn.net/lukas_sun/article/details/53811087?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-1-53811087-blog-52914507.235%5Ev38%5Epc_relevant_sort&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-1-53811087-blog-52914507.235%5Ev38%5Epc_relevant_sort&utm_relevant_index=2

https://blog.csdn.net/lukas_sun/article/details/53770959

https://blog.csdn.net/pl0321/article/details/115507286


//带权区间调度问题
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;
const int MAX_N=10000;
int N,S[MAX_N],T[MAX_N],V[MAX_N];

struct Assign{
    int start,end,value;
    Assign(int s,int t,int v):start(s),end(t),value(v){}
};

Assign itv [MAX_N];
bool cmp(Assign a,Assign b){
    if(a.end==b.end) return a.value>b.value;
    return a.end<b.end;
}
// 求最多的区间调度
void solve_nums(){
    int ans=0,t=0;
    for(int i=0;i<N;++i){
        if(t<itv[i].start){
            ++ans;
            t=itv[i].end;
        }
    }
    cout<<ans;
}

//求最大的区间调度
void solve_maxLen(){
    vector<int> dp(N,0);
    dp[0]=itv[0].end-itv[0].start;
    for(int i=1;i<N;++i){
        
    }
}
int main(){

    sort(itv,itv+N,cmp);
    return 0;
}
