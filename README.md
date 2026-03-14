# balshoy-dispetcher

```mermaid
sequenceDiagram
    participant C as Клиент
    participant R as Роутер (Router)
    participant Reg as Registry (Git)
    participant M as Мозг (LLM, опционально)
    participant A as Airflow (Pipeline)
    participant Model1 as Мини-модель А
    participant Model2 as Мини-модель Б
    participant ModelN as MCP-серверы

    C->>R: Запрос (например, "сделай отчет")
    
    rect rgb(240, 240, 255)
        Note over R,Reg: Фаза 1: Маршрутизация
        R->>Reg: Какие модели умеют X?
        Reg-->>R: Список моделей и их MCP-эндпоинты
        
        alt Правила сработали
            R->>R: Быстрый rule-based выбор
        else Нужен "мозг"
            R->>M: Сложный случай, помоги выбрать
            M-->>R: Рекомендация: ModelA -> ModelB
        end
    end
    
    rect rgb(255, 240, 240)
        Note over R,A: Фаза 2: Построение пайплайна
        R->>A: Сформировать DAG (ModelA → ModelB)
        A-->>R: Пайплайн создан
    end
    
    rect rgb(240, 255, 240)
        Note over A,ModelN: Фаза 3: Выполнение через MCP
        A->>Model1: MCP-вызов с данными
        Model1-->>A: Результат А
        
        A->>Model2: MCP-вызов(результат А)
        Model2-->>A: Результат Б
        
        Note over A,ModelN: ... и так далее по DAG
    end
    
    A-->>R: Финальный результат
    R-->>C: Ответ клиенту
```

```mermaid
graph TB
    subgraph "Твоя система"
        Router[Роутер<br/>гибридный: правила + LLM]
        
        subgraph "Реестр (Git)"
            direction TB
            Git[(Git-репозиторий)]
            Configs[model-card.yml<br/>эндпоинты<br/>метаданные]
        end
        
        subgraph "Пайплайн"
            Airflow[Airflow<br/>с AI SDK]
        end
        
        subgraph "Модели как MCP-серверы"
            direction LR
            M1[MCP: извлечение дат]
            M2[MCP: SQL-генератор]
            M3[MCP: тональность]
            M4[...]
        end
    end
    
    subgraph "Внешнее"
        Client[Клиент]
        Brain[LLM-мозг<br/>для сложных случаев]
    end
    
    Client --> Router
    Router --> Git
    Router --> Brain
    
    Router --> Airflow
    Airflow --> M1
    Airflow --> M2
    Airflow --> M3
    Airflow --> M4
    
    M1 --> Airflow
    M2 --> Airflow
    M3 --> Airflow
    M4 --> Airflow
    
    Airflow --> Router
    Router --> Client
    ```
