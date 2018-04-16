#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <stdint.h>
#include <pthread.h>
#define BIT_SIZE 32
long long getBitmapSize(long long size) {
return ( size/BIT_SIZE + (!!(size%BIT_SIZE)) );
}
void BitSet(uint32_t *bitmap, int index) {
bitmap[index / BIT_SIZE] |= 1 << (index % BIT_SIZE);
}
void BitClear(uint32_t *bitmap, int index) {
bitmap[index / BIT_SIZE] &= ~(1 << (index % BIT_SIZE));
}
int BitGet(uint32_t *bitmap, int index) {
return 1 & (bitmap[index / BIT_SIZE] >> (index % BIT_SIZE));
}
#define MIN 10
#define MAX 1000
typedef struct thread_data {
pthread_t thread;
int thread_id;
} THREAD;
uint32_t *process_id_map;
pthread_mutex_t lock;
int allocate_map(long long);
int allocate_process_id();
void release_process_id(int);
void* processFunc(void*);
int main(int argc, char** arv) {
if (allocate_map(MAX) == -1) {
printf("memory for process_id Map cannot be allocated");
exit(-1);
}
int nThreads;
printf("Please define the number of thread  required ");
scanf("%d", &nThreads);
THREAD *threads = (THREAD *)malloc(nThreads * sizeof(*threads));
for(int i=0; i<nThreads; i++) threads[i].thread_id = i+1;
pthread_attr_t attr;
pthread_attr_init(&attr);
for(int i=0; i<nThreads; i++) {
pthread_create(&threads[i].thread, &attr, processFunc, (void *)&threads[i]);
 }
for(int i=0; i<nThreads; i++)
 {
pthread_join(threads[i].thread, NULL);
  }
printf("All of the threads are executed");
free(process_id_map);
free(threads);
return 0;
}

int allocate_map(long long size) {
process_id_map = (uint32_t *)calloc(getBitmapSize(size), sizeof(uint32_t));
if(process_id_map == NULL)
return -1;
return 1;
}
int allocate_process_id() {
for(int i=MIN; i<MAX; i++) {
if(BitGet(process_id_map, i-1) == 0) {
BitSet(process_id_map, i-1);
return i;
}
}
return -1; 
}
void release_process_id(int process_id) {
BitClear(process_id_map, process_id-1);
}
void* processFunc(void* arg) {
THREAD *th = (THREAD *)arg;
int thread_id = th->thread_id;
sleep(rand()%5); 
pthread_mutex_lock(&lock);
int process_id = allocate_process_id();
if(process_id == -1) { 
printf("Thread %d : Unable to allocate process_id! All process_ids in use.", thread_id);
pthread_mutex_unlock(&lock);
pthread_exit(NULL);
}
pthread_mutex_unlock(&lock);
 printf("Thread %d : Allocated process_id %d", thread_id, process_id);
printf("Thread %d : Running code...", thread_id);
sleep(rand()%10+1); 
printf("Thread %d : Code Completed", thread_id);
pthread_mutex_lock(&lock);
printf("Thread %d : Releasing allocated process_id %d", thread_id, process_id);
release_process_id(process_id);	
printf("Thread %d : process_id %d Released", thread_id, process_id);
pthread_mutex_unlock(&lock);
pthread_exit(NULL);
}
