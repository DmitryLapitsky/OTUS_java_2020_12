## Оценка сборщиков мусора
-XX:+UseG1GC
-XX:+UseParallelGC
-XX:+UseSerialGC
'-XX:+UseConcMarkSweepGC
Для проведения оценок был составлен стенд, приводящий к out of memory exception (oom). Для базового теста использовался сборщик мусора (gc) -XX:+UseG1GC, для которого были определены константы для получения oom за примерно 5 минут. Использовался подход, когда в 1-м цикле запускался 2-й цикл с увеличивающимся количеством объектов в каждой итерации (количество создаваемых объектов отслеживалось). Половина объектов после инициализации (но без ссылки на объект) "уничтожается" - заменяется значением null.
В процессе подсчитывается колчиство сборщиком young и old generation, а так же суммируется по ним время отдельно для каждого типа.

##проверка времени на сборки за минуту
для сборщика мусора -XX:+UseG1GC


#####хип 256 мб
|Test time [s]|Amount of young gen.|young total time [ms]|Amount of old gen.|old total time [ms]|gc total time [s]|gc builds time per minute|
| ------------- | ------------- |------------- |------------- |------------- |------------- |------------- |
|271|2707|142445|267|21944|164.389|36.39608856|
|246|2593|135062|271|20970|156.032|38.05658537|
|276|2960|146408|329|33771|180.179|39.16934783|
|537|2816|251639|324|45796|297.435|33.23296089|
|398|2835|198865|311|31026|229.891|34.65693467|
|305|3316|154193|380|41184|195.377|38.43481967|
|306|2886|172287|324|29267|201.554|39.52039216|
|315|3377|159677|398|46663|206.34|39.30285714|
|260|2728|140366|285|24413|164.779|38.02592308|
|282|2938|146885|339|34572|181.457|38.60787234|
| | | | | |Average|37.54037817|

#####хип 384 мб
|Test time [s]|Amount of young gen.|young total time [ms]|Amount of old gen.|old total time [ms]|gc total time [s]|gc builds time per minute|
| ------------- | ------------- |------------- |------------- |------------- |------------- |------------- |
|670|4944|336774|584|100482|437.256|39.15725373|
|658|4757|325094|552|90966|416.06|37.93860182|
|713|5187|351287|645|118851|470.138|39.56280505|
| | | | | |Average|38.8862202|

#####хип 512 мб
|Test time [s]|Amount of young gen.|young total time [ms]|Amount of old gen.|old total time [ms]|gc total time [s]|gc builds time per in minute|
| ------------- | ------------- |------------- |------------- |------------- |------------- |------------- |
|1106|6673|605907|656|75551|681.458|36.96878843|
|1125|6652|599889|634|70910|670.799|35.77594667|
|1091|6666|600654|644|73373|74.027|37.06839597|
| | | | | |Average|36.60437702|

Среднее количество сборок gc за минуту примерно 37,6 с $$\sigma$$=0.94
oom для 1024 получить не удалось с теми же параметрами. А изменять их - наружшить постоянство среды.


##сравнение gc
т.к. различные gc используют свои термины, то рассматривалось количество объектов, созданных перед падением (количество объектов /15000).
######-XX:+UseSerialGC

|Test time [s]|number of objects before fail / 15000| gc total time [s] | gs builds|objects per sec/15000 [1/s]|gc builds time per minute|
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- |
|553|427350|451.063|5095|772.7848101|48.93992767|
|565|427350|463.652|5095|756.3716814|49.23738053|
|557|427350|455.863|5095|767.2351885|49.10552962|
| | | |Average|765.4638934|49.09427927|

######-XX:+UseParallelGC

|Test time [s]|number of objects before fail / 15000| gc total time [s] | gs builds|objects per sec/15000 [1/s]|gc builds time per minute|
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- |
|309|190036|261.569|3355|615.0032362|50.79009709|
|309|190036|263.760|3369|615.0032362|51.21553398|
|314|190036|267.894|3365|605.2101911|51.18993631|
| | | |Average|611.7388879|51.06518912|


######-XX:+UseG1GC

|Test time [s]|number of objects before fail / 15000| gc total time [s] | gs builds|objects per sec/15000 [1/s]|gc builds time per minute|
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- |
|269|199396|162.321|3040|741.2490706|36.20542751|
|324|208335|204.267|3174|643.0092593|37.82722222|
|319|199396|183.612|3090|625.0658307|34.53517241|
| | | |Average|669.7747202|36.18927405|

######-XX:+UseConcMarkSweepGC

|Test time [s]|number of objects before fail / 15000| gc total time [s] | gs builds|objects per sec/15000 [1/s] |gc builds time per minute|
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- |
|850|427350|708.818|5126|502.7647059|50.03421176|
|859|427350|717.843|5122|497.4970896|50.14037253|
|814|427350|668.247|5137|525|49.25653563|
| | | |Average|508.4205985|49.81037331|

итоговая таблица

|GC|objects per sec |gc builds time per minute|builded objects before oom|
| ------------- | ------------- | ------------- | ------------- | 
|Serial|765.4638934|49.09427927|427350|
|Parallel|611.7388879|51.06518912|190036|
|CMS|508.4205985|49.81037331|427350|
|G1|669.7747202|36.18927405|202375|

Из таблицы следует, что если не важна проблема stf (stop the world), то serial выигрывает как по скорости (количетсво созданных объектов до ошибки oom/общее время прогона теста), так и по времени, требуемому для сбора мусора.
Если проблема stf имеет первостепенное значение, то лучший из рассмотренных - G1.
Удивительно одинаковые результаты по количеству созданных перед oom объектов показали serial и cms gc.