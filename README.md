# Тема проекта -  Vkontakte
## *MVP*  
 NewsFeed - приложение, в котором можно подписываться на других людей, смотреть их посты, и выкладывать свои.
### Целевая аудитория:
* [77% от всей мобильной аудитории в России – первое место.](https://vk.com/@cerebro_vk-polzovateli-socsetei-v-rossii-statistika-i-portrety-auditori)
* 73 миллионов активных пользователей в месяц.
* [42 млн человек в день.](https://ppc.world/articles/auditoriya-shesti-krupneyshih-socsetey-v-rossii-v-2020-godu-izuchaem-insayty/)
* Россиская Федерация
### Рассчет нагрузки
1) #### *Рассчет нагрузки на создание постов:*
    По данным статистики за [май 2017](https://vk.com/@cerebro_vk-polzovateli-socsetei-v-rossii-statistika-i-portrety-auditori):
    - 25 721 668 - число авторов, сделавших пост за месяц
    - 310 795 150 - число постов в месяц 
    
    Зная число постов в месяц, можем посчитать сколько постов в среднем делается за один день:

       310 795 150 / 31 = 10 025 650 постов/день


Теперь посчитаем число запросов в секунду, мы знаем что наибольший пик активности в Вконтакте приходится на обед с 11 до 13,
и вечером с 17 до 23 => Итого: [8 часов в день](https://postium.ru/luchshee-vremya-dlya-publikacii-postov-v-instagram-vk/)

Тогда для получения числа постов в секунду:


    10 025 650 / (8 * 3600) ~ 348 запросов/c


2) #### *Рассчет нагрузки на создание ленты:*
    Пусть каждый пользователь делает по 7 запросов в день на создание ленты, зная число пользователей
в среднем в день (42 млн) и наибольший пик активности равный 8 часам, получим:
   

     42 * 10^6 * 7 / (8 * 3600) = 10 208 запросов/c


### Итоги по РПС сервиса:
*  Создание постов ~ 350 запросов / секунду (на запись)
* Составление ленты ~ 10 000 запросов / секунду (на чтение)
