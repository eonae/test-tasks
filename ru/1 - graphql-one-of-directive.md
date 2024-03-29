# Директива @oneOf для graphql

|  |  |
|--|--|
| Уровень кандидата | middle+/senior |
| Ожидаемая трудоёмкость | 8 - 16 часов |
---

Трудоёмкость зависит от уровня кандидата. Задание подразумевает не proof-of-concept, а `законченную функциональность, выполненную полноценно и качественно.` Наградой за потраченное время для кандидата будет не только итоговое успешное трудоустройство (если повезёт), но и объективно полезная для сообщества библиотека в opensource (в любом случае)

---

## Контекст

В GraphQL есть известная проблема, с полиморфизмом инпутов. Если в для объектных типов "type" можно определять union'ы и interface'ы, то для input'ов такой возможности нет.

Допустим есть задача: на вход в методе login принимать либо телефон, либо email пользователя.

``` graphql
type Mutation {
  login(input: LoginInput!): TokenPair!
}

input LoginInput {
  login: Login!
  password: Password!
}
```

Чтобы определить тип логина можно либо использовать скаляр:

```graphql
scalar Login
```

и уже на уровне приложения смотреть что пришло.

Также можно использовать объект такого вида:

```graphql
mutation Login {
  email: Email
  phone: Phone
}
```

Мы как бы имеем ввиду, что должно быть либо одно, либо другое. Но, к сожалению, это только "договорённость", мы никак это не может энфорсить.

Сказать (задекларировать), что мы хотим такое поведение, можно при помощи директив (есть встроенные, типа @deprecated, а можно делать и кастомные). К примеру, можно сделать директиву `@oneOf` (не дефолтная):

```graphql
mutation Login @oneOf {
  email: Email
  phone: Phone
}
```

А дальше мы хотим, чтобы запрос типа:

```graphql
# Запрос
mutation {
  login(input: { login: { phone: "xxx", email: "yyy" }, password: "zzz" })
}
```

сделать было невозможно. То есть, любой клиентский код из такой схемы просто не должен сгенерироваться и никакой playground не должен такой запрос пропустить. Но это задача фронтовая, в скоуп данного тестового задания она не входит (хотя, если самому будет интересно...)

А на бэкенде, если всё-таки такой запрос пришёл (например, его сделали просто postman'ом - сырой HTTP), мы хотим, чтобы он был отбит, как не соответствующий схеме.

## Задача

1. Создать директиву `@oneOf`, которую можно будет навесить на тип `input`.
2. Создать обработчик для этой директивы, который будет отклонять вопрос с одновременно указанным email и phone, как невалидный **на уровне graphql, основываясь исключительно на схеме**. Разработчик метода login на бэке не должен написать ни одной проверки на одновременное присутствие email и phone. В коде приложения мы можем только проверить **что именно нам пришло**, например, при помощи type guard'ов `isPhone` или `isEmail` (написать их разработчику конечно придётся), будучи уверенными в том, что там либо одно, либо другое.
3. Написать тестовое приложение. Оно только должно принимать входные данные и логировать их - больше ничего. Ни БД ни какого-то бы ни было другого функционала не требуется.

### Дополнительные требования (обязательные)

1. Обработчик директивы должен быть отделяем (а лучше отделён - см. ниже) от кода приложения. То есть, это должна быть библиотека, которую можно использовать в разных проектах.
2. На бэкенде используем Nest.js с apollo (там ещё есть опция с mercurius - она нас не интересует)
3. Подход должен быть schema-first (не code-first!)
4. Обязательно тесты. Тип, объём и архитектура тестов - на усмотрение исполнителя.
5. Библиотечная часть должна быть задокументирована так, как будто бы это будет выложено в opensource для публичного использования (так и будет, в случае успеха)

### Рекомендации (на усмотрение кандидата)

1. Будет удобно, если решение будет оформлено в виде монорепозитория (например, под управлением lerna), в котором одним из пакет будет сервис, а вторым - библиотека.

## Что будет оцениваться

1. "Вникновение" а задачу. Здесь она поставлена примерно в том же формате, который будет на практике в проекте. Важно, чтобы кандидат по описанному смог понять контект и, при необходимости, догуглить остальное.
2. Корректность - запустить и проверить не поленимся!
2. Качество кода. Лаконичность, форматирование (как ни пошло это звучит), ясность и простота восприятия. Оценка субъективная, но покажет, насколько у нас с кандидатам общее "чувство прекрасного".
3. Качество и полнота тестов.
4. Качество и ясность документации.
5. Общая аккуратность и внимание к деталям.

### Удачи! 🚀🚀🚀