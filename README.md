Система анализа качества разговоров 📞
Добро пожаловать в репозиторий Conversation Quality Analysis System! Это модульное и масштабируемое решение для анализа качества клиентских разговоров с интеграцией с Creatio CRM. Проект использует инструменты ИИ, Docker-контейнеры и рабочие процессы n8n для оценки метрик разговоров, таких как тональность, вежливость и соблюдение скрипта, а также предоставляет рекомендации для улучшения взаимодействия с клиентами.

📑 Обзор проекта
Система автоматизирует анализ клиентских разговоров (например, телефонных звонков), оценивает их качество и генерирует рекомендации для менеджеров. Она интегрируется с Creatio CRM для получения данных и хранения результатов, используя:

AI-инструменты: Модели для анализа тональности (Hugging Face), извлечения тем (DeepPavlov), морфологического анализа (pymorphy2) и генерации рекомендаций (GPT-4o).
Docker-контейнеры: Для модульности и упрощения развертывания.
n8n: Для автоматизации процессов и интеграции между компонентами.
PostgreSQL: Для хранения результатов анализа.

Проект разработан для упрощения мониторинга, отладки и масштабирования, с возможностью добавления новых модулей анализа.

🚀 Возможности

Анализ качества разговоров:
Тональность (позитивная, нейтральная, негативная).
Вежливость (подсчет вежливых фраз).
Скорость ответа и соблюдение скрипта (частично реализовано).
Выявление проблемных зон и ключевых тем.
Активное слушание и перебивания.


Числовая оценка: Итоговый балл по 10-балльной шкале на основе взвешенных метрик.
Генерация рекомендаций: Использует GPT-4o для создания персонализированных предложений.
Интеграция с Creatio CRM: Получение данных через Webhook и отправка результатов через OData API.
Модульная архитектура: Легкое добавление новых инструментов анализа.
Логирование и мониторинг: Доступ к логам через docker logs.


📂 Структура проекта

sentiment_server.py  

Python-скрипт для анализа текста (тональность, темы, проблемы, рекомендации).  
Развернут в контейнере sentiment-analyzer, принимает HTTP-запросы от n8n.  
Использует модель blanchefort/rubert-base-cased-sentiment (Hugging Face) и GPT-4o.


Docker-контейнеры  

preprocessor: Предобработка текста (spaCy, pymorphy2).  
sentiment-analyzer: Анализ тональности и рекомендаций.  
ner-analyzer: Извлечение тем (DeepPavlov NER).  
summarizer: Обобщение разговоров (T5).  
Все контейнеры объединены в сеть my-network.


n8n workflows  

Управляет процессом: получение текста через Webhook, отправка на анализ, сохранение результатов.  
Интеграция с Creatio CRM и PostgreSQL.


PostgreSQL  

Хранит результаты анализа (CallId, Sentiment, Themes, Issues, Score, Recommendations).




⚙️ Установка и запуск
1. Требования

Docker и Docker Compose
Python 3.8+
PostgreSQL
Аккаунт OpenAI для GPT-4o
Доступ к Creatio CRM (OData API)

2. Установка зависимостей
Создайте файл requirements.txt:
fastapi
uvicorn
transformers
torch
sentence-transformers
pymorphy2
spacy
deeppavlov
openai
psycopg2
requests

Установите зависимости:
pip install -r requirements.txt

3. Конфигурация

Создайте файл .env в корневой директории:

DB_NAME=your_database
DB_USER=your_user
DB_PASSWORD=your_password
DB_HOST=your_host
DB_PORT=5432
OPENAI_API_KEY=your_openai_key
CREATIO_URL=https://your-creatio-instance.com
CREATIO_LOGIN=your_creatio_login
CREATIO_PASSWORD=your_creatio_password


Настройте Docker Compose (docker-compose.yml):

version: '3.8'
services:
  preprocessor:
    build: ./preprocessor
    networks:
      - my-network
  sentiment-analyzer:
    build: ./sentiment-analyzer
    ports:
      - "8000:8000"
    networks:
      - my-network
  ner-analyzer:
    build: ./ner-analyzer
    networks:
      - my-network
  summarizer:
    build: ./summarizer
    networks:
      - my-network
  postgres:
    image: postgres:latest
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - my-network
  n8n:
    image: n8n
    ports:
      - "5678:5678"
    networks:
      - my-network
networks:
  my-network:
    driver: bridge
volumes:
  postgres_data:


Настройте базу данных PostgreSQL:

CREATE TABLE call_analysis (
    call_id VARCHAR(255),
    sentiment VARCHAR(50),
    themes TEXT[],
    issues TEXT[],
    score INTEGER,
    recommendations TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

4. Запуск

Запустите Docker Compose:

docker-compose up -d


Настройте n8n workflow:

Создайте Webhook для получения текста от Creatio.
Настройте HTTP-запросы к контейнерам (http://sentiment-analyzer:8000, etc.).
Настройте сохранение в PostgreSQL и отправку в Creatio.


Настройте Creatio:

Создайте объект CallAnalysis с полями: CallId, Sentiment, Themes, Issues, Score, Recommendations.
Настройте Webhook и OData API для интеграции.




📊 Использование

Отправка текста разговора:
Creatio отправляет расшифровку через Webhook в n8n.
Пример входных данных (JSON):



{
  "call_id": "67890",
  "text": "Клиент: Добрый день, меня зовут Сергей. Я получил заказ, номер 67890, но товар пришёл с дефектом..."
}


Анализ:

Текст очищается, анализируется на тональность, темы, проблемы, вежливость.
Рассчитывается оценка (0–10).
Генерируются рекомендации через GPT-4o.


Результат:

Сохраняется в PostgreSQL.
Отправляется в Creatio (OData API).
Пример результата:



{
  "call_id": "67890",
  "sentiment": "negative",
  "themes": ["доставка", "качество"],
  "issues": ["жду", "дефект", "неловкий"],
  "score": 0,
  "recommendations": "Проблема не решена: клиент остался недоволен. Рекомендуется предложить немедленный возврат средств..."
}


📄 Метрики анализа

Тональность: Позитивная/нейтральная/негативная.
Вежливость: Количество вежливых слов.
Проблемные зоны: Список проблемных слов/фраз.
Темы: Ключевые темы разговора.
Оценка: Итоговый балл (0–10).
Рекомендации: Персонализированные предложения для менеджера.


🔧 Интеграция с Telegram (опционально)
Для мониторинга системы можно интегрировать c Telegram


