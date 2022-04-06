## System-V IPC信号量

### 进程信号量概念

- 信号量与已经介绍过的信号、管道、FIFO以及消息列队不同， 它本质上是一个计数器，用于协调多进程间对共享数据对象的读取，它不以传送数据为主要目的， 它主要是用来保护共享资源，使得该临界资源在一个时刻只有一个进程独享

- 信号的操作 PV操作

  - P操作：如果有可用的资源，占用一个资源，并且信号量值-1，并进入临界代码区（操作临界资源的代码）

    ​			  如果没有可用的资源，则阻塞等待，知道系统将资源分配给该进程

  - V操作：如果此时信号量的等待队列中有进程等待该资源，则唤醒一个阻塞的进程；

    ​		      如果没有进程等待，则释放资源，并且信号量+1

### 信号量创建函数——semget()

`int semget(key_t key,int nsems,int semflg);`

- key:与消息队列一样的是，参数key用来标识系统内的信号量， 如果指定的key已经存在，则意味着打开这个信号量，这时nsems参数指定为0，semflg参数也指定为0。          
- nsems:创建信号量的时候，表示可用的信号量数目
- semflg:指定标志位，主要有IPC_CREAT,IPC_EXCL和权限mode

 					IPC_CREAT标志创建新的信号量，如果该信号量已经存在（具有同一个键值的信号量已经存在在系统中）也不会出错

​					 如果同时指定IPC_EXCL标志，可以创建一个新的唯一的信号量，如果信号量已经存在，则函数返回错误

## 信号量操作函数——semop()

`int semop(int semid,struct sembuf *sops,size_t nsops);`

- semid：System V信号量的标识符，标识一个信号量。

- sops： 指向一个struct sembuf的结构体指针，该数组是一个信号量操作数组

  struct sembuf{

  ​	usigned short int sem_num;   ***信号量的序号 从0~nsem-1***

  ​	short int sem_op;					***对信号量的操作， >0,=0,<0***

  ​	short int sem_flg;					***操作标识：0,IPC_WAIT,SEM_UNDO***

  }

- nsops:表示上面sops数组的数量，如果只有一个数组，nsops就设定为1

## semctl()属性函数

主要是对信号量集进行一系列的操作，根据cmd命令的不同，执行不同的操作，第四个参数可选

`int semctl(int semid,int semnum,int cmd,...);`

- semid： 同上
- semnum：表示信号量集中的第sumnum个信号量，取值范围 0~nsem-1
- cmd操作：
  - IPC_RMID：从系统中删除该信号量集合。
  - IPC_STAT：获取此信号量集合的semid_ds结构，存放在第四个参数的buf中。
  - SETVAL：设置第semnum个信号量的值，该值由第四个参数中的val指定。
- 第四个参数是可选的：如果使用该参数，该参数的类型是union semun，它是多个特定命令的联合体

​		union semun{

​			int				          val;  ***Value for SETVAl***

​			struct semid_ds  *buf;   ***Buffer  for IPC_STAT,IPC_SET***

​			unsigned short   * array； ***Array for GETVAL,SETALL***

​			struct seminfo   *__buf;      ***Buffer for IPC_INFO(Linux-specific)***

​		}