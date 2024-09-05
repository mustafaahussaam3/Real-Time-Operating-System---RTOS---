# Real Time Operating System ( RTOS ) 

 Third part of Eng Mohamed Tarek Advanced Embedded Course that work with RTOS.

- Unlike background/foreground systems that is less complex, freeRTOS is task based system which controlling complex systems using several task. 
- Each task think that it own's the processor, but actually they all own the processor but in multitasking procedure buy controlling this through our tick. 
- Each tick call the schedular and check for the tasks in the ready state, if a task has a higher priority it win the race and enter the run state.
- In this part we will study all the features of the real time operating system include: 
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
### Lab1: Multitasking with FreeRTOS (Serial)

This introductory lab demonstrates the idea of multitasking with FreeRTOS, The setup involves two tasks, vPeriodicTask1 and vPeriodicTask2, both having equal priority. FreeRTOS smoothly switches between these tasks, enabling each to print its message "Task x is running\r\n" through UART in a loop every 1 second.

```c 
xTaskCreate(vPeriodicTask1, "Task 1", 256, NULL, 1,NULL);    
xTaskCreate(vPeriodicTask2, "Task 2", 256, NULL, 1, NULL);`
``` 
- Creating two tasks with the same priority with preemptive schedule, that will check the round robin case, if it true as the default it will gives each task the same time.
- In this case, the last task created is the first task runs in the schedular.
```c 
void vPeriodicTask2(void *pvParameters)
{
    for (;;)
    {
        Delay_MS(1000);
        UART0_SendString("Task 2 is running\r\n");
    }
}
```

```c 
void vPeriodicTask1(void *pvParameters)
{
    for (;;)
    {
        UART0_SendString("Task 1 is running\r\n");
        Delay_MS(1000);
    }
}
```
- In this case, Task2 runs first and it will consume 10ms from it's delay, then the schedular will switch to task 1 that will send " Task 1 is running " (if the 10 ms is suitabke for sending it !) then the processor switch back to task 2 and conume other 10ms, and it will switching forth and back for 2 seconds, after that it will display "Task2 is running" then "Task 1 is running" and repeat itself. 

### Lab2: Introduction to Multitasking with FreeRTOS (LEDs)
It is required to implement a FreeRTOS app which consists of two tasks, vPeriodicGreenLedTask and vPeriodicRedLedTask, which are implemented with equal priority. The scheduler efficiently alternates between these tasks, resulting in a constant cycle where each task blinks an LED for one second. The vPeriodicGreenLedTask toggles the green LED, while the vPeriodicRedLedTask toggles the red LED.

```c
void vPeriodicGreenLedTask(void *pvParameters)
{
    for (;;)
    {
        GPIO_GreenLedToggle();
        Delay_MS(500);
    }
}

void vPeriodicRedLedTask(void *pvParameters)
{
    for (;;)
    {
        Delay_MS(500);
        GPIO_RedLedToggle();
    }
}
```
- Same as the last code but we change function impentation.

### Lab3: Passing parameters to tasks (1st serial example)
This example serves as an introduction to the concept of passing parameters to tasks using FreeRTOS's xTaskCreate function. The application features a single task, vPeriodicTask, which is responsible for periodically sending a predefined text, "Task 1 is running\r\n," via UART every second. The task creation involves passing the text as a parameter to the task through pvParameters.

```c
xTaskCreate(  vPeriodicTask, "Task 1", 256, "Task 1 is running\r\n" , 1, NULL);

