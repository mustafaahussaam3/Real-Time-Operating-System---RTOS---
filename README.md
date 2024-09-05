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

### Lab6: FreeRTOS Delay & Idle Task
This lab is to explain the concept of using vTaskDelay as an alternative to traditional blocking delays. This example serves as an introduction to the efficient management of task delays within FreeRTOS. Additionally, the lab illustrates the idea of the idle task in the overall system. Tasks, represented by " vPeriodicGreenLedTask" and " vPeriodicRedLedTask," utilize vTaskDelay to achieve dynamic LED toggling between green and red LEDs using non-blocking delays of 1sec.


