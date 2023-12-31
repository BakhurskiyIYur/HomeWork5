# Домашнее задание №5 (Тема "MVCC, vacuum и autovacuum.")

Описание/Пошаговая инструкция выполнения домашнего задания:
 
* 1 cоздать инстанс ВМ с 2 ядрами и 4 Гб ОЗУ и SSD 10GB
* 2 Установить на него PostgreSQL 15 с дефолтными настройками
  > <img src="pic/2.JPG" align="center" />
* 3 Создать БД для тестов: выполнить pgbench -i postgres
  > <img src="pic/3.JPG" align="center" />
* 4 Запустить pgbench -c8 -P 6 -T 60 -U postgres postgres
<br>__*-c8 (Количество моделируемых клиентов, т. е. количество одновременных сеансов базы данных)*__
<br>__*-P 6 (Отображает отчет о ходе выполнения каждые N секунд)*__
<br>__*-T 60 (Выполняется тест в течение этого количества секунд, а не фиксированного количества транзакций на клиента)*__
<br>__*-U postgres postgres (Имя пользователя, к которому необходимо подключиться)*__
  > <img src="pic/4.JPG" align="center" />
* 5 Применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла
<br>__*Для настройки сервера использовал файл — postgresql.auto.conf.*__
  > <img src="pic/5_2.jpg" align="center" />
<br>__*После задания параметров сервер был перезапущен, чтобы применилась новая конфигурация.*__  
  > <img src="pic/5_1.JPG" align="center" />
* 6 Протестировать заново (pgbench -c8 -P 6 -T 60 -U postgres postgres)
  > <img src="pic/6.JPG" align="center" />
* 7 Что изменилось и почему?
  <br>__*pgbench - это программа для запуска тестов производительности на PostgreSQL. Она выполняет одну и ту же последовательность команд SQL снова и снова, а затем вычисляет среднюю скорость транзакций (транзакций в секунду). 
Задав по-новому значения параметров настройки PostgreSQL, сервер был переконфигурирован. В результате TPS (средняя скорость транзакций?) снизилась с  361 до 305, количество обработанных транзакций за время теста тоже снизилось – с 21662 до 18351.*__ 

  <br>__*Параметр «shared_buffer» - буфер PostgreSQL, этот параметр устанавливает, сколько выделенной памяти будет использоваться PostgreSQL для кеширования. До изменения «shared_buffer» был 128МБ, увеличили до 1ГБ.*__ 
  <br>__*Значение для «min_wal_size» ограничивает снизу число файлов WAL, которые будут переработаны для будущего использования. До изменения min_wal_size был 80МБ, увеличили до 4ГБ. «max_wal_size» увеличили с 1ГБ до 16ГБ.*__ 
  <br>__*Параметр «effective_cache_size» оценивает объем памяти, доступной для кэширования диска операционной системой и самой базой данных.*__ 
  <br>__*Параметр «work_mem» - задаёт базовый максимальный объём памяти, который будет использоваться во внутренних операциях при обработке запросов (например, для сортировки или хеш-таблиц), прежде чем будут задействованы временные файлы на диске.*__ 
* 8 Создать таблицу с текстовым полем и заполнить случайными или сгенерированными данным в размере 1млн строк
  > <img src="pic/8.JPG" align="center" />
* 9 Посмотреть размер файла с таблицей
  > <img src="pic/9.JPG" align="center" />
* 10 Обновить 5 раз все строчки и добавить к каждой строчке любой символ
  > <img src="pic/10.JPG" align="center" />
* 11 Посмотреть количество мертвых строчек в таблице и когда последний раз приходил автовакуум 
  > <img src="pic/11.JPG" align="center" />
* 12 Подождать некоторое время, проверяя, пришел ли автовакуум
  > <img src="pic/12.JPG" align="center" />
* 13 Обновить 5 раз все строчки и добавить к каждой строчке любой символ
  > <img src="pic/13.JPG" align="center" />
* 14 Посмотреть размер файла с таблицей
  > <img src="pic/14.JPG" align="center" />
* 15 Отключить Автовакуум на конкретной таблице
  > <img src="pic/15.JPG" align="center" />
* 16 Обновить 10 раз все строчки и добавить к каждой строчке любой символ
  > <img src="pic/16.JPG" align="center" />
* 17 Посмотреть размер файла с таблицей
  > <img src="pic/17.JPG" align="center" />
* 18 Объясните полученный результат
  <br>__*Таблица test сильно выросла в размерах. В процессе обновления 1млн. записей растет число «мертвых», то есть неактуальных, версий строк. Число мертвых версий постоянно собирается коллектором.   За последние 10 обновлений число таких строк увеличилось на 10млн., по 1млн. строк за каждый UPDATE. Все эти данные также хранятся в таблице, хоть мы их и не видим SELECTом.  Считается, что очистка необходима, если число «мертвых» версий строк превышает установленное пороговое значение. Так как мы отключили процесс Автовакуум для этой таблицы, то  мертвые версии строк не очищались.*__
  > <img src="pic/18.JPG" align="center" />
* 19 Не забудьте включить автовакуум)
  <br>__*Включил для таблицы test автовакуум, подождав немного убедился, что автовакуум прошел.*__
  > <img src="pic/19_3.JPG" align="center" />

# Задание со *:
Написать анонимную процедуру, в которой в цикле 10 раз обновятся все строчки в искомой таблице. 
<br>Не забыть вывести номер шага цикла.
  > <img src="pic/exp.JPG" align="center" />
