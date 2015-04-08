title: 高并发系统设计
date: 2015-04-08 22:51:57
categories: concurrent
tags:
---

注：本文大多数观点和代码都是从网上或者开源代码中抄来的，为了疏理和组织这片文章，作者也费了不少心血，为了表示对我劳动的尊重，请转载时注明作者和出处。
 
## 一、引子
最近失业在家，闲来无事。通过网上查找资料和查看开源代码，研究了一下互联网高并发系统的一些设计。这里主要从服务器内部设计和整个系统设计两个方面讨论，更多的是从互联网大型网站设计方面考虑，高性能计算之类系统没有研究过。<!--more-->
 
## 二、服务器内部设计
服务器设计涉及Socket的阻塞/非阻塞，操作系统IO的同步和异步（之前被人问到过两次。第一次让我说说知道的网络模型，我说ISO模型和TCP/IP模型，结果被鄙视了。最后人说了解linux epoll吗？不了解呀！汉，回去查资料才知道是这回事。第二次让我说说知道线程模型，汉！这个名词感觉没有听说过,线程？模型？半同步/半异步，领导者/跟随者知道吗。再汉，我知道同步/异步，还有半同步/半异步？啥呀？领导者/跟随者，我现在没有领导。回去一顿恶补，原来是ACE框架里边经常有这样的提法，Reactor属于同步/半同步，PREACTOR属于领导者/跟随者模式。瀑布汗。小插曲一段，这些不懂没关系，下边我慢慢分解），事件分离器，线程池等。内部设计希望通过各个模块的给出一个简单设计，经过您的进一步的组合和打磨，就可以实现一个基本的高并发服务器。
### 1. Java高并发服务器
Java设计高并发服务器相对比较简单。直接是用ServerSocket或者Channel+selector实现。前者属于同步IO设计，后者采用了模拟的异步IO。为什么说模拟的异步IO呢？记得网上看到一篇文章分析了java的selector。在windows上通过建立一个127.0.0.1到127.0.0.1的连接实现IO的异步通知。在linux上通过建立一个管道实现IO的异步通知。考虑到高并并发系统的要求和java上边的异步IO的限制（通常操作系统同时打开的文件数是有限制的）和效率问题，java的高并发服务器设计不做展开深入的分析，可以参考C高并发服务器的分析做同样的设计。
 
### 2. C高并发服务器设计
#### 1) 基本概念   
Ø  阻塞和非阻塞socket  
所谓阻塞Socket，是指其完成指定的任务之前不允许程序调用另一个函数，在Windows下还会阻塞本线程消息的发送。所谓非阻塞Socket，是指操作启动之后，如果可以立即得到结果就返回结果，否则返回表示结果需要等待的错误信息，不等待任务完成函数就返回。一个比较有意思的问题是accept的Socket是阻塞的还是非阻塞的呢？下边是MSDN上边的一段话：The accept function extracts thefirst connection on the queue of pending connections on socket s. It thencreates and returns a handle to the new socket. The newly created socket is thesocket that will handle the actual connection; it has the same properties assocket s, including the asynchronous events registered with the WSAAsyncSelector WSAEventSelect functions.
Ø  同步/异步IO  
有两种类型的文件IO同步：同步文件IO和异步文件IO。异步文件IO也就是重叠IO。
 
在同步文件IO中，线程启动一个IO操作然后就立即进入等待状态，直到IO操作完成后才醒来继续执行。而异步文件IO方式中，线程发送一个IO请求到内核，然后继续处理其他的事情，内核完成IO请求后，将会通知线程IO操作完成了。
 
如果IO请求需要大量时间执行的话，异步文件IO方式可以显著提高效率，因为在线程等待的这段时间内，CPU将会调度其他线程进行执行，如果没有其他线程需要执行的话，这段时间将会浪费掉（可能会调度操作系统的零页线程）。如果IO请求操作很快，用异步IO方式反而还低效，还不如用同步IO方式。 

同步IO在同一时刻只允许一个IO操作，也就是说对于同一个文件句柄的IO操作是序列化的，即使使用两个线程也不能同时对同一个文件句柄同时发出读写操作。重叠IO允许一个或多个线程同时发出IO请求。异步IO在请求完成时，通过将文件句柄设为有信号状态来通知应用程序，或者应用程序通过GetOverlappedResult察看IO请求是否完成，也可以通过一个事件对象来通知应用程序。高并发系统通常采用异步IO方式提高系统性能。

Ø  事件分离器  
事件分离器的概念是针对异步IO来说的。在同步IO的情况下，执行操作等待返回结果，不要事件分离器。异步IO的时候，发送请求后，结果是通过事件通知的。这是产生了事件分离器的需求。事件分离器主要任务是管理和分离不同文件描述符上的所发生的事件，让后通知相应的事件，派发相应的动作。下边是lighthttpd事件分离器定义：
		
		/** 
		 * fd-event handler for select(), poll() andrt-signals on Linux 2.4 
		 * 
		 */  
		typedef struct fdevents {  
		         fdevent_handler_t type;  
		   
		         fdnode **fdarray;  
		         size_t maxfds;  
		   
		#ifdef USE_LINUX_SIGIO  
		         int in_sigio;  
		         int signum;  
		         sigset_t sigset;  
		         siginfo_t siginfo;  
		         bitset *sigbset;  
		#endif  
		#ifdef USE_LINUX_EPOLL  
		         int epoll_fd;  
		         struct epoll_event *epoll_events;  
		#endif  
		#ifdef USE_POLL  
		         struct pollfd *pollfds;  
		   
		         size_t size;  
		         size_t used;  
		   
		         buffer_int unused;  
		#endif  
		#ifdef USE_SELECT  
		         fd_set select_read;  
		         fd_set select_write;  
		         fd_set select_error;  
		   
		         fd_set select_set_read;  
		         fd_set select_set_write;  
		         fd_set select_set_error;  
		   
		         int select_max_fd;  
		#endif  
		#ifdef USE_SOLARIS_DEVPOLL  
		         int devpoll_fd;  
		         struct pollfd *devpollfds;  
		#endif  
		#ifdef USE_FREEBSD_KQUEUE  
		         int kq_fd;  
		         struct kevent *kq_results;  
		         bitset *kq_bevents;  
		#endif  
		#ifdef USE_SOLARIS_PORT  
		         int port_fd;  
		#endif  
		         int (*reset)(struct fdevents *ev);  
		         void (*free)(struct fdevents *ev);  
		   
		         int (*event_add)(struct fdevents *ev, int fde_ndx, int fd,int events);  
		         int (*event_del)(struct fdevents *ev, int fde_ndx, int fd);  
		         int (*event_get_revent)(struct fdevents *ev, size_t ndx);  
		         int (*event_get_fd)(struct fdevents *ev, size_t ndx);  
		   
		         int (*event_next_fdndx)(struct fdevents *ev, int ndx);  
		   
		         int (*poll)(struct fdevents *ev, int timeout_ms);  
		   
		         int (*fcntl_set)(struct fdevents *ev, int fd);  
		} fdevents;  
		   
		fdevents *fdevent_init(size_tmaxfds, fdevent_handler_t type);  
		int fdevent_reset(fdevents*ev);  
		void fdevent_free(fdevents*ev);  
		   
		int fdevent_event_add(fdevents*ev, int *fde_ndx, int fd, int events);  
		int fdevent_event_del(fdevents*ev, int *fde_ndx, int fd);  
		intfdevent_event_get_revent(fdevents *ev, size_t ndx);  
		intfdevent_event_get_fd(fdevents *ev, size_t ndx);  
		fdevent_handlerfdevent_get_handler(fdevents *ev, int fd);  
		void *fdevent_get_context(fdevents *ev, int fd);  
		   
		int fdevent_event_next_fdndx(fdevents*ev, int ndx);  
		   
		int fdevent_poll(fdevents *ev,int timeout_ms);  
		   
		int fdevent_register(fdevents*ev, int fd, fdevent_handler handler, void *ctx);  
		int fdevent_unregister(fdevents*ev, int fd);  
		   
		int fdevent_fcntl_set(fdevents*ev, int fd);  
		   
		intfdevent_select_init(fdevents *ev);  
		int fdevent_poll_init(fdevents*ev);  
		intfdevent_linux_rtsig_init(fdevents *ev);  
		intfdevent_linux_sysepoll_init(fdevents *ev);  
		intfdevent_solaris_devpoll_init(fdevents *ev);  
		intfdevent_freebsd_kqueue_init(fdevents *ev);  



