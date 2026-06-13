# Решение

1. osago-aggregator получает своё хранилище — отдельный Redis Cluster (короткоживущее состояние опроса СК + общее состояние Circuit Breaker).

2. API core-app для osago-aggregator:
    - API к core-app: `POST /osago/requests` -> 202 Accepted, результаты — через Kafka-топик `osago.offer.received`.

3. Cредство интеграции между сервисами core-app и osago-aggregator:
    - core-app -> osago-aggregator RAST API
    - osago-aggregator -> core-app через Kafka

4. API core-app для веба:
    - API веба: `POST /api/osago/applications`.

5. Паттерны отказоустойчивости:
    - Для взаимодействия osago-aggregator и страховые компании требуются:
        - Circuit Breaker (если СК не доступен, то берем состояние из редиса)
        - Retry
        - Timeout
