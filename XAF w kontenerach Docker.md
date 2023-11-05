# Rozwijaj rozwiązanie DevExpress XAF i Web API w kontenerach Docker

na bazie: https://medium.com/devexpress-technical/develop-a-devexpress-xaf-and-web-api-solution-in-docker-fb9dc6bcf416



Kontenery są wszędzie — czy już wdrożyłeś swoją aplikację na klaster Kubernetesa, czy nawet w środowisku chmurowym, czy używasz kontenerów jako lokalnego środowiska uruchomieniowego i deweloperskiego, trudno jest ich ignorować!"



Napisałem poniższy szczegółowy opis jako część posta dotyczącego użytkowania DevExpress XAF Web API w połączeniu z aplikacją opartą na JavaScript. Tekst ten zostanie wkrótce opublikowany. Zdecydowano się jednak zachować część związaną z Dockerem w oddzielnym przewodniku, ponieważ może być stosowana niezależnie.

Jeśli chcesz przetestować przykładową konfigurację na własnej maszynie, możesz przejść przez pierwszą część przewodnika, aby przygotować swoje środowisko, a następnie skorzystać z przygotowanego repozytorium, nie musząc samodzielnie wykonywać kroków. Z drugiej strony, poniższe instrukcje obejmują wszystkie wymagane kroki do stworzenia środowiska deweloperskiego opartego na Dockerze, dzięki czemu będziesz mógł utworzyć własny przykład i podążać za instrukcjami.

Dlaczego warto używać Dockera?
Wielu programistów może być zadowolonych z używania swojej aplikacji opartej na Cross-Platform .NET App UI (XAF) lub ich osobnego projektu usługi Web API w standardowym środowisku opartym na systemie Windows i Visual Studio. Jednak możliwość uruchamiania takiego środowiska w kontenerach ma kilka zalet:
1. Abstrahuje uruchomienie i jednoznacznie określa wymagania projektu.
2. Możliwe jest prowadzenie prac deweloperskich na dowolnej maszynie, zarówno z lokalnie skonfigurowanym kontenerem, jak i dostępem do zdalnego środowiska. Struktura, którą opiszę w tym poście, pozwala programistom korzystać z dowolnej maszyny deweloperskiej z systemem Windows, macOS lub Linux.
3. Inni programiści mogą odtworzyć wymagane środowisko jednym poleceniem.
4. Umożliwia podjęcie istotnych kroków w kierunku wdrożenia lub oczywiście ciągłej integracji i wdrażania.

Niektóre z tych kwestii są rozpoznawane na całym świecie programistycznym, na przykład w przypadku sukcesu koncepcji `Development Containers`, które są używane z narzędziami takimi jak `Visual Studio Code Dev Containers` lub `GitHub Codespaces`.

Uwaga: możesz zastąpić `Docker` innymi środowiskami, jeśli bardziej podoba Ci się koncepcja `Podman/Buildah`, lub (w zależności od Twojego środowiska) być może środowisko LXD. Niektóre inne środowiska są w pełni kompatybilne z Dockerem, inne wymagają różnych instrukcji, ale podążają za tymi samymi zasadami.

Decyzja o użyciu Dockera nie oznacza, że nigdy więcej nie będziesz mógł używać Visual Studio w tym projekcie! Na przykład Edytor Modeli w `XAF` jest dostępny tylko w Visual Studio, więc jeśli chcesz dokonać istotnych zmian w modelu, najlepiej jest otworzyć rozwiązanie w Visual Studio. Co prawda są niezależne edytory modelu, dzialjące na Mac lub Linux, ale są w fazie testowo-ćwiczebnej i ich użycie kwalifikuje sie jako skrajny masochizm, a to już trezba leczyć.

### Cele tego posta

Opisane poniżej ustawienie jest przeznaczone dla programistów! Jest to istotne, ponieważ nie będę wyjaśniać, jak adresować scenariusze wdrożenia związane z kontenerami — takie jak korzystanie z klastrów Kubernetes, monitorowanie, skalowanie itd. To są skomplikowane tematy same w sobie, ale wymagania są zupełnie inne niż podczas pracy deweloperskiej. Struktura systemu aplikacji zgodna z kontenerami stanowi zaletę w każdym przypadku, ale głównym celem tego posta są troski deweloperów.

