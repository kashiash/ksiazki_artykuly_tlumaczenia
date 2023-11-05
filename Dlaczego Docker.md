Docker dla Opornych

Słyszałeś już niejednokrotnie słowo Docker ☁️☁️☁️ Docker tu, Docker tam, Docker jest wszędzie. Nawet jeśli nie jesteś inżynierem DevOps, musisz wiedzieć, czym jest Docker, wypada coś o nim wiedzieć. 

Jeśli jesteś zainteresowany tym, czym do diabła jest Docker, jak różni się od maszyny wirtualnej i jak można skonteneryzować prostą aplikację C#, przeczytaj ten tekst do końca, a obiecuję, że niektóre wątpliwości zostaną rozwiane, dojdzie 3x wiecej innych. Jesteś gotowy? Jeśli tak, to przejdźmy od razu do sedna.

### Dlaczego Docker?

Jak często słyszałeś lub nawet sam mówiłeś coś takiego:

Działa na moim komputerze 😖

To właśnie próbuje rozwiązać Docker. Uruchamia nie tylko twoją aplikację, ale także wszystko, czego wymaga do wykonania, takie rzeczy jak biblioteki, pakiety, skrypty i tak dalej. W ten sposób, jeśli uda ci się uruchomić swoją aplikację w Dockerze, można śmiało powiedzieć, że wszyscy mogą uruchomić twoją aplikację.

