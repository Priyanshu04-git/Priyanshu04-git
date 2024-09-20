#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <signal.h>
#include <string.h>
#include <time.h>
#include <pthread.h>

#define MAX_PROCESSES 10
#define TIME_SLICE 1
#define NUM_THREADS 4

typedef struct {
    pid_t pid;
    int priority;
    int cpu_usage; // CPU time consumed
    int wait_time; // Time waiting for CPU
    pthread_t thread_id;
} Process;

Process processes[MAX_PROCESSES];
int process_count = 0;
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;

void *simulate_process(void *arg) {
    Process *p = (Process *)arg;

    // Simulate process execution
    for (int i = 0; i < 10; i++) {
        sleep(TIME_SLICE);
        pthread_mutex_lock(&mutex);
        p->cpu_usage++;
        printf("Process %d running. CPU usage: %d\n", p->pid, p->cpu_usage);
        pthread_mutex_unlock(&mutex);
    }

    pthread_exit(NULL);
}

void run_processes() {
    pthread_t threads[NUM_THREADS];

    for (int i = 0; i < process_count; i++) {
        pthread_create(&processes[i].thread_id, NULL, simulate_process, &processes[i]);
    }

    for (int i = 0; i < process_count; i++) {
        pthread_join(processes[i].thread_id, NULL);
    }
}

void adjust_priority(Process *p) {
    // Simple policy: if CPU usage is high, lower priority
    if (p->cpu_usage > 10) {
        p->priority++;
    }
    // If wait time is high, increase priority
    if (p->wait_time > 5) {
        p->priority--;
    }
}

void print_processes() {
    printf("Processes:\n");
    for (int i = 0; i < process_count; i++) {
        printf("PID: %d, Priority: %d, CPU Usage: %d, Wait Time: %d\n",
               processes[i].pid, processes[i].priority, processes[i].cpu_usage, processes[i].wait_time);
    }
}

int main() {
    printf("Dynamic Process Priority Adjustment System\n");

    // Initialize processes
    for (int i = 0; i < MAX_PROCESSES; i++) {
        processes[i].pid = 0; // No process created yet
        processes[i].priority = rand() % 10; // Random initial priority
        processes[i].cpu_usage = 0;
        processes[i].wait_time = 0;
        process_count++;
    }

    run_processes();

    // Adjust priorities based on simulated performance
    for (int i = 0; i < process_count; i++) {
        adjust_priority(&processes[i]);
    }

    print_processes();

    return 0;
}
