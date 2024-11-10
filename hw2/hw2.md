# Домашняя работа 2 по курсу Системный дизайн высоконагруженных проектов от DevHands.io

## Условие
###### Кейс ДЗ#2. Планирование мощностей
1. неавторизованная главная страница Avito
2. неавторизованная главная страница Я.Маркет
3. неавторизованная главная страница Wikipedia
4. неавторизованная главная страница новостного портала


Сколько нужно ресурсов для обслуживания главной страницы 100M DAU (Daily Active Users)?

### Решение
###### Выбрал Avito

На вход на главную без авторизации по поддоменам только на домены Avit уходят следующие запросы с продолжительностью:

| ДЗ | Домен | Поддомен | Время   |
|----|-------|----------|---------|
| ru | avito | cs       | 88.9    |
| ru | avito | cs       | 101.64  |
| ru | avito | cs       | 79.62   |
| ru | avito | cs       | 78.64   |
|    |       | Total    | 348.8   |
|    |       |          |         |
| ru | avito | sntr     | 67.65   |
|    |       | Total    | 67.65   |
|    |       |          |         |
| ru | avito | stats    | 73.29   |
| ru | avito | stats    | 74.1    |
| ru | avito | stats    | 71.74   |
| ru | avito | stats    | 112.1   |
| ru | avito | stats    | 140.2   |
|    |       | Total    | 471.43  |
|    |       |          |         |
| ru | avito | www      | 55.28   |
| ru | avito | www      | 45.21   |
| ru | avito | www      | 43.54   |
| ru | avito | www      | 26.27   |
| ru | avito | www      | 25.77   |
| ru | avito | www      | 25.86   |
| ru | avito | www      | 43.7    |
| ru | avito | www      | 686.23  |
| ru | avito | www      | 455.31  |
| ru | avito | www      | 51.07   |
|    |       | Total    | 1458.24 |


Итого для упрощения будем считать 4 сервиса:

| Сервис | Время   |
|--------|---------|
| cs     | 348.8   |
| sntr   | 67.65   |
| stats  | 471.43  |
| www    | 1458.24 |

Для упрощения также будем считать:

1. пользователь заходит на страницу 1 раз в день.
2. В дневные промежутки времени (с 8 до 16) и вечерние промежутки времени (с 16 до 24) пользователи заходят в 2 раза чаще, чем в ночные промежутки времени (с 0 до 8)
3. Сервисы расположены на одной машине и во время ожидания ответа входят все вычисления от маршрутизации запроса до считывания данных из носителя хранилища

Тогда RPS будет распределяться следующим образом:

- в дневные и вечерние промежутки времени `100М / 24 / 60 / 60 / 5 * 2 = 473`
- в ночные промежутки времени `100М / 24 / 60 / 60 / 5 * 2 = 234`

И нагрузка на сервисы, соответственно, будет следующая:

| Сервис | Время, мс | Выч.нагрузка день, с | Выч.нагрузка ночь, с |
|--------|-----------|----------------------|----------------------|
| cs     | 348.8     | 112                  | 82                   |
| sntr   | 67.65     | 32                   | 16                   |
| stats  | 471.43    | 223                  | 111                  |
| www    | 1458.24   | 690                  | 342                  |

Для обработки этих запросов необходимы вычислительные мощности кратные в количестве ядер процессора равному нагрузке на сервис в секундах. Так как предполагалось что днем и вечером нагрузка разная, то за нормальную нагрузку берется максимальная. Предположив, что на серверах стоят 24 ядерные процессоы AMD EPYC 9274F OEM, получаем таблицу с количеством процессоров на сервис.


| Сервис | Выч.нагрузка день, с | Количество процессоров |
|--------|----------------------|------------------------|
| cs     | 112                  | 3                      |
| sntr   | 32                   | 2                      |
| stats  | 223                  | 10                     |
| www    | 690                  | 29                     |