void vPeriodicTask(void *pvParameters)
{
    for (;;)
    {
        UART0_SendString((const uint8*)pvParameters);
        Delay_MS(1000);
    }
}
```
- We will type cast what we send because the parameter is a generic pointer.

### Lab4: Passing parameters to tasks (2nd serial example)
In this lab, It’s required to implement 2 tasks ("Task 1" and "Task 2") of the same priority sharing the same task body (function) but for varied behavior, each task shall print its message like "Task 1 is running\r\n" and "Task 2 is running\r\n" sent over UART every second.

```c 
xTaskCreate( vPeriodicTask, "Task 2", 256, (void* pcTextForTask2, 1, NULL );

xTaskCreate( vPeriodicTask, "Task 1", 256, (void*)pcTextForTask1, 1, NULL );

void vPeriodicTask(void *pvParameters)
{
    for (;;)
    {
        UART0_SendString(( uint8 * ) pvParameters);
        Delay_MS(1000);
    }
}
```

- This code will share the same function between the two tasks and also UART, but it differentiate between the two task throught the task parameter. 
- The output will not be correct, but we will learn how to correct it later on.

### Lab5: Passing multiple parameters to task (LEDs with serial)
In the lab, it is required to advance the concept of task parameterization by sending multiple task information containing both the string to be printed and the LED to be toggled. Tasks "Task 1" and "Task 2" share the same task body but now receive 2 pieces of data, allowing differentiation in both messages and LED controls. The FreeRTOS scheduler efficiently switches between tasks, observing the green and red LEDs toggling in addition to alternating messages sent via UART every second.

```c
/* Create a struct which consist of information to the function */
typedef struct {
    const uint8 *pcParam;
    uint8 ledNumber;
}Configurations_t;

Configurations_t Task1 = { "Task1 is running \r\n" , GREEN};
Configurations_t Task2 = { "Task2 is running \r\n" , RED}; 
```
```c
/* Create the two functions and pass the address of the structs */
xTaskCreate( vPeriodicTask, "Task 1", 256, (void*)&Task1, 1, NULL );
xTaskCreate( vPeriodicTask, "Task 2", 256, (void*)&Task2, 1, NULL );
```
```c
/* typecast the struct and switch between the structs */
void vPeriodicTask(void *pvParameters)
{
  Configurations_t *Task = (Configurations_t*)pvParameters;

    for (;;)
    {
        switch (Task->ledNumber)
        {
            case GREEN:
                         Delay_MS(500);
                         UART0_SendString(Task->pcParam);
                         GPIO_GreenLedToggle();
                         break;
            case   RED:
                        UART0_SendString(Task->pcParam);
                        GPIO_RedLedToggle();
                        Delay_MS(500);
                        break;
        }

    }
}
```
## Idle Task 
- Is a special task that is created by the schedular to be called when there is no other tasks running on the CPU.
- There must be at least one task running so, the idle task solve this issue.
- It puts the CPU at low-power mode to conserve energy or executing simple loop that continuously checks for the availability of other tasks.
- It has the lowest priotity (zero), so we start our tasks priority with 1.
- vTaskDelay enters blocking state until it timeout, allow other tasks to run.

### Lab6: FreeRTOS Delay & Idle Task
This lab is to explain the concept of using vTaskDelay as an alternative to traditional blocking delays. This example serves as an introduction to the efficient management of task delays within FreeRTOS. Additionally, the lab illustrates the idea of the idle task in the overall system. Tasks, represented by " vPeriodicGreenLedTask" and " vPeriodicRedLedTask," utilize vTaskDelay to achieve dynamic LED toggling between green and red LEDs using non-blocking delays of 1sec.

- add this to configuration file to use v task delay API 
```c
#define INCLUDE_vTaskDelay                      1
```

## Sheduling Algorithms 

- Free RTOS Provide 3 several algorithms

   1- Preemptive Scheduling with Time Slicing. 

   2- Preemptive Scheduling without Time Slicing.

   3- Cooperative Scheduling.

### Lab7: Fixed Priority Preemptive Scheduling with Time Slicing
This example demonstrates the first scheduling type offered by FreeRTOS - Fixed Priority Preemptive Scheduling with Time Slicing. The application comprises three tasks, each assigned a specific priority - "Task 1" with a priority of 2, and "Task 2" and "Task 3" with a priority of 1. As "Task 1" starts, it prints its string and then enters a non-blocking delay of 2 seconds using vTaskDelay. This delay allows the two lower-priority tasks to execute in a time-sliced manner, each running for 1 second before returning to "Task 1."

- In this Example Task 1 start first, then it will enters the block state due to vTaskDelay, then the processor will share the processor between task2 and 3 until task 1 finishes. 
- The last one entered the schedule will be the first to run on the time slice and this is unlike preemptive without time slicing.

```c
    xTaskCreate( Task1 , "Task 1", 256, NULL, 2, NULL );
	xTaskCreate( Task2 , "Task 2", 256, NULL, 1, NULL );
    xTaskCreate( Task3 , "Task 3", 256, NULL, 1, NULL );
```

### Lab8: Fixed Priority Preemptive Scheduling without Time Slicing
This example demonstrates the second scheduling type offered by FreeRTOS - Fixed Priority Preemptive Scheduling without Time Slicing. The application is the same as the previous lab which comprises three tasks, each assigned a specific priority - "Task 1" with a priority of 2, and "Task 2" and "Task 3" with a priority of 1. As "Task 1" initiates, it prints its message and introduces a delay of 2 seconds using vTaskDelay. This delay ensures that "Task 2" and "Task 3," both with lower priorities, execute one after the other, rather than in a time-sliced manner.

- You need to add this to configuration file to use without time slice.

```c
#define configUSE_TIME_SLICING                (0)
```

- In this example, Task1 run then enter the block state for 2 seconds, in this time task2 ( which is entered the schedular first before task 3 ) will enters the running state, and after task 1 interrupted it and finishes, task3 will run. 

### Lab9: Fixed Priority Cooperative Scheduling
This example demonstrates the third scheduling type offered by FreeRTOS - Fixed Priority Cooperative Scheduling. The application is the same as the previous lab which comprises three tasks, each assigned a specific priority - "Task 1" with a priority of 2, and "Task 2" and "Task 3" with a priority of 1.

- Change this configurations

```c
#define configUSE_PREEMPTION                  (0)
#define configUSE_TIME_SLICING                (1)
```

- In this Example, Task1 will run as it's the higher priority.
- then it enters the block state.
- task2 will enters now and run infinitely because it will not leave the processor.

## Task Yield

- We use Task Yield with Cooperative scheduling to make the task leave the CPU. 
- When it leaves the schedular checks for a task with higher priority or same priority, if there no higher priority task, it will take the processor another time. 
- No effect if task yield if the preemption equal one. 
- Task Yield Simply triggers pendSV for context switch.

## Task Suspend and Resume 

- Task Can suspend itself or any other task during runtime.
- The Task will be in the suspended state until it explicitly resumed.
- We can enter Suspende state from any other state.
- If task want to suspend itself it has to send NULL to the Handle 

To use Task Suspend and Resume we have to add this to the Configurations file
```c
#define INCLUDE_vTaskSuspend 1
```

## Task Priority Set and Get APIs

- This APIs allow you to set and get the task priority dynamically during runtime.
- The task can query it's task by sending NULL to the Handle
- This configuration must set to on, to use the APIs 
```c
#define INCLUDE_uxTaskPriorityGet  1 
#define INCLUDE_vTaskPrioritySet   1 
```
- A context switch will occur before the function returns if the priority being set is higher than the currently executing task.
- Priority must be less than configMax_PRIORITIES-1

## vTaskDelay vs vTaskDelayUntil APIs

- vTaskDelay time is relative to the time it enters the task.
- But, vTaskDelayUntil is delays the task until a specific absolute time. it allows the tasks to execute periodically throught a fixed time intervels.
- You must define this in the configurations to be able to use this API.
```c
#define INCLUE_vTaskDelayUntil  1
```
- Has two parameters:

    1- pxPerviousWakeTime: Hold the time at which task left the blocked state.
    - Used as reference time to calculate the time at which task should next leave the blocked state.

    2- xTimeIncrement: Specifies the time in Ticks.

    ![image](<Images/vTaskDelayUntil.png>)

## Delete a Task
- Task can delete itself or any other task throught vTaskDelete() Api
- To use API you Have to Include this in configurations.
```c 
#define INCLUDE_vTaskDelete  1
```
- It will removes the task from freeRTOS Kernel Management.
- NOTE: it must be essential that the idle task is not starved of processing time. This because the Idle task is responsible for cleaning up kernel resources after a task has been deleted.

## Compile Time Configurations

- System can be dynamically configured, statially configured, dynamically and statically.

- Dyanamic System:
```c
configSUPPORT_STATIC_ALLOCATION  0 (default)
configSUPPORT_DYNAMIC_ALLOCATION 1 (default)
```
- Static System:
```c 
configSUPPORT_STATIC_ALLOCATION  1
configSUPPORT_DYNAMIC_ALLOCATION 0
```
- Mixed System: 
```c
configSUPPORT_STATIC_ALLOCATION  1
configSUPPORT_DYNAMIC_ALLOCATION 1
```
- In this manner, you will able to create your own stack for TCB and Task through puxStackBuffer and pcTaskBuffer.
