# Docker Network

Как связать 2 контейнера в одной сети для их взаимодействия

Код проекта будет выложен блоками для лучшего понимания и представления всего проекта (обучаещего эксперимента)

## Создание образов (Image)
В данном примере необходимо 3 файла:
1. App.py
2. requirements.txt (необходимые библиотеки для *Python3*)
3. Dockerfile (для создания *Docker Image*)

## ping-service

*app.py*

```
bin/python3

from flask import Flask
app = Flask(__name__)

import requests

@app.route('/')
def ping_service():
    return 'Hello, I am ping service!'

@app.route('/ping')
def do_ping():
    ping = 'Ping ...'

    response = ''
    try:
        response = requests.get('http://pong-container:5001/pong')
    except requests.exceptions.RequestException as e:
        print('\n Cannot reach the pong service.')
        return 'Ping ...\n'

    return 'Ping ... ' + response.text + ' \n'

if __name__ == "__main__":
    app.run(host ='0.0.0.0', port = 5000, debug = True)

```
*requirements.txt*
```
Flask==3.0.2
requests==2.32.3
```
*Dockerfile*
```
FROM python                             # используем за основу уже готовый образ (DEB-linux + Python3.13)
WORKDIR /app                            # создаем рабочую папку в корневойм каталоге app
COPY . /app                             # копируем всё содержимое проекта в папку /app
RUN pip install -r requirements.txt     # устанавливаем все необходимы библиотеки для python3
ENTRYPOINT [ "python3" ]                # будем использовать среду Python3
CMD [ "app.py" ]                        # команда в среде Python3 (запуск приложения app.py)
```

## pong-service

*app.py*

```
bin/python3

from flask import Flask
app = Flask(__name__)

@app.route('/')
def pong_service():
    return 'Hello, I am pong service!'

@app.route('/pong')
def do_pong():
    return 'Pong'

if __name__ == "__main__":
    app.run(host ='0.0.0.0', port = 5001, debug = True)

```
*requirements.txt*
```
Flask==3.0.2
requests==2.32.3
```
*Dockerfile*
```
FROM python                             # используем за основу уже готовый образ (DEB-linux + Python3.13)
WORKDIR /app                            # создаем рабочую папку в корневойм каталоге app
COPY . /app                             # копируем всё содержимое проекта в папку /app
RUN pip install -r requirements.txt     # устанавливаем все необходимы библиотеки для python3
ENTRYPOINT [ "python3" ]                # будем использовать среду Python3
CMD [ "app.py" ]                        # команда в среде Python3 (запуск приложения app.py)
```









