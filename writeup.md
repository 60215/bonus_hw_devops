#  Бонусное ДЗ девопс2
## Выполнил Шумай Артём


## Задание 1
```
artem@DESKTOP-NBAE2LL:~/projects/hw_bonus$ ls
Dockerfile  url-probe
```
Контенты докер файла:
```
artem@DESKTOP-NBAE2LL:~/projects/hw_bonus$ cat Dockerfile
FROM python:3.10-alpine

RUN adduser prober -D
USER prober

WORKDIR /probe-script
ADD url-probe/probe.py .

ENV PROBE_TIMEOUT=5

CMD ["python", "probe.py", "urls.txt"]
```
Построим образ
```
artem@DESKTOP-NBAE2LL:~/projects/hw_bonus$ docker build -t url-probe .
[+] Building 8.2s (9/9) FINISHED                                                                                            docker:default
 => [internal] load build definition from Dockerfile                                                                                  0.1s
 => => transferring dockerfile: 208B                                                                                                  0.0s
 => [internal] load metadata for docker.io/library/python:3.10-alpine                                                                 2.5s
 => [internal] load .dockerignore                                                                                                     0.1s
 => => transferring context: 2B                                                                                                       0.0s
 => [1/4] FROM docker.io/library/python:3.10-alpine@sha256:8ca9f9d0f4df2af674092e7e9d79ce7985e783cecfe070f8ea850801b6520bf7           3.9s
 => => resolve docker.io/library/python:3.10-alpine@sha256:8ca9f9d0f4df2af674092e7e9d79ce7985e783cecfe070f8ea850801b6520bf7           0.1s
 => => sha256:a830925508620210045650b6ca2b9f7ec8f821035a6f7dcb883fac0643d29937 249B / 249B                                            0.3s
 => => sha256:accc0b865ae09ef0bfaa380023a2aefb88de3582fd6beb2ff7f5d8947dc0b863 18.15MB / 18.15MB                                      2.2s
 => => sha256:d2d6feadfdd46cbe837c67160780ec2775660324353138c725dd19659ddc891f 416.28kB / 416.28kB                                    1.5s
 => => sha256:e2de96513ba9eb53b431787ec8a65cdde380ac4772a3e4c4b714dcfde2a102b5 3.85MB / 3.85MB                                        2.6s
 => => extracting sha256:e2de96513ba9eb53b431787ec8a65cdde380ac4772a3e4c4b714dcfde2a102b5                                             0.1s
 => => extracting sha256:d2d6feadfdd46cbe837c67160780ec2775660324353138c725dd19659ddc891f                                             0.2s
 => => extracting sha256:accc0b865ae09ef0bfaa380023a2aefb88de3582fd6beb2ff7f5d8947dc0b863                                             0.6s
 => => extracting sha256:a830925508620210045650b6ca2b9f7ec8f821035a6f7dcb883fac0643d29937                                             0.0s
 => [internal] load build context                                                                                                     0.1s
 => => transferring context: 3.99kB                                                                                                   0.0s
 => [2/4] RUN adduser prober -D                                                                                                       0.6s
 => [3/4] WORKDIR /probe-script                                                                                                       0.1s
 => [4/4] ADD url-probe/probe.py .                                                                                                    0.1s
 => exporting to image                                                                                                                0.7s
 => => exporting layers                                                                                                               0.4s
 => => exporting manifest sha256:aa95a1e7cd6e05d4785cd9b9106a8a062aa9c4044aa2139d4a46f05430a75b1f                                     0.0s
 => => exporting config sha256:a150bd4255e3940fa18915f7bc8a66e1a14adf032ac449d1d83a948982dda518                                       0.0s
 => => exporting attestation manifest sha256:245357e2c4b3532f9018775f45e68600b5ef0f7381f0ef068cd37ae078c02238                         0.1s
 => => exporting manifest list sha256:34e098f0b9fac71bbc3c97a41a6e91fa63dbf388e49a40eeeb2b5d58539794d2                                0.0s
 => => naming to docker.io/library/url-probe:latest                                                                                   0.0s
 => => unpacking to docker.io/library/url-probe:latest
```
Все успешно. Запустим образ с нашими параметрами.
```
artem@DESKTOP-NBAE2LL:~/projects/hw_bonus$ docker run --rm -v $PWD/url-probe/urls.txt:/probe-script/urls.txt -e PROBE_TIMEOUT=3 url-probe
OK   200  0.073s https://example.com
FAIL 404  0.034s https://example.com/no-such-page
http://no-such-host.invalid: <urlopen error [Errno -2] Name does not resolve>
FAIL   0  0.013s http://no-such-host.invalid
http://10.255.255.1: <urlopen error timed out>
FAIL   0  3.003s http://10.255.255.1
# HELP probe_success 1, если получен ответ с кодом 2xx/3xx, иначе 0
# TYPE probe_success gauge
probe_success{url="https://example.com"} 1
probe_success{url="https://example.com/no-such-page"} 0
probe_success{url="http://no-such-host.invalid"} 0
probe_success{url="http://10.255.255.1"} 0
# HELP probe_http_status_code Код ответа HTTP, 0 - ответа нет
# TYPE probe_http_status_code gauge
probe_http_status_code{url="https://example.com"} 200
probe_http_status_code{url="https://example.com/no-such-page"} 404
probe_http_status_code{url="http://no-such-host.invalid"} 0
probe_http_status_code{url="http://10.255.255.1"} 0
# HELP probe_duration_seconds Сколько заняла проверка
# TYPE probe_duration_seconds gauge
probe_duration_seconds{url="https://example.com"} 0.073
probe_duration_seconds{url="https://example.com/no-such-page"} 0.034
probe_duration_seconds{url="http://no-such-host.invalid"} 0.013
probe_duration_seconds{url="http://10.255.255.1"} 3.003
# HELP probe_last_run_timestamp_seconds Время последнего запуска
# TYPE probe_last_run_timestamp_seconds gauge
probe_last_run_timestamp_seconds 1792942045
```
Все хорошо, файл успешно передан и таймаут равен 3м секундам. Проверим размер
```
artem@DESKTOP-NBAE2LL:~/projects/hw_bonus$ docker images url-probe
                                                                                                                       i Info →   U  In Use
IMAGE              ID             DISK USAGE   CONTENT SIZE   EXTRA
url-probe:latest   34e098f0b9fa       89.4MB         22.4MB
```