Jeśli jesteś zainteresowany konfiguracją wdrożenia dla XAF w Dockerze i Kubernetes, zapraszam do przeczytania tego wpisu na blogu: [Deploy and scale an XAF Blazor Server app: use Azure Kubernetes Service to serve hundreds of users](link).

(ostatnio zaktualizowano obrazy XAF Docker hub do .NET 7 i DevExpress v22.2 — sprawdź to).

Jako programista będziesz w stanie uruchomić całą skoordynowaną strukturę kontenerową jednym poleceniem na końcu tego przewodnika. Foldery z kodem źródłowym zostaną zamontowane do działających kontenerów i będą śledzone pod kątem zmian, dzięki czemu możesz korzystać z dowolnego środowiska IDE lub edytora na maszynie gospodarza do pracy nad kodem źródłowym, a kontenery będą — w większości przypadków — uruchamiane ponownie i odbudowywane w razie potrzeby. Czasami będzie konieczne ręczne restartowanie kontenerów, aby uwzględnić zmiany, ale nie będziesz potrzebować Visual Studio ani nawet .NET zainstalowanego na maszynie gospodarza. Nie będziesz również potrzebować lokalnego serwera SQL Server!

Zauważ, że lokalna instalacja .NET może być przydatna do pracy z funkcjami takimi jak Intellisense w Twoim środowisku IDE. Ale to nie jest wymaganie techniczne!
### Zainstaluj Dockera
Aby śledzić instrukcje lub uruchomić przykładowy projekt w Dockerze, będziesz potrzebować działającej instalacji Dockera. Jeśli jesteś nowy w Dockerze, polecam sprawdzenie kilku praktycznych przewodników na stronie internetowej Docker. Jeśli Twoja maszyna jeszcze nie ma zainstalowanego Dockera, możesz znaleźć odpowiednie pobranie dla swojego systemu operacyjnego i użyć pakietu "Docker Desktop" do instalacji.
### Utwórz rozwiązanie

W kontekście poniższych instrukcji, szczegóły tworzenia rozwiązania nie są istotne! Możesz użyć dowolnego rozwiązania, które już masz, lub utworzyć nowe za pomocą kreatora w programie Visual Studio. Instrukcje zakładają, że masz rozwiązanie o następującej strukturze:



