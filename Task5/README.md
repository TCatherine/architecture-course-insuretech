# GraphQL-схема сервиса client-info.

Эквивалент REST-контракта (Swagger 2.0, /v1):
  GET /clients/{id}            -> Client { id, name, age }
  GET /clients/{id}/documents  -> [Document]
  GET /clients/{id}/relatives  -> [Relative]

## Cущность Client
Все три ресурса объединены в одну сущность Client с вложенными полями.
Потребитель сам выбирает, какие поля и под-объекты вернуть в ответе —
это закрывает оба сценария:
  - не нужно делать несколько отдельных запросов за разными
    объектами карточки клиента (как в REST с множеством ресурсов);
  - не передаются все ~500 атрибутов клиента целиком, если они не нужны
    (как было бы при одном "толстом" REST-ресурсе).

```
type Query {
  """
  Единая точка входа для получения данных клиента.
  Заменяет GET /clients/{id}, GET /clients/{id}/documents,
  GET /clients/{id}/relatives — конкретный набор полей/связей
  определяется запросом потребителя.
  """
  client(id: ID!): Client
}
```

## Карточка клиента
Базовые поля соответствуют definitions.Client из Swagger.
Документы и родственники — вложенные ресурсы, ранее доступные
только через отдельные REST-эндпоинты.
```
type Client {
  id: ID!
  name: String
  age: Int

  """
  Эквивалент GET /clients/{id}/documents.
  Запрашивается только если явно указано в выборке полей.
  """
  documents: [Document!]

  """
  Эквивалент GET /clients/{id}/relatives.
  Запрашивается только если явно указано в выборке полей.
  """
  relatives: [Relative!]

  # Точка расширения: остальная часть из ~500 атрибутов карточки
  # клиента (контакты, адреса, банковские реквизиты и т.д.)
  # подключается как дополнительные типы/поля по тому же принципу,
  # без изменения существующих полей и без отдельных REST-ресурсов:
  #
  # contacts: [Contact!]
  # addresses: [Address!]
}
```

Соответствует definitions.Document из Swagger.
```
type Document {
  id: ID!
  type: String
  number: String
  issueDate: String
  expiryDate: String
}
```


Соответствует definitions.Relative из Swagger.
```GrapthQL
type Relative {
  id: ID!
  relationType: String
  name: String
  age: Int
}
```