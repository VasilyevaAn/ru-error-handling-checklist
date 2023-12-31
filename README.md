# Чек-лист "Безопасность обработки ошибок"

## Список проблем безопасности связанных с ошибками

* Раскрывают информацию об объекте. Из текста ошибки можно узнать на каком ЯП, фреймворке написанно приложение, иногда версию используемой библиотеки. 
* Раскрывают информацию об инфраструктуре. Stacktrace содержит секреты: connection strings, токены. "Ошибка" от фреймворка так же может возвращать секреты.
* Раскрывают персональные данные
* Уязвимости в валидации. Текст ошибки вида "value = 111 invalid" - поможет эксплуатировать узявимость в таком случае.
* Иногда метод срабатывает, но выводит текст ошибки

## Чек-лист для DEV
* Не выводить StackTrace в тексте ошибки на проде
* Если выводите StackTrace только на тестовых средах - проверяете, что что конфигурация правильная и не выводятся на продакшен средах.
* Храните StackTrace в системе логирования
* Не выводите в текст ошибки информацию, которая не должна попадать в систему логирования.
Например, заваемый пароль пользователем
* Не отдавайте в текст ошибки информацию об обьекте, если у пользователя нет к нему прав
* Если метод парсит XML и текст ошибки возвращает результат обработки парсером, то при наличии XXE уязвимости можно получить в тексте ошибки локальный файл сервера, доступный под пользователем под которым запущенно приложение.

## Чек-лист по текстам ошибок для QA
* Проверьте как приложение обрабатывает не валидное значение. Например если метод POST/PATCH/PUT, то пошлите метод с пустым телом сообщения. Если метод GET/DELETE передайте u000 в параметре
* Если в методе есть идентификаторы, попробуйте выполнить его с ID другого пользователя. Если метод выполнится - вы нашли IDOR уязвимость. Если нет, но вернет не доступную публично информацию об обьекти в тексте ошибки, то это проблема расскрытие персональных данных.
* Если есть интеграция, то проверяйте разные варианты: оборвал соединение, не смогли достучаться, вернул не предсказуемое тело ответа
* Если метод обрабатывает XML и метод возвращает "value = 111 invalid".

Меняем исходный XML по следующему правилу
После 
```
<?xml version="1.0">
```
Добавляем
```
<!DOCTYPE replace [<!ENTITY ent SYSTEM "file:///etc/passwd"> ]>
```
И заменяем любое поле <>test</> на  
```
<>&xxe;</>
```
* Отправляем измененный документ
Если вернется содержимое файла, то это XXE

## Дополнительные матерьялы
* Статья [Не все "ошибки" одинаково полезны - с точки зрения безопасности.](https://teletype.in/@8ug8eer/error_handler_security_problem)
