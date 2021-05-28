# Тема проекта - Vkontakte

## *MVP*

NewsFeed - приложение, в котором можно подписываться на других людей, смотреть их посты, и выкладывать свои.

### Целевая аудитория:

* [77% от всей мобильной аудитории в России – первое место.](https://vk.com/@cerebro_vk-polzovateli-socsetei-v-rossii-statistika-i-portrety-auditori)
* 73 миллионов активных пользователей в месяц.
* [42 млн человек в день.](https://ppc.world/articles/auditoriya-shesti-krupneyshih-socsetey-v-rossii-v-2020-godu-izuchaem-insayty/)
* Россиская Федерация

### Рассчет нагрузки

1) #### *Рассчет нагрузки на создание постов:*
   По данным статистики
   за [май 2017](https://vk.com/@cerebro_vk-polzovateli-socsetei-v-rossii-statistika-i-portrety-auditori):
    - 25 721 668 - число авторов, сделавших пост за месяц
    - 310 795 150 - число постов в месяц

   Зная число постов в месяц, можем посчитать сколько постов в среднем делается за один день:

       310 795 150 / 31 = 10 025 650 постов/день

Теперь посчитаем число запросов в секунду, мы знаем что наибольший пик активности в Вконтакте приходится на обед с 11 до
13, и вечером с 17 до 23 =>
Итого: [8 часов в день](https://postium.ru/luchshee-vremya-dlya-publikacii-postov-v-instagram-vk/)

Тогда для получения числа постов в секунду:

    10 025 650 / (8 * 3600) ~ 348 запросов/c

2) #### *Рассчет нагрузки на создание ленты:*
   Пусть каждый пользователь делает по 7 запросов в день на создание ленты, зная число пользователей в среднем в день (
   42 млн) и наибольший пик активности равный 8 часам, получим:


      42 * 10^6 * 7 / (8 * 3600) = 10 208 запросов/c


3) #### *Рассчет нагрузки на создание комментариев:*
    Мы знаем что в день создается примерно *~10 млн запросов*
    Значит, если учесть что в среднем каждый созданный запрос за 8 активных часов соберет
    50 комментариев в среднем, то тогда:
   
   
 
    10 000 000 запросов/день * 50 комментариев / (8 * 3600)сек = 17 360 запросов/сек



4) #### *Рассчет нагрузки на чтение комментариев:*
    Так как, у нас *10 208 запросов/с* на создание ленты, пусть у каждого такого поста пользователь
    будет читать в среднем 10 комментариев, то есть подгружать с каждым таким постом, в среднем 10 комментов
    Тогда:
   


    10 208 * 10 = 102 080 запросов/сек на чтение комментов



5) #### *Рассчет нагрузки на создание лайков:*
   Мы знаем что в день создается примерно *~10 млн запросов*
   Значит, если учесть что в среднем каждый созданный запрос за сутки активных соберет
   500 лайков в среднем, то тогда:



    10 000 000 запросов/день * 500 лайков / (24 * 3600)сек = 57 870 запросов/сек



### Итоги по РПС сервиса:

* Создание постов ~ 350 запросов / секунду (на запись)
* Составление ленты ~ 10 000 запросов / секунду (на чтение)
* Создание комментариев ~ 17 000 запросов / секунду (на запись)
* Чтение комментариев ~ 102 000 запросов / секунду (на чтение)
* Создание лайков ~ 58 000 запросов / секунду (на запись)

### Итого:
* ~ 110 000 запросов/сек на чтение
* ~ 75 000 запросов/сек на запись

### Логичекая схема Базы Данных

Сущности в моем проекте:

1) Пользователь
2) Пост
3) Подписки
4) Лайки
5) Темы
6) Рекомендации
7) Комментарии

Каждая из этих моделей необходима для формирования ленты для пользователя, если немного углубиться, то подразумевается
две ленты одна формируется исходя из подписок пользователя, вторая из рекомендаций. Также необходимо пояснить что такое
темы, темы - это механизм, который используется для формирования рекомендаций, с их помощью мы будем давать пользователю
только тот контент, который ему интересен. (пример тем: спорт, автомобили и т.д)
Рекомендации намеренно денормализованы так, чтобы для каждого пользователя хранить рекомендуемые ему посты. Также пользователи,
хранят внутри себя последние 10 выложенных постов, в будущем это будет использоваться
для составления ленты. Остальные
модели хранят исключительно id, без полных сущностей.

![Логическая схема БД](https://github.com/Amaterasy17/HighLoad-courseWork/raw/main/images/highload_physical.svg)

### Физическая схема Базы Данных

Для хранения большинства данных будем
использовать реляционную СУБД PostgreSql, она обладает высокой надежностью и ее будет достаточно для создания новых в
ней данных, таких как посты, пользователи, лайки. В ее схему будут входит следующие модели:
пользователь, лайк, подписки, темы, пост, промежуточные таблицы для отношения М-М. Для ускорения записи
применим шардирование, чтобы выдерживать нагрузку на запись лайков. Для постов можно использовать шардирование по
числу лайков, это будет чем-то вроде рейтинга, с помощью которого мы будем делить записи.

Комментарии я решил вынести в отдельную сущность БД PostgreSql. Сделано это по следующей причине, нам нужно будет поддерживать
нагрузку не только на запись в БД, но и на чтение для комментов. Поэтому мы применим шардирование у комментариев, так как 
мы комментируем какой либо пост, можно шардировать по числу лайков у поста. Также мы будем использовать реплики,
для повышения скорости чтения, и огромное число комментариев будет реплицироваться, и число реплик для комментов,
будет куда больше
чем число реплик необходимое для тех же лайков, постов и других сущностей из 1 БД. Таким образом, я их вынес, чтобы не затрачивать
больше памяти чем в действительности нужно.


Также у нас будет большая нагрузка на чтение рекомендаций, следовательно оптимально будет использовать для чтения нереляционную
in-memory БД, такую, например, Redis. Она быстрая и доступ к данным внутри нее будет практически равен доступу к
оперативной памяти. Поэтому список рекоммендаций будем хранить в данной нереляционной СУБД. Рекомендации будут
представлять собой id пользователя, и список рекомендуемых ему постов, которые будут определяться по темам, интересных
пользователю.

Также часть нагрузки на составление ленты, будет распределятья на PostgreSql в случае получения человеком постов по
подпискам(иными словами получение постов друзей). Для увелечения отказоустойчивости и ускорения чтения, будем использовать
реплики. Пользователь, который загружает новые посты и не заметит отсутствия постов, которые были созданы только что.
Еще для ускорения составления ленты по списку друзей, добавим в сущность Пользователь, массив постов, который будет содержать
последние 10 свежих постов пользователя, с учетом того у каждого пользователя в среднем по ~1000 подписок + добавляются все новые посты,
засчет такой денормализации мы получим выйгрыш в скорости выполнения запросов и сможем создавать ленту быстрее.

Для хранений сессий пользователей будет оптимальным использовать нереляционную in-memory СУБД, так что и для данной
задачи нам подойдет Redis.

![Физическая схема БД](https://github.com/Amaterasy17/HighLoad-courseWork/raw/main/images/highload_physical_scheme.svg)

### Выбор технологий

#### *Frontend*

Здесь мы будем использовать следующее сочетание: *HTML* + *SCSS* + *TypeScript*. Используем TypeScript так как он
статически типизируемый и обладает большим спектром возможностей, чем JS, потому что является его надстройкой. Важно
выбрать библиотеку для фронта, на самом деле все зависит от команды разработки, от того с чем они знакомы, но для
примера возьмем одну из популярных библиотек такую как React. Для сборки бандлов и поддержки старых браузеров будем
исползовать webpack и babel. Также можно использовать SSR (Server Side Rendering) на Next.js для продвижения сайта вверх
в списке поиска и улучшения восприятия пользователем сайта, благодаря быстрой загрузке стартовой страницы.

#### *Backend*

Здесь будем использовать микросервисную архитектуру, которая позволит легко масштабироваться, а также контролировать
нагрузку на сервисы. Основным языком в микросервисах будет *Golang*, так как он поддерживает параллелизм из коробки и
является очень быстрым, производительным. Обязательно будет микросервис рекомендаций, который будет написан на *C++*,
используем его, так как нам важна максимальная скорость и производительность. Данный микросервис на плюсах будет ходить
в реплики postgreSQL для формирования рекомендаций пользователям, результат работы будет складывать в Redis.
Микросервисы будут общаться между собой по gRPC.

#### *Mobile*

Здесь можно использовать *Kotlin* для Android, *Swift* для IOS.

### Схема проекта

1. *Выбор оборудования под сервера и его конфигурация*
   
   В нашем проекте, нам нужно рассчитать конфигурации и число серверов для:
    * frontend
    * backend
    * сервера СУБД PostgreSQL
    * сервера СУБД PostgreSQL комментариев
    * сервера СУБД Redis
    * балансировщика

*Балансировщик*

Из личного практического опыта, Nginx выдает на компьютере с 4 логическими ядрами выдает *~12000 rps*
Значит одно ядро дает примерно 3000 rps. Возьмем тогда массовый современный процессор Intel E5-2620 v4 – 8 ядер Он будет
выдавать *8 * 2 * 2 * 3000 rps = 96000 rps*
Два балансировщика с такой конфигурацией нам будет хватать в притык исходя из нашей нагрузки. Для будущего масштабирования,
DNS балансировки возьмем 4 балансировщика нагрузки с данной конфигурацией.

*Frontend*

Так как для любого приложения важна скорость, нужно определиться с размером подгружаемых бандлов, мы будем всячески
оптимизировать код и стараться сделать из [5Мб](https://habr.com/ru/company/oleg-bunin/blog/433324/) -> 700 Kб. Пусть,
возьмем с запасом бандл будет ~ 1Мб, тогда зная среднее число пользователей в день - 42 000 000, а значит в среднем в
час 1 750 000, значит в худшем случае в час гоняется 1 750 000 Мб данных. Получается, ~ 30Гб/мин, => 500Mb/s. 1 SSD
имеет скорость чтения/записи: [250MB/s](https://cloud.mail.ru/public/pF76/yAuuaRA4r)
Значит поставим на 1 фронтенд сервер 2 SSD на 1 ТБ. и Возьмем прозапас 2 фронт сервера. Так как фронтовые сервера, обычно
располагаются на той же машине что и виртуалка, так как в основном они отдают сбандленную статику, фронтовых серверов будет как и 
балансировщиков 4.

*Backend*

Считаю, будет разумным учитывать rps не только самого языка бэкенда, а сочетание языка + СУБД. Также из моего
практического опыта(из курса по БД):
Сочетание golang + postgreSQL, с уловием что 1 запрос - 1 SQL запрос, в компьютере с 8Гб ОЗУ + процессором на физических 2 ядра (логических 4),
дает на запись ~2200 rps (вносилось около 1,7 млн записей за 12 минут), это означает что при использовании разумного
шардинга, а также серверной конфигурации, проблем с выдерживанием рассчитанной нагрузки на запись не будет. Загвоздка
возникает для чтения, также из практического опыта, комп с 2 ядрами дает 2000rps, в то время как комп с 4 дает 4000rps.
Можно заметить линейный рост при чтении при увеличении числа ядер. Значит, 1 бэк с 56 ядрами, 64Гб ОЗУ даст примерно
28000 rps. Получается, что для того чтобы в притык выдерживать нагрузку на сервис нужно минимум 7 серверов.
Также производительность на ядрах может снижаться при увеличении числа хранимых данных, поэтому лучше взять с запасом.
Таких серверов лучше взять 35 (для обеспечения отказоустойчивости). Таким образом, если производительность на 1 ядре упадет
до 100 rps сервис выдержит нагрузку.

*Примечание: из практики можно заметить что 1 физическое ядро дает около 1000 rps, логическое же от 500-600rps*

Также возьмем еще один сервер с подобной конфигурацией для микросервиса на C++, нам необходимо чтобы внесение новых
рекомендаций в Redis, было чуть медленнее чтения из него, значит для 8000rps на запись, нам понадобится процессор с

*8000/600rps = 13,3 логический ядер*

То есть у данного сервера будет 64Гб ОЗУ + массовый процессор Intel E5-2620 v4 – 8 ядер

*СУБД PostgreSQL*

Здесь нам важно взять как можно больше ОЗУ, для кэширования данных, а также необходимо рассчитать число дисков под БД.
Вспомним модели и примерный вес 1шт каждой из них. Будем считать что int - 4 байта, date - 15 байт, varchar - 2 байта.

* Пользователь ~ (1300 + 10 * 2500) байт/запись * [460млн](https://ru.wikipedia.org/wiki/%D0%92%D0%9A%D0%BE%D0%BD%D1%82%D0%B0%D0%BA%D1%82%D0%B5#:~:text=%D0%9F%D0%BE%20%D0%B4%D0%B0%D0%BD%D0%BD%D1%8B%D0%BC%20%D0%BD%D0%B0%20%D0%B0%D0%B2%D0%B3%D1%83%D1%81%D1%82%202017,%D0%BC%D0%B5%D1%81%D1%82%D0%BE%20%D0%BF%D0%BE%20%D0%BF%D0%BE%D0%BF%D1%83%D0%BB%D1%8F%D1%80%D0%BD%D0%BE%D1%81%D1%82%D0%B8%20%D0%B2%20%D0%BC%D0%B8%D1%80%D0%B5.) = 12 Тб
* Пост ~ 2500 байт/запись * 310 млн * 12 ~ 10 Тб данных/год
* Подписки ~ 12 байт/запись, учитывая, что каждый подписан в среднем на 1000 человек, тогда выделим примерно 200Тб
* Лайки ~ 12 байт/запись, учитываем что каждый пост в среднем набирает 500 лайков, тогда выделим примерно 2.5 Тб/год => выделим 100 Тб
* Темы ~ 200 байт/запись, примерно 2Тб
  Итого нужно с запасом: 300 Тб SATA памяти для лайков и подписок, а
  также 215ТБ SSD для пользователя, постов и тем.
  Таких серверов у нас будет 4 (1 мастер и 3 реплики)



*СУБД PostgreSQL комментариев*

Здесь нам важно взять также как можно больше ОЗУ, для кэширования данных, а также необходимо рассчитать число дисков под БД.

* Комментарий ~ 450 байт/запись, так как 310 млн постов в год и они в среднем собирают по 100 комментов,
получается 14 Тб/год, значит нужно выделить примерно 300 Тб.

  Итого нужно с запасом: 300 Тб SSD памяти в данной БД, берем SSD для увеличения скорости чтения.
  Таких серверов у нас будет 15 (1 мастер и 14 реплики)

  
*СУБД Redis*

Здесь главное рассчитать нужное число ОЗУ для данной СУБД, так как
один пост ~ 2500 байт запись, а в месяц мы храним около 310 млн постов, желательно хранить хотя бы 2 месяца, для
рекомендаций, тогда получается ~ 2500 * 310 ~ 750 Гб ОЗУ/месяц, тогда берем в одну БД 1.5 Тб ОЗУ (макс конфигурацию)
Таких у нас будет два сервера, один мастер для записи и реплика для чтения.

*Итого*

| Оборудование      | CPU + ядра                  | RAM, Гб | SSD, Tб |   HDD, Tб  |  количество     |
| ----------------- | ------------------| ------- | ------- | ----------- | -------------------------------|         
| Балансировщик + Front|Intel E5-2620 v4 – 8 ядер |   64    |    2 * 1 |     -    |     4  |
| backend            |Intel E5-2660 v4 – 14 ядер  |   64    |    1    |      1      |      35  |
| service(C++)  |    Intel E5-2620 v4 – 8 ядер    |   64    |    1    |      -      |       1 |
| PostgreSQL    |   Intel E5-2660 v4 – 14 ядер     |   512    |    215    |      300     |     4   |
| PostgreSQL комментов    |   Intel E5-2660 v4 – 14 ядер     |   512    |    300    |      -     |     15   |
| Redis        |    Intel E5-2620 v4 – 8 ядер     |   1536    |    1    |      -      |     2   |

2. *Балансировка*
    
В своем проекте будем использовать несколько видов балансировки:
* будем использовать `Round Robin` у DNS, тем самым будем равномерно распределять нагрузку
на наши вебсервера Nginx
* Балансировку на уровне Nginx, для этого будем использовать `L7` балансировку

3. *Хостинг*
    
    Для хостинга будем использовать сервера Yandex Cloud, так как они расположены в дата центрах Яндекса,
которые находятся в России, как раз под нашу целевую аудиторию. Будут удобно расположены с точки зрения топологии.
   Для хранения метаданных постов, то есть фотографий, видео или музыки будем использовать облачное хранилище MCS (Mail.ru Cloud Solutions).
   Оно отлично подойдет благодаря высокой степени надежности хранения. Самые просматриваемые метаданные постов можно будет хранить
   в MCS на виртуалке с SSD, для еще более
   быстрого доступа.

4. *Отказоустойчивость*
    
    Можно заметить что исходя из мощностей тех серверов, которые были выбраны при "выборе оборудования"
    у данного проекта, есть запас по мощностям. Но начинка железа это
   не главное. Чем достигается отказоустойчивость?
   * Наличие L7 балансировки, причем, мы настроим балансировку следующим образом:
        1. 5 бекендов будут в состоянии "запасного", мы будем включать их в балансировку,
           только когда какой-то из работающих перестанет отвечать на запросы.
        2. Когда будем проставлять бекендам весам, проставим 15 бекендам вес, равный 2,
            тем самым создадим себе двойной запас, если они отвалятся, мы перезапустим их с меньшим весом,
           и будем знать, что у нас искуственного двойного запаса больше нет, это отобразится на мониторинге.
   * Мы будем использовать на серверах `Prometheus + Grafana` для мониторинга и быстрого устранения проблем.
   * Сделаем у бэкендов отдельный микросервис для комментирования и в случае большой нагрузки будем выключать
    данный функционал
   * Для повышения надежности в использовании БД, будем использовать реплицирование, 
    так у нас в PostgreSQL без комментов будет схема **Master - Slave(3)**, в мастер будут писаться новые данные, а читаться
     они уже будут из slave. Так слейвами будут пользоваться не только бэкенды на чтение, но и сервис рекомендаций
     Аналогично PostgreSql комментариев будет иметь **Master - Slave(14)**
   * Аналогично будем использовать реплику в Redis, по схеме _Master-Slave_, в мастер будут писаться рекомендации
    а для получения рекомендаций пользователь будет ходить в реплику. Если вдруг пользователь решит посмотреть
     рекомендации 3 месячной давности тогда, нужные рекомендации будут подбираться с помощью PostgreSQL,
     благодаря кешированию и SSD, сильной просадки по времени быть не должно.





