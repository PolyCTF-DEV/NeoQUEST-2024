# Иерархия | PPC

## Информация
> Если по Данте кругов ада было 9, то для Вас их 101.

## Файл задания
[task.zip](task.zip)

## Описание
Надо перебирать все ссылки на странице, пока не найдётся нужная и будет осуществлён переход на следующую страницу. Предполагается автоматизация данного процесса.

## Решение
После регистрации и входа пользователю предоставляется 10 ссылок. При нажатии на каждую открывается капча(проверка на антиробот). При первом решении можно понять, что не все ссылки отправят нас на следующий шаг, а после тестов, становится очевидным, что есть только одна верная ссылка. Для решения задачи можно использовать перебор по всем ссылкам автоматизированным решением. Для решения капчи-примера есть два способа: использовать онлайн решение, например mathway можно довольно легко спарсить, или использовать локальные считыватели, например pytesseract.

```python
import requests
import base64
import time
import pytesseract
from PIL import Image

USER = "test2"
PASSWORD = "test"

ses = requests.Session() # Создание сессии, иначе не будет работать авторизация
req = ses.post("http://127.0.0.1:25555/login", data={"login": USER, "password": PASSWORD}) # Авторизация
while True:
	req = ses.get("http://127.0.0.1:25555/index") # Запрос страницы с ссылками
	data = req.text
	data = list(filter(lambda y: "openModal('" in y and 'back' not in y, map(lambda x: x.split(')">')[0], data.split('<a href="#" onclick="')))) # Начальный парсинг списка ссылок
	print(data) # Печать в основном для виденья процесса, что что-то идёт
	safe_data = data[:]
	image = ses.get(f"http://127.0.0.1:25555/static/captchas/{USER}.jpg").content # Загрузка капчи
	with open("captcha.jpg", "wb") as f:
    	f.write(image)
	img = Image.open("captcha.jpg")
	text = pytesseract.image_to_string(img) # Преобразование картинки в выражение
	expression = text.replace(" ", "").replace("\n", "")
	captcha = eval(expression) # Решение выражения
	i = 0
	while i < len(safe_data): # Перебор
    	req = ses.post("http://127.0.0.1:25555/captcha", data={"captcha": captcha, "link": safe_data[i].split("openModal('")[-1][:-1]}) # Отправка капчи на проверку и получение новой/старой страницы
    	data = req.text
    	image = ses.get(f"http://127.0.0.1:25555/static/captchas/{USER}.jpg").content # Загрузка капчи
    	with open("captcha.jpg", "wb") as f:
        	f.write(image)
    	img = Image.open("captcha.jpg")
    	text = pytesseract.image_to_string(img) # Преобразование картинки в выражение
    	expression = text.replace(" ", "").replace("\n", "")
    	captcha = eval(expression) # Решение выражения
    	i += 1 # Переменная перебора
    	data = list(filter(lambda y: "openModal('" in y and 'back' not in y, map(lambda x: x.split(')">')[0], data.split('<a href="#" onclick="')))) # Начальный парсинг списка ссылок
    	if safe_data != data: # Если список ссылок поменялся
        	break
	if not data: # Если список отсутствует - закончились ссылки - вывод флага
    	break
req = ses.get("http://127.0.0.1:25555/index") # Запрос страницы с флагом
data = req.text
print(data.split("<h2>")[1].split("</h2>")[0]) # Вывод флага
ses.close()
```

## Флаг
`NQ202437621cddbd096f10e2c7b41eb3e07ad11773fe85c6f1973477e57fbcd1d2c1ca`