Создал нового юзера - запускаем не от рут. Алпайн питон - 22мб вес, меньше 100. При запуске успешно передали таймаут и файл, не хардкоженны в докер файл.

## Задание 2
Ежеминутный запуск крон:
```
artem@DESKTOP-NBAE2LL:~/projects/hw_bonus$ cat Dockerfile
FROM python:3.10-alpine

RUN adduser prober -D

WORKDIR /probe-script
ADD url-probe/probe.py .
ADD crontab /etc/crontabs/root

RUN chmod 0600 /etc/crontabs/root

ENV PROBE_TIMEOUT=5

CMD ["crond", "-f", "-l", "2"]


artem@DESKTOP-NBAE2LL:~/projects/hw_bonus$ cat crontab
* * * * * su -p  -s /bin/sh prober -c "/usr/local/bin/python3 /probe-script/probe.py /probe-script/urls.txt" > /proc/1/fd/1 2> /proc/1/fd/2
```     
Сделал через задачу крон которая раннится на руте, но вызывает питон скрипт с юзера, не с рута, как по заданию. Если раннить весь кронтаск с юзера то там возникает куча проблем с пермами писать в логи.

```  
docker run -d --rm -v $PWD/url-probe/urls.txt:/probe-script/urls.txt -e PROBE_TIMEOUT=3 url-probe
d309cbf8d263865c641afd2e2cfb3d986e575911541bc89928592981e976df81
artem@DESKTOP-NBAE2LL:~/projects/hw_bonus$ docker logs d30
OK   200  0.059s https://example.com
FAIL 404  0.033s https://example.com/no-such-page
http://no-such-host.invalid: <urlopen error [Errno -2] Name does not resolve>
FAIL   0  0.012s http://no-such-host.invalid
http://10.255.255.1: <urlopen error timed out>
FAIL   0  3.003s http://10.255.255.1
OK   200  0.891s https://google.com
# HELP probe_success 1, если получен ответ с кодом 2xx/3xx, иначе 0
# TYPE probe_success gauge
probe_success{url="https://example.com"} 1
probe_success{url="https://example.com/no-such-page"} 0
probe_success{url="http://no-such-host.invalid"} 0
probe_success{url="http://10.255.255.1"} 0
probe_success{url="https://google.com"} 1
# HELP probe_http_status_code Код ответа HTTP, 0 - ответа нет
# TYPE probe_http_status_code gauge
probe_http_status_code{url="https://example.com"} 200
probe_http_status_code{url="https://example.com/no-such-page"} 404
probe_http_status_code{url="http://no-such-host.invalid"} 0
probe_http_status_code{url="http://10.255.255.1"} 0
probe_http_status_code{url="https://google.com"} 200
# HELP probe_duration_seconds Сколько заняла проверка
# TYPE probe_duration_seconds gauge
probe_duration_seconds{url="https://example.com"} 0.059
probe_duration_seconds{url="https://example.com/no-such-page"} 0.033
probe_duration_seconds{url="http://no-such-host.invalid"} 0.012
probe_duration_seconds{url="http://10.255.255.1"} 3.003
probe_duration_seconds{url="https://google.com"} 0.891
# HELP probe_last_run_timestamp_seconds Время последнего запуска
# TYPE probe_last_run_timestamp_seconds gauge
probe_last_run_timestamp_seconds 1792949164
```  
Все работает, все видно, таймаут 3, попробуем другой таймаут
``` 
artem@DESKTOP-NBAE2LL:~/projects/hw_bonus$ docker run -d --rm -v $PWD/url-probe/urls.txt:/probe-script/urls.txt -e PROBE_TIMEOUT=4 url-prob
e
9ea892313817ee29e513b5b9a8ff2ec68cf0e49968fb1f7995df3e2fdc36e965

artem@DESKTOP-NBAE2LL:~/projects/hw_bonus$ docker logs 9ea
OK   200  0.043s https://example.com
FAIL 404  0.034s https://example.com/no-such-page
http://no-such-host.invalid: <urlopen error [Errno -2] Name does not resolve>
FAIL   0  0.007s http://no-such-host.invalid
http://10.255.255.1: <urlopen error timed out>
FAIL   0  4.002s http://10.255.255.1
OK   200  1.014s https://google.com
# HELP probe_success 1, если получен ответ с кодом 2xx/3xx, иначе 0
# TYPE probe_success gauge
probe_success{url="https://example.com"} 1
probe_success{url="https://example.com/no-such-page"} 0
probe_success{url="http://no-such-host.invalid"} 0
probe_success{url="http://10.255.255.1"} 0
probe_success{url="https://google.com"} 1
# HELP probe_http_status_code Код ответа HTTP, 0 - ответа нет
# TYPE probe_http_status_code gauge
probe_http_status_code{url="https://example.com"} 200
probe_http_status_code{url="https://example.com/no-such-page"} 404
probe_http_status_code{url="http://no-such-host.invalid"} 0
probe_http_status_code{url="http://10.255.255.1"} 0
probe_http_status_code{url="https://google.com"} 200
# HELP probe_duration_seconds Сколько заняла проверка
# TYPE probe_duration_seconds gauge
probe_duration_seconds{url="https://example.com"} 0.043
probe_duration_seconds{url="https://example.com/no-such-page"} 0.034
probe_duration_seconds{url="http://no-such-host.invalid"} 0.007
probe_duration_seconds{url="http://10.255.255.1"} 4.002
probe_duration_seconds{url="https://google.com"} 1.014
# HELP probe_last_run_timestamp_seconds Время последнего запуска
# TYPE probe_last_run_timestamp_seconds gauge
probe_last_run_timestamp_seconds 1792949406
``` 
Все работает, таймаут 4 секунды. 

