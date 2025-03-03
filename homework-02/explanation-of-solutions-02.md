## Объяснение решений

### Бизнес-домены
**Главная проблема, которую решает MCF**: борьба со снижением мотивации котов-тестировщиков через делегирование рутинных задач специально отобранным котам-воркерам
#### Выделенные поддомены
Раз уж "благодаря отсеву и матчингу компания планирует выделяться на фоне конкурентов", то они и будут главными конкурентными преимуществами. Для каждого из них выделяем свой core-поддомен.

В целом, выделяем следующие поддомены:
- отбор воркеров: создание и конфигурация тестов, прохождение тестов кандидатами, отбор кандидатов. 
- матчинг воркеров под услуги: собственно, сам матчинг. Пока связывает клиента и воркера чисто по рандому, но явно планируется усложнение алгоритма.
- заказ и выполнение работ: оформление заказа на выполнение работы, само выполнение работы, расчеты с клиентами и воркерами. Явно важный поддомен, хоть и не ключевой (пойдет в supporting), явно сложный. Матчинг отделила как конкурентное преимущество, оценку качества работы — как то, что явно будет не так сильно меняться, да и сама по себе она явно проще, чем остальные компоненты, проблемы те же, что и у других компаний.
- оценка качества работы: именно то, что написано. Выделена отдельно по причинам, описанным выше. Идет в generic.
- снабжение расходниками: заказ fur-tune-печенья и сбор расходников под заказ. Можно ли использовать готовое решение? Возможно, не будь печенья, но пока напишем сами. Вдруг catGPT еще что-нибудь придумает. Тоже пусть будет supporting.
- управление ставками: тут все тоже довольно очевидно. Пишем сами — не думаю, что есть готовое решение под наши нужды, поскольку завязано на статусах наших заказов. Сильных изменений здесь не предвижу, бизнес без этого поддомена прожить вполне сможет, но оставим чисто, чтобы менеджеров мотивировать.

| Вид поддомена | Конкурентное преимущество | Сложность доменной модели | Изменчивость | Варианты реализации | Интерес проблемы | Предполагаемый вид поддомена |
|---|---|---|---|---|---|---|
| Отбор воркеров | да | высокая | высокая | ин-хаус | высокий | core |
| Матчинг воркеров под услуги | да | пока низкая, но со временем может стать высокой | высокая | ин-хаус | высокий | core |
| Заказ и выполнение работ | нет | высокая | высокая | ин-хаус | низкий | supporting |
| Оценка качества работы | нет | низкая | низкая | отдадим менее опытной команде | низкий | generic |
| Снабжение расходниками | нет | низкая | низкая | отдадим менее опытной команде | низкий | supporting |
| Управление ставками | нет | низкая | низкая | отдадим менее опытной команде, no-code | низкий | generic |

#### Core Domain Chart

