#   Emulacja działania malware – dokumentacja 

## Opis ogólny

Aplikacja składa się z modułów odpowiedzialnych za różne ich działanie. Każdy moduł ma 10% na uruchomienie.
Aplikacja została zaprojektowana z założeniem posiadania uprawnień administratora, dlatego jest uruchamiana z użyciem użytkownika root. 
Jest to przypadek kiedy dostęp do stacji został uzyskany i system został zarażony.

###	Moduł uruchomieniowy –„run”

Moduł odpowiedzialny wyłącznie za przygotowanie środowiska poprzez instalację potrzebnych programów oraz dodanie zadania do crontab.
###	Moduł główny – „main”

Moduł uruchamiający skrypty. Są one pobierane z platformy github oraz od razu uruchamiane. Każdy skrypt ma 10% szans uruchomienia
Sterowanie zostało zrealizowane poprzez flagi:
 -	DEF
 -	LOG
 -	NET
 -	MINER
 -	PORT
 -	BLOCK
 -	TEST

Każda z flag przyjmuje wartości true/false i odpowiada za możliwość uruchomienia danego modułu.
###	Moduł sieciowy – „force”

Moduł sieciowy wykonujący atak słownikowy wykorzystując protokół ftp.
Sterowanie zostało zrealizowane poprzez flagi:
-	ADDRESS - określenie specyficznego adresu ataku bądź adresu sieci. Aktualnie adresy sieci rozpoznawane są z maskami 0,8,16 i 24. 

### Moduł zabezpieczeń – „def”

Moduł odpowiedzialny za wyłączenie zapory ubuntu –„ufw”, wyczyszczenie iptables oraz wyłączenie nmi_watchdog. Zostaje również zwiększona ilość maksymalnej ilości otwartych plików do 65535. (Zachowanie modułu podobne do GitPaste-12)

###	Moduł szyfrowania – „ransom”

Moduł odpowiedzialny za szyfrowanie wszystkich plików w określonej lokalizacji. 
Sterowanie zostało zrealizowane poprzez flagi :
- 	RMKEY – wygenerowany klucz zostaje (true) / nie zostaje (false) usunięty.
- 	RMFILES – oryginalne pliki zostają usunięte (true, zostają tylko pliki zaszyfrowane) / nie zostaję usunięte (false, zostają zarówno szyfrowane jak i oryginalne pliki).
-	DIR – określa folder w którym pliki mają zostać zaszyfrowane. Pliki wyszukiwane są rekurencyjnie więc wszystkie zostaną zaszyfrowane.
Ustawiona na false, przyjmuje wartość obecnej lokalizacji.  

###	Moduł portów – „port”

Moduł odpowiedzialny za otworzenie portów 30004 oraz 30005 w trybie nasłuchiwania na połączenia. Otwiera możliwość na przykład reverse shell.

###	Moduł koparki kryptowalut –„miner”

Jako koparka kryptowalut został wybrany program xmrig z powodu braku konieczności jego instalacji. Do konfiguracji również wystarczy 1 plik w formacie json. Program został skonfigurowany, zarchiwizowany i umieszczony na platformie github.
Wykorzystany program: https://web.xmrpool.eu/xmr-monero-easy-mining-guide.html

###	Moduł logów – „logs”

Moduł odpowiedzialny za usuwanie logów systemowych  i historii wpisanych komend. Po uruchomieniu modułu zostają usunięte wszystkie logi z lokalizacji /var/log, zostaje wyczyszczona cała historia wpisywanych komend oraz zostają ustawione specjalne flagi „i”, „u”, „a” dla folderów /tmp/ i /var/tmp/. (Zachowanie modułu podobne do GitPaste-12)

###	Moduł testowy – „test”

Moduł zaczerpnięty z https://blogs.juniper.net/en-us/threat-research/gitpaste-12.
Teoretycznie moduł powinien być pierwszym krokiem do rozprzestrzeniania się GitPaste-12, jednak nie udało mi się do tego doprowadzić. Moduł zostaje jako testowy, domyślnie wyłączony z użytku.

## Ukrywanie procesów

Pożądanym zachowaniem każdego malware dla atakującego jest jego jak najmniejsza wykrywalność. Dlatego często dodatkowym modułem jaki zapewnia malware jest możliwość ukrycia swojej obecności przed standardowym użytkownikiem. Jest to osiągalne poprzez preload bibliotek związanych z wyświetlaniem aktualnych procesów. Biblioteka za to odpowiedzialna to libc. Wystarczy zmodyfikować funkcję readdir() z biblioteki libc w taki sposób by folder z nazwą PID programy który ma zostać ukryty, nie był czytany. Zmodyfikowaną bibliotekę można umieścić w zmiennej środowiskowej  LD_PRELOAD lub dodać do konfiguracji. 
Takie podejście jednak nie działa dla sysdig. Program sysdig nasłuchuje wywołań systemowych, nie czyta plików tak jak ps. Nie można go oszukać w tak prosty sposób. Jednak domyślnie sysdig nie jest zainstalowany na wszystkich dystrybucjach linuxa, więc zmniejsza to potencjalne prawdopodobieństwo wykrycia ukrytych procesów.
Źródło: https://sysdig.com/blog/hiding-linux-processes-for-fun-and-profit/ 

