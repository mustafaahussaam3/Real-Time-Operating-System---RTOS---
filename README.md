# Real Time Operating System ( RTOS ) 
 Third part of Eng Mohamed Tarek Advanced Embedded Course that work with RTOS.

- In this part we will study all the features of the real time operating system include: 

- Unlike background/foreground systems that is less complex, freeRTOS is task based system which controlling complex systems using several task. 
- Each task think that it own's the processor, but actually they all own the processor but in multitasking procedure buy controlling this through our tick. 
- Each tick call the schedular and check for the tasks in the ready state, if a task has a higher priority it win the race and enter the run state.

 1. Memory Management.
 2. Task Management. 
 3. Tick and Time Management.
 4. Synchronization and Communication.
    - Event Groups.
    - Semaphore and Mutex.
    - Queue Management.
 5. Interrupt Management.
 6. Software Timers.
 7. Hook Functions (Hook Routines).

 ## Memory Management 
 - FreeRTOS provides 5 main memory schemes to control RTOS objects (like Tasks) to create memory dynamically during run time ( and also statically during compile time).
 - Each scheme has it's property and we choose the suitable for our project. 

 ### Heap_1 
- It is the simplest.
- Does not have the free features so, the memory which is accessed by the task stack does not freed.  ![image](<Images/Heap_1.png>)

### Heap_2
- In heap 2 we can free the memory but it does not coalescence adjacent free blocks.
- We can add only the same size of the deleted task.
- Not recommended and kept for backward commutability.
![image](<Images/Heap_2.png>)

### Heap_4

- In heap 4 we can free the memory and also there is coalescence adjacent free blocks to avoid fragmentation faults.
- We can add any size of task or queue in the deleted memory section.

![image](<Images/Heap_4.webp>)

### Heap_5

- Same as Heap_4 with the ability to span the heap across multiple non-adjacent memory areas.
- used if we have several rams.
![image](<Images/Heap_5.png>)

### Heap_3
- It's the actual C layout and doesn't use RTOS Feature.
- wraps to standard malloc() and free() for thread safety.
- Use critical section to avoid the multithread corruption.
- Tasks created dynamically in Heap and does not created in Global section.
![image](<Images/Heap_3.png>)

## Task Management
The second feature in freeRTOS, that involve to creating, scheduling, and managing tasks, which is the basic unit of exexution in the system.

- Task has several states 
1. Ready : it the state after creating task.
2. Running : state that enters the higher priority task to run on the CPU.
3. Blocked : task wait for some event or to synchronize with other task to happen. (ex: Queue is full or empty, wait for resourse to be available)

![image](<Images/Task States.jpg>)

### Task Creation 
Task is a simple C function that return void and has a parameter void pointer 
```c
void vTaskFunction(void *pvParameters);
```
### Task Creation API
We use ``` xTaskCreate()``` API to create a new task and add it to the list of tasks that are ready to run.
```c
BaseType_t xTaskCreate( 
 TaskFunction_t pxTaskCode,
 const char * const pcName,
 const configSTACK_DEPTH_TYPE usStackDepth,
 void * const pvParameters,
 UBaseType_t uxPriority,
 TaskHandle_t * const pxCreatedTask )
```
### ![Lab1: Multitasking with FreeRTOS (Serial)](<>)

This introductory lab demonstrates the idea of multitasking with FreeRTOS, The setup involves two tasks, vPeriodicTask1 and vPeriodicTask2, both having equal priority. FreeRTOS smoothly switches between these tasks, enabling each to print its message "Task x is running\r\n" through UART in a loop every 1 second.
