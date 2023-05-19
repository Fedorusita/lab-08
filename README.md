# lab-08
1.Создадим директорию Server и добавим туда слеюшие файлы:  
   -Server.py
 ```Python
 import http.server
import socketserver

handler = http.server.SimpleHTTPRequestHandler
with socketserver.TCPServer(("", 1234), handler) as httpd:
   httpd.serve_forever()
   ```
   -index.html(с текстом который выведется нам в консоль)  
 ```
 Im server
 ```
   -Dockerfile
 ```
 FROM python:latest

ADD server.py /server/
ADD index.html /server/

WORKDIR /server/
```
2.Создадим директорию client и добавим туда следующие файлы:
   -Client.py
```Python
import urllib.request

fp = urllib.request.urlopen("http://localhost:1234/")

encodedContent = fp.read()
decodedContent = encodedContent.decode("utf8")

print(decodedContent)

fp.close()
```
   -Dockerfile
```
FROM python:latest

ADD client.py /client/

WORKDIR /client/
```
3. В главной директории создадим файл docker-compose.yml
```
version: "3"

services:

  server:

    build: server/
    command: python ./server.py
    ports:
      - 1234:1234

  client:

    build: client/
    command: python ./client.py
    network_mode: host
    depends_on:
      - server
```
4. И наконец сделаем Action.yml
```
name: Docker Compose

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Login to DockerHub
      uses: docker/login-action@v1
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    - name: Build Docker images
      run: |
        docker-compose build
    - name: Push Docker images
      run: |
        docker-compose push
    - name: Deploy with Docker Compose
      run: |
        docker-compose up -d
        ```
 

