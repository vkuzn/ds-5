# Распределённое программирование. Задание #5 Масштабирование вычислительных ресурсов: конкурирующие потребители (Competing Consumers), каналы обработки (Pipes)
## Постановка задачи
Функционал системы оценки текста постепенно развивался и теперь это сложный процесс с множеством этапов. Также значительно выросло количество пользователей системы. 
Появились следующие проблемы:
1.	Процесс оценки текста занимает продолжительное время, что привело к увеличению времени ожидания ответа пользователем. Нужно свести время ожидания пользователя к приемлемому уровню.
2.	Определённые этапы оценивания требуют специфичного дорогостоящего оборудования. Вследствие монолитной реализации мы вынуждены разворачивать и выполнять на этом дорогостоящем оборудовании все этапы оценивания, что неэффективно с точки зрения финансовых затрат.
3.	Монолитный модуль оценки текста стал неудобен для развития и разворачивания новых версий.
## Реализация
Модифицируется компонент *TextRankCalc*. Добавляются компоненты *VowelConsCounter* и *VowelConsRater*.
1.	**TextRankCalc** – при запуске начинает слушать сообщения TextCreated в очереди backend-api. При получении сообщения отправляет сообщение TextRankTask(contextId) в очередь text-rank-tasks.
2.	**VowelConsCounter** – при запуске начинает слушать сообщения TextRankTask в очереди text-rank-tasks. При получении сообщения по contextId извлекает из Redis соответствующий текст, подсчитывает количество гласных и согласных букв и отправляет сообщение VowelConsCounted (contextId, vowelNum, consNum) в очередь vowel-cons-counter.
3.	**VowelConsRater** – при запуске начинает слушать сообщения VowelConsCounted в очереди vowel-cons-counter. При получении подсчитывает оценку текста как отношение значений vowelNum / consNum из сообщения и записывает в Redis полученную оценку по идентификатору контекста contextId, как это прежде делал TextRankCalc.
В скриптах запуска/остановки нужно реализовать возможность запускать/останавливать несколько экземпляров VowelConsCounter и VowelConsRater. Количество экземпляров каждого указывается через конфигурационный файл (например VowelConsCounter - 3 экземпляра, VowelConsRater – 2 экземпляра).

## Замечания
В RabbitMQ нужно использовать Direct Exchange для очередей text-rank-tasks и vowel-cons-counter

## Ссылки:

https://docs.microsoft.com/en-us/azure/architecture/patterns/pipes-and-filters

https://docs.microsoft.com/en-us/azure/architecture/patterns/competing-consumers