![Solution structure in the VS Code Explorer tool window](https://miro.medium.com/v2/1*G2uUM46x2cf7yjQzmQ4P3A.png)

Standardowa aplikacja XAF używana tutaj składa się z projektu frontendowego o nazwie XAFApp.Blazor.Server oraz drugiego projektu frontendowego o nazwie XAFApp.WebApi. Oba te projekty współdzielą moduł XAFApp.Module. W przewodniku zostaną utworzone dwa oddzielne kontenery dla tych dwóch front-endów.

Należy zauważyć, że w trakcie przewodnika zakłada się, że testowy projekt używa EF Core do swojego modelu danych. To wprowadza tylko niewielką różnicę w niektórych przykładach kodu; proces byłby w dużej mierze taki sam dla aplikacji opartej na XPO.

### Tworzenie obrazu Dockera

Teraz pora rozpocząć pracę nad tworzeniem obrazów Dockera! Obrazy Dockera są określonymi środowiskami uruchomieniowymi "w spoczynku", czyli przed ich uruchomieniem. Jeśli "uruchamiasz kontener", obraz jest używany jako szablon dla tego kontenera. Tworzenie obrazów jest zatem pierwszym ważnym krokiem, ponieważ tutaj definiujesz, co będzie zawierało twoje środowisko uruchomieniowe i jak będzie działać.

Ogólne instrukcje dotyczące uruchamiania aplikacji .NET w kontenerach Dockera są dostępne od Microsoftu. Oto punkt wejścia, jeśli jesteś zainteresowany. Istnieją także odpowiednie strony w dokumentacji DevExpress, na przykład ta informacja na temat wymaganych pakietów dla Linuxa, od zespołu Reporting. Raportowanie nie jest używane w przykładowej aplikacji, ale dodaję wymagane pakiety do pliku Dockerfile, aby zilustrować, jak to działa.

Oto plik Dockerfile na poziomie folderu z rozwiązaniem. Często można zobaczyć osobny plik Dockerfile dla każdego obrazu, ale w tym przypadku plik jest ponownie używany, ponieważ jest bardzo podobny dla projektów Blazor i WebApi.

```Dockerfile
FROM mcr.microsoft.com/dotnet/sdk:7.0

RUN apt-get update \
    && apt-get install -y wait-for-it libc6 libicu-dev libfontconfig1 libgdiplus \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /src

COPY . ./

ARG DXNUGETKEY
RUN dotnet nuget add source https://nuget.devexpress.com/api -n DXFeed --store-password-in-clear-text -u DevExpress -p $DXNUGETKEY

ARG STARTSCRIPT
ENV STARTSCRIPT $STARTSCRIPT

CMD wait-for-it -t 0 sql:1433 -- $STARTSCRIPT
```

Powyższy plik Dockerfile rozpoczyna od standardowego obrazu .NET 7 SDK. Linie `apt-get` instalują dodatkowe pakiety, które mogą być wymagane później przez DevExpress Reporting. Następnie pliki projektu są kopiowane do obrazu. Wkrótce ustawimy też tak, że żywy folder projektu będzie zamontowany do działającego kontenera. Ściśle rzecz biorąc, wymagane jest tylko jedno z tych podejść, ale nie zaszkodzi zastosować obie opcje.

Linie związane z DXNUGETKEY instalują feed NuGet dla pakietów DevExpress w obrazie. Wartość klucza musi być podana jako argument dla procesu budowania, jak zobaczysz, gdy skonfiguruję "docker compose".

Zauważ, że to podejście pozostawia tajny klucz do feedu w obrazie! To nie jest najlepsze podejście podczas wdrażania aplikacji, ale ten obraz jest przeznaczony jako środowisko deweloperskie, które nie opuszcza Twojej maszyny, i wygodnie jest mieć klucz feedu dostępny w obrazie.

### Orkiestracja wymaganych usług
Zmienna STARTSCRIPT jest przekazywana do pliku Dockerfile z zewnątrz. Jak wspomniałem wcześniej, ten plik Dockerfile będzie ponownie używany z dwoma różnymi skryptami startowymi. Oba z nich również znajdują się w folderze rozwiązania. Oto plik start-blazor.sh:

```bash
#!/bin/sh

dotnet watch run --project XAFApp.Blazor.Server
```

A oto start-webapi.sh:

```bash
#!/bin/sh

dotnet watch run --project XAFApp.WebApi
```

Znowu, te skrypty są zaprojektowane tak, aby działać jako środowisko deweloperskie. W obrazie wdrożeniowym wykonujesz polecenie `dotnet restore` tylko jako część procesu budowania, podczas gdy w tej konfiguracji jest to niejawna część procesu uruchamiania. Dla scenariusza deweloperskiego to podejście jest wygodne, ponieważ obrazy nie będą musiały być przebudowywane, jeśli będą wymagane dodatkowe paczki. Nie wywołując `dotnet restore` explicite, polecenie `run` może samo zdecydować, które projekty należy przywrócić w każdym przypadku.

Wróćmy do pliku Dockerfile: ostatnia linia używa polecenia `wait-for-it`, aby upewnić się, że prawdziwy skrypt startowy nie zostanie uruchomiony przed gotowością serwera SQL na porcie 1433. Nie martw się, że jeszcze nie mamy serwera SQL — wkrótce dodasz go do konfiguracji!

Przed rozpoczęciem orkiestracji usług startowych dla rozwiązania demonstracyjnego wymagany jest jeszcze jeden szybki krok. Dodaj plik o nazwie `.dockerignore` w folderze z rozwiązaniem XAF i umieść w nim następującą zawartość:

```plaintext
**/bin
**/obj
```

Ten plik wyklucza foldery bin i obj z kontekstu, który jest kopiowany do obrazu podczas procesu budowania. Oszczędza to trochę czasu, ponieważ te foldery mogą być dość duże, a także pomaga uniknąć zamieszania, jeśli Twoje lokalne środowisko deweloperskie nie jest podobne do środowiska Linux wewnątrz kontenera.

Powracając do tematu orkiestracji: teraz możesz utworzyć plik docker-compose.yml na najwyższym poziomie repozytorium (czyli poza folderem rozwiązania XAF). Oto jego początkowa zawartość:



```powershell
version: '3.8'

services:
  sql:
    image: mcr.microsoft.com/azure-sql-edge
    environment:
      ACCEPT_EULA: 1
      MSSQL_SA_PASSWORD: ${SQL_SA_PASSWD}
      MSSQL_TELEMETRY_ENABLED: 'FALSE'
    cap_add:
      - SYS_PTRACE
    volumes:
      - sql-data:/var/opt/mssql

  webapi:
    build:
      context: ./XAFApp
      args:
        DXNUGETKEY: ${DXNUGETKEY}
        STARTSCRIPT: '/src/start-webapi.sh'
    depends_on:
      - sql
    environment:
      SQL_DBNAME: ${SQL_DBNAME}
      SQL_SA_PASSWD: ${SQL_SA_PASSWD}
      DOTNET_WATCH_RESTART_ON_RUDE_EDIT: 1
    ports:
      - '5273:5273'
    volumes:
      - ./XAFApp:/src

  blazor:
    build:
      context: ./XAFApp
      args:
        DXNUGETKEY: ${DXNUGETKEY}
        STARTSCRIPT: '/src/start-blazor.sh'
    depends_on:
      - sql
    environment:
      SQL_DBNAME: ${SQL_DBNAME}
      SQL_SA_PASSWD: ${SQL_SA_PASSWD}
      DOTNET_WATCH_RESTART_ON_RUDE_EDIT: 1
    ports:
      - '5274:5274'
    volumes:
      - ./XAFApp:/src

volumes:
  sql-data:
```

Jeśli nie jesteś z nim obeznany, zwróć uwagę, że format pliku YAML wymaga prawidłowego wyrównania białych znaków! Ja używam do edycji tych plików Visual Studio Code, tak jak wszystkich innych źródeł, i obsługuje on YAML prawidłowo, bez żadnych specjalnych trików.

Oczywiście format pliku jest szczegółowo udokumentowany na stronie internetowej Docker, ale szybkie podsumowanie konfiguracji może być pomocne. Pierwsza usługa nosi nazwę sql i używa obrazu firmy Microsoft o nazwie azure-sql-edge do uruchomienia serwera SQL. Za pomocą zmiennej środowiskowej akceptowany jest EULA tego obrazu, a przed zaakceptowaniem warto oczywiście przeczytać to zanim to zrobisz. Dokumentacja dotycząca obrazu Docker jest dostępna [tutaj](link), a samo EULA jest dostępne do przeczytania na tej [stronie](link). W momencie pisania tego tekstu EULA zawierała następujące postanowienie:

*Możesz instalować i używać kopii oprogramowania na dowolnym urządzeniu, w tym urządzeniach współdzielonych przez strony trzecie, w celu projektowania, rozwijania, testowania i demonstracji swoich programów. Nie możesz używać oprogramowania w środowisku produkcyjnym.*



Upewnij się, że przestrzegasz warunków licencji, jeśli korzystasz z tego obrazu.

Należy zauważyć, że w tej konfiguracji używam obrazu `mcr.microsoft.com/azure-sql-edge` zamiast `mcr.microsoft.com/mssql/server`, ponieważ obsługuje on architektury procesorów inne niż amd64 i może działać natywnie na komputerach Mac oraz innych maszynach nie będących opartych na architekturze Intel/AMD.

W konfiguracji docker-compose.yml można również zobaczyć, że skonfigurowano dodatkową zdolność (linię SYS_PTRACE) oraz podłączono wolumin Dockera, aby przechowywać dane dla kontenera. O zmienną SQL_SA_PASSWD jeszcze wspomnę za chwilę.

Obie usługi xaf i blazor są bardzo podobne i obie używają Dockerfile, który stworzyłeś w poprzednim kroku. Różnią się one tylko STARTSCRIPT i używają różnych portów — to wszystko. Ta konfiguracja spowoduje bezpośrednią budowę dwóch obrazów podczas uruchamiania skonfigurowanej orkiestracji, a folder źródłowy zostanie zamontowany do działających kontenerów, aby polecenie dotnet watch w każdym skrypcie startowym mogło przebudować projekty w razie potrzeby.

Trzy zmienne środowiskowe DXNUGETKEY, SQL_DBNAME i SQL_SA_PASSWD muszą być skonfigurowane, a w przypadku nazwy bazy danych i hasła sa, muszą być wykorzystane w projektach. Podanie wartości jest łatwe, ponieważ Docker odczyta je automatycznie z lokalnego pliku .env. Utwórz ten plik na najwyższym poziomie swojego repozytorium i dodaj trzy linie, podobne do poniższych:

```swift
SQL_DBNAME=XAFDatabase
SQL_SA_PASSWD=Super7%Pwd
DXNUGETKEY=MYKEY
```

Hasło SQL SA zostało podane jako przykład. Azure SQL Edge ma określone wymagania dotyczące hasła, takie jak małe i duże litery, cyfry oraz znaki specjalne. Możesz użyć tego przykładowego hasła bezpośrednio, jeśli chcesz, ponieważ będzie ono używane tylko lokalnie w Twoim środowisku deweloperskim.

Jeśli chodzi o klucz NuGet, musisz podstawić swój własny klucz w miejscu znacznika MYKEY. Jeśli nigdy nie korzystałeś z pakietów NuGet od DevExpress, sprawdź tę stronę dokumentacji na stronie DevExpress, gdzie znajdziesz instrukcje, jak uzyskać swój klucz dostępu do feedu.

Ważne: Proszę dodaj plik .gitignore na najwyższym poziomie swojego repozytorium w tym momencie, aby upewnić się, że plik .env nie jest umieszczany w systemie kontroli wersji! Po prostu utwórz plik o nazwie .gitignore, edytuj go i dodaj do niego wpis .env. Z oczywistych powodów nigdy nie powinieneś dodawać haseł ani kluczy do systemu kontroli wersji, a plik .env będzie musiał być utworzony ponownie przez innych programistów pracujących nad tym samym repozytorium, aby dostosować go do ich własnych lokalnych konfiguracji.

Pozostaje jeszcze wykorzystanie zmiennych SQL_DBNAME i SQL_SA_PASSWD w działających projektach .NET jako części ciągu połączenia. Na pewno istnieje więcej niż jeden sposób na to, ale podejście, które wybrałem, polega na umieszczeniu w ciągu połączenia symbolu zastępczego. To ogólne podejście można wykorzystać w innych podobnych scenariuszach, gdzie aplikacja w kontenerze Dockera otrzymuje pewne parametry poprzez zmienne środowiskowe.

W każdym z projektów XAFApp.Blazor.Server i XAFApp.WebApi edytuj plik appsettings.json. W bloku ConnectionStrings (zwykle blisko początku pliku), zmień ConnectionString, aby wyglądał tak:

```json
Pooling=false;Data Source=sql;Initial Catalog=<SQL_DBNAME>;MultipleActiveResultSets=true;User ID=sa;Password=<SQL_SA_PASSWD>;
```

Wartości SQL_DBNAME i SQL_SA_PASSWD zostaną dynamicznie zastąpione przez wartości przekazane jako zmienne środowiskowe podczas uruchamiania kontenerów.

Wartość Data Source jest ustawiona na sql. Jest to nazwa usługi serwera SQL, skonfigurowana w pliku docker-compose.yml. Orkiestracja sprawia, że wirtualny serwer jest dostępny pod tą nazwą wewnątrz kontenera .NET, dzięki czemu ta nazwa rozwiązuje się do wymaganego adresu IP, aby nawiązać połączenie z usługą.

Wartość Initial Catalog jest ustawiona z zewnątrz, przy użyciu zmiennej środowiskowej SQL_DBNAME.

Wartość User ID jest ustalona na sa. Powtórzę jeszcze raz, że jest to konfiguracja deweloperska — w przypadku wdrożenia warto rozważyć użycie oddzielnego konta dostępu.

Hasło jest ponownie ustawiane z zewnątrz, przy użyciu zmiennej środowiskowej, którą już wcześniej dodano do pliku docker-compose.yml.

Aby zastąpić zmienne środowiskowe w ciągu połączenia, dodaj odwołanie do pakietu do każdego pliku projektu. Wstaw tę linię obok istniejących wpisów PackageReference:

```xml
<PackageReference Include="StringTemplate4" Version="4.0.8" />
```

Powyższa linia oznacza, że projekt korzysta z pakietu StringTemplate4 w wersji 4.0.8, który pozwoli na dynamiczną zamianę zmiennych środowiskowych w ciągu połączenia.

Teraz edytuj plik Startup.cs, raz dla projektu WebApi i raz dla projektu Blazor. Znajdź blok, w którym ciąg połączenia jest pobierany z pliku konfiguracyjnego i przekazywany do metody UseSqlServer. W projekcie WebApi są tylko dwie istotne linie:

```csharp
string connectionString = Configuration.GetConnectionString("ConnectionString");
options.UseSqlServer(connectionString);
```

Zastąp te dwie linie poniższym kodem:

```csharp
var connectionStringTemplate = new Template(Configuration.GetConnectionString("ConnectionString"));
connectionStringTemplate.Add("SQL_DBNAME", System.Environment.GetEnvironmentVariable("SQL_DBNAME"));
connectionStringTemplate.Add("SQL_SA_PASSWD", System.Environment.GetEnvironmentVariable("SQL_SA_PASSWD"));
options.UseSqlServer(connectionStringTemplate.Render());
```

Upewnij się, że rozwiązałeś typ `Template`, dodając linijkę `using Antlr4.StringTemplate;` na początku pliku, aby uniknąć błędów kompilacji. Ten kod pozwala na dynamiczne zamiany zmiennych środowiskowych w ciągu połączenia.



W projekcie Blazor kod jest bardzo podobny, ale jest przerywany przez dodatkowy blok, który obsługuje specjalny ciąg połączenia EASYTEST. W celu tego przykładu możesz zastąpić ten fragment i użyć tego samego kodu co dla projektu WebApi. Jeśli potrzebujesz konfiguracji EasyTest, oczywiście możesz edytować kod próbki zgodnie z wymaganiami.

### Konfiguracja sieci



Aby uruchomić aplikacje w kontenerach, możesz uprościć konfigurację sieci w standardowym procesie uruchamiania. W skoordynowanym scenariuszu Dockera nie ma potrzeby uruchamiania każdego kontenera osobno z punktem końcowym HTTPS. Zwykle w wdrożonym środowisku opartym na kontenerach używany jest wspólny serwer proxy, który działa w swoim własnym kontenerze, być może nginx lub traefik, lub może YARP. Wymaganie dotyczące protokołu HTTPS zostanie wówczas spełnione przez serwer proxy i nie będzie dotyczyło poszczególnych aplikacji — tak jak powinno być podczas pracy deweloperskiej, co stanowi kolejną świetną zaletę myślenia w kategoriach kontenerów!

Znowu edytuj pliki Startup.cs dla obu projektów i znajdź linie, które wywołują app.UseHttpsRedirection();. Zakomentuj je — nie będą potrzebne w tej konfiguracji. Następnie edytuj pliki Properties/launchSettings.json dla każdego projektu. Usuń wszystkie dodatkowe bloki, pozostawiając tylko jeden dla każdego projektu. Oto jak powinien wyglądać wynik dla projektu WebApi:

```swift
{
  "$schema": "http://json.schemastore.org/launchsettings.json",
  "profiles": {
    "XAFDemoApp.WebApi": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": false,
      "launchUrl": "swagger",
      "applicationUrl": "http://*:5273",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    }
  }
}
```

A dla wersji blazor :

```swift
{
  "profiles": {
    "XAFDemoApp.Blazor.Server": {
      "commandName": "Project",
      "launchBrowser": false,
      "applicationUrl": "http://*:5274",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    }
  }
}
```

Obie te wpisy używają odpowiednich numerów portów, które są również skonfigurowane w pliku docker-compose.yml (dość losowy wybór, swoją drogą, więc śmiało możesz wybrać własne!). Zmienna applicationUrl używa także symbolu * jako zamiennika dla stałej nazwy serwera. Jest to istotna zmiana w porównaniu do standardowej konfiguracji, ponieważ w domyślnej konfiguracji URL używa lokalnego adresu (localhost), co nie działałoby poprawnie w orkiestracji Dockera.

**Zdefiniuj model danych EF Core i dodaj trochę danych testowych**

Aby przetestować skonfigurowane środowisko i zobaczyć wyniki z aplikacji testowej, musisz dodać obiekt biznesowy i kilka danych testowych. Oczywiście, jeśli masz już dane, możesz z nich skorzystać! Jeśli podążasz za tym opisem z nowym projektem, znajdź folder BusinessObjects w projekcie Module i dodaj plik SaleProduct.cs:



```csharp
using DevExpress.Persistent.Base;
using DevExpress.Persistent.BaseImpl.EF;

namespace XAFApp.Module.BusinessObjects {
  [DefaultClassOptions]
  public class SaleProduct : BaseObject {
    public SaleProduct() {
    }

    public virtual string Name { get; set; }
    public virtual decimal? Price { get; set; }
  }
}
```

Zakładając, że twój projekt testowy używa EF Core, otwórz plik XAFAppDbContext.cs w tym samym folderze i dodaj linię do klasy XAFAppEFCoreDbContext w celu zarejestrowania encji:

~~~swift
```csharp
public class XAFAppEFCoreDbContext : DbContext 
{
    // ...
    public DbSet<SaleProduct> SaleProducts { get; set; }
    // ...
}
```
~~~

Nadal w projekcie Module, edytuj plik DatabaseUpdate/Updater.cs i dodaj kod z danymi testowymi do metody UpdateDatabaseAfterUpdateSchema, podobnie jak poniżej:

```csharp
public class Updater : ModuleUpdater 
{
    // ...
    public override void UpdateDatabaseAfterUpdateSchema()
    {
        base.UpdateDatabaseAfterUpdateSchema();

public override void UpdateDatabaseAfterUpdateSchema() {
  base.UpdateDatabaseAfterUpdateSchema();

    var rubberChicken = ObjectSpace.FirstOrDefault<SaleProduct>(p => p.Name == "Rubber Chicken");
    if (rubberChicken == null) {
      // we assume that the demo data doesn't exist yet
      rubberChicken = ObjectSpace.CreateObject<SaleProduct>();
      rubberChicken.Name = "Rubber Chicken";
      rubberChicken.Price = 13.99m;
      var pulley = ObjectSpace.CreateObject<SaleProduct>();
      pulley.Name = "Pulley";
      pulley.Price = 3.99m;
      var enterprise = ObjectSpace.CreateObject<SaleProduct>();
      enterprise.Name = "Starship Enterprise";
      enterprise.Price = 149999999.99m;
      var lostArk = ObjectSpace.CreateObject<SaleProduct>();
      lostArk.Name = "The Lost Ark";
      lostArk.Price = 1000000000000m;
    }

    ObjectSpace.CommitChanges();
  }
    }
    // ...
}
```

Projekt Web API używa własnego mechanizmu do określenia, które typy danych mogą być dostępne przez usługę. Aby udostępnić typ testowy, znajdź blok z wywołaniem AddXafWebApi w pliku Startup.cs. Jest tam komentarz z przykładową linią, a Ty musisz dodać swój typ danych tam za pomocą linii podobnej do tej:

```csharp
options.BusinessObject<XAFApp.Module.BusinessObjects.SaleProduct>();
```

To pozostaje tylko upewnić się, że updater zostanie wywołany w celu utworzenia przykładowych danych, a dla tego scenariusza demonstracyjnego wystarczy zrobić to z poziomu usługi Web API. Aplikacja Blazor może być używana od czasu do czasu, ale działa z tą samą wersją wszystkich struktur danych, więc nie potrzebuje uruchamiać swoich własnych aktualizacji. Edytuj plik Services/WebApiApplicationSetup.cs w projekcie WebApi i usuń komentarze przed kilkoma ostatnimi liniami. Ustanawia to obsługę zdarzenia application.DatabaseVersionMismatch, które uruchamia updater.

```csharp
public class WebApiApplicationSetup : IWebApiApplicationSetup 
{
    public void SetupApplication(AspNetCoreApplication application) 
    {
        // ...
        application.DatabaseVersionMismatch += (s, e) => 
        {
            e.Updater.Update();
            e.Handled = true;
        };
    }
}
```

Teraz nadszedł czas, aby uruchomić przykładowe środowisko! Przy użyciu Dockera jest to teraz możliwe jednym poleceniem:

```
> docker compose up --build -d
```

Polecenie automatycznie wczytuje konfigurację z pliku docker-compose.yml w bieżącym katalogu. Parametr --build nie jest teraz ściśle wymagany, ale jeśli powtarzasz polecenie w przyszłości, będzie często potrzebny, aby upewnić się, że kontenery są ponownie generowane po każdej większej zmianie, którą wprowadziłeś. Małe zmiany nie wymagają ponownego uruchomienia, więc przeważnie będziesz chciał przeprowadzić odbudowę tylko w przypadku restartu.

Parametr -d odłącza działające kontenery od konsoli. Jeśli używasz Docker Desktop, możesz łatwo uzyskać dostęp do dzienników dla każdego kontenera. W konsoli będziesz musiał użyć polecenia takiego jak docker compose logs -f xaf, aby śledzić wyjście logów z każdego kontenera. Ogólnie rzecz biorąc, będzie to konieczne tylko w przypadku wystąpienia jakichś problemów.

Jeśli aplikacja została poprawnie uruchomiona, teraz możesz uzyskać dostęp do aplikacji Blazor w przeglądarce pod adresem http://localhost:5274. Powinna wyświetlić dane demonstracyjne dla typu SaleProduct, zgodnie z oczekiwaniami.

![img](https://miro.medium.com/v2/1*Xm9LssXnAAk48dD2Ec8hVQ.png)

Usługa Web API działa na porcie 5273 i udostępnia interfejs Swagger UI pod adresem http://localhost:5273/swagger. Możesz tam zobaczyć i przetestować obsługiwane interfejsy API, co jest zgodne z oczekiwaniami w interfejsie Swagger. Wpisów punktów końcowych dla typu `SaleProduct` są dostępne, ponieważ dodałeś ten typ za pomocą wywołania `BusinessObject<>()`.



![img](https://miro.medium.com/v2/1*6EG3XrviWzBqCvr8vuVK9A.png)

Jeśli przetestujesz przykładowe wywołanie punktu końcowego OData pod adresem /api/odata/SaleProduct — po prostu kliknij przycisk "Spróbuj to!" — zobaczysz, że dane demonstracyjne są dostępne także w usłudze Web API.



Ponieważ obie kontenery używają narzędzia dotnet watch do uruchamiania swoich usług, a foldery ze źródłami są montowane do kontenerów, wszelkie zmiany w kodzie źródłowym po stronie hosta zostaną rozpoznane przez działające procesy. Następnie podejmą próby przeładowania odpowiednich plików lub nawet uruchomienia ponownie procesów, ponieważ ustawiliśmy `DOTNET_WATCH_RESTART_ON_RUDE_EDIT.`

Z uruchomionym systemem aplikacyjnym możesz przetestować to zachowanie, wprowadzając niewielką zmianę. Na przykład możesz zmienić klasę `SaleProduct`, dodając atrybut `DisplayName` do właściwości `Name`:

```swift
using System.ComponentModel;
...

[DefaultClassOptions]
public class SaleProduct : BaseObject {
  public SaleProduct() {
  }

  [DisplayName("Product Name")]
  public virtual string Name { get; set; }
  public virtual decimal? Price { get; set; }
}
```

Zapisz swoje zmiany i jeśli chcesz, sprawdź logi Dockera dla kontenera aplikacji Blazor - zobaczysz, jak przeładowanie jest realizowane. Być może będziesz musiał odświeżyć aplikację Blazor w przeglądarce, ale zmiana w nagłówku kolumny dla pola Name zostanie natychmiast rozpoznana.

### Podsumowanie
Mamy nadzieję, że przykład ten okaże się przydatny jako punkt wyjścia i demonstracja wprowadzenia nowoczesnych podejść do XAF i pokrewnych technologii.

Dla pokrewnych informacji prosimy sprawdzić następujące artykuły: [XAF Blazor](link) | [Poradniki dla początkujących](link) | [Najczęściej zadawane pytania dotyczące nowej usługi DevExpress Web API](link). Możesz uzyskać swoją darmową kopię .NET App Security Library & Web API Service tutaj: [https://www.devexpress.com/security-api-free](https://www.devexpress.com/security-api-free). Aby dowiedzieć się o zaawansowanych/płatnych funkcjach naszej usługi Web API, zapoznaj się z następującym tematem pomocy: [Uzyskaj raport z punktu końcowego kontrolera Web API](link).