Одна проблема что при обновлении файла скрипт не подстраивается, потому что файл перезаписывается, а не апдейтится. Пофиксим мантированием директории на уровень выше самого файла, нашей директории с настройками. Теперь ссылка на неизменную директорию и даже если файл перезаписывается, мы просто обратимся к новому.
``` 
artem@DESKTOP-NBAE2LL:~/projects/hw_bonus$ cat crontab
* * * * * su -p  -s /bin/sh prober -c "/usr/local/bin/python3 /probe-script/probe.py /probe-script/settings/urls.txt" > /proc/1/fd/1 2> /proc/1/fd/2
artem@DESKTOP-NBAE2LL:~/projects/hw_bonus$ docker run -d --rm -v $PWD/url-probe/:/probe-script/settings -e PROBE_TIMEOUT=4 url-probe
a0ab5b1ada33e92a85becd77a0bbdfed03c14510213299aba216412a6907bcff
artem@DESKTOP-NBAE2LL:~/projects/hw_bonus$ docker logs a0a
OK   200  0.048s https://example.com
FAIL 404  0.035s https://example.com/no-such-page
http://no-such-host.invalid: <urlopen error [Errno -2] Name does not resolve>
FAIL   0  0.008s http://no-such-host.invalid
http://10.255.255.1: <urlopen error timed out>
FAIL   0  4.004s http://10.255.255.1
# HELP probe_success 1, если получен ответ с кодом 2xx/3xx, иначе 0
# TYPE probe_success gauge
probe_success{url="https://example.com"} 1
probe_success{url="https://example.com/no-such-page"} 0
probe_success{url="http://no-such-host.invalid"} 0
probe_success{url="http://10.255.255.1"} 0
# HELP probe_http_status_code Код ответа HTTP, 0 - ответа нет
# TYPE probe_http_status_code gauge
probe_http_status_code{url="https://example.com"} 200
probe_http_status_code{url="https://example.com/no-such-page"} 404
probe_http_status_code{url="http://no-such-host.invalid"} 0
probe_http_status_code{url="http://10.255.255.1"} 0
# HELP probe_duration_seconds Сколько заняла проверка
# TYPE probe_duration_seconds gauge
probe_duration_seconds{url="https://example.com"} 0.048
probe_duration_seconds{url="https://example.com/no-such-page"} 0.035
probe_duration_seconds{url="http://no-such-host.invalid"} 0.008
probe_duration_seconds{url="http://10.255.255.1"} 4.004
# HELP probe_last_run_timestamp_seconds Время последнего запуска
# TYPE probe_last_run_timestamp_seconds gauge
probe_last_run_timestamp_seconds 1792950065
artem@DESKTOP-NBAE2LL:~/projects/hw_bonus$ echo "https://google.com" >> url-probe/urls.txt
artem@DESKTOP-NBAE2LL:~/projects/hw_bonus$ docker logs a0a
OK   200  0.048s https://example.com
FAIL 404  0.035s https://example.com/no-such-page
http://no-such-host.invalid: <urlopen error [Errno -2] Name does not resolve>
FAIL   0  0.008s http://no-such-host.invalid
http://10.255.255.1: <urlopen error timed out>
FAIL   0  4.004s http://10.255.255.1
# HELP probe_success 1, если получен ответ с кодом 2xx/3xx, иначе 0
# TYPE probe_success gauge
probe_success{url="https://example.com"} 1
probe_success{url="https://example.com/no-such-page"} 0
probe_success{url="http://no-such-host.invalid"} 0
probe_success{url="http://10.255.255.1"} 0
# HELP probe_http_status_code Код ответа HTTP, 0 - ответа нет
# TYPE probe_http_status_code gauge
probe_http_status_code{url="https://example.com"} 200
probe_http_status_code{url="https://example.com/no-such-page"} 404
probe_http_status_code{url="http://no-such-host.invalid"} 0
probe_http_status_code{url="http://10.255.255.1"} 0
# HELP probe_duration_seconds Сколько заняла проверка
# TYPE probe_duration_seconds gauge
probe_duration_seconds{url="https://example.com"} 0.048
probe_duration_seconds{url="https://example.com/no-such-page"} 0.035
probe_duration_seconds{url="http://no-such-host.invalid"} 0.008
probe_duration_seconds{url="http://10.255.255.1"} 4.004
# HELP probe_last_run_timestamp_seconds Время последнего запуска
# TYPE probe_last_run_timestamp_seconds gauge
probe_last_run_timestamp_seconds 1792950065
OK   200  0.052s https://example.com
FAIL 404  0.036s https://example.com/no-such-page
http://no-such-host.invalid: <urlopen error [Errno -2] Name does not resolve>
FAIL   0  0.005s http://no-such-host.invalid
http://10.255.255.1: <urlopen error timed out>
FAIL   0  4.004s http://10.255.255.1
# HELP probe_success 1, если получен ответ с кодом 2xx/3xx, иначе 0
# TYPE probe_success gauge
probe_success{url="https://example.com"} 1
probe_success{url="https://example.com/no-such-page"} 0
probe_success{url="http://no-such-host.invalid"} 0
probe_success{url="http://10.255.255.1"} 0
probe_success{url="https://google.com"} 1
# HELP probe_http_status_code Код ответа HTTP, 0 - ответа нет
# TYPE probe_http_status_code gauge
probe_http_status_code{url="https://example.com"} 200
probe_http_status_code{url="https://example.com/no-such-page"} 404
probe_http_status_code{url="http://no-such-host.invalid"} 0
probe_http_status_code{url="http://10.255.255.1"} 0
probe_http_status_code{url="https://google.com"} 200
# HELP probe_duration_seconds Сколько заняла проверка
# TYPE probe_duration_seconds gauge
probe_duration_seconds{url="https://example.com"} 0.052
OK   200  1.038s https://google.com
probe_duration_seconds{url="https://example.com/no-such-page"} 0.036
probe_duration_seconds{url="http://no-such-host.invalid"} 0.005
probe_duration_seconds{url="http://10.255.255.1"} 4.004
probe_duration_seconds{url="https://google.com"} 1.038
# HELP probe_last_run_timestamp_seconds Время последнего запуска
# TYPE probe_last_run_timestamp_seconds gauge
probe_last_run_timestamp_seconds 1792950126
artem@DESKTOP-NBAE2LL:~/projects/hw_bonus$ docker inspect -f '{{.State.StartedAt}}' a0a
2026-10-25T17:40:50.76629747Z
artem@DESKTOP-NBAE2LL:~/projects/hw_bonus$ docker inspect a0a --format '{{.RestartCount}}'
0
``` 
Теперь все обновляется. Ура. Парсить немного сложно поэтому вот главные линии:
``` 
probe_duration_seconds{url="http://10.255.255.1"} 4.004
...
probe_last_run_timestamp_seconds 1792950065
...

probe_duration_seconds{url="http://10.255.255.1"} 4.004
probe_duration_seconds{url="https://google.com"} 1.038
...
probe_last_run_timestamp_seconds 1792950126

artem@DESKTOP-NBAE2LL:~/projects/hw_bonus$ docker inspect -f '{{.State.StartedAt}}' a0a
2026-10-25T17:40:50.76629747Z
artem@DESKTOP-NBAE2LL:~/projects/hw_bonus$ docker inspect a0a --format '{{.RestartCount}}'
0
``` 

