krok 1:

 C:\Users\Antek\projekt-metody\breach-checker> curl.exe http://localhost:5000/health
{"status":"ok"}

PS C:\Users\Antek\projekt-metody\breach-checker> docker logs backend
Wczytano 89 hashy z /app/data/leaked-sha1.txt
 * Serving Flask app 'app'
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:5000
 * Running on http://172.18.0.2:5000
Press CTRL+C to quit
172.18.0.1 - - [12/Nov/2026 15:02:54] "GET /health HTTP/1.1" 200 -
172.18.0.1 - - [12/Nov/2026 15:02:55] "GET /favicon.ico HTTP/1.1" 404 -
172.18.0.1 - - [12/Nov/2026 15:03:29] "GET /health HTTP/1.1" 200 -

krok 2:
PS C:\Users\Antek\projekt-metody\breach-checker> docker run --rm --network breach-net healthcheck-probe
backend: OK
PS C:\Users\Antek\projekt-metody\breach-checker> docker run --rm healthcheck-probe
backend: UNREACHABLE
PS C:\Users\Antek\projekt-metody\breach-checker> docker stop backend
backend
PS C:\Users\Antek\projekt-metody\breach-checker> docker run --rm --network breach-net healthcheck-probe
backend: UNREACHABLE
PS C:\Users\Antek\projekt-metody\breach-checker> docker start backend
backend
PS C:\Users\Antek\projekt-metody\breach-checker>

odpowiedz: Bez --network kontener trafia do domyślnej sieci bridge, która nie ma DNS-a po nazwach, więc nazwa backend nie jest tłumaczona na adres, mimo że backend działa (w innej sieci).



krok 3:
PS C:\Users\Antek\projekt-metody\breach-checker> docker exec backend sh -c "echo notatka > /tmp/notatka.txt"
PS C:\Users\Antek\projekt-metody\breach-checker> docker restart backend
backend
PS C:\Users\Antek\projekt-metody\breach-checker> docker exec backend cat /tmp/notatka.txt
notatka
PS C:\Users\Antek\projekt-metody\breach-checker> docker rm -f backend
backend
PS C:\Users\Antek\projekt-metody\breach-checker> docker run -d --name backend --network breach-net -p 5000:5000 -v ./data/leaked-sha1.txt:/app/data/leaked-sha1.txt:ro breach-checker-backend:v1

fd1aa3d178cf510a79c95086512a9499bf96ca8531be7d2fa3f0582645463bf0
PS C:\Users\Antek\projekt-metody\breach-checker> docker exec backend cat /tmp/notatka.txt
cat: /tmp/notatka.txt: No such file or directory


PS C:\Users\Antek\projekt-metody\breach-checker> docker run --rm -v breach-dane:/dane alpine sh -c "date >> /dane/historia.txt"
PS C:\Users\Antek\projekt-metody\breach-checker> docker run --rm -v breach-dane:/dane alpine sh -c "date >> /dane/historia.txt; cat /dane/historia.txt"
Thu Nov 12 15:12:11 UTC 2026
Thu Nov 12 15:12:22 UTC 2026
PS C:\Users\Antek\projekt-metody\breach-checker> docker volume ls
DRIVER    VOLUME NAME
local     breach-dane
local     dane
local     inwentaryzacjasprztuit_db_data
local     metody-wirtualizacji-projekt-main_db_data
PS C:\Users\Antek\projekt-metody\breach-checker> docker volume rm breach-dane
breach-dane