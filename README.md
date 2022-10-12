### UWAGA: By to zadziałało trzeba się logować bezpośrednio na tego usera, np poprzez połaczenie SSH. 
Logowanie na użytkownika przez `su` lub `sudo` nie działa na niektórych dystrybucjach.

## Zapomnij o pracy z uprawnieniami root'a !!! 

**Niezbędne umiejętności:**

-   ręczna konfiguracja plików konfiguracyjnych nginx
-   ogarnianie używania modułu python venv do wirtualnych środowisk projektowych
-   podstawowe komendy linux
-   podstawowa wiedza co to jest adres IP, porty sieciowe itp

_System Linux pozwala uruchamiać aplikacje z uprawnieniami zwykłego użytkownika nasłuchujące na portach powyżej 1024 (TCP/UDP). Do wystawienia aplikacji na portach http/https (80/443) będziesz potrzebować uprawnień root do skonfigurowania apache/nginx w trybie proxy._

Zaczynamy >>>

**1. Tworzymy katalog** `mkdir -p ./.config/systemd/user/`

**2. Tworzymy plik** `<nazwa>.service`, gdzie w miejsce `<nazwa>` podajemy naszą nazwę usługi, np: `moj_pierwszy_program.service`.

**3. Wklejamy poniższą zawartość do utworzonego pliku i edytujemy pozycje:** `WorkingDirectory=/home/user/web_site` <- Podajemy pełną ścieżkę lokalizacji naszego projektu.

`Environment="PATH=/home/user/web_site/VENV/bin"` <- w miejsce `/home/user/web_site/VENV` wpisujemy lokalizację własnego wirtualnego środowiska dla instancji python naszego projektu.

W wierszy zaczynającym się `ExecStart=` w miejsce: `/home/user/web_site/.venv/bin/gunicorn` <- podajemy pełną ścieżkę do pliku `gunicorn` z naszego wirtualnego środowiska python. `--bind 127.0.0.1:5001` <- Podajemy na jakim porcie ma się uruchomić nasz projekt.

```
[Unit]
After=network.target

[Service]
WorkingDirectory=/home/user/web_site
Environment="PATH=/home/user/web_site/VENV/bin"

ExecStart=/home/user/web_site/VENV/bin/gunicorn --workers 3 --bind 127.0.0.1:5001 app:app

Restart=on-failure
ExecReload=/bin/kill -s HUP $MAINPID
KillMode=mixed
TimeoutStopSec=5

Restart=always
RestartSec=60
PrivateTmp=true

[Install]
WantedBy=default.target
```

**4. Uruchomienie**

Wydajemy komendy, gdzie w miejsce podajemy naszą nazwę usługi, np: moj_pierwszy_program.service

```
systemctl --user daemon-reload
systemctl --user enable <nazwa>
systemctl --user start <nazwa>
loginctl enable-linger $USER
```

**5. NGINX**

Przykład pliku konfiguracyjnego NGINX w trybie proxy do udostępniania aplikacji na porcie HTTP WWW (port tcp 80)

```
server {
        listen 80;
        listen [::]:80;

        root /home/user/web_site/templates/html;
        index index.html index.htm;

        server_name web_site.pl www.web_site.pl;

        location / {
                include proxy_params;
                proxy_pass http://127.0.0.1:5001;
        }
}
```


6. Apache 2 z PROXY i SSL

ToDo ...