具体系统的事件操作通过：
fdevent_freebsd_kqueue.c
fdevent_linux_rtsig.c
fdevent_linux_sysepoll.c
fdevent_poll.c
fdevent_select.c
fdevent_solaris_devpoll.c
几个文件实现。

Ø  线程池
线程池基本上比较简单，实现线程的借入和借出，创建和销毁。最完好可以做到通过一个事件触发一个线程开始工作。下边给出一个简单的，没有实现根据事件触发的，linux和windows通用的线程池模型：

	spthread.h  
	#ifndef __spthread_hpp__  
	#define __spthread_hpp__  
	   
	#ifndef WIN32  
	   
	/// pthread  
	   
	#include <pthread.h>  
	#include <unistd.h>  
	   
	typedef void *sp_thread_result_t;  
	typedef pthread_mutex_tsp_thread_mutex_t;  
	typedef pthread_cond_t  sp_thread_cond_t;  
	typedef pthread_t       sp_thread_t;  
	typedef pthread_attr_t  sp_thread_attr_t;  
	   
	#definesp_thread_mutex_init(m,a)  pthread_mutex_init(m,a)  
	#definesp_thread_mutex_destroy(m) pthread_mutex_destroy(m)  
	#definesp_thread_mutex_lock(m)    pthread_mutex_lock(m)  
	#define sp_thread_mutex_unlock(m)   pthread_mutex_unlock(m)  
	   
	#definesp_thread_cond_init(c,a)   pthread_cond_init(c,a)  
	#definesp_thread_cond_destroy(c)  pthread_cond_destroy(c)  
	#definesp_thread_cond_wait(c,m)   pthread_cond_wait(c,m)  
	#definesp_thread_cond_signal(c)   pthread_cond_signal(c)  
	   
	#definesp_thread_attr_init(a)       pthread_attr_init(a)  
	#definesp_thread_attr_setdetachstate pthread_attr_setdetachstate  
	#defineSP_THREAD_CREATE_DETACHED    PTHREAD_CREATE_DETACHED  
	   
	#define sp_thread_self    pthread_self  
	#define sp_thread_create  pthread_create  
	   
	#define SP_THREAD_CALL  
	typedef sp_thread_result_t ( *sp_thread_func_t )( void * args );  
	   
	#define sp_sleep(x) sleep(x)  
	   
	#else///////////////////////////////////////////////////////////////////////  
	   
	// win32 thread  
	   
	#include <winsock2.h>  
	#include <process.h>  
	   
	typedef unsigned sp_thread_t;  
	   
	typedef unsignedsp_thread_result_t;  
	#define SP_THREAD_CALL__stdcall  
	typedef sp_thread_result_t (__stdcall * sp_thread_func_t )( void * args );  
	   
	typedef HANDLE  sp_thread_mutex_t;  
	typedef HANDLE  sp_thread_cond_t;  
	typedef DWORD   sp_thread_attr_t;  
	   
	#defineSP_THREAD_CREATE_DETACHED 1  
	#define sp_sleep(x)Sleep(1000*x)  
	   
	int sp_thread_mutex_init(sp_thread_mutex_t * mutex, void * attr )  
	{  
	         *mutex = CreateMutex( NULL, FALSE, NULL );  
	         return NULL == * mutex ? GetLastError() : 0;  
	}  
	   
	int sp_thread_mutex_destroy(sp_thread_mutex_t * mutex )  
	{  
	         int ret = CloseHandle( *mutex );  
	   
	         return 0 == ret ? GetLastError() : 0;  
	}  
	   
	int sp_thread_mutex_lock(sp_thread_mutex_t * mutex )  
	{  
	         int ret = WaitForSingleObject( *mutex, INFINITE );  
	         return WAIT_OBJECT_0 == ret ? 0 : GetLastError();  
	}  
	   
	int sp_thread_mutex_unlock(sp_thread_mutex_t * mutex )  
	{  
	         int ret = ReleaseMutex( *mutex );  
	         return 0 != ret ? 0 : GetLastError();  
	}  
	   
	int sp_thread_cond_init(sp_thread_cond_t * cond, void * attr )  
	{  
	         *cond = CreateEvent( NULL, FALSE, FALSE, NULL );  
	         return NULL == *cond ? GetLastError() : 0;  
	}  
	   
	int sp_thread_cond_destroy(sp_thread_cond_t * cond )  
	{  
	         int ret = CloseHandle( *cond );  
	         return 0 == ret ? GetLastError() : 0;  
	}  
	   
	/* 
	Caller MUST be holding themutex lock; the 
	lock is released and the calleris blocked waiting 
	on 'cond'. When 'cond' issignaled, the mutex 
	is re-acquired before returningto the caller. 
	*/  
	int sp_thread_cond_wait(sp_thread_cond_t * cond, sp_thread_mutex_t * mutex )  
	{  
	         int ret = 0;  
	   
	         sp_thread_mutex_unlock( mutex );  
	   
	         ret = WaitForSingleObject( *cond, INFINITE );  
	   
	         sp_thread_mutex_lock( mutex );  
	   
	         return WAIT_OBJECT_0 == ret ? 0 : GetLastError();  
	}  
	   
	int sp_thread_cond_signal(sp_thread_cond_t * cond )  
	{  
	         int ret = SetEvent( *cond );  
	         return 0 == ret ? GetLastError() : 0;  
	}  
	   
	sp_thread_t sp_thread_self()  
	{  
	         return GetCurrentThreadId();  
	}  
	   
	int sp_thread_attr_init(sp_thread_attr_t * attr )  
	{  
	         *attr = 0;  
	         return 0;  
	}  
	   
	intsp_thread_attr_setdetachstate( sp_thread_attr_t * attr, int detachstate )  
	{  
	         *attr |= detachstate;  
	         return 0;  
	}  
	   
	int sp_thread_create(sp_thread_t * thread, sp_thread_attr_t * attr,  
	                   sp_thread_func_t myfunc, void * args )  
	{  
	         // _beginthreadex returns 0 on an error  
	         HANDLE h = (HANDLE)_beginthreadex( NULL, 0, myfunc, args, 0,thread );  
	         return h > 0 ? 0 : GetLastError();  
	}  
	   
	#endif  
	   
	#endif  
	threadpool.h  
	/** 
	 * threadpool.h 
	 * 
	 * This file declares the functionalityassociated with 
	 * your implementation of a threadpool. 
	 */  
	   
	#ifndef __threadpool_h__  
	#define __threadpool_h__  
	   
	#ifdef __cplusplus  
	extern "C" {  
	#endif  
	   
	// maximum number of threadsallowed in a pool  
	#define MAXT_IN_POOL 200  
	   
	// You must hide the internaldetails of the threadpool  
	// structure from callers, thusdeclare threadpool of type "void".  
	// In threadpool.c, you willuse type conversion to coerce  
	// variables of type"threadpool" back and forth to a  
	// richer, internal type.  (See threadpool.c for details.)  
	   
	typedef void *threadpool;  
	   
	// "dispatch_fn"declares a typed function pointer.  A  
	// variable of type"dispatch_fn" points to a function  
	// with the followingsignature:  
	//  
	//     void dispatch_function(void *arg);  
	   
	typedef void(*dispatch_fn)(void *);  
	   
	/** 
	 * create_threadpool creates a fixed-sizedthread 
	 * pool. If the function succeeds, it returns a (non-NULL) 
	 * "threadpool", else it returnsNULL. 
	 */  
	threadpoolcreate_threadpool(int num_threads_in_pool);  
	   
	   
	/** 
	 * dispatch sends a thread off to do somework.  If 
	 * all threads in the pool are busy, dispatchwill 
	 * block until a thread becomes free and isdispatched. 
	 * 
	 * Once a thread is dispatched, this functionreturns 
	 * immediately. 
	 * 
	 * The dispatched thread calls into thefunction 
	 * "dispatch_to_here" with argument"arg". 
	 */  
	intdispatch_threadpool(threadpool from_me, dispatch_fn dispatch_to_here,  
	               void *arg);  
	   
	/** 
	 * destroy_threadpool kills the threadpool,causing 
	 * all threads in it to commit suicide, andthen 
	 * frees all the memory associated with thethreadpool. 
	 */  
	voiddestroy_threadpool(threadpool destroyme);  
	   
	#ifdef __cplusplus  
	}  
	#endif  
	   
	#endif  
	threadpool.c  
	/** 
	 * threadpool.c 
	 * 
	 * This file will contain your implementation ofa threadpool. 
	 */  
	   
	#include <stdio.h>  
	#include <stdlib.h>  
	//#include <unistd.h>  
	//#include <sp_thread.h>  
	#include <string.h>  
	   
	#include"threadpool.h"  
	#include "spthread.h"  
	   
	typedef struct _thread_st {  
	         sp_thread_t id;  
	         sp_thread_mutex_t mutex;  
	         sp_thread_cond_t cond;  
	         dispatch_fn fn;  
	         void *arg;  
	         threadpool parent;  
	} _thread;  
	   
	// _threadpool is the internalthreadpool structure that is  
	// cast to type"threadpool" before it given out to callers  
	typedef struct _threadpool_st {  
	         // you should fill in this structure with whatever you need  
	         sp_thread_mutex_t tp_mutex;  
	         sp_thread_cond_t tp_idle;  
	         sp_thread_cond_t tp_full;  
	         sp_thread_cond_t tp_empty;  
	         _thread ** tp_list;  
	         int tp_index;  
	         int tp_max_index;  
	         int tp_stop;  
	   
	         int tp_total;  
	} _threadpool;  
	   
	threadpoolcreate_threadpool(int num_threads_in_pool)  
	{  
	         _threadpool *pool;  
	   
	         // sanity check the argument  
	         if ((num_threads_in_pool <= 0) || (num_threads_in_pool> MAXT_IN_POOL))  
	                   return NULL;  
	   
	         pool = (_threadpool *) malloc(sizeof(_threadpool));  
	         if (pool == NULL) {  
	                   fprintf(stderr, "Out of memory creating a newthreadpool!\n");  
	                   return NULL;  
	         }  
	   
	         // add your code here to initialize the newly createdthreadpool  
	         sp_thread_mutex_init( &pool->tp_mutex, NULL );  
	         sp_thread_cond_init( &pool->tp_idle, NULL );  
	         sp_thread_cond_init( &pool->tp_full, NULL );  
	         sp_thread_cond_init( &pool->tp_empty, NULL );  
	         pool->tp_max_index = num_threads_in_pool;  
	         pool->tp_index = 0;  
	         pool->tp_stop = 0;  
	         pool->tp_total = 0;  
	         pool->tp_list = ( _thread ** )malloc( sizeof( void * ) *MAXT_IN_POOL );  
	         memset( pool->tp_list, 0, sizeof( void * ) * MAXT_IN_POOL);  
	   
	         return (threadpool) pool;  
	}  
	   
	int save_thread( _threadpool *pool, _thread * thread )  
	{  
	         int ret = -1;  
	   
	         sp_thread_mutex_lock( &pool->tp_mutex );  
	   
	         if( pool->tp_index < pool->tp_max_index ) {  
	                   pool->tp_list[ pool->tp_index ] = thread;  
	                   pool->tp_index++;  
	                   ret = 0;  
	   
	                   sp_thread_cond_signal( &pool->tp_idle );  
	   
	                   if( pool->tp_index >= pool->tp_total ) {  
	                            sp_thread_cond_signal(&pool->tp_full );  
	                   }  
	         }  
	   
	         sp_thread_mutex_unlock( &pool->tp_mutex );  
	   
	         return ret;  
	}  
	   
	sp_thread_result_tSP_THREAD_CALL wrapper_fn( void * arg )  
	{  
	         _thread * thread = (_thread*)arg;  
	         _threadpool * pool = (_threadpool*)thread->parent;  
	   
	         for( ; 0 == ((_threadpool*)thread->parent)->tp_stop; ){  
	                   thread->fn( thread->arg );  
	   
	                   if( 0 !=((_threadpool*)thread->parent)->tp_stop ) break;  
	   
	                   sp_thread_mutex_lock( &thread->mutex );  
	                   if( 0 == save_thread( thread->parent, thread )) {  
	                            sp_thread_cond_wait(&thread->cond, &thread->mutex );  
	                            sp_thread_mutex_unlock(&thread->mutex );  
	                   } else {  
	                            sp_thread_mutex_unlock(&thread->mutex );  
	                            sp_thread_cond_destroy(&thread->cond );  
	                            sp_thread_mutex_destroy(&thread->mutex );  
	   
	                            free( thread );  
	                            break;  
	                   }  
	         }  
	   
	         sp_thread_mutex_lock( &pool->tp_mutex );  
	         pool->tp_total--;  
	         if( pool->tp_total <= 0 ) sp_thread_cond_signal(&pool->tp_empty );  
	         sp_thread_mutex_unlock( &pool->tp_mutex );  
	   
	         return 0;  
	}  
	   
	intdispatch_threadpool(threadpool from_me, dispatch_fn dispatch_to_here, void*arg)  
	{  
	         int ret = 0;  
	   
	         _threadpool *pool = (_threadpool *) from_me;  
	         sp_thread_attr_t attr;  
	         _thread * thread = NULL;  
	   
	         // add your code here to dispatch a thread  
	         sp_thread_mutex_lock( &pool->tp_mutex );  
	   
	         if( pool->tp_index <= 0 && pool->tp_total>= pool->tp_max_index ) {  
	                   sp_thread_cond_wait( &pool->tp_idle,&pool->tp_mutex );  
	         }  
	   
	         if( pool->tp_index <= 0 ) {  
	                   _thread * thread = ( _thread * )malloc( sizeof(_thread ) );  
	                   memset( &( thread->id ), 0, sizeof(thread->id ) );  
	                   sp_thread_mutex_init( &thread->mutex, NULL);  
	                   sp_thread_cond_init( &thread->cond, NULL );  
	                   thread->fn = dispatch_to_here;  
	                   thread->arg = arg;  
	                   thread->parent = pool;  
	   
	                   sp_thread_attr_init( &attr );  
	                   sp_thread_attr_setdetachstate( &attr,SP_THREAD_CREATE_DETACHED );  
	   
	                   if( 0 == sp_thread_create( &thread->id,&attr, wrapper_fn, thread ) ) {  
	                            pool->tp_total++;  
	                            printf( "create thread#%ld\n",thread->id );  
	                   } else {  
	                            ret = -1;  
	                            printf( "cannot createthread\n" );  
	                            sp_thread_mutex_destroy(&thread->mutex );  
	                            sp_thread_cond_destroy(&thread->cond );  
	                            free( thread );  
	                   }  
	         } else {  
	                   pool->tp_index--;  
	                   thread = pool->tp_list[ pool->tp_index ];  
	                   pool->tp_list[ pool->tp_index ] = NULL;  
	   
	                   thread->fn = dispatch_to_here;  
	                   thread->arg = arg;  
	                   thread->parent = pool;  
	   
	                   sp_thread_mutex_lock( &thread->mutex );  
	                   sp_thread_cond_signal( &thread->cond ) ;  
	                   sp_thread_mutex_unlock ( &thread->mutex );  
	         }  
	   
	         sp_thread_mutex_unlock( &pool->tp_mutex );  
	   
	         return ret;  
	}  
	   
	void destroy_threadpool(threadpooldestroyme)  
	{  
	         _threadpool *pool = (_threadpool *) destroyme;  
	   
	         // add your code here to kill a threadpool  
	         int i = 0;  
	   
	         sp_thread_mutex_lock( &pool->tp_mutex );  
	   
	         if( pool->tp_index < pool->tp_total ) {  
	                   printf( "waiting for %d thread(s) tofinish\n", pool->tp_total - pool->tp_index );  
	                   sp_thread_cond_wait( &pool->tp_full,&pool->tp_mutex );  
	         }  
	   
	         pool->tp_stop = 1;  
	   
	         for( i = 0; i < pool->tp_index; i++ ) {  
	                   _thread * thread = pool->tp_list[ i ];  
	   
	                   sp_thread_mutex_lock( &thread->mutex );  
	                   sp_thread_cond_signal( &thread->cond ) ;  
	                   sp_thread_mutex_unlock ( &thread->mutex );  
	         }  
	   
	         if( pool->tp_total > 0 ) {  
	                   printf( "waiting for %d thread(s) toexit\n", pool->tp_total );  
	                   sp_thread_cond_wait( &pool->tp_empty,&pool->tp_mutex );  
	         }  
	   
	         for( i = 0; i < pool->tp_index; i++ ) {  
	                   free( pool->tp_list[ i ] );  
	                   pool->tp_list[ i ] = NULL;  
	         }  
	   
	         sp_thread_mutex_unlock( &pool->tp_mutex );  
	   
	         pool->tp_index = 0;  
	   
	         sp_thread_mutex_destroy( &pool->tp_mutex );  
	         sp_thread_cond_destroy( &pool->tp_idle );  
	         sp_thread_cond_destroy( &pool->tp_full );  
	         sp_thread_cond_destroy( &pool->tp_empty );  
	   
	         free( pool->tp_list );  
	         free( pool );  
	}  

