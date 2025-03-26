# Web_documentation
Документация для работ по Web-Программированию 2025-2025

# Лабораторная работа #1
## Выполнил: Залетов Артём Дмитриевич, группа К3339
## Задание #1

**Задание:** Реализовать клиентскую и серверную часть приложения. Клиент отправляет серверу сообщение «Hello, server», и оно должно отобразиться на стороне сервера. В ответ сервер отправляет клиенту сообщение «Hello, client», которое должно отобразиться у клиента.  

**Клиент:**
```
import socket
def udp_client(host='127.0.0.1', port=12345):
    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    message = 'Hello, server'
    sock.sendto(message.encode(), (host, port))
    data, addr = sock.recvfrom(1024)  # Буфер 1024 байта
    print(f"Получено сообщение : {data.decode()}")

if __name__ == "__main__":
    udp_client()
```

**Сервер:**
```
import socket
def udp_server(host='127.0.0.1', port=12345):
    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    sock.bind((host, port))
    print(f"Сервер запущен")
    while True:
        data, addr = sock.recvfrom(1024)
        print(f"Получено сообщение: {data.decode()}")
        response = 'Hello, client'
        sock.sendto(response.encode(), addr)

if __name__ == "__main__":
    udp_server()
```
## Задание #2

**Задание:** Реализовать клиентскую и серверную часть приложения. Клиент запрашивает выполнение математической операции, параметры которой вводятся с клавиатуры. Сервер обрабатывает данные и возвращает результат клиенту. 

**Клиент:**
```
import socket
def get_positive_number(numb):
    while True:
        try:
            value = float(input(numb))
            if value > 0:
                return value
            else:
                print("Ошибка: значение должно быть положительным.")
        except ValueError:
            print("Ошибка: введено неверное число.")

def start_client():
    client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    client_socket.connect(('localhost', 12345))
    a = get_positive_number("Введите длину первого катета: ")
    b = get_positive_number("Введите длину второго катета: ")
    data = f"{a},{b}"
    client_socket.sendall(data.encode())
    result = client_socket.recv(1024).decode()
    print(f"Гипотенуза: {result}")
    client_socket.close()

if __name__ == '__main__':
    start_client()
```

**Сервер:**
```
import socket
import math
def start_server():
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.bind(('localhost', 12345))
    server_socket.listen()
    while True:
        conn, addr = server_socket.accept()
        try:
            data = conn.recv(1024).decode()
            if not data:
                break
            a, b = map(float, data.split(','))
            c = math.sqrt(a ** 2 + b ** 2)
            conn.sendall(str(c).encode())
        except Exception as e:
            conn.sendall(f"Ошибка: {str(e)}".encode())
        finally:
            conn.close()
if __name__ == '__main__':
    start_server()
```
## Задание #3

**Задание:** Реализовать серверную часть приложения. Клиент подключается к серверу, и в ответ получает HTTP-сообщение, содержащее HTML-страницу, которая сервер подгружает из файла index.html.

**Клиент:**
```
import socket
def main():
    host = '127.0.0.1'
    port = 8080
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as client_socket:
        client_socket.connect((host, port))
        request = "GET / HTTP/1.1\r\nHost: {}\r\n\r\n".format(host)
        client_socket.sendall(request.encode())
        response = client_socket.recv(4096).decode()
        print("Ответ сервера:\n", response)
if __name__ == "__main__":
    main()
```

**Сервер:**
```
import socket
def start_server(host='127.0.0.1', port=8080):
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as server_socket:
        server_socket.bind((host, port))
        server_socket.listen(1)
        while True:
            client_socket, _ = server_socket.accept()
            with client_socket:
                request = client_socket.recv(1024).decode()
                print(request)
                if 'GET / HTTP/1.1' in request:
                    response = "HTTP/1.1 200 OK\r\nContent-Type: text/html; charset=utf-8\r\n\r\n"
                    try:
                        with open('index.html', 'r', encoding='utf-8') as f:
                            content = f.read()
                            response += content
                    except FileNotFoundError:
                        response = "HTTP/1.1 404 Not Found\r\n\r\n<h1>404 Not Found</h1>"
                else:
                    response = "HTTP/1.1 404 Not Found\r\n\r\n<h1>404 Not Found</h1>"
                client_socket.sendall(response.encode())
if __name__ == "__main__":
    start_server()
```

## Задание #4

**Задание:** Реализовать двухпользовательский или многопользовательский чат. Для максимального количества баллов реализуйте многопользовательский чат.

**Клиент:**
```
import socket
import threading

class Client:
    def __init__(self, ip, port, encoding="utf-8", buffer_size=1024):
        self.running = True
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.sock.connect((ip, port))
        print("Connected to the server")

        self.encoding = encoding
        self.buffer_size = buffer_size

        self.receive_thread = threading.Thread(target=self.receive_thread_method)
        self.receive_thread.start()

        self.send_thread = threading.Thread(target=self.send_thread_method)
        self.send_thread.start()

    def disconnect(self):
        self.running = False
        self.sock.close()
        print("Disconnected from the server")

    def send_message(self, message: str):
        self.sock.sendall(message.encode(self.encoding))

    def receive_thread_method(self):
        print("Started receiving the messages from the server")
        while self.running:
            try:
                message = self.sock.recv(self.buffer_size).decode(self.encoding)
                print(message)
            except:
                self.disconnect()
                break

    def send_thread_method(self):
        while self.running:
            self.send_message(input())

if __name__ == "__main__":
    server_addr = "localhost"
    server_port = 8080
    Client(server_addr, server_port)

```

**Сервер:**
```
import socket
import threading
from dataclasses import dataclass
from enum import Enum

class UserState(Enum):
    NONE = -1
    CONNECTED = 0
    READY = 1

@dataclass
class User:
    conn: socket.socket
    state: UserState = UserState.NONE
    name: str = ""

class Server:
    def __init__(self, ip, port):
        self.running = True
        self.users: list[User] = []

        self.sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.sock.bind((ip, port))
        self.sock.listen()
        print("Server is listening")

        self.receive()

    def stop(self):
        self.running = False
        self.sock.close()
        print("Server stopped")

    def broadcast(self, label: str, message: str, except_list: list[User] = None):
        if except_list is None:
            except_list = []

        full_message = f"{label}: {message}"
        print(full_message)

        for user in self.users:
            if user.state != UserState.READY or user in except_list:
                continue
            user.conn.sendall(full_message.encode(encoding))

    def send_message(self, user: User, message: str):
        user.conn.sendall(message.encode(encoding))

    def process_user(self, conn: socket.socket):
        user = User(conn=conn, state=UserState.CONNECTED, name="")
        self.users.append(user)
        while self.running:
            try:
                if user.state == UserState.NONE:
                    user.state = UserState.CONNECTED
                elif user.state == UserState.CONNECTED:
                    self.send_message(user, "What is your name?")
                    name = conn.recv(buffer_size).decode(encoding)
                    if not name:
                        raise ValueError()

                    user.name = name
                    user.state = UserState.READY
                    self.broadcast(user.name, "connected")
                elif user.state == UserState.READY:
                    message = conn.recv(buffer_size).decode(encoding)
                    self.broadcast(user.name, message, [user])
            except:
                conn.close()
                self.users.remove(user)
                self.broadcast(user.name, "disconnected")
                break

    def receive(self):
        while self.running:
            conn, addr = self.sock.accept()
            print("Incoming connection", addr)

            thread = threading.Thread(target=self.process_user, args=(conn,))
            thread.start()

if __name__ == "__main__":
    encoding = "utf-8"
    buffer_size = 1024
    server_addr = "localhost"
    server_port = 8080
    Server(server_addr, server_port)

```

## Задание #5

**Задание:** Написать простой веб-сервер для обработки GET и POST HTTP-запросов с помощью библиотеки socket в Python.
**Сервер должен:**
* Принять и записать информацию о дисциплине и оценке по дисциплине.
* Отдать информацию обо всех оценках по дисциплинам в виде HTML-страницы.


**Сервер:**
```
import socket
from urllib.parse import parse_qs

grades = {}


class MyHTTPServer:
    def __init__(self, host, port):
        self.host = host
        self.port = port

    def serve_forever(self):
        serv_sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

        try:
            serv_sock.bind((self.host, self.port))
            serv_sock.listen()

            while True:
                conn, _ = serv_sock.accept()
                try:
                    self.serve_client(conn)
                except Exception as e:
                    print('Fail', e)
        finally:
            serv_sock.close()

    def serve_client(self, client):
        try:
            req = self.parse_request(client)
            resp = self.handle_request(req)
            self.send_response(client, resp)
        except ConnectionResetError:
            client = None

        if client:
            client.close()

    def parse_request_line(self, rfile):
        line = rfile.readline()
        line = line.decode('utf-8')
        return line.split()

    def parse_request(self, conn):
        rfile = conn.makefile('rb')
        method, target, ver = self.parse_request_line(rfile)

        request = {'data': {}, 'method': method}

        if method == 'POST':
            content_length = 0
            headers = []
            while True:
                header_line = rfile.readline().decode('utf-8')
                if header_line == '\r\n':
                    break
                headers.append(header_line.strip())

                if header_line.lower().startswith('content-length'):
                    content_length = int(header_line.split(':')[1].strip())

            body = rfile.read(content_length).decode('utf-8')
            request['data'] = parse_qs(body)

        elif '?' in target:
            request['method'] = 'GET'
            values = target.split('?')[1].split('&')
            for value in values:
                a, b = value.split('=')
                request['data'][a] = b

        return request

    def handle_request(self, req):
        if req['method'] == 'POST':
            return self.handle_post(req)
        else:
            return self.handle_get()

    def handle_get(self):
        content_type = 'text/html; charset=utf-8'
        body = '''
        <html>
        <head>
            <style>
                body {
                    background-color: #f4f7f6;
                    font-family: 'Arial', sans-serif;
                    color: #333;
                    margin: 0;
                    padding: 0;
                    display: flex;
                    justify-content: center;
                    align-items: center;
                    height: 100vh;
                }

                .container {
                    background-color: #fff;
                    padding: 30px;
                    border-radius: 15px;
                    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
                    width: 80%;
                    max-width: 600px;
                }

                h1 {
                    text-align: center;
                    color: #fa8e47;
                    font-size: 36px;
                    margin-bottom: 20px;
                }

                .form-group {
                    margin-bottom: 20px;
                }

                .form-group label {
                    font-size: 18px;
                    color: #555;
                    display: block;
                    margin-bottom: 5px;
                }

                .form-group input {
                    width: 100%;
                    padding: 12px;
                    font-size: 16px;
                    border: 1px solid #ddd;
                    border-radius: 8px;
                    margin-bottom: 10px;
                    transition: border-color 0.3s;
                }

                .form-group input:focus {
                    border-color: #fa8e47;
                    outline: none;
                }

                .form-group button {
                    width: 100%;
                    padding: 14px;
                    background-color: #fa8e47;
                    color: #fff;
                    font-size: 18px;
                    border: none;
                    border-radius: 8px;
                    cursor: pointer;
                    transition: background-color 0.3s;
                }

                .form-group button:hover {
                    background-color: #e07c3c;
                }

                table {
                    width: 100%;
                    margin-top: 30px;
                    border-collapse: collapse;
                }

                table th, table td {
                    padding: 12px;
                    text-align: center;
                    border: 1px solid #ddd;
                    font-size: 18px;
                }

                table th {
                    background-color: #fa8e47;
                    color: #fff;
                }

                table tr:nth-child(even) {
                    background-color: #f9f9f9;
                }
            </style>
        </head>
        <body>
            <div class="container">
                <h1>Добавить оценку</h1>
                <form method="post">
                    <div class="form-group">
                        <label for="discipline">Предмет</label>
                        <input type="text" id="discipline" name="discipline" required />
                    </div>
                    <div class="form-group">
                        <label for="grade">Оценка</label>
                        <input type="number" id="grade" name="grade" min="1" max="5" required />
                    </div>
                    <div class="form-group">
                        <button type="submit">Добавить</button>
                    </div>
                </form>

                <table>
                    <thead>
                        <tr>
                            <th>Дисциплина</th>
                            <th>Оценки</th>
                        </tr>
                    </thead>
                    <tbody>
        '''
        for subject in grades:
            body += f'<tr> <td>{subject}</td> <td>{", ".join(grades[subject])}</td> </tr>'
        body += '''
                    </tbody>
                </table>
            </div>
        </body>
        </html>
        '''
        body = body.encode('utf-8')
        headers = [('Content-Type', content_type)]
        return Response(200, 'OK', headers, body)

    def handle_post(self, request):
        discipline = request['data']['discipline'][0]
        grade = request['data']['grade'][0]

        if discipline not in grades:
            grades[discipline] = []
        if 1 <= int(grade) <= 5:
            grades[discipline].append(grade)

        return self.handle_get()

    def send_response(self, conn, resp):
        rfile = conn.makefile('wb')
        status_line = f'HTTP/1.1 {resp.status} {resp.reason}\r\n'
        rfile.write(status_line.encode('utf-8'))

        if resp.headers:
            for (key, value) in resp.headers:
                header_line = f'{key}: {value}\r\n'
                rfile.write(header_line.encode('utf-8'))

        rfile.write(b'\r\n')

        if resp.body:
            rfile.write(resp.body)

        rfile.flush()
        rfile.close()


class Response:
    def __init__(self, status, reason, headers=None, body=None):
        self.status = status
        self.reason = reason
        self.headers = headers
        self.body = body


if __name__ == '__main__':
    serv = MyHTTPServer('127.0.0.1', 8080)
    serv.serve_forever()

```

# Lab 2
# Лабораторная работа #2
## Выполнил: Залетов Артём Дмитриевич, группа К3339
## Вариант №2 Система для конференций
### Модели
```
from django.contrib.auth.models import AbstractUser, User
from django.db import models
from django.conf import settings


class Conference(models.Model):
    name = models.CharField("conference", max_length=50)
    topic = models.CharField("topic", blank=True, choices=[
        ("business", "business"),
        ("design", "design"),
        ("physics", "physics"),
    ], max_length=10)
    location = models.CharField("location", max_length=100)
    start_date = models.DateField("start date")
    end_date = models.DateField("end date")
    description = models.CharField("conference description", max_length=200)
    location_description = models.CharField("location description", max_length=200)
    terms = models.CharField("participation terms", max_length=1000)
    speaker = models.ManyToManyField(User, related_name="speaker")
    recommend = models.CharField("recommend", choices=[
        ("yes", "yes"),
        ("no", "no"),
    ], max_length=3)


    class Meta:
        verbose_name = "conference"
        verbose_name_plural = "conferences"

    def __str__(self):
        return f"{self.topic}: {self.name}"

    def written_by(self):
        return ", ".join([str(p) for p in self.speaker.all()])


class Comment(models.Model):
    name = models.ForeignKey(Conference, on_delete=models.CASCADE, verbose_name="conference")
    author = models.ForeignKey(User, on_delete=models.CASCADE, verbose_name="comment author")
    text = models.CharField("comment", max_length=100)
    rating = models.CharField("rating", choices =[
        ("1","1"),
        ("2","2"),
        ("3","3"),
        ("4","4"),
        ("5","5"),
        ("6","6"),
        ("7","7"),
        ("8","8"),
        ("9","9"),
        ("10","10"),

    ], max_length=2)


    class Meta:
        verbose_name = "comment"
        verbose_name_plural = "comments"

    def __str__(self):
        return f"{self.author}: {self.text}"

```
### Юрлы
```
from django.urls import path

from .views import index, ConferenceView, ConferenceDetailView

urlpatterns = [
    path("", index, name="index"),
    path("conferences/", ConferenceView.as_view(), name="conferences"),
    path(
        "conferences/<slug:pk>/",
        ConferenceDetailView.as_view(),
        name="conference-detail",
    ),
]
```
### Views
```
from django.shortcuts import render, redirect
from django.views import generic
from django.views.generic.edit import FormMixin
from django.http import HttpResponseRedirect
from django.urls import reverse
from .forms import PostComment
from .models import Conference, Comment
from django.contrib.auth.mixins import LoginRequiredMixin

def index(request):
    return render(request, "index.html")


class ConferenceView(generic.ListView):
    model = Conference
    context_object_name = "conferences"
    queryset = Conference.objects.all()
    template_name = "conferences.html"


class ConferenceDetailView(FormMixin, generic.DetailView):
    model = Conference
    template_name = "conference-detail.html"
    form_class = PostComment

    def get_context_data(self, **kwargs):
        context = super(ConferenceDetailView, self).get_context_data(**kwargs)
        context["form"] = PostComment(
            initial={"name": self.object, "author": self.request.user}
        )
        context["comments"] = Comment.objects.filter(name=self.get_object()).all()
        return context

    def post(self, request, *args, **kwargs):
        self.object = self.get_object()
        form = self.get_form()
        if form.is_valid():
            form.save()
        return HttpResponseRedirect(
            reverse("conference-detail", args=(self.object.pk,))
        )

```
# Lab 3
# Лабораторная работа #3
## Выполнил: Залетов Артём Дмитриевич, группа К3339
## Вариант №2 библиотека
### Модели
```
class User(AbstractUser):
    tel = models.CharField(verbose_name='Телефон', max_length=15, null=True, blank=True)

    REQUIRED_FIELDS = ['first_name', 'last_name', 'tel']

    def __str__(self):
        return self.username


class Instance(models.Model):
    id_instance = models.AutoField("ID_экземпляра", primary_key=True)
    section = models.CharField(max_length=20, verbose_name='Раздел')
    code = models.CharField(max_length=20, verbose_name='Артикул')
    year = models.IntegerField(verbose_name='Год издания')
    conditions = (
        ('х', 'хорошее'),
        ('у', 'удовлетворительное'),
        ('с', 'старое'),
    )
    condition = models.CharField(max_length=1, choices=conditions, verbose_name='Состояние экземпляра')
    book = models.ForeignKey('Book', verbose_name='Книга', on_delete=CASCADE)

    def __str__(self):
        return self.code


class Book(models.Model):
    id_book = models.AutoField("ID_книги", primary_key=True)
    name = models.CharField(max_length=50, verbose_name='Название')
    author = models.CharField(max_length=70, verbose_name="ФИО автора")
    publisher = models.CharField(max_length=30, verbose_name='Издательство')

    def __str__(self):
        return self.name


class Reader(models.Model):
    ticket = models.CharField(max_length=20, verbose_name='Номер читательского билета')
    name = models.CharField(max_length=70, verbose_name="ФИО")
    passport = models.CharField(max_length=20, verbose_name='Номер паспорта')
    birth_date = models.DateField(verbose_name='Дата рождения')
    address = models.CharField(max_length=100, verbose_name='Адрес')
    phone_number = models.CharField(max_length=20, verbose_name='Номер телефона')
    educations = (
        ('н', 'начальное'),
        ('с', 'среднее'),
        ('в', 'высшее'),
    )
    education = models.CharField(max_length=1, choices=educations, verbose_name='Образование')
    degree = models.BooleanField(default=False, verbose_name='Наличие ученой степени')
    registration_date = models.DateField(verbose_name='Дата регистрации')
    instances = models.ManyToManyField('Instance', verbose_name='Взятые книги', through='ReaderBook',
                                       related_name='reader_book')
    room = models.ForeignKey('Room', verbose_name='Зал, за которым закреплен читатель', on_delete=CASCADE, null=True)

    def __str__(self):
        return self.name


class ReaderRoom(models.Model):
    reader = models.ForeignKey('Reader', verbose_name='Читатель', on_delete=CASCADE)
    room = models.ForeignKey('Room', verbose_name='Зал', on_delete=CASCADE)
    date = models.DateField(verbose_name='Дата закрепления зала', null=True)


class BookInst(models.Model):
    inst = models.ForeignKey('Instance', verbose_name='Экземпляр', on_delete=CASCADE)
    book = models.ForeignKey('Book', verbose_name='Книга', on_delete=CASCADE)


class ReaderBook(models.Model):
    reader = models.ForeignKey('Reader', verbose_name='Читатель', on_delete=CASCADE)
    book = models.ForeignKey('Instance', verbose_name='Экземпляр', on_delete=CASCADE)
    date = models.DateField(verbose_name='Дата выдачи экземпляра книги', null=True)


class BookRoom(models.Model):
    book = models.ForeignKey('Instance', verbose_name='Книга', on_delete=CASCADE)
    room = models.ForeignKey('Room', verbose_name='Зал', on_delete=CASCADE)


class Room(models.Model):
    name = models.CharField(max_length=20, verbose_name='Название')
    capacity = models.IntegerField(verbose_name='Вместимость')
    books = models.ManyToManyField('Instance', verbose_name='Книги', through='BookRoom', related_name='book_room')

    def __str__(self):
        return self.name
```
### Юрлы
```
urlpatterns = [
    path('readers/list/', ReaderListAPIView.as_view()),
    path('readers/create/', CreateReader.as_view()),
    path('readers/<int:pk>/', OneReader.as_view()),
    path('books/list/', BookListAPIView.as_view()),
    path('books/create/', CreateBook.as_view()),
    path('books/<int:pk>/', OneBook.as_view()),
    path('inst/list/', InstanceListAPIView.as_view()),
    path('inst/create/', CreateInstance.as_view()),
    path('inst/<int:pk>/', OneInstance.as_view()),
    path('rooms/list/', RoomListAPIView.as_view()),
    path('rooms/create/', RoomCreateAPIView.as_view()),
    path('rooms/<int:pk>/', OneRoom.as_view()),
    path('book/readers/', BookReaders.as_view()),
    path('book/room/', RoomBook.as_view()),
    path('room/readers/', RoomReader.as_view()),
    path('book/inst/', BookInst.as_view()),
    path('readers/inst/<int:pk>', ReadersInst.as_view()),
]
```
### Views
```
class ReaderListAPIView(ListAPIView):
    serializer_class = ReaderSerializer
    queryset = Reader.objects.all()


class CreateReader(CreateAPIView):
    serializer_class = ReaderSerializer
    queryset = Reader.objects.all()


class BookListAPIView(ListAPIView):
    serializer_class = BookSerializer
    queryset = Book.objects.all()


class CreateBook(CreateAPIView):
    serializer_class = BookSerializer
    queryset = Book.objects.all()


class InstanceListAPIView(ListAPIView):
    serializer_class = InstanceSerializer
    queryset = Instance.objects.all()


class CreateInstance(CreateAPIView):
    serializer_class = InstanceSerializer
    queryset = Instance.objects.all()


class OneBook(RetrieveUpdateDestroyAPIView):
    serializer_class = BookSerializer
    queryset = Book.objects.all()


class OneInstance(RetrieveUpdateDestroyAPIView):
    serializer_class = InstanceSerializer
    queryset = Instance.objects.all()


class OneReader(RetrieveUpdateDestroyAPIView):
    serializer_class = ReaderSerializer
    queryset = Instance.objects.all()


class BookReaders(CreateAPIView):
    serializer_class = ReaderBookSerializer
    queryset = ReaderBook.objects.all()


class RoomListAPIView(ListAPIView):
    serializer_class = RoomSerializer
    queryset = Room.objects.all()


class RoomCreateAPIView(CreateAPIView):
    serializer_class = RoomSerializer
    queryset = Room.objects.all()


class OneRoom(RetrieveUpdateDestroyAPIView):
    serializer_class = RoomSerializer
    queryset = Room.objects.all()


class RoomBook(CreateAPIView):
    serializer_class = BookRoomSerializer
    queryset = BookRoom.objects.all()


class RoomReader(CreateAPIView):
    serializer_class = ReaderRoomSerializer
    queryset = ReaderRoom.objects.all()


class BookInst(CreateAPIView):
    serializer_class = BookInstSerializer
    queryset = BookInst.objects.all()


class ReadersInst(generics.RetrieveAPIView):
    serializer_class = ReaderInstsSerializer
    queryset = Reader.objects.all()
```
### Serializers
```
class ReaderSerializer(serializers.ModelSerializer):
    books = serializers.SlugRelatedField(read_only=True, many=True, slug_field='books')

    class Meta:
        model = Reader
        fields = "__all__"


class BookSerializer(serializers.ModelSerializer):
    class Meta:
        model = Book
        fields = "__all__"


class InstanceSerializer(serializers.ModelSerializer):
    class Meta:
        model = Instance
        fields = "__all__"


class ReaderBookSerializer(serializers.ModelSerializer):
    class Meta:
        model = ReaderBook
        fields = "__all__"


class RoomSerializer(serializers.ModelSerializer):
    books = serializers.SlugRelatedField(read_only=True, many=True, slug_field='id_instance')

    class Meta:
        model = Room
        fields = "__all__"


class BookRoomSerializer(serializers.ModelSerializer):
    class Meta:
        model = BookRoom
        fields = "__all__"


class ReaderRoomSerializer(serializers.ModelSerializer):
    class Meta:
        model = ReaderRoom
        fields = "__all__"


class BookInstSerializer(serializers.ModelSerializer):
    class Meta:
        model = BookInst
        fields = "__all__"


class ReaderInstsSerializer(serializers.ModelSerializer):
    class Meta:
        model = Reader
        fields = ["instances"]


class RecentlyBookDateSerializer(serializers.ModelSerializer):
    class Meta:
        model = ReaderBook
        fields = ["reader"]

```
# Lab 4
# Лабораторная работа #4
## Выполнил: Залетов Артём Дмитриевич, группа К3339
### Routers
```
import Vue from 'vue';
import VueRouter from 'vue-router';


import SignIn from '../views/reader/SignIn.vue'
import SignUp from '../views/reader/SignUp.vue'
import Home from '../views/Home.vue'
import Profile from '../views/reader/Profile.vue'
import ProfileEdit from '../views/reader/ProfileEdit.vue'
import LogOut from '../views/reader/LogOut.vue'
import ParticipationView from "@/components/ParticipationView.vue";
import ParticipantsView from "@/components/ParticipantsView.vue";
import DogRegister from "@/views/reader/DogRegister.vue"
import DogGrade from "@/views/reader/DogGrade.vue"
import DogEdit from "@/views/reader/DogEdit.vue"
import ParticipationEdit from "@/views/reader/ParticipationEdit.vue";

Vue.use(VueRouter)

const routes = [
  {
    path: '/',
    name: 'home',
    component: Home
  },
  {
    path: '/show/signup',
    name: 'signup',
    component: SignUp
  },
  {
    path: '/show/logout',
    name: 'logout',
    component: LogOut
  },
  {
    path: '/show/signin',
    name: 'signin',
    component: SignIn
  },
  {
    path: '/show/profile',
    name: 'profile',
    component: Profile
  },
  {
    path: '/show/profile/edit',
    name: 'profile_edit',
    component: ProfileEdit
  },
  {
    path: '/participants/DogEdit/:dogId',
    name: 'Dog_edit',
    component: DogEdit,
    props: true
  },
  {
    path: '/participation/ParticipationEdit/:dogId',
    name: 'Participation_Edit',
    component: ParticipationEdit,
    props: true
  },
  {
    path: '/about',
    name: 'about',
    component: () => import('../views/AboutView.vue')
  },
  {
    path: '/participation',
    component: ParticipationView
  },
  {
    path: '/participants',
    component: ParticipantsView
  },

  {
    path: '/show/profile/regdog',
    name: 'regdog',
    component: DogRegister
  },
  {
    path: '/show/profile/grading',
    name: 'grading',
    component: DogGrade
  }

]

const router = new VueRouter({
  mode: 'history',
  base: process.env.BASE_URL,
  routes
})

export default router
```
### Главная страница
```
<template>
  <div>
    <v-card elevation="2" outlined class="my-4 card-container">
      <v-card-text>
        <h2 class="section-title">Меню</h2>
        <div class="nav-links">
          <!--<v-btn text @click="goParticipations" class="nav-btn">Participations</v-btn>
          <v-btn text @click="goParticipants" class="nav-btn">Participants</v-btn>-->

          <template v-if="authorized">
            <v-btn text @click="goParticipations" class="nav-btn">Результаты</v-btn>
            <v-btn text @click="goParticipants" class="nav-btn">Участники</v-btn>
            <!--<v-btn text @click="goLogOut" class="nav-btn">Клубы</v-btn>-->
            <v-btn text @click="goProfile" class="nav-btn">Профиль</v-btn>
            <v-btn text @click="goLogOut" class="nav-btn">Выйти</v-btn>
          </template>

          <template v-else>
            <v-btn text @click="goSignIn" class="nav-btn">Войти</v-btn>
            <v-btn to="/show/signup" class="nav-btn">Регистрация</v-btn>
          </template>
        </div>
      </v-card-text>
    </v-card>
  </div>
</template>

<script>
export default {
  name: 'HomePage', // Renamed component to multi-word

  data: () => ({
    authorized: false
  }),

  created () {
    if (sessionStorage.getItem('auth_token')) {
      if (sessionStorage.getItem('auth_token') !== '-1') {
        this.authorized = true
      }
    }
  },

  methods: {
    goParticipations() {
      this.$router.push('/participation');
    },

    goParticipants() {
      this.$router.push('/participants');
    },

    goProfile() {
      this.$router.push({ name: 'profile' });
    },

    goLogOut() {
      this.$router.push({ name: 'logout' });
    },

    goSignIn() {
      this.$router.push({ name: 'signin' });
    }
  }
}
</script>

<style scoped>
.card-container {
  max-width: 500px;
  margin: 2rem auto;
  padding: 2rem;
  border-radius: 8px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
}

.section-title {
  font-size: 1.5rem;
  font-weight: bold;
  color: #3f51b5;
  margin-bottom: 1.5rem;
  text-align: center;
}

.nav-links {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  align-items: center;
}

.nav-btn {
  text-decoration: none;
  color: #283593;
  font-weight: bold;
  width: 100%;
  max-width: 250px;
}

.nav-btn:hover {
  background-color: #f1f1f1;
  transition: background-color 0.3s ease;
}
</style>

```
###Страница со списком участников
```
<template>
  <div class="participants-container">
    <v-row>
      <v-col cols="12" md="6">
        <v-select
          v-model="selectedBreed"
          :items="breedOptions"
          label="Фильтр по породе"
          clearable
          class="custom-select"
        ></v-select>
      </v-col>
      <v-col cols="12" md="6">
        <v-select
          v-model="selectedClub"
          :items="clubOptions"
          label="Фильтр по клубу"
          clearable
          class="custom-select"
        ></v-select>
      </v-col>
    </v-row>
    <v-row>
      <v-col v-for="participant in filteredParticipants" :key="participant.id" cols="12" md="6" lg="4">
        <v-card elevation="5" class="participant-card">
          <v-card-title class="card-title">{{ participant.name }}</v-card-title>
          <v-card-text>
            <div class="info-item"><strong>Порода:</strong> {{ participant.breed }}</div>
            <div class="info-item"><strong>Возраст:</strong> {{ participant.age }}</div>
            <div class="info-item"><strong>Родословная:</strong> {{ participant.family }}</div>
            <div class="info-item"><strong>Информация о владельце:</strong> {{ participant.owner_data }}</div>
            <div class="info-item"><strong>Клуб:</strong> {{ participant.club }}</div>
          </v-card-text>
          <v-card-actions>
            <v-btn color="primary" class="edit-btn" @click="editParticipant(participant.id)">Редактировать</v-btn>
          </v-card-actions>
        </v-card>
      </v-col>
    </v-row>
  </div>
</template>

<script>
export default {
  props: {
    participants: {
      type: Array,
      required: true
    }
  },
  data() {
    return {
      selectedBreed: null,
      selectedClub: null
    };
  },
  computed: {
    breedOptions() {
      return [...new Set(this.participants.map(p => p.breed))];
    },
    clubOptions() {
      return [...new Set(this.participants.map(p => p.club))];
    },
    filteredParticipants() {
      return this.participants.filter(p => {
        const matchesBreed = this.selectedBreed ? p.breed === this.selectedBreed : true;
        const matchesClub = this.selectedClub ? p.club === this.selectedClub : true;
        return matchesBreed && matchesClub;
      });
    }
  },
  methods: {
    editParticipant(participantId) {
      this.$router.push({ name: 'Dog_edit', params: { dogId: participantId } });
    }
  }
};
</script>

<style scoped>
.participants-container {
  padding: 24px;
  background: #f0f4ff;
  border-radius: 12px;
}

.custom-select {
  max-width: 400px;
  margin: 0 auto 16px;
}

.participant-card {
  background-color: #ffffff;
  border-radius: 16px;
  padding: 20px;
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.15);
  transition: transform 0.2s ease-in-out;
}

.participant-card:hover {
  transform: translateY(-5px);
}

.card-title {
  font-size: 20px;
  font-weight: bold;
  color: #1e3a8a;
  text-align: center;
  margin-bottom: 12px;
}

.info-item {
  font-size: 16px;
  margin-bottom: 8px;
  padding: 8px 12px;

```
### Страница с результатами 
```
<template>
  <div class="participation-container">
    <h2 class="title">Participation Details</h2>
    <div class="card-grid">
      <v-card v-for="participation in participations" :key="participation.id" elevation="3" class="small-card">
        <v-card-text>
          <p><strong>Medal:</strong> {{ participation.medal }}</p>
          <p><strong>Vaccine:</strong> {{ participation.vaccinated }}</p>
          <p><strong>Verified:</strong> {{ participation.dismissed }}</p>
          <p><strong>Grade:</strong> {{ participation.final_grade }}</p>
          <p><strong>Participant:</strong> {{ participation.participant}}</p>
          <v-btn color="primary" class="edit-button" @click="goParticipationsEdit(participation.id)">Edit</v-btn>
        </v-card-text>
      </v-card>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    participations: {
      type: Array,
      required: true
    }
  },
  methods:
      {
        goParticipationsEdit(dogId) {
        this.$router.push({ name: "Participation_Edit", params: { dogId: dogId } });
}


  }
}
</script>

<style scoped>
.participation-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 2rem;
}

.title {
  font-size: 2rem;
  font-weight: bold;
  color: #3f51b5;
  margin-bottom: 1rem;
}

.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1rem;
  width: 100%;
  max-width: 800px;
}

.small-card {
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
  border-radius: 8px;
  padding: 1rem;
  background-color: #ffffff;
  text-align: center;
  position: relative;
}

p {
  font-size: 0.9rem;
  margin: 5px 0;
}

.edit-button {
  margin-top: 10px;
  width: 100%;
}
</style>

```
### Профиль
```
<template>
  <div class="edit">
    <h2 class="profile-title">Profile</h2>
    <h3 class="welcome-message">Nice to see you, {{ login() }}!</h3>
    <v-card class="profile-card">
      <v-card-text class="profile-info">
        <div class="text--primary">
          <p><strong>First name:</strong> {{ first_name }}</p>
          <p><strong>Last name:</strong> {{ last_name }}</p>
          <p><strong>Telephone number:</strong> {{ tel }}</p>
        </div>
        <div class="profile-actions">
          <v-btn @click="goRegister" class="profile-button primary">Зарегистрировать собаку</v-btn>
          <v-btn @click="goGrade" class="profile-button secondary">Поставить оценку</v-btn>
          <v-btn @click="goEdit" class="profile-button success">Редактировать профиль</v-btn>
          <v-btn @click="goHome" class="profile-button error">На главную</v-btn>
        </div>
      </v-card-text>
    </v-card>
  </div>
</template>

<script>
export default {
  name: 'UserProfile',
  data() {
    return {
      first_name: '',
      last_name: '',
      tel: '',
    };
  },
  created() {
    this.loadReaderData();
  },
  methods: {
    async loadReaderData() {
      try {
        const response = await this.axios.get('http://127.0.0.1:8000/auth/users/me/', {
          headers: {
            Authorization: `Token ${sessionStorage.getItem('auth_token')}`
          }
        });
        this.first_name = response.data.first_name;
        this.last_name = response.data.last_name;
        this.tel = response.data.tel;
      } catch (error) {
        console.error('Error fetching user data:', error);
      }
    },
    goHome() {
      this.$router.push({ name: 'home' });
    },
    goRegister() {
      this.$router.push({ name: 'regdog' });
    },
    goEdit() {
      this.$router.push({ name: 'profile_edit' });
    },
    goGrade() {
      this.$router.push({ name: 'grading' });
    },
    login() {
      return sessionStorage.getItem('username');
    }
  }
};
</script>

<style scoped>
.edit {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 2rem;
}

.profile-title {
  font-size: 2.5rem;
  font-weight: bold;
  color: #3f51b5;
  margin-bottom: 0.5rem;
}

.welcome-message {
  font-size: 1.5rem;
  color: #607d8b;
  margin-bottom: 2rem;
}

.profile-card {
  width: 100%;
  max-width: 600px;
  box-shadow: 0 6px 15px rgba(0, 0, 0, 0.15);
  border-radius: 12px;
  padding: 2rem;
  background-color: #ffffff;
  text-align: center;
}

.profile-info p {
  font-size: 1.1rem;
  line-height: 1.6;
  color: #444;
  margin: 8px 0;
}

.profile-actions {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  align-items: center;
  margin-top: 1.5rem;
}

.profile-button {
  width: 100%;
  max-width: 400px;
  padding: 10px;
  font-size: 1.1rem;
  border-radius: 8px;
  transition: background-color 0.3s ease;
}

.primary {
  background-color: #3f51b5;
  color: white;
}

.secondary {
  background-color: #009688;
  color: white;
}

.success {
  background-color: #4caf50;
  color: white;
}

.error {
  background-color: #f44336;
  color: white;
}

.profile-button:hover {
  filter: brightness(1.2);
}
</style>

```
### Регистрация собаки
```
<template>
   <div class="edit">
    <h3>Dog registration</h3>
      <v-form
      @submit.prevent="signDogs"
      ref="editForm"
      class="my-2">
      <v-row>
        <v-col cols="5" class="mx-auto">

          <v-text-field
            label="Dog's name"
            v-model="editForm.name"
            name="name"/>

          <v-select
            v-model="editForm.breed"
            :items="options"
            name="breed"
            label="Breed"
          ></v-select>

          <v-text-field
            label="Age"
            v-model="editForm.age"
            name="age"
            type="number"/>

          <v-text-field
            label="Pedigree"
            v-model="editForm.family"
            name="family"/>


          <v-text-field
            label="Owner info"
            v-model="editForm.owner_data"
            name="owner_data"/>

            <v-btn type="submit" color="#283593" dark>Register the dog</v-btn>
        </v-col>
      </v-row>
    </v-form>
    <v-card>
      <v-card-text style="margin-top:1cm">
        <a @click.prevent="goProfile" style="text-decoration: none; color: #283593">Back</a>
      </v-card-text>
    </v-card>
   </div>
</template>

<script>
import $ from "jquery";
export default {
  name: 'DogRegister',
  data: () => ({ 
    editForm: {
      //participant: Object,
      name: '',
      breed: '',
      age: '',
      family: '',
      owner_data: '',
      //club,

    },

    options: ['h', 'b', 't'],
    //participants: 
  }),
    methods: {
        async signDogs () {
            console.log(1)
        
        $.ajax({
                    type: "POST",
                    data: {
                            name: this.editForm.name,
                            breed: this.editForm.breed,
                            age: this.editForm.age,
                            family: this.editForm.family,
                            owner_data: this.editForm.owner_data
                    },
                    url: "http://127.0.0.1:8000/participants/"
                }).done(function () {
                    console.log(this.data)
                    alert("Registration succeed")
                    //this.$router.push({ name: 'participants' })
                });
        },
        
        goProfile () {
        this.$router.push({ name: 'profile' })
    },
    }
}

</script>
```
### Выставление оценки
```
<template>
  <div class="edit">
    <h3>Grade</h3>
    <v-form
      @submit.prevent="signPart" 
      ref="editFormPart"
      class="my-2">
      <v-row>
        <v-col cols="5" class="mx-auto">
          
          <v-select
            v-model="editFormPart.participant"
            :items="participants"
            item-text="name"
            item-value="id"
            name="participant"
            label="Participant"
          ></v-select>

          <v-select
            v-model="editFormPart.medal"
            :items="medals"
            name="medal"
            label="Medal"
          ></v-select>

          <v-text-field
            label="Date of vaccination"
            v-model="editFormPart.vaccinated"
            name="vaccinated"
            type="date"/>

          <v-checkbox
            v-model="editFormPart.dismissed"
            :label="'Dismissed'"
          ></v-checkbox>


          <v-text-field
            label="Enter the grade"
            v-model="editFormPart.final_grade"
            type='number'
            name="final_grade"/>

            <v-btn type="submit" color="#283593" dark>Grade</v-btn>
        </v-col>
      </v-row>
    </v-form>
    <v-card>
      <v-card-text style="margin-top:1cm">
        <a @click.prevent="goProfile" style="text-decoration: none; color: #283593">Back</a>
      </v-card-text>
    </v-card>
  </div>
</template>

<script>
import $ from "jquery";
const array1 = [];
import axios from "axios";

export default {
  name: 'DogGrade',
  data: () => ({

    editFormPart: {
      medal: "",
      vaccinated: "",
      dismissed: "",
      final_grade: "",
      participant: "",
    },

    medals: ['g', 's', 'b'],
    participants: array1, 
  }),
    methods: {
        
        async signPart () {
            console.log(1)
        
        $.ajax({
                    type: "POST",
                    data: {
                            medal: this.editFormPart.medal,
                            vaccinated: this.editFormPart.vaccinated,
                            dismissed: this.editFormPart.dismissed,
                            final_grade: this.editFormPart.final_grade,
                            participant: this.editFormPart.participant,
                    },
                    url: "http://127.0.0.1:8000/participation/"
                }).done(function () {
                    console.log(this.data)
                    alert("Done")
                    //this.$router.push({ name: 'participants' })
                });
        },

        goProfile () {
        this.$router.push({ name: 'profile' })
    }
    },
      beforeMount: function () {
        this.$nextTick(async function () {
            const response = await axios.get('http://127.0.0.1:8000/participants/?format=json')
            for (let part of response.data) {
                array1.push(part);
      }
    })
  }
        
}


</script>

```
### Изменение данных о собаке
```
<template>
  <div class="edit-container">
    <v-card class="edit-card">
      <v-card-title>Edit Dog Information</v-card-title>
      <v-card-text>
        <v-form @submit.prevent="updateDog" ref="editForm">
          <v-text-field label="Dog's Name" v-model="editForm.name" outlined dense />
          <v-text-field label="Breed" v-model="editForm.breed" outlined dense />
          <v-text-field label="Age" v-model="editForm.age" type="number" outlined dense />
          <v-text-field label="Pedigree" v-model="editForm.family" outlined dense />
          <v-text-field label="Owner Info" v-model="editForm.owner_data" outlined dense />

          <v-btn color="primary" block class="mt-3" @click="updateDog">
            <v-icon left>mdi-content-save</v-icon> Update Dog Info
          </v-btn>
        </v-form>
      </v-card-text>
      <v-card-actions>
        <v-btn text color="blue darken-2" class="back-link" @click="goParticipants">
          <v-icon left>mdi-arrow-left</v-icon> Back
        </v-btn>
      </v-card-actions>
    </v-card>
  </div>
</template>

<script>
export default {
  props: {
    dogId: {
      type: [String, Number],
      required: true
    }
  },
  data() {
    return {
      editForm: {
        name: '',
        breed: '',
        age: '',
        family: '',
        owner_data: ''
      }
    };
  },
  mounted() {
    console.log("Dog ID:", this.dogId);
    this.loadDogData();
  },
  methods: {

  goParticipants() {
    this.$router.push({ path: "/participants" });
  },


    async loadDogData() {
      try {
        const response = await this.axios.get(`http://127.0.0.1:8000/participants/${this.dogId}`);
        this.editForm = response.data;
      } catch (error) {
        console.error("Error loading dog data:", error);
      }
    },
    async updateDog() {
      try {
        const response = await this.axios.patch(`http://127.0.0.1:8000/participants/${this.dogId}/`, this.editForm, {
          headers: {
            Authorization: `Token ${sessionStorage.getItem('auth_token')}`
          }
        });
        console.log(response);
        this.$router.push({ path: "/participants" });
      } catch (e) {
        console.error(e.response ? e.response.data : e.message);
      }
    }
  }
};
</script>

<style scoped>
.edit-container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}

.edit-card {
  width: 400px;
  padding: 20px;
  border-radius: 12px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
}

.back-link {
  text-transform: none;
  font-weight: bold;
}
</style>

```

### Изменение профиля эксперта
```
<template>
  <div class="edit">
    <h2 class="edit-title">Edit the Profile</h2>
    <v-form
      @submit.prevent="saveChanges"
      ref="changeForm"
      class="form-container">
      <v-row>
        <v-col cols="12" md="6" class="mx-auto">
          <v-text-field
            label="First name"
            v-model="changeForm.first_name"
            name="first_name"
            outlined
            dense
            class="input-field"/>

          <v-text-field
            label="Last name"
            v-model="changeForm.last_name"
            name="last_name"
            outlined
            dense
            class="input-field"/>

          <v-text-field
            label="Telephone number"
            v-model="changeForm.tel"
            name="tel"
            outlined
            dense
            class="input-field"/>

          <v-btn type="submit" class="save-button" color="primary" dark>Save</v-btn>
        </v-col>
      </v-row>
    </v-form>
    <p class="back-link">
      <router-link to="/show/profile" class="back-button">Back</router-link>
    </p>
  </div>
</template>

<script>
export default {
  name: 'ProfileEdit',

  data: () => ({
    reader_old: Object,
    changeForm: {
      first_name: '',
      last_name: '',
      tel: '',
    },
  }),

  methods: {
    async saveChanges () {
      for (const [key, value] of Object.entries(this.changeForm)) {
        if (value === '') {
          delete this.changeForm[key]
        }
      }
      try {
        const response = await this.axios
          .patch('http://127.0.0.1:8000/auth/users/me/',
            this.changeForm, {
              headers: {
                Authorization: `Token ${sessionStorage.getItem('auth_token')}`
              }
              })
        console.log(response)
        this.$refs.changeForm.reset()
        await this.$router.push({name: 'profile'})
      } catch (e) {
        console.error(e.message)
      }
    }
  }
}
</script>

<style scoped>
.edit {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 2rem;
}

.edit-title {
  font-size: 2rem;
  color: #3f51b5;
  margin-bottom: 2rem;
}

.form-container {
  width: 100%;
  max-width: 500px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
  border-radius: 8px;
  padding: 2rem;
  background-color: #fff;
}

.input-field {
  margin-bottom: 1.5rem;
}

.save-button {
  width: 100%;
  padding: 10px;
  font-weight: bold;
  border-radius: 8px;
}

.back-link {
  margin-top: 1.5rem;
  text-align: center;
}

.back-button {
  text-decoration: none;
  color: #283593;
  font-weight: bold;
  transition: color 0.3s ease;
}

.back-button:hover {
  color: #1a237e;
}
</style>

```


