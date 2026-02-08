# Домашнее задание к занятию 3 «Использование Ansible»

## Подготовка к выполнению

1. Подготовьте в Yandex Cloud три хоста: для `clickhouse`, для `vector` и для `lighthouse`.
2. Репозиторий LightHouse находится [по ссылке](https://github.com/VKCOM/lighthouse).

## Основная часть

1. Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает LightHouse.
2. При создании tasks рекомендую использовать модули: `get_url`, `template`, `yum`, `apt`.
3. Tasks должны: скачать статику LightHouse, установить Nginx или любой другой веб-сервер, настроить его конфиг для открытия LightHouse, запустить веб-сервер.
4. Подготовьте свой inventory-файл `prod.yml`.
5. Запустите `ansible-lint site.yml` и исправьте ошибки, если они есть.
6. Попробуйте запустить playbook на этом окружении с флагом `--check`.
7. Запустите playbook на `prod.yml` окружении с флагом `--diff`. Убедитесь, что изменения на системе произведены.
8. Повторно запустите playbook с флагом `--diff` и убедитесь, что playbook идемпотентен.
9. Подготовьте README.md-файл по своему playbook. В нём должно быть описано: что делает playbook, какие у него есть параметры и теги.
10. Готовый playbook выложите в свой репозиторий, поставьте тег `08-ansible-03-yandex` на фиксирующий коммит, в ответ предоставьте ссылку на него.

---

### Как оформить решение задания

Выполненное домашнее задание пришлите в виде ссылки на .md-файл в вашем репозитории.

---
Решение
Второй запуск с --diff

```
PLAY RECAP *********************************************************************************************************************************************************************************************************************************************************************************************************
clickhouse-vm              : ok=14   changed=2    unreachable=0    failed=0    skipped=5    rescued=0    ignored=0   
lighthouse-vm              : ok=8    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
vector-vm                  : ok=8    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

скрин с браузера
что делает playbook, какие у него есть параметры и теги:
Playbook автоматизирует создание инфраструктуры мониторинга в Yandex Cloud и установку трёх компонентов на отдельных виртуальных машинах:
ClickHouse — колоночная СУБД для хранения метрик и логов
Vector — сбор, преобразование и маршрутизация логов
LightHouse — веб-интерфейс для визуализации метрик из ClickHouse

```
Playbook поддерживает выборочный запуск через теги:

Тег	Что устанавливает	Когда использовать
clickhouse	Только ClickHouse	Когда нужно обновить/переустановить только СУБД
vector	Только Vector	Для работы с конвейером логов отдельно
lighthouse	Nginx + LightHouse	Для настройки веб-интерфейса
all	Все три компонента	Полное развёртывание (используется по умолчанию)
```

```
Примеры использования:
# Только LightHouse
ansible-playbook -i inventory/prod.yml site.yml --tags lighthouse

# ClickHouse + Vector
ansible-playbook -i inventory/prod.yml site.yml --tags clickhouse,vector

# Всё (явно)
ansible-playbook -i inventory/prod.yml site.yml --tags all
```

```
Параметры и переменные
 Ключевые переменные в playbook (site.yml):
 
Переменная	Значение по умолчанию	Описание
vector_version	"0.39.0"	Версия Vector для установки
lighthouse_version	"master"	Версия/ветка LightHouse (master всегда доступна)
vector_install_dir	"/opt/vector"	Директория для файлов Vector
vector_config_dir	"/etc/vector"	Директория для конфигов Vector
```
```
Параметры инфраструктуры (через Terraform):

Параметр	По умолчанию	Описание
vm_cores	2	Количество vCPU для каждой ВМ
vm_memory_gb	1	Объём ОЗУ в ГБ
vm_image_id	Ubuntu 22.04	Образ операционной системы
vm_enable_nat	true	Назначить публичный IP-адрес
vm_ssh_user	"ubuntu"	Пользователь для SSH-подключения
```

```
Inventory автоматическая генерация
Playbook использует inventory-файл, который автоматически создаётся Terraform с актуальными IP-адресами ВМ:
```