2)    常见的设计模式
根据Socket的阻塞非阻塞，IO的同步和异步。可以分为如下4中情形
阻塞同步     |    阻塞异步
_________|______________
非阻塞同步  |   非阻塞异步
 
阻塞同步方式是原始的方式，也是许多教科书上介绍的方式，因为Socket和IO默认的为阻塞和同步方式。基本流程如下：


	listen_fd = socket( AF_INET,SOCK_STREAM,0 )  
	bind( listen_fd, (struct sockaddr*)&my_addr, sizeof(struct sockaddr_in))  
	listen( listen_fd,1 )  
	accept( listen_fd,  (struct sockaddr*)&remote_addr,&addr_len )  
	recv( accept_fd ,&in_buf ,1024 ,0 )  
	close(accept_fd) 
 
阻塞异步方式有所改进，但是Socket的阻塞方式，前一个连接没有处理完成，下一个连接不能接入，是高并发服务器所不可接收的方式。只不过在上边阻塞同步方式的基础上使用select（严格来说select是一种IO多路服用技术。因为linux尚没有完整的实现异步IO，而winsock实在理解socket没有linux上面那么直观。，这里为了方便，没有做严格的区分）或者其它异步IO方式。
非阻塞同步方式，通过设置socket选项为NONBLOCK，可以很快的接收连接，但是处理采用同步IO方式，服务器处理性能也比较差。
上边三种方式不做深入介绍。下边主要从非阻塞异步IO方式介绍。
非阻塞异步IO方式中，由于异步IO方式在同一系统可能有多种实现，不同系统也有不同实现，下边介绍几种常见的IO方式和服务器框架。
 
Ø  Select
Select采用轮训注册的fd方式。是一种比较老的IO多路服用实现方式，效率相对要差一些。Select方式在windows和linux上都支持。
基本框架如下：

	socket( AF_INET,SOCK_STREAM,0 )  
	fcntl(listen_fd, F_SETFL,flags|O_NONBLOCK);  
	bind( listen_fd, (structsockaddr *)&my_addr,sizeof(struct sockaddr_in))  
	listen( listen_fd,1 )  
	FD_ZERO( &fd_sets );  
	FD_SET(listen_fd,&fd_sets);  
	for(k=0; k<=i; k++){  
	         FD_SET(accept_fds[k],&fd_sets);  
	}  
	events = select( max_fd + 1,&fd_sets, NULL, NULL, NULL );  
	if(FD_ISSET(listen_fd,&fd_sets) ){  
	accept_fd = accept( listen_fd, (structsockaddr *)&remote_addr,&addr_len );  
	}  
	for( j=0; j<=i; j++ ){  
	         if( FD_ISSET(accept_fds[j],&fd_sets) ){  
	                   recv( accept_fds[j] ,&in_buf ,1024 ,0 );  
	         }  
	}  

Ø  Epoll
Epoll是linux2.6内核以后支持的一种高性能的IO多路服用技术。服务器框架如下：
[cpp] view plaincopy

	
	socket( AF_INET,SOCK_STREAM,0 )  
	fcntl(listen_fd, F_SETFL,flags|O_NONBLOCK);  
	bind( listen_fd, (structsockaddr *)&my_addr,sizeof(struct sockaddr_in))  
	listen( listen_fd,1 )  
	epoll_ctl(epfd,EPOLL_CTL_ADD,listen_fd,&ev);  
	ev_s = epoll_wait(epfd,events,20,500 );  
	for(i=0; i<ev_s;i++){  
	                   if(events[i].data.fd==listen_fd){  
	                            accept_fd = accept( listen_fd,(structsockaddr *)&remote_addr,&addr_len );  
	                            fcntl(accept_fd, F_SETFL,flags|O_NONBLOCK);  
	                            epoll_ctl(epfd,EPOLL_CTL_ADD,accept_fd,&ev);  
	                   }  
	                   else if(events[i].events&EPOLLIN){  
	                            recv( events[i].data.fd ,&in_buf,1024 ,0 );  
	                   }  
	}  


