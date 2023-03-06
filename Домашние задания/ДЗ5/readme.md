### MVCC, vacuum и autovacuum.
Выполнял на своей виртуальной машине Ubuntu 22.04, т.к. в приоритете на данный момент on-premise решение.
___
1. Установить PostgreSQL 14 и применить настройки из ДЗ
>Внести настройки в конфиг файл и применить

>max_connections = 40  
shared_buffers = 1GB  
effective_cache_size = 3GB  
maintenance_work_mem = 512MB  
checkpoint_completion_target = 0.9  
wal_buffers = 16MB  
default_statistics_target = 500  
random_page_cost = 4  
effective_io_concurrency = 2  
work_mem = 6553kB  
min_wal_size = 4GB  
max_wal_size = 16GB  

2. Подготовка и выполнение pgbench
>pgbench -i postgres

3. Первая итерация - настройки из ДЗ.

progress: 60.0 s, 796.3 tps, lat 10.038 ms stddev 9.792
progress: 120.0 s, 535.7 tps, lat 14.932 ms stddev 20.648
progress: 180.0 s, 784.7 tps, lat 10.192 ms stddev 10.602
progress: 240.0 s, 792.9 tps, lat 10.089 ms stddev 10.938
progress: 300.0 s, 779.4 tps, lat 10.261 ms stddev 10.236
progress: 360.0 s, 805.2 tps, lat 9.932 ms stddev 11.567
progress: 420.0 s, 578.2 tps, lat 13.828 ms stddev 18.259
progress: 480.0 s, 796.8 tps, lat 10.041 ms stddev 11.857
progress: 540.0 s, 801.5 tps, lat 9.979 ms stddev 10.374
progress: 600.0 s, 797.2 tps, lat 10.028 ms stddev 11.211

4. Вторая и далее...
А  дальше, сколько бы я ни менял настройки, картина остается плюс минус такой же. Первым делом воспроизвел настройки из лекции "рекоемндуемые". 790-830 tps с провалами на некоторых шагах. Пробовал устанавливать крайние значения для всех настроек, связанных с вауумом. Так же сбрасывал в минимум настройки по памяти для рабочего процесса и игрался с io concurrency, но тпс по-прежнему оставался в пределах 790-830 с провалами до 500. От последних не удалось избавиться никак. 

progress: 60.0 s, 819.4 tps, lat 9.755 ms stddev 8.890
progress: 120.0 s, 825.8 tps, lat 9.684 ms stddev 8.590
progress: 180.0 s, 824.3 tps, lat 9.705 ms stddev 8.729
progress: 240.0 s, 566.1 tps, lat 14.129 ms stddev 17.582
progress: 300.0 s, 800.9 tps, lat 9.987 ms stddev 9.131
progress: 360.0 s, 813.6 tps, lat 9.829 ms stddev 9.593
progress: 420.0 s, 785.4 tps, lat 10.185 ms stddev 10.196
progress: 480.0 s, 799.9 tps, lat 9.999 ms stddev 10.633
progress: 540.0 s, 568.8 tps, lat 14.061 ms stddev 18.231
progress: 600.0 s, 820.4 tps, lat 9.749 ms stddev 8.724

Единственный вывод, который я смог сделать для себя: моё узкое место - диск. Теперь знаю, где копаться и как менять, но красивого графика не вышло. 
