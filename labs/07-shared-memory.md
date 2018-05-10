# Поділяєма оперативна пам'ять

## Основні положення

### Мета
Ознайомитись з поділяємою оперативною пам'яттю та маніпуляціями над нею.

## Приклад лістингу

### shmtool.c
Дозволяє записати, зчитати, видалити та змінити дозволи на певний сегмент поділяємої пам'яті.

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <string.h>
#include <ctype.h>

#define SEGSIZE 100

int writeshm(int shmid, char *segptr, char *text);
int readshm(int shmid, char *segptr);
int removeshm(int shmid);
int changemode(int shmid, char *mode);
int usage();

int main(int argc, char *argv[])
{
  key_t key;
  int   shmid, cntr;
  char  *segptr;
  if (argc == 1)
    usage();

  key = ftok(".", 'S');

  if ((shmid = shmget(key, SEGSIZE, IPC_CREAT|IPC_EXCL|0666)) == -1)
  {
    printf("Shared memory segment exists - opening as client\n");
    if((shmid = shmget(key, SEGSIZE, 0)) == -1)
    {
      perror("shmget");
      exit(1);
    }
  }
  else
  {
    printf("Creating new shared memory segment\n");
  }

  if ((segptr = shmat(shmid, 0, 0)) == ((void *) -1))
  {
    perror("shmat");
    exit(1);
  }

  switch (tolower(argv[1][0]))
  {
    case 'w': writeshm(shmid, segptr, argv[2]);
      break;
    case 'r': readshm(shmid, segptr);
      break;
    case 'd': removeshm(shmid);
      break;
    case 'm': changemode(shmid, argv[2]);
      break;
    default: usage();
  }

  return 0;
}

int writeshm(int shmid, char *segptr, char *text)
{
  strcpy(segptr, text);
  printf("Done...\n");
}

int readshm(int shmid, char *segptr)
{
  printf("segptr: %s\n", segptr);
}

int removeshm(int shmid)
{
  shmctl(shmid, IPC_RMID, 0);
  printf("Shared memory segment marked for deletion\n");
}

int changemode(int shmid, char *mode)
{
  struct shmid_ds myshmds;
  shmctl(shmid, IPC_STAT, &myshmds);
  printf("Old permissions were: %o\n", myshmds.shm_perm.mode);
  unsigned int shm_perm_mode;
  sscanf(mode, "%o", &shm_perm_mode);
  myshmds.shm_perm.mode = shm_perm_mode;
  shmctl(shmid, IPC_SET, &myshmds);
  printf("New permissions are : %o\n", myshmds.shm_perm.mode);
}

int usage()
{
  fprintf(stderr, "shmtool - A utility for tinkering with shared memory\n");
  fprintf(stderr, "\nUSAGE:  shmtool (w)rite <text>\n");
  fprintf(stderr, "                (r)ead\n");
  fprintf(stderr, "                (d)elete\n");
  fprintf(stderr, "                (m)ode change <octal mode>\n");
  exit(1);
}
```