![Core Domain Chart](https://github.com/RaileyHartheim/make-cats-free/blob/main/homework-02/MCF-core-domain-chart-02.png?raw=true)

Планируем, что матчинг в будущем заметно усложнится.

#### Боундед-контексты

| Вид поддомена | Предполагаемый вид поддомена | Выделенный боундед-контекст |
|---|---|---|
| Отбор воркеров | core | - Отбор воркеров|
| Матчинг воркеров под услуги | core | - Матчинг |
| Заказ и выполнение услуг | supporting | - Оформление и отслеживание заказов<br>- Выполнение работ<br>- Расчеты с клиентами и воркерами |
| Оценка качества работы | generic | - Оценка качества работы |
| Снабжение расходниками | supporting | - Снабжение расходниками |
| Управление ставками | generic | - Управление ставками |

Есть расхождение с Event Storming из предыдущей домашки: 
- Я не выделяла отдельно "Оценку качества работы": все соответствующие события были в составе контекста "Заказы". Но раз это то, что особо меняться не должно, да и не является критически важным для работы поддоменом, выделю отдельно и в ES.
- Создание клиентов и менеджеров можно включить внутрь других боундед-контекстов, а не выделять его как отдельный контекст.
- Выполнение работ, пожалуй, можно отделить от оформления и отслеживания заказов — чтобы не усложнять чересчур контекст.
- Еще решила названия контекстов поменять, а то они звучали не слишком по-деловому.

Кажется, Event Storming надо переделывать...

### Event Storming и модель данных

![](https://media.tenor.com/JF49Wc6yTgoAAAAe/eminem-mask.png)

И нет, это не Слим Шейди, а Event Storming и модель данных — правда, слегка обновленные.

![Event Storming](https://github.com/RaileyHartheim/make-cats-free/blob/main/homework-02/ES_02.png?raw=true)

![Data model](https://github.com/RaileyHartheim/make-cats-free/blob/main/homework-02/data_model_02.png?raw=true)

### Выделенные характеристики
Пробежимся по требованиям:
- *для решения проблемы утомляемости делегируем рутинные задачи котов-тестировщиков котам-воркерам* (обобщено из контекста) -> не надо утомлять тестировщиков дополнительно и усложнять им пользование сервисом: **usability**, **simplicity**
- *клиенты пока только тестировщики happy cat box, но компания планирует расширяться в будущем* + *Общая нагрузка на систему не будет превышать 10 заказов в день и 100 клиентов. Воркеров будет около 20 котов* -> нагрузка кажется не слишком высокой, но меняться в случае чего понадобится: **modifiability**, **agility**, **elaciticity**
- *для этого она планирует реализовать проект с нуля по заданным требованиям* -> проект будет меняться и явно усложняться со времнем: **modifiability**, **agility**, **testability**, **deployability**
- *Все коты-тестировщики (клиенты) автоматически попадают в систему*: -> храним сведения о клиентах: **securability**
- *Исследователи MCF пока думают о том, как должна работать инновационная система матчинга (тм)*: **modifiability**, **agility**, **testability**, **deployability**
- *Мы ожидаем 1к заявок в день от рандомных котов, также, судя по отзывам, наши конкуренты могут попытаться нас заддосить в этом месте*: **availability**, **scalability**, **elasticity**
- *По результатам тестов кот либо добавляется в общий пул рабочих с полученными характеристиками (сильными/слабыми сторонами) и прочей информацией о коте (имя, возраст, порода, цвет шерсти, фото), либо бракуется* -> храним информацию о работниках: **securability**
- *Бизнесу необходим низкий ТТМ (Time To Market), чтобы конкурировать на рынке*: **agility**, **testability**, **deployability**
- *Для бизнеса критично проверять новые гипотезы по отсеву котов и изменять уже существующие с максимальной скоростью и надёжностью*: **agility**, **performance**, **security**, **deployability**, **testability**
- _Деньги на данный момент не критичны, happy cat box готовы потратить столько, сколько потребуется_: явно не **cost**

В общем, для нас важны:
- modifiability
- agility
- deployability
- testability
- availability
- elaciticity
- securability
- performance
- usability
- simplicity

### Выбор архитектурного стиля
Посмотрели на характеристики и поняли...

![meme](https://github.com/RaileyHartheim/make-cats-free/blob/main/homework-02/meme-small-i-choose-you.png?raw=true)

Пожалуй, в этот раз остановимся на микросервисах: они должны лучше всего подойти для выбранных характеристик (изменяемость, гибкость, безопасность и вот это вот всё), да и о цене нам беспокоиться не надо.
Возможный DDOS заявками не должен повлиять на остальные сервисы.

### Итоговая модель системы

Каждому поддомену — свой сервис!

![Итоговая модель](https://github.com/RaileyHartheim/make-cats-free/blob/main/homework-02/final-structure-02.png?raw=true)

Коммуникации синхронные, поскольку с асинхронной коммуникацией не слишком знакома.
