# StarveFree-Readers-Writers-Problem
The readers-writers problem is a classical one in computer science:
we have a resource that can be accessed by readers, who do not modify the resource, and writers, who can modify the resource. When a writer is modifying the resource, no-one else can access it at the same time: another writer could corrupt the resource, and another reader could read a partially modified value.

Readers: who want to read the shared resources.

Writers: who want to modify the shared resources. 

There are three variations to this problem.

## First readers–writers problem
### Give readers priority: 
When there is at least one reader currently accessing the resource, allow new readers to access it as well. This can cause starvation if there are writers waiting to modify the resource and new readers arrive all the time, as the writers will never be granted access as long as there is at least one active reader.

## Second readers–writers problem
### Give writers priority: 
Here, readers may starve because this problem is also similar to the first problem. The writers are given preference in this scenario. The readers must wait until the last writer exits the critical section by modifying the resource and release the lock and allow readers to access the resource.

## Third readers–writers problem
### Give neither priority: 
All readers and writers will be granted access to the resource in their order of arrival. If a writer arrives while readers are accessing the resource, it will wait until those readers free the resource, and then modify it. New readers arriving in the meantime will have to wait.
### Semaphore
Designing a Semaphore with FIRST-IN-FIRST-OUT(FIFO) queue to maintain the list of blocked processes
```cpp
// The code for a FIFO semaphore.
struct Semaphore{
  int value = 1;
  FIFO_Queue* Q = new FIFO_Queue();
}
    
void wait(Semaphore *S,int* process_id){
  S->value--;
  if(S->value < 0){
  S->Q->push(process_id);
  block(); //this function will block the process by using system call and will transfer it to the waiting queue
           //the process will remain in the waiting queue till it is waken up by the wakeup() system calls
           //this is a type of non busy waiting
  }
}
    
void signal(Semaphore *S){
  S->value++;
  if(S->value <= 0){
  int* PID = S->Q->pop();
  wakeup(PID); //this function will wakeup the process with the given pid using system calls
  }
}


//The code for the queue which will allow us to make a FIFO semaphore.
struct FIFO_Queue{
    ProcessBlock* front, rear;
    int* pop(){
        if(front == NULL){
            return -1;            // Error : underflow.
        }
        else{
            int* val = front->value;
            front = front->next;
            if(front == NULL)
            {
                rear = NULL;
            }
            return val;
        }
    }
    void* push(int* val){
        ProcessBlock* blk = new ProcessBlock();
        blk->value = val;
        if(rear == NULL){
            front = rear = n;
            
        }
        else{
            rear->next = blk;
            rear = blk;
        }
    }
    
}

// A Process Block.
struct ProcessBlock{
    ProcessBlock* next;
    int* process_block;
}
```
### Intialization
In this solution 3 semaphores are used. turn semaphore is used to specific whose chance is it nextto enter the critical section.  The process holding this semaphore gets the next chance to enter thecritical section.rwtsemaphore is the semaphore required to access the critical section.rmutexsemaphore is required to change thereadcountvariable which maintain the number of activereader.
```cpp
int read_count = 0;                      //Integer representing the number of reader executing critical section
Semaphore turn = new Semaphore();        //seamphore representing the order in which the writer and 
                                         //reader are requesting access to critical section
Semaphore rwt = new Semaphore();         //seamphore required to access the critical section
Semaphore r_mutex = new Semaphore();     //seamphore required to change the read_count variable
```
### Reader's Code
```cpp
do{
/* ENTRY SECTION */
       wait(turn,process_id);              //waiting for its turn to get executed
       wait(r_mutex,process_id);           //requesting access to change read_count
       read_count++;                       //update the number of readers trying to access critical section 
       if(read_count==1)                   // if I am the first reader then request access to critical section
         wait(rwt);                        //requesting  access to the critical section for readers
       signal(turn);                       //releasing turn so that the next reader or writer can take the token
                                           //and can be serviced
       signal(r_mutex);                    //release access to the read_count
/* CRITICAL SECTION */
       
/* EXIT SECTION */
       wait(r_mutex,process_id)            //requesting access to change read_count         
       read_count--;                       //a reader has finished executing critical section so read_count decrease by 1
       if(read_count==0)                   //if all the reader have finished executing their critical section
        signal(rwt);                       //releasing access to critical section for next reader or writer
       signal(r_mutex);                    //release access to the read_count  
       
/* REMAINDER SECTION */
       
}while(true);
```
### Writer's code
```cpp
do{
/* ENTRY SECTION */
      wait(turn,process_id);              //waiting for its turn to get executed
      wait(rwt,process_id);               //requesting  access to the critical section
      signal(turn,process_id);            //releasing turn so that the next reader or writer can take the token
                                          //and can be serviced
/* CRITICAL SECTION */

/* EXIT SECTION */
      signal(rwt)                         //releasing access to critical section for next reader or writer

/* REMAINDER SECTION */

}while(true);
```
## Correctness of Solution
### Mutual Exclusion
The rwt semaphore ensures that only a single writer can access the critical section at any moment of time thus ensuring mutual exclusion between the writers and also when the first reader try to access the critical section it has to acquire the rwt mutex lock to access the critical section thus ensuring mutual exclusion between the readers and writers.
### Bounded Waiting
Before accessing the critical section any reader or writer have to first acquire the turn semaphore which uses a FIFO queue for the blocked processes. Thus as the queue uses a FIFO policy, every process has to wait for a finite amount of time before it can access the critical section thus meeting the requirement of bounded waiting.
### Progress Requirement
The code is structured so that there are no chances for deadlock and also the readers and writers takes a finite amount of time to pass through the critical section and also at the end of each reader writer code they release the semaphore for other processes to enter into critical section.
## Explanation of the Code

We will be using three semaphores:

1. rMutex --> this semaphore will be used for updating the readers count. So, it would only be available to readers method.

2. rwt --> This semaphore would either be in control of the reader or the writer. If the readers are reading and the writers are trying to access the critical section, it would get blocked and vice versa. However, if one of the readers is reading and another reader tries to access, then there won't be any problem. This semaphore gets updated in three instances. First, when the first reader arrives. Secondly, when the last reader left the critical section. And lastly, when any writer writes to the resource.

3. turn --> This semaphore is used at the start of the entry section of readers/writers code. It checks whether there is a process already waiting for its turn. If so, it gets blocked. If not, then it can access the semaphore, and no new reader or writer could now execute before this process. Therefore, it helps in preserving the order (turn) of the process.

We will also be using a variable countRead to update the number of readers at a particular time.
For the read method we would first call wait for order and rMutex. If any process is already in queue order would be one and thus calling process would be blocked. Otherwise, it would make order "1". rMutex would check that no other process is updating the readers count. If the reader count was 0, then don't allow the writer to access the critical section. After countRead is updated, both the semaphores are released. After reading of resource, countRead is decremented by getting hold of rMutex. If countRead is 0, writers can access the critical section.
For write method, we would first check the turn semaphore and then writer would get access with the rwt semaphore. Since, the turn would be preserved we could release the turn semaphore. Then writers modifies the resourcse and finally release rwt.
In this way, no process would starve, and our objective is accomplished.;;
