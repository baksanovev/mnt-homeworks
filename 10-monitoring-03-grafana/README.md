# Домашнее задание к занятию 14 «Средство визуализации Grafana»

## Задание 1. Развёртывание Grafana и Prometheus

Для выполнения домашнего задания использована виртуальная машина Ubuntu 24.04 в Yandex Cloud.

С помощью Docker Compose из директории `help` развёрнуты следующие сервисы:

- **Grafana** — визуализация метрик.
- **Prometheus** — сбор и хранение метрик.
- **Node Exporter** — экспорт метрик операционной системы.

Для запуска использована команда:

```bash
docker compose up -d
```

В Grafana добавлен источник данных Prometheus:

```text
http://prometheus:9090
```

Prometheus установлен как источник данных по умолчанию.

**Скриншот подключённого Data Source:**

<img width="1322" height="923" alt="task1-prometheus" src="https://github.com/user-attachments/assets/c4f42421-6619-412a-a143-dd68ce5cc7c0" />
<img width="1309" height="677" alt="11" src="https://github.com/user-attachments/assets/fef08de8-fca0-4f3c-a58b-b8880c8bd859" />


---

## Задание 2. Создание Dashboard

Создан Dashboard **Netology — System Monitoring**, содержащий четыре панели мониторинга.

### 1. CPU Utilization

Отображение загрузки процессора в процентах:

```promql
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

### 2. CPU Load Average 1/5/15

Средняя нагрузка за 1, 5 и 15 минут:

```promql
node_load1
```

```promql
node_load5
```

```promql
node_load15
```

### 3. Available RAM

Количество доступной оперативной памяти:

```promql
node_memory_MemAvailable_bytes
```

Единица измерения в Grafana — bytes (IEC).

### 4. Available Disk Space

Количество доступного дискового пространства:

```promql
node_filesystem_avail_bytes{mountpoint="/",fstype!~"tmpfs|overlay|squashfs"}
```

Единица измерения — bytes (IEC).

**Результат — Dashboard с четырьмя панелями:**

<img width="1316" height="811" alt="task2-dashboard" src="https://github.com/user-attachments/assets/15ed89e9-71aa-48b2-9545-c8bf287587c1" />


---

## Задание 3. Настройка Alert Rules

Для контроля состояния сервера созданы четыре правила оповещений.

| Название Alert | Условие | Время ожидания |
|---|---|---|
| High CPU Usage | CPU выше 80% | 5 минут |
| High CPU Load | Load Average выше 2 | 5 минут |
| Low Available RAM | Свободной памяти меньше 500 MiB | 5 минут |
| Low Disk Space | Свободного места меньше 2 GiB | 5 минут |

Интервал проверки правил — 1 минута.

Настроены следующие пороговые значения:

**High CPU Usage:**

```text
WHEN avg() OF query(A, 5m, now) IS ABOVE 80
```

**High CPU Load:**

```text
WHEN avg() OF query(A, 5m, now) IS ABOVE 2
```

**Low Available RAM:**

```text
WHEN avg() OF query(A, 5m, now) IS BELOW 524288000
```

**Low Disk Space:**

```text
WHEN avg() OF query(A, 5m, now) IS BELOW 2147483648
```

После настройки и устранения ошибок проверки все четыре правила перешли в состояние **OK**.

**Скриншот настроенных Alert Rules:**

<img width="1205" height="849" alt="task3-alerts" src="https://github.com/user-attachments/assets/f94f1cc8-b1ee-4417-b870-d56754038822" />


---

## Задание 4. Экспорт Dashboard в JSON

Созданный Dashboard экспортирован через интерфейс Grafana:

**Dashboard Settings → JSON Model**

Конфигурация сохранена в файл `dashboard.json` и загружена в репозиторий GitHub.

**Полный листинг экспортированной конфигурации:**

[Посмотреть dashboard.json](dashboard.json)

В JSON содержатся конфигурации четырёх панелей, PromQL-запросы и параметры отображения данных.

**Экспортированная конфигурация:**

<details>
<summary>Показать полный JSON Dashboard</summary>

```json
  
