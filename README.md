# Домашнее задание к занятию 3 «Использование Ansible»

## Подготовка к выполнению

1. Подготовьте в Yandex Cloud три хоста: для clickhouse, для vector и для lighthouse.
2. Репозиторий LightHouse находится [LightHouse](https://github.com/VKCOM/lighthouse) 
![рис 1](https://github.com/ysatii/ansible-hw3/blob/main/img/img_ansble1.jpg)  

## Основная часть
### Задание 1.
 Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает LightHouse.

## Решение  1.
плайбук переписан https://github.com/ysatii/ansible-hw3/blob/main/playbook/site.yml
 LightHouse устанавливаеться с помощью роли 

```
name: Install and configure LightHouse
hosts: lighthouse
  become: true
  roles:
    - lighthouse
```
[handlers](https://github.com/ysatii/ansible-hw3/blob/main/playbook/roles/lighthouse/handlers/main.yml)
[tasks](https://github.com/ysatii/ansible-hw3/blob/main/playbook/roles/lighthouse/tasks/main.yml)
[lighthouse_config.j2](https://github.com/ysatii/ansible-hw3/blob/main/playbook/roles/lighthouse/templates/lighthouse_config.j2)


### Задание 2.
 При создании tasks рекомендую использовать модули: `get_url`, `template`, `yum`, `apt`.

## Решение  2.
apt - не требуеться у нас операционная Centos7 и


### Задание 3. 
Tasks должны: скачать статику LightHouse, установить Nginx или любой другой веб-сервер, настроить его конфиг для открытия LightHouse, запустить веб-сервер.
![рис 8](https://github.com/ysatii/ansible-hw3/blob/main/img/img_ansble8.jpg) 
![рис 9](https://github.com/ysatii/ansible-hw3/blob/main/img/img_ansble9.jpg) 
![рис 10](https://github.com/ysatii/ansible-hw3/blob/main/img/img_ansble10.jpg) 

### Задание 4.
 Подготовьте свой inventory-файл `prod.yml`.
```
clickhouse:
  hosts:
    clickhouse-01:
      ansible_host: 89.169.158.76
      ansible_connection: ssh
      ansible_user: lamer
      ansible_ssh_private_key_file: ssh_env/id_rsa_insecure_clickhouse-01

vector:
  hosts:
    vector-02:
      ansible_host: 158.160.34.19
      ansible_connection: ssh
      ansible_user: lamer
      ansible_ssh_private_key_file: ssh_env/id_rsa_insecure_vector-02


lighthouse:
  hosts:
    lighthouse-03:
      ansible_host: "158.160.39.131"
      ansible_connection: ssh
      ansible_user: lamer
      ansible_ssh_private_key_file: ssh_env/id_rsa_insecure_lighthouse-03
```

### Задание 5.
 Запустите `ansible-lint site.yml` и исправьте ошибки, если они есть.

## Решение  5.
 ![рис 2](https://github.com/ysatii/ansible-hw3/blob/main/img/img_ansble2.jpg)  
 ошибки исправлены!

### Задание 6.
 Попробуйте запустить playbook на этом окружении с флагом `--check`.

## Решение  6.
флаг `--check` Имитация выполнения Playbook — Ansible проверяет, какие задачи будут выполнены, но не изменяет систему.
![рис 3](https://github.com/ysatii/ansible-hw3/blob/main/img/img_ansble3.jpg) 
деййствие плайбука преостанавливаеться поскольку пакеты *.rpm не были реально скачены

### Задание 7. 
Запустите playbook на `prod.yml` окружении с флагом `--diff`. Убедитесь, что изменения на системе произведены.

## Решение  7.
![рис 4](https://github.com/ysatii/ansible-hw3/blob/main/img/img_ansble4.jpg) 
все изменения были примены т.к. до этого плайбук на запускался!

### Задание 8.
 Повторно запустите playbook с флагом `--diff` и убедитесь, что playbook идемпотентен.

## Решение  8.
![рис 5](https://github.com/ysatii/ansible-hw3/blob/main/img/img_ansble5.jpg) 
![рис 6](https://github.com/ysatii/ansible-hw3/blob/main/img/img_ansble6.jpg) 
![рис 7](https://github.com/ysatii/ansible-hw3/blob/main/img/img_ansble7.jpg) 


произошла только часть изменений каких как перезапуск веб сервера и попытка создания таблиц

### Задание 9.
 Подготовьте README.md-файл по своему playbook. В нём должно быть описано: что делает playbook, какие у него есть параметры и теги.

 # Описание Ansible Playbook

## Решение  9.


##  Что делает Playbook?
Этот Ansible Playbook предназначен для автоматизированной установки и настройки следующих компонентов:

1. **ClickHouse** – мощная колонночная СУБД.
2. **Lighthouse** – веб-интерфейс для работы с ClickHouse.
3. **Nginx** – веб-сервер для работы с Lighthouse.  **Vector** – система сбора и обработки логов.

# site.yml (основной плейбук)

Этот плейбук выполняет несколько ключевых действий:

    Устанавливает временную зону на Europe/Moscow для всех серверов.
    Устанавливает ClickHouse на серверах clickhouse:
        Загружает .rpm пакеты ClickHouse.
        Устанавливает их с помощью yum.
        Разворачивает конфигурационные файлы ClickHouse (config.xml, users.xml).
        Создаёт базу данных logs и таблицу vector_logs.
        Перезапускает ClickHouse.
    Устанавливает и настраивает LightHouse на lighthouse:
        Устанавливает Nginx и необходимые зависимости.
        Загружает и разархивирует LightHouse.
        Настраивает конфигурацию Nginx.
        Перезапускает Nginx.
    Проверяет доступность ClickHouse и выводит URL LightHouse.

#  Основные этапы выполнения Playbook

##  Установите сначала коллекции их может не быть
```sh
ansible-galaxy collection install community.general
```
## Команды для управления и мониторинга
перезапуск вектор на машине lighthouse или вектор 
```
sudo systemctl restart vector
```

редактирование файла настроек
```
nano /etc/vector/vector.toml
```

просмотр логов 
```
sudo journalctl -u vector -n 50 --no-pager -f
```

проверка приема данных на основном сервере vector! сервер lighthouse/nginx с помощью vector получает логи с nginx и передает их на основной вектор

```
echo '{"timestamp":"2025-03-14T21:50:00Z","host":"lighthouse","log_type":"nginx","message":"test log"}' | nc 62.84.119.228 6000
```

Статистика запросов к вектор необхдимо включить  
 [api]  
enabled = true  

```
/usr/local/bin/vector top 
```

###  **1. Установка ClickHouse**
- Загружает и устанавливает указанную версию ClickHouse.
- Запускает службу и включает её в автозагрузку.

###  **2. Установка и настройка Lighthouse**
- Устанавливает Nginx.
- Скачивает и разворачивает Lighthouse.
- Настраивает конфигурацию Nginx для работы с Lighthouse.
- Запускает Nginx и включает его в автозагрузку.

###  **3. Установка и настройка Vector**
- Создает директорию конфигурации Vector.
- Копирует конфигурационный файл.
- Создает systemd-сервис для Vector.
- Запускает и включает Vector в автозагрузку.
Vector устанавливаеться на двух машинах на самой машине vector и на машине Lighthouse/nginx 

с машины Lighthouse/nginx логи передаються на машину с vector далее в бд установленную на машине ClickHouse

 
# install_vector.yml (установка и настройка Vector)

Этот playbook выполняет установку и настройку Vector на серверах групп vector и lighthouse:

    Создаёт системного пользователя vector.
    Создаёт необходимые директории (/var/lib/vector, /etc/vector).
    Загружает и устанавливает Vector версии 0.34.0.
    Разворачивает конфигурационный файл Vector (vector.toml).
    Создаёт systemd-сервис для Vector и включает его автозапуск.
    На сервере с Nginx (группа lighthouse) Vector собирает логи Nginx и отправляет их на отдельный сервер с Vector (группа vector).
    Перезапускает сервис Vector.

##  Переменные Playbook
| Переменная            | Значение (по умолчанию)             | Описание |
|----------------------|----------------------------------|-----------|
| `clickhouse_version` | `23.8.2.7`                      | Версия ClickHouse |
| `download_dir`       | `/home/{{ ansible_user }}`      | Каталог для скачивания файлов |

##  Используемые роли
- **`lighthouse`** – установка и настройка Lighthouse.
- **`vector`** – установка и настройка Vector.

##  Как запустить Playbook?
1. **Проверка синтаксиса:**
   ```bash
   ansible-lint site.yml
   ```
2. **Тестовый запуск (без изменений):**
   ```bash
   ansible-playbook -i inventory/prod.yml --check site.yml
   ```
3. **Реальный запуск:**
   ```bash
   ansible-playbook -i inventory/prod.yml site.yml --diff
   ```

После выполнения Playbook ClickHouse, Lighthouse будут настроены и запущены автоматически.
Установка и настройка vector осществляеться плайбуком [install_vector.yml](https://github.com/ysatii/ansible-hw3/blob/main/playbook/install_vector.yml)




### Задание 10. 
Готовый playbook выложите в свой репозиторий, поставьте тег `08-ansible-03-yandex` на фиксирующий коммит, в ответ предоставьте ссылку на него.

---

### Как оформить решение задания

Выполненное домашнее задание пришлите в виде ссылки на .md-файл в вашем репозитории.

 roles/lighthouse/tasks/main.yml (установка Lighthouse)

    Устанавливает nginx и unzip через yum.
    Загружает LightHouse с GitHub.
    Распаковывает файлы в /usr/share/nginx/html/.
    Настраивает конфигурацию Nginx для работы с LightHouse.
    Перезапускает Nginx.

 roles/vector/tasks/main.yml (установка Vector)

    Устанавливает и настраивает Vector на vector, clickhouse, lighthouse.
    Создаёт необходимые каталоги и системного пользователя.
    Загружает и устанавливает Vector.
    Разворачивает соответствующие конфигурационные файлы для каждого типа узла:
        На сервере vector Vector принимает логи от других серверов.
        На сервере lighthouse Vector собирает логи Nginx и отправляет их в Vector на vector.
        vector_nginx_lighthouse.toml.j2 – для сбора логов Nginx и отправки на vector.
    Разворачивает systemd-сервис и запускает Vector.