Так ну и на последок чтобы контейнер сам перезапускался при падении поменяем флаги запуска
``` 
artem@DESKTOP-NBAE2LL:~/projects/hw_bonus$ docker run -d --restart unless-stopped -v $PWD/url-probe/:/probe-script/settings -e PROBE_TIMEOUT=4 url-probe
93fa65ba180bce84eb33e31a2b79968a610b0e9358b796cac061486860d180e4
artem@DESKTOP-NBAE2LL:~/projects/hw_bonus$ sudo systemctl restart docker
[sudo] password for artem:
artem@DESKTOP-NBAE2LL:~/projects/hw_bonus$ docker ps --filter "id=93f"
CONTAINER ID   IMAGE       COMMAND           CREATED              STATUS          PORTS     NAMES
93fa65ba180b   url-probe   "crond -f -l 2"   About a minute ago   Up 21 seconds             boring_williams
``` 
Рестарт докера пережил и так же пережил

## Задание 3
Сделаем текстфайл в контейнер и добавим наш сервис. Остальные файлы с шаблона оставил неизменными.
```
artem@DESKTOP-NBAE2LL:~/projects/hw_bonus$ cat Dockerfile
FROM python:3.10-alpine

RUN adduser prober -D
RUN mkdir /textfile
RUN chown prober:prober /textfile

artem@DESKTOP-NBAE2LL:~/projects/hw_bonus$ cat compose.yaml
...
  url-probe:
    build: .
    image: url-probe
    environment:
      - PROBE_OUTPUT_DIR=/textfile
    volumes:
      - ./url-probe:/probe-script/settings
      - textfile:/textfile
    restart: unless-stopped
...
```
Проверим что запускается
```
artem@DESKTOP-NBAE2LL:~/projects/hw_bonus$ docker compose build
[+] Building 1.3s (15/15) FINISHED
 => [internal] load local bake definitions                                                                                                                         0.0s
 => => reading from stdin 515B                                                                                                                                     0.0s
 => [internal] load build definition from Dockerfile                                                                                                               0.0s
 => => transferring dockerfile: 308B                                                                                                                               0.0s
 => [internal] load metadata for docker.io/library/python:3.10-alpine                                                                                              0.8s
 => [internal] load .dockerignore                                                                                                                                  0.0s
 => => transferring context: 2B                                                                                                                                    0.0s
 => [1/8] FROM docker.io/library/python:3.10-alpine@sha256:8ca9f9d0f4df2af674092e7e9d79ce7985e783cecfe070f8ea850801b6520bf7                                        0.0s
 => => resolve docker.io/library/python:3.10-alpine@sha256:8ca9f9d0f4df2af674092e7e9d79ce7985e783cecfe070f8ea850801b6520bf7                                        0.0s
 => [internal] load build context                                                                                                                                  0.0s
 => => transferring context: 96B                                                                                                                                   0.0s
 => CACHED [2/8] RUN adduser prober -D                                                                                                                             0.0s
 => CACHED [3/8] RUN mkdir /textfile                                                                                                                               0.0s
 => CACHED [4/8] RUN chown prober:prober /textfile                                                                                                                 0.0s
 => CACHED [5/8] WORKDIR /probe-script                                                                                                                             0.0s
 => CACHED [6/8] ADD url-probe/probe.py .                                                                                                                          0.0s
 => CACHED [7/8] ADD crontab /etc/crontabs/root                                                                                                                    0.0s
 => CACHED [8/8] RUN chmod 0600 /etc/crontabs/root                                                                                                                 0.0s
 => exporting to image                                                                                                                                             0.2s
 => => exporting layers                                                                                                                                            0.0s
 => => exporting manifest sha256:e52f517657af5b43760a3578a2c5f35169c0ffc6a3d3cd68620310b0481ffcc8                                                                  0.0s
 => => exporting config sha256:6c16962c2792ba8b46e3b8b88c24c9140ca53fd52629f83e8a3b94f73de4c07d                                                                    0.0s
 => => exporting attestation manifest sha256:4fb1fcc95044337953dd677d0bc92fcfa49994a7f298f7f329b8824e2770c0f1                                                      0.1s
 => => exporting manifest list sha256:67d48becaffd10ed921e69abde8d96f8e4b712fecfc554a0896595bfe8fffc9c                                                             0.0s
 => => naming to docker.io/library/url-probe:latest                                                                                                                0.0s
 => => unpacking to docker.io/library/url-probe:latest                                                                                                             0.0s
 => resolving provenance for metadata file                                                                                                                         0.0s
[+] build 1/1
 ✔ Image url-probe Built                                                                                                                                            1.6s
artem@DESKTOP-NBAE2LL:~/projects/hw_bonus$ docker compose up -d
[+] up 4/4
 ✔ Network hw_bonus_default           Created                                                                                                                       0.0s
 ✔ Container hw_bonus-url-probe-1     Started                                                                                                                       0.5s
 ✔ Container hw_bonus-node-exporter-1 Started                                                                                                                       0.7s
 ✔ Container hw_bonus-prometheus-1    Started
```
Все запустилось. Проверю прометея по ссылке(копипаст с браузера).
```
Load time: 7ms   Result series: 5

probe_success{instance="node-exporter:9100", job="node", url="http://10.255.255.1"}	0
probe_success{instance="node-exporter:9100", job="node", url="http://no-such-host.invalid"}	0
probe_success{instance="node-exporter:9100", job="node", url="https://example.com"}	1
probe_success{instance="node-exporter:9100", job="node", url="https://example.com/no-such-page"}	0
probe_success{instance="node-exporter:9100", job="node", url="https://google.com"}	1
```