Ø  AIO
在windows上微软实现了异步IO，通过AIO可以方便的实现高并发的服务器。框架如下：
[cpp] view plaincopy

	WSAStartup( 0x0202 ,  & wsaData)  
	CreateIoCompletionPort(INVALID_HANDLE_VALUE,NULL,  0 ,  0 )  
	WSASocket(AF_INET,SOCK_STREAM,  0 , NULL,  0 , WSA_FLAG_OVERLAPPED)  
	bind(Listen, (PSOCKADDR)  & InternetAddr,  sizeof (InternetAddr))  
	listen(Listen,  5 )  
	WSAAccept(Listen, NULL, NULL,NULL,  0 )  
	PerHandleData  = (LPPER_HANDLE_DATA) GlobalAlloc(GPTR, sizeof (PER_HANDLE_DATA)  
	CreateIoCompletionPort((HANDLE)Accept, CompletionPort, (DWORD) PerHandleData, 0 )  
	PerIoData= (LPPER_IO_OPERATION_DATA)GlobalAlloc(GPTR, sizeof (PER_IO_OPERATION_DATA))  
	WSARecv(Accept,&(PerIoData->DataBuf),1,&RecvBytes,&Flags,&(PerIoData->Overlapped), NULL)  
	(GetQueuedCompletionStatus(CompletionPort,  & BytesTransferred,  
	         (LPDWORD) & PerHandleData,(LPOVERLAPPED  * )  & PerIoData, INFINITE)  
	if  (PerIoData -> BytesRECV  > PerIoData -> BytesSEND){  
	WSASend(PerHandleData-> Socket,  & (PerIoData ->DataBuf),  1 ,  & SendBytes,  0 ,  
	             & (PerIoData ->Overlapped), NULL)  
	}  

3)    引入线程池和事件分离器后
由于上边只是单纯的使用非阻塞Socket和异步IO的方式。提高了接收连接和处理的速度。但是还是不能解决两个客户端同时连接的问题。这时就需要引入多线程机制。引入多线程后，又有许多策略。Linux上通常采用主进程负责接收连接，之后fork子进程处理连接。Windows通常采用线程池方式，避免线程创建和销毁的开销，当然linux上也可以采用线程池方式。采用多进程和多线程方式后。事件处理也可以再优化，定义一个简单的事件处理器，把所有事件放入一个队列，各个线程去事件队列取相应的事件，然后自己开始工作。这就是我上边提到的半同步/半异步方式了。如果线程工作的时候是接收到连接后，自己处理后续的发送和接收，然后选出另外一个线程作为领导继续接收连接，其它线程作为追随者。这就是领导者/追随者模式了。具体可以参考ACE的Reactor和Preactor的具体实现。半同步和/半异步网上也有很多的讨论，可以自己深入研究。代码就比较复杂了，这里就不给出代码了。给出一个linux下类似，相对简单的fork子进程+epoll方式的实现：

	#include<sys/socket.h>  
	#include <sys/wait.h>  
	#include <netinet/in.h>  
	#include <netinet/tcp.h>  
	#include <sys/epoll.h>  
	#include <sys/sendfile.h>  
	#include <sys/stat.h>  
	#include <unistd.h>  
	#include <stdio.h>  
	#include <stdlib.h>  
	#include <string.h>  
	#include <strings.h>  
	#include <fcntl.h>  
	#include <errno.h>  
	  
	#define HANDLE_INFO   1  
	#define HANDLE_SEND   2  
	#define HANDLE_DEL    3  
	#define HANDLE_CLOSE  4  
	  
	#define MAX_REQLEN         1024  
	#define MAX_PROCESS_CONN    3  
	#define FIN_CHAR           0x00  
	#define SUCCESS  0  
	#define ERROR   -1  
	  
	typedef struct event_handle{  
	    int socket_fd;  
	    int file_fd;  
	    int file_pos;  
	    int epoll_fd;  
	    char request[MAX_REQLEN];  
	    int request_len;  
	    int ( * read_handle )( struct event_handle * ev );  
	    int ( * write_handle )( struct event_handle * ev );  
	    int handle_method;  
	} EV,* EH;  
	typedef int ( * EVENT_HANDLE )( struct event_handle * ev );  
	  
	int create_listen_fd( int port ){  
	    int listen_fd;  
	    struct sockaddr_inmy_addr;  
	    if( ( listen_fd = socket( AF_INET, SOCK_STREAM, 0 ) ) == -1 ){  
	        perror( "create socket error" );  
	        exit( 1 );  
	    }  
	    int flag;  
	    int olen = sizeof(int);  
	    if( setsockopt( listen_fd, SOL_SOCKET, SO_REUSEADDR  
	                       , (const void *)&flag, olen ) == -1 ){  
	        perror( "setsockopt error" );  
	    }  
	    flag = 5;  
	    if( setsockopt( listen_fd, IPPROTO_TCP, TCP_DEFER_ACCEPT, &flag, olen ) == -1 ){  
	        perror( "setsockopt error" );  
	    }  
	    flag = 1;  
	    if( setsockopt( listen_fd, IPPROTO_TCP, TCP_CORK, &flag, olen ) == -1 ){  
	        perror( "setsockopt error" );  
	    }  
	    int flags = fcntl( listen_fd, F_GETFL, 0 );  
	    fcntl( listen_fd, F_SETFL, flags|O_NONBLOCK );  
	    my_addr.sin_family = AF_INET;  
	    my_addr.sin_port = htons( port );  
	    my_addr.sin_addr.s_addr = INADDR_ANY;  
	    bzero( &( my_addr.sin_zero ), 8 );  
	    if( bind( listen_fd, ( struct sockaddr * )&my_addr,  
	    sizeof( struct sockaddr_in ) ) == -1 ) {  
	        perror( "bind error" );  
	        exit( 1 );  
	    }  
	    if( listen( listen_fd, 1 ) == -1 ){  
	        perror( "listen error" );  
	        exit( 1 );  
	    }  
	    return listen_fd;  
	}  
	  
	int create_accept_fd( int listen_fd ){  
	    int addr_len = sizeof( struct sockaddr_in );  
	    struct sockaddr_inremote_addr;  
	    int accept_fd = accept( listen_fd,  
	        ( struct sockaddr * )&remote_addr, &addr_len );  
	    int flags = fcntl( accept_fd, F_GETFL, 0 );  
	    fcntl( accept_fd, F_SETFL, flags|O_NONBLOCK );  
	    return accept_fd;  
	}  
	  
	int fork_process( int process_num ){  
	    int i;  
	    int pid=-1;  
	    for( i = 0; i < process_num; i++ ){  
	        if( pid != 0 ){  
	            pid = fork();  
	        }  
	    }  
	    return pid;  
	}  
	  
	int init_evhandle(EH ev,int socket_fd,int epoll_fd,EVENT_HANDLEr_handle,EVENT_HANDLE w_handle){  
	    ev->epoll_fd = epoll_fd;  
	    ev->socket_fd = socket_fd;  
	    ev->read_handle = r_handle;  
	    ev->write_handle = w_handle;  
	    ev->file_pos = 0;  
	    ev->request_len = 0;  
	    ev->handle_method = 0;  
	    memset( ev->request, 0, 1024 );  
	}  
	//accept->accept_queue->request->request_queue->output->output_queue  
	//multi process sendfile  
	int parse_request(EH ev){  
	    ev->request_len--;  
	    *( ev->request + ev->request_len - 1 ) = 0x00;  
	    int i;  
	    for( i=0; i<ev->request_len; i++ ){  
	        if( ev->request[i] == ':' ){  
	            ev->request_len = ev->request_len-i-1;  
	            char temp[MAX_REQLEN];  
	            memcpy( temp, ev->request, i );  
	            ev->handle_method = atoi( temp );  
	            memcpy( temp, ev->request+i+1, ev->request_len );  
	            memcpy( ev->request, temp, ev->request_len );  
	            break;  
	        }  
	    }  
	    //handle_request(ev );  
	    //registerto epoll EPOLLOUT  
	  
	    struct epoll_eventev_temp;  
	    ev_temp.data.ptr = ev;  
	    ev_temp.events = EPOLLOUT|EPOLLET;  
	    epoll_ctl( ev->epoll_fd, EPOLL_CTL_MOD, ev->socket_fd, &ev_temp );  
	    return SUCCESS;  
	}  
	  
	int handle_request(EH ev){  
	    struct statfile_info;  
	    switch( ev->handle_method ){  
	        case HANDLE_INFO:  
	            ev->file_fd = open( ev->request, O_RDONLY );  
	            if( ev->file_fd == -1 ){  
	               send( ev->socket_fd, "open file failed\n", strlen("open file failed\n"), 0 );  
	               return -1;  
	            }  
	            fstat(ev->file_fd, &file_info);  
	            char info[MAX_REQLEN];  
	            sprintf(info,"filelen:%d\n",file_info.st_size);  
	            send( ev->socket_fd, info, strlen( info ), 0 );  
	            break;  
	        case HANDLE_SEND:  
	            ev->file_fd = open( ev->request, O_RDONLY );  
	            if( ev->file_fd == -1 ){  
	               send( ev->socket_fd, "open file failed\n", strlen("open file failed\n"), 0 );  
	               return -1;  
	            }  
	            fstat(ev->file_fd, &file_info);  
	            sendfile( ev->socket_fd, ev->file_fd, 0, file_info.st_size );  
	            break;  
	        case HANDLE_DEL:  
	            break;  
	        case HANDLE_CLOSE:  
	            break;  
	    }  
	    finish_request( ev );  
	    return SUCCESS;  
	}  
	  
	int finish_request(EH ev){  
	    close(ev->socket_fd);  
	    close(ev->file_fd);  
	    ev->handle_method = -1;  
	    clean_request( ev );  
	    return SUCCESS;  
	}  
	  
	int clean_request(EH ev){  
	    memset( ev->request, 0, MAX_REQLEN );  
	    ev->request_len = 0;  
	}  
	  
	int read_hook_v2( EH ev ){  
	    char in_buf[MAX_REQLEN];  
	    memset( in_buf, 0, MAX_REQLEN );  
	    int recv_num = recv( ev->socket_fd, &in_buf, MAX_REQLEN, 0 );  
	    if( recv_num ==0 ){  
	        close( ev->socket_fd );  
	        return ERROR;  
	    }  
	    else{  
	        //checkifoverflow  
	        if( ev->request_len > MAX_REQLEN-recv_num ){  
	            close( ev->socket_fd );  
	           clean_request( ev );  
	        }  
	        memcpy( ev->request + ev->request_len, in_buf, recv_num );  
	        ev->request_len += recv_num;  
	        if( recv_num == 2 && ( !memcmp( &in_buf[recv_num-2], "\r\n", 2 ) ) ){  
	           parse_request(ev);  
	        }  
	    }  
	    return recv_num;  
	}  
	  
	int write_hook_v1( EH ev ){  
	    struct statfile_info;  
	    ev->file_fd = open( ev->request, O_RDONLY );  
	    if( ev->file_fd == ERROR ){  
	        send( ev->socket_fd, "openfile failed\n", strlen("openfile failed\n"), 0 );  
	        return ERROR;  
	    }  
	    fstat(ev->file_fd, &file_info);  
	    int write_num;  
	    while(1){  
	        write_num = sendfile( ev->socket_fd, ev->file_fd, (off_t *)&ev->file_pos, 10240 );  
	        ev->file_pos += write_num;  
	        if( write_num == ERROR ){  
	            if( errno == EAGAIN ){  
	               break;  
	            }  
	        }  
	        else if( write_num == 0 ){  
	            printf( "writed:%d\n", ev->file_pos );  
	            //finish_request(ev );  
	            break;  
	        }  
	    }  
	    return SUCCESS;  
	}  
	  
	int main(){  
	    int listen_fd = create_listen_fd( 3389 );  
	    int pid = fork_process( 3 );  
	    if( pid == 0 ){  
	        int accept_handles = 0;  
	        struct epoll_eventev, events[20];  
	        int epfd = epoll_create( 256 );  
	        int ev_s = 0;  
	  
	        ev.data.fd = listen_fd;  
	        ev.events = EPOLLIN|EPOLLET;  
	        epoll_ctl( epfd, EPOLL_CTL_ADD, listen_fd, &ev );  
	        struct event_handleev_handles[256];  
	        for( ;; ){  
	            ev_s = epoll_wait( epfd, events, 20, 500 );  
	            int i = 0;  
	            for( i = 0; i<ev_s; i++ ){  
	               if( events[i].data.fd == listen_fd ){  
	                   if( accept_handles < MAX_PROCESS_CONN ){  
	                       accept_handles++;  
	                       int accept_fd = create_accept_fd( listen_fd );  
	                       init_evhandle(&ev_handles[accept_handles],accept_fd,epfd,read_hook_v2,write_hook_v1);  
	                       ev.data.ptr = &ev_handles[accept_handles];  
	                       ev.events = EPOLLIN|EPOLLET;  
	                       epoll_ctl( epfd, EPOLL_CTL_ADD, accept_fd, &ev );  
	                   }  
	               }  
	               else if( events[i].events&EPOLLIN ){  
	                   EVENT_HANDLE current_handle = ( ( EH )( events[i].data.ptr ) )->read_handle;  
	                   EH current_event = ( EH )( events[i].data.ptr );  
	                   ( *current_handle )( current_event );  
	               }  
	               else if( events[i].events&EPOLLOUT ){  
	                   EVENT_HANDLE current_handle = ( ( EH )( events[i].data.ptr ) )->write_handle;  
	                   EH current_event = ( EH )( events[i].data.ptr );  
	                   if( ( *current_handle )( current_event )  == 0 ){  
	                       accept_handles--;  
	                   }  
	               }  
	            }  
	        }  
	    }  
	    else{  
	        //managerthe process  
	        int child_process_status;  
	        wait( &child_process_status );  
	    }  
	  
	    return SUCCESS;  
	}  


## 三. 分布式系统设计
前面讲述了分布式系统中的核心的服务器的实现。可以是http服务器，缓存服务器，分布式文件系统等的内部实现。下边主要从一个高并发的大型网站出发，看一个高并发系统的设计。下边是一个高并发系统的逻辑结构：

主要是参考这篇文章http://www.chinaz.com/web/2010/0310/108211.shtml。下边主要想从这个架构的各个部分的实现展开。
1.     缓存系统
缓存是每一个高并发，高可用系统不可或缺的模块。下边就几个常见缓存系统系统进行介绍。
Squid
Squid作为一个前端缓存，通常部署在网络的离用户最近的地方，通过缓存网站的页面，使用户不必每次都跑到服务器去取数据，提高系统响应和性能。实现应该比较简单：一个带有存储功能的代理。用户访问页面的时候，由它代理，然后存储请求结果，下次再访问的时候，查看是否需要更新，有更新就去服务器取新数据，否则直接返回用户页面。
 
Ehcache
Ehcache是一个对象缓存系统。通常在J2EE中配合Hibernate使用，这里请原谅作者本人之前是做J2EE开发的，其它使用方式暂不是很了解。应用查询数据库，对经常需要查询，却更新不频繁的数据，可以放入ehcache缓存，提高访问速度。Ehcahe支持内存缓存和硬盘两种方式，支持分布式缓存。数据缓存的基本原理就是：为需要缓存的对象建立一个map，临时对象放入map，查询的时候先查询map，没有找到再查找数据库。关机时可以序列化到硬盘。分布式缓存没有研究过。
页面缓存和动态页面静态化
在大型网站经常使用的一种缓存技术就是动态页面的缓存。由于动态页面经常更新，上边的缓存就不起作用了。通常会采用SSI(Server side include)等技术将动态页面的或者页面片段进行缓存。
还有一种就是动态页面静态化。下边是一本讲spring的书中给出的j2EE中的动态页面静态化的示例：
[java] view plaincopy

	/** 
	 * 动态内容静态化的Filter。将变化非常缓慢的动态文件生成静态文件。 
	 */  
	package com.zsl.cache.filter;  
	   
	import java.io.File;  
	import java.io.IOException;  
	import java.io.UnsupportedEncodingException;  
	import java.net.URLEncoder;  
	import java.util.Map;  
	   
	importjavax.management.RuntimeErrorException;  
	importjavax.servlet.FilterChain;  
	importjavax.servlet.ServletException;  
	importjavax.servlet.ServletRequest;  
	importjavax.servlet.ServletResponse;  
	importjavax.servlet.http.HttpServletRequest;  
	importjavax.servlet.http.HttpServletResponse;  
	   
	importorg.apache.commons.io.FilenameUtils;  
	importorg.apache.http.HttpResponse;  
	importorg.springframework.core.io.Resource;  
	   
	importcom.sun.xml.bind.v2.runtime.output.Encoded;  
	   
	   
	/** 
	 * @author zsl 
	 * 
	 */  
	public class FileCacheFilterextends AbstractCacheFilter{  
	         private String root;  
	          
	         private final String SUFFIX = ".html";  
	          
	         public final void setFileDir(Resource dir){  
	                   try {  
	                            File f = dir.getFile();  
	                            f.mkdirs();  
	                            if(!f.isDirectory()){  
	                                     throw newIllegalArgumentException("Invalid directory: "+f.getPath());  
	                            }  
	                            if(!f.canWrite())  
	                                     throw newIllegalArgumentException("Cannot write to directory: "+f.getPath());  
	                            root = f.getPath();  
	                             
	                            if(!root.endsWith("/")&&!root.endsWith("//"))  
	                                     root = root+"/";  
	                   } catch (IOException e) {  
	                            // TODO Auto-generated catch block  
	                            throw new IllegalArgumentException(e);  
	                   }  
	         }  
	          
	         public void afterPropertiesSet() throws Exception {  
	                   super.afterPropertiesSet();  
	                   if(!new File(root).isDirectory()){  
	                            throw newIllegalArgumentException("No directory: "+root);  
	                   }  
	         }  
	          
	         public void doFilter(ServletRequest request,ServletResponseresponse,FilterChain chain) throws IOException,ServletException {  
	                   HttpServletRequest httpRequest =(HttpServletRequest)request;  
	                   String key = getKey(httpRequest);  
	                   if(key == null){  
	                            chain.doFilter(request,response);  
	                   }else{  
	                            File file = key2File(key);  
	                            if(file.isFile()){  
	                                     HttpServletResponse httpResponse= (HttpServletResponse)response;  
	                                     httpResponse.setContentType(getContentType());  
	                                     httpResponse.setHeader("Content-Encoding","gzip");  
	                                     httpResponse.setContentLength((int)file.length());  
	                                     FileUtil.readFil(file,httpResponse.getOutputStream());  
	                            }else{  
	                                     //缓存未找到文件  
	                                     HttpServletResponse httpResponse= (HttpServletResponse)response;  
	                                     CachedResponseWrapper wrapper =new CachedResponseWrapper(httpResponse);  
	                                     chain.doFilter(request,response);  
	                                     if(wrapper.getStatus() ==HttpServletResponse.SC_OK){  
	                                               byte[] data =GZipUtil.gzip(wrapper.getResponseData());  
	                                               FileUtil.writeFile(file,data);  
	                                               httpResponse.setContentType(getContentType());  
	                                               httpResponse.setHeader("Content-Encoding","gzip");  
	                                               httpResponse.setContentLength(data.length);  
	                                               httpResponse.getOutputStream().write(data);  
	                                     }  
	                            }  
	                   }  
	                    
	         }  
	          
	         private File key2File(String key){  
	                   int  hash =key.hashCode();  
	                   int dir1 = (hash &0xff00)>>8;  
	                   int dir2 = hash & 0xff;  
	                   String  dir= root+dir1+"/"+dir2;  
	                   File fdir = new File(dir);  
	                   if(!fdir.isAbsolute()){  
	                            if(!fdir.mkdirs()){  
	                                     return null;  
	                            }  
	                   }  
	                   return newFile(dir+"/"+encode(key)+SUFFIX);  
	         }  
	          
	         private String encode(String key){  
	                   try {  
	                            return URLEncoder.encode(key,"UTF-8");  
	                   } catch (UnsupportedEncodingException e) {  
	                            throw new RuntimeException(e);  
	                   }  
	                    
	         }  
	          
	         public void remove(String url,Map<String,String>parameters){  
	                   String key =getKey(HttpServletRequestFactory.create(url,parameters));  
	                   if(key != null){  
	                            FileUtil.remveFile(key2File(key));  
	                   }  
	         }  
	}  

书中给到的另外一个客户端缓存，不知道该归那类：


	/** 
	 * 网页静态资源（gif图像，css资源的缓存时间设置filter，避免频繁请求静态资源 
	 */  
	package com.zsl.cache.filter;  
	   
	import java.io.IOException;  
	import java.util.Enumeration;  
	import java.util.Map;  
	   
	importjavax.servlet.FilterChain;  
	importjavax.servlet.FilterConfig;  
	importjavax.servlet.ServletException;  
	import javax.servlet.Filter;  
	importjavax.servlet.ServletRequest;  
	importjavax.servlet.ServletResponse;  
	importjavax.servlet.http.HttpServletRequest;  
	importjavax.servlet.http.HttpServletResponse;  
	   
	importorg.apache.commons.collections.map.HashedMap;  
	importorg.apache.commons.logging.Log;  
	importorg.apache.commons.logging.LogFactory;  
	   
	   
	/** 
	 * @author zsl 
	 * 
	 */  
	public class ExpireFilterimplements Filter {  
	   
	         private Log log = LogFactory.getLog(ExpireFilter.class);  
	          
	         private Map<String, Long> map = new HashedMap();  
	   
	         @Override  
	         public void destroy() {  
	                   log.info("destory ExpiredFilter");  
	         }  
	   
	         @Override  
	         public void doFilter(ServletRequest request, ServletResponseresponse,  
	                            FilterChain chain) throws IOException,ServletException {  
	                            String uriString =((HttpServletRequest)request).getRequestURI();  
	                            int n = uriString.lastIndexOf('.');  
	                            if(n!= -1){  
	                                     String ext =uriString.substring(n);  
	                                     Long exp = map.get(ext);  
	                                     if(exp != null){  
	                                               HttpServletResponseresp = (HttpServletResponse)response;  
	                                               resp.setHeader("Expires",System.currentTimeMillis()+exp*1000+"");  
	                                     }  
	                            }  
	                            chain.doFilter(request,response);  
	         }  
	   
	         @Override  
	         public void init(FilterConfig config) throwsServletException {  
	                   Enumeration em = config.getInitParameterNames();  
	                   while(em.hasMoreElements()){  
	                            String paramName =em.nextElement().toString();  
	                            String paramValue = config.getInitParameter(paramName);  
	                            try {  
	                                     int time =Integer.valueOf(paramValue);  
	                                     if(time>0){  
	                                               log.info("set"+paramName + " expired seconds: "+time);  
	                                               map.put(paramName, newLong(time));  
	                                     }  
	                            } catch (Exception e) {  
	                                     log.warn("Exception ininitilizing ExpiredFilter.",e);  
	                            }  
	                   }  
	         }  
	}  


2.     负载均衡系统
Ø  负载均衡策略
负载均衡策略有随机分配，平均分配，分布式一致性hash等。随机分配就是通过随机数选择一个服务器来服务。平均分配就是一次循环分配一次。分布式一致性hash算法，比较负载，把资源和节点映射到一个换上，然后通过一定的算法资源对应到节点上，使得添加和去掉服务器变得非常容易，减少对其它服务器的影响。很有名的一个算法，据说是P2P的基础。了解不是很深，就不详细说了，要露马脚了。
Ø  软件负载均衡
软件负载均衡可以采用很多方案，常见的几个方案有：
基于DNS的负载均衡，通过DNS正向区域的配置，将一个域名根据一定的策略解析到多个ip地址，实现负载均衡，这里需要DNS服务器的配合。
基于LVS的负载均衡。LVS可以将多个linux服务器做成一个虚拟的服务器，对外提供服务器，实现负载均衡。
基于Iptables的负载均衡。Iptables可以通过做nat，对外提供一个虚拟IP，对内映射到多个服务器实现负载均衡。基本上可以和硬件均衡方案一致了，这里的linux服务器相当于一台路由器。
Ø  硬件负载均衡
基于路由器的负载均衡，在路由器上配置nat实现负载均衡。对外网一个虚拟IP，内网映射几个内网IP。
一些网络设备厂商也提供了一些负载均衡的设备，如F5，不过价格不菲哦。
数据库的负载均衡
数据库的负载均衡可以是数据库厂商提供的集群方案。
――――――――――――――――――――――――――――――――――――――
今天先写到这里，这个题目太大了，东西太多。后边是将来要写的。还没有组织出来。
好多东西也没有展开。东西太多了。
――――――――――――――――――――――――――――――――――――――
分布式文件系统
Gfs
hfs
Map Reduce系统
 
云计算

	


转自：http://blog.csdn.net/shatty/article/details/6629896