{
  "annotations": {
    "list": [
      {
        "builtIn": 1,
        "datasource": "-- Grafana --",
        "enable": true,
        "hide": true,
        "iconColor": "rgba(0, 211, 255, 1)",
        "name": "Annotations & Alerts",
        "type": "dashboard"
      }
    ]
  },
  "editable": true,
  "gnetId": null,
  "graphTooltip": 0,
  "id": 2,
  "links": [],
  "panels": [
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": null,
      "fieldConfig": {
        "defaults": {
          "custom": {}
        },
        "overrides": []
      },
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 8,
        "w": 12,
        "x": 0,
        "y": 0
      },
      "hiddenSeries": false,
      "id": 2,
      "legend": {
        "avg": false,
        "current": false,
        "max": false,
        "min": false,
        "show": true,
        "total": false,
        "values": false
      },
      "lines": true,
      "linewidth": 1,
      "nullPointMode": "null",
      "options": {
        "alertThreshold": true
      },
      "percentage": false,
      "pluginVersion": "7.4.0",
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "expr": "100 - (avg by (instance) (rate(node_cpu_seconds_total{mode=\"idle\"}[5m])) * 100)",
          "interval": "",
          "legendFormat": "",
          "refId": "A"
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "CPU Utilization",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "$$hashKey": "object:54",
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "$$hashKey": "object:55",
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": null,
      "fieldConfig": {
        "defaults": {
          "custom": {}
        },
        "overrides": []
      },
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 8,
        "w": 12,
        "x": 12,
        "y": 0
      },
      "hiddenSeries": false,
      "id": 4,
      "legend": {
        "avg": false,
        "current": false,
        "max": false,
        "min": false,
        "show": true,
        "total": false,
        "values": false
      },
      "lines": true,
      "linewidth": 1,
      "nullPointMode": "null",
      "options": {
        "alertThreshold": true
      },
      "percentage": false,
      "pluginVersion": "7.4.0",
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "expr": "node_load1",
          "interval": "1m",
          "legendFormat": "min1",
          "refId": "A"
        },
        {
          "expr": "node_load5",
          "hide": false,
          "interval": "1m",
          "legendFormat": "min5",
          "refId": "B"
        },
        {
          "expr": "node_load15",
          "hide": false,
          "interval": "1m",
          "legendFormat": "min15",
          "refId": "C"
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "CPU Load Average 1/5/15",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "$$hashKey": "object:289",
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "$$hashKey": "object:290",
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": null,
      "fieldConfig": {
        "defaults": {
          "custom": {},
          "unit": "bytes"
        },
        "overrides": []
      },
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 8,
        "w": 12,
        "x": 0,
        "y": 8
      },
      "hiddenSeries": false,
      "id": 6,
      "legend": {
        "avg": false,
        "current": false,
        "max": false,
        "min": false,
        "show": true,
        "total": false,
        "values": false
      },
      "lines": true,
      "linewidth": 1,
      "nullPointMode": "null",
      "options": {
        "alertThreshold": true
      },
      "percentage": false,
      "pluginVersion": "7.4.0",
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "expr": "node_memory_MemAvailable_bytes",
          "format": "time_series",
          "instant": false,
          "interval": "",
          "legendFormat": "Available RAM",
          "refId": "A"
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "Available RAM",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "$$hashKey": "object:461",
          "format": "bytes",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "$$hashKey": "object:462",
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": null,
      "fieldConfig": {
        "defaults": {
          "custom": {},
          "unit": "bytes"
        },
        "overrides": []
      },
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 8,
        "w": 12,
        "x": 12,
        "y": 8
      },
      "hiddenSeries": false,
      "id": 8,
      "legend": {
        "avg": false,
        "current": false,
        "max": false,
        "min": false,
        "show": true,
        "total": false,
        "values": false
      },
      "lines": true,
      "linewidth": 1,
      "nullPointMode": "null",
      "options": {
        "alertThreshold": true
      },
      "percentage": false,
      "pluginVersion": "7.4.0",
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "expr": "node_filesystem_avail_bytes{mountpoint=\"/\",fstype!~\"tmpfs|overlay|squashfs\"}",
          "interval": "",
          "legendFormat": "",
          "refId": "A"
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "Available Disk Space",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "$$hashKey": "object:550",
          "format": "bytes",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "$$hashKey": "object:551",
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    }
  ],
  "schemaVersion": 27,
  "style": "dark",
  "tags": [],
  "templating": {
    "list": []
  },
  "time": {
    "from": "now-15m",
    "to": "now"
  },
  "timepicker": {},
  "timezone": "",
  "title": "Netology — System Monitoring Copy",
  "uid": "F33V_keDz",
  "version": 1
}
```

</details>

---

## Вывод

В результате выполнения домашнего задания:

1. Развёрнуты Grafana, Prometheus и Node Exporter.
2. Подключён источник данных Prometheus.
3. Создан Dashboard для мониторинга CPU, Load Average, RAM и дискового пространства.
4. Настроены четыре правила оповещений.
5. Проверена работоспособность Alert Rules.
6. Выполнен экспорт конфигурации Dashboard в JSON.