Все работает. Мы видим результаты. Переходим к алертам, 3.2.

Добавим наш вебсайт для теста и добавим его к нашим ссылкам. Добавим правило что если нет успеха минуту кинуть алерт
```
  updown-webbie:
    image: nginx
    restart: unless-stopped

artem@DESKTOP-NBAE2LL:~/projects/hw_bonus$ cat url-probe/urls.txt
...
http://updown-webbie



artem@DESKTOP-NBAE2LL:~/projects/hw_bonus$ docker run --rm -v ./prometheus:/etc/prometheus:ro --entrypoint promtool \
>   prom/prometheus:v3.5.0 check config /etc/prometheus/prometheus.yml
Checking /etc/prometheus/prometheus.yml
  SUCCESS: 1 rule files found
 SUCCESS: /etc/prometheus/prometheus.yml is valid prometheus config file syntax

 
artem@DESKTOP-NBAE2LL:~/projects/hw_bonus$ cat prometheus/rules.yml
groups:
  - name: url-probe
    rules:
      - alert: webbieDown
        expr: probe_success == 0
        for: 1m
        labels:
          severity: page
        annotations:
          summary: "Website({{ $labels.url }}) is down" # TODO: ваше правило
```
Проверим правила
```
Checking /etc/prometheus/rules.yml
  SUCCESS: 1 rules found
```

