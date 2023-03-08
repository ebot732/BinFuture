# ebotFuture
E-Bot Future.
Бот для торговли на Binance Futures USDⓈ-M с использованием стратегии мартингейла (усреднения) и выбором растущей (падающей) монеты. Может работать и в LONG и в SHORT.

E-Bot выбирает монету, выросшую за указанный в настройках период времени на указанный процент (при работе в  LONG, а при работе в SHORT упавшую), 
- покупает ее (встает в LONG) маркет-ордером (старт-ордер) на указанный объем (при работе в SHORT продает), 
- выставляет купленные монеты на продажу (для закрытия позиции в плюс) лимитным sell-ордером (FIX-ордером) по курсу на указанный процент прибыли выше курса покупки (в SHORT-е на покупку лимтным buy-ордером) ,
- выставляет лимитный buy-ордер (усред-ордер) на покупку этой же монеты по курсу ниже предыдущей покупки на указанный процент (на случай падения курса монеты и уменьшения средней цены входа в сделку)(в SHORT-е лимитный SELL-ордер на случай повышения курса).

Затем, в зависимости от того, какой ордер исполнился (примеры для LONG, для SHORT- наоборот):
- если исполнился sell-ордер (FIX-ордер), бот фиксирует прибыль и отменяет buy-ордер (усред-ордер) для усреднения(если buy-ордер при этом успел исполниться частично или полностью- выставляется sell-ордер), затем снова ищет подходящую пару,
- если исполнился buy-ордер (усред-ордер), бот отменяет sell-ордер (FIX-ордер),выставляет новый sell-ордер (FIX-ордер) уже с новым количеством монет и по новой цене(средняя цена входа + указанный процент прибыли), выставляет новый buy-ордер (усред-ордер),
- если buy-ордер (усред-ордер) исполнился больше, чем наполовину и прошло более 5-ти минут после этого , бот отменяет sell-ордер (FIX-ордер), и выставляет новый sell-ордер (FIX-ордер) уже с новым количеством монет и по новой цене(средняя цена входа + указанный процент прибыли).

Рекомендуется E-Bot установить на VPS сервер ubuntu 20 и запускать в SCREEN (чтобы бот не отключался при разрыве SSH-соединения с VPS), настроить telegram-бот и канал, куда будет приходит информация о работе бота.

