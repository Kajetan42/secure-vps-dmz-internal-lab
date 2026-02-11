# 11 luty 2026 r.

## Aktualizacja kluczy SSH

Poprzedniego dnia wdrożyłem metodę uwierzytelniania wyłącznie poprzez klucze zarówno dla dmz vps'a jak i internal vps'a. Dodatkowo zmieniłem reguły na firewall'u, które zezwalają na połączenie do internala (ssh) tylko mojemu domowemu adresowi IP, natomiast logowanie SSH do DMZ, tylko i wyłącznie adresowi IP należącemu do internala. Za breaking-glass robi KVM od dostawcy VPS, w awaryjnej sytuacji posłuży jako access point do systemu DMZ.

(Zrzuty ekranu będą opublikowane do 12 lutego 2026 r.)

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

Więcej zrzutów ekranu zostanie dodanych w kolejnej aktualizacji changelog'a.


<img width="1919" height="1031" alt="wazuh_dashboard" src="https://github.com/user-attachments/assets/a0b96ed7-70c5-49d8-8673-11cd1e66253a" />



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
