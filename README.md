# balshoy-dispetcher

```mermaid
sequenceDiagram
    autonumber
    actor User as Пользователь

    box rgb(227, 242, 253) "Слой 3: Оркестратор (LLM-as-a-OS)"
        participant Router as Роутер / Agentic Orchestrator<br>(GPT-4 / Llama-3)
    end

    box rgb(255, 243, 224) "Слой 1: Реестр (Registry)"
        participant Registry as Model Registry<br>(Harbor / Hugging Face)
    end

    box rgb(232, 245, 233) "Слой 4: Пайплайн (Execution)"
        participant Pipeline as Execution Pipeline<br>(Ray / DAG Runner)
    end

    box rgb(243, 229, 245) "Слой 2: Интерфейсы (Inference)"
        participant KubeFlow as Кластер Моделей<br>(Kubernetes / Kubeflow)
    end

    %% Основной сценарий
    User->>Router: Запрос: "Сделай отчет по продажам за вчера"

    %% Этап планирования
    Note over Router, Registry: ФазаDiscovery: Поиск "умений"
    Router->>Registry: Запросить список доступных моделей и метаданные
    Registry-->>Router: Model Cards (Model A: Дата, Model B: SQL, Model C: Аналитика)

    Note over Router: Фаза Planning (Agentic)<br/>1. Понять Intent<br/>2. Составить план (DAG)<br/>3. Выбрать модели A -> B -> C

    %% Этап выполнения
    Router->>Pipeline: Передать DAG (План выполнения)
    activate Pipeline

    %% Шаг 1: Извлечение даты
    Pipeline->>KubeFlow: Вызов Model A (Извлечение даты)
    KubeFlow-->>Pipeline: Результат: "2024-01-01"
    Note right of Pipeline: Промежуточный результат 1

    %% Шаг 2: Генерация SQL
    Pipeline->>KubeFlow: Вызов Model B (SQL Генератор)<br/>Вход: "2024-01-01"
    KubeFlow-->>Pipeline: Результат: Data Set (Продажи)
    Note right of Pipeline: Промежуточный результат 2

    %% Шаг 3: Анализ
    Pipeline->>KubeFlow: Вызов Model C (Аналитик)<br/>Вход: Data Set
    KubeFlow-->>Pipeline: Результат: Текст отчета
    
    Pipeline-->>Router: Финальный результат (Отчет)
    deactivate Pipeline

    Router-->>User: Ответ: "Вот отчет по продажам..."
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
