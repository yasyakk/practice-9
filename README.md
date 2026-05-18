# Практична робота №9  
## Тема: Системні виклики UNIX/Linux. Права доступу, UID та привілеї користувачів


## Мета роботи
Дослідити механізми керування користувачами та правами доступу в Linux, роботу UID, виконання команд від імені різних користувачів, а також поведінку системи при зміні прав доступу до файлів.


## Теоретичні відомості
У Linux кожен процес та користувач ідентифікується через UID. Саме UID визначає права доступу до файлів і ресурсів системи.

Файли можуть виконуватися:
- напряму (якщо є право `x`)
- через інтерпретатор, навіть без права виконання

Системний адміністратор має повний доступ до всіх файлів і може змінювати власників та права доступу через `chown` і `chmod`.


## Завдання 9.1 — Аналіз користувачів системи

### Опис
Програма отримує список користувачів через `getent passwd` та визначає звичайних користувачів (UID ≥ 1000), окрім поточного.

### Код

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

int main() {
    FILE *fp = popen("getent passwd", "r");
    char line[1024];
    uid_t myuid = getuid();

    printf("Звичайні користувачі:\n");

    while (fgets(line, sizeof(line), fp)) {
        char *user = strtok(line, ":");
        strtok(NULL, ":");
        char *uid_str = strtok(NULL, ":");

        int uid = atoi(uid_str);

        if (uid >= 1000 && uid != myuid) {
            printf("User: %s UID: %d\n", user, uid);
        }
    }

    pclose(fp);
    return 0;
}

## Завдання 9.2 — Доступ до /etc/shadow

### Опис

Програма виконує команду cat /etc/shadow з використанням sudo.

### Код

#include <stdlib.h>

int main() {
    system("sudo cat /etc/shadow");
    return 0;
}

## Завдання 9.3 — Робота з файлами та правами root

### Опис

Програма створює файл, копіює його через root, змінює права та тестує можливість редагування і видалення.

### Код

#include <stdlib.h>

int main() {
    system("echo 'Test content' > userfile.txt");

    system("sudo cp userfile.txt /home/$USER/rootcopy.txt");
    system("sudo chown root:root /home/$USER/rootcopy.txt");
    system("sudo chmod 600 /home/$USER/rootcopy.txt");

    printf("Спроба змінити файл:\n");
    system("echo 'change' >> /home/$USER/rootcopy.txt");

    printf("Спроба видалити файл:\n");
    system("rm /home/$USER/rootcopy.txt");

    return 0;
}

## Завдання 9.4 — Перевірка користувача

### Опис

Програма виконує whoami та id.

### Код

#include <stdlib.h>

int main() {
    system("whoami");
    system("id");
    return 0;
}

## Завдання 9.5 — Зміна прав доступу

## Опис

Створюється файл, змінюється власник і права через root, після чого перевіряється доступ.

## Код

#include <stdlib.h>

int main() {
    system("echo 'temp data' > temp.txt");

    system("sudo chown root:root temp.txt");
    system("sudo chmod 400 temp.txt");

    printf("Читання:\n");
    system("cat temp.txt");

    printf("Запис:\n");
    system("echo 'test' >> temp.txt");

    return 0;
}

### Результат

- Читання: можливе або обмежене
- Запис: заборонено

## Завдання 9.6 — Аналіз прав у системі

### Опис

Програма переглядає права доступу у різних директоріях.

### Код

#include <stdlib.h>

int main() {
    system("ls -l ~");
    system("ls -l /usr/bin | head");
    system("ls -l /etc | head");

    printf("Спроба доступу до /etc/shadow:\n");
    system("cat /etc/shadow");

    return 0;
}

## Варіант 9 
### Чи можна мати однаковий UID?

Так, у Linux можна створити два облікові записи з однаковим UID.

### Приклад

sudo useradd userA
sudo useradd userB

sudo usermod -u 2000 userA
sudo usermod -u 2000 userB

### Наслідки

- Обидва користувачі мають однакові права
- Система не розрізняє їх
- Повний доступ до однакових файлів
- Серйозна загроза безпеці

## Висновок

У ході роботи було досліджено механізми управління користувачами в Linux, роботу UID та вплив прав доступу на файли. Було показано, що root має повний контроль над системою, а звичайні користувачі обмежені правами доступу. Також доведено, що UID є ключовим ідентифікатором користувача, і його дублювання призводить до порушення моделі безпеки системи.

## Результати роботи програми:

Усі скріншоти виконання практичної роботи знаходяться в папці:

[screenshots](./screenshots)
