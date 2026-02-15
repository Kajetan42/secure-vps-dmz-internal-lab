# 15 luty 2026 r.

## ChatGPT API (DMZ -> Internal)

Nadszedł dzień, w którym na Internal VPS wdrożony został skrypt umożliwiający poprzez VPN'a promptowanie do ChatGPT (model 4o) przez DMZ VPS'a. Początkowa architektura zakładała wywołanie prompta z poziomu serwera Hytale, lecz niestety na aktualnym (wczesnym) etapie rozwoju tej gry, a także z powodu moich umiejętności, bardziej ich braku nie jestem w stanie stworzyć pluginu/moda dodającego nową komendą do serwera gry oraz działającego w przedstawiony przeze mnie sposób (nawet z pomocą dostępnych mi narzędzi) wliczając w to zapewnienie bezpieczeństwa jakiego oczekuję od technologii zbudowanych w tym projekcie.

(Disclaimer: Dużą rolę we wprowadzeniu API do ChataGPT miało narzędzie o tej samej nazwie (tj. ChatGPT), który pomógł mi bardzo we wdrożeniu tego rozwiązania ze względu na napotkane przeze mnie przeszkody ze zgodnością modułów Pythona, a także ze względu na mój brak znajomości tego języka programowania)

### Co utworzyłem?

1. Nowego użytkownika na Internal VPS o nazwie "api".
2. Izolowane środowisko do Pythona (venv) oraz instalacja modułów:
   - fastapi
   - uvicorn
   - openai
   - pydantic
3. Skrypt napisany w pythonie, który zostaje uruchomiony po każdym reboocie, dzięki systemd.
4. Plik konfiguracyjny, który przechowuje mój klucz API do ChatGPT, a także token potrzebny do wywołania prompta.


### Jak działa skrypt?

1. Waliduje obecność klucza API oraz tokenu w plikach konfiguracyjnych, po czym osadza je w zmiennych potrzebnych do daleszego działania skryptu.
2. Gromadzi logi w pliku "ai.log"
3. Uruchamia aplikację FastAPI wystawioną przez serwer ASGI (uvicorn) na adresie 10.10.10.1 oraz porcie 8000.
4. Udostępnia endpoint: POST /ask , który przyjmuje JSON w formacie: {"prompt": "Treść zapytania"}
5. Sprawdza nagłówek x-token przesłany przez DMZ (gdy zły zwraca 403 Forbidden, gdy dobry kontynuuje)
6. Wysyła zapytanie do modelu gpt-4o-mini poprzez oficjalne SDK OpenAI.
7. Loguje zużycie tokenów w celu monitorowania kosztów.
8. Zwraca odpowiedź w formacie JSON do DMZ.


### Dodatkowe zabezpieczenia

- port 8000 jest dostępny wyłącznie z adresu VPN 10.10.10.2 (DMZ)
- użytkownik systemowy api nie posiada dostępu do SSH
- klucz API nie znajduje się w repozytorium ani w kodzie źródłowym
- serwis jest zarządzany przez systemd


<img width="761" height="713" alt="API-before-prompt" src="https://github.com/user-attachments/assets/07e9c78e-5433-48ab-857c-cee47c6b7966" />
<br></br>
<img width="756" height="867" alt="API-after-prompt" src="https://github.com/user-attachments/assets/beeabf3e-7130-4846-9266-00e5cad38413" />
<br></br>
<img width="844" height="394" alt="api_systemd" src="https://github.com/user-attachments/assets/ae74077b-360b-4428-b457-a17c750752b4" />
<br></br>
<img width="836" height="323" alt="api_env_conf" src="https://github.com/user-attachments/assets/b6aeb387-4eba-4667-8559-8f2a343e694a" />
<br></br>
<img width="841" height="826" alt="python-api" src="https://github.com/user-attachments/assets/b4e91219-0951-4db7-b0a5-2285b0e3af01" />
<br></br>
<img width="844" height="570" alt="python-api2" src="https://github.com/user-attachments/assets/0645e170-57cb-4e69-924a-3952c339a58f" />






# 11 luty 2026 r.

## Aktualizacja kluczy SSH

Poprzedniego dnia wdrożyłem metodę uwierzytelniania wyłącznie poprzez klucze zarówno dla dmz vps'a jak i internal vps'a. Dodatkowo zmieniłem reguły na firewall'u, które zezwalają na połączenie do internala (ssh) tylko mojemu domowemu adresowi IP, natomiast logowanie SSH do DMZ, tylko i wyłącznie adresowi IP należącemu do internala. Za breaking-glass robi KVM od dostawcy VPS, w awaryjnej sytuacji posłuży jako access point do systemu DMZ.

<img width="472" height="118" alt="SSH-password-internal" src="https://github.com/user-attachments/assets/c0e498e7-744f-4c0a-bc72-d1ffa942905a" />
<br></br>
<img width="503" height="103" alt="SSH-key-internal" src="https://github.com/user-attachments/assets/5579eff6-bef5-4a7d-adbc-0d701935e1c9" />

(Powyżej dwa przykłady logowania do Internal VPS z mojej domowej sieci, pierwszy zablokowany przy próbie z hasłem, natomiast drugi prawidłowy z privatekey)
<br></br>
<br></br>
<img width="663" height="158" alt="ssh-pass-mypc-dmz" src="https://github.com/user-attachments/assets/7b3bfbe2-8d8a-4b92-858e-4ac64f9d7439" />

(Powyższy zrzut ekranu przedstawia logowanie z mojej domowej sieci na dmz, firewall zezwala tylko na IP Internal VPS'a)
<br></br>
<br></br>
<img width="435" height="66" alt="ssh-pass-internal-dmz" src="https://github.com/user-attachments/assets/2a8cad04-0f27-468b-ba04-19234cf696f9" />
<br></br>
<img width="528" height="78" alt="SSH-key-internal-dmz" src="https://github.com/user-attachments/assets/1bf292f5-8686-4390-967f-ca1e597bc7d8" />

(Powyżej dwa przykłady logowania do DMZ VPS z Internal VPS, pierwszy zablokowany przy próbie z hasłem, natomiast drugi prawidłowy z privatekey)
<br></br>
<br></br>



## Patchowanie serwera Hytale

Po reboocie serwera okazało się, że zablkowanie użytkownikowi hytale dostępu do /bin/bash było kardynalnym błędem,
który uniemożliwił uruchomienie tmux. Prawdopodobnie wcześniejsze, ręczne uruchomienie tego procesu musiało pozostać aktywne co odciągnęło moją uwagę i stworzyło wrażenie działającego poprawnie procesu.
Ciężko mi to jednoznacznie stwierdzić, gdyż cały ten projekt jest moją pierwszą taką infrastrukturą w oparciu o Linuxa, co sprawia, że ciągle doświadczam i uczę się nowych rzeczy.
Na dzień dzisiejszy jest już w porządku, serwer Hytale startuje po reboocie, natomiast blokada SSH dla użytkownika hytale jest opisana wyłącznie w plikach konfiguracyjnych.
<br></br>

# 7 luty 2026 r.

## Wazuh

Nadszedł moment na wdrożenie narzędzia do monitorowania bezpieczeństwa serwerów. W tym celu zdecydowałem się na Wazuh, czyli darmową, open-source’ową platformę służącą do wykrywania zagrożeń, korelacji zdarzeń oraz centralnej agregacji logów.
Etap implementacji Wazuh okazał się jednym z najbardziej wymagających w całym projekcie. Wynikało to z wysokiego poziomu złożoności architektury, konieczności poprawnej konfiguracji komponentów oraz problemów napotkanych podczas integracji strefy DMZ z siecią Internal.
<br></br>
Architektura wdrożenia:

<strong>DMZ VPS:</strong> Wazuh Agent

<strong>Internal VPS:</strong> Wazuh Manager, Wazuh Indexer, Wazuh Dashboard
<br></br>
W trakcie wdrożenia wystąpiły problemy z widocznością agenta po stronie managera oraz niestabilnością dashboardu. Aby je rozwiązać:
- dodałem regułę UFW zezwalającą na komunikację na porcie 1514/tcp (próby konfiguracji UDP wraz z modyfikacją plików konfiguracyjnych nie przyniosły rezultatu w wersji 4.14.2-1)
- zaktualizowałem wersję usług Wazuh na Internal VPS z 4.7.5 do 4.14.2-1, gdyż agent na DMZ VPS miał 4.14.2-1 co było przyczyną niekompatybilności
- skonfigurowałem odpowiednio agenta
- przeniosłem certyfikaty do właściwych katalogów oraz nadałem im poprawne uprawnienia.

Aktualnie Wazuh monitoruje:
- logi systemowe
- logowania przez SSH
- działania reguł zapory UFW
- reakcje i bany generowane przez fail2ban


<img width="689" height="173" alt="wazuh_agents" src="https://github.com/user-attachments/assets/42809731-69f1-4557-8a2e-0a5e556e7877" />
<br></br>
<img width="1919" height="1031" alt="wazuh_dashboard" src="https://github.com/user-attachments/assets/a0b96ed7-70c5-49d8-8673-11cd1e66253a" />

(Aby zalogować się na stronę dashboardu należy połączyć się z IP Internal VPS, z sieci domowej na porcie 443, a następnie podać dane do logowania)

<br></br>

<img width="1919" height="965" alt="wazuh-agent" src="https://github.com/user-attachments/assets/b0d5453c-4453-4ce4-a1c8-2f50f2d81a29" />



# 6 luty 2026 r.

## Aktualizacja serwera Hytale (DMZ)

W związku z potrzebą uruchamiania serwera Hytale niezależnie od mojej obecności na SSH, zdecydowałem się wykorzystać narzędzie tmux, które pozwala na zarządzanie sesjami terminala oraz utrzymywanie procesów w tle nawet po rozłączeniu SSH.

Dodatkowo, podobnie jak w przypadku VPN skonfigurowałem autostart serwera Hytale po reboocie systemu. W tym celu utworzyłem nowego użytkownika "hytale", ograniczonego wyłącznie do uruchamiania skryptu startującego serwer gry. Połączenia SSH dla tego użytkownika zostały całkowicie zablokowane.

Mechanizm działania wygląda następująco:
- usługa systemd uruchamia tmux
- tmux tworzy nową sesję, w kontekście użytkownika hytale wykonywany jest skrypt start.sh
- skrypt znajduje się w katalogu /opt/hytale/server

W trakcie konfiguracji pojawiły się drobne komplikacje związane z:
- nieprawidłowymi uprawnieniami użytkownika hytale
- dodatkowymi zabezpieczeniami sesji tego samego użytkownika, które miały zwiększyć bezpieczeństwo, ale w praktyce ograniczały poprawne działanie serwera

Finalnie zdecydowałem się uprościć to rozwiązanie i oprzeć bezpieczeństwo głównie na zablokowanym dostępie SSH dla użytkownika hytale, co pozwoliło zachować równowagę między bezpieczeństwem a stabilnym działaniem serwera.

<img width="396" height="33" alt="image" src="https://github.com/user-attachments/assets/4f763a08-2232-40bf-8bee-bdb31033f413" />
<br></br>
<img width="1204" height="256" alt="hytale_systemctl" src="https://github.com/user-attachments/assets/1c3c38a3-a6b3-446c-8cce-bb74b986a9d2" />
<br></br>
Wideo (niska jakość ze względu na limit GitHub'a): https://github.com/user-attachments/assets/672a2064-f9c9-4261-beac-1feb1d0a7782
<br></br>
<img width="941" height="648" alt="hytale_tmux" src="https://github.com/user-attachments/assets/928a24eb-4ea6-4658-b7e0-c78454dd91a3" />





# 5 luty 2026 r.

## VPN S2S
Na dzień dzisiejszy skonfigurowałem WireGuard'a (VPN site-to-site). Wybrałem go ze względu na prostotę oraz wiążące się z nią bezpieczeństwo (prosta konfiguracja, nowoczesna kryptografia). Na obu serwerach włączyłem również uruchamianie się usługi przy każdym włączeniu/restarcie systemu VPS'a, dzięki czemu komunikacja nie zostanie zakłócona (dodatkowo da mi to większe możliwości przy regułach dostępu w oparciu o zasadę least privilege).


<img width="715" height="344" alt="dmz_systemctl" src="https://github.com/user-attachments/assets/26990fd3-2012-44f4-98e3-e6354babc08d" />
<br></br>
<img width="673" height="340" alt="internal_systemctl" src="https://github.com/user-attachments/assets/5d3001f9-0e40-4d3c-8624-12604ba60582" />
<br></br>
<img width="1430" height="596" alt="wireguard" src="https://github.com/user-attachments/assets/cf06bef8-3ef2-47e5-bd67-b0209527382f" />


### Szczegóły zmian:
- skonfigurowanie VPN site-to-site
- dodanie nowej reguły do firewall'a internal vps'a (nasłuchiwanie pod vpn)
<br></br>

# Pierwotny stan projektu
DISCLAIMER: Za pierwotny stan projektu postrzegam ten, który był aktualnym w momencie tworzenia repozytorium "secure-vps-dmz-internal-lab" na GitHubie.

## DMZ

W początkowym stanie VPS'a DMZ otwarte są na nim 2 porty: 
- 22/tcp secure shell
- 5520/udp serwer do gry Hytale

Na serwerze wdrożyłem serwer do gry Hytale, po pobraniu Java 25 (Adoptium) i uwierzytelnieniu się na stronie gry (serwer gry będzie również elementem projektu)

Zabezpieczenia:
- UFW - firewall blokuje domyślnie ruch przychodzący, a zezwala na wychodzący oraz na port 22 i 5520
- Fail2Ban - z uwagi na tymczasowe zabezpieczenie łączności ssh z VPS'ami za pomocą haseł, pomimo ich skomplikowania potrzebny był program, który będzie egzekwował zasady stworzone tak, aby ograniczyć próby logowania poprzez blokowanie dostępu do secure shell'a adresom, które przekroczą w danym okresie czasu, określoną liczbę nieudanych prób



 ## Internal

W początkowym stanie internal vps'a jest na nim dostępny tylko jeden port 22/tcp dla secure shell'a


Zabezpieczenia:
- UFW - firewall blokuje domyślnie ruch przychodzący, a zezwala na wychodzący oraz na port 22
- Fail2Ban - z uwagi na tymczasowe zabezpieczenie łączności ssh z VPS'ami za pomocą haseł, pomimo ich skomplikowania potrzebny był program, który będzie egzekwował zasady stworzone tak, aby ograniczyć próby logowania poprzez blokowanie dostępu do secure shell'a adresom, które przekroczą w danym okresie czasu, określoną liczbę nieudanych prób
