# Ansible Role: MySQL Primary-Replica Setup

## Описание
Эта Ansible-роль предназначена для автоматического развертывания MySQL-кластера с репликацией Master-Replica на серверах с **Ubuntu**.

## Функциональность
- Устанавливает MySQL-сервер 8 версии на мастер- и реплика-сервере
- Настраивает конфигурационные файлы для репликации
- Создаёт пользователя для репликации
- Экспортирует данные с мастер-сервера и импортирует их на реплику
- Запускает репликацию
- Проверяет статус репликации

## Требования
- Ansible 2.9+
- Python 3
- Коллекция **community.mysql** (`ansible-galaxy collection install community.mysql`)
- Доступ по SSH к серверам

## Переменные роли
Переменные определяются в `defaults/main.yml` или могут передаваться через `extra_vars`.

| Переменная               | Описание                                      |
|--------------------------|----------------------------------------------|
| `mysql_user`            | MySQL root-пользователь                      |
| `mysql_user_for_replica`| Имя пользователя для репликации              |
| `password_for_replica_user` | Пароль пользователя репликации          |
| `mysql_master_host`      | IP-адрес или хост мастера                    |
| `mysql_replica_host`     | IP-адрес или хост реплики                    |

## Инструкция по использованию

1. **Настроить inventory** (`inventory.ini`):

```ini
[all]
master ansible_host=192.168.1.10
replica ansible_host=192.168.1.11

[mysql_master]
master

[mysql_replica]
replica

```

2. **Заполнить переменные** в `defaults/main.yml`:

```yaml
mysql_user: "root"
mysql_user_for_replica: "replica"
password_for_replica_user: "StrongPassword123"
mysql_master_host: "192.168.1.10"
mysql_replica_host: "192.168.1.11"
```

3. **Запустить плейбук**:

```sh
ansible-playbook -i inventory.yml playbook.yml
```

## Проверка работы
На реплика-сервере выполните команду в MySQL:

```sql
SHOW REPLICA STATUS\G;
```

Ожидаемый результат:
- `Replica_IO_Running: Yes`
- `Replica_SQL_Running: Yes`
- `Seconds_Behind_Source: 0` (или небольшое значение)

## Возможные ошибки и решения

### Проблема: Репликация не работает
**Решение:**
1. Проверьте, что мастер открыт для подключения:
   ```sh
   ss -tlnp | grep 3306
   ```
2. Проверьте пользователя репликации на мастере:
   ```sql
   SELECT user, host FROM mysql.user;
   ```
3. Проверьте лог-файлы MySQL (`/var/log/mysql/error.log`).