![img](https://lh7-us.googleusercontent.com/WWpMayXu-7mDLF22YRxSJE5NsWZDGgM26X7hrMqpuw0El5bObV5mLzAxglCrkPcwYVKBamvacd-GxlQHwjIHKifUmagW-Yr7qzsMmV9sOC6IaXSxhbvvh3GK4JNiM21g0uropccLR-xt4ncFFOBszYo)

### Co to jest Docker?

Koncepcja Dockera jest podobna do maszyn wirtualnych. Obie można wykorzystać do uruchamiania aplikacji w izolacji od siebie. Wraz z popularnością mikrousług, Docker wyłonił się jako doskonała alternatywa dla maszyn wirtualnych. Dlaczego tak jest? Aby odpowiedzieć na to pytanie, musisz zrozumieć różnicę między nimi. Spójrz na poniższe obrazy i upewnij się, że wszystko rozumiesz.

![img](https://lh7-us.googleusercontent.com/YJxfH5Bs1V5WNblQv4AHsNp41Yencn8lDjUkfFJbZgxNCJlI-gkqkpeSLK4M3OldHRWIb3vwmClbcB4BO1lqNJe1QFDSMAFFYgO8Kpwza1JO-384RpHzOxaj8FuA3KclNZlrOgyZZfd2uQjxWQ_Vw3w)

Zaczniemy od dołu do góry. Maszyna wirtualna to emulacja komputera, która działa na twoim urządzeniu. Aby ją uruchomić, potrzebujesz hosta, czyli komputera, na którym działa maszyna wirtualna. Następnie potrzebujesz oprogramowania, które tworzy i uruchamia maszynę wirtualną - nazywa się to hipervisor. Jak widać na samym szczycie, mamy trzy aplikacje, każda z własnymi zależnościami, plikami binarnymi i bibliotekami, a każda z nich działa w osobnej maszynie wirtualnej z własnym systemem operacyjnym (OS).

Teraz przejdźmy do prawego obrazu. Już teraz można powiedzieć, że jest znacznie prostszy. Docker Daemon to coś na kształt hipervisora, oprogramowanie do uruchamiania Dockera. Dzięki Dockerowi, twoje aplikacje mogą korzystać z systemu operacyjnego hosta. Oznacza to, że programy działające w Dockerze są tak samo "natywne" jak rzeczywiste procesy natywne działające na hoście.

Aby uruchomić swój program na maszynie wirtualnej, musisz skonfigurować samą maszynę wirtualną, oddzielny system operacyjny dla niej, przypisać stały limit zasobów itp. Maszyny wirtualne są wolniejsze i nie tak lekkie jak kontenery. Z drugiej strony, z Dockerem zużywasz mniej zasobów i czasu, aby osiągnąć ten sam poziom izolacji.

Więc jeśli maszyna wirtualna to emulacja komputera, Docker jest emulacją procesu, bo przecież to wszystko, co robi - uruchamia procesy.

### Jak “Dokerzyć”?

Zanim zaczniemy z Dockerem, potrzebujemy małego programu do uruchomienia. Prosta aplikacja konsolowa będzie odpowiednia dla naszych potrzeb.

```swift
int counter = 0;

 while (true)
 {
   Console.WriteLine($"Counter {counter++}");
   Thread.Sleep(1000);
 }
```

Jak widać, nie robi wiele. Po prostu wypisuje wartość, zwiększa ją o jeden i czeka sekundę, aby kontynuować liczenie.

Zanim zaczniesz z Dockerem, ważne jest, aby nauczyć się uruchamiać swój program jako proces. Więc, jak uruchomić ten program? Mam na myśli, że prawie zawsze korzystasz z przycisku "Uruchom" w VisualStudio. Ale jak uruchomilibyśmy aplikację bez pomocy VisualStudio? Można to osiągnąć za pomocą dobrego, starego interfejsu wiersza polecenia (CLI). Otwórz konsolę i wykonaj każde polecenie kolejno:

```swift
:: zainstaluj DotNet SDK

:: upewnij się, że masz aplikację do uruchomienia na swoim komputerze ¯\_(ツ)_/¯

:: otwórz folder zawierający nasz program
:: "cd" oznacza "zmień katalog"
cd C:\<ŚcieżkaDoTwojejAplikacji>

dotnet build
dotnet run --no-build
```

I oto jest, skończyliśmy. Nasz program działa pomyślnie:

![img](https://lh7-us.googleusercontent.com/6XCzFIZPMEm4C7semrxXnBjqOXb1v4SxlsERy-Pt-u5WCWeOpDKzTqLHvBIOdwmgn_muvBfRQwuzmv--87VocAPGtzL-064Ej0bnqo3iAqsPd_9b5-Q0v_A3WvNK-XP94rQv48v9G3Qvz9tLEqcMV4w)

Na samym końcu, jak można zauważyć, wcisnąłem ctrl + C, aby zatrzymać i zamknąć ten program.

Teraz, gdy wiemy, jak uruchomić naszą aplikację tylko za pomocą poleceń konsoli, jesteśmy właściwie gotowi, aby zacząć z Dockerem. W tym celu musimy utworzyć plik z rozszerzeniem .Dockerfile i wypełnić go następującą treścią:

```swift
FROM mcr.microsoft.com/dotnet/sdk:6.0   # instalacja DotNet SDK

COPY . /app     # upewnij się, że masz aplikację do uruchomienia
WORKDIR /app    # otwórz folder zawierający nasz program
RUN dotnet build  # uruchamia się raz podczas budowy obrazu

CMD dotnet run --no-build    # uruchamia się za każdym razem przy       uruchomieniu kontenera
```

Co właśnie zrobiliśmy? 🤔

Stworzyliśmy Dockerfile, plik z zestawem instrukcji potrzebnych do uruchomienia aplikacji przy użyciu Dockera.

Przeanalizujmy ten plik linia po linii:

**FROM** – poprawny Dockerfile powinien zaczynać się od instrukcji FROM. Ustala ona obraz bazowy. Na razie nie zastanawiaj się nad tym zbytnio, wyjaśnię, czym jest obraz, później. Widziałeś, że w przypadku naszego programu, aby go uruchomić, musieliśmy zainstalować DotNet SDK, inaczej nasze polecenia dotnet nie będą dostępne. To samo dotyczy Dockera. Przed rozpoczęciem musimy zainstalować SDK, a dokładnie to robi instrukcja FROM.

**COPY** — Instrukcja COPY kopiuje całą zawartość z folderu źródłowego na Twoim komputerze do folderu docelowego w Dockerze. W naszym przypadku kopiujemy naszą aplikację do Dockera, do folderu app.

**WORKDIR** — za pomocą WORKDIR ustawiamy katalog roboczy dla wszystkich przyszłych instrukcji. Jeśli taki katalog nie istnieje, zostanie utworzony. Innymi słowy, zmienia katalog wewnątrz Dockera.

**RUN** — RUN wykonuje polecenie wewnątrz Dockera

**CMD** — CMD konfiguruje polecenie, które będzie uruchamiane za każdym razem, gdy uruchamiamy aplikację w Dockerze. Można to również zrobić na różne inne sposoby, np.

 

```swift
CMD ["dotnet", "run", " — no-build"], ENTRYPOINT ["dotnet", "run", " — no-build"]
```

Dobrze, opisaliśmy Dockerowi, jak uruchomić naszą aplikację 😏. Zanim przejdziemy dalej, poświęć trochę czasu i zbadaj różnice i podobieństwa między tym, co zrobiliśmy ręcznie, aby uruchomić aplikację, a Dockerfile. 

![img](https://lh7-us.googleusercontent.com/ZBSwi2wcS228GF2QafKIAvc8VvHrXC5Iu7vzqEE3M1soeL32bJXCQQenQ6PzqhyoqHZjoIrIWGyqXLMEf6Hz7iVpf2LT5rg1Dz06tO6-D_tC8ryECiU0b68K_HfqY1Pc2R8Z18Qms_wU2j9rSmXEAKA)

Jak widać są to podobne kroki.

Więc kontynuujmy. Tak samo jak nasz program wymagał kompilacji przed uruchomieniem, musimy również zbudować ten Dockerfile. Można to zrobić za pomocą następującego polecenia:

```swift
docker build --tag counter-image C:\<FullPathToFolderWithDockerfile>
```

Wynikiem wykonania tego polecenia będzie obraz o nazwie counter-image. Obraz to szablon do uruchomienia Twojego programu, zawiera kod źródłowy, biblioteki i zależności potrzebne do uruchomienia aplikacji. W skrócie, są to artefakty zbudowane z Twojego programu.

Możesz sprawdzić, czy budowanie zakończyło się sukcesem za pomocą następującego polecenia:

```swift
docker images
```

Powinno pojawić się coś w tym stylu:

```swift
REPOSITORY             TAG    IMAGE ID    CREATED     SIZE
counter-image           latest  2f15637dc1f6  10 minutes ago  208MB
```

Następnym krokiem będzie zbudowanie kontenera. Kontener to proces, który uruchamia Twój program. Aby to zrobić, użyj następnego polecenia, które tworzy nowy kontener na podstawie obrazu:

```swift
docker create --name counter-container counter-image
```

Kontener został utworzony, ale jeszcze nie działa. Aby uzyskać listę istniejących kontenerów (procesów), uruchom:

```swift
docker ps --all

To pokaże Ci coś podobnego do:

CONTAINER ID  IMAGE      COMMAND         CREATED     STATUS  PORTS   NAMES
b4081a6b20a2  counter-image  "/bin/sh -c 'dotnet …"  2 minutes ago  Created 
```

Teraz nadszedł czas, aby w końcu uruchomić nasz kontener:

```swift
docker start --interactive counter-container
```

Potrzebujemy flagi --interactive, aby zobaczyć wynik programu w konsoli.

Jak widać, program działa w ten sam sposób, co wcześniej:

![img](https://lh7-us.googleusercontent.com/Qkc35AndA28c9xH8W0V_9H7rNFqGQF4mf0NYo6PrSwxI5o37T4t2mdZY4dfrziZaJo4LmKFz1IwZXlTAVhFz-aqDQRKedrOCsF5cOIe2QyM_VGov-h8nC4D2M6v0gPyGSKJYOG6IolcSYBXXfN9zfJ4)

Aby zobaczyć listę tylko aktywnych kontenerów, użyj następującego polecenia:

```swift
docker ps
```

Świetnie Ci poszło, jesteś bohaterem Internetów. Nauczyłeś/aś się, jak uruchomić swoją aplikację z użyciem Dockera 😃

Następnym krokiem będzie posprzątanie wszystkiego 😒

Zatrzymaj swój kontener:

```swift
docker stop counter-container
```

Skasuj kontener

```swift
docker rm counter-container
```

A teraz skasuj obraz:

```swift
docker rmi counter-image
```

I to wszystko 😃 Nauczyliśmy się, jak uruchamiać naszą aplikację bez VisualStudio. Nauczyliśmy również Dockera, jak to zrobić. Następnie zbudowaliśmy obraz i na jego podstawie kontener. Następnie uruchomiliśmy ten kontener i posprzątaliśmy. 

### Oto lista najczęściej używanych poleceń:

Oto kilka poleceń, które uważam za przydatne do uruchamiania w PowerShell:

```swift
\# Obraz

docker images 				*# wyświetl obrazy*

docker build -t <NazwaObrazu> . 	*# zbuduj obraz w bieżącym katalogu*

docker rmi <NazwaObrazu> 		*# usuń obraz*

docker rmi $(docker images -q) --force *# usuń wszystkie obrazy*

\# Kontener

docker ps -a 				*# wyświetl kontenery*

docker ps 					*# wyświetl działające kontenery*

docker create --name <NazwaKontenera> <NazwaObrazu> *# utwórz nowy kontener*

docker start -i <NazwaKontenera> 	*# uruchom kontener*

docker stop <NazwaKontenera> 		*# zatrzymaj kontener*

docker stop $(docker ps -aq) 		*# zatrzymaj wszystkie działające kontenery*

docker rm <NazwaKontenera> 		*# usuń kontener*

docker rm -v $(docker ps -aq -f status=exited) *# usuń wszystkie zatrzymane kontenery*

docker run -it --rm <NazwaObrazu> *# utwórz i uruchom kontener, usuń go po zatrzymaniu* 

docker system prune 			*# usuń nieużywane dane*
```

Niektórzy mogą nie lubić korzystać z poleceń i woleliby interfejs graficzny. Dla takich osób istnieje Docker Desktop. Jednak czasami konieczne jest użycie CLI, zwłaszcza jeśli korzystasz z ciągłej integracji (CI pipeline).

To była pierwsza część tego artykułu. W kolejnej części omówimy docker-compose. Zrób sobie przerwę i idź napić się kawy ☕️

### docker-compose

Zróbmy nasz program nieco bardziej skomplikowanym.

Obecnie za każdym razem, gdy uruchamiamy naszą aplikację, zaczyna ona liczyć od 0. Co zrobić, jeśli chcemy kontynuować liczenie od miejsca, w którym się zatrzymaliśmy? Aby to zrobić, musimy przechowywać każdą wartość w jakimś magazynie, na przykład w Redisie.

Zaktualizujmy nasz kod odrobinę:

```swift
using StackExchange.Redis;

 int counter = GetLastValue();

 while (true)
 {
   Console.WriteLine($"Counter {counter++}");
   SaveValue(counter);
   Thread.Sleep(1000);
 }

 int GetLastValue()
 {
   var database = GetRedisDatabase();
   var value = database.StringGet("counter-value");
   return string.IsNullOrWhiteSpace(value) ? 0 : int.Parse(value);
 }



 void SaveValue(int value)
 {
   var database = GetRedisDatabase();

   database.StringSet("counter-value", value);
 }

 IDatabase GetRedisDatabase()
 {
   var redisConnectionString = Environment.GetEnvironmentVariable("RedisConnectionString") ?? "localhost";
   var connnnection = ConnectionMultiplexer.Connect(redisConnectionString);

   return connnnection.GetDatabase();
 }
```

Teraz możemy uruchomić nasz program za pomocą Docker CLI, tak jak robiliśmy to wcześniej, uruchomić Redis za pomocą Dockera, skonfigurować sieć między nimi i tak dalej. Jednak istnieje lepszy sposób. Możemy zamiast tego użyć docker-compose.

docker-compose to narzędzie do zarządzania wieloma kontenerami. Aby zacząć z nim pracować, dodaj plik docker-compose.yml o następującej zawartości:

```swift
version: '3.9'

 services:
 my-counter:
 build: .
   
 my-redis:
 image: redis
```

Więc co mamy tutaj:

version: określa specyfikację języka dla pliku compose. Różne wersje obsługują różne instrukcje.

services: ta sekcja opisuje konfigurację kontenerów.

my-counter/my-redis: niestandardowe nazwy kontenerów. Rozpoczyna sekcję z konfiguracją naszego kontenera.

build: mówi Dockerowi, który folder powinien być używany do zlokalizowania pliku Dockerfile. Zbuduje on obraz na podstawie tego pliku lub użyje już zbudowanego.

b: mówi Dockerowi, którego obrazu użyć. Obraz Redis nie jest dostępny na naszym komputerze, więc za pierwszym razem zostanie pobrany z Internetu i przechowany lokalnie.

Pamiętasz wszystkie te kroki, które wykonywaliśmy z Dockerem, aby uruchomić naszą aplikację? Z compose nie potrzebujesz ich. Wystarczy jedno polecenie:

```swift
docker-compose up -d
```

Obrazy są budowane, obrazy są pobierane, kontenery uruchamiane... i otrzymaliśmy błąd 😕. Nasza aplikacja nie może połączyć się z Redisem... To dość powszechny problem, że jeden kontener nie może uzyskać dostępu do drugiego.

Jeśli przeczytasz trochę dokumentacji, możesz znaleźć coś takiego:

*Domyślnie Compose ustawia jedną sieć dla Twojej aplikacji.*

*Każdy kontener dla usługi dołącza do domyślnej sieci i jest osiągalny zarówno dla innych kontenerów w tej sieci, jak i dla nich odnajdywalny pod nazwą hosta identyczną z nazwą kontenera.*

Oznacza to, że powinieneś użyć my-redis zamiast localhost, aby celować w kontenery między sobą.

Zatrzymajmy nasze kontenery...

```swift
docker-compose down
```

I przepiszmy plik yml. Ale tym razem przekazujemy poprawny ciąg połączenia za pomocą zmiennej środowiskowej:

```swift
version: '3.9'

 services:
  my-counter:
   build: .
   environment:
    RedisConnectionString: "my-redis"
 
  my-redis:
   image: redis
```

...i możemy ponownie uruchomić nasz compose.

Tym razem wszystko działa zgodnie z oczekiwaniami 😏

Możemy to jednak zrobić również w inny sposób:

```swift
version: '3.9'

 services:
  my-counter:
   build: .
   network_mode: host
 
  my-redis:
   image: redis
   network_mode: host
```

network_mode: host daje kontenerowi pełny dostęp do sieci hosta. Jest to szczególnie przydatne, gdy chcesz uzyskać dostęp do aplikacji na swoim komputerze z kontenera.

Przeanalizujmy jeszcze jeden plik docker-compose.yml z niektórymi powszechnymi instrukcjami, których jeszcze nie widzieliśmy:

```swift
version: "3.2"

 services:
  web:
   build: .
   ports:
    - "80:8000"
   depends_on:
    - "rabbitmq"
   restart: always

  rabbitmq:
   image: rabbitmq:3-management-alpine
   container_name: 'rabbitmq'
   ports:
    - 5672:5672
    - 15672:15672
   volumes:
    - ~/.docker-conf/rabbitmq/data/:/var/lib/rabbitmq/
    - .:/var/log/rabbitmq
```

Co tu jest nowego:

**ports:** lista portów, które zostaną odwzorowane z kontenera na zewnątrz. Pierwsza wartość opisuje dostępny port na Twoim komputerze, a druga w Twoim kontenerze. Jest to szczególnie przydatne, gdy chcesz uzyskać dostęp do kontenera z aplikacji na swoim komputerze.

**depends_on:** określa zależność między usługami. Ma wpływ na kolejność uruchamiania usług. Nie sprawia, że jeden kontener czeka, aż drugi będzie gotowy! Ludzie zawsze są w jakiś sposób zdezorientowani przez to 😒

**restart:** określa w jakich okolicznościach kontener powinien zostać uruchomiony ponownie no|always|on-failure|unless-stopped. Domyślnie jest to no.

container_name: ustawia nazwę kontenera, która ma być użyta zamiast tej używanej w usługach

**volumes:** gdzie mapujemy dziennik i dane z kontenera do naszego lokalnego folderu. Pozwala nam to przeglądać pliki bezpośrednio w ich lokalnej strukturze folderów, zamiast musieć łączyć się z kontenerem. Ponownie, pierwsza wartość w Twoim komputerze, a druga w kontenerze.

Pełna specyfikacja wszystkich poleceń może być znaleziona tutaj:

https://docs.docker.com/compose/compose-file/compose-file-v3/

Wróćmy do naszego przykładu po raz ostatni 😃. Przechowujemy wartości w Redisie, ale nawet zwykły plik tekstowy też by zadziałał. Spróbuj sam rozszerzyć ten przykład. Tym razem zrób tak, aby nasza aplikacja i Docker używały tego samego źródła z wartościami. Spróbuj użyć w tym celu *woluminów*. Kiedy skończysz, możesz zweryfikować swoje rozwiązanie z tym: https://github.com/iamprovidence/My_Little_Programs/tree/master/FunWithDocker/DockerVolume

I to był koniec drugiej części 😃. Widzę, że już zasypiasz😴. Idź napić się kolejnej kawy ☕️ Mam nadzieję, że nie masz problemów z sercem 💔

### DockerHub

DockerHub to ostatnia rzecz, której musisz się nauczyć. Podobnie jak w przypadku dzielenia się kodem za pomocą GitHub, możemy udostępniać obrazy za pomocą DockerHub.

Już widziałeś, jak korzystaliśmy z publicznie dostępnych obrazów, takich jak Redis i DotNet SDK. Jeśli chcesz udostępnić swój własny obraz innym, możesz to zrobić w następujący sposób:

1. Stwórz konto na DockerHub. https://hub.docker.com/
2. Zaloguj się na to konto.

docker login

Wyślij swój obraz:

docker push <ImageName>

I to wszystko!✨ Teraz każdy z komputerem 💻 i dostępem do Internetu 🌐 może pobrać twój program i uruchomić go lokalnie.

Możesz również pobrać dowolny obraz, który Ci się podoba z DockerHub, używając następującej komendy:

docker pull <ImageName>

### Podsumowanie

To była długa droga 😪 Ale udało ci się 🎇 Gratulacje.

Pamiętaj, że nie jesteśmy tutaj inżynierami DevOps, tylko programistami, a jeśli jesteś w stanie zrozumieć wszystko, co dzieje się tutaj, możesz uznać się za zaawansowanego użytkownika Dockera.

Omówiliśmy wszystkie ważne tematy, takie jak Dockerfile, Docker CLI, docker-compose i DockerHub. To dobry start i szczerze mówiąc, prawdopodobnie nie będziesz potrzebować nic więcej 😉.

na podstawie artykułu : 

https://medium.com/@iamprovidence/docker-for-dummies-2fcf4b8496af

przetłumaczył CzadGłupete-4