Все проходит, проверим на практике:
```
artem@DESKTOP-NBAE2LL:~/projects/hw_bonus$ docker compose build
[+] Building 1.2s (15/15) FINISHED
 => [internal] load local bake definitions                                                                                                                         0.0s
 => => reading from stdin 515B                                                                                                                                     0.0s
 => [internal] load build definition from Dockerfile                                                                                                               0.0s
 => => transferring dockerfile: 308B                                                                                                                               0.0s
 => [internal] load metadata for docker.io/library/python:3.10-alpine                                                                                              0.7s
 => [internal] load .dockerignore                                                                                                                                  0.0s
 => => transferring context: 2B                                                                                                                                    0.0s
 => [1/8] FROM docker.io/library/python:3.10-alpine@sha256:8ca9f9d0f4df2af674092e7e9d79ce7985e783cecfe070f8ea850801b6520bf7                                        0.0s
 => => resolve docker.io/library/python:3.10-alpine@sha256:8ca9f9d0f4df2af674092e7e9d79ce7985e783cecfe070f8ea850801b6520bf7                                        0.0s
 => [internal] load build context                                                                                                                                  0.0s
 => => transferring context: 96B                                                                                                                                   0.0s
 => CACHED [2/8] RUN adduser prober -D                                                                                                                             0.0s
 => CACHED [3/8] RUN mkdir /textfile                                                                                                                               0.0s
 => CACHED [4/8] RUN chown prober:prober /textfile                                                                                                                 0.0s
 => CACHED [5/8] WORKDIR /probe-script                                                                                                                             0.0s
 => CACHED [6/8] ADD url-probe/probe.py .                                                                                                                          0.0s
 => CACHED [7/8] ADD crontab /etc/crontabs/root                                                                                                                    0.0s
 => CACHED [8/8] RUN chmod 0600 /etc/crontabs/root                                                                                                                 0.0s
 => exporting to image                                                                                                                                             0.2s
 => => exporting layers                                                                                                                                            0.0s
 => => exporting manifest sha256:e52f517657af5b43760a3578a2c5f35169c0ffc6a3d3cd68620310b0481ffcc8                                                                  0.0s
 => => exporting config sha256:6c16962c2792ba8b46e3b8b88c24c9140ca53fd52629f83e8a3b94f73de4c07d                                                                    0.0s
 => => exporting attestation manifest sha256:2e036abd3acd69ab3f50f014b8ea68c9ec4d89da80d2c2111d7826b24eada409                                                      0.1s
 => => exporting manifest list sha256:85e06b26a19e53e4e56b10a1d485cd406a8cfd1af1bfc196fdef365b96b6cb8f                                                             0.0s
 => => naming to docker.io/library/url-probe:latest                                                                                                                0.0s
 => => unpacking to docker.io/library/url-probe:latest                                                                                                             0.0s
 => resolving provenance for metadata file                                                                                                                         0.0s
[+] build 1/1
 ✔ Image url-probe Built

artem@DESKTOP-NBAE2LL:~/projects/hw_bonus$ docker compose up -d
[+] up 4/4
 ✔ Container hw_bonus-updown-webbie-1 Running                                                                                                                       0.0s
 ✔ Container hw_bonus-node-exporter-1 Running                                                                                                                       0.0s
 ✔ Container hw_bonus-url-probe-1     Running                                                                                                                       0.0s
 ✔ Container hw_bonus-prometheus-1    Started
 ```