При необходимости E-Bot можно использовать и на Windows, для этого нужно скачать в отдельную папку версию для Windows (https://github.com/ebot732/ebotFuture/releases/download/ebotFuture-9.0/ebotFuture-9.0.exe), и запустить.

На Ubuntu:
- Запуск бота командой:         ./ebotFuture-9.0  
- Остановка бота командой:      ctrl+c    (важно!: не останавливайте бот в момент совершения сделок, возможна ошибка записи в базу данных бота).
- В white_list (список пар для работы) можно внести от 1 до нескольких сотен пар, главное, чтобы квотируемая валюта была одна: если торгуете к USDT, то пары ETHUSDT, BTCUSDT, XRPUSDT и т.д., если торгуете к BUSD, то пары ETHBUSD, XRPBUSD, DASHBUSD и т.д.
- При изменении квотируемой валюты не забывайте проверять и min_order в настройках бота (в USDT min_order должен быть больше 11)
- При работе с парами к USDT активы должны находится на фьючерсном балансе USDT, если работаете с парами к BUSD, активы должны находится на фьючерсном балансе  BUSD.
- Для прокрутки экрана терминала вверх есть команда: ctrl+a, esc и далее стрелка вверх. Для выхода из этого режима: esc, esc.

Для работы E-Bota можно использовать BNB для оплаты комиссий биржи (нужно перевести нужное количество BNB на фьючерсный счет в лк binance) и следить за наличием BNB на Futures аккаунте.

После закрытия каждой сделки E-Bot:
- отправляет сообщение в telegram-канал, 
- каждую минуту в описание канала отправляет информацию об открытой позиции, 
- в полночь в канал отправляет суточный отчёт о работе. Если не было прибыли за сутки, а также если бот находится в режиме поиска подходящей монеты, то суточный отчёт в telegram не придёт, а в следующий отчёт будет прибавлена предыдущая прибыль за сутки. Точные данные по прибыли наблюдать лучше в лк binance, так как бот показывает приблизительные значения.

Настройки бота (в основном описано для LONG, для SHORT применяется наоборот):
- fix_perc: процент повышения цены для продажи при LONG или падения для SHORT,
- step_aver: ввод step_aver1, step_aver2, ... step_aver7 шагов изменения цены для выставления усредов,
- qty_aver: кратное увеличение объема выставляемого ордера,
- min_order: минимальная покупка (продажа) в квотируемой валюте (например, в паре ETH/USDT это USDT, ставить больше 11 и учитывайте, что
в зависимости от выбранного leverage (кредитного плеча) будет использоваться меньше USDT, например: если min_order указан 20, а leverage указан 10, то для ордера будет использовано 20/10=2 USDT),
- min_bal_perc: минимальный процент от депозита, ниже которого E-Bot не будет выставлять усредняющий ордер,
- delta_start: на сколько % должен подняться курс (упасть, при SHORT указываем с минусом, например -1.7%) от цены открытия выбранной свечи до текущей цены для старта,
- stop_loss: на сколько % должен измениться курс монеты от средней цены входа для закрытия в минус,
- used_stop_loss: включить использование stop_loss для закрытия в минус (да/нет),
- pause_after_stop_loss: ставить E-Bot на паузу после срабатывания stop_loss и закрытия позиции по рынку или продолжить работу,
- completed: поставить бота на паузу при закрытии очередной сделки (1-вкл/0-выкл),
- kline_interval: интервал свечей для анализа (1m, 3m, 5m, 15m, 30m, 1h, 2h, 4h, 8h, 12h, 1d, 3d),
- interval_limit: какое количество свечей анализируем,
- super_asset: пара для бесконечной торговли независимо от delta_start, при этом пары из white_list не будут работать (вводится командой -super_asset_add в формате ETHUSDT),
- manual_aver: команда для ручного усреднения по рынку, не дожидаясь цены лимитного усред-ордера, но не сработает, если усред-ордер, выставленный ботом, исполнен частично ('PARTIALLY_FILLED'),
- fix_loss: команда для закрытия открытой позиции по рынку, 
- clear: сброс из базы данных сведений об открытых ботом ордерах,
- leverage: размер кредитного плеча от 1 до 120 (оно разное для разных пар, смотрите в лк биржи),
- marginType: ISOLATED или CROSSED
- direction: LONG или SHORT (менять направление работы LONG/SHORT строго рекомендуется в терминале и при отсутствии открытых позиций)
- clear_profit: сброс из базы данных сведений о прибыли,
- t_sleep: при получении от биржи ошибки о превышении лимита api-запросов, можно выбрать значение паузы в секундах (например: 0.5, 1, 3),
- white_list: список пар, которые бот будет использовать для анализа и выбора подходящей для открытия сделки (вводится по одной паре командой -w_list_add),
- api_key: открытый api-ключ от биржи с разрешением на фьючерсную торговлю,
- api_secret: секретный api-ключ от биржи,
- botID: api телеграм бота полученный от @BotFather (пример: 5656544920:AAHrXhjhujhfdf7RPJlheqJXEulBW),
- channelID: ID канала telegram бота для уведомлений, полученное от @userinfobot (пример: -1001656543985),
- tguserid: ID основного user-a телеграм, полученное от @userinfobot (пример: 346549043)

При изменении marginType, direction, leverage убедитесь, что нет открытых позиций

Здесь E-Bot-Future представлен для ознакомления и использования в течении пробного периода до 15 мая 2023 г.
Если Вы хотите увеличить время работы до 1/6/12 месяцев: напишите в телеграм, по данным, указанным при запуске E-Bot.

Если хотите испытать E-Bot на фьючерсной тестовой бирже Binance- 
переходите на:
https://testnet.binancefuture.com/ru/futures/
Где получите тестовые api-ключи, пропишите их в настройках бота, в used_testnet запишите: да, и экспериментируйте.

E-Bot поставляется по принципу «как есть». Никаких гарантий не прилагается и  не  предусматривается. Вы берете на себя весь риск относительно использования этого бота и должны понимать, что торговля на криптобиржах сопряжена с повышенным риском, и подходить к управлению рисками со всей ответственностью. 

Пояснения по установке, запуску, настройке бота и телеграм, screen, ошибке на  VPS utf-8.

Иногда бот может получить от биржи неправильные ответы на api-запросы и выдавать ошибку, поэтому рекомендуется периодически заглядывать в лк binance, и, если бот показывает открытые ордера а в лк binance их нет (или наоборот), нужно использовать команду -clear, чтобы сбросить в боте данные о неактуальных ордерах.
Редко, но бывает, что сервера telegram кратковременно недоступны, и в этот момент сообщение от E-Bot может не доходить в канал бота. 
Для управления ботом на VPS сервере с телефона можно использовать приложение JuiceSSH (или другое для SSH-соединения).
Если возникла ошибка 'code: -4061' «Order’s position side does not match user’s setting», значит биржа не дает открыть позицию, так как у Вас установлен хедж вариант торговли «Hedge Mode». Боту нужен односторонний режим "One-way".

Установка и запуск E-Bot:
- на VPS-сервере ubuntu 20 создайте новую папку, например, ebotFuture (mkdir ebotFuture)
- зайдите в эту папку (cd ebotFuture)
- перенесите в эту папку файл бота ebotFuture-9.0 (или скачайте с github командой: wget https://github.com/ebot732/ebotFuture/releases/download/ebotFuture-9.0/ebotFuture-9.0)
- откройте screen-сессию (например: screen -S ebotFuture)
- дайте права запуска файлу (команда: chmod 755 ebotFuture-9.0)
- запустите E-Bot (команда: ./ebotFuture-9.0)
- команда для остановки бота: ctrl+c
- после запуска бота введите свои параметры: api_key и т.д.
- откорректируйте, при необходимости, настройки
- жмите ENTER и наблюдайте
- для выхода из SCREEN перед закрытием SSH-сессии используйте команду ctrl+a, d 
- для входа в screen работающего бота используйте команду: screen -x ebotFuture

Для удобства настройки E-Bot можно использовать телеграм бота, которого нужно сделать админом в телеграм канале. Необходимые данные телеграм бот возьмет из БД E-Bot и будет управляться через чат telegram-Botа.

Табличка Future_усреды_Ebot.xls показывает приблизительные расчёты усреднений и цены ликвидации, точные данные смотрите в лк binance.


             Скриншоты

![Screenshot](https://github.com/ebot732/ebotFuture/blob/main/screenshots/Screenshot_20221124-185022_Telegram.jpg)

====================================================================================================================================

![Screenshot](https://github.com/ebot732/ebotFuture/blob/main/screenshots/Screenshot_20230308-120835.png)

====================================================================================================================================

![Screenshot](https://github.com/ebot732/ebotFuture/blob/main/screenshots/Screenshot_20230308-123701.png)

====================================================================================================================================

![Screenshot](https://github.com/ebot732/ebotFuture/blob/main/screenshots/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%20%D0%BE%D1%82%202023-03-05%2008-16-08.png)

====================================================================================================================================

![Screenshot](https://github.com/ebot732/ebotFuture/blob/main/screenshots/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%20%D0%BE%D1%82%202023-03-08%2012-13-27.png)

====================================================================================================================================


