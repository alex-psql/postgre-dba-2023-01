### Механизм блокировок
Выполнял на своей виртуальной машине Ubuntu 22.04, т.к. в приоритете на данный момент on-premise решение. Установлен кластер PG14.
___
1. Настройте сервер так, чтобы в журнал сообщений сбрасывалась информация о блокировках, удерживаемых более 200 миллисекунд. 
>В конфиг файле:  
>log_lock_waits = on  
>deadlock_timeout = 200ms

2. Воспроизведите ситуацию, при которой в журнале появятся такие сообщения.  
>Использую таблицу из предыдущих дз.  
<pre>
id | color | hexcode  
----+-------+---------  
  1 | black | 000000  
  2 | white | ffffff  
  3 | red   | FF0001  
</pre>  
>Сессия1:  
begin;  
UPDATE colorhexcodes SET hexcode = 'FF0000' WHERE color = 'red';  
UPDATE 1

>Сессия2:  
begin;  
UPDATE colorhexcodes SET hexcode = 'FF0002' WHERE color = 'red';  

Транзакция во второй сессии не подтверждает апдейт, т.к. заблокирована транзакцией из первой сессии. Соответствующая запись в логе:
>[339386] postgres@colors DETAIL:  Process holding the lock: 339345. Wait queue: 339386.  

3. Смоделируйте ситуацию обновления одной и той же строки тремя командами UPDATE в разных сеансах. Изучите возникшие блокировки в представлении pg_locks и убедитесь, что все они понятны. Пришлите список блокировок и объясните, что значит каждая.  
<pre>
  pid   |   locktype    |     lockid      |       mode       | granted
--------+---------------+-----------------+------------------+---------
 340838 | relation      | colorhexcodes   | RowExclusiveLock | t
 339345 | relation      | colorhexcodes   | RowExclusiveLock | t
 340532 | relation      | colorhexcodes   | RowExclusiveLock | t
 340838 | transactionid | 9460670         | ShareLock        | f
 339345 | transactionid | 9460670         | ExclusiveLock    | t
 340532 | transactionid | 9460672         | ExclusiveLock    | t
 340838 | tuple         | colorhexcodes:6 | ExclusiveLock    | t
 340838 | transactionid | 9460671         | ExclusiveLock    | t
 340532 | tuple         | colorhexcodes:6 | ExclusiveLock    | f
(9 rows)
</pre>

relation/RowExclusiveLock - блокировка строки в таблице
transactionid/ShareLock - следущая транзакция в очереди смотрит на tid того, кто заблокировал ресурс
transactionid/ExclusiveLock - транзакция никому не отдаст свой уникальный id
tuple/ExclusiveLock - блокировка, которая ждет, когда освободится желаемый ресурс

4. Воспроизведите взаимоблокировку трех транзакций. 

<pre>
 id |     cardnumber      | balance
----+---------------------+---------
  1 | 1234 1234 1234 1234 |  100.50
  2 | 2234 1234 1234 1234 |  200.50
  3 | 3234 1234 1234 1234 |  300.50
(3 rows)
</pre>
>Сессия1:  
UPDATE cards SET balance = balance - 50.00 WHERE cardnumber = '1234 1234 1234 1234';  
UPDATE 1  
Сессия2:  
UPDATE cards SET balance = balance - 30.00 WHERE cardnumber = '2234 1234 1234 1234';  
UPDATE 1  
Сессия1:  
UPDATE cards SET balance = balance + 50.00 WHERE cardnumber = '2234 1234 1234 1234';  
UPDATE 1  
Сессия2:  
UPDATE cards SET balance = balance + 30.00 WHERE cardnumber = '1234 1234 1234 1234';  
ERROR:  deadlock detected  
DETAIL:  Process 340838 waits for ShareLock on transaction 9460684; blocked by process 339345.  
Process 339345 waits for ShareLock on transaction 9460685; blocked by process 340838.  
HINT:  See server log for query details.  
CONTEXT:  while updating tuple (0,1) in relation "cards"  

Запись в логе:  
>[340838] postgres@colors LOG:  process 340838 detected deadlock while waiting for ShareLock on transaction 9460684 after 200.135 ms  
[340838] postgres@colors ERROR:  deadlock detected  

Если достать запись лога по pid, то можно распутать клубок при условии, что блокировки логируются и все попали в запись (подразумевается, что порог должен быть минимальным, иначе теряется информативность, т.к. в проде долгие блокировки это исключение, а не правило). Если логирование блокировок не включено, мы будем знать только о том, что взаимоблокировка произошла.

5. Могут ли две транзакции, выполняющие единственную команду UPDATE одной и той же таблицы (без where), заблокировать друг друга?  
Могут. В одной транзакции сделал апдейт в таблтице, увеличив баланс всех карт. Во второй запустил такой же апдейт. 
<pre>
  pid   |   locktype    | lockid  |     mode      | granted
--------+---------------+---------+---------------+---------
 340838 | transactionid | 9460687 | ExclusiveLock | t
 340838 | tuple         | cards:1 | ExclusiveLock | t
 340838 | transactionid | 9460686 | ShareLock     | f
 339345 | transactionid | 9460686 | ExclusiveLock | t
(4 rows)
</pre>
