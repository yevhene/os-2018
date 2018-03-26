# Робота з поіменованими каналами. Реалізація технології "клієнт-сервер"

## Основні положення

### Мета
Розібрати програми роботи з іменованими кана­лами, привести приклад реалізування технології "клієнт-сервер", ознайомити з оперативним та фоновим режимами роботи.

### Завдання

## Матеріали

### Робота з іменованими каналами з терінала

#### Створення іменованого канала
```bash
mknod name p
```

Аргументи:
1. Ім'я канала (файла);
2. Тип канала:
  - *b* - буфер;
  - *c*, *u* - символьний;
  - *p* - FIFO черга.

### Робота з іменованими каналами з C

### Необхідні заголовочні файли
```c
#include <sys/stat.h>
#include <unistd.h>
#include <linux/stat.h>
```

### Скидування привілегій
Всі нові файли (в тому числі і канали) будуть створюватися з дозволом для будь якого користувача на запис та зчитування)
```c
umask(0);
```

### Створення іменованого канала
```bash
mknod(FIFO_FILE, S_IFIFO | 0666, 0);
```
Аргументи:
1. Ім'я канала (файла);
2. Тип канала - вказує одночасно режим доступу до файлу (0666 - всі можуть писати/читати але не запускати) та тип канала (S_IFIFO - іменований канал в режимі черги);
3. Номер пристрою. В нашому випадку ігнорується.

## Приклад роботи програми
```bash
$ ./send Hello
$ ./client
Received string: Hello
```

## Приклад лістингу

### client.c
Очікує на дані в іменованому каналі і виводить їх на екран.

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/stat.h>
#include <unistd.h>
#include <linux/stat.h>
#define FIFO_FILE "MYFIFO"

int main(void)
{
  FILE *fp;
  char readbuf[80];

  /* Створення каналу FIFO, якщо той не існує */
  umask(0);
  mknod(FIFO_FILE, S_IFIFO | 0666, 0);

  while(1)
  {
    fp = fopen(FIFO_FILE, "r");
    fgets(readbuf, 80, fp);
    printf("Received string: %s\n", readbuf);
    fclose(fp);
  }

  return(0);
}
```

### send.c
Відправляє дані в іменований канал.

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/stat.h>
#include <unistd.h>
#define FIFO_FILE "MYFIFO"

int main(int argc, char *argv[])
{
  FILE *fp;
  if (argc != 2) {
    printf("USAGE: send [string]\n");
    exit(1);
  }

  /* Створення каналу FIFO, якщо той не існує */
  umask(0);
  mknod(FIFO_FILE, S_IFIFO | 0666, 0);

  if ((fp = fopen(FIFO_FILE, "w")) == NULL) {
    perror("fopen");
    exit(1);
  }

  fputs(argv[1], fp);
  fclose(fp);

  return(0);
}
```
