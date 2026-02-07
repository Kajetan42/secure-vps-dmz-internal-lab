# Secure VPS DMZ <-> Internal Lab

Laboratorium VPS ukierunkowane na bezpieczeństwo, prezentujące architekturę z serwerem w strefie DMZ oraz serwerem wewnętrznym, z kontrolowanym dostępem pomiędzy strefami.

🚧 **Status projektu:** w trakcie realizacji, informacje o bieżących zmianach dostępne w CHANGELOG.md


## Cele projektu

Celem projektu jest przećwiczenie zagadnień związanych z bezpieczeństwem infrastruktury sieci w praktyce.
Projekt przedstawia uproszczoną architekturę infrastruktury serwerowej w formie laboratorium edukacyjnego, obejmującą:
- wydzieloną strefę DMZ dla usług publicznych
- serwer wewnętrzny o podwyższonym poziomie zaufania
- kontrolowaną komunikację pomiędzy strefami
- podstawowe mechanizmy monitorowania i zbierania logów

## Architektura

Projekt składa się z dwóch serwerów VPS:

### VPS DMZ
Serwer wystawiony na ruch publiczny, pełniący rolę strefy DMZ.

Parametry techniczne:
- vCPU: 6
- RAM: 12 GB
- Dysk: 100 GB SSD NVMe
- OS: Ubuntu 24.04 LTS


Funkcje:
- serwer gry (Hytale)
- podstawowe zabezpieczenia sieciowe
- ograniczony dostęp administracyjny
- źródło logów bezpieczeństwa

Zabezpieczenia:
- firewall (UFW)
- Fail2Ban
- SSH (docelowo: tylko klucze)
- monitoring zdarzeń systemowych
<br></br>

### VPS Internal
Serwer wewnętrzny z restrykcyjną polityką dostępu.

Parametry techniczne:
- vCPU: 6
- RAM: 12 GB
- Dysk: 100 GB SSD NVMe
- OS: Ubuntu 24.04 LTS

Funkcje:
- usługi backendowe (DB, API)
- centralny punkt monitorowania
- zarządzanie bezpieczeństwem infrastruktury

Zabezpieczenia:
- firewall (UFW)
- Fail2Ban
- SSH z ograniczonym dostępem
- planowany system SIEM / HIDS


## Segmentacja i łączność

- Serwery połączone są bezpiecznym tunelem VPN (site-to-site).
- Komunikacja pomiędzy strefą DMZ a strefą wewnętrzną jest **ściśle ograniczona** do niezbędnych usług.
- Domyślna polityka dostępu: **deny by default**.
- Ruch administracyjny jest separowany od ruchu publicznego.


## Model dostępu administracyjnego (SSH)

Docelowy model zakłada:
- brak uwierzytelniania hasłem
- dostęp oparty wyłącznie o klucze SSH
- ograniczenie źródeł dostępu
- dostępu do serwera wewnętrznego ograniczony do zdefiniowanych IP


## Monitoring i logowanie

Planowane / wdrażane elementy:
- centralizacja logów systemowych
- monitoring prób logowania (SSH)
- analiza zdarzeń generowanych przez firewall i Fail2Ban
- wdrożenie darmowego rozwiązania klasy SIEM

Celem jest symulacja podstawowych zadań realizowanych w SOC:
- detekcja zdarzeń
- analiza logów
- korelacja alertów
- reakcja na incydenty


## Technologie i narzędzia

- Linux (Ubuntu Server)
- UFW
- Fail2Ban
- SSH
- VPN
- SIEM / HIDS


## Roadmapa

- [x] Utworzenie infrastruktury VPS
- [x] Podstawowe zabezpieczenia (UFW, Fail2Ban)
- [x] Konfiguracja VPN site-to-site
- [x] Wdrożenie SIEM / IDS 
- [x] Centralizacja logów
- [ ] Wdrożenie modelu SSH opartego o klucze
- [ ] Postawienie bazy danych i API
- [ ] Testowe scenariusze ataków (bruteforce, skany)


## Informacja końcowa

Projekt ma charakter edukacyjny i demonstracyjny.  

Bieżące informacje o postępach projektu dostępne w pliku CHANGELOG.md



