# Docker Network
Owner: Dismont
Date: 20.10.2024

Как связать 2 контейнера в одной сети для их взаимодействия

Код проекта будет выложен блоками для лучшего понимания и представления всего проекта (обучаещего эксперимента)

## Создание образов (Image)
В данном примере необходимо 3 файла:
1. App.py
2. requirements.txt (необходимые библиотеки для *Python3*)
3. Dockerfile (для создания *Docker Image*)

## ping-service

Устройство нашего проекта в папке
```
./ping-pong-service/
        /ping-service/
            /app.py
            /requirements.txt
            /Dockerfile
```

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

Устройство нашего проекта в папке
```
./ping-pong-service/
        /pong-service/
            /app.py
            /requirements.txt
            /Dockerfile
```

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
Тепереь имея некоторые заготовки для наших будующих образов мы должны их создать и в последствии уж создать на их основе полноценные контейнеры
```
cd ping-service
docker build -t ping-service .

cd ..

cd pong-service
docker build -t pong-service .
```
## Организация общей сети в Docker

Теперь нам необходимо создать контейнеры на основе наших уже созданных образов, открыв соответсвующие порты *ping = 5000*, *pong = 5001*

```
docker run -it --name ping-container -p 5000:5000 ping-service
docker run -it --name pong-container -p 5001:5001 pong-service
```
Для того чтобы контейнеры могли найти друг друга в сети нужно создать для них сеть и поместить их в неё

```
docker network create ping-pong-network                             # создаем новую сеть ping-pong-network
docker network connect ping-pong-network ping-service-container     # поключаем к сети контейнер ping-container
docker network connect ping-pong-network pong-service-container     # поключаем к сети контейнер pong-containe

```

## Проверка работоспособности сетевого взаимодействия

Имеено данная часть кода реализует связь между 2-я контейнерами в Docker

*app.py*
```
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
```

Запускаем наши контейнеры и в разделе *Logs* можем наблюдать ссылки на Flask-проекты

*ping-container "Logs"*
```
2024-10-20 12:00:02    WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
2024-10-20 12:00:02  * Running on all addresses (0.0.0.0)
2024-10-20 12:00:02  * Running on http://127.0.0.1:5000
2024-10-20 12:00:02  * Running on http://172.17.0.3:5000
2024-10-20 12:00:02    Press CTRL+C to quit
2024-10-20 12:00:02  * Restarting with stat
2024-10-20 12:00:02  * Debugger is active!
2024-10-20 12:00:02  * Debugger PIN: 113-342-598
```

*pong-container "Logs"*
```
2024-10-20 12:00:01    WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
2024-10-20 12:00:01  * Running on all addresses (0.0.0.0)
2024-10-20 12:00:01  * Running on http://127.0.0.1:5001
2024-10-20 12:00:01  * Running on http://172.17.0.2:5001
2024-10-20 12:00:01    Press CTRL+C to quit
2024-10-20 12:00:01  * Restarting with stat
2024-10-20 12:00:01  * Debugger is active!
2024-10-20 12:00:01  * Debugger PIN: 978-957-277
```
Итог нашего проекта по данной ссылке: http://localhost:5000/ping

```
Ping ... Pong

```
## Заключение 

Данный пример позволяет понять основы сетевого взаимодействия 2 и более контейнеров в программе *Docker*










