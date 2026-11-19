# Zadanie 5

## Ćwiczenie 1: Healthcheck backendu

Wynik `docker compose up -d --build`:

```
[+] up 6/6
 ✔ Image breach-checker-frontend       Built               13.5s
 ✔ Image breach-checker-backend        Built               13.4s
 ✔ Network breach-checker_default      Created              0.0s
 ✔ Container breach-checker-db-1       Healthy              6.1s
 ✔ Container breach-checker-backend-1  Healthy             11.7s
 ✔ Container breach-checker-frontend-1 Started             11.8s
```

Wynik `docker compose ps`:

```
NAME                        IMAGE                     COMMAND                  SERVICE    CREATED              STATUS                        PORTS
breach-checker-backend-1    breach-checker-backend    "python app.py"          backend    About a minute ago   Up About a minute (healthy)   5000/tcp
breach-checker-db-1         postgres:16-alpine        "docker-entrypoint.s…"   db         About a minute ago   Up About a minute (healthy)   5432/tcp
breach-checker-frontend-1   breach-checker-frontend   "/docker-entrypoint.…"   frontend   About a minute ago   Up About a minute             0.0.0.0:8080->80/tcp, [::]:8080->80/tcp
```

---

## Ćwiczenie 2: Trwałość bazy

Po `INSERT`:

```
PS> '{"hash":"bc45408273d7e1914b741c3407b0b559764f7876"}' | curl.exe -X POST http://localhost:8080/api/check -H "Content-Type: application/json" -d '@-'
{"found":true}
```

Po `docker compose down` + `docker compose up -d`:

```
PS> '{"hash":"bc45408273d7e1914b741c3407b0b559764f7876"}' | curl.exe -X POST http://localhost:8080/api/check -H "Content-Type: application/json" -d '@-'
{"found":true}
```

Po `docker compose down -v` + `docker compose up -d`:

```
PS> '{"hash":"bc45408273d7e1914b741c3407b0b559764f7876"}' | curl.exe -X POST http://localhost:8080/api/check -H "Content-Type: application/json" -d '@-'
{"found":false}
```

**Odpowiedź:**

Skrót przetrwał `down`, bo to polecenie usuwa tylko kontenery i sieć, a dane bazy leżą w wolumenie `pgdata`, który zostaje. Nie przetrwał `down -v`, bo to polecenie usuwa też wolumen, czyli całą zawartość bazy.


---

## Ćwiczenie 3: Skalowanie

Wynik `docker compose ps` z trzema replikami:

```
NAME                        IMAGE                     COMMAND                  SERVICE    CREATED         STATUS                    PORTS
breach-checker-backend-1    breach-checker-backend    "python app.py"          backend    5 minutes ago   Up 26 seconds (healthy)   5000/tcp
breach-checker-backend-2    breach-checker-backend    "python app.py"          backend    3 minutes ago   Up 26 seconds (healthy)   5000/tcp
breach-checker-backend-3    breach-checker-backend    "python app.py"          backend    3 minutes ago   Up 26 seconds (healthy)   5000/tcp
breach-checker-db-1         postgres:16-alpine        "docker-entrypoint.s…"   db         5 minutes ago   Up 29 seconds (healthy)   5432/tcp
breach-checker-frontend-1   breach-checker-frontend   "/docker-entrypoint.…"   frontend   5 minutes ago   Up 29 seconds             0.0.0.0:8080->80/tcp, [::]:8080->80/tcp
```

Wynik `docker compose exec frontend nslookup backend`:

```
Server:         127.0.0.11
Address:        127.0.0.11:53

Non-authoritative answer:

Non-authoritative answer:
Name:   backend
Address: 172.18.0.5
Name:   backend
Address: 172.18.0.6
Name:   backend
Address: 172.18.0.3
```

Logi `POST /check` przed restartem frontendu (cały ruch trafił do repliki `backend-1`):

```
backend-1  | check: hash=bc454...
backend-1  | 172.18.0.4 - - [19/Nov/2026 15:22:35] "POST /check HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-1  | 172.18.0.4 - - [19/Nov/2026 15:22:35] "POST /check HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-1  | 172.18.0.4 - - [19/Nov/2026 15:22:35] "POST /check HTTP/1.1" 200 -
```

Logi `POST /check` po restarcie frontendu (cały ruch trafił do repliki `backend-2`):

```
backend-2  | check: hash=bc454...
backend-2  | 172.18.0.4 - - [19/Nov/2026 15:22:55] "POST /check HTTP/1.1" 200 -
backend-2  | check: hash=bc454...
backend-2  | 172.18.0.4 - - [19/Nov/2026 15:22:55] "POST /check HTTP/1.1" 200 -
backend-2  | check: hash=bc454...
backend-2  | 172.18.0.4 - - [19/Nov/2026 15:22:55] "POST /check HTTP/1.1" 200 -
```

**Odpowiedź:**

Przed restartem ruch szedł do jednej repliki, bo nginx tłumaczy nazwę `backend` na adres IP tylko raz, przy starcie, i potem używa tego jednego adresu. Po restarcie frontendu nginx znowu zapytał DNS, dostał inną kolejność adresów i wybrał inną replikę (`backend-2`).

Skalowanie byłoby niemożliwe z `ports: "5000:5000"`, bo port 5000 na hoście może zajmować tylko jeden kontener. Druga i trzecia replika nie mogłyby wystartować, bo ten port byłby już zajęty.
