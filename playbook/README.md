# Ansible Playbook: Установка ClickHouse и Vector

## 📌 Описание
Этот playbook предназначен для **автоматической установки, настройки и запуска**:
- **ClickHouse** (на сервере `clickhouse-01`)
- **Vector** (на сервере `clickhouse-02`)

**Используемые технологии:**
- **Ansible** для автоматизированного развёртывания
- **CentOS** как целевая операционная система
- **RPM-пакеты** для установки ClickHouse и Vector
- **Systemd** для управления сервисами

---

## ⚙ **Структура проекта**
```
.
├── img
│   ├── img_ansble1.jpg
│   ├── img_ansble2.jpg
│   ├── img_ansble3.jpg
│   ├── img_ansble4.jpg
│   ├── img_ansble5.jpg
│   ├── img_ansble6.jpg
│   └── img_ansble7.jpg
├── playbook
│   ├── group_vars
│   │   └── clickhouse
│   │       └── vars.yml
│   ├── inventory
│   │   └── prod.yml
│   ├── README.md
│   ├── roles
│   │   └── vector
│   │       ├── handlers
│   │       │   └── main.yml
│   │       ├── tasks
│   │       │   └── main.yml
│   │       └── templates
│   │           └── vector.toml.j2
│   ├── site.yml
│   └── ssh_env
│       ├── id_rsa_insecure_clickhouse-01
│       └── id_rsa_insecure_clickhouse-02
└── README.md

11 directories, 17 files

```

## Переменные Playbook
```
clickhouse_version: "23.8.2.7"
vector_version: "0.27.0"
download_dir: "/home/{{ ansible_user }}/clickhouse"
```
Переменная	Описание
clickhouse_version	- Версия ClickHouse
vector_version	- Версия Vector
download_dir	- Каталог, куда скачиваются RPM-пакеты

## Теги Playbook
Тег                          	Описание	                                               Пример вызова
clickhouse	 Установка и настройка ClickHouse	         ansible-playbook -i inventory/prod.yml site.yml --tags clickhouse
vector	     Установка и настройка Vector	             ansible-playbook -i inventory/prod.yml site.yml --tags vector

# проверка успешности развертывания 
## Проверка статуса сервисов
```systemctl status clickhouse-server```
```systemctl status vector```
Должен быть Active: active (running)

##  Проверка базы в ClickHouse
```clickhouse-client -q "SHOW DATABASES;"```
 Должна быть база logs  

## Проверка логов Vector
```journalctl -u vector --no-pager | tail -20```
Должны быть строки с Started Vector Service 

Проверка конфигурации Vector
```cat /etc/vector/vector.toml```
Должен отобразиться развёрнутый конфиг

##  **Подготовка перед запуском Playbook**
Чтобы playbook работал **корректно**, необходимо выполнить следующие шаги:

### ** Предоставить доступ Ansible к серверам по SSH**
На управляющей машине (где установлен Ansible) **должны быть SSH-ключи** для доступа к целевым серверам.
Проверь, что ключи находятся в **`ssh_env/`**, а затем попробуй зайти на серверы вручную:

```bash
ssh -i ssh_env/id_rsa_insecure_clickhouse-01 lamer@<CLICKHOUSE_IP>
ssh -i ssh_env/id_rsa_insecure_clickhouse-02 lamer@<VECTOR_IP>
```