Изначальный статус
![](image1.png)
```
artem@DESKTOP-NBAE2LL:~/projects/hw_bonus$ docker compose stop updown-webbie
[+] stop 1/1
 ✔ Container hw_bonus-updown-webbie-1 Stopped
```
статус после падения
![](image2.png)
статус если еще чуть чуть подождать
![](image3.png)

```
artem@DESKTOP-NBAE2LL:~/projects/hw_bonus$ docker compose start updown-webbie
[+] start 1/1
 ✔ Container hw_bonus-updown-webbie-1 Started                                                                                                                       0.1s
```
статус после поднятия
![](image4.png)


 ### Все требования всех заданий таким выполненны.


 Вопросы:

Почему textfile collector подходит для url-probe, а сервису вроде audit-svc лучше отдавать свой /metrics? 

Потому что юрл проба это короткая программа которая не имеет стейта, отработала сбросила забыла, легче забрать нод экспортером который постоянно работает. Аудит свц имеет комплексный стейт который легче просто раздавать посмотоянно вместо того чтобы пересчитывать.

Планировщик в контейнере перестал запускать проверки, но контейнер жив. Что при этом покажут probe_success и up? Как заметить такую поломку?

Раз контейнер жив ап будет показываться как успешная. При этом результат пробы останется тем же что и был в последний раз до поломки. Это можно заметить например добавив таймстемп в текстфайл, и проверять на стейлность данных.