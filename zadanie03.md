# Zadanie 3

Data: 2026-11-12
Autor: Antoni Iwiński

---

## Ćwiczenie 1: Własne etykiety i pochodzenie warstw

### Wynik `docker inspect --format '{{json .Config.Labels}}' ...`

```
{"data":"2026-11-12","maintainer":"Antoni Iwiński"}
```

### Wynik `docker history antoni-frontend:v1`

```
IMAGE          CREATED          CREATED BY                                      SIZE      COMMENT
733e71c9cd68   31 seconds ago   EXPOSE [80/tcp]                                 0B        buildkit.dockerfile.v0
<missing>      31 seconds ago   RUN /bin/sh -c htmlhint /usr/share/nginx/htm…   4.1kB     buildkit.dockerfile.v0
<missing>      32 seconds ago   RUN /bin/sh -c date > /usr/share/nginx/html/…   24.6kB    buildkit.dockerfile.v0
<missing>      32 seconds ago   COPY . /usr/share/nginx/html/ # buildkit        32.8kB    buildkit.dockerfile.v0
<missing>      40 minutes ago   RUN /bin/sh -c apk add --no-cache nodejs npm…   99.4MB    buildkit.dockerfile.v0
<missing>      40 minutes ago   LABEL maintainer=Antoni Iwiński data=2026-11…   0B        buildkit.dockerfile.v0
<missing>      6 days ago       RUN /bin/sh -c set -x     && apkArch="$(cat …   52.4MB    buildkit.dockerfile.v0
<missing>      6 days ago       ENV ACME_VERSION=0.4.1                          0B        buildkit.dockerfile.v0
<missing>      6 days ago       ENV NJS_RELEASE=1                               0B        buildkit.dockerfile.v0
<missing>      6 days ago       ENV NJS_VERSION=1.0.2                           0B        buildkit.dockerfile.v0
<missing>      6 days ago       CMD ["nginx" "-g" "daemon off;"]                0B        buildkit.dockerfile.v0
<missing>      6 days ago       STOPSIGNAL SIGQUIT                              0B        buildkit.dockerfile.v0
<missing>      6 days ago       EXPOSE map[80/tcp:{}]                           0B        buildkit.dockerfile.v0
<missing>      6 days ago       ENTRYPOINT ["/docker-entrypoint.sh"]            0B        buildkit.dockerfile.v0
<missing>      6 days ago       COPY 30-tune-worker-processes.sh /docker-ent…   16.4kB    buildkit.dockerfile.v0
<missing>      6 days ago       COPY 20-envsubst-on-templates.sh /docker-ent…   12.3kB    buildkit.dockerfile.v0
<missing>      6 days ago       COPY 15-local-resolvers.envsh /docker-entryp…   12.3kB    buildkit.dockerfile.v0
<missing>      6 days ago       COPY 10-listen-on-ipv6-by-default.sh /docker…   12.3kB    buildkit.dockerfile.v0
<missing>      6 days ago       COPY docker-entrypoint.sh / # buildkit          8.19kB    buildkit.dockerfile.v0
<missing>      6 days ago       RUN /bin/sh -c set -x     && addgroup -g 101…   5.75MB    buildkit.dockerfile.v0
<missing>      6 days ago       ENV DYNPKG_RELEASE=1                            0B        buildkit.dockerfile.v0
<missing>      6 days ago       ENV PKG_RELEASE=1                               0B        buildkit.dockerfile.v0
<missing>      6 days ago       ENV NGINX_VERSION=1.31.8                        0B        buildkit.dockerfile.v0
<missing>      6 days ago       LABEL maintainer=NGINX Docker Maintainers <d…   0B        buildkit.dockerfile.v0
<missing>      6 days ago       CMD ["/bin/sh"]                                 0B        buildkit.dockerfile.v0
<missing>      6 days ago       ADD alpine-minirootfs-3.24.4-x86_64.tar.gz /…   9.09MB    buildkit.dockerfile.v0
```

### Odpowiedź

Warstwy na dole historii pochodzą z obrazu bazowego nginx:alpine, natomiast warstwy na samej górze pochodzą z mojego pliku Dockerfile. Niektóre warstwy mają rozmiar 0B, ponieważ modyfikują jedynie metadane obrazu i konfigurację (np. instrukcje LABEL, ENV, EXPOSE, CMD), a nie dodają do niego żadnych fizycznych plików. Wynik polecenia inspect pokazuje spłaszczony stan metadanych dla końcowego obrazu – nowo zdefiniowane etykiety z mojego Dockerfile nadpisują lub przykrywają wartości maintainer dziedziczone z obrazu bazowego.

---

## Ćwiczenie 2: RUN działa raz

### build.txt z pierwszego kontenera (port 8080)

```
StatusCode        : 200
StatusDescription : OK
Content           : Thu Nov 12 11:48:37 UTC 2026

RawContent        : HTTP/1.1 200 OK
                    Connection: keep-alive
                    Accept-Ranges: bytes
                    Content-Length: 29
                    Content-Type: text/plain
                    Date: Thu, 12 Nov 2026 11:49:22 GMT
                    ETag: "6af5a795-1d"
                    Last-Modified: Thu, 12 Nov 2026 1...
Forms             : {}
Headers           : {[Connection, keep-alive], [Accept-Ranges,
                    bytes], [Content-Length, 29], [Content-Type
                    , text/plain]...}
Images            : {}
InputFields       : {}
Links             : {}
ParsedHtml        : System.__ComObject
RawContentLength  : 29
```

### build.txt z drugiego kontenera (port 8081)

```
StatusCode        : 200
StatusDescription : OK
Content           : Thu Nov 12 11:48:37 UTC 2026

RawContent        : HTTP/1.1 200 OK
                    Connection: keep-alive
                    Accept-Ranges: bytes
                    Content-Length: 29
                    Content-Type: text/plain
                    Date: Thu, 12 Nov 2026 11:50:41 GMT
                    ETag: "6af5a795-1d"
                    Last-Modified: Thu, 12 Nov 2026 1...
Forms             : {}
Headers           : {[Connection, keep-alive], [Accept-Ranges,
                    bytes], [Content-Length, 29], [Content-Type
                    , text/plain]...}
Images            : {}
InputFields       : {}
Links             : {}
ParsedHtml        : System.__ComObject
RawContentLength  : 29
```

### Wynik `docker ps`

```
CONTAINER ID   IMAGE                COMMAND                  CREATED          STATUS          PORTS                                     NAMES
31b008c8d265   antoni-frontend:v2   "/docker-entrypoint.…"   48 seconds ago   Up 47 seconds   0.0.0.0:8081->80/tcp, [::]:8081->80/tcp   test-run-2
92fda6e91001   antoni-frontend:v2   "/docker-entrypoint.…"   2 minutes ago    Up 2 minutes    0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   test-run-1
```

### Odpowiedź

Czas zapisany w pliku build.txt jest identyczny w obu kontenerach i nie jest to czas ich uruchomienia, ponieważ instrukcja RUN date wykonała się tylko raz – podczas budowania obrazu. Wynik tego polecenia (plik tekstowy z wygenerowaną wtedy datą) został zapisany i "zamrożony" jako jedna z warstw obrazu, z której teraz korzystają oba uruchomione kontenery.

---

## Ćwiczenie 3: Nadpisanie CMD

### Linia `root` z wypisanej konfiguracji

```
root   /usr/share/nginx/html;
```

(pełna konfiguracja wypisana przez `docker run --rm ... cat /etc/nginx/conf.d/default.conf` zawierała blok `location /` z tą linią)

### Odpowiedź

Ta linia konfiguracji wyjaśnia, dlaczego musimy kopiować pliki strony do katalogu /usr/share/nginx/html/ – jest to domyślna ścieżka główna (DocumentRoot) skonfigurowana w serwerze Nginx, z której serwuje on statyczne pliki klientom.

---

## Ćwiczenie 4: Diagnoza błędnego COPY

### Wynik `docker exec ... ls /usr/share/nginx`

```
index.html
script.js
style.css
```

### Wynik `docker exec ... ls /usr/share/nginx/html`

```
50x.html
build.txt
index.html
```

### Odpowiedź

Linter htmlhint sprawdził oryginalny plik index.html znajdujący się lokalnie na moim dysku hosta. Zbudowanie obrazu (build) zakończyło się bez błędu, ponieważ komenda COPY . /usr/share/nginx/ była poprawna pod względem składniowym – pliki zostały pomyślnie skopiowane do obrazu, po prostu trafiły do niewłaściwego katalogu z punktu widzenia konfiguracji serwera Nginx.

---

## Ćwiczenie 5: Usunięcie tagu a usunięcie obrazu

### `docker rmi antoni-frontend:latest`

```
Untagged: antoni-frontend:latest
```

### `docker rmi antoni-frontend:v1`

```
Untagged: antoni-frontend:v1
Deleted: sha256:8a3bb3b17fb23d12ad04fccfb9a0a7369e5a4a5589f6334962ecccc3772ae704
```

### Odpowiedź

Polecenie docker rmi faktycznie usuwa z dysku pliki obrazu (kasuje jego warstwy – komunikaty Deleted: ...) dopiero w momencie, gdy usunięty zostanie ostatni tag wskazujący na dany Image ID, a sam obraz nie jest używany przez żaden aktualnie istniejący (nawet zatrzymany) kontener. Usunięcie tylko jednego z wielu tagów (np. latest) skutkuje wyłącznie usunięciem wskaźnika (Untagged: ...), bez zwalniania miejsca na dysku.