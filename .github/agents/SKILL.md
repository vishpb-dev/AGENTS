---
name: freertos-task
description: Create, modify, or debug FreeRTOS tasks. Use when adding new RTOS tasks, configuring priorities, implementing task synchronization, or troubleshooting concurrency issues.
---

# FreeRTOS Task Implementation

This skill guides you through creating and managing FreeRTOS tasks in the SBS2 system.

## When to Use This Skill

- Creating a new FreeRTOS task
- Modifying task priorities or stack sizes
- Implementing inter-task communication (queues, mutexes, semaphores)
- Debugging task scheduling or timing issues
- Adding software timers
- Troubleshooting concurrency problems

## Key Files

- `Core/Src/kcesbs_tasks.c` - Task creation and management
- `Core/Inc/kcesbs_tasks.h` - Task declarations and handles
- `Core/Src/freertos.c` - RTOS initialization
- `Core/Inc/FreeRTOSConfig.h` - RTOS configuration
- Task implementation files (e.g., `adc.c` for ADC task)

## Current Task Architecture

### Existing Tasks

| Task Name | Priority | Stack (words) | API | Purpose |
|-----------|----------|---------------|-----|---------|
| defaultTask | Normal (2) | 256 | CMSIS-RTOS2 | Main control loop |
| adc_task | 4 | 1024 | Native FreeRTOS | ADC data processing |
| IdleTask | 0 | 128 | FreeRTOS kernel | Idle hook |
| TimerTask | 2 | 256 | FreeRTOS kernel | Software timers |

### Priority Guidelines

```
Priority 0:  Idle (kernel)
Priority 1:  Low priority background tasks
Priority 2:  Normal (defaultTask, TimerTask)
Priority 3:  Above normal
Priority 4:  High (adc_task)
Priority 5+: Critical/ISR-level tasks (avoid unless necessary)
```

**Rule**: Higher number = higher priority. ISRs should use priority 5 for NVIC to work with FreeRTOS API.

## Creating a New Task

### Step 1: Define Task Function

```c
// In your module .c file (e.g., new_module.c)
#include "FreeRTOS.h"
#include "task.h"
#include "queue.h"

// Task handle (static for encapsulation)
static TaskHandle_t new_module_task_handle = NULL;

// Task function
static void new_module_task(void *argument) {
    // Initialize module-specific resources
    module_init_resources();
    
    // Task main loop
    for (;;) {
        // Wait for event/data (blocking call)
        uint32_t notification = ulTaskNotifyTake(pdTRUE, portMAX_DELAY);
        
        // Process event
        if (notification) {
            process_module_event();
        }
        
        // Alternative: Use delay for periodic execution
        // vTaskDelay(pdMS_TO_TICKS(100)); // 100 ms delay
    }
}
```

### Step 2: Create Task in Initialization

**Using Static Allocation** (Preferred - predictable memory):
```c
// In kcesbs_tasks.c

// Static task buffer and stack
static StaticTask_t new_module_task_buffer;
static StackType_t new_module_task_stack[512]; // 512 words = 2048 bytes

void create_new_module_task(void) {
    new_module_task_handle = xTaskCreateStatic(
        new_module_task,              // Task function
        "NewModuleTask",              // Name (for debugging)
        512,                          // Stack size in words
        NULL,                         // Task parameter
        4,                            // Priority (same as ADC task)
        new_module_task_stack,        // Stack buffer
        &new_module_task_buffer       // Task buffer
    );
    
    if (new_module_task_handle == NULL) {
        // Task creation failed
        Error_Handler();
    }
}
```

**Using Dynamic Allocation** (Less preferred - heap fragmentation risk):
```c
void create_new_module_task(void) {
    BaseType_t result = xTaskCreate(
        new_module_task,              // Task function
        "NewModuleTask",              // Name (for debugging)
        512,                          // Stack size in words
        NULL,                         // Task parameter
        4,                            // Priority
        &new_module_task_handle       // Task handle pointer
    );
    
    if (result != pdPASS) {
        // Task creation failed
        Error_Handler();
    }
}
```

### Step 3: Register Task Creation

Add to module public API:
```c
// In new_module.h
void new_module_init(void);
TaskHandle_t new_module_get_task_handle(void);
```

```c
// In new_module.c
void new_module_init(void) {
    // Create supporting objects (queues, mutexes, etc.)
    // ... 
    
    // Create task
    create_new_module_task();
}

TaskHandle_t new_module_get_task_handle(void) {
    return new_module_task_handle;
}
```

Call from main initialization:
```c
// In main.c, before osKernelStart()
void MX_FREERTOS_Init(void) {
    // Existing initialization
    adc_init();
    
    // New module initialization
    new_module_init();
}
```

## Inter-Task Communication

### Queues (Producer-Consumer Pattern)

**Example: ADC Task Queue**

Creating a queue:
```c
// In module .c file
static QueueHandle_t module_queue = NULL;

void module_init(void) {
    // Create queue: 10 items, each uint32_t
    module_queue = xQueueCreate(10, sizeof(uint32_t));
    
    if (module_queue == NULL) {
        Error_Handler();
    }
}
```

Sending to queue (from ISR):
```c
void DMA_ISR_Handler(void) {
    BaseType_t xHigherPriorityTaskWoken = pdFALSE;
    uint32_t message = EVENT_DATA_READY;
    
    // Send from ISR - use FromISR variant
    xQueueSendFromISR(module_queue, &message, &xHigherPriorityTaskWoken);
    
    // Yield if higher priority task was woken
    portYIELD_FROM_ISR(xHigherPriorityTaskWoken);
}
```

Receiving from queue (in task):
```c
void module_task(void *argument) {
    uint32_t message;
    
    for (;;) {
        // Block indefinitely waiting for message
        if (xQueueReceive(module_queue, &message, portMAX_DELAY) == pdTRUE) {
            // Process message
            process_event(message);
        }
    }
}
```

### Mutexes (Shared Resource Protection)

**Example: ADC Results Mutex**

Creating a mutex:
```c
// In module .c file
static SemaphoreHandle_t module_mutex = NULL;
static StaticSemaphore_t module_mutex_buffer; // For static allocation

void module_init(void) {
    // Create recursive mutex (static allocation)
    module_mutex = xSemaphoreCreateRecursiveMutexStatic(&module_mutex_buffer);
    
    if (module_mutex == NULL) {
        Error_Handler();
    }
}
```

Using a mutex:
```c
// Writing to shared resource (producer)
void update_shared_data(void) {
    // Take mutex (blocks if already taken)
    if (xSemaphoreTakeRecursive(module_mutex, portMAX_DELAY) == pdTRUE) {
        // Critical section - modify shared data
        shared_data.value = new_value;
        shared_data.timestamp = get_timestamp();
        
        // Release mutex
        xSemaphoreGiveRecursive(module_mutex);
    }
}

// Reading from shared resource (consumer)
void read_shared_data(void) {
    uint32_t local_copy;
    
    if (xSemaphoreTakeRecursive(module_mutex, pdMS_TO_TICKS(100)) == pdTRUE) {
        // Critical section - read shared data
        local_copy = shared_data.value;
        
        xSemaphoreGiveRecursive(module_mutex);
        
        // Use local copy outside critical section
        process_data(local_copy);
    }
}
```

### Task Notifications (Lightweight Signaling)

**Faster than queues for simple notifications**

Sending notification:
```c
// From ISR
void Timer_ISR_Handler(void) {
    BaseType_t xHigherPriorityTaskWoken = pdFALSE;
    
    // Notify task (increment notification value)
    vTaskNotifyGiveFromISR(target_task_handle, &xHigherPriorityTaskWoken);
    
    portYIELD_FROM_ISR(xHigherPriorityTaskWoken);
}

// From task
void notify_other_task(void) {
    xTaskNotifyGive(target_task_handle);
}
```

Receiving notification:
```c
void task_function(void *argument) {
    for (;;) {
        // Wait for notification, clear on exit
        uint32_t notification_value = ulTaskNotifyTake(pdTRUE, portMAX_DELAY);
        
        if (notification_value > 0) {
            // Process notification
            handle_event();
        }
    }
}
```

## Software Timers

Creating a software timer:
```c
static TimerHandle_t module_timer = NULL;

void timer_callback(TimerHandle_t xTimer) {
    // Timer expired - perform action
    trigger_periodic_action();
}

void module_init(void) {
    // Create one-shot timer: 5 seconds
    module_timer = xTimerCreate(
        "ModuleTimer",                // Name
        pdMS_TO_TICKS(5000),         // Period (5000 ms)
        pdFALSE,                     // Auto-reload: false (one-shot)
        NULL,                        // Timer ID
        timer_callback               // Callback function
    );
    
    if (module_timer == NULL) {
        Error_Handler();
    }
}

void start_timer(void) {
    xTimerStart(module_timer, 0);
}

void stop_timer(void) {
    xTimerStop(module_timer, 0);
}
```

For periodic timers, set auto-reload to `pdTRUE`:
```c
module_timer = xTimerCreate(
    "PeriodicTimer",
    pdMS_TO_TICKS(1000),            // 1 second period
    pdTRUE,                         // Auto-reload: true (periodic)
    NULL,
    timer_callback
);
```

## ISR Safety

### Rules for ISR Context

1. **Use `FromISR` variants only**
```c
// ✓ Correct
xQueueSendFromISR(queue, &data, &xHigherPriorityTaskWoken);
xSemaphoreGiveFromISR(semaphore, &xHigherPriorityTaskWoken);
vTaskNotifyGiveFromISR(task_handle, &xHigherPriorityTaskWoken);

// ✗ Wrong - will cause crashes
xQueueSend(queue, &data, 0);
xSemaphoreGive(semaphore);
xTaskNotifyGive(task_handle);
```

2. **Yield if necessary**
```c
void ISR_Handler(void) {
    BaseType_t xHigherPriorityTaskWoken = pdFALSE;
    
    // ISR operations...
    xQueueSendFromISR(queue, &data, &xHigherPriorityTaskWoken);
    
    // Yield to higher priority task if it was woken
    portYIELD_FROM_ISR(xHigherPriorityTaskWoken);
}
```

3. **Keep ISRs short**
```c
// ✓ Good - minimal work in ISR
void DMA_ISR_Handler(void) {
    BaseType_t xHigherPriorityTaskWoken = pdFALSE;
    
    // Quick notification
    vTaskNotifyGiveFromISR(processing_task, &xHigherPriorityTaskWoken);
    
    portYIELD_FROM_ISR(xHigherPriorityTaskWoken);
}

// ✗ Bad - heavy processing in ISR
void DMA_ISR_Handler(void) {
    // Don't do this!
    process_all_data();
    calculate_results();
    update_display();
}
```

## Stack Size Calculation

### Guidelines

**Minimum Stack Sizes**:
- Simple tasks: 128 words (512 bytes)
- Normal tasks: 256 words (1024 bytes)
- Complex tasks: 512-1024 words (2-4 KB)

**Factors Affecting Stack Size**:
- Local variables and arrays
- Function call depth
- ISR stack usage (if task can be preempted)
- printf/sprintf usage (very stack-hungry)

### Stack Overflow Detection

Enable in `FreeRTOSConfig.h`:
```c
#define configCHECK_FOR_STACK_OVERFLOW  2  // Method 2 (most thorough)
```

Implement stack overflow hook:
```c
// In main.c or kcesbs_tasks.c
void vApplicationStackOverflowHook(TaskHandle_t xTask, char *pcTaskName) {
    // Stack overflow detected!
    DEBUG_TRACE(DEBUG_CATEGORY_ERROR, 
               timestamp,
               "Stack overflow in task: %s",
               pcTaskName);
    
    // Halt or reset
    Error_Handler();
}
```

## Task Priority Selection

### Priority Assignment Strategy

**Priority 4 (High)**: Time-critical tasks
- ADC data processing (existing)
- High-speed communication handlers
- Safety-critical monitoring

**Priority 2-3 (Normal-High)**: Regular application tasks
- Main control loop (defaultTask at Normal/2)
- User interface tasks
- Medium-speed communication

**Priority 1 (Low)**: Background tasks
- Logging
- Non-critical diagnostics
- Low-priority data processing

**Priority 0**: Reserved for Idle task (kernel)

### Starvation Prevention

```c
// ✓ Good - periodic task yields CPU
void moderate_priority_task(void *argument) {
    for (;;) {
        do_work();
        vTaskDelay(pdMS_TO_TICKS(10)); // Yield every 10 ms
    }
}

// ✗ Bad - starves lower priority tasks
void high_priority_task(void *argument) {
    for (;;) {
        do_work();
        // No delay - runs continuously!
    }
}
```

## Common Task Patterns

### Periodic Task
```c
void periodic_task(void *argument) {
    TickType_t xLastWakeTime = xTaskGetTickCount();
    const TickType_t xFrequency = pdMS_TO_TICKS(100); // 100 ms period
    
    for (;;) {
        // Execute periodically
        perform_periodic_action();
        
        // Wait for next cycle (accounts for execution time)
        vTaskDelayUntil(&xLastWakeTime, xFrequency);
    }
}
```

### Event-Driven Task
```c
void event_driven_task(void *argument) {
    module_event_t event;
    
    for (;;) {
        // Wait for event
        if (xQueueReceive(event_queue, &event, portMAX_DELAY) == pdTRUE) {
            // Process event
            handle_event(&event);
        }
    }
}
```

### State Machine Task
```c
void state_machine_task(void *argument) {
    task_state_e state = STATE_INIT;
    
    for (;;) {
        switch (state) {
            case STATE_INIT:
                state = handle_init_state();
                break;
            case STATE_RUNNING:
                state = handle_running_state();
                break;
            case STATE_ERROR:
                state = handle_error_state();
                break;
        }
        
        // Yield CPU
        vTaskDelay(pdMS_TO_TICKS(10));
    }
}
```

## Debugging Tips

### Enable Runtime Statistics

In `FreeRTOSConfig.h`:
```c
#define configGENERATE_RUN_TIME_STATS   1
#define configUSE_STATS_FORMATTING_FUNCTIONS  1
```

Get task statistics:
```c
void print_task_stats(void) {
    TaskStatus_t *task_array;
    UBaseType_t task_count;
    uint32_t total_runtime;
    
    // Get number of tasks
    task_count = uxTaskGetNumberOfTasks();
    
    // Allocate array
    task_array = pvPortMalloc(task_count * sizeof(TaskStatus_t));
    
    if (task_array != NULL) {
        // Get task info
        task_count = uxTaskGetSystemState(task_array, task_count, &total_runtime);
        
        // Print info
        for (UBaseType_t i = 0; i < task_count; i++) {
            DEBUG_TRACE(DEBUG_CATEGORY_RTOS,
                       timestamp,
                       "Task=%s Pri=%d Stack=%d",
                       task_array[i].pcTaskName,
                       task_array[i].uxCurrentPriority,
                       task_array[i].usStackHighWaterMark);
        }
        
        vPortFree(task_array);
    }
}
```

### Common Issues and Solutions

| Symptom | Likely Cause | Solution |
|---------|--------------|----------|
| Hard fault/crash | Stack overflow | Increase stack size, check for recursion |
| Task not running | Wrong priority or blocking forever | Check priorities, add timeout to blocking calls |
| Deadlock | Mutex not released or circular wait | Use recursive mutex, review lock ordering |
| Random behavior | Race condition on shared data | Add mutex protection |
| High CPU usage | Task not yielding | Add vTaskDelay() calls |
| ISR crash | Using non-ISR API | Use FromISR variants only |

## Testing Checklist

After creating or modifying a task:

- [ ] Task function compiles and links correctly
- [ ] Stack size adequate (check high watermark)
- [ ] Priority appropriate for task's timing requirements
- [ ] Task yields CPU regularly (doesn't starve others)
- [ ] Queue/mutex initialization successful
- [ ] ISR synchronization uses FromISR variants
- [ ] Shared data protected by mutex
- [ ] Task can be deleted cleanly if needed
- [ ] No memory leaks in task loop
- [ ] Debug logging added for diagnostics
- [ ] Task behavior validated with runtime statistics

## References

- **FreeRTOS Documentation**: https://www.freertos.org/
- **CMSIS-RTOS2**: ARM CMSIS RTOS2 API documentation
- **ADC Task**: See `Core/Src/adc.c` for working example
- **RTOS Study**: `SBS2_DOCUMENTS/Study/RTOS.docx`
- **System Configuration**: `SBS2_DOCUMENTS/Study/System_Configuration_Overview.docx`
