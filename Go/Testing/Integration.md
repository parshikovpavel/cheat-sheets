# Integration testing

Интеграционные тесты – тестируют взаимодействие кода с каким-то внешним ресурсом:

- сервисом
- базой

Интеграционный тест гораздо сложнее unit теста, т.к. требует еще подготовки зависимости (`setUp`).

Например, для БД это означает, что перед запуском теста необходимо:

- поместить в таблицы фикстуры (setup)
- протестировать
- удалить все за собой (tearDown).

Эти тесты должны уже большей частью проверять не структуру кода (покрытие кода), а детали интеграции. Например, для БД – это проверка поведения кода при разных значениях в таблицах.