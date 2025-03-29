# Применение профайлера pprof

1. Создал активную нагрузку (бесконечные запросы на эндпоинт /api/address/search).

Собрал cpu.pprof за 10 секунд:
```bash
newadik@newad-pc:~/projects/task4.1.3$ curl -o cpu.pprof \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJsb2dpbiI6ImFkIn0.RKCOH5cZiA6xgdss25jDklLuGa-kG3Cw3vDAX_YONd4" \
  "http://localhost:8080/mycustompath/pprof/profile?seconds=10"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1807  100  1807    0     0    180      0  0:00:10  0:00:10 --:--:--   476
```
Собрал trace.out за 5 секунд:
```bash
newadik@newad-pc:~/projects/task4.1.3$ curl -o trace.out \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJsb2dpbiI6ImFkIn0.RKCOH5cZiA6xgdss25jDklLuGa-kG3Cw3vDAX_YONd4" \
  "http://localhost:8080/mycustompath/pprof/trace?seconds=5"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 84839    0 84839    0     0  16961      0 --:--:--  0:00:05 --:--:-- 16753
```

2. После создания файлов cpu.pprof и trace.out с помощью команд:

```bash
pprof -http=:8081 cpu.pprof
```
и
```bash
go tool trace trace.out
```

Ознакомился с их содержимым:

- cpu.pprof:
* Основное время тратится на функции http.(*conn).serve и handlers.SearchHandler.
* Существуют затраты на json.Unmarshal - чтобы отдать ответ от DaData.
* Насколько я понял, вроде всё ок.
- trace.out:
* Ошибок не увидел, все горутины корректно завершаются.
* Самые длинные задержки - это poll.(*FD).Accept и poll.(*FD).Read - нашел информацию, что это нормальное поведение сервера.
* Насколько я понял, вроде всё ок.

3. Понял, что можно добавить кэширование ответов от DaData (этим и займусь в задаче 3.2.3, которую не приняли из-за того, что нужно было применить паттерн прокси именно к этому геосервису. Как раз увижу разницу в показателях :) ).
