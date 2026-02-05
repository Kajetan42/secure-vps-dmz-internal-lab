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
