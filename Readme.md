# Instrukcja instalacji prototypu SSIDL-LOINC

## Instalacja środowiska wdrożenia prototypy SSIDL-LOINC w systemie Rocky Linux 9

### Wstępne przygotowanie systemu Rocky Linux

Aktualizacja listy dostępnych pakietów oprogramowania w systemie Rocky Linux:

```bash
sudo dnf update -y
```

### Instalacja środowiska konteneryzacji Docker

Dodanie oficjalnego repozytorium pakietów oprogramowania Docker do rejestru systemu i menadżera pakietów `dnf`:

```bash
sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo    
```

Instalacja wszystkich niezbędnych pakietów oprogramowania dla uruchomienia usługi Docker:

```bash
sudo dnf -y install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Weryfikacja poprawności instalacji usługi Docker:


```bash
docker --version
```

Aktywacja automatycznego startu przy uruchamianiu systemu oraz uruchomienie usługi Docker:

```bash
sudo systemctl --now enable docker
sudo systemctl start docker
```

Dodanie możliwości zarządzania usługą Docker dla obecnie zalogowanego uzytkownika systemu Rocky Linux:

```bash
sudo usermod -a -G docker $(whoami)
```
> Przypisanie do grupy `docker` wymaga ponownego zalogowania się do systemu, aby zmiany zaczęły obowiązywać.

Weryfikacja poprawnego przypisania do grupy uzytkowników usługi Docker po ponownym zalogowaniu:

```bash
id
```
> Przykładowy wynik wykonania polecenia 
> `uid=1001(loinc) gid=1001(loinc) groups=1001(loinc),990(docker)`

### Instalacja systemu kontroli wersji Git

Aktualizacja listy dostępnych pakietów oprogramowania w systemie Rocky Linux:

```bash
sudo dnf update -y
```

Instalacja najnowszej dotępnej wersji systemu kontroli wersji Git:

```bash
sudo dnf -y install git
```

Weryfikacja poprawności zainstalowanego środowiska:

```bash
git --version
```

### Instalacja systemu bazodanowego PostgreSQL

Instalacja oficjalnego repozytorium pakietów oprogramowania PostgreSQL:

```bash
sudo dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-9-x86_64/pgdg-redhat-repo-latest.noarch.rpm
```

Wyłączenie domyślnego pakietu oprogramowania PostgreSQL 13 w systemie Rocky Linux:

```bash
sudo dnf -qy module disable postgresql
```

Instalacja pakietu oprogramowania serwera bazodanowego PostgreSQL 17:

```bash
sudo dnf install -y postgresql17-server
```

Weryfikacja poprawności instalacji oprogramowania serwera bazodanowego PostgreSQL 17:

```bash
postgresql-17-setup --version
```

Inicjalizacja bazy danych PostgreSQL:

```bash
sudo postgresql-17-setup initdb
```

Aktywacja automatycznego startu serwera bazodanowego PostgreSQL 17 przy uruchamianiu systemu:

```bash
sudo systemctl enable postgresql-17
```

> Wynik wykonanoa polecenia:
> `Created symlink /etc/systemd/system/multi-user.target.wants/postgresql-17.service → /usr/lib/systemd/system/postgresql-17.service.`

Uruchomienie serwera bazodanowego PostgreSQL 17:

```bash
sudo systemctl start postgresql-17
```

Dodanie plików wykonywalnych serwera bazodanowego PostgreSQL 17 do zmiennej środowiskowej PATH:

```bash
sudo sh -c 'if grep -q "^PATH=" /etc/environment; then \
  sed -i "s|^PATH=\"\([^\"]*\)\"|PATH=\"\1:/usr/pgsql-17/bin\"|" /etc/environment; \
else \
  echo PATH=\"$PATH:/usr/pgsql-17/bin\" >> /etc/environment; \
fi'
```

### Instalacja narzędzia Maven automatyzującego budowę oprogramowania na plaformę Java

Instalacja najnowszej dostępnej wersji pakietu opergramowania narzędzia Maven:

```bash
sudo dnf -y install maven
```

Weryfikacja poprawności zainstalowanego narzędzia Maven:

```bash
mvn --version
```

## Instalacja prototypu SSIDL

### Konfiguracja bazy danych PostgreSQL

#### Dodanie użytkownika oraz bazy danych PostgreSQL dla usługi uwierzytelniania i autoryzacji Keycloak:

Uruchmienie wiersza poleceń serwera bazodanowego:

```bash
sudo -u postgres psql
```

Utworzenie uzytkownika dla usłygi Keycloak:

```sql
CREATE USER keycloak WITH PASSWORD '551dl(Keycloack!)';
```

Utworzenie bazy danych dla usługi Keycloak:

```sql
CREATE DATABASE keycloak OWNER keycloak;
```

Wyjście z wiersza poleceń serwera bazodanowego:

```bash
exit
```

#### Dodanie użytkownika oraz bazy danych PostgreSQL dla usługi Katalog:

Uruchmienie wiersza poleceń serwera bazodanowego:

```bash
sudo -u postgres psql
```

Utworzenie uzytkownika dla usłygi Keycloak:

```sql
CREATE USER ssidl WITH PASSWORD '551dl(Prototype!)';
```

Utworzenie bazy danych dla usługi Katalog usług SSIDL:

```sql
CREATE DATABASE service_catalog OWNER ssidl;
```

Wyjście z wiersza poleceń serwera bazodanowego:

```bash
exit
```

### Przygotowanie obrazu kontenera Docker dla komponentu Serwer terminologii SSIDL

### Przygotowanie obrazu kontenera Docker dla komponentu Katalog usług SSIDL

### Uruchomienie usług prototypu SSIDL

Ustawienie katalogu roboczego:

```bash
cd /opt
```

Pobranie repozytorium z kodem źródłowym skrytów konfiguracyjnych Docker dla usług prototypu SSIDL:

```bash
sudo git clone https://github.com/SSIDL/ssidl-deployment.git
```

Zmiana uprawnień dla utworzonego katalogu ze skryptami konfiguracyjnymi:

```bash
sudo chown -R $USER:$USER /opt/ssidl-deployment
```

Wejście do katalogu z plikami konfiguracyjnymi:

```bash
cd /opt/ssidl-deployment
```

Uruchomienie konfiguracji Docker dla prototypu SSIDL:

```bash
docker compose up -d