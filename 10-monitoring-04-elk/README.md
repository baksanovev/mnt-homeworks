# Домашнее задание к занятию 15 «Система сбора логов Elastic Stack»

## Задание 1. Развёртывание Elastic Stack

### Цель задания

Развернуть с помощью Docker и настроить взаимодействие компонентов:

- Elasticsearch — две ноды `es-hot` и `es-warm`.
- Logstash — приём и обработка логов.
- Kibana — просмотр и анализ логов.
- Filebeat — сбор логов Docker-контейнеров.

### 1. Подготовка окружения

Для выполнения домашнего задания использована виртуальная машина в Yandex Cloud.

| Параметр | Значение |
|---|---|
| Облачная платформа | Yandex Cloud |
| Операционная система | Ubuntu 24.04 LTS |
| CPU | 2 vCPU |
| RAM | 8 ГБ |
| Диск | 40 ГБ |
| Docker | 29.9.3 |
| Elasticsearch | 8.7.0 |
| Kibana | 8.7.0 |
| Logstash | 8.7.0 |
| Filebeat | 8.7.0 |

Для корректной работы Elasticsearch настроен параметр ядра:

```bash
sudo sysctl -w vm.max_map_count=1048576
```

Для сохранения настройки после перезагрузки:

```bash
echo 'vm.max_map_count=1048576' | sudo tee /etc/sysctl.d/99-elasticsearch.conf
```

Проверка:

```bash
sysctl vm.max_map_count
```

Результат:

```text
vm.max_map_count = 1048576
```

### 2. Запуск Elastic Stack

Для развёртывания использовались Docker Compose и конфигурационные файлы из директории `help` репозитория Нетологии.

В конфигурации Docker Compose были ограничены внешние подключения к Elasticsearch и Kibana. Их порты опубликованы только на `127.0.0.1`.

Запуск контейнеров:

```bash
docker compose up -d
```

Проверка:

```bash
docker compose ps
```

Были запущены следующие контейнеры:

| Контейнер | Назначение |
|---|---|
| es-hot | Elasticsearch, нода hot |
| es-warm | Elasticsearch, нода warm |
| logstash | Обработка поступающих логов |
| kibana | Веб-интерфейс для просмотра логов |
| filebeat | Сбор Docker-логов |
| some_app | Тестовое приложение, генерирующее события |

Всего запущено шесть контейнеров, поскольку учебный Compose также содержит дополнительное приложение `some_app`, используемое для генерации логов.

**Скриншот `docker ps` после 18 минут работы:**

<img width="1087" height="201" alt="task1-docker-ps" src="https://github.com/user-attachments/assets/743307c0-d70a-4a15-8bf3-e8ecb09f0051" />


### 3. Проверка Elasticsearch

Для проверки состояния кластера выполнена команда:

```bash
curl -sS http://127.0.0.1:9200/_cluster/health?pretty
```

Результат:

```json
{
  "cluster_name": "es-docker-cluster",
  "status": "yellow",
  "timed_out": false,
  "number_of_nodes": 2,
  "number_of_data_nodes": 2,
  "active_primary_shards": 9,
  "active_shards": 9,
  "unassigned_shards": 1
}
```

Кластер успешно запущен, работают две ноды Elasticsearch.

Статус `yellow` означает, что основные шарды доступны, но одна из реплик не назначена. Для выполнения учебного задания это не препятствует записи и поиску логов.

Проверка нод:

```bash
curl -sS http://127.0.0.1:9200/_cat/nodes?v
```

В результате обнаружены обе ноды:

- `es-hot`
- `es-warm`

### 4. Проверка индексов Elasticsearch

Для проверки поступающих логов выполнена команда:

```bash
curl -sS 'http://127.0.0.1:9200/_cat/indices?v'
```

В Elasticsearch был обнаружен индекс:

```text
logstash-2026.11.09
```

На момент проверки индекс содержал 1002 документа.

Это подтверждает передачу логов по цепочке:

**Docker → Filebeat → Logstash → Elasticsearch**

### 5. Проверка Kibana

Для доступа к Kibana использован SSH-туннель:

```bash
ssh -N -L 5601:127.0.0.1:5601 USER@PUBLIC_IP
```

После подключения Kibana доступна в браузере:

`http://localhost:5601`

Веб-интерфейс Kibana успешно открылся.

**Скриншот интерфейса Kibana:**

<img width="1314" height="847" alt="task1-kibana" src="https://github.com/user-attachments/assets/1aee3a72-31be-4c51-94f9-9cd27ef5c4e6" />


**Вывод по заданию №1:** стек Elastic Stack успешно развёрнут в Docker. Обе ноды Elasticsearch работают, Filebeat и Logstash обеспечивают доставку событий, Kibana доступна через веб-интерфейс.

---

## Задание 2. Работа с Data Views и Discover в Kibana

### 1. Создание Data Views

В Kibana открыт раздел:

**Stack Management → Kibana → Data Views**

Созданы два Data Views:

| Название | Index pattern | Поле времени |
|---|---|---|
| `logstash-*` | `logstash-*` | `@timestamp` |
| `logstash-2026.11.09` | `logstash-2026.11.09` | `@timestamp` |

Первый Data View позволяет работать со всеми ежедневными индексами `logstash-*`.

Второй Data View предназначен для просмотра логов конкретного индекса от 9 ноября 2026 года.

Data View `logstash-*` установлен по умолчанию.

**Скриншот созданных Data Views:**

<img width="1309" height="917" alt="task2-data-views" src="https://github.com/user-attachments/assets/db91dc84-ba8d-464f-abd1-a2eba9ff73ab" />


### 2. Просмотр логов в Discover

Для анализа логов открыт раздел:

**Analytics → Discover**

Выбран Data View:

```text
logstash-*
```

Для поиска документов, содержащих поле `message`, использовался запрос KQL:

```text
message: *
```

В интерфейсе Discover были доступны:

- Гистограмма распределения событий по времени.
- Таблица документов.
- Поле `@timestamp`.
- Поля Filebeat и Docker-контейнеров.
- Возможность раскрывать документы и анализировать их содержимое.
- Фильтрация и поиск событий с использованием KQL.

На момент проверки Kibana отобразила **1489 событий**.

Это подтверждает, что логи тестового приложения поступают в Elasticsearch и доступны для поиска и анализа.

**Скриншот Kibana Discover:**

<img width="1306" height="926" alt="task2-discover" src="https://github.com/user-attachments/assets/de18c207-bb13-45df-b3ca-80d4a6ec7900" />


### 3. Результат

С помощью Kibana Discover удалось просмотреть события, изучить структуру документов и проверить поиск логов.

Также подтверждена работа всей цепочки сбора и обработки событий:

**Docker → Filebeat → Logstash → Elasticsearch → Kibana**

**Вывод по заданию №2:** созданы два Data Views, настроен просмотр логов в Discover, выполнен поиск по полю `message`. События успешно отображаются в Kibana.

---

## Общий вывод

В ходе выполнения домашнего задания:

1. Подготовлена виртуальная машина Ubuntu в Yandex Cloud.
2. Настроены параметры ядра для Elasticsearch.
3. Развёрнут Elastic Stack с помощью Docker Compose.
4. Запущены две ноды Elasticsearch: hot и warm.
5. Настроена передача Docker-логов через Filebeat и Logstash.
6. Проверено создание индекса `logstash-*`.
7. Запущен веб-интерфейс Kibana.
8. Созданы два Data Views.
9. Изучены поиск и просмотр логов через Kibana Discover.
