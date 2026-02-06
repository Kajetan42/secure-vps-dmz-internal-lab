# 6 luty 2026 r.

## Aktualizacja serwera Hytale

W związku z potrzebą działania serwera niezależnie od mojej łączności przez SSH, zdecydowałem się wykorzystać narzędzie tmux umożliwiające zarządzanie różnymi sesjami terminala oraz utrzymywanie procesów w tle, nawet po rozłączeniu SSH.
Dodatkowo, podobnie jak w przypadku VPN'a uruchomiłem autostart serwera Hytale po reboocie poprzez stworzenie nowego usera 'hytale' ograniczonego do wykonania skryptu uruchamiającego serwer gry (połączenie ssh jest dla niego zablokowane). Usługa systemd uruchamia narzędzie tmux, które tworzy nową sesję, a następnie w kontekście użytkownika "hytale" wykonuje skrypt "start.sh" znajdujący się w folderze serwera gry (/opt/hytale/server). Po drodze były lekkie komplikacje związane ze złymi uprawnieniami usr 'hytale', z niepotrzebnymi zabezpieczeniami sesji tego samego użytkownika (miało to zwiększyć bezpieczeństwo serwera, ale w finale ograniczyło prawidłowe funkcjonowanie, więc zdecydowałem się w pełni polegać na zablokowanym połączeniu ssh)

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
