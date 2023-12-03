#  100 Błędów w SQL Server i Jak Ich Unikać



## 1. Wprowadzenie do SQL Servera

Ten rozdział obejmuje:

- Co powinieneś wiedzieć o tej książce
- Błąd indeksu SQL Server (Błąd 0)
- Przegląd SQL Servera
- Dlaczego warto nadal zwracać uwagę na SQL Servera
- Dlaczego ważne jest prawidłowe korzystanie z SQL Servera

W ostatnich latach termin "DBA" stał się swoistym określeniem dla wszystkich, którzy pracują z bazami danych. Zawsze mam wrażenie, że to w pewien sposób szkodzi zarówno technologii, jak i specjalistom pracującym z nią. SQL Server to ogromny produkt, a złożone aplikacje danych wymagają, aby zespoły posiadały różnorodne kompetencje, aby wykorzystać jego pełną moc. Dlatego, gdy w tej książce mówimy o DBA, odnosimy się konkretnie do Administratorów Baz Danych.

Będziemy omawiać tematy istotne dla osób pełniących następujące role:

- Administratorów Baz Danych (DBAs)
- Deweloperów Baz Danych
- Inżynierów Extract, Transform & Load (ETL)
Choć może to również być interesujące dla członków następujących społeczności, jeśli mają pokrywające się odpowiedzialności:

- Architektów Baz Danych
- Deweloperów hurtowni danych
- Testerów
- Inżynierów Cyberbezpieczeństwa
- Deweloperów Business Intelligence (BI)

### 1.1 Błąd Indeksu w SQL Serverze (Błąd 0)

Współczesny ekosystem SQL Servera jest obszerny i złożony, co doskonale prowadzi do omówienia błędu 0, terminologii zapożyczonej z wirusologii, gdzie "przypadek indeksowy" (lub przypadek 0) opisuje pierwszego pacjenta, który zaraził się wirusem i zainfekował innych ludzi. W tym kontekście jest to źródło wszystkich innych błędów związanych z SQL Serverem. Ten błąd zazwyczaj nie jest popełniany przez profesjonalistów od baz danych. Zazwyczaj popełniają go Architekci Rozwiązań, Menedżerowie Dostaw, Menedżerowie Programów i Analitycy Biznesowi. Można go streścić jednym cytatem. Cytatem, którym chciałbym zarobić dolar za każdym razem, gdy go słyszę.

"Ale to tylko baza danych, prawda?"

To założenie prostoty prowadzi do niewiarygodnej liczby projektów, które nie uwzględniają zasobów potrzebnych do rozwoju odpowiedniej aplikacji warstwy danych ani wystarczająco dużo czasu na jej odpowiedni rozwój. Powoduje to, że nie poświęca się należytej uwagi operacyjnej obsłudze środowiska po zakończeniu projektu, a termin "DBA" jest używany do opisywania każdej osoby pracującej z bazami danych, bez względu na jej kompetencje.

Jak można rozwiązać te problemy? Witaj w świecie "DBA przypadkowego" - osoby, która ma ograniczoną wiedzę na temat SQL Servera, często dlatego, że jest deweloperem aplikacji, który w przeszłości stworzył kilka małych baz danych jako magazyn danych na backendzie. Nagle ta osoba jest odpowiedzialna za rozwój, optymalizację, administrację i bezpieczeństwo wszystkiego, co związane z SQL Serverem.

Czy ten scenariusz wydaje się znajomy? Jeśli tak, to zdecydowanie warto kontynuować czytanie, ponieważ ta książka będzie badać powszechne błędy popełniane przez profesjonalistów od baz danych, którzy uczą się na własnych błędach, wiele z nich wpadając w pułapkę przypadkowego DBA.

W tej książce omówimy błędy popełniane przez osoby wykonujące zadania związane z rozwojem, administracją, wysoką dostępnością (HA) i odzyskiem po katastrofie (DR), a także bezpieczeństwem.



### 1.2 Przegląd SQL Servera
SQL Server to wiodący System Zarządzania Bazami Danych Relacyjnych (RDBMS) produkowany przez firmę Microsoft. W najprostszej formie dostarcza platformy do hostowania i zarządzania bazami danych. Posiada również wiele innych funkcji, które umożliwiają zaawansowane działania, takie jak raportowanie, transformacja danych czy zarządzanie danymi głównymi.

MODEL C4

Oprócz diagramów technicznych, w tej książce będziemy również korzystać z modelu C4, tam gdzie to odpowiednie. Model C4 to zestaw standardowych diagramów architektonicznych. W swojej istocie C4 zawiera cztery standardowe diagramy, składające się z diagramu kontekstu systemu, diagramu kontenera, diagramu komponentu i diagramu kodu. Diagram kontekstu systemu znajduje się na najwyższym poziomie i ilustruje interfejsy aplikacji z użytkownikami i innymi aplikacjami. Każdy kolejny poziom wchodzi w konkretny obszar aplikacji, dostarczając coraz bardziej szczegółowych informacji. Najniższym poziomem granulacji jest diagram kodu na dole.

Model zawiera również dodatkowe diagramy, w tym diagram krajobrazu systemowego, który ilustruje, jak portfolio aplikacji współdziała. Dynamiczny diagram, który ilustruje, jak elementy statyczne współpracują w czasie rzeczywistym, tworząc funkcję. Ostatnim diagramem jest diagram wdrożenia, który ilustruje platformę, na której aplikacja jest wdrożona.

*PORADA*

*Chociaż pełna dyskusja na temat modelu C4 wykracza poza zakres tej książki, pełny opis można znaleźć na stronie c4model.com.*

Aby lepiej zrozumieć zakres funkcji SQL Servera, wyobraźmy sobie scenariusz użycia firmy cukierniczej o nazwie **MagicChoc**. Posiadają oni stronę internetową, hostowaną w chmurze publicznej, która sprzedaje ich czekoladowe dobroci. Posiadają także niewielkie centrum danych w swojej fabryce, które obsługuje aplikację produkcyjną, aplikację do kontroli zapasów oraz rozwiązanie do raportowania. Ilustruje to diagram krajobrazu systemowego przedstawiony na rysunku 1.1.





![img](https://drek4537l1klr.cloudfront.net/carter/v-1/Figures/01__image001.png)



Ciekawym aspektem tego krajobrazu systemowego jest to, że każda z czterech aplikacji zawiera komponent SQL Servera. Strona internetowa MagicChoc jest hostowana w usłudze Azure i korzysta z bazy danych Azure SQL Database do przechowywania informacji w tle. System kontroli zapasów i aplikacja produkcyjna posiadają bazy danych SQL Server, a te bazy danych są hostowane na tym samym wystąpieniu SQL Servera, które działa na wirtualnej maszynie w centrum danych. Narzędzie raportowe firmy składa się z hurtowni danych, hostowanej w centrum danych, ale wykorzystuje również usługi raportowania SQL Server (SSRS) jako interfejs użytkownika oraz usługi analizy SQL Server (SSAS) do tworzenia wielowymiarowych modeli danych. Wykorzystuje również usługi integracji SQL Server (SSIS), aby pobierać dane z innych aplikacji i przekształcać je w zdenormalizowaną strukturę zoptymalizowaną pod kątem raportowania.

Przyjrzyjmy się bliżej diagramowi kontenera, który skupia się na aplikacji raportowej. Ten diagram można znaleźć na rysunku 1.2 i pozwala nam zacząć dostrzegać szerokość stosu technologicznego SQL Servera.

![img](https://drek4537l1klr.cloudfront.net/carter/v-1/Figures/01__image002.png)

UWAGA

Diagramy C4 powinny zawsze być używane odpowiednio, a do danego scenariusza powinno się wybierać i łączyć najbardziej odpowiednie diagramy. W tym rozdziale diagram krajobrazu systemowego i diagram kontenera są najbardziej odpowiednie do zilustrowania tego scenariusza, ale będziemy korzystać z innych diagramów C4 w trakcie trwania książki.

Można zauważyć, jak aplikacja raportowa faktycznie korzysta z wielu komponentów SQL Servera, aby spełnić wymagania biznesowe, a także bazy danych, które znajdują się obok hurtowni danych i umożliwiają funkcjonowanie tych komponentów. Pełna lista głównych komponentów SQL Servera 2022 jest szczegółowo opisana w Tabeli 1.1.

Tabela 1.1 Główne Komponenty SQL Servera

| Komponent                               | Opis                                                         |
| --------------------------------------- | ------------------------------------------------------------ |
| Analysis Services (SSAS)                | Pozwala na tworzenie i hostowanie wielowymiarowych modeli danych oraz modeli danych tabularnych, które można wykorzystać do zaawansowanego raportowania. |
| Azure Connected Services                | Obsługuje ściśłą integrację między Azure a SQL Serverem hostowanym on-premises. Obejmuje możliwość łatwego integrowania instancji SQL Servera hostowanych on-premises z funkcjami Azure, takimi jak Synapse, Purview i Microsoft Defender. Dodatkowo, podczas instalacji instancji można skonfigurować prostą łączność Azure Arc. Pozwala to na jednolite spojrzenie na instalacje SQL Servera w wielu chmurach i on-premises, a także na w pełni zautomatyzowane oceny techniczne. |
| Database Engine                         | Główna usługa zarządzania bazą danych, która umożliwia tworzenie i hostowanie baz danych relacyjnych. Obejmuje komponenty drugiego poziomu systemu operacyjnego do zarządzania pamięcią i zasobami procesora. Zawiera także zabezpieczenia danych, technologie wysokiej dostępności, replikację danych, integracje z heterogenicznymi źródłami danych oraz obsługę danych półstrukturalnych, takich jak XML i JSON. |
| Data Quality Services (DQS)             | Rozwiązanie kontroli jakości danych, które dostarcza bazę wiedzy do obsługi kluczowych zadań związanych z jakością danych, takich jak standaryzacja i usuwanie duplikatów. Komponent serwera DQS zawiera funkcje i przechowywanie jakości danych, podczas gdy komponent klienta Quality Client dostarcza interfejs graficzny, który może być używany przez ekspertów dziedzinowych danych. |
| Data Virtualization with PolyBase       | Pozwala programistom używać T-SQL do zapytań zewnętrznych źródeł danych, takich jak Azure Blob, Delta Tables (domyślny format tabel w Azure Databricks), Hadoop, MongoDB, Oracle, S3 i Teradata. |
| Integration Services (SSIS)             | Zapewnia wszechstronne operacje Extract, Transform and Load (ETL). Często używane do pobierania danych z heterogenicznych źródeł, denormalizacji danych i zasilania hurtowni danych. Stosowane również do integrowania danych z zewnętrznych źródeł, takich jak usługi internetowe i strony FTP, do baz danych SQL Servera. |
| Machine Learning Services (In-Database) | Pozwala programistom używać skryptów R i Pythona wewnątrz baz danych. Skrypty te mogą być używane do przygotowywania danych do uczenia maszynowego, a także do trenowania, oceny i wdrażania modeli uczenia maszynowego. |
| Master Data Services (MDS)              | Rozwiązanie zarządzania danymi głównymi (MDM), które umożliwia stewardom danych zarządzanie danymi głównymi firmy. Stewardowie mogą tworzyć modele danych i reguły. Dane główne mogą również być eksportowane, aby można było je łatwo udostępniać zainteresowanym stronom biznesowym. |
| Reporting Services (SSRS)               | Zapewnia użytkownikom narzędzie graficznego raportowania. Raporty mogą zawierać dane tabelaryczne, a także wykresy, mapy i inne elementy graficzne. Raporty mogą być złożone, a obsługuje się parametry, zmienne, połączone raporty i buforowanie raportów. |

Jak widać, SQL Server to obszerny i złożony produkt, który może obejmować wiele tomów. Dlatego też w tej książce skoncentrujemy się głównie na silniku baz danych, choć również poruszymy kwestie integracji z chmurą i SSIS. Przejdźmy więc trochę głębiej w silnik baz danych.

#### 1.2.1 Przegląd Silnika Baz Danych
Aby wyjaśnić, czym jest Silnik Baz Danych, przeanalizujmy, jak przebiega podróż zapytania, gdy jest wykonywane przez użytkownika. Wyobraźmy sobie, że klient MagicChoc przegląda stronę internetową i postanawia przyjrzeć się bliżej tabliczce czekolady LushBar. Strona internetowa wykonuje poniższe zapytanie:

```sql
SELECT
      ProductName
    , ProductDescription AS Description
    , CASE
        WHEN StockQty >= 10
            THEN N'In Stock'
        WHEN StockQty > 0 AND StockQty < 10
            THEN CAST(StockQty AS NVARCHAR) + ' left in stock'
        ELSE 'Out of stock'
      END
    , ProductImage
FROM dbo.Products WITH (NOLOCK)
WHERE ProductID = @ProductID
```

UWAGA

Użycie NOLOCK w tym kontekście to błąd, który omówimy w Rozdziale 4.



Realizacja tego zapytania jest przedstawiona na Rysunku 1.3.

![img](https://drek4537l1klr.cloudfront.net/carter/v-1/Figures/01__image003.png)

W tym diagramie możesz zobaczyć, że użytkownik komunikuje się z SQL Serverem za pomocą protokołu warstwy aplikacji Tabular Data Streams (TDS). SQL Server odbiera te dane za pomocą SNI, które jest częścią warstwy protokołowej SQL Servera. Warstwa protokołowa przesyła następnie żądanie do Silnika Relacyjnego.

Tutaj zapytanie jest analizowane. Jeśli popełniłeś błąd w składni zapytania, to analizator to komponent, który spowoduje wystąpienie błędu. Ten proces sprawdza nie tylko poprawność składni zapytania, ale również obejmuje algebraizację; proces, który konwertuje nazwy obiektów na identyfikatory obiektów. Wynikiem tego procesu jest utworzenie bardzo znormalizowanego drzewa zapytań, które jest przekazywane do Optymalizatora Zapytań.

Optymalizator Zapytań to bardzo zaawansowany proces, który znajduje się w samym sercu SQL Servera. Różnica między SQL a wieloma innymi językami, takimi jak C czy BASIC, polega na tym, że SQL jest językiem opisowym, a nie nakazowym. W językach nakazowych programista dokładnie określa, co chce, aby język zrobił. W języku opisowym, jakim jest SQL, programista po prostu opisuje wyniki, które chce uzyskać. To Optymalizator Zapytań jest odpowiedzialny za określenie najbardziej efektywnego sposobu uzyskania pożądanego zestawu wyników. Jak można sobie wyobrazić, ma to ogromny wpływ na wydajność Silnika Baz Danych. Dlatego optymalizator korzysta z typów obiektów, typów danych i statystyk, między innymi z metadanych, aby opisać dane w kolumnach i indeksach, i ocenić koszt różnych planów. Chociaż optymalizator to niesamowity komponent, nie zastąpi on dobrze napisanego kodu. Możesz pomóc optymalizatorowi, dbając o to, aby nie wpadać w pułapki programowania w T-SQL, takie jak korzystanie z kursorów, o czym będziemy rozmawiać w rozdziałach 4 i 8. Innym błędem, który może utrudnić optymalizatorowi, jest zaniedbanie utrzymania statystyk i indeksów. Będziemy to badać odpowiednio w rozdziałach 9 i 10.



*PORADA*

*SQL Server 2022 może również korzystać z informacji zwrotnej od optymalizatora. Do tego celu wykorzystuje magazyn zapytań (Query Store) i inteligentne przetwarzanie zapytań (Intelligent Query Processing) w celu optymalizacji aspektów planu zapytania, takich jak przyznania pamięci i maksymalny stopień równoległości, w oparciu o wydajność danego zapytania w czasie. Będzie to omówione bardziej szczegółowo w Rozdziale 9.*

Po ustaleniu odpowiedniego planu, jest on przesyłany do Wykonawcy Zapytania. Ten komponent współdziała ze Składowym Silnikiem w celu odczytywania (lub zapisywania) wymaganych danych. Obejmuje to Menedżera Transakcji, który jest odpowiedzialny za zarządzanie i rozprowadzanie wyników transakcji atomowej. Nawet jeśli nasze zapytanie nie zostało uruchomione w kontekście jawnej transakcji, nadal znajdzie się w obrębie transakcji domyślnej. Wydajność twoich transakcji może być wpływana, jeśli wybrałeś suboptymalny poziom izolacji transakcji, o czym będziemy rozmawiać w rozdziale 9. Menedżer Blokad jest odpowiedzialny za blokowanie obiektów w celu zapewnienia spójności transakcyjnej. W przypadku naszego zapytania możliwe jest odczytywanie wierszy, które nigdy nie zostały zatwierdzone, ponieważ użyliśmy wskazówki zapytania NOLOCK. Omówimy to bardziej szczegółowo w rozdziale 4. Menedżer Buforów to komponent, który współdziała z danymi w pamięci podręcznej.

Trzy obszary pamięci podręcznej przedstawione na diagramie to Pamięć Cache Planu, która przechowuje złożone plany zapytań, co oznacza, że następne wywołania zapytania mogą unikać procesu optymalizacji. Pamięć Cache Logu przechowuje rekordy dziennika transakcji przed ich zapisaniem na dysk.

> *PORADA*
>
> *Rekordy dziennika zawsze są zapisywane na dysku przed zatwierdzeniem transakcji, chyba że jest używana opóźniona trwałość (Delayed Durability).*

Pamięć Buforowa **Buffer Cache** przechowuje strony danych, które zostały odczytane z dysku. Należy pamiętać, że zapytanie jest zawsze realizowane z pamięci podręcznej, a nigdy bezpośrednio z danych przechowywanych na dysku. Nawet jeśli wymagane strony danych nie znajdują się w pamięci podręcznej, zostaną wczytane z dysku do pamięci podręcznej, a następnie zapytanie zostanie zrealizowane na podstawie danych w pamięci podręcznej. Częstym błędem jest zwracanie w zapytaniu większej ilości danych, niż jest to potrzebne. Jeśli to zrobisz, pamięć podręczna bufora zapełni się szybciej, niż jest to konieczne. Spowoduje to wcześniejsze zwolnienie starszych danych z pamięci podręcznej. To z kolei może prowadzić do słabej wydajności, ponieważ dane muszą być częściej odczytywane z dysku. Omówimy to szerzej w rozdziale 4.

#### 1.2.2 Platformy Heterogeniczne
Ważne jest, aby pamiętać, że SQL Server to już nie tylko "baza danych na systemie Windows". Zamiast tego jest obsługiwany na wielu różnych platformach. Po pierwsze, rozważmy systemy operacyjne, na których jest obsługiwany SQL Server 2022. Te systemy operacyjne są przedstawione na Rysunku 1.4.

![img](https://drek4537l1klr.cloudfront.net/carter/v-1/Figures/01__image004.png)

> UWAGA
>
> Edycja SQL Server Express i Standard może również być instalowana na systemach Windows 10 i Windows 11.
>

Zauważysz, że SQL Server można instalować nie tylko na systemie Windows Server Core, który jest wersją systemu Windows obsługiwaną wyłącznie za pomocą PowerShell i nie posiada interfejsu graficznego, ale również na trzech różnych wersjach systemu Linux. Oznacza to, że specjalista od baz danych może pracować nie tylko z interfejsem graficznym, ale także z PowerShell i Bash.

Warto również omówić platformy, na których można zainstalować SQL Server 2022, co przedstawiono na Rysunku 1.5.

![img](https://drek4537l1klr.cloudfront.net/carter/v-1/Figures/01__image005.png)

Officialne wsparcie dla SQL Servera na platformie VMWare istnieje od pewnego czasu. To, co może być bardziej zaskakujące, to fakt, że najnowsze wersje SQL Servera są również obsługiwane w kontenerach. Oznacza to, że niektórzy specjaliści od baz danych mogą potrzebować zapoznać się z technologiami takimi jak Docker i Kubernetes.

> UWAGA
>
> W chwili pisania tego tekstu SQL Server jest obsługiwany tylko na kontenerach Linux. SQL Server na kontenerach Windows był dostępny w wersji Beta, ale ten program został odwołany. Nic nie stoi na przeszkodzie, abyś samodzielnie tworzył własne kontenery Windows z zainstalowanym SQL Serverem, opierając się na podstawowym obrazie kontenerowym Windows Core. Należy jednak pamiętać, że ponieważ nie jest to obsługiwane, nigdy nie powinieneś tego robić w środowisku produkcyjnym.
>

Należy również rozważyć obsługę chmury dla SQL Servera. SQL Server jest obsługiwany na maszynach wirtualnych w ramach infrastruktury jako usługi (IaaS) w chmurze. W Azure i GCP są nazywane maszynami wirtualnymi Azure, a w AWS - instancjami EC2. Istnieją także oferty platformy jako usługi (PaaS) od tych dostawców.

W AWS dostępne są obrazy maszynowe aplikacji (AMIs) z zainstalowanym SQL Serverem. W zależności od użytego AMI, możesz zakupić SQL Servera wraz z instancją EC2 na podstawie umowy licencyjnej z dostawcą usług (SPLA). W tym modelu licencja jest uwzględniona w koszcie godzinowym instancji EC2. Alternatywnie możesz używać swojej licencji (pod warunkiem, że jest to zgodne z umową licencyjną z Microsoftem).

Istnieje również opcja korzystania z RDS, czyli oferty bazy danych jako usługi (DBaaS). Podstawowy serwer i instancję SQL Servera zarządza AWS, a ty jesteś odpowiedzialny tylko za zarządzanie bazami danych hostowanymi w obrębie tego środowiska.

W Azure można tworzyć maszyny wirtualne SQL Server z obrazów maszyn wirtualnych Azure, na których zainstalowany jest SQL Server. Podobnie jak w przypadku AWS, można korzystać z modelu z uwzględnioną licencją, gdzie koszt licencji SQL Server jest uwzględniony w koszcie maszyny wirtualnej, pozwalając na płacenie zgodnie z użyciem. Można także skorzystać z modelu Azure Hybrid Benefit (AHB), który umożliwia korzystanie z istniejącej licencji SQL Server. Wreszcie, w Azure licencję HA/DR można wykorzystać do hostowania repliki SQL Servera, która jest używana wyłącznie w celach HA lub DR.

Azure oferuje także usługę Azure SQL Database, która jest ofertą DBaaS, podobną do opcji hostingu RDS w AWS. Dodatkowo dostępna jest usługa Azure SQL Managed Instance, która oferuje równowagę między maszyną wirtualną a DBaaS, umożliwiając użytkownikom zarządzanie własną instancją SQL Server, podczas gdy podstawowy system operacyjny i maszyna wirtualna są zarządzane przez Azure.

Wreszcie, Azure oferuje Azure SQL Edge, który dostarcza bazę danych Internetu Rzeczy (IoT) obejmującą przesyłanie i przetwarzanie danych.

### 1.3 Dlaczego nadal powinniśmy dbać o SQL Server
W ostatnich latach słyszałem pytania ludzi, czy w ogóle powinniśmy jeszcze zwracać uwagę na SQL Server. Zazwyczaj pojawiają się one z jednego z dwóch powodów. Pierwszy wynika z założenia, że bazy danych relacyjne nie są już tak naprawdę potrzebne, ponieważ wszystko korzysta z NoSQL.

To założenie nie jest zbyt poprawne. Owszem, NoSQL zdecydowanie wyrosło na przestrzeni ostatniej dekady i jest właściwym wyborem w wielu przypadkach związanych z analizą danych, ale prawda jest taka, że istnieje wciąż duże miejsce dla baz danych relacyjnych. W skrócie, sprowadza się to do starego porzekadła "Nie wkładaj kwadratowego kołka w okrągłą dziurę!" Po prostu zawsze należy używać odpowiedniego narzędzia do danego zadania. Próba przymuszenia zestawu danych, który naturalnie pasuje do bazy danych relacyjnej, do środowiska nierelacyjnego, jest tak samo zła, jak próba przymuszenia naturalnie nierelacyjnych danych do bazy danych SQL Server.

Drugi powód to założenie, że DBaaS oznacza, że po prostu nie trzeba już uruchamiać swoich własnych serwerów SQL, więc nie ma potrzeby posiadania umiejętności związanych z SQL Server.

Ponownie, to założenie nie jest zbyt trafne. DBaaS i zarządzane instancje są bardzo pomocnymi narzędziami w arsenale każdego specjalisty od baz danych. Nie eliminują one jednak potrzeby posiadania umiejętności związanych z SQL Server. Wynika to z dwóch powodów. Pierwszym powodem jest to, że nawet bazy danych hostowane w DBaaS muszą być rozwijane i ich kod musi być zoptymalizowany, aby uniknąć słabej wydajności. Drugim powodem jest to, że DBaaS lub nawet zarządzane instancje nie zawsze są dobrym wyborem. Czasami nie są nawet wykonalne dla danej obciążenia. Chociaż istnieje wiele powodów, jednym powszechnym powodem jest koszt. Płacisz dostawcy chmury za hostowanie, aktualizowanie, zabezpieczanie i optymalizowanie wszystkich warstw poniżej bazy danych. To kosztuje. Jeśli przeniesiesz każdą bazę danych do RDS lub Azure SQL Database, możesz być w stanie zrekompensować ten koszt kosztem swojego fizycznego zespołu DBA. Jeśli jednak istnieją powody, dla których nie możesz przenieść wszystkich swoich baz danych do oferty DBaaS, to skończysz płacąc dwukrotnie.

> FIZYCZNI ADMINISTRATORZY BAZ DANYCH W PORÓWANIU DO ADMINISTRATORÓW BAZ DANYCH ODPOWIEDZIALNYCH ZA STRUKTURĘ
>
> Termin "fizyczny administrator baz danych" odnosi się do administratorów baz danych (DBA), którzy posiadają dobre umiejętności zarządzania instancjami SQL Server, ale nie są specjalistami w zakresie struktury bazy danych. Odpowiednikiem są tutaj "administratorzy baz danych odpowiedzialni za strukturę" (ang. Logical DBAs), którzy są specjalistami w zakresie tworzenia i optymalizacji baz danych, ale nie mają doświadczenia w zarządzaniu instancjami.
>
> W moim doświadczeniu widziałem firmy, które organizują swoje zespoły baz danych w ten sposób, zwłaszcza gdy mają dostawcę usług zarządzania infrastrukturą, który buduje i obsługuje instancje SQL Server, dba o ich dostępność. Często odpowiedzialność za bazy danych przechodzi następnie na wewnętrzny zespół wsparcia aplikacji, który odpowiada za obsługę i optymalizację konstrukcji na poziomie bazy danych.
>

W praktyce niemal zawsze zachodzi konieczność utrzymania niektórych baz danych poza ofertą DBaaS (ang. Database as a Service). Jednym z powodów jest wsparcie dostawcy. Jeśli korzystasz z komercyjnego produktu (Commercial Off The Shelf - COTS), który ma bazę danych SQL Server, jesteś zależny od zgody dostawcy na wsparcie produktu, jeśli baza danych zostanie przeniesiona do usługi DBaaS.

Kolejnym powodem jest to, że wiele firm ma duże, złożone aplikacje z warstwą danych, czasem z setkami tysięcy linii kodu osadzonych w komponentach, takich jak SQL Server Integration Services. Często te aplikacje nie mogą być po prostu przeniesione bez żadnej transformacji czy nowoczesnej aktualizacji. Wymagają one raczej przebudowy, co oznacza znaczne inwestycje zarówno czasowe, jak i finansowe, które często są trudne do uzasadnienia.



### 1.4 Dlaczego ważne jest właściwe skonfigurowanie SQL Servera?

W młodości często grałem w planszową grę o nazwie Othello. Slogan tej gry brzmiał: „Minuta na naukę, całe życie na opanowanie”. SQL Server zawsze przypomina mi tę dewizę. Jednym z atutów SQL Servera w porównaniu z niektórymi konkurentami jest łatwość implementacji. Jednakże, bardzo trudno go jest właściwie skonfigurować.

Ma to znaczenie, ponieważ źle skonfigurowana instancja SQL Servera może prowadzić do problemów, takich jak:

1. Słaba wydajność
2. Kradzież danych
3. Ataki ransomware
4. Niezgodność z przepisami
5. Utrata danych w przypadku katastrofy
6. Długi okres przestoju w przypadku katastrofy

Podobnie, źle napisany kod bazy danych może prowadzić do:

1. Słabej wydajności
2. Naruszeń bezpieczeństwa
3. Nieobsłużonych awarii
4. Kodu, który jest trudny do utrzymania

Te problemy mogą skutkować ogromnymi wydatkami dla firm, utratą przychodów, utratą reputacji, wzrostem rotacji pracowników, a nawet postępowaniem sądowym. Dlatego rzeczywiście ważne jest właściwe skonfigurowanie SQL Servera.



### 1.5 Podsumowanie

• Źródłem błędów SQL Servera jest założenie, że bazy danych są proste i łatwe.
• SQL Server to popularny system zarządzania relacyjnymi bazami danych (RDBMS)
• SQL Server ma duży, złożony ekosystem z wieloma głównymi komponentami
• Podstawowym składnikiem SQL Server jest silnik bazy danych
• Silnik bazy danych zawiera komponenty, w tym silnik relacyjny i silnik magazynu
• SQL Server jest obsługiwany w systemach operacyjnych Windows i Linux
• SQL Server jest obsługiwany na serwerach fizycznych, maszynach wirtualnych i kontenerach Linux
• W chmurze publicznej SQL Server jest dostępny w ramach IaaS lub jako oferta PaaS.
• Ważne jest, aby dobrze zaimplementować SQL Server, aby uniknąć problemów, takich jak słaba wydajność, utrata danych i naruszenia bezpieczeństwa.



## 2 Standardy Rozwoju


W tym rozdziale omówione zostaną:

- Wprowadzenie do standardów rozwoju
- Standardy nazewnictwa
- Standardy kodowania

W tym rozdziale zgłębimy, dlaczego posiadanie (i przestrzeganie!) standardów rozwoju jest tak istotne. Zaczniemy od dokładnego zdefiniowania, co rozumiemy, gdy mówimy o standardach rozwoju w kontekście tworzenia aplikacji warstwy danych przy użyciu SQL Servera. Standardy rozwoju obejmują następujące obszary:

- Konwencje nazewnicze
- Standardy kodowania
  - Stylowe
  - Techniczne

W szczególności przyjrzymy się powszechnym błędom popełnianym przez specjalistów od baz danych i omówimy, jak unikać tych samych pułapek. Na przykład, wyobraź sobie topologię przedstawioną na rysunku 2.1.

![img](https://drek4537l1klr.cloudfront.net/carter/v-1/Figures/02__image001.png)

W tej sytuacji otrzymujesz zgłoszenie od zespołu aplikacyjnego odpowiedzialnego za aplikację o nazwie TimeChewer, informujące o wolno działających zapytaniach. Wspominają, że baza danych aplikacji jest hostowana na serwerze o nazwie `sql-shared-app-server`. 

Po połączeniu się z serwerem, odkrywasz instancje o nazwach SQL01 i SQL02, bez jasnych wskazówek, która instancja zawiera bazę danych aplikacji. Po zbadaniu każdej instancji znajdujesz bazy danych o nazwach DB01, DB02, DB03 i DB04.

Po skontaktowaniu się z zespołem aplikacyjnym udzielają oni informacji, że TimeChewer łączy się z DB03 na SQL01. Następnie przystępujesz do analizy jednej ze słabo działających procedur składowanych, a poniżej znajduje się definicja tej procedury:

```sql
ALTER PROCEDURE dbo.proc01

AS
BEGIN
;with t3 as (select col1, col2, col3 from tbl03) select t1.col1, t2.col2, t1.col2, t1.col3, t3.col1, t3.col2 from dbo.tbl01 t1 inner join tbl02 t2 on t1.col1 = t2.col1 and t1.col3 < 55 inner join t3 on t3.col1 = t1.col1 union select col1, col2, col3, NULL, NULL, '0' from dbo.tbl04 ;
END
```

Wyobraź sobie, że właśnie próbujesz przyjrzeć się procedurze składowanej, starając się zrozumieć, co właściwie robi. W skrócie, poświęciłeś 30 minut na próby zidentyfikowania, która baza danych sprawia problemy, i teraz będziesz musiał poświęcić kolejne 15 minut na sformatowanie kodu tak, aby był czytelny, oraz zrozumienie, co właściwie robi ta bardzo prosta procedura składowana. To zmarnowane 45 minut, zanim w ogóle zaczniesz badać problem.

Choć to jest skrajny przykład, ilustruje, dlaczego konwencje nazewnictwa i standardy kodowania są tak istotne, oraz dlaczego nieprawidłowe postępowanie w tej dziedzinie będzie sprawiać frustrację tobie i innym, którzy próbują zrozumieć kod.

### 2.1 #1 Niewłaściwe nazwy obiektów

Choć przykład użyty w wprowadzeniu do tego rozdziału dotyczący nieopisowych nazw baz danych był dość ekstremalny, widziałem przypadki, gdzie ludzie stosowali takie standardowe nazwy instancji jak SQL01, SQL02 itp., co naprawdę sprawia kłopoty. Jednak bardziej powszechne stosowanie nieopisowych nazw występuje na poziomie kodu, i to na tym aspekcie skupimy się w tym rozdziale.

> PORADA
>
> Więcej rozmawiamy o konkretnych problemach spowodowanych nieprawidłowym nazewnictwem instancji w rozdziale 8.
>

Aby to omówić, wróćmy do naszego przykładu MagicChoc, który wprowadziliśmy w rozdziale 1. W tym rozdziale omówimy bazę danych SalesDB, która jest podstawą dla strony internetowej. Rysunek 2.2 przedstawia diagram komponentów tej bazy danych, pokazujący obiekty bazy danych (takie jak tabele, procedury i widoki).

![img](https://drek4537l1klr.cloudfront.net/carter/v-1/Figures/02__image003.png)

Zauważysz, że nazwy obiektów w tej bazie danych nie zostały zbyt dobrze przemyślane. Zawsze należy dążyć do tego, aby kod był samodokumentujący się. Wbrew powszechnemu przekonaniu, nie oznacza to koniecznie zasypania kodu komentarzami. Komentarze mogą być z pewnością pomocne, aby wyjaśnić skomplikowaną logikę, ale znacznie ważniejsze jest to, aby kod był napisany i strukturyzowany w taki sposób, że każdy, kto czyta kod i jest biegły w danym języku, może zrozumieć, co kod próbuje osiągnąć. Znaczące nazwy obiektów stanowią duży element tego.

Wyobraź sobie, że zostałeś poproszony o naprawienie błędu w procedurze, która aktualizuje system inwentarza po złożeniu zamówienia. Nie jesteś zaznajomiony z tym procesem, więc Twoim pierwszym krokiem może być uruchomienie zapytania zwracającego listę procedur składowanych w bazie danych. Wpis 2.1 przedstawia trzy sposoby, w jakie możesz wykonać to zadanie. Wszystkie trzy zapytania zwrócą te same wyniki.

Wpis 2.1 Zwróć listę procedur składowanych

```sql
SELECT 
    name
FROM sys.procedures ;
 
SELECT 
    name
FROM sys.objects
WHERE type_desc = 'SQL_STORED_PROCEDURE' ;
 
SELECT 
    name
FROM sys.objects
WHERE type = 'P' ;
```

Oczywiście możesz również wybrać korzystanie z SQL Server Management Studio (SSMS), aby wyświetlić procedury za pomocą interfejsu graficznego. Niezależnie od tego, jak zdecydujesz się wyświetlić obiekty, zobaczysz, że istnieją procedury o nazwach updateStock i stockUpdate. Która procedura jest potrzebna do debugowania? Kto wie? Zanim rozpoczniesz zadanie, będziesz musiał przejrzeć definicje obu procedur składowanych lub dowiedzieć się, gdzie procedura jest wywoływana, aby ustalić nazwę odpowiedniej procedury.

W naszym przypadku ustalenie, gdzie jest wywoływana procedura składowana, będzie trochę kłopotliwe, ponieważ jest faktycznie wywoływana przez inną procedurę składowaną – procedurę składowaną orders. Ponownie ta procedura ma generyczną, dość bezsensowną nazwę, która nie wyraża celu procedury. Więc będziesz musiał prześledzić aplikację, która uruchomiła ten proces, aby odkryć pierwszą procedurę w łańcuchu.

Teraz więc wyobraź sobie, że ustaliłeś, że procedura składowana `orders` to pierwsza procedura składowana w tym łańcuchu. Możesz znaleźć definicję tej procedury składowanej za pomocą SSMS lub wykonując zapytanie z listingu 2.2.

**Listing 2.2 Pobierz definicję procedury składowanej**

```sql
SELECT s.definition
FROM sys.objects o 
INNER JOIN sys.sql_modules s
    ON o.object_id = s.object_id
WHERE o.name = 'sp_orders' ;
```

> **Wskazówka**
>
> Osobiście wolę generować definicję procedury składowanej z Eksploratora obiektów w SSMS, zamiast pobierać definicję z katalogowego widoku `sys.sql_modules`. Wynika to z tego, że pobranie z `sys.sql_modules` nie zachowa formatowania.
>

Definicję procedury składowanej sp_orders można znaleźć w listingu 2.3. Parametr `@AddressID` jest bezsensowny. Czy to jest adres rozliczeniowy czy adres dostawy? Parametr `@Address` jest zarówno bezsensowny, jak i mylący. Nazwa parametru sugeruje rzeczywisty adres, podczas gdy jest to przeznaczone do przechowywania identyfikatora adresu. Ponadto, odnosi się on do któregoś z adresów? Kolejną bezsensowną nazwą parametru jest `@date`. Czy to jest data zamówienia? Data dostawy? Może nawet sugerować znacznik czasowy, kiedy rekord został wstawiony do tabeli! Wewnątrz ciała procedury mamy również mylące nazwy zmiennych. Zmienna `@stock` ma przechowywać ilość zamówionych produktów, a `@product` przechowuje identyfikator produktu. Te nazwy po prostu nie są jasne, i trzeba sięgnąć do kodu, aby zrozumieć ich cel.

Listing 2.3 Definicja procedury sp_orders

```sql
CREATE PROCEDURE sp_orders
    @CustomerID INT,
    @LineItems XML,
    @AddressID INT,
    @Address INT,
    @date DATETIME
AS
BEGIN
    DECLARE @Stock INT = 0 ;
    DECLARE @Product INT = 0 ;
    
    INSERT INTO tbl_orders (CustomerID, LineItems, BillingAddressID, DeliveryAddressID, Date)
    VALUES (@CustomerID, @LineItems, @AddressID, @Address, @date) ;
 
    SET @Stock = @LineItems.value('(/Product/@qty)[1]', 'int') ;
    SET @Product = @LineItems.value('(/Product/@ProductID)[1]', 'int') ;
 
    EXEC sp_stockUpdate @product, @stock ;
END
```

> **Wskazówka**
>
> Jeśli nie jesteś zaznajomiony z tym, jak korzystać z XML w SQL Server, to nie jesteś sam. Faktycznie, niewłaściwe wykorzystanie XML to kolejny powszechny błąd, który omówimy w rozdziale 3.
>

Czytając definicję tej procedury składowanej, możemy już zauważyć, że bezsensowne nazwy parametrów i zmiennych sprawiają nam kłopoty. Jeśli będziemy musieli rozszerzać lub debugować funkcjonalność związana z adresami lub czasem, to będziemy musieli poświęcić czas na zrozumienie, co oznaczają dane.

Na razie jednak musimy rozwiązać problem z aktualizacją zapasów, a ta definicja procedury składowanej pozwoliła nam ustalić, że procedura sp_stockUpdate prawdopodobnie zawiera kod, który musimy debugować. Przyjrzyjmy się więc procedurze z Listingu 2.4.

Widzimy, że `sp_stockUpdate` to prosta procedura składowana, która aktualizuje poziom zapasów w SalesDB, a następnie za pomocą Linked Server aktualizuje poziom zapasów w systemie Inventory. Widzimy również, że przyczyną błędu jest używanie niekonsekwentnych nazw, co wprowadza programistę w błąd i powoduje przekazanie identyfikatora produktu do obliczeń aktualizujących ilość pozostałą na magazynie, jednocześnie próbując filtrować identyfikator produktu w oparciu o ilość zamówionego towaru, zamiast faktycznego identyfikatora produktu.

Listing 2.4 Definicja procedury sp_stockUpdate

```sql
CREATE PROCEDURE sp_stockUpdate
    @ProductStockLevel INT,
    @StockID INT
AS
BEGIN
    UPDATE tbl_products
    SET StockQty = StockQty - @productStockLevel
    WHERE ProductID = @StockID ;
 
    UPDATE [DCSVR01\Inventory].InventoryDB.dbo.productStock
    SET StockQty = StockQty - @productStockLevel
    WHERE ProductID = @StockID ;
END
```

> **PRZYKŁAD Z RZECZYWISTEGO ŚWIATA**
>
> Ten przykład jest dość prosty, służy tylko celom ilustracyjnym, ale możesz sobie wyobrazić, że przy skomplikowanym kodzie te wyzwania szybko zamienią się w koszmar. Przykład w tej sekcji luźno oparty jest na rzeczywistym przykładzie, z którym się spotkałem. Programista napisał bardzo złożony proces, z pięcioma warstwami procedur składowanych i wieloma punktami wejścia z zewnętrznych aplikacji. Wszystkich procedur było około 2000 linii kodu. Po jego odejściu odkryto błąd, i próbując go rozwiązać, stwierdziliśmy, że programista używał zmiennych, nazw parametrów i nazw kolumn zamiennie. Sam błąd był prosty i powinien zostać rozwiązany w ciągu kilku godzin, ale z powodu plątaniny nazw zajęło to trzy dni wysiłku, aby rozwiązać problem. Następnie zajęło kolejne dwie kolejne tygodnie, aby zaktualizować wszystkie nazwy, by były spójne, i przeprowadzić testy regresji.
>



> **PORADA**
>
> Pozytywnym przykładem dobrego nazewnictwa w naszym przykładzie są kolumny klucza głównego tabel. Zauważ, że tabela tbl_products ma ProductID, tabela tbl_orders ma OrderID, a tabela tbl_addresses ma AddressID. Powszechnym błędem jest używanie ID jako samodzielnej nazwy kolumny dla kolumn klucza głównego we wszystkich tabelach, co oznacza, że kolumna klucza głównego we wszystkich tabelach ma tę samą nazwę. Pewnie możesz sobie wyobrazić, jak łatwo mogłoby to stać się mylące w kontekście skomplikowanej procedury składowanej.
>

Jak widziałeś, niedostateczne uwzględnienie nazw może wprowadzać błędy, kosztować cenny czas podczas próby naprawy problemów i utrudniać rozwijanie funkcji. Dlatego mam nadzieję, że zgadzasz się, iż warto poświęcić trochę czasu, aby utrzymywać spójność i znaczenie nazw, to praktyka, którą wszyscy powinniśmy stosować.

To może brzmieć łatwo, i zazwyczaj tak jest w przypadku małych projektów baz danych. Ale kiedy masz duży projekt, który jest budowany przez długi czas przez wielu programistów, może to być dość trudne. Może być jeszcze trudniejsze, jeśli projekt jest dostarczany zgodnie z metodologią Agile. W przeciwieństwie do tradycyjnych projektów kaskadowych, projekty Agile korzystają z metodologii takich jak Sprint lub Kanban, które dzielą zbiór prac na małe kawałki, umożliwiające dostarczanie przyrostowej funkcjonalności w szybszym tempie.

Podejście Agile może działać bardzo dobrze i przynosi wiele korzyści w porównaniu do projektów kaskadowych. Kiedy jednak jest źle wdrożone, może pojawić się założenie, że projektowanie, architektura i planowanie nie są wymagane. W projektach bazodanowych architektura jest zawsze kluczowa, ponieważ musisz rozważyć ważne elementy projektowe, takie jak schemat tabel. Niezastosowanie się do tego może skutkować plątaniną tabel, które nie są znormalizowane, prawdopodobnie duplikacją danych i problemami z wydajnością, a także zwiększeniem złożoności. Jeśli zamierzasz poświęcić czas na początku na analizę projektu tabeli, to jest to proste i stosunkowo szybkie zadanie, aby spojrzeć również na inne elementy projektowe, takie jak nazewnictwo obiektów.



**PORADA**

Dobrze zarządzany projekt baz danych, korzystający z metodyki Agile, będzie miał albo wiele sprintów projektowych na początku projektu, albo będzie miał okresowe sprinty, podczas których projektowane jest konkretne miejsce (zazwyczaj schemat) w bazie danych.

A co powinien zrobić programista, który napisał nasze procedury składowane dla MagicChoc inaczej? Przyjrzyjmy się nowej definicji procedury sp_orders w listingu 2.5. Po pierwsze, parametry Address są teraz oczywiście ID. Co ważniejsze, jasno widać, który parametr odpowiada za Billing Address, a który za Delivery Address. Parametr @date to teraz oczywiście data zamówienia. Nie tylko zmieniliśmy jednak parametr, ale również zaktualizowaliśmy nazwę kolumny w tabeli, co eliminuje niejednoznaczność na tym poziomie. Zmienną @product zaktualizowano na @ProductID, a zmienną @stock na @OrderQty. Dodatkowo zaktualizowaliśmy definicję XML, dopasowując dane na całej strukturze. Wreszcie zaktualizowaliśmy wykonanie procedury sp_updateProductStockLevel, aby przekazać zmienną we właściwej kolejności. Ostatnia zmiana naprawi błąd.

**Listing 2.5 Nowa definicja procedury sp_orders**

```sql
ALTER PROCEDURE sp_orders
    @CustomerID INT,
    @LineItems XML,
    @BillingAddressID INT,
    @DeliveryAddressID INT,
    @OrderDate DATETIME
AS
BEGIN
    DECLARE @OrderQty INT = 0 ;
    DECLARE @ProductID INT = 0 ;
    
    INSERT INTO tbl_orders (CustomerID, LineItems, BillingAddressID, DeliveryAddressID, OrderDate)
    VALUES (@CustomerID, @LineItems, @BillingAddressID, @DeliveryAddressID, @OrderDate) ; 
 
    SET @OrderQty = @LineItems.value('(/Product/@OrderQty)[1]', 'int') ; 
    SET @ProductID = @LineItems.value('(/Product/@ProductID)[1]', 'int') ;
 
    EXEC sp_updateProductStockLevel @OrderQty, @ProductID ; 
END
```

> **PORADA**
>
> W stylu mojego ulubionego prywatnego detektywa z lat 80. - "Wiem, co myślisz, i masz rację. Z pewnością powinniśmy także zmienić nazwę procedury, prawda?" Jednak na razie zostawiłem ją bez zmian, ponieważ będziemy o niej rozmawiać bardziej szczegółowo w Błędzie #2.
>

Proszę również zwrócić uwagę na nową definicję procedury sp_stockUpdate w listingu 2.6. Zauważ, że nazwa procedury została zmieniona na `sp_updateProductStockLevel`, co jest znacznie bardziej sensowną nazwą dla procedury. Dzięki tej nazwie nie mielibyśmy problemu z jej odnalezieniem na początku tego przykładu.

**Listing 2.6 Nowa definicja procedury sp_updateProductStockLevel**

```sql
CREATE PROCEDURE sp_updateProductStockLevel
    @OrderQty INT,
    @ProductID INT
AS
BEGIN
    UPDATE tbl_products
    SET StockQty = StockQty - @OrderQty
    WHERE ProductID = @ProductID 
 
    UPDATE [DCSVR01\Inventory].InventoryDB.dbo.productStock
    SET StockQty = StockQty - @OrderQty
    WHERE ProductID = @ProductID ;
END
```

Po przejrzeniu powyższych dwóch nowych definicji procedur przechowywanych zauważysz, jak znacząca i spójna nazwa znacznie pomaga w samodzielnym dokumentowaniu kodu. Chociaż moglibyśmy dodać komentarze w obu procedurach, nie wniosłyby one zbyt wiele korzyści. Każdy, kto potrafi czytać T-SQL, może teraz czytać te definicje i będzie bardzo jasne, co robią te procedury i jakie dane są przekazywane. To nie oznacza, że komentarze nie mają swojego miejsca, ale zazwyczaj służą one do wyjaśnienia koncepcyjnej logiki biznesowej w bardzo długich procedurach przechowywanych, lub do określenia zewnętrznych wejść/wyjść, takich jak umowa API. Jeśli twój kod jest samodzielnym dokumentem, komentarze nie powinny być wymagane do wyjaśnienia definicji kodu na niskim poziomie.

### 2.2 #2 Używanie przedrostków obiektów

Niektórzy programiści uwielbiają używać przedrostków obiektów. Przedrostki obiektów to dodawanie tbl_ na początku wszystkich nazw tabel, sp_ na początku nazw przechowywanych procedur, fn_ na początku nazw funkcji, a v_ na początku widoków.

Wyobraź sobie, że masz bazę danych zawierającą 250 tabel. Szukasz definicji kolumny w tabeli, więc szybko przeglądasz listę tabel w Eksploratorze obiektów. Jednak wszystkie tabele mają przedrostek tbl_. To automatycznie zwiększa czas analizy podczas przeglądania.

To dlatego, że twoje mózg jest przystosowany do śledzenia wspólnych wzorców. Na przykład znasz alfabet całkiem dobrze, prawda? Świetnie! Spróbuj teraz powiedzieć go od tyłu - od z do a. To zaskakująco trudne. To dlatego, że twój mózg nie jest dostosowany do tego wzorca. Nawet jeśli przeszkolisz swój mózg, aby łatwo analizować ciąg od piątego znaku, nie oznacza to, że inni znajdą to łatwe.

To samo dotyczy skanowania obiektów z przedrostkami. Nawet jeśli przeszkoliłeś swój mózg, aby łatwo skanować ciąg od piątego znaku, twoi współpracownicy programiści, deweloperzy BI i administratorzy mogą tego nie znaleźć łatwym, więc nieumyślnie utrudnisz im pracę.

Jeśli istniałaby dobra przyczyna, a istnieją one w niektórych językach, takich jak ARM w Azure, wtedy rozważyłbym to. W przypadku SQL Server jednak nie ma ku temu dobrej przyczyny. Powodem, który zazwyczaj jest podawany, to „Bo potrzebuję sposobu na łatwe zidentyfikowanie, które obiekty są jakiego rodzaju”.

Rozważmy więc tę przyczynę na przykładzie MagicChoc poprzez kilka różnych spojrzeń. W szczególności w SalesDB istnieje tabela o nazwie tbl_Orders i przechowywana procedura o nazwie sp_orders. Dodatkowo istnieje tabela o nazwie tbl_customers i widok o nazwie v_customers. Pierwsze spojrzenie to czyste zidentyfikowanie rodzaju obiektu. Skorzystajmy z obiektów klientów, aby o tym pomyśleć. Istnieją dwie możliwości eksploracji tych obiektów. Zarówno za pośrednictwem SSMS, jak i kodu. Rysunek 2.3 pokazuje oba obiekty w Eksploratorze obiektów.

![img](https://drek4537l1klr.cloudfront.net/carter/v-1/Figures/02__image005.png)

Jak widzisz, jest dość łatwo określić, która to tabela, a która to widok. Tabela jest wyświetlana węzle Tabele, a widok w węźle Widoki.

A co jeśli chciałbyś iterować po swoich obiektach za pomocą kodu? Zapytanie w listingu 2.7 pokazuje, jak można pozyskać szczegóły obu obiektów za pomocą T-SQL. Zauważ, że obiekty są łatwo rozróżnialne dzięki kolumnie type_desc w sys.objects.

Listing 2.7 Zwróć obiekty klientów

```sql
SELECT
       name
     , type_desc
FROM sys.objects
WHERE name LIKE '%customers%' ;
```

Wyniki tego zapytania są poniżej:

```
Name                 type_desc

tbl_customers     USER_TABLE
v_customers             VIEW
```

Znowu w głosie legendarnego detektywa lat 80. „Wiem, co myślisz, i masz rację. Gdybyśmy odrzucili przedrostek, tabela i widok klientów mieliby identyczną nazwę, a to nie jest dozwolone.” I to ładnie prowadzi mnie do drugiej perspektywy.

Pamiętasz, że w Błędzie nr 1 rozmawialiśmy o nadawaniu obiektom znaczących nazw, które sprawiają, że twój kod jest samodokumentujący. Pomyślmy o tym z perspektywy obiektów klientów. Istnieje tabela i widok o dokładnie tej samej nazwie. Jeśli nie pełnią dokładnie tej samej funkcji, to to nie może być właściwe, chyba że widok zwraca po prostu wszystkie dane z jednej podstawowej tabeli. A jeśli widok zwraca dokładnie te same dane co tabela, to po co w ogóle istnieje ten widok?



> POWODY ISTNIENIA WIDOKÓW, KTÓRE DOKŁADNIE ODPOWIADAJĄ TABELI
>
> Istnieją dwie przyczyny, dla których niektórzy ludzie mogą mieć widoki, które zwracają wszystkie kolumny z pojedynczej tabeli. Pierwszym jest sposób blokowania tabel, aby uniknąć przypadkowej zmiany schematu tabeli. Jeśli utworzysz widok Z WIĄZANIEM SCHEMY, nie można zmienić definicji tabeli bez wcześniejszego usunięcia (lub usunięcia WIĄZANIA SCHEMY) z widoku. Nie polecam tego podejścia. Dużo lepiej i bardziej zrozumiale jest po prostu ograniczenie dostępu, aby uniemożliwić modyfikacje schematu przy użyciu odpowiedniej strategii zabezpieczeń. Istnieje zasada projektowania zwana Zasadą Najmniejszego Zaskoczenia. Podpisuję się pod tą teorią.
>
> Drugim powodem, dla którego możesz zobaczyć widok zwracający wszystkie kolumny z jednej tabeli, jest rygorystyczna zasada, zgodnie z którą wszystkie aplikacje muszą uzyskiwać dostęp do danych za pomocą warstwy abstrakcji. Zdecydowanie popieram zasadę leżącą u podstaw tego podejścia, którą jest utrzymanie złożoności na poziomie serwera SQL i umożliwienie programistom aplikacji prostego pobierania danych z widoków i przechowywanych procedur. Jednak gdy jest to przesadzone, dodaje się tylko dodatkowe obiekty bez żadnych korzyści. Po prostu masz więcej obiektów do utrzymania.
>



W `SalesDB` tabela `tbl_customers` zawiera szczegółowe informacje o klientach. Dlatego ma znaczącą nazwę. Widok `v_customers` zwraca jednak zarówno dane klientów, jak i dane zamówień. Dlatego jego nazwa jest niewłaściwa. Bardziej odpowiednią nazwą mogą być `CustomerOrders`.

Tabela `tbl_addresses` nie ma odpowiednich widoków, procedur ani funkcji, więc możemy po prostu usunąć z niej przedrostek. Tabela tbl_products ma odpowiadającą procedurę składowaną, ale procedura ta ma obecnie przedrostek sp_, więc możemy śmiało zmienić również nazwę tej tabeli. Powinniśmy jednak zmienić nazwę procedury składowanej na `sp_addOrder`, aby była zrozumiała.

> WSKAZÓWKA
>
> Zgadłeś, jak powiedział Magnum PI: „Wiem, za co dziękujesz i masz rację. Czy nie powinniśmy również usunąć przedrostka sp_ z procedur przechowywanych?” Tak, powinniśmy, ale omówimy to szerzej w Błędzie nr 3, więc na razie zostawimy je bez zmian.

Skrypt z aukcji 2.8 usunie przedrostek z tabel i widoku.

Listing 2.8 Drop the prefix

```sql
EXEC sp_rename 'tbl_addresses', 'addresses' ;
 
EXEC sp_rename 'tbl_orders', 'orders' ;
 
EXEC sp_rename 'tbl_customers', 'customers' ;
 
DROP VIEW dbo.v_customers ;
GO
 
CREATE VIEW dbo.customerOrders
AS    
SELECT    
      c.FirstName    
    , c.LastName    
    , c.email
    , o.LineItems.value('(/Product/@ProductName)[1]', 'int') AS ProductName
    , o.LineItems.value('(/Product/@OrderQty)[1]', 'int') AS OrderQty
    , o.OrderDate
FROM dbo.customers c
INNER JOIN dbo.orders o
    ON c.CustomerID = o.CustomerID ;
 
DROP PROCEDURE dbo.sp_orders ;
GO
 
CREATE PROCEDURE dbo.sp_addOrder
    @CustomerID INT,
    @LineItems XML,
    @BillingAddressID INT,    
    @DeliveryAddressID INT,    
    @OrderDate DATETIME    
AS
BEGIN
    DECLARE @OrderQty INT = 0 ;    
    DECLARE @ProductID INT = 0 ;    
    
    INSERT INTO orders (CustomerID, LineItems, BillingAddressID, DeliveryAddressID, OrderDate)
    VALUES (@CustomerID, @LineItems, @BillingAddressID, @DeliveryAddressID, @OrderDate) ;    
 
    SET @OrderQty = @LineItems.value('(/Product/@OrderQty)[1]', 'int') ;    
    SET @ProductID = @LineItems.value('(/Product/@ProductID)[1]', 'int') ;
 
    EXEC sp_updateProductStockLevel @OrderQty, @ProductID ;    
END
GO
 
DROP PROCEDURE dbo.sp_updateProductStockLevel ;
GO
 
CREATE PROCEDURE dbo.sp_updateProductStockLevel
    @OrderQty INT,    
    @ProductID INT    
AS
BEGIN
    UPDATE products
    SET StockQty = StockQty - @OrderQty
    WHERE ProductID = @ProductID ;
 
    UPDATE [DCSVR01\Inventory].InventoryDB.dbo.productStock
    SET StockQty = StockQty - @OrderQty
    WHERE ProductID = @ProductID ;
END
```

### 2.3 #3 Przeklęty prefiks sp_

W błędzie #2 poświęciliśmy czas na omówienie, dlaczego prefiksy nie są dobrym pomysłem, więc dlaczego mamy błąd poświęcony prefiksowi sp_? Najlepszym sposobem, aby wyjaśnić, dlaczego prefiks sp_ zasługuje na szczególne wyróżnienie, jest przedstawienie przykładu. Dlatego chciałbym, abyś rozważył definicję procedury składowanej sp_AddUser w bazie danych SalesDB, którą można znaleźć w listingu 2.9. Procedura ta po prostu dodaje nowego użytkownika do aplikacji, gdy rejestruje się na konto online za pośrednictwem strony internetowej MagicChoc.

**Listing 2.9 Definicja procedury sp_AddUser**

```sql
CREATE PROCEDURE sp_addUser
    @UserDetails XML
AS
BEGIN
    OPEN SYMMETRIC KEY MagicChocKey
        DECRYPTION BY CERTIFICATE MagicChocCertificate ;

    INSERT INTO dbo.customers (
        FirstName,
        LastName,
        email,
        UserPassword
    )
    VALUES (
        @UserDetails.value('(/User/FirstName)[1]','nvarchar(128)'),
        @UserDetails.value('(/User/LastName)[1]','nvarchar(128)'),
        @UserDetails.value('(/User/email)[1]','nvarchar(512)'),
        ENCRYPTBYKEY(KEY_GUID('MagicChocKey'), @UserDetails.value('(/User/UserPassword)[1]','nvarchar(128)'))
    );

    CLOSE SYMMETRIC KEY MagicChocKey;
END
```

Wygląda na to, że aplikacja zgłasza dziwny błąd. Aby spróbować zdiagnozować problem, możemy zasymulować aplikację wywołującą procedurę składowaną za pomocą skryptu z Listingu 2.10.

**Listing 2.10 Wywołanie procedury składowanej sp_AddUser**

```sql
DECLARE @UserDetails XML ;
 
SET @XML = N'<User>
    <FirstName>Peter</FirstName>
    <LastName>Carter</LastName>
    <email>peter@carter.com</email>
    <UserPassword>myPaSSw0rd</UserPassword>
</User>' ;
 
EXEC sp_adduser @UserDetails ;
```

Korzystanie z tego skryptu do wykonania procedury składowanej powoduje wystąpienie następującego błędu:

```
Msg 257, Level 16, State 3, Procedure sp_adduser, Line 0 [Batch Start Line 94]
Implicit conversion from data type xml to nvarchar is not allowed. Use the CONVERT function to run this query.
```

To dziwne! Błąd wydaje się dotyczyć Linii 0 procedury składowanej, co sugeruje problem z przekazywaniem zmiennej @UserDetails. Jednak nie zachodzi konwersja z NVARCHAR do XML. Zmienna jest typu danych XML, a parametr procedury jest zdefiniowany jako typ danych XML. Co się dzieje?

Spróbujmy przeprowadzić eksperyment. Uruchommy procedurę bez podawania parametrów, używając polecenia EXEC sp_adduser. Poniżej przedstawiono wynik błędu:

```
Msg 201, Level 16, State 4, Procedure sp_adduser, Line 0 [Batch Start Line 104]
Procedure or function 'sp_adduser' expects parameter '@loginame', which was not supplied.

```

Mamy więc inny błąd, ale to jest jeszcze dziwniejsze. Reklamuje, że nie przekazujemy parametru @loginname. Ale nasza procedura składowana nie ma parametru @loginname. To prawie jakbyśmy wykonali nieprawidłową procedurę składowaną.

Dobrze, musimy dotrzeć do sedna tego problemu. Aby to zrobić, uruchommy zapytanie z Listingu 2.11, które zwraca wyniki z katalogowej widoku sys.all_objects. Ten obiekt dostarcza sumy obiektów użytkownika i obiektów systemowych.

**Listing 2.11 Pobieranie szczegółów z sys.all_objects**

```sql
SELECT 
      name
    , SCHEMA_NAME(schema_id) AS SchemaName
    , type_desc
    , is_ms_shipped
FROM sys.all_objects
WHERE name = 'sp_adduser';
```

Wyniki tego zapytania są poniżej:

```
name           SchemaName    type_desc           is_ms_shipped

sp_adduser    dbo            SQL_STORED_PROCEDURE    0
sp_adduser    sys            SQL_STORED_PROCEDURE    1
```

Wyniki pokazują, że istnieją dwie procedury o tej samej nazwie. Pierwszy wynik to nasza procedura składowana. Możemy to stwierdzić, ponieważ znajduje się w schemacie dbo, a flaga is_ms_shipped wynosi false. Drugi wynik to systemowa procedura składowana o tej samej nazwie. Możemy to stwierdzić, ponieważ znajduje się w schemacie sys, a flaga is_ms_shipped wynosi true.

Procedury składowane systemu są przechowywane w "ukrytej" tylko do odczytu bazie danych o nazwie resource, która przechowuje wszystkie obiekty systemowe. Jednak te obiekty wydają się znajdować we wszystkich bazach danych. Oznacza to, że można łatwo uzyskiwać do nich dostęp ze wszystkich baz danych. Wszystkie systemowe procedury składowane mają przedrostek sp_, a ten przedrostek powinien być zarezerwowany wyłącznie dla procedur składowanych systemu.

Gdy wykonujesz procedurę składowaną z przedrostkiem sp_, SQL Server najpierw poszuka tej procedury w bazie danych master. Tylko jeśli nie znajdzie procedury tam, będzie próbował odnaleźć procedurę w lokalnej bazie danych, w której została wykonana procedura.

Można to zademonstrować za pomocą skryptu w Listingu 2.12. Ten skrypt tworzy procedurę o nazwie sp_listing_2_12 w bazie danych master, a następnie wykonuje ją z bazy danych SalesDB, używając jednoczęściowej nazwy.

**Listing 2.12 Dostęp do procedury w master z bazy danych użytkownika**

```sql
USE MASTER
GO

CREATE PROCEDURE sp_listing_2_12
AS
BEGIN
SELECT 'Hello! I am in the master database!' ;
END
GO

USE SalesDb
GO

EXEC sp_listing_2_12 ;
```

Teraz, gdy rozumiemy, dlaczego używanie przedrostka sp_ to zły pomysł, oczyszczmy nazwy naszych procedur. Możemy to osiągnąć za pomocą skryptu w Listingu 2.13. Ponieważ nie musimy aktualizować definicji żadnych procedur, możemy użyć procedury sp_rename do wykonania tego zadania.

**Listing 2.13 Oczyszczanie nazw procedur**

```sql
EXEC sp_rename 'sp_addOrder', 'addOrder' ;

EXEC sp_rename 'sp_addUser', 'addUser' ;

EXEC sp_rename 'sp_updateProductStockLevel', 'updateProductStockLevel' ;

EXEC sp_rename 'sp_products', 'addProduct' ;

EXEC sp_rename 'sp_updateStock', 'updateStockLevelAfterManufacture' ;
```

Teraz, gdy rozwiązaliśmy wszystkie błędy w nazwach obiektów, warto spojrzeć, co to zrobiło z naszym diagramem komponentów. Zaktualizowany diagram można zobaczyć na rysunku 2.4.



![img](https://drek4537l1klr.cloudfront.net/carter/v-1/Figures/02__image007.png)

### 2.4 #4 Brak czasu na przestrzeganie standardów kodowania

Wyobraź sobie, że jesteś częścią zespołu 10 programistów pracujących nad dużą aplikacją warstwy danych. Wszyscy jesteście doświadczonymi programistami, a waszym celem jest bardzo krótki termin. Kusi, aby od razu zabrać się za pisanie kodu. W końcu wszyscy byliście już kilka razy na tym bloku. Wszyscy wiecie, że należy wcięciać kod czterema spacjami, prawda? Poczekaj chwilę! Czy to są cztery spacje czy tabulator?

Najszybszym sposobem na wywołanie sporu między dwoma programistami jest poruszenie tematu standardów kodowania. Każdy ma swoje standardy, których przestrzega, i każdy uważa, że jego standardy są najlepsze.

W zależności od wielkości i złożoności danego projektu standardy kodowania będą różnić się pod względem stopnia kompleksowości. Obejmą one zarówno wybory stylu, jak i kwestie techniczne.

W przeciwieństwie do standardów technicznych, istotą standardów stylu jest to, że tak naprawdę nie ma znaczenia, jakie są standardy, pod warunkiem, że są one i są konsekwentnie wdrażane w całym projekcie.

Standardy kodowania są miejscem, w którym architektura zaczyna odgrywać rolę. W Błędzie #1 mówiliśmy o potrzebie architektury w projektach SQL Server, aby zapewnić optymalne i wydajne projektowanie schematów bazy danych. Wspomnieliśmy również, że ta architektura powinna uwzględniać konwencje nazewnicze. Innym aspektem architektury jest zapewnienie istnienia zestawu standardów kodowania, do których ludzie mogą się stosować.

Wybory stylu mogą obejmować elementy takie jak:

- Wcięcia kodu czterema spacjami czy tabulatorem?
- Czy wszystkie instrukcje powinny kończyć się średnikiem?
- Przy wyświetlaniu nazw kolumn, czy separator przecinka ma być na końcu wiersza czy na początku następnego?
- Czy klauzule ON powinny być określane w tej samej linii co klauzula JOIN?
- Camel case vs Pascal case

Standardy techniczne prawdopodobnie obejmują następujące kwestie:

- Nie używaj kursorów.
- Używaj UNION ALL zamiast UNION, gdy to możliwe.
- Nie używaj * w liście SELECT.
- Nie używaj NOLOCK.
- Unikaj używania DISTINCT.

> UWAGA
>
> Te listy mają na celu dać smak rozważaniom, które wpływają na standardy kodowania. W żaden sposób nie mają być wyczerpujące.
>

A co się dzieje, jeśli nie masz standardów kodowania? Prosta odpowiedź brzmi, że programiści robią po swojemu. Aby zastanowić się nad konsekwencjami tego, należy rozważyć wybory stylu i kwestie techniczne oddzielnie.



W kwestii wyborów stylu problem pojawia się przy utrzymaniu kodu. W poniższym skrypcie (który widzieliśmy wcześniej w tym rozdziale, ale jest powtórzony tu dla wygody) przedstawiony jest skrajny przykład tego, co może się zdarzyć, gdy brak jest ustalonych standardów stylu kodowania:

```sql
ALTER PROCEDURE [dbo].[proc01]
AS
BEGIN
;with t3 as (select col1, col2, col3 from tbl03) select t1.col1, t2.col2, t1.col2, t1.col3, t3.col1, t3.col2 from dbo.tbl01 t1 inner join tbl02 t2 on t1.col1 = t2.col1 and t1.col3 < 55 inner join t3 on t3.col1 = t1.col1 union select col1, col2, col3, NULL, NULL, '0' from dbo.tbl04 ;
END
```

Chociaż w rzeczywistym świecie mało prawdopodobne jest, aby kod był pisany tak kiepsko, pamiętajmy o naszej rozmowie w Błędzie #2 na temat tego, jak nasze mózgi przyzwyczajają się do pewnych wzorców podczas analizowania list obiektów. To samo dotyczy analizy kodu.

Rozważ zapytania w Listingu 2.14. Wyobraź sobie, że właśnie przeanalizowałeś 50 zapytań w stylu pierwszego zapytania. Następnie trafiasz na zapytanie napisane w stylu drugiego. W zależności od złożoności zapytania, może to spowolnić cię, może na kilka sekund, może na kilka minut. Obie zapytania zwracają te same wyniki. Są napisane tylko z użyciem innych standardów.

**Listing 2.14 Zmiana stylu kodu**

```sql
SELECT
      c.FirstName
    , c.LastName
    , o.LineItems
FROM dbo.customers c
LEFT JOIN dbo.orders o
    ON c.CustomerID = o.CustomerID
WHERE c.CustomerID >= 2 AND c.CustomerID <= 3
    AND c.CustomerID <> 2 ;
 
SELECT
    cust.FirstName,
    cust.LastName,
    ord.LineItems
FROM customers cust
LEFT JOIN (SELECT CustomerID, LineItems FROM Orders) ord ON cust.CustomerID = ord.CustomerID
WHERE cust.CustomerID BETWEEN 2 AND 3 AND cust.CustomerID != 2 ;
```

Ostatecznie brak stosowania spójnego stylu kodowania spowolni proces rozwiązywania błędów i rozwijania nowych ulepszeń, gdy twoja aplikacja warstwy danych przechodzi przez swoje cykle życia.

Jeśli chodzi o techniczne standardy kodowania, to są one znacznie bardziej normatywne. Jeśli programiści nie przestrzegają standardów technicznych, to prawdopodobnie twoja aplikacja doświadczy obniżonej wydajności, a nawet zwróci nieoczekiwane wyniki. Wszystkie wymienione wyżej standardy techniczne są powszechnymi błędami w SQL Server i zostaną omówione w rozdziale 5.

### 2.5 #5 Używanie pozycji kolumny porządkowej
SQL Server obsługuje użycie porządkowych numerów kolumn w klauzuli ORDER BY. Oznacza to, że zamiast porządkować według nazwy kolumny, można uporządkować według jej pozycji porządkowej w klauzuli SELECT. Rozważmy na przykład zapytanie z aukcji 2.15.

Listing 2.15 Sortowanie Według Numeru Pozycji Kolumny

```sql
SELECT * 
FROM SYS.databases 
ORDER BY 54;
```

Ok, więc według której kolumny posortowaliśmy zapytanie? Odpowiedź brzmi - według kolumny `log_reuse_wait_desc`. Jednak jedynymi sposobami ustalenia tego faktu są albo policzenie kolumn w zbiorze wynikowym, aż dojdziesz do 54. kolumny, albo uruchomienie zapytania z listingu 2.16, które pobiera nazwę kolumny z metadanych przechowywanych w widokach katalogu.

Listing 2.16 Pobierz Nazwę Kolumny z Metadanych

```sql
SELECT 
    c.name
FROM SYS.all_columns c
INNER JOIN SYS.all_objects o
    ON o.object_id = c.object_id
WHERE o.name = 'databases'
    AND c.column_id = 54;
```

> PORADA
>
> Jeśli obiektem, który otrzymujemy w wyniku zapytania, byłby obiekt użytkownika, a nie obiekt systemowy, to zapytanie również zadziałałoby, gdybyśmy dołączyli `sys.objects` do `sys.columns`.
>

### 2.6 Podsumowanie

- Zawsze powinieneś używać nazw obiektów, które mają znaczenie, z myślą o tym, aby twój kod był samodokumentujący.
- Zawsze zastanawiaj się nad architekturą bazy danych, czyli projektem swojej bazy danych, nawet w ramach projektów Agile.
- Powinieneś unikać używania prefiksów dla obiektów bazy danych, ponieważ mogą one sprawić, że obiekty będą trudniejsze do zlokalizowania.
- Bądź szczególnie ostrożny, aby unikać używania prefiksu "sp_" dla procedur składowanych, ponieważ wskazuje to, że są to procedury składowane systemowe, a nie zdefiniowane przez użytkownika.
- Zawsze poświęć czas, aby upewnić się, że twoja aplikacja warstwy danych stosuje standardy kodowania. Powinny one stanowić część twoich wysiłków architektonicznych i powinny uwzględniać zarówno wybory stylistyczne, jak i standardy techniczne.
- Unikaj sortowania według numerów kolumn, chyba że lubisz problemy!

## 3 Typy danych


W tym rozdziale znajdziesz:

- Wprowadzenie do typów danych i dlaczego są one ważne
- Konsekwencje używania niewłaściwego standardowego typu danych
- Błąd polegający na niekorzystaniu z zaawansowanych typów danych
- Korzyści z pracy z danymi XML i JSON

W tym rozdziale zgłębimy temat typów danych i dowiemy się, dlaczego tak istotne jest wybieranie odpowiednich typów danych dla kolumn naszych tabel. Rozpoczniemy od zbadania kilku prostych typów danych, które są powszechnie używane, ale których wielu ludzi używa nieprawidłowo. Zobaczymy, jaki wpływ może to mieć na koszty i wydajność.

Następnie przeanalizujemy niektóre zaawansowane typy danych w SQL Serverze, które są znacząco niedostatecznie wykorzystywane. Zbadamy przypadki użycia, które sprawiają, że są one tak użyteczne, oraz przyjrzymy się skutkom ich pomijania. Warto jednak zauważyć, że choć ten rozdział eksploruje HierarchyID, XML i JSON, istnieją także inne specjalizowane typy danych, takie jak GEOGRAPHY i GEOMETRY, przeznaczone dla danych geoprzestrzennych. Gorąco zachęcam do zbadania wszystkich zaawansowanych typów danych dostępnych w SQL Serverze.

MagicChoc zdecydowało, że potrzebuje nowej aplikacji obsługującej Zasoby Ludzkie. Skrypt w listingu 3.1 tworzy bazę danych `HumanResources`, a następnie tworzy pierwszą tabelę - `dbo.employees`. Zauważysz, że ta tabela została utworzona, używając typu danych NVARCHAR(MAX) dla każdej kolumny. Wygląda to dziwnie, prawda? Ale dlaczego? Wszystkie dane, które chcemy przechowywać, mogą być wprowadzane do tego rozległego typu danych. Czy to naprawdę ma znaczenie? Będziemy zgłębiać ten przykład przez cały ten rozdział.

Listing 3.1 Utwórz tabelę pracowników

```sql
CREATE DATABASE HumanResources ;
GO
 
USE HumanResources
GO
 
CREATE TABLE dbo.Employees (
    EmployeeID               NVARCHAR(MAX)    NOT NULL,
    FirstName                NVARCHAR(MAX)    NOT NULL,
    LastName                 NVARCHAR(MAX)    NOT NULL,
    DateOfBirth              NVARCHAR(MAX)    NOT NULL,
    EmployeeStartDate        NVARCHAR(MAX)    NOT NULL,
    ManagerID                NVARCHAR(MAX)    NULL,
    Salary                   NVARCHAR(MAX)    NOT NULL,
    Department               NVARCHAR(MAX)    NOT NULL,
    DepartmentCode           NVARCHAR(MAX)    NOT NULL,
    Role                     NVARCHAR(MAX)    NOT NULL,
    WeeklyContractedHours    NVARCHAR(MAX)    NOT NULL,
    StaffOrContract          NVARCHAR(MAX)    NOT NULL,
    ContractEndDate          NVARCHAR(MAX)    NULL
) ;
```

Dlaczego więc wybór prawidłowego typu danych jest tak ważny? Większość osób, które pracowały z SQL Server przez jakiś czas, rozumie znaczenie ograniczeń. Ograniczenia te występują w wielu postaciach, takich jak klucze obce, ograniczenia sprawdzające i ograniczenia NULL.

Ograniczenia są niezbędne do zapewnienia jakości danych w bazie danych. Na przykład klucz obcy zapewni, że wartość istnieje w innej tabeli przed jej wstawieniem lub aktualizacją. Ograniczenie `NOT NULL` wymusza, aby wartość w kolumnie była obowiązkowa i kolumna nie mogła zawierać wartości NULL. Ograniczenie sprawdzające zapewnia, że wartości w kolumnie spełniają określone kryteria. Na przykład zapewnienie, że data rozpoczęcia pracy pracownika nie jest wcześniejsza niż jego 16. urodziny.

Wiele osób nie bierze jednak pod uwagę tego, że typ danych jest również ograniczeniem. Ograniczenie mające zastosowanie do każdej kolumny w każdej tabeli w każdej pojedynczej bazie danych. To podstawa jakości danych, ale także funkcjonalności. Dodatkowo może odegrać rolę w zapewnieniu, że nasz kod jest samodokumentujący. Kod samodokumentujący został omówiony bardziej szczegółowo w rozdziale 2.

Na przykładzie naszej tabeli `Employees` można zauważyć kilka bezpośrednich problemów. Po pierwsze, nie możemy utworzyć ograniczenia klucza podstawowego, ponieważ klucze podstawowe nie są obsługiwane w kolumnach `NVARCHAR(MAX)`. Po drugie, obliczenia kolumn dat będą kłopotliwe i będą wymagały konwersji. Daty można również wstawiać w sprzecznych formatach. Na przykład 13.01.2023 i 13.01.2023. Po trzecie, do dowolnej kolumny możemy wstawić dowolne dane. Możemy wstawić datę do kolumny Wynagrodzenie, tekst do kolumn oczekujących wartości liczbowych lub daty do deskryptorów tekstowych, takich jak Imię, Nazwisko i Dział. Wreszcie, ponieważ w każdej kolumnie i każdym wierszu możemy przechowywać do 2 GB danych, teoretycznie tabela ta mogłaby szybko stać się bardzo duża.

Powinniśmy natychmiast rozwiązać te problemy, a skrypt z Listingu 3.2 może je rozwiązać, usuwając i ponownie tworząc tabelę. Zmieni kolumny numeryczne na INT, kolumny daty na DATE i zmniejszy kolumny deskryptorów tekstowych do odpowiedniej długości. Tabela doda także klucz podstawowy do kolumny EmployeeID.

Listing 3.2 Aktualizacja typów kolumn tabeli pracowników

```sql
 
DROP TABLE dbo.Employees ;
GO
 
CREATE TABLE dbo.Employees (
    EmployeeID             INT          NOT NULL  PRIMARY KEY,
    FirstName              NVARCHAR(32) NOT NULL,
    LastName               NVARCHAR(32) NOT NULL,
    DateOfBirth            DATE         NOT NULL,
    EmployeeStartDate      DATE         NOT NULL,
    ManagerID              INT          NULL,
    Salary                 MONEY        NOT NULL,
    Department             NVARCHAR(64) NOT NULL,
    DepartmentCode         NVARCHAR(4)  NOT NULL,
    Role                   NVARCHAR(64) NOT NULL,
    WeeklyContractedHours  INT          NOT NULL,
    StaffOrContract        INT          NOT NULL,
    ContractEndDate        DATE         NULL
) ;
 
```

> NOTATKA
>
> Chociaż nowo wybrane typy danych są znacznie bardziej przydatne niż zwykła wartość NVARCHAR(MAX), nadal są one dalekie od doskonałości. Zagadnienia te omówimy w kolejnych sekcjach tego rozdziału.

### 3.1 #6 Zawsze przechowuj liczby całkowite w INT
Wyobraź sobie, że mamy dużą hurtownię danych. Jedna z tabel faktów ma miliard wierszy i łączy się z pięcioma wymiarami, z których każdy ma 30 000 wierszy. Wydajność jest niska, a pamięć jest zawsze zapełniona ze względu na ilość danych w buforze pamięci podręcznej. Podczas wykonywania zapytań duża ilość danych jest buforowana do TempDB. Zoptymalizowaliśmy zapytania. Dokonaliśmy także przeglądu naszej strategii indeksowania i zadbaliśmy o to, aby zarówno indeksy, jak i statystyki były dobrze utrzymywane. Wygląda na to, że jedyne, co można zrobić, to zaradzić problemowi większą ilością sprzętu, jednak na podstawie naszej tendencji obserwowanej przez ostatnie dwa lata podejrzewamy, że jeśli dodamy więcej pamięci RAM, tylko popchniemy problem dalej. Co powinniśmy zrobić? Punktem wyjścia byłoby rozważenie przeglądu naszych numerycznych typów danych. Zwłaszcza te używane w relacjach klucz podstawowy/obcy.

INT jest najczęściej używanym, ale także najczęściej nadużywanym typem danych w SQL Server. W rzeczywistości mamy cztery typy danych przeznaczone konkretnie do przechowywania liczb całkowitych. Te typy danych szczegółowo opisano w tabeli 3.1.

| Data Type | Range                                                   | Size (Bytes) |
| --------- | ------------------------------------------------------- | ------------ |
| TINYINT   | 0 to 255                                                | 1            |
| SMALLINT  | -32,768 to 32,767                                       | 2            |
| INT       | -2,147,483,648 to 2,147,483,647                         | 4            |
| BIGINT    | -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807 | 8            |

Możemy to również sprawdzić w kodzie, uruchamiając zapytanie z aukcji 3.3. To zapytanie używa funkcji CAST() do konwersji wartości 1 na każdy z typów danych całkowitych. Ta przekonwertowana wartość jest następnie przekazywana do funkcji DATALENGTH(), która oblicza rozmiar wartości wejściowej w bajtach.

```sql
SELECT 
      DATALENGTH(CAST(1 AS TINYINT)) AS TinyIntSize
    , DATALENGTH(CAST(1 AS SMALLINT)) AS SmallIntSize
    , DATALENGTH(CAST(1 AS INT)) AS IntSize
    , DATALENGTH(CAST(1 AS BIGINT)) AS BigIntSize ;
```

Wyniki tego zapytania pokazano poniżej:

```swift
TinyIntSize    SmallIntSize    IntSize    BigIntSize
1              2               4          8
```

Wyobraź sobie, że nasze pięciowymiarowe tabele używają INT jako kolumny klucza podstawowego. Każda tabela wymiarów zawiera 30 000 wierszy, a typ danych SMALLINT może zawierać ponad 32 000 wartości dodatnich. Oznacza to, że gdybyśmy nie spodziewali się dramatycznego wzrostu wymiarów, moglibyśmy zaoszczędzić dwa bajty w każdym wierszu.

> WSKAZÓWKA
>
> W rzeczywistości w zakresie SMALLINT istnieje ponad 64 000 możliwych wartości, ale aby użyć ich więcej niż 32 000, musielibyśmy rozpocząć naszą sekwencję numerowania od -32 000. To może nie pasować do zasady najmniejszej niespodzianki.

W tym momencie możesz pomyśleć: „Dlaczego zawracamy sobie głowę oszczędzaniem dwóch bajtów?” Odpowiedź na to pytanie wymaga prostej matematyki. W każdej z pięciu tabel wymiarów znajduje się 30 000 wierszy. Zatem przechodząc na SMALLINT zaoszczędzilibyśmy jedynie 58 KB na tabelę. Ale nasza tabela faktów ma miliard wierszy. Oznacza to, że zaoszczędzilibyśmy 1,86 GB na kluczu. Pomnóż to przez pięć tabel wymiarów, a to oznacza, że dla każdego zapytania dotyczącego tabeli faktów, które dotyczy wszystkich wierszy i wszystkich pięciu kluczy, zaoszczędzimy 9,3 GB. Skaluj to na podstawie ośmiu tabel faktów w hurtowni danych. Powinniśmy również wziąć pod uwagę wielkość indeksów zbudowanych na tych kolumnach klucza obcego. Rozważmy teraz sesje równoległe uruchamiające różne zapytania. Nagle nasz wybór typu danych ma bezpośredni i namacalny wpływ na zużycie pamięci.

Jak to wszystko ma się do naszej tabeli Pracownicy? Udokumentujmy naszą tabelę w formie diagramu relacji encji (ERD). ERD to diagram przedstawiający jednostki danych (tabele). Pokazuje relacje między tymi jednostkami (ograniczenie klucza podstawowego/obcego) i szczegółowo opisuje atrybuty (kolumny) jednostki. Opcjonalnie może również szczegółowo określić typ danych każdej kolumny.

Podczas budowania diagramów C4 dla aplikacji warstwy danych często używa się ERD jako diagramu kodu, który dokumentuje strukturę tabeli. ERD na rysunku 3.1 przedstawia obecnie tylko encję naszych pracowników, ale będziemy na niej opierać się w rozdziale 4. Diagram rejestruje aktualnie zdefiniowane typy danych, ale został opatrzony adnotacją objaśniającą wartości, których oczekujemy w każdej kolumnie.

Figure 3.1 Employees ERD

![img](https://drek4537l1klr.cloudfront.net/carter/v-1/Figures/03__image001.png)

Kolumny `EmployeeID` i `ManagerID` będą wymagały maksymalnie 300 unikalnych wartości. Dlatego typ danych TINYINT będzie zbyt restrykcyjny. Moglibyśmy jednak użyć typu danych SMALLINT. Dzięki temu zaoszczędzimy dwa bajty na wiersz w porównaniu z aktualnie posiadanym typem danych INT.

Ciekawym przypadkiem jest kolumna `StaffOrContract`. Będzie przechowywać wartości całkowite, ale tylko w zakresie od 0 do 1. Wprowadza to w grę dodatkowy typ danych, o którym jeszcze nie rozmawialiśmy. Mianowicie BIT. Typ danych BIT jest technicznie typem danych całkowitym, ale może przechowywać tylko wartości 0, 1 i NULL. Jest przeznaczony do przechowywania wartości logicznych, takich jak flagi, i posiada przydatne dodatkowe funkcje.

Jeśli użytkownik wstawi „`TRUE`” lub „`FALSE`” do kolumny `BIT`, SQL Server automatycznie przekonwertuje je odpowiednio na 0 i 1. Dodatkowo, jeśli użytkownik wstawi jakąkolwiek wartość liczbową inną niż 0 lub 1, SQL Server automatycznie przekonwertuje tę wartość na 1. Na przykład, jeśli uruchomimy zapytanie `SELECT CAST(86.2 AS BIT)`, zwróci ono wartość 1.

Dodatkowo, jeśli tabela zawiera wiele kolumn `BIT`, SQL Server optymalizuje ich przechowywanie. Pierwsze osiem kolumn BIT zajmuje tylko jeden bajt miejsca. Kolejne osiem kolumn wykorzystuje dodatkowy bajt i tak dalej. Kolumna StaffOrContract jest idealnym przypadkiem użycia dla typu danych BIT i pozwala zaoszczędzić trzy bajty w każdym wierszu.

Ponieważ nie mamy jeszcze danych w tabeli `employees`, usuńmy ją i utwórzmy ponownie, używając naszych preferowanych typów danych całkowitych. Możemy to zrobić za pomocą skryptu z Listingu 3.4.

```sql
DROP TABLE dbo.Employees
GO
 
CREATE TABLE dbo.Employees (
    EmployeeID             SMALLINT      NOT NULL  PRIMARY KEY,
    FirstName              NVARCHAR(32)  NOT NULL,
    LastName               NVARCHAR(32)  NOT NULL,
    DateOfBirth            DATE          NOT NULL,
    EmployeeStartDate      DATE          NOT NULL,
    ManagerID              SMALLINT      NULL,
    Salary                 MONEY         NOT NULL,
    Department             NVARCHAR(64)  NOT NULL,
    DepartmentCode         NVARCHAR(4)   NOT NULL,
    Role                   NVARCHAR(64)  NOT NULL,
    WeeklyContractedHours  INT           NOT NULL,
    StaffOrContract        BIT           NOT NULL,
    ContractEndDate        DATE          NULL
) ;
```

Zawsze powinniśmy mieć na uwadze ilość miejsca wymaganego przez nasze typy danych. Jest to szczególnie prawdziwe, jeśli tabela jest bardzo duża lub jeśli kolumna będzie używana w indeksie. Zamiast INT należy używać SMALLINT i TINYINT, jeśli nie oczekujemy, że wartości przekroczą te rozmiary, aby zmniejszyć marnowanie miejsca.

### 3.2 #7 Zawsze używaj ciągów o zmiennej długości
Wyobraź sobie, że mamy tabelę przechowującą adresy. Prawidłowo staramy się mieć pewność, że nasze dane zajmują jak najmniej miejsca. Dlatego we wszystkich kolumnach używamy ciągów o zmiennej długości, które zawierają każdą linię adresu i kod pocztowy. Wiemy, że Mooselookmeguntic w Maine i Kleinfeltersville w Pensylwanii mają najdłuższe nazwy miast w USA, każda zawierająca 17 znaków, dlatego ustawiliśmy kolumnę CityName na `VARCHAR(17)`. Wiemy, że najdłuższa nazwa stanu w USA to State of Rhode Island i Providence Plantations, dlatego ustawiliśmy kolumnę State na `VARCHAR(48)`. Wiemy, że kod pocztowy będzie miał dokładnie dziesięć znaków, ale ponieważ wiemy, że będzie miał za każdym razem tę samą długość, powinniśmy więc użyć `VARCHAR(10)` lub `CHAR(10)`. Czy to w ogóle ma znaczenie? Tak czy inaczej, dane mają długość dziesięciu znaków, więc zajmą dziesięć bajtów miejsca, prawda? A jeśli to prawda, dlaczego w ogóle mamy ciągi o stałej długości? Faktem jest, że założenie to nie jest prawidłowe. Aby zrozumieć dlaczego, musimy trochę zrozumieć, w jaki sposób SQL Server przechowuje dane.

> NOTATKA
>
> Łańcuchy o stałej długości, `CHAR` i `NCHAR` uzupełniają ciąg białymi znakami, jeśli nie został wypełniony danymi. Na przykład, jeśli mamy `CHAR(8)` i wstawimy wartość „Hello!”, wówczas zajmie to osiem bajtów pamięci, ponieważ dwa znaki zapasowe zostaną wypełnione spacją.

SQL Server przechowuje dane w serii stron o rozmiarze 8 KB, przy czym każda seria ośmiu stron tworzy zakres 64 KB, co zwykle stanowi najmniejszą ilość odczytywanych danych. Każda strona danych ma 96-bajtowy nagłówek strony, w którym przechowywane są informacje dotyczące całej strony, takie jak jej unikalny identyfikator i identyfikator obiektu tabeli (lub indeksu), do której należy.

Dane tworzące wiersze są następnie przechowywane w slotach na stronie. Jednak te sloty służą nie tylko do przechowywania danych. Muszą także przechowywać niewielką ilość metadanych, aby dane były przydatne. Te metadane obejmują informacje o typie rekordu przechowywanego w tym slocie. Na przykład, czy jest to rekord danych, czy rekord indeksu? Czy obejmuje dane, które zostały logicznie usunięte, ale jeszcze nie zostały usunięte fizycznie?

Inne metadane obejmują długość danych o stałej długości (dotyczy to nie tylko danych znakowych o stałej długości, ale także danych takich jak liczby całkowite), mapę bitową o wartości NULL, która śledzi, czy kolumny o zmiennej długości zawierają wartości NULL, oraz znacznik wersji, czyli wykorzystywane przez operacje takie jak odbudowa indeksu online lub transakcje z optymistycznym poziomem izolacji transakcji. Poziomy izolacji omówimy w rozdziale 10.

Jednak metadanymi, które nas tutaj naprawdę interesują, jest tablica przesunięć kolumn **column offset array**. Służy do śledzenia, gdzie zaczyna się każda kolumna o zmiennej długości w wierszu. Ponieważ dane o zmiennej długości mogą mieć dowolną długość, ta tabela przesunięć jest dla SQL Server jedynym sposobem na oddzielenie miejsca, w którym kończy się jeden fragment danych, a zaczyna następny.

Każda kolumna o zmiennej długości wymaga w tej tabeli dwubajtowego przesunięcia, co oznacza, że każda kolumna o zmiennej długości zajmuje o dwa bajty więcej miejsca niż w przypadku kolumny o stałej długości. Ma to zastosowanie nawet wtedy, gdy kolumna przechowuje wartość NULL. Dlatego gdybyśmy użyli CHAR(10) dla naszego kodu pocztowego, zajęłoby to dziesięć bajtów miejsca, ale gdybyśmy użyli VARCHAR(10), zajęłoby 12 bajtów miejsca, mimo że rzeczywista długość danych wynosi dziesięć bajtów .

> `CHAR` I `VARCHAR` VS `NCHAR` I `NVARCHAR`
>
> `NCHAR` i `NVARCHAR` mogą przechowywać pełne dane `UNICODE`, podczas gdy `CHAR` i `VARCHAR` mogą przechowywać tylko 8-bitową stronę kodową. Od SQL Server 2019 możliwe było używanie sortowania z obsługą UTF-8, a typy danych `CHAR`/`VARCHAR` mogą przechowywać pełny zakres znaków UTF-8. Jednak do pełnej obsługi UTF-16 nadal wymagane są typy danych `NCHAR` i `NVARCHAR`.
>
> Typy danych `CHAR` i `VARCHAR` używają jednego bajtu na znak. `NCHAR` i `NVARCHAR` używają dwóch bajtów na znak. Dodatkowa przestrzeń jest wymagana dla pełnej 16-bitowej strony kodowej UTF-16. Dlatego typy danych UTF-16 nie tylko zajmują więcej miejsca, ale także ograniczają liczbę znaków, które można przechowywać na stronie. SQL Server narzuca maksymalną długość danych UTF-16 o stałej długości wynoszącą 4000 znaków, w przeciwieństwie do maksymalnej długości 8-bitowych danych strony kodowej o stałej długości, która wynosi 8000 znaków.
>
> Możliwe jest przechowywanie danych o zmiennej długości do 2 GB przy użyciu typów danych `VARCHAR(MAX)` i `NVARCHAR(MAX)`, ale wszelkie pola, które nie mieszczą się na stronie danych, są przechowywane w innym typie jednostki alokacji i nazywane są jako dane dotyczące przepełnienia wierszy.
>
> Ograniczenie rozmiaru dotyczy również wielu kolumn. Niezależnie od rozmieszczenia kolumn, maksymalna długość danych na stronie wynosi 8060 bajtów. Dlatego też, jeśli mamy tabelę składającą się z kolumn `CHAR(5000)` i `VARCHAR(5000)`, a dane zostaną wstawione do kolumny o zmiennej długości, która ma długość 2000 znaków (4000 bajtów), wówczas zmienna- dane dotyczące długości zostaną dynamicznie przeniesione na inną stronę i zapisane w jednostce alokacji przepełnienia wiersza.
>
> Niektórzy argumentują, że ponieważ przechowywanie i pamięć są stosunkowo tańsze niż kiedyś, powinniśmy po prostu używać typów danych `NCHAR` i `NVARCHAR`, aby uniknąć potencjalnych problemów z niezgodnością stron kodowych między kolumnami. Mój pogląd na ten temat, wzmocniony przez dodanie zestawień UTF-8, jest taki sam, jak w przypadku każdego innego typu danych. Powinniśmy używać najbardziej restrykcyjnego typu danych, który nie spowoduje przepełnienia.
>
> Innymi słowy, jeśli wiemy, że ze względu na naturę naszych danych, że nigdy nie napotkamy żadnych problemów ze zgodnością strony kodowej, powinniśmy używać 8-bitowych typów danych strony kodowej. Ostatecznie zapewni nam to najlepszą wydajność i efektywne wykorzystanie zasobów w najlepszej cenie. Jeśli jednak nie można tego zagwarantować, na przykład jeśli będziemy musieli pobrać dane z Internetu lub innych niezaufanych źródeł, lub w rzeczywistości, jeśli wiemy, że może zaistnieć potrzeba wymieszania danych z wielu zestawień, wówczas powinniśmy skorzystać z typów danych zgodnych z  UTF-16.

W kontekście naszej tabeli `Employees` większość naszych kolumn absolutnie musi mieć zmienną długość. Jednak nasza kolumna DepartmentCode będzie zawsze zawierać wartość dwuznakową, np. HR dla zasobów ludzkich, SA dla sprzedaży i MA dla produkcji. Dlatego powinniśmy zmienić definicję kolumny DepartmentCode na NCHAR(2). Pozwoli to zaoszczędzić dwa bajty informacji w każdym wierszu. Używamy Unicode, ponieważ w sekcji 3.4 będziemy pobierać dane od zewnętrznego partnera biznesowego.

Instrukcja na liście 3.5 dokonuje żądanej aktualizacji tabeli `Employees` za pomocą polecenia `ALTER TABLE..ALTER COLUMN`.

Listing 3.5 Zaktualizuj kolumnę DepartmentCode do łańcucha o stałej długości

```sql
ALTER TABLE dbo.Employees
    ALTER COLUMN DepartmentCode NCHAR(2) ;
```

Używanie ciągów o zmiennej długości jest właściwym rozwiązaniem, jeśli wartości w naszej kolumnie mogą mieć różną długość. Pozwala to uniknąć dopełniania krótszych wartości białymi znakami i oszczędza miejsce. Jeśli jednak spodziewamy się, że wartości kolumny będą zawsze tej samej długości, powinniśmy użyć łańcucha o stałej długości. Niezastosowanie się do tego spowoduje dodanie dodatkowych 2 bajtów w każdym wierszu, które zostaną wykorzystane dla tablicy przesunięć kolumn, określającej początek każdej wartości o zmiennej długości.

### 3.3 #8 Tworzenie własnego kodu hierarchii
Jeśli spojrzymy na naszą tabelę Employees, prawdopodobnie zdaliśmy sobie sprawę, że kolumny `EmployeeID` i `ManagerID` służą do modelowania hierarchii pracowników. Dlatego też rozważ schemat organizacyjny przedstawiony na rysunku 3.2, który modeluje strukturę organizacyjną kadry kierowniczej wyższego szczebla w MagicChoc.

##### Figure 3.2 MagicChoc management org chart

![img](https://drek4537l1klr.cloudfront.net/carter/v-1/Figures/03__image002.png)

Aby zademonstrować jak modelować tę hierarchię przy wykorzystaniu tradycyjnego podejścia, z którego wciąż korzysta zaskakująca liczba programistów, zapraszam do uruchomienia skryptu z Listingu 3.6, który wstawi rekordy pracowników do tabeli Pracownicy. Zwróć uwagę, że kolumna `ManagerID` zawiera identyfikator pracownika osoby, której podlega.

```sql
INSERT INTO dbo.Employees (
    EmployeeID
  , FirstName
  , LastName
  , DateOfBirth
  , EmployeeStartDate
  , ManagerID
  , Salary
  , Department
  , DepartmentCode
  , Role
  , WeeklyContractedHours
  , StaffOrContract
  , ContractEndDate
)
VALUES 
    (1, 'Simon', 'Gomez', '19691001', '20180101', NULL, 980000, 'C-Suite', 'CS', 'CEO', 40, 1, NULL),
    (2, 'Sanjay', 'Gupta', '19761001', '20180101', 1, 640000, 'C-Suite', 'CS', 'COO', 40, 1, NULL),
    (3, 'Ed', 'Ling', '19690403', '20200801', 1, 320000, 'C-Suite', 'CS', 'CPO', 40, 1, NULL),
    (4, 'Amanda', 'Ballard', '19830401', '20200301', 1, 350000, 'C-Suite', 'CS', 'Sales Director', 40, 1, NULL),
 
    (5, 'Bob', 'Walford', '19780908', '20191201', 2, 96000, 'Technology', 'TE', 'Head Of Technology', 40, 1, NULL),
    (6, 'Brian', 'Tilly', '19710102', '20181001', 2, 89000, 'Manufacturing', 'MA', 'Head Of Manufacturing', 40, 1, NULL),
    (7, 'Sally', 'Nugent', '19790302', '20220601', 2, 80000, 'Procurement', 'PR', 'Head Of Procurement', 40, 1, NULL),
    (8, 'Jamie', 'Briggs', '19900102', '20190601', 3, 65000, 'Human Resources', 'HR', 'HR Manager', 40, 1, NULL),
    (9, 'Lance', 'Bernard', '19910707', '20210601', 4, 98000, 'Sales & Marketing', 'SA', 'Head Of Sales', 40, 1, NULL),
    (10, 'Jo', 'Carver', '19900810', '20191201', 4, 70000, 'Sales & Marketing', 'SA', 'Head Of Marketing', 40, 1, NULL),
 
    (11, 'John', 'O''Shea', '19700609', '20180601', 5, 70000, 'Technology', 'TE', 'Development Manager', 40, 1, NULL),
    (12, 'Eric', 'Bristow', '20000109', '20221001', 5, 72000, 'Technology', 'TE', 'Infrastructure Manager', 40, 1, NULL),
    (13, 'Ronald', 'Sanders', '19601209', '20190101', 6, 45000, 'Manufacturing', 'MA', 'Shift Manager', 45, 1, NULL),
    (14, 'Amy', 'Fry', '19921101', '20190101', 6, 45000, 'Manufacturing', 'MA', 'Shift Manager', 45, 1, NULL),
    (15, 'Greg', 'Andrews', '19871212', '20190101', 6, 45000, 'Manufacturing', 'MA', 'Shift Manager', 45, 1, NULL),
    (16, 'Dave', 'Turney', '19760609', '20190101', 6, 52000, 'Manufacturing', 'MA', 'Warehouse Manager', 48, 1, NULL),
    (17, 'Mark', 'Sokolowski', '19960209', '20190901', 16, 42000, 'Manufacturing', 'MA', 'Goods Inn Manager', 40, 1, NULL),
 
    (18, 'Robin', 'Round', '19940409', '20190601', 3, 60000, 'Human Resources', 'HR', 'Recruitment Manager', 40, 1, NULL),
    (19, 'Lucy', 'Sykes', '19890201', '20200201', 9, 65000, 'Sales', 'SA', 'US Sales Manager', 40, 1, NULL),
    (20, 'Bruce', 'Bryant', '19860304', '20200301', 9, 70000, 'Sales', 'SA', 'International Sales Manager', 40, 1, NULL),
    (21, 'Ashwin', 'Kumar', '20010212', '20210601', 20, 55000, 'Sales', 'SA', 'Euro Sales Manager', 40, 1, NULL),
    (22, 'Emma', 'Roberts', '20000208', '20210601', 20, 55000, 'Sales', 'SA', 'APAC Sales Manager', 40, 1, NULL) ;
```



A teraz wyobraźcie sobie, że poproszono nas o napisanie raportu zawierającego listę wszystkich bezpośrednich podwładnych Sanjaya Gupty. Można to osiągnąć za pomocą następującego prostego zapytania:

```sql
SELECT 
    FirstName
  , LastName
FROM dbo.Employees
WHERE ManagerID = 2 ;
```

Ale co, jeśli zostaniemy poproszeni o napisanie zapytania, które zwróci wszystkie bezpośrednie i pośrednie raporty Sanjaya Gupy? To nagle staje się bardziej złożone. Istnieje kilka sposobów tworzenia tego raportu, w tym przerażające kursory (które omówimy w rozdziale 5), ale najlepszą metodą dla programistów byłoby użycie rekurencyjnego wspólnego wyrażenia tabelowego (CTE). CTE to tymczasowy zestaw wyników, do którego można odwoływać się wielokrotnie w zapytaniu. Może również odwoływać się do samego siebie, co pozwala na rekurencję.

> NOTATKA
>
> Rekurencja oznacza, że błąd w zapytaniu może spowodować nieskończoną pętlę. Aby tego uniknąć, istnieje ustawienie obejmujące całą instancję, które kontroluje maksymalny poziom rekurencji. Domyślnie jest to ustawione na 100. Jeśli chcemy zastąpić tę wartość w przypadku konkretnego zapytania, możemy skorzystać ze wskazówki dotyczącej zapytania MAXRECURSION.

Rekurencja jest kluczem do wdrożenia hierarchii w tym scenariuszu. Na przykład zapytanie na liście 3.7 używa CTE do zwrócenia wszystkich pracowników, którzy podlegają bezpośrednio i pośrednio Sanjayowi Gupcie, którego identyfikator pracownika wynosi 2. Zgodnie z definicją CTE, pierwsze zapytanie zwraca rekord pracownika samego Sanjaya. Następnie klauzula UNION ALL dołącza wyniki drugiego zapytania. To drugie zapytanie jest rekurencyjne, ponieważ łączy wyniki z tabeli Pracownicy z wynikami pierwszego zapytania w obrębie CTE. Ponieważ połączenie odbywa się na identyfikatorze pracownika w pierwszym zestawie wyników i identyfikatorze menedżera w drugim zestawie wyników, drugi zestaw wyników zawiera szczegółowe informacje o podległych pracownikach. Końcowa instrukcja SELECT znajduje się poza CTE. Zwraca wszystkie rekordy z CTE, ale następnie łączy się z bazową tabelą Pracownicy, aby wypełnić imię i nazwisko Menedżera.



> WSKAZÓWKA
>
> Zauważysz, że średnik kończący instrukcję SET znajduje się na początku linii rozpoczynającej klauzulę WITH CTE. To wybór stylistyczny. CTE musi zawsze być początkiem instrukcji. Dlatego powszechną praktyką jest rozpoczynanie instrukcji zawsze średnikiem. To zatrzyma błąd kompilacji, nawet jeśli zapomnieliśmy zakończyć poprzednią instrukcję. Alternatywnie możesz wymusić standard, zgodnie z którym wszystkie instrukcje są zawsze kończone średnikiem.

```sql
DECLARE @ManagerID INT ;
 
SET @ManagerID = 2
 
;WITH EmployeeCTE AS (
    SELECT 
          EmployeeId
        , FirstName
        , LastName
        , ManagerId
    FROM dbo.Employees 
    WHERE EmployeeId = @ManagerID
    UNION ALL
    SELECT 
          Emp.EmployeeId
        , Emp.FirstName
        , Emp.LastName
        , Emp.ManagerId 
    FROM dbo.Employees AS Emp
    INNER JOIN EmployeeCTE AS CTE 
        ON CTE.EmployeeId=Emp.ManagerId
)
SELECT 
      Emp.EmployeeID
    , Emp.FirstName
    , Emp.LastName
    , Emp.ManagerID
    , Mgr.FirstName AS ManagerFirstName
    , Mgr.LastName AS ManagerLastName
FROM EmployeeCTE Emp
INNER JOIN Employees Mgr 
    ON Emp.ManagerID = Mgr.EmployeeID ; 
```

Innym typowym zapytaniem, o które możemy zostać poproszeni, jest ustalenie, kto zarządza menadżerem pracownika. Jest to częste zjawisko podczas konstruowania ścieżek eskalacji. Zapytanie na aukcji 3.8 pokazuje, w jaki sposób możemy ustalić, kto zarządza menedżerem Emmy Robert. W tym zapytaniu dodajemy stałą 0 w pierwszym zapytaniu w obrębie CTE, z nazwą kolumny Level do zestawu wyników i określamy, że Emma Roberts jest na poziomie 0. Następnie zapytanie rekurencyjne zwiększa numer poziomu dla każdej warstwy hierarchia. Ostateczne zapytanie, poza CTE, jest następnie filtrowane według poziomu 2, aby zwrócić jednostkę dwa poziomy powyżej Emmy.

```sql
DECLARE @EmployeeID INT ;
 
SET @EmployeeID = 22
 
; WITH EmployeeCTE AS  
(  
   SELECT 
       employeeid
     , firstname
     , lastname
     , managerid
     , Role
     , 0 as Level
   FROM dbo.Employees 
   WHERE EmployeeID = @EmployeeID
   UNION ALL  
   SELECT 
       emp.EmployeeID
     , emp.FirstName
     , emp.LastName
     , emp.ManagerID
     , emp.Role
     , Level + 1
   FROM dbo.employees emp
   INNER JOIN EmployeeCTE cte 
      ON emp.EmployeeID = cte.ManagerID  
)
 
SELECT 
    firstname
  , lastname
  , Role 
FROM EmployeeCTE 
WHERE Level = 2 ;
```

Wyzwaniem związanym z tym tradycyjnym podejściem do implementowania hierarchii jest to, że wymaga od programisty napisania raczej dużej ilości kodu. Przykłady w tej sekcji są proste i służą celom ilustracyjnym, ale w rzeczywistych scenariuszach rekurencyjne CTE mogą stać się złożone i trudne w zarządzaniu. Za każdym razem, gdy firma otrzymuje nieco inne żądanie, programiści muszą napisać kod, który będzie w odpowiedni sposób przechodził przez hierarchię. Na przykład możemy zostać poproszeni o zwrócenie listy wszystkich menedżerów na trzecim poziomie struktury organizacyjnej.

Wielu programistów nie zdaje sobie sprawy, że Microsoft wykonał już za nas większość ciężkiej pracy, implementując typ danych HIERARCHYID. Jest to zaawansowany typ danych, napisany w .NET i pozwalający na wywołanie przeciwko niemu metod, które za nas przemierzą hierarchię, bez konieczności pisania skomplikowanych CTE.

Aby zobaczyć, jak to działa, dodajmy nową kolumnę do tabeli Pracownicy o nazwie ManagerHierarchyID, która będzie typu HIERARCHYID. Możemy to osiągnąć za pomocą skryptu z Listingu 3.9.

Listing 3.9 Dodaj kolumnę Manager HierarchyID

```sql
ALTER TABLE dbo.Employees ADD
    ManagerHierarchyID HIERARCHYID NULL ;
```

Aby skorzystać z funkcji hierarchii SQL Server, musimy najpierw modelować hierarchię. Aby wyjaśnić, jak działa to modelowanie, zapoznaj się z następującą sekcją naszego schematu organizacyjnego. Simon Gomez znajduje się na szczycie hierarchii, którą nazwiemy korzeniem.

Hierarchia jest modelowana przy użyciu formatu ukośnego. Dlatego korzeń hierarchii jest reprezentowany jako /. Sanjay Gupta i Ed Ling znajdują się na drugim poziomie hierarchii i każdy z nich wymaga jednoznacznej identyfikacji. Dlatego Sanjay będzie reprezentowany jako `/1/`, a Ed będzie reprezentowany jako `/2/`

Zarówno Jamie Briggs, jak i Robin Round podlegają Edowi Lingowi. Dlatego obaj znajdują się na trzecim poziomie hierarchii, ale też muszą być jednoznacznie zidentyfikowani. Dlatego byłyby one reprezentowane odpowiednio jako `/1/2/1/` i `/1/2/2/`. Od tego momentu poziomy są nadal budowane. Są one następnie zapisywane w tabeli jako ciągi bitów i wyświetlane, niesformatowane, jako wartości szesnastkowe. Na przykład identyfikator hierarchii Simona Gomeza na poziomie głównym jest przechowywany jako 0x, podczas gdy Sanjay Gupta na drugim poziomie jest przechowywany jako `0x58`, a Ed Ling, również na drugim poziomie, jest przechowywany jako `0x68`. Jamie Briggs i Robin Round są przechowywane odpowiednio jako `0x6AC0` i `0x6B40`.

Ale nie bój się. Nie ma potrzeby ręcznego modelowania hierarchii. Zamiast tego zastosujemy podejście dwuetapowe, które eliminuje ciężką pracę z procesu modelowania. Pierwszym krokiem będzie utworzenie tabeli tymczasowej, która będzie składać się z trzech kolumn; `EmployeeID`, `ManagerID` oraz numer wiersza, który wygenerujemy korzystając z funkcji `ROW_NUMBER()` i dzieląc liczby według `ManagerID`. Ta funkcja da nam potrzebne liczby przyrostowe na każdym poziomie gałęzi hierarchii.

Drugim krokiem jest zdefiniowanie zapytania CTE zbudowanego na naszej tabeli tymczasowej, który zdefiniuje pierwszy element (root) hierarchii za pomocą globalnej metody `GetRoot()` względem typu `HIERARCHYID` dla Simona Gomeza, który znajduje się na szczycie hierarchii, a zatem ma `NULL` `ManagerID`.

Zapytanie rekurencyjne zbuduje kolejne poziomy hierarchii poprzez połączenie kolejnego, kolejnego poziomu hierarchii z wartością przyrostową wygenerowaną za pomocą `ROW_NUMBER()`. Następnie możemy użyć instrukcji `UPDATE`, aby pobrać modelowane identyfikatory hierarchii z CTE i zaktualizować tabelę `Employees` w oparciu o dołączenie do identyfikatora pracownika `EmployeeID`. Pokazano to na Listingu 3.10.

```sql
SELECT    
    EmployeeID
  , ManagerID
  , ROW_NUMBER() OVER (PARTITION BY ManagerID ORDER BY ManagerID) AS Incremental
INTO #Hierachy
FROM dbo.Employees 
 
;WITH HierarchyPathCTE AS (  
    SELECT 
         hierarchyid::GetRoot() AS ManagerHierarchyID
       , EmployeeID   
    FROM #Hierachy AS C   
    WHERE ManagerID IS NULL   
    UNION ALL   
    SELECT   
        CAST(hpc.ManagerHierarchyID.ToString() + CAST(h.Incremental AS VARCHAR(30)) + '/' AS HIERARCHYID)   
       ,h.EmployeeID  
FROM #Hierachy AS h 
JOIN HierarchyPathCTE AS hpc   
   ON h.ManagerID = hpc.EmployeeID   
)  
 
UPDATE e
    SET ManagerHierarchyID = hp.ManagerHierarchyID
FROM dbo.Employees e
INNER JOIN HierarchyPathCTE hp
    ON e.EmployeeID = hp.EmployeeID ;
```

> ROZRÓŻNIANIE WIELKOŚCI LITER
>
> Ważną informacją podczas pracy z typem danych HIERARCHYID jest to, że w metodach uwzględniana jest wielkość liter. Na przykład poniższe zapytanie zwróci błąd:
>
> ```sql
> SELECT ManagerHierarchID.tostring() FROM dbo.Employees ;
> ```
>
> Natomiast następujące zapytanie zostałoby wykonane pomyślnie:
>
> ```sql
> SELECT ManagerHierarchID.ToString() FROM dbo.Employees ;
> ```
>
> 

Od tego momentu praca z hierarchiami staje się niezwykle prosta. Przypomnij sobie zapytanie, które napisaliśmy, aby wygenerować listę wszystkich osób, które podlegają bezpośrednio i pośrednio Sanjayowi Gupcie. Możemy zastąpić całe to zapytanie rekurencyjne prostym zapytaniem z Listingu 3.11, które wykorzystuje metodę `IsDescendantOf()` do filtrowania zapytania poprzez identyfikację każdego, kto znajduje się w gałęzi hierarchii Sanji Gupty.

```sql
DECLARE @Manager HIERARCHYID ;  
 
SELECT @Manager = ManagerHierarchyID  
FROM dbo.Employees
WHERE EmployeeID = 2 ;  
 
SELECT
      EmployeeID
    , FirstName
    , LastName
    , ManagerHierarchyID.ToString()
FROM dbo.Employees
WHERE ManagerHierarchyID.IsDescendantOf(@Manager) = 1 ;
```

W podobny sposób przypomnij sobie zapytanie rekurencyjne, które napisaliśmy, aby dowiedzieć się, kto zarządza menedżerem Emmy Robert. Można to zastąpić prostym skryptem z Listingu 3.12, który wykorzystuje metodę `GetAncestor()` do poruszania się po hierarchii.

```swift
DECLARE @EmployeeID INT ;
 
SET @EmployeeID = 22 ;
 
SELECT
    FirstName
    , LastName
    , Role
    , ManagerhierarchyID.ToString() AS ManagerHierarchyID
FROM dbo.Employees
WHERE ManagerHierarchyID = (
    SELECT ManagerHierarchyID.GetAncestor(2)
    FROM dbo.Employees
    WHERE EmployeeID = @EmployeeID
) ;
```

Być może pamiętacie, że wspomniałem również, że możemy zostać poproszeni o przemierzenie hierarchii na wiele sposobów i zasugerowałem, że możemy napotkać potrzebę znalezienia rodzeństwa w hierarchii. Aby przekonać się, jakie to proste, wyobraź sobie, że MagicChoc przeprowadza analizę porównawczą wynagrodzeń. W szczególności Amy Fry poprosiła o podwyżkę, a zespół HR chce wiedzieć, czy jej wynagrodzenie jest na tym samym poziomie, co wynagrodzenie innych osób na jej szczeblu organizacji.

W tym scenariuszu możemy użyć metody GetLevel() w celu określenia poziomu hierarchii, na którym ona się znajduje, a następnie zwrócić szczegółowe informacje o wszystkich pozostałych pracownikach na tym samym poziomie. Technikę tę pokazano na aukcji 3.13.

```sql
DECLARE @EmployeeID INT ;
 
SET @EmployeeID = 14 ;
 
SELECT 
    FirstName
  , LastName
  , Salary
FROM dbo.Employees
WHERE ManagerHierarchyID.GetLevel() = (
    SELECT 
        ManagerHierarchyID.GetLevel()
    FROM dbo.Employees
    WHERE EmployeeID = @EmployeeID
) ;
```

### 3.4 #9 Brak przechowywania danych XML w natywnym formacie XML
Kiedy dane XML są przekazywane do bazy danych, programiści często czują się zmuszeni do przechowywania tych danych w relacyjnych strukturach danych. Czasami wynika to z poczucia najlepszej praktyki – sugestia, że jest to baza danych, więc na pewno lepiej przechowywać dane w formacie relacyjnym? Innym razem wyboru dokonuje się ze strachu przed XMLem i potencjalną koniecznością napisania złożonych instrukcji XQuery, aby uzyskać dostęp do danych. Szczerze mówiąc, w wielu przypadkach przechowywanie danych w formacie relacyjnym jest dobrym pomysłem, ale w niektórych scenariuszach jest to dalekie od prawdy.

Wróćmy myślami do naszej tabeli `Employees`. Do tej pory dodaliśmy jedynie dane dotyczące pracowników kadry zarządzającej, którzy są pracownikami zatrudnionymi na stałe. Magazyn jednak często zatrudnia pracowników tymczasowych poprzez agencję pracy o nazwie Total Warehouse Jobs. MagicChoc współpracuje z tą agencją i dlatego firmy zdecydowały się na integrację swoich systemów.

Kiedy pracownik tymczasowy opuszcza MagicChoc, dane dotyczące jego pracownika są usuwane z systemu. Jednakże Total Warehouse Jobs prowadzi rejestr swoich poprzednich umów. Jeżeli dana osoba podpisze nową umowę z MagicChoc, wówczas Total Warehouse Jobs ma obowiązek dostarczyć listę swoich poprzednich umów wraz z pełnionymi rolami. Pomaga to MagicChoc przydzielać im odpowiednią pracę, w której mają doświadczenie.

Charakter XML oznacza, że jest on rozszerzalny, czytelny dla człowieka, obsługuje walidację schematu i może być uniwersalnie przetwarzany. To sprawia, że jest to popularny wybór w przypadku integracji systemów. W naszym scenariuszu ustalono, że po podpisaniu umowy Total Warehouse Jobs prześle dane dotyczące poprzednich umów pracownika w dokumencie XML. Informacje te należy przechowywać w bazie danych HumanResources.

Aplikacja magazynowa codziennie wykorzystuje te dane do obliczenia, którym osobom zostaną przydzielone jakie zadania, w oparciu o codzienne priorytety produkcyjne. Proces ETL musi wysłać dane do aplikacji hurtowni w formacie XML.

#### 3.4.1 Rozdrabnianie XML

Sposób, w jaki niektórzy programiści podchodzą do tego scenariusza, polega na podzieleniu otrzymanych danych XML na tabelę relacyjną, a następnie zrekonstruowaniu dokumentu XML w celu wysłania go do odbiorcy.

> WSKAZÓWKA
>
> Rozdrabnianie XML to proces usuwania (lub niszczenia) znaczników z danych i organizowania danych w wartości relacyjne.
>

Gdybyśmy przyjęli to podejście w naszym przykładzie MagicChoc, pierwszą rzeczą, którą będziemy musieli zrobić, to utworzyć schemat, który będzie przechowywał historyczne dane kontraktu. Charakter danych oznacza, że będziemy potrzebować trzech tabel, aby uniknąć powielania danych. Jest to kluczowa koncepcja normalizacji, którą omówimy w rozdziale 4. Przykład tego, jak może wyglądać ta tabela, można znaleźć na aukcji 3.14.

```sql
CREATE TABLE dbo.ContractHistory (
    ContractHistoryID    SMALLINT    NOT NULL    PRIMARY KEY    IDENTITY,
    EmployeeID           SMALLINT    NOT NULL    REFERENCES dbo.Employees(EmployeeID),
    ContractStartDate    DATE        NOT NULL,
    ContractEndDate      DATE        NOT NULL
) ;
 
CREATE TABLE dbo.Skills (
    SkillsID    SMALLINT    NOT NULL    PRIMARY KEY    IDENTITY,
    Skill       VARCHAR(30) NOT NULL
) ;
 
CREATE TABLE dbo.ContractSkills (
    ContractSkillsID    INT         NOT NULL    PRIMARY KEY    IDENTITY,
    ContractHistoryID   SMALLINT    NOT NULL    REFERENCES dbo.ContractHistory(ContractHistoryID),
    SkillID             SMALLINT    NOT NULL    REFERENCES dbo.Skills(SkillsID)
) ;
```

Ponieważ właśnie utworzona tabela `Skills` jest tabelą referencyjną, zanim przejdziemy dalej, dodajmy kilka przykładowych danych, korzystając ze skryptu z Listingu 3.15.

```swift
INSERT INTO dbo.Skills (Skill)
VALUES
    ('Picker'),
    ('Packer'),
    ('Stock Take'),
    ('Forklift Driver'),
    ('Machine 1 operator'),
    ('Machine 2 operator'),
    ('Machine 3 operator'),
    ('Machine 4 operator') ;
```

Historyczne dane kontraktu składają się z dokumentu XML skoncentrowanego na elementach, z elementem głównym o nazwie <EmployeeContracts>.

Mapowania skoncentrowane na elementach i atrybutach

W dokumencie XML skoncentrowanym na elementach element zawiera elementy podrzędne, które przechowują właściwości elementu. Jest to przeciwieństwo dokumentu XML skoncentrowanego na atrybutach, w którym właściwości elementu są przechowywane w atrybutach elementu. Przykładowo we fragmencie:

```xml
<Employee “ID”></Employee> 
```


`„ID”` jest atrybutem elementu <Employee>. W przypadku prostych dokumentów wybór elementów zamiast atrybutów jest w dużej mierze wyborem stylistycznym. Jeśli jednak masz złożone węzły, które będą się powtarzać lub muszą znajdować się w określonej kolejności, wówczas należy użyć elementów, ponieważ nie można tego osiągnąć za pomocą atrybutów.

Pod elementem głównym znajduje się element złożony o nazwie <Employee>, który zawiera elementy potomne, które będą przechowywane w naszej tabeli Pracownicy. Zawiera także zagnieżdżony złożony element o nazwie <Contracts>, który zawiera powtarzający się element o nazwie <Contrac>, zawierający szczegóły każdej umowy zawartej przez pracownika. Kolejny złożony element jest zagnieżdżony w <Contracts> o nazwie <Skills>, zawierający z kolei powtarzający się element o nazwie <Skill>. Przykład danych, które otrzymujemy z Total Warehouse Jobs pokazano poniżej:

```xml
<EmployeeContracts>
    <Employee>
        <EmployeeID>23</EmployeeID>
        <FirstName>Robert</FirstName>
        <LastName>Blake</LastName>
        <DateOfBirth>19781212</DateOfBirth>
        <Contracts>
            <Contract>
                <StartDate>20200101</StartDate>
                <EndDate>20203006</EndDate>
                <Skills>
                    <Skill>Forklift driver</Skill>
                    <Skill>Picker</Skill>
                    <Skill>Packer</Skill>
                </Skills>
            </Contract>
            <Contract>
                <StartDate>20210101</StartDate>
                <EndDate>20211212</EndDate>
                <Skills>
                    <Skill>Picker</Skill>
                    <Skill>Stock Take</Skill>
                </Skills>
            </Contract>
        </Contracts>
    </Employee>
</EmployeeContracts>
```

Zatem, aby zastosować podejście polegające na przechowywaniu tego zbioru danych w tradycyjnej strukturze relacyjnej, naszym pierwszym zadaniem będzie poszatkowanie otrzymanych danych. Aby to zrobić, możemy użyć funkcji `OPENXML()`, która zwróci zestaw wierszy z dokumentu XML, lub możemy użyć kombinacji metod XQuery `nodes()` i `*-/`. W tym przykładzie użyjemy OPENXML(), ponieważ widzę, że ludzie najczęściej używają tej metody.

Pierwsza złożoność funkcji OPENXML() polega na tym, że nie ma ona wbudowanego analizatora składni XML. Zamiast tego musimy przeanalizować dokument XML przy użyciu parsera MSXML przed przekazaniem go do funkcji. Przygotowanie dokumentu możemy wykonać za pomocą procedury składowanej sp_xml_preparedocument. Ta procedura przeanalizuje dokument, a obiekt wyjściowy będzie zawierał uchwyt do drzewiastej reprezentacji węzłów w dokumencie, który można następnie przekazać do funkcji

> NOTATKA
>
> W przypadku korzystania z sp_xml_preparedocument ważne jest, aby po funkcji OPENXML() zastosować procedurę sp_xml_removedocument. Spowoduje to zwolnienie zużywanej pamięci.

Musimy także przekazać funkcję OPENXML(), czyli wzorzec XPath identyfikujący wiersze do przetworzenia. Wskażemy to wyrażenie XPath na najniższy poziom naszej hierarchii, a następnie w naszych mapowaniach użyjemy operatora „../”, aby przejść na wyższe poziomy.

> WSKAZÓWKA
>
> XPath jest zdefiniowany przez W3C i jest językiem używanym do nawigacji po węzłach w dokumencie XML. Wyrażenie XPath służy do wybierania określonych węzłów w dokumencie XML.

Ostatni parametr jest opcjonalny i wskazuje, jak wypełnić kolumnę overflow/splitover ????. Możliwe wartości przedstawiono szczegółowo w tabeli 3.2.

| Wartość | Opis                                                         |
| ------- | ------------------------------------------------------------ |
| 0       | Mapowanie skoncentrowane na atrybuty                         |
| 1       | Zastosuj mapowanie skoncentrowane na atrybuty, a następnie mapowanie skoncentrowane na elementy |
| 2       | Zastosuj mapowanie skoncentrowane na elementy, a następnie mapowanie skoncentrowane na atrybuty |
| 8       | Nie kopiuj danych do właściwości przepełnienia               |

Zastosowanie klauzuli WITH w połączeniu z funkcją OPENXML() służy do określenia typu danych i mapowań węzłów wewnątrz dokumentu w środowisku SQL Server. Jeśli klauzula WITH zostanie pominięta, SQL Server zwraca tabelę krawędzi XML, która zawiera szczegółową strukturę dokumentu, w tym informacje takie jak namespace URI, namespace prefix oraz wskaźniki do następnych i poprzednich elementów potomnych.

Skrypt w listingu 3.16 przedstawia proces rozbierania danych i ich wstawiania do tabel. Pierwszym krokiem jest zadeklarowanie zmiennych do przechowywania surowego dokumentu XML i uchwytu do drzewa analizy w pamięci przygotowanego dokumentu. Następnie definiowane są dwie zmienne tabeli. Pierwsza przechowuje wyniki funkcji OPENXML(), a druga zawiera wewnętrzne dane pracownika, symulujące pobieranie ich z aplikacji HR i konieczne do zapełnienia tabeli Employees. Kolejnym krokiem jest analiza danych XML przed wywołaniem funkcji OPENXML() i wstawienie wyników do zmiennej tabeli. Następnie uruchamiane są zapytania w celu zduplikowania danych i wstawienia ich do tabel. Te instrukcje INSERT są zawinięte w transakcję (o której będzie mowa w rozdziale 10), co oznacza, że jeśli jedno z wstawień nie powiedzie się, wszystkie zostaną cofnięte. Zapobiega to uzyskaniu niejednolitych danych między tabelami, które musielibyśmy ręcznie rozplątywać w przypadku awarii. Przed rozpoczęciem transakcji włączane jest XACT_ABORT. Uniemożliwia to kontynuowanie transakcji, nawet jeśli jedno ze zdań zawiedzie z powodu drobnego błędu, takiego jak nieudane naruszenie klucza obcego.

> WSKAZÓWKA
>
> Tak! Masz absolutną rację! Powinniśmy dodać obsługę błędów do tego kodu, zwłaszcza, że dane XML pochodzą z zewnętrznego źródła. Obsługa błędów omówimy w rozdziale 7.

```sql
DECLARE @RawContractDetails XML ;
DECLARE @ParsedContractDetails INT ;
DECLARE @ShreddedData TABLE (
      EmployeeID           INT
    , FirstName            NVARCHAR(32)
    , LastName             NVARCHAR(32)
    , DateOfBirth          DATE
    , ContractStartDate    DATE
    , ContractEndDate      DATE
    , Skill                NVARCHAR(30)
) ;
DECLARE @InternalEmployeeData TABLE (
      EmployeeStartDate        DATE    
    , ManagerID                SMALLINT
    , Salary                   MONEY
    , Department               NVARCHAR(64)
    , DepartmentCode           NCHAR(2)
    , Role                     NVARCHAR(64)
    , WeeklyContractedHours    INT
    , StaffOrContract          BIT
    , ContractEndDate          DATE
    , ManagerHierarchyID       HIERARCHYID
) ;
 
INSERT INTO @InternalEmployeeData
VALUES ('20230101', 14, 39000, 'Manufacturing', 'MA', 'Warehouse Operative', 40, 0, '20231231', '/1/2/2/1/') ;
 
SET @RawContractDetails = N'<EmployeeContracts>
    <Employee>
        <EmployeeID>23</EmployeeID>
        <FirstName>Robert</FirstName>
        <LastName>Blake</LastName>
        <DateOfBirth>19781212</DateOfBirth>
        <Contracts>
            <Contract>
                <StartDate>20200101</StartDate>
                <EndDate>20200603</EndDate>
                <Skills>
                    <Skill>Forklift driver</Skill>
                    <Skill>Picker</Skill>
                    <Skill>Packer</Skill>
                </Skills>
            </Contract>
            <Contract>
                <StartDate>20210101</StartDate>
                <EndDate>20211231</EndDate>
                <Skills>
                    <Skill>Picker</Skill>
                    <Skill>Stock Take</Skill>
                </Skills>
            </Contract>
        </Contracts>
    </Employee>
</EmployeeContracts>' ;
 
EXEC sp_xml_preparedocument @ParsedContractDetails OUTPUT, @RawContractDetails ;
 
INSERT INTO @ShreddedData
SELECT *   
FROM OPENXML(@ParsedContractDetails, '/EmployeeContracts/Employee/Contracts/Contract/Skills/Skill', 2)
WITH (
    EmployeeID           SMALLINT        '../../../../EmployeeID',
    FirstName            NVARCHAR(32)    '../../../../FirstName',
    LastName             NVARCHAR(32)    '../../../../LastName',
    DateOfBirth          DATE            '../../../../DateOfBirth',
    ContractStartDate    DATE            '../../StartDate',
    ContractEndDate      DATE            '../../EndDate',
    Skill                NVARCHAR(30)    'text()'
) ;
 
SET XACT_ABORT ON ;
 
BEGIN TRANSACTION
    INSERT INTO dbo.Employees
    SELECT 
          s.EmployeeID
        , s.FirstName
        , s.LastName
        , s.DateOfBirth
        , i.EmployeeStartDate
        , i.ManagerID
        , i.Salary
        , i.Department
        , i.DepartmentCode
        , i.Role
        , i.WeeklyContractedHours
        , i.StaffOrContract
        , i.ContractEndDate
        , i.ManagerHierarchyID
    FROM @ShreddedData s
    INNER JOIN @InternalEmployeeData i
        ON 1=1
    GROUP BY 
          s.EmployeeID
        , s.FirstName
        , s.LastName
        , s.DateOfBirth
        , i.EmployeeStartDate
        , i.ManagerID
        , i.Salary
        , i.Department
        , i.DepartmentCode
        , i.Role
        , i.WeeklyContractedHours
        , i.StaffOrContract
        , i.ContractEndDate
        , i.ManagerHierarchyID ;
 
    INSERT INTO dbo.ContractHistory(EmployeeID, ContractStartDate, ContractEndDate)
    SELECT
          EmployeeID
        , ContractStartDate
        , ContractEndDate
    FROM @ShreddedData
    GROUP BY 
          EmployeeID
        , ContractStartDate
        , ContractEndDate ;
 
    INSERT INTO dbo.ContractSkills(ContractHistoryID, SkillID)
    SELECT
          ch.ContractHistoryID  
        , s.SkillsID 
    FROM @ShreddedData sd
    INNER JOIN Skills s
        ON TRIM(s.Skill) = TRIM(sd.Skill)
    INNER JOIN ContractHistory ch
        ON sd.EmployeeID = ch.EmployeeID
            AND sd.ContractStartDate = ch.ContractStartDate
            AND sd.ContractEndDate = ch.ContractEndDate ;
 
COMMIT
 
EXEC sp_xml_removedocument @ParsedContractDetails ;
```

Ok, więc to była ciężka praca, prawda? To była ciężka praca również dla SQL Server! W moim środowisku laboratoryjnym, czyli instancji t2.large EC2, w której działa tylko ten skrypt, wykonanie skryptu zajęło 31 ms. Przetworzenie pięciu wierszy danych w trzech tabelach zajmuje 31 ms. Jak dotąd jedyne, co zrobiliśmy, to wyłuszanie danych z XML. W następnej sekcji przyjrzymy się procesowi wymaganemu do zrekonstruowania dokumentu XML, aby można go było wysłać do klienta. Być może zaczynasz rozumieć, dlaczego nie polecam tego podejścia w naszym konkretnym przypadku użycia?

#### 3.4.2 Rekonstrukcja XML
Teraz, gdy podzieliliśmy dane XML na tabele, musimy napisać proces, którego ETL użyje do wysłania danych do aplikacji hurtowni. Oznacza to rekonstrukcję dokumentu XML na podstawie danych zapisanych w tabelach. Możemy to osiągnąć za pomocą instrukcji SELECT, która określa klauzulę FOR XML. FOR XML ma cztery możliwe tryby, które szczegółowo opisano w tabeli 3.3.

| Tryb     | Opis                                                         |
| -------- | ------------------------------------------------------------ |
| RAW      | Najbardziej podstawowy tryb do konstrukcji XML. Generuje płaski dokument XML z jednym elementem na wiersz. |
| AUTO     | Generuje dokumenty XML z zagnieżdżonymi elementami. Zagnieżdżanie jest kontrolowane przez warunki łączenia w zapytaniu. Automatyczne formatowanie oznacza minimalną kontrolę nad formatowaniem XML. |
| PATH     | Umożliwia zaawansowaną kontrolę nad formatem dokumentu XML, mapując kolumny w zapytaniu na węzły XML w określonym miejscu hierarchii. |
| EXPLICIT | Oferuje podobną kontrolę nad formatowaniem jak tryb PATH, ale jest nadmiernie złożony. Zazwyczaj nie ma potrzeby korzystania z tego trybu. |

Aby utworzyć poprawny kształt naszego dokumentu XML, będziemy musieli użyć FOR XML PATH z podzapytaniami, które również korzystają z klauzuli FOR XML PATH. Podzapytania są wymagane, abyśmy mogli poradzić sobie z powtarzającymi się elementami na poziomie podrzędnym dokumentu.

Zapytanie z Listingu 3.17 pokazuje, jak możemy to osiągnąć, używając dwóch warstw podzapytań. Najbardziej wewnętrzne zapytanie zwraca umiejętności powiązane z danym kontraktem. Wyniki te są konwertowane do formatu XML przy użyciu FOR XML PATH. Ta klauzula używa słowa kluczowego TYPE do zdefiniowania, że wyniki będą w poprawnym formacie XML, i używa ROOT do określenia nazwy węzła głównego w dokumencie. Zewnętrzne podzapytanie wykorzystuje ten sam proces do pobrania szczegółów z powtarzającego się elementu Contracts. Na koniec zapytanie zewnętrzne zwraca dane pracownika, które będą znajdować się na górze hierarchii.

Listing 3.17 Zrekonstruuj dokument XML

```swift
SELECT
      e.EmployeeID 'EmployeeID'
    , e.FirstName 'FirstName'
    , e.LastName 'LastName'
    , e.DateOfBirth 'DateOfBirth'
    , (
        SELECT 
             ch.ContractStartDate 'StartDate'
           , ch.ContractEndDate 'EndDate'
           , (
                SELECT
                    s.Skill 'Skill'
            FROM dbo.Skills s
            INNER JOIN ContractSkills cs
                ON s.SkillsID = cs.SkillID 
                   WHERE cs.ContractHistoryID = ch.ContractHistoryID
                   FOR XML PATH(''), TYPE, ROOT('Skills')
        )
        FROM dbo.ContractHistory ch
        WHERE EmployeeID = e.EmployeeID
        FOR XML PATH('Contract'), TYPE, ROOT('Contracts')
    )
FROM dbo.Employees e
WHERE EmployeeID = 23
FOR XML PATH('Employee'), ROOT('EmployeeContracts') ;
```

W moim środowisku laboratoryjnym wykonanie tego zapytania zajęło 52 ms. Teraz pomyśl o środowisku produkcyjnym, w którym procesy te są realizowane stale przez wiele osób. Zastanów się także, gdzie takie procesy muszą być uruchamiane w przypadku dużych, złożonych dokumentów XML. Możesz zobaczyć, dlaczego może to stać się dość kosztowne.

#### 3.4.3 Unikanie kosztów ogólnych poprzez przechowywanie danych w formacie XML
Konwersja XML na dane relacyjne i z powrotem wymagała sporo wysiłku, a łącznie zajęła 83 ms. Moglibyśmy zaoszczędzić czas programowania i kompilacji/wykonania, gdybyśmy przechowywali dane w formacie XML. Przyjrzyjmy się zatem wpływowi rozwoju i przetwarzania, gdybyśmy przechowywali dane w ich natywnym formacie.

Ponieważ wszystkie szczegóły dotyczące umiejętności i umów pracownika są przechowywane w jednym dokumencie XML, nie ma potrzeby tworzenia tabel `EmployeeContracts` i `EmployeeSkills`. Zamiast tego możemy po prostu wstawić dokument XML do tabeli Pracownicy. Zanim więc zaczniemy, zaktualizujmy tabelę Pracownicy i dodajmy kolumnę o nazwie Poprzednie umowy, w której będą przechowywane dane. Listing 3.18 zawiera polecenie, które możemy uruchomić, aby to osiągnąć.

```sql
ALTER TABLE dbo.Employees 
    ADD PreviousContracts XML NULL ;
```

W naszym konkretnym scenariuszu nie będziemy w stanie uniknąć użycia niewielkiej ilości XQuery. Dzieje się tak dlatego, że dane zostaną zapisane w identyfikatorze pracownika `EmployeeID` i jest ono dostarczane przez Total Warehouse Jobs w dokumencie XML. Dlatego będziemy musieli wyodrębnić tę wartość wraz z imieniem i datą urodzenia pracownika.

Skrypt za pomocą którego możemy dodać nowego użytkownika do tabeli `Employees` pokazano poniżej na aukcji 3.19. Podobnie jak w poprzednim przykładzie, symulujemy nasz system HR, podając niektóre wartości, które wypełnią naszą tabelę Pracownicy, za pomocą zmiennej tabeli o nazwie `@InternalEmployeeData`. Następnie używamy metody XQuery value() w celu wyodrębnienia węzłów `EmployeeID`, `FirstName`, `LastName` i `DateOfBirth` z dokumentu XML. Za pomocą metody value() do węzła, który chcemy wyodrębnić, przekazywana jest ścieżka XPath, po której następuje typ danych SQL Server, na który zostanie zamapowany węzeł. value() wymaga pojedynczej wartości, dlatego żądany indeks węzła jest obowiązkowy, nawet jeśli węzeł się nie powtarza. Należy zauważyć, że identyfikator pracownika w dokumencie XML został zwiększony, aby uniknąć niepowodzenia wstawiania z powodu naruszenia ograniczenia klucza podstawowego.

> WSKAZÓWKA
>
> Wyodrębniamy wartość elementu, ale gdybyśmy wyodrębniali atrybut, musielibyśmy poprzedzić nazwę atrybutu symbolem @.

```sql
DECLARE @RawContractDetails XML ;
DECLARE @InternalEmployeeData TABLE (
      EmployeeStartDate        DATE    
    , ManagerID                SMALLINT
    , Salary                   MONEY
    , Department               NVARCHAR(64)
    , DepartmentCode           NCHAR(2)
    , Role                     NVARCHAR(64)
    , WeeklyContractedHours    INT
    , StaffOrContract          BIT
    , ContractEndDate          DATE
    , ManagerHierarchyID       HIERARCHYID
) ;
 
INSERT INTO @InternalEmployeeData
VALUES ('20230101', 14, 39000, 'Manufacturing', 'MA', 'Warehouse Operative', 40, 0, '20231231', '/1/2/2/1/') ;
 
SET @RawContractDetails = N'<EmployeeContracts>
    <Employee>
        <EmployeeID>25</EmployeeID>
        <FirstName>Robert</FirstName>
        <LastName>Blake</LastName>
        <DateOfBirth>19781212</DateOfBirth>
        <Contracts>
            <Contract>
                <StartDate>20200101</StartDate>
                <EndDate>20200603</EndDate>
                <Skills>
                    <Skill>Forklift driver</Skill>
                    <Skill>Picker</Skill>
                    <Skill>Packer</Skill>
                </Skills>
            </Contract>
            <Contract>
                <StartDate>20210101</StartDate>
                <EndDate>20211231</EndDate>
                <Skills>
                    <Skill>Picker</Skill>
                    <Skill>Stock Take</Skill>
                </Skills>
            </Contract>
        </Contracts>
    </Employee>
</EmployeeContracts>' ;
 
INSERT INTO dbo.Employees
SELECT
      @RawContractDetails.value(‘(/EmployeeContracts/Employee/EmployeeID)[1]’, ‘SMALLINT’) AS EmployeeID
    , @RawContractDetails.value(‘(/EmployeeContracts/Employee/FirstName)[1]’, ‘NVARCHAR(32)’) AS FirstName
    , @RawContractDetails.value(‘(/EmployeeContracts/Employee/LastName)[1]’, ‘NVARCHAR(32)’) AS LastName
    , @RawContractDetails.value(‘(/EmployeeContracts/Employee/DateOfBirth)[1]’, ‘DATE’) AS DateOfBirth
    , EmployeeStartDate   
    , ManagerID   
    , Salary  
    , Department
    , DepartmentCode   
    , Role       
    , WeeklyContractedHours  
    , StaffOrContract  
    , ContractEndDate   
    , ManagerHierarchyID  
    , @RawContractDetails AS PreviousContracts
FROM @InternalEmployeeData ;
```

Wykonanie tego skryptu w moim środowisku laboratoryjnym zajęło 33 ms. Możemy teraz pobrać dokument XML dla aplikacji magazynowej za pomocą prostej instrukcji SELECT z Listingu 3.20.

 

```swift
SELECT
    PreviousContracts
FROM dbo.Employees
WHERE EmployeeID = 25 ;
```

Wykonanie tej instrukcji SELECT w moim środowisku laboratoryjnym zajęło mniej niż 1 ms. Jeśli zaokrąglimy to w górę do 1 ms, oznacza to, że wstawienie i zwrócenie danych XML zajęło łącznie 34 ms. Oznacza to, że ogólnie rzecz biorąc, kompleksowe przetwarzanie było ponad dwukrotnie szybsze, gdy uniknęliśmy niszczenia.

Czy to oznacza, że nigdy nie powinniśmy niszczyć danych XML? Nie, absolutnie nie! Istnieje wiele dobrych powodów niszczenia danych. Ogólna zasada, której używam, jest taka, że T-SQL jest znacznie wydajniejszy niż XQuery. Dlatego jeśli nasz scenariusz wymaga od nas częstego wysyłania zapytań do danych, dobrym pomysłem jest ich zniszczenie. I odwrotnie, jeśli często zapisujemy dane, ale rzadko je wysyłamy, najlepszym rozwiązaniem będzie przechowywanie danych w natywnym formacie XML. Błędem, którego powinniśmy unikać, jest zawsze niszczenie danych (lub, w równym stopniu, zawsze przechowywanie danych w formacie XML). Zamiast tego podejmuj decyzję projektową w oparciu o wymagania naszej aplikacji.

### 3.5 #10 Ignorowanie JSON
Tak jak programiści SQL mają tendencję do unikania danych XML, tak też mają tendencję do unikania danych JSON. Jak widzieliśmy, mówiąc o języku XML, może to prowadzić do nieoptymalnego projektu tabeli i nieoptymalnej wydajności. Wyobraźmy sobie, że zostaliśmy poproszeni o rozszerzenie naszego schematu danych dla jednostki pracowników, tak aby pozwolił nam przechowywać ich adresy domowe.

Typowym modelem modelowania adresów w SQL Server byłaby tabela Adresy z identyfikatorem adresu jako kluczem podstawowym, która następnie zawierałaby kolumnę o tej samej nazwie w tabeli Pracownicy, a także potencjalnie inne tabele, w których adres jest istotny, np. Biura lub placówki w przypadku bazy danych zasobów ludzkich. Kolumna w tabeli Pracownicy będzie miała ograniczenie klucza obcego, dzięki czemu można będzie przechowywać tylko odpowiednie wartości z tabeli Adresy.

Ten model jest w porządku, ale stanie się szeroką, rzadką tabelą, ponieważ nie wszystkie adresy mają tę samą liczbę wierszy i musimy być w stanie uwzględnić każdą ewentualność. Sytuacja jest jeszcze gorsza w przypadku MagicChoc, która jest firmą międzynarodową. Pomyśl o kodach pocztowych. Są one znane jako kody pocztowe w Wielkiej Brytanii, yóu dì qū h→o w Tajlandii i numery blokowe w Bahrajnie, żeby wymienić tylko kilka. Ponieważ mają one różne formaty w różnych krajach, albo przechowujemy je w bardzo liberalnym typie danych, albo moglibyśmy użyć oddzielnych, słabo zapełnionych kolumn dla każdej z nich. Warto również zauważyć, że wiele krajów, takich jak Bahamy, Dominika, Fidżi i wiele krajów afrykańskich, nie ma żadnego odpowiednika kodu pocztowego, co jeszcze bardziej zwiększa skąpy charakter tabeli.

W zależności od wymagań naszej aplikacji lepszym rozwiązaniem może być przechowywanie danych w formacie JSON. Podobnie jak XML, JSON ma format częściowo ustrukturyzowany, który pozwala na pominięcie nieistotnych danych. Staje się coraz bardziej popularny, ponieważ jest to bardzo lekki format ze znacznie mniejszą liczbą znaczników niż XML.

Ze względu na wydajność języka T-SQL w zakresie wykonywania zapytań o dane, modelując dane, pracuję według tej samej reguły w przypadku JSON, co w przypadku XML. Mianowicie, jeśli dane będą często zapisywane i rzadko odpytywane, wówczas dobrym wyborem będzie JSON. Jeśli jednak dane są odpytywane znacznie częściej, niż są zapisywane, wolę przechowywać dane relacyjne.

Inne przypadki użycia JSON obejmują modelowanie domen danych, które w przeciwnym razie musiałyby zostać podzielone na dane relacyjne i NoSQL. Dzieje się tak dlatego, że w SQL Server oba typy danych mogą być przechowywane w tym samym schemacie. Wsparcie komunikacji z API REST lub innymi integracjami systemowymi wysyłającymi i odbierającymi dane w formacie JSON. Obecnie możliwe jest również przechowywanie i analizowanie danych dzienników w SQL Server, gdzie dzienniki są oparte na formacie JSON.

Zobaczmy więc, jak możemy uniknąć błędu polegającego na ignorowaniu formatu JSON, modelując dane adresowe naszych pracowników w formacie JSON. Jeśli nasze adresy mogą odnosić się do wielu podmiotów, takich jak biura lub witryny, nadal możemy chcieć zbudować tabelę Adresy, aby adresy w formacie JSON mogły być używane przez wiele podmiotów. Jesteśmy jednak przekonani, że Pracownicy to jedyny podmiot, który będzie musiał wchodzić w interakcję z danymi adresowymi, dlatego do naszej tabeli Pracownicy dodamy dodatkową kolumnę do przechowywania danych adresowych. Listing 3.21 przedstawia polecenie, które utworzy tę kolumnę.

```sql
ALTER TABLE dbo.Employees
    ADD Address NVARCHAR(MAX) ;
```

> NOTATKA
>
> Ale poczekaj chwilę! Będziemy modelować dane w JSON, więc po co właśnie stworzyliśmy kolumnę typu NVARCHAR(MAX)? Odpowiedź na to pytanie jest taka, że JSON nie ma własnego typu danych w SQL Server. Jest przechowywany jako VARCHAR lub NVARCHAR, a następnie jest indeksowany jako tekst. Dzięki temu JSON jest w pełni kompatybilny z dowolną funkcjonalnością SQL Server obsługującą tekst.

Na początek przyjrzyjmy się formatowi dokumentu JSON, którego będziemy używać do przechowywania adresów. Format JSON wykorzystuje proste pary nazwa:wartość, zawarte w „”. Zagnieżdżanie jest oznaczone za pomocą {}. Użycie [] oznacza tablicę. Nasz dokument adresowy ma korzeń EmployeeAddress i zawiera węzły dla EmployeeID i Address. Węzeł Adres zawiera dalsze węzły zagnieżdżone pod spodem, w których przechowywane są poszczególne wiersze adresu. Można dodać dodatkowe linie lub zastąpić istniejące węzły, w zależności od wymagań każdego adresu, co oznacza, że eliminujemy problem rzadkich danych. Przykładowy dokument można zobaczyć poniżej:

```json
{
   "EmployeeAddress":[
      {
         "EmployeeID":1,
         "Address":{
            "Line1":"5331 Rexford Court",
            "City":"Montgomery",
            "State":"AL",
            "ZipCode":"36116"
         }
      }
   ]
}
```

Dokument ten możemy wygenerować za pomocą instrukcji SELECT z klauzulą FOR JSON. Istnieją dwie opcje przetwarzania FOR JSON: AUTO i PATH. AUTO może służyć do automatycznego formatowania dokumentu na podstawie kolejności kolumn i kolejności sekwencji JOIN w tabeli. Tryb PATH zapewnia szczegółową kontrolę nad formatowaniem.

> WSKAZÓWKA
>
> FOR XML AUTO można używać tylko w odniesieniu do tabeli, więc teoretycznie nie można go używać zgodnie z planem, ale łatwo jest obejść to ograniczenie, dodając klauzulę FROM do dowolnej tabeli. Zwykle używam do tego celu sys.tables, ale wystarczy jakakolwiek tabela. Jeśli zastosujemy tę technikę, będziemy musieli dodać TOP 1 do naszej klauzuli SELECT, aby uniknąć zwracania węzła dla każdego rekordu w tabeli

Zapytanie z aukcji 3.22 wygeneruje nasz przykładowy dokument JSON. Pamiętaj, że używamy trybu PATH, więc każda kolumna ma alias określający nazwę węzła i jego pozycję w hierarchii JSON.

```sql
SELECT
      1 AS 'EmployeeID'
    , '5331 Rexford Court' AS 'Address.Line1'
    , 'Montgomery'         AS 'Address.City'
    , 'AL'                 AS 'Address.State'
    , '36116'              AS 'Address.ZipCode'
FOR JSON PATH, ROOT ('EmployeeAddress') ;
```

Skrypt na aukcji 3.23 pokazuje, jak moglibyśmy zaktualizować tabelę Pracownicy, aby dodać adres pracownika, jeśli otrzymalibyśmy dane adresowe w formacie JSON. Klauzula WHERE instrukcji UPDATE wykorzystuje funkcję JSON_VALUE() do pobrania identyfikatora pracownika z dokumentu. Pierwszym parametrem akceptowanym przez tę funkcję jest dokument JSON, w którym należy wyszukiwać. Drugi parametr określa ścieżkę do węzła. Ścieżka JSON zawsze zaczyna się od $. który reprezentuje cały dokument. Węzeł główny - EmployeeAddress jest tablicą (oznaczoną nawiasami kwadratowymi), co oznacza, że musimy przekazać indeks tablicy, aby określić, dla którego rekordu chcemy zwrócić wartość. Jeśli nie przekażemy tego indeksu, zwrócimy wynik NULL, nawet jeśli istnieje tylko jeden rekord, jak ma to miejsce w naszym scenariuszu.

```sql
 
DECLARE @EmployeeAddress NVARCHAR(MAX) ;
 
SET @EmployeeAddress = (
    SELECT
          1 AS 'EmployeeID'
        , '5331 Rexford Court' AS 'Address.Line1'
        , 'Montgomery'         AS 'Address.City'
        , 'AL'                 AS 'Address.State'
        , '36116'              AS 'Address.ZipCode'
    FOR JSON PATH, ROOT ('EmployeeAddress')
) ;
 
UPDATE dbo.Employees
SET Address = @EmployeeAddress
WHERE EmployeeID = 
    JSON_VALUE(@EmployeeAddress, '$.EmployeeAddress[0].EmployeeID') ;
```

Jeśli kiedykolwiek będziemy musieli zniszczyć ten dokument JSON do formatu relacyjnego, możemy użyć funkcji `OPENJSON()`. Format i funkcjonalność `OPENJSON()` będą wyglądać znajomo tym z Was, którzy czytali o `OPENXML()` wcześniej w tym rozdziale. Zaletą funkcji `OPENJSON()` jest jednak to, że nie ma konieczności przygotowywania dokumentu przed użyciem. Oszczędza to wykorzystanie zasobów.

Na aukcji 3.24 używamy funkcji `OPENJSON()` do niszczenia dokumentu adresowego naszego pracownika. Pierwszym parametrem przekazywanym do funkcji jest sam dokument JSON. Drugi parametr określa ścieżkę do najwyższego poziomu hierarchii, który chcemy przepytać. Pamiętaj, że $ reprezentuje cały dokument i podobnie jak w przypadku `JSON_VALUE()`, przekazywana przez nas ścieżka musi uwzględniać tablicę poprzez określenie indeksu tablicy. Klauzula WITH określa kolumny, które chcemy zwrócić z dokumentu, wraz z typami danych SQL, na które będą one mapowane, oraz ścieżką do odpowiedniego węzła, jeśli węzeł nie znajduje się na tym samym poziomie co wyrażenie ścieżki.

```sql
DECLARE @EmployeeAddress NVARCHAR(MAX) ;
 
SET @EmployeeAddress = (
    SELECT
          1 AS 'EmployeeID'
        , '5331 Rexford Court' AS 'Address.Line1'
        , 'Montgomery'         AS 'Address.City'
        , 'AL'                 AS 'Address.State'
        , '36116'              AS 'Address.ZipCode'
    FOR JSON PATH, ROOT ('EmployeeAddress')
) ;
 
SELECT *
FROM OPENJSON(@EmployeeAddress, '$.EmployeeAddress[0]')
WITH (
    EmployeeID SMALLINT,
    Line1 NVARCHAR(64) '$.Address.Line1',
    City NVARCHAR(64) '$.Address.City',
    [State] NCHAR(2) '$.Address.State',
    ZipCode NVARCHAR(10) '$.Address.ZipCode'
) ;
```

Widzimy, że JSON może odgrywać ważną rolę w SQL Server, jeśli jest odpowiednio używany. Możemy wykorzystać obsługę JSON SQL Server do denormalizacji danych i uproszczenia złożonych modeli danych, integracji z NoSQL lub uniknięcia przetwarzania danych wysyłanych do i z interfejsów API.

### 3.6 Podsumowanie
• Zawsze używaj najbardziej restrykcyjnego typu danych, który pozwoli nam przechowywać wszystkie potencjalnie wymagane wartości. Jest to szczególnie prawdziwe w przypadku wartości całkowitych używanych w ograniczeniach kluczowych lub wartości indeksowanych.
• Zawsze używaj ciągów o stałej długości, aby zmniejszyć obciążenie pamięci, jeśli długość ciągów w kolumnie jest stała. Zawsze używaj ciągów o zmiennej długości, gdy długość nie będzie stała.
• Unikaj pisania własnego kodu i tworzenia tabel z odniesieniami do siebie podczas modelowania i tworzenia hierarchii. Zamiast tego użyj typu danych HIERARCHYID, ponieważ typ ten obejmuje wiele metod, które eliminują ciężkie prace rozwojowe.
• Kiedy mamy do czynienia z danymi XML, nie powinniśmy mieć poczucia, że zawsze musimy dzielić te dane na model relacyjny. Zawsze należy wziąć pod uwagę wymagania aplikacji i odpowiednio modelować schemat danych. Może to obejmować przechowywanie danych w natywnym formacie XML, ale nie musi.
• Nie bój się danych JSON. Istnieją przypadki użycia, w których użycie danych JSON jest dobrym wyborem w celu uproszczenia naszych modeli danych.

## 4 Projekt bazy danych

Ten rozdział obejmuje
• Wprowadzenie do błędów projektowych w SQL Server i dlaczego ważne jest, aby ich unikać
• Błąd polegający na braku normalizacji bazy danych
• Błędy popełniane przy projektowaniu i tworzeniu kluczy
W tym rozdziale omówimy typowe błędy popełniane przy projektowaniu baz danych. Błędy te mogą powodować wiele wyzwań, prowadzących do problemów, takich jak słabo działający kod.

Błędy projektowe wprowadzane są na najwcześniejszym etapie cyklu rozwojowego, zanim zostanie napisany jakikolwiek kod. Aby to zilustrować, skupmy się na MagicChoc. Firma uważa, że jej procesy są zbyt fragmentaryczne, dlatego uruchomiła nową aplikację, która połączy funkcje sprzedaży i zakupów poza Internetem w jeden interfejs z jednym backendem. W tym celu kierownictwo MagicChoc oświadczyło, że chce przechowywać następujące elementy danych:

| Angielska nazwa pola              | Polska nazwa pola                         |
| --------------------------------- | ----------------------------------------- |
| Sales Order Date                  | Data zamówienia sprzedaży                 |
| Sales Order Number                | Numer zamówienia sprzedaży                |
| Sales Person Name                 | Imię i nazwisko przedstawiciela sprzedaży |
| Sales Person Email                | Adres e-mail przedstawiciela sprzedaży    |
| Sales Area Name                   | Nazwa obszaru sprzedaży                   |
| Sales Area Manager                | Kierownik obszaru sprzedaży               |
| Customer Company Name             | Nazwa firmy klienta                       |
| Customer Contact Name             | Imię i nazwisko kontaktu klienta          |
| Customer Contact Email            | Adres e-mail kontaktu klienta             |
| Customer Invoice Address          | Adres faktury klienta                     |
| Customer Delivery Addresses       | Adres dostawy klienta                     |
| Sales Order Delivery Due Date     | Przewidywana data dostawy zamówienia      |
| Sales Order Delivery Actual Date  | Rzeczywista data dostawy zamówienia       |
| Sales Order Item                  | Pozycja zamówienia sprzedaży              |
| Sales Order Quantities            | Ilość w zamówieniu sprzedaży              |
| Currier Used for Delivery         | Firma kurierska użyta do dostawy          |
| Product Name                      | Nazwa produktu                            |
| Product Stock Level               | Poziom zapasów produktu                   |
| Product Next Manufacture Date     | Planowana data następnej produkcji        |
| Product Next Manufacture Quantity | Planowana ilość następnej produkcji       |
| Back Order Manufacturing ID       | ID produkcji oczekującej                  |
| Product Type Name                 | Nazwa typu produktu                       |
| Product Type Description          | Opis typu produktu                        |
| Product Category Name             | Nazwa kategorii produktu                  |
| Product Category Description      | Opis kategorii produktu                   |
| Product Subcategory Name          | Nazwa podkategorii produktu               |
| Product Subcategory Description   | Opis podkategorii produktu                |
| Supplier Name                     | Nazwa dostawcy                            |
| Supplier Contact Name             | Imię i nazwisko kontaktu dostawcy         |
| Supplier Contact Email            | Adres e-mail kontaktu dostawcy            |
| Supplier Address                  | Adres dostawcy                            |
| Purchase Order Date               | Data zamówienia zakupu                    |
| Purchase                          |                                           |

W tym rozdziale zaprojektujemy strukturę tabeli potrzebną do przechowywania tych danych. Po drodze zastanowimy się, jak uniknąć typowych błędów projektowych. W szczególności przyjrzymy się problemom, które mogą wystąpić, jeśli nie uda nam się znormalizować naszej bazy danych. Następnie przyjrzymy się problemom z wydajnością, które mogą być związane ze złym wyborem klucza podstawowego. Na koniec zbadamy konsekwencje nietworzenia ograniczeń klucza obcego.

### 4.1 #11 Niepowodzenie normalizacji
Wiele razy pytałem programistów: „Czy baza danych jest znormalizowana?” a odpowiedź niezmiennie brzmi: „Tak!” Niestety, w wielu przypadkach programiści byli stosunkowo niedoświadczeni w projektowaniu baz danych i błędnie zrozumieli moje pytanie jako „Czy zaprojektowałeś bazę danych do przetwarzania transakcji online (OLTP) zamiast hurtowni danych?” W rzeczywistości zamiast normalizować swoją bazę danych, po prostu na podstawie własnego osądu podjęli decyzję o schemacie bazy danych.

Aby zaprojektować nasz schemat bazy danych, utworzymy diagram relacji encji (ERD), który jest diagramem opisującym encje będące obiektami w modelu danych oraz atrybuty używane do opisu obiektu danych. ERD opisuje również, w jaki sposób obiekty odnoszą się do siebie w modelu.

> WSKAZÓWKA
>
> ERD jest często diagramem poziomu kodu w modelu C4, gdy C4 jest używany do dokumentowania projektu bazy danych. Dlatego w tej sekcji domyślnie będziemy budować diagram kodu C4.

W kolejnych sekcjach najpierw przyjrzymy się niewłaściwemu sposobowi projektowania schematu bazy danych, czyli zastosowaniu podejścia opartego na ocenie. W tym podejściu do uporządkowania danych będziemy wykorzystywać wyłącznie nasze doświadczenie. Ponieważ doświadczenia każdego z nas są nie tylko na różnych poziomach, ale także po prostu inne, istnieje wiele problemów, które mogą pojawić się, gdy zastosujemy to nieustrukturyzowane podejście. Przeanalizujemy niektóre z bardziej typowych problemów, które mogą wystąpić, takie jak wartości nieatomowe i duplikacja danych.

Na koniec zastanowimy się, jak uniknąć błędów, używając normalizacji do zaprojektowania naszego schematu. Przyjrzymy się temu ustrukturyzowanemu podejściu do modelowania baz danych, które zostało po raz pierwszy opracowane przez Edgara Codda w latach 70. XX wieku i naprawdę przetrwało próbę czasu. Jest to zestaw metodycznych kroków, które służą do usunięcia nadmiarowości z naszych danych. Stosując to podejście będziemy kierować się ścisłym zestawem zasad, które mają na celu uniknięcie duplikacji danych i uczynienie schematu możliwie najbardziej efektywnym. Dotkniemy również tego, jak możemy przetestować naszą normalizację za pomocą diagramu relacji encji, a także przyjrzymy się koncepcji generalizacji danych.

#### 4.1.1 Projektowanie schematu na podstawie osądu
Kierujmy się więc naszą oceną, aby zaprojektować schemat nowej bazy danych sprzedaży i zakupów, o którą prosił MagicChoc. Chcą modelować 35 elementów danych, więc uprośćmy sprawę i podzielmy je na sekcje. Zacznijmy od danych Zamówienia Sprzedaży. W szczególności następujące elementy danych:

- Sales Order Date
- Sales Order Number
- Sales Person Name
- Sales Person Email
- Sales Area Name
- Sales Area Manager
- Sales Order Delivery Due Date
- Sales Order Delivery Actual Date
- Sales Order Item
- Sales Order Quantities
- Currier Used for Delivery

Atrybut Numer zamówienia sprzedaży będzie unikalny, więc wygląda na to, że jest dobrym kandydatem na klucz dla naszej jednostki. Kłopotliwym aspektem jest pozycja zamówienia sprzedaży i ilości zamówienia sprzedaży. Zauważamy, że istnieje oczywista relacja jeden do wielu. Jest to relacja między encjami, w której pojedynczy rekord w jednej encji może być powiązany z wieloma encjami w innej. W tym przypadku w ramach jednego zamówienia może być zamówionych więcej niż jeden produkt, dlatego decydujemy się na podzielenie zamówionych pozycji na inną jednostkę. Uważamy również, że skoro mamy obowiązek przechowywania ogólnych szczegółów produktu, sensowne byłoby pominięcie szczegółów produktu i pozostawienie jedynie klucza do jednostki produktów, którą wkrótce zaprojektujemy.

> KLUCZE
>
> Relacyjne bazy danych łączą tabele za pomocą kluczy. Istnieją dwa podstawowe typy kluczy – klucz podstawowy i klucz obcy. Klucz podstawowy służy do jednoznacznej identyfikacji wiersza w tabeli. Klucz obcy służy do wskazania wartości klucza podstawowego lub unikalnego ograniczenia z tabeli dodatkowej.
>
> Wyobraźmy sobie na przykład stół z pracownikami. Musisz zarejestrować biuro, w którym pracuje każdy pracownik. Możesz dodać kolumnę Biuro do tabeli pracowników, ale oznaczałoby to wielokrotne powielanie tych samych kilku nazw biur (i ewentualnie adresów, numerów telefonów itp.). Jeśli więc przeniesiesz szczegóły biura do tabeli biur, możesz użyć klucza podstawowego, powiedzmy OfficeID, aby jednoznacznie zidentyfikować każde biuro. Następnie możesz użyć klucza obcego w tabeli pracowników (często, ale nie zawsze, o tej samej nazwie co klucz podstawowy). Ten klucz obcy będzie odnosił się do wartości w kolumnie klucza podstawowego tabeli biur, aby mieć pewność, że do klucza obcego zostaną wprowadzone tylko wartości, które istnieją w kluczu podstawowym. Wymusza to tak zwaną integralność referencyjną.
>
> Jeśli klucz ma znaczenie biznesowe, np. SocialSecurityNumber, wówczas nazywany jest kluczem naturalnym. Jeśli klucz nie ma znaczenia biznesowego, np. jest to dowolna, rosnąca liczba, np. EmployeeID, wówczas nazywa się go kluczem sztucznym.
>
> Klucz naturalny może składać się z wielu kolumn. Jeśli klucz składa się z wielu kolumn, nazywa się go kluczem złożonym. Jeśli to możliwe, najlepiej zminimalizować użycie kluczy złożonych.

Podobnie, ponieważ zakładamy, że klienci mogą składać wiele zamówień i wiemy, że musimy przechowywać dane klienta w ramach wymagań, decydujemy, że powinniśmy utworzyć klucz w tabeli zamówień, który połączy się z tabelą klientów, gdy ją utworzymy. Na koniec zauważamy, że nie ma dobrego kandydata na klucz podstawowy, więc utworzymy klucz sztuczny. Pozostaje nam początek ERD, co widać na rysunku 4.1.

##### Figure 4.1 ERD for Sales Orders

![img](https://drek4537l1klr.cloudfront.net/carter/v-1/Figures/04__image001.png)

Następnie przyjrzymy się wymaganiom dotyczącym danych produktów, w których musimy przechowywać następujące dane:

- Product Name
- Product Stock Level
- Product Next Manufacture Date
- Product Next Manufacture Quantity
- Back Order Manufacturing ID
- Product Type Name
- Product Type Description
- Product Category Name
- Product Category Description
- Product SubCategory Name
- Product SubCategory Description

Decydujemy się na tabelę dla Produktów, a ponieważ nie ma oczywistego klucza naturalnego, utworzymy klucz sztuczny, który będzie powiązany z atrybutem ProductID w encji Szczegóły zamówienia sprzedaży. Zauważamy również, że pomiędzy produktami i kategoriami produktów może istnieć relacja jeden do wielu, dlatego decydujemy się na utworzenie osobnej encji Kategorie produktów, którą połączymy z encją Produkty za pomocą innego sztucznego klucza. Jeśli dodamy te podmioty do zleceń sprzedaży, otrzymamy ERD pokazany na rysunku 4.2.

##### Figure 4.2 ERD with Products added

![img](https://drek4537l1klr.cloudfront.net/carter/v-1/Figures/04__image002.png)

Następnie zbudujemy encję Klienci, o której wiemy, że będzie miała następujące atrybuty:

- Customer Company Name
- Customer Contact Name
- Customer Contact Email
- Customer Invoice Address
- Customer Delivery Addresses

Aby uniknąć szerokiej i rzadkiej tabeli, decydujemy się przenieść adresy do innej jednostki. Zdajemy sobie również sprawę, że możemy mieć jedną jednostkę Adresy, która będzie kluczem zarówno do adresu na fakturze klienta, jak i adresu dostawy klienta. Pozwala to uniknąć powielania danych w przypadku, gdy adres do faktury jest taki sam jak adres dostawy. Osiągamy to poprzez dodanie flag dla każdego typu adresu. Stworzymy sztuczne klucze dla obu podmiotów.

Dołączenie tych elementów do naszego istniejącego projektu daje nam ERD pokazany na rysunku 4.3.

##### Figure 4.3 ERD with Customers added

![img](https://drek4537l1klr.cloudfront.net/carter/v-1/Figures/04__image003.png)

Na koniec dodajmy encje dla Dostawców i Zamówień zakupu. Atrybuty danych, które musimy modelować dla tych jednostek, to:

- Supplier Name

- Supplier Contact Name

- Supplier Contact Email

- Supplier Address

- Purchase Order Date

- Purchase Order Number

- Purchase Order Items

- Purchase Order Quantities

  

Decydujemy, że będziemy modelować naszych Dostawców i Zamówienia Zakupu w podobny sposób, w jaki modelowaliśmy naszych Klientów i Zamówienia Sprzedaży. Będziemy mieć jednostkę Dostawcy, która będzie połączona z naszą istniejącą jednostką Adresy w celu przechowywania adresów dostawców. Podzielimy nasze zamówienia zakupu na dwie jednostki, jedną dla nagłówków zamówień zakupu, a drugą dla szczegółów zamówienia zakupu. Ponownie ma to umożliwić nam zamówienie wielu artykułów w tym samym zamówieniu, bez konieczności powielania danych zamówienia dla każdego artykułu. Decydujemy się na utworzenie sztucznego klucza dla Dostawców i Szczegóły zamówienia zakupu, ale numer zamówienia zakupu wykorzystujemy jako klucz naturalny w encji Nagłówki zamówień zakupu.

Jeśli dodamy te pozostałe elementy do naszego istniejącego projektu, otrzymamy ERD pokazany na rysunku 4.4.

##### Figure 4.4 ERD with Suppliers and Purchase Orders

![img](https://drek4537l1klr.cloudfront.net/carter/v-1/Figures/04__image004.png)

Mamy więc gotowy projekt schematu bazy danych. To nie było takie trudne, prawda? Jaki jest zatem problem w stosowaniu tego podejścia? Problem polega na tym, że w projekcie schematu znajduje się wiele błędów, które utrudniają pracę z naszą bazą danych i potencjalnie powodują problemy z wydajnością. W następnej części omówimy niektóre błędy i ich konsekwencje.

#### 4.1.2 Problemy ze schematem naszej bazy danych
Na pierwszy rzut oka schemat naszej bazy danych może wyglądać całkiem rozsądnie, ale jeśli przyjrzymy się bliżej niektórym z naszych wyborów projektowych, zobaczymy, gdzie wystąpią problemy. Zacznijmy od przyjrzenia się nazwom klientów i dostawców, co jest jednym z najłatwiejszych do wykrycia problemów.

Zarówno atrybut ProviderName encji Dostawcy, jak i atrybut CustomerName encji Customers będą przechowywać imiona i nazwiska. Może to powodować różne problemy. Wyobraźmy sobie, że nasza aplikacja sprzedażowa posiada funkcję korespondencji seryjnej i chcemy rozpoczynać listy do klientów od słów „Szanowny Robercie”, gdzie Robert jest imieniem klienta. Aby odzyskać to imię, musielibyśmy uruchomić zapytanie takie jak:

```sql
SELECT SUBSTRING(CustomerName, 0, CHARINDEX(' ', CustomerName, 1))
FROM Customers ;
```

To nie tylko sprawia, że nasz kod jest bardziej skomplikowany do odczytu i zapisu, ale jest także mniej wydajny niż zwykłe wybranie kolumny FirstName z tabeli. Dlatego powinniśmy zawsze upewnić się, że nasze wartości są niepodzielne. Wartość atomowa to taka, której nie można dalej rozłożyć i która nadal zachowuje swoje znaczenie. Na przykład imię Peter A Carter można podzielić na trzy wartości atomowe – imię, inicjał drugiego imienia i nazwisko. Argument, że można go podzielić na 12 znaków, nie jest słuszny, ponieważ wartości te nie miałyby znaczenia kontekstowego.

> WSKAZÓWKA
>
> Wyjątkiem od tej reguły jest sytuacja, gdy po normalizacji schematu celowo podejmujesz decyzję o ponownej denormalizacji danych. Na przykład możesz zdecydować się na przechowywanie danych adresowych w formacie JSON, jak omówiono w rozdziale 3.

Kolejny problem leży w naszej encji ProductCategories. Podmiot ten składa się zarówno z kategorii produktów, jak i podkategorii. Jednak w ramach jednej kategorii może znajdować się wiele podkategorii. Oznacza to, że będziemy musieli zduplikować ProductCategoryName i ProductCategoryDescription dla każdej podkategorii produktów. Powielanie danych powoduje marnowanie miejsca, co powoduje, że tabela jest większa, a przez to wolniejsza w czytaniu, a także zajmuje więcej pamięci i zmniejsza wydajność operacji indeksowania. Większy problem dotyczy jednak wkładek i aktualizacji. Staną się one znacznie mniej wydajne, ponieważ będziemy musieli zaktualizować szczegóły kategorii produktów w wielu wierszach. Moglibyśmy tego uniknąć, dzieląc dane na więcej jednostek i łącząc je relacją klucza podstawowego/obcego.

Zdecydowaliśmy się przechowywać SalesAreaName w encji SalesOrderHeader. Oznacza to, że nazwa zostanie powielona wiele razy, ponieważ może dotyczyć wielu wierszy. Oznacza to również, że przy każdym rekordzie trzeba powtórzyć pracę naszego Menedżera Obszaru Sprzedaży. Ponownie, gdyby szczegóły obszaru sprzedaży zostały podzielone na inną jednostkę i połączone z powrotem relacją klucza podstawowego/obcego, można byłoby tego uniknąć.

Mamy podobny problem z nazwą kontaktu dostawcy. Nie tylko wartość nie jest atomowa, ale w zależności od reguł biznesowych możemy w efekcie zduplikować dane. Co by było, gdyby dwóch naszych dostawców było własnością tej samej osoby? A co, jeśli w przyszłości będziemy mieć wymagania dotyczące przechowywania dodatkowych informacji kontaktowych, takich jak numer telefonu konkretnego kontaktu? To tylko pogorszy sprawę i sprawi, że nasi dostawcy będą bardzo poszerzeni.

To stawia nas z wieloma pytaniami. Jak moglibyśmy uniknąć tych problemów projektowych? Czy są jeszcze jakieś problemy z naszym projektem? Jak możemy to w ogóle przetestować, aby się tego dowiedzieć, bez budowania całego schematu i wypełniania go danymi?

Odpowiedzią na wszystkie te pytania jest wykorzystanie procesu normalizacji do zaprojektowania naszego schematu danych. O tym, jak korzystać z tego procesu, porozmawiamy w następnej sekcji.

> NOTATKA
>
> Dodatkowy problem, którego nie bylibyśmy w stanie wykryć bez zrozumienia danych biznesowych i reguł biznesowych. Numery zamówień sprzedaży będą miały format ABC1234D-E12. To da nam bardzo szeroki klucz podstawowy. Zagadnienie to zostanie omówione w dalszej części tego rozdziału.

#### 4.1.3 Projektowanie schematu bazy danych z wykorzystaniem normalizacji
Jak wspomniano wcześniej w tym rozdziale, normalizacja to formalny proces modelowania danych, który został po raz pierwszy stworzony przez Edgara Codda w latach 70. XX wieku i naprawdę przetrwał próbę czasu. Istnieje 10 normalnych form lub etapów procesu, które z czasem zostały dodane do metodologii. Najnowszy dodatek Essential Tuple Normal Form został dodany dopiero w 2012 roku. W szczególności normalne formy to:

• Pierwsza postać normalna (1NF)
• Druga postać normalna (2NF)
• Trzecia postać normalna (3NF)
• Elementarna postać normalna klucza (EKNF)
• Postać normalna Boyce'a-Codda (BCNF lub 3.5NF)
• Czwarta postać normalna (4NF)
• Niezbędna postać normalna krotki (ETNF)
• Piąta postać normalna (5NF)
• Postać normalna klucza domeny (DKNF)
• Szósta postać normalna (6NF)

W zdecydowanej większości przypadków nie ma potrzeby przekraczania 3NF. Należy również pamiętać, że w przypadku nadmiernej normalizacji schematu mogą pojawić się inne problemy. Dlatego ten rozdział skupi się na modelowaniu naszego schematu danych w 3NF.

Aby modelować dane w 1NF, upewnimy się, że wszystkie atrybuty są niepodzielne, mają unikalną nazwę i że ich kolejność jest nieistotna. Zadbamy również o to, aby powtarzające się grupy atrybutów zostały przeniesione do innej jednostki i aby kolejność wierszy nie była istotna.

Modelując dane w 2NF, upewnimy się, że wszystkie atrybuty są zależne od wszystkich atrybutów tworzących złożony klucz podstawowy. Jeśli tego nie zrobią, przeniesiemy ich do nowego podmiotu. Wreszcie, kiedy będziemy modelować dane w 3NF, upewnimy się, że atrybuty nie są zależne od atrybutów, które nie są obecnie częścią klucza podstawowego. Jeśli tak, to znowu znak, że należy je przenieść do nowego podmiotu.

WSKAZÓWKI NORMALIZACYJNE

Chociaż dostępne są pewne narzędzia do normalizacji danych, szczerze uważam, że najlepszym narzędziem do tego procesu jest prosty arkusz kalkulacyjny. Osobiście w przypadku mniejszych zbiorów danych, takich jak ten, o którym tu mówimy, używam kolumny w postaci normalnej, z odstępami między wierszami między jednostkami. Dzięki temu mogę porównać moje modele obok siebie i przemyśleć logikę, której używam.

Kolejną ważną wskazówką dotyczącą normalizacji jest zrozumienie danych. Oznacza to współpracę z analitykiem biznesowym lub właścicielem aplikacji w celu zrozumienia znaczenia biznesowego i reguł biznesowych stojących za każdym atrybutem. Niezastosowanie się do tego nieuchronnie doprowadzi do fałszywych założeń, które będą miały wpływ na Twój model.

Zapewnienie tego zrozumienia pomoże Tobie lub Twojemu analitykowi biznesowemu stworzyć kolejny ważny dokument zwany słownikiem danych. Ponownie, często jest to tworzone w arkuszu kalkulacyjnym i zawiera szczegółowe informacje, takie jak:

• Nazwa atrybutu
• Opis/znaczenie atrybutu
• Reguły biznesowe, takie jak wartości oczekiwane, niepowtarzalność itp.
• Przykładowe dane
• Typ danych
• Zależności nadrzędne
• Pochodzenie danych
Pochodzenie danych jest ważne w złożonych aplikacjach warstwy danych. Śledzi ścieżkę, którą element danych przeszedł, aby dotrzeć do tabeli, w której aktualnie się znajduje. Na przykład, jeśli mamy bazę danych używaną do marketingu internetowego, możesz przechowywać identyfikator pliku cookie. Pochodzenie może wyglądać mniej więcej tak:

1. Identyfikator śledzenia (w pliku CSV wyświetleń dostarczonym przez dostawcę plików cookie) >
2. TrackingID (w tabeli plików cookie w schemacie pomostowym) >
3. CookieID (w schemacie marketingowym tabeli plików cookie)
Wreszcie, nie bój się popełniać błędów. Zbuduj swój model, przetestuj go, a jeśli popełniłeś błąd, po prostu napraw go, opłucz i powtórz.

Zanim zaczniemy, podzielimy nasze atrybuty na trzy szerokie typy danych: Sprzedaż, Zakupy i Produkty, które odnoszą się zarówno do Sprzedaży, jak i Zakupów. To daje nam nasze dane wyjściowe w nieznormalizowanej formie, jak pokazano poniżej:

```swift
Sales

Sales Order Date
Sales Order Number
Sales Person Name
Sales Person Email
Sales Area Name
Sales Area Manager
Customer Company Name
Customer Contact Name
Customer Contact Email
Customer Invoice Address
Customer Delivery Addresses
Sales Order Delivery Due Date
Sales Order Delivery Actual Date
Sales Order Item
Sales Order Quantities
Currier Used for Delivery

Purchasing
Supplier Name
Supplier Contact Name
Supplier Contact Email
Supplier Address
Purchase Order Date
Purchase Order Number
Purchase Order Items
Purchase Order Quantities

Products
Product Name
Product Stock Level
Product Next Manufacture Date
Product Next Manufacture Quantity
Back Order Manufacturing ID
Product Type Name
Product Type Description
Product Category Name
Product Category Description
Product Subcategory Name
Product Subcategory Description
```

**Pierwsza postać normalna (1NF)**
Aby relacja znalazła się w 1NF musi spełniać następujące zasady:

• Wszystkie atrybuty są niepodzielne
• Każdy atrybut ma unikalną nazwę
• Kolejność atrybutów nie ma znaczenia
• Grupy powtarzające się są usuwane
• Wszystkie wiersze muszą być unikalne (musi istnieć klucz)



> NOTATKA
>
> W normalizacji relacja jest odpowiednikiem jednostki w ERD lub tabeli w bazie danych

W naszym przykładzie każdy atrybut ma już unikalną nazwę, a kolejność atrybutów nie ma znaczenia, więc dobrze zaczynamy. Musimy jednak rozbić niektóre z naszych atrybutów, aby były atomowe. Musimy także sprawdzić nasze dane pod kątem powtarzających się grup, czyli grup kolumn, które się powtarzają. Musimy także znaleźć klucz kandydujący *candidate key*, czyli zbiór kolumn tworzących minimalny superklucz. Super klucz to klucz, który jednoznacznie identyfikuje każdy wiersz i jest minimalny, jeśli usunięcie jakiejkolwiek kolumny w tym kluczu spowodowałoby brak możliwości jednoznacznej identyfikacji każdego wiersza.

Najpierw zidentyfikujmy klucze, które będą używane do jednoznacznej identyfikacji wierszy w każdej relacji. Dane sprzedaży można jednoznacznie zidentyfikować na podstawie numeru zamówienia sprzedaży. Nie ma zatem potrzeby stosowania klucza złożonego, czyli klucza składającego się z wielu atrybutów. Jeśli klucz składa się z pojedynczego atrybutu, nazywa się go atrybutem głównym *Prime Attribute*.

Dane dotyczące zakupów można jednoznacznie zidentyfikować na podstawie numeru zamówienia. Dlatego jest to superklucz sam w sobie i możemy go ponownie użyć jako klucza kandydującego z jednym atrybutem.

Danych Produktów nie można zidentyfikować na podstawie samej Nazwy Produktu, ponieważ niektóre części zamawiane u dostawców mają tę samą nazwę, co produkty sprzedawane klientom. Dlatego musimy użyć zarówno nazwy produktu, jak i nazwy typu produktu, aby jednoznacznie zidentyfikować dowolny rekord, a zatem te dwa atrybuty utworzą nasz klucz kandydujący.

Ten proces pomógł nam już wykryć pierwszy problem z naszym modelem danych. Zarówno nasze dane dotyczące sprzedaży, jak i dane dotyczące zakupów zawierają nazwy produktów (odpowiednio pozycje zamówienia sprzedaży i pozycje zamówienia zakupu). Problem polega na tym, że odkryliśmy, że do jednoznacznej identyfikacji Produktu wymagana jest zarówno nazwa produktu, jak i nazwa typu produktu. Dlatego do każdego z tych podmiotów dodajmy nazwę typu produktu. Do każdej relacji łączącej się z relacją Produkty powinniśmy także dodać relacje. Gdy już tam jesteśmy, zmieńmy nazwy atrybutu Pozycje zamówienia sprzedaży i atrybutu Pozycje zamówienia zakupu na Nazwa produktu, aby relacja była jasna.

Następnie powinniśmy szukać wartości nieatomowych. Wartości nieatomowe w naszych nieznormalizowanych danych, które musimy rozbić, to:

- Imię sprzedawcy – podzielimy to na imię sprzedawcy i nazwisko sprzedawcy

- Menedżer obszaru sprzedaży – podzielimy to na imię i nazwisko menedżera obszaru sprzedaży

- Imię osoby kontaktowej klienta — które podzielimy na Imię osoby kontaktowej i Nazwisko osoby kontaktowej

- Adres faktury klienta – który podzielimy na następujące atrybuty:

  - Adres do faktury Ulica
  - Obszar adresu faktury
  - Miasto adresu faktury
  - Adres do faktury, kod pocztowy
  - Kraj adresu faktury

- Adres dostawy klienta – który podzielimy na następujące atrybuty:

  - Adres dostawy ul

  - Obszar adresu dostawy

  - Adres dostawy Miasto

  - Adres dostawy Kod pocztowy

  - Kraj adresu dostawy

- Nazwa produktu i nazwa typu produktu (w relacji sprzedaży) – to trudne pytanie. Ten atrybut będzie przechowywać wiele produktów, które zostały zamówione w ramach jednego zamówienia, ale tak naprawdę nie możemy podzielić tego na wiele atrybutów, ponieważ nie ma określonej liczby pozycji, które można umieścić w jednym zamówieniu. Dlatego rozdzielimy to na inną relację i przeniesiemy Numer Zamówienia Sprzedaży, który wybraliśmy jako klucz, aby nowa relacja mogła ponownie połączyć się z pierwotną relacją. Zabierzemy ze sobą również ilości zamówienia sprzedaży (które skrócimy do ilości), które w przeciwnym razie miałyby ten sam problem.
- Nazwa kontaktu z dostawcą – którą podzielimy na Imię osoby kontaktowej z dostawcą i Nazwisko osoby kontaktowej z dostawcą
- Adres dostawcy – który podzielimy na następujące atrybuty:
  - Adres dostawcy ul
  - Obszar adresowy dostawcy
  - Adres dostawcy Miasto
  - Adres dostawcy Kod pocztowy
  - Kraj adresu dostawcy
- Nazwa produktu i Nazwa typu produktu (w relacji Zakupy) – Podobnie jak w relacji Sprzedaż, będziemy musieli przenieść te atrybuty do nowej relacji i przenieść Numer zamówienia zakupu, abyśmy mogli połączyć podmioty. Ponownie zastosujemy atrybut Ilości zamówienia zakupu i skrócimy go do Ilość.



Na koniec musimy poszukać powtarzających się grup. Być może zauważyłeś już na podstawie powyższych punktorów, że atrybuty tworzące Adres do faktury są dokładnym powtórzeniem atrybutów tworzących Adres Dostawy w relacji sprzedaży. Dlatego oba te adresy powinniśmy przenieść w nową relację, którą nazwiemy Adresami. Do nowej relacji dodamy dodatkowy atrybut, dla którego kluczem kandydującym będzie ulica i kod pocztowy. Następnie dwukrotnie przekażemy je z powrotem do relacji sprzedaży, raz w przypadku adresu do faktury i ponownie w przypadku adresu dostawy.

> WSKAZÓWKA
>
> Chociaż nie jest to wyraźnie wymagane w zasadach normalizacji, w tym momencie sensowne byłoby dodanie adresu dostawcy również do relacji Adresy. Będziemy wtedy mieli pojedynczą relację, która przechowuje wszystkie adresy w naszej bazie danych. Minimalizuje to powielanie, a w konkretnym przypadku adresu oznacza, że pojedyncza walidacja adresu pozostawia go gotowym do wykorzystania przez dowolny podmiot.



> GENERALIZACJA DANYCH
>
> Dodając wszystkie trzy typy adresów do pojedynczej relacji, wykonaliśmy technikę modelowania danych zwaną generalizacją, w której łączymy atrybuty ze zbioru relacji lub encji i tworzymy nową encję, która jest bardziej ogólna.
>
> Bardziej zaawansowanym typem uogólnienia byłoby utworzenie nadtypów i podtypów. Dzięki tej technice możemy na przykład zdecydować się na uogólnienie sprzedawców, kontaktów z klientami, kontaktów z dostawcami i kierowników obszarów sprzedaży. Wszystkie ich wspólne atrybuty, takie jak imię, nazwisko i adres e-mail, zostaną przeniesione do encji Osoby.
>
> Jednak niektóre typy osób mogą mieć unikalne atrybuty. Na przykład sprzedawca może mieć także atrybuty Cel sprzedaży i Numer pracownika. W tym przypadku moglibyśmy utworzyć dodatkową encję dla Sprzedawców, która zawierałaby unikalne atrybuty i ponownie połączyć się z encją Osoby.
>
> W tym przypadku Ludzie stają się supertypem. Sprzedawcy to podtyp encji Osoby. Dobrym przykładem tego modelowania jest baza danych AdventureWorks, która jest przykładową bazą danych SQL Server. W tej bazie danych tabela Pracownik jest podtypem nadtypu Osoba.
>
> Bazę danych AdventureWorks można pobrać ze strony https://github.com/Microsoft/sql-server-samples/releases/download/adventureworks/AdventureWorks2022.bak

Poniższa struktura ilustruje, w jaki sposób nasze dane są modelowane w 1NF, korzystając z powyższych punktów dyskusji:

```swift
    Sales

    Sales Order Date
PK  Sales Order Number
    Sales Person First Name
    Sales Person Last Name
    Sales Person Email
    Sales Area Name
    Sales Area Manager First Name
    Sales Area Manager Last Name
    Customer Company Name
    Customer Contact First Name
    Customer Contact Last Name
    Customer Contact Email
FK  Invoice Address Street
FK  Invoice Address Zip Code
FK  Delivery Address Street
FK  Delivery Address Zip Code
    Sales Order Delivery Due Date
    Sales Order Delivery Actual Date
    Currier Used for Delivery
    
      Sales Order Details
PK FK Product Name
PK FK Product Type Name
      Quantity
PK FK Sales Order Number
    
    Address
PK  Street
    Area
    City
PK  Zip Code
    
    Purchasing
    Supplier Name
    Supplier Contact First Name
    Supplier Contact Last Name
    Supplier Contact Email
FK  Supplier Address Street
FK  Supplier Address Zip Code
    Purchase Order Date
PK  Purchase Order Number
    
      Purchase Order Details
PK FK Product Name
PK FK Product Type Name
      Quantity
PK FK Purchase Order Number
    
    Products
PK  Product Name
    Product Stock Level
    Product Next Manufacture Date
    Product Next Manufacture Quantity
    Back Order Manufacturing ID
PK  Product Type Name
    Product Type Description
    Product Category Name
    Product Category Description
    Product Subcategory Name
    Product Subcategory Description
```

To świetnie. Nasze relacje zaczynają bardziej przypominać schemat bazy danych. Jednak posiadanie bazy danych modelowanej na 1NF jest zwykle uważane za zły projekt. Przejdźmy więc dalej i zacznijmy zastanawiać się, jak przenieść naszą bazę danych do 2NF.



**Druga postać normalna (2NF)**
Aby relacje były w 2NF muszą spełniać następujące zasady:

• Relacje muszą być w 1NF
• Wszystkie atrybuty muszą być funkcjonalnie zależne od całości klucza

OK, więc wiemy, że nasze relacje są już w 1NF, ale jak zapewnić zależność funkcjonalną od całości klucza? Cóż, po pierwsze warto zauważyć, że ta zasada dotyczy tylko relacji, które mają klucz złożony. Z definicji, jeśli klucz naturalny składa się z atrybutu pierwszego, wówczas wszystkie inne atrybuty muszą być od niego zależne.

W naszym przykładzie mamy tylko jeden atrybut, który nie jest zależny od wszystkich atrybutów w kluczu złożonym. Jest to atrybut Opis typu produktu w relacji Produkt. Ten atrybut zależy wyłącznie od atrybutu Nazwa typu produktu. Dlatego wyodrębnijmy atrybuty Nazwa typu produktu i Opis typu produktu w osobną relację, gdzie Nazwa typu produktu stanie się atrybutem głównym. Przesuniemy również nazwę typu produktu z powrotem do relacji produktu jako klucz obcy.

Dlatego poniższe dane przedstawiają nasze dane modelowane w 2NF:

```swift
    Sales

    Sales Order Date
PK  Sales Order Number
    Sales Person First Name
    Sales Person Last Name
    Sales Person Email
    Sales Area Name
    Sales Area Manager First Name
    Sales Area Manager Last Name
    Customer Company Name
    Customer Contact First Name
    Customer Contact Last Name
    Customer Contact Email
FK  Invoice Address Street
FK  Invoice Address Zip Code
FK  Delivery Address Street
FK  Delivery Address Zip Code
    Sales Order Delivery Due Date
    Sales Order Delivery Actual Date
    Currier Used for Delivery
    
      Sales Order Details
PK FK Product Name
PK FK Product Type Name
      Quantity
PK FK Sales Order Number
    
    Address
PK  Street
    Area
    City
PK  Zip Code
    
    Purchasing
    Supplier Name
    Supplier Contact First Name
    Supplier Contact Last Name
    Supplier Contact Email
FK  Supplier Address Street
FK  Supplier Address Zip Code
    Purchase Order Date
PK  Purchase Order Number
    
      Purchase Order Details
PK FK Product Name
PK FK Product Type Name
      Quantity
PK FK Purchase Order Number
    
      Products
PK    Product Name
      Product Stock Level
      Product Next Manufacture Date
      Product Next Manufacture Quantity
      Back Order Manufacturing ID
PK FK Product Type Name
      Product Category Name
      Product Category Description
      Product Subcategory Name
      Product Subcategory Description
    
    Product Types
PK  Product Type Name
    Product Type Description
```

**Trzecia postać normalna (3NF)**
Na koniec musimy przekształcić nasze relacje w 3NF. Reguła 3NF stanowi:

• Relacje muszą być w 2NF
• Wszystkie atrybuty muszą być nieprzechodnie zależne od klucza
Nasze relacje są już w 2NF, co jest świetne. Musimy jednak zapewnić zależność nieprzechodnią, co oznacza, że atrybuty nie są zależne od żadnych atrybutów innych niż pierwsze w relacji.

Jeśli przejrzymy nasze dane, zobaczymy, że mamy sporo przykładów atrybutów, które bardziej zależą od atrybutów innych niż pierwsze niż od klucza kandydującego. Pierwszy przykład dotyczy relacji sprzedaży. Imię sprzedawcy i nazwisko sprzedawcy. Rozsądnie można oczekiwać, że adres e-mail sprzedawcy będzie unikalny, co oznacza, że będzie można go wykorzystać do identyfikacji każdej unikalnej kombinacji tych atrybutów. W związku z tym przeniesiemy wszystkie trzy atrybuty do nowej relacji sprzedawcy, w której adres e-mail sprzedawcy jest atrybutem głównym, i przywrócimy atrybut E-mail sprzedawcy do relacji sprzedaży jako klucz obcy.

Relacja sprzedaży zawiera także imię i nazwisko kierownika obszaru sprzedaży. Obydwa te atrybuty można jednoznacznie zidentyfikować za pomocą atrybutu Nazwa obszaru sprzedaży. Dlatego sprowadzimy je do relacji Obszary sprzedaży i zachowamy Nazwę obszaru sprzedaży jako klucz obcy.

Imię osoby kontaktowej klienta, nazwisko osoby kontaktowej klienta, adres e-mail klienta, wraz z kluczami adresu dostawy i kluczami adresu faktury można jednoznacznie zidentyfikować za pomocą atrybutu Nazwa firmy klienta. Dlatego przeniesiemy je do relacji Klienci i ponownie dołączymy do klucza Nazwa firmy klienta. Jest to jednak interesujący przykład, ponieważ po utworzeniu tej nowej relacji możesz zauważyć, że możemy teraz również zidentyfikować imię osoby kontaktowej klienta i nazwisko osoby kontaktowej klienta za pomocą adresu e-mail klienta. Dlatego powinniśmy przenieść te atrybuty do jeszcze jednej relacji, zwanej Kontaktem z Klientem, która ponownie połączy się z relacją Klienta, używając jako klucza adresu e-mail kontaktu z Klientem.

W podobny sposób, gdy spojrzymy na relację Zakupy, zobaczymy, że imię osoby kontaktowej z dostawcą, nazwisko osoby kontaktowej z dostawcą i klucze adresowe dostawcy można jednoznacznie zidentyfikować za pomocą nazwy dostawcy i dlatego powinny zostać przeniesione do relacji z dostawcami . Jednak imię osoby kontaktowej z dostawcą, nazwisko osoby kontaktowej z dostawcą i adres e-mail kontaktu z dostawcą powinny następnie przejść w dół do relacji osoby kontaktowej z dostawcą, wpisanej w adresie e-mail kontaktu z dostawcą.

> WSKAZÓWKA
>
> Kontakty z dostawcami, Kontakty z klientami i Sprzedawca byłyby dobrymi kandydatami do uogólnienia. Gdybyśmy chcieli uogólnić te dane, moglibyśmy stworzyć jedną encję Kontakty, która zawierałaby wszystkie typy kontaktów. Aby ułatwić filtrowanie, moglibyśmy utworzyć w encji atrybut Typ kontaktu. Podmiot ten dołączyłby następnie do podmiotów Dostawcy, Klienci i Sprzedaż.

W relacji Produkty nazwa kategorii produktu, opis kategorii produktu i opis podkategorii produktu mogą być jednoznacznie zidentyfikowane poprzez nazwę podkategorii produktu, dlatego przeniesiemy te atrybuty w dół do relacji Podkategorie produktów. Co więcej, powinniśmy następnie przenieść Opis kategorii produktu w dół do relacji Kategorie produktów, która jest powiązana z atrybutem Nazwa kategorii produktu.

Wreszcie atrybuty ProductNextManufactureDate i ProductNextManufactureQuantity można zidentyfikować za pomocą atrybutu BackOrderManufacturingID, dlatego przeniesiemy te atrybuty do nowej relacji Zamówienia zaległe.

Powinniśmy teraz stwierdzić, że nasze dane znajdują się w formacie 3NF, jak pokazano poniżej:

```swift
    Sales

    Sales Order Date
PK  Sales Order Number
FK  Sales Person Email
FK  Sales Area Name
FK  Customer Company Name
    Sales Order Delivery Due Date
    Sales Order Delivery Actual Date
    Currier Used for Delivery
    
    Sales Person
    Sales Person First Name
    Sales Person Last Name
PK  Sales Person Email
    
    Sales Areas
PK  Sales Area Name
    Sales Area Manager First Name
    Sales Area Manager Last Name
    
    Customer
PK  Customer Company Name
FK  Customer Contact Email
FK  Invoice Address Street
FK  Invoice Address Zip Code
FK  Delivery Address Street
FK  Delivery Address Zip Code
    
    Customer Contact
    Customer Contact First Name
    Customer Contact Last Name
PK  Customer Contact Email
    
      Sales Order Details
PK FK Product Name
PK FK Product Type Name
      Quantity
PK FK Sales Order Number

    Address
PK  Street
    Area
    City
PK  Zip Code
    
    Purchasing
FK  Supplier Name
    Purchase Order Date
PK  Purchase Order Number
    
    Suppliers
PK  Supplier Name
FK  Supplier Contact Email
FK  Supplier Address Street
FK  Supplier Address Zip Code
    
    Supplier Contact
    Supplier Contact First Name
    Supplier Contact Last Name
PK  Supplier Contact Email
    
      Purchase Order Details
PK FK Product Name
PK FK Product Type Name
      Quantity
PK FK Purchase Order Number
    
      Products
PK    Product Name
      Product Stock Level
FK    Back Order Manufacturing ID
PK FK Product Type Name
FK    Product Subcategory Name
    
    Product Tupes
PK  Product Type Name
    Product Type Description
    
    Product SubCategories
FK  Product Category Name
PK  Product Subcategory Name
    Product Subcategory Description
    
    Product Categories
PK  Product Category Name
    Product Category Description

    BackOrders
    ProductNextManufactureDate
    ProductNextManufactureQuantity
PK    BackOrderManufacturingID
```

Powinniśmy teraz poświęcić czas na uporządkowanie kilku rzeczy. Po pierwsze, nazwy naszych relacji. Po modelowaniu Sales nie jest już znaczącą nazwą danych, które reprezentuje. Zmieńmy to na Nagłówek zamówienia sprzedaży `Sales Order Header`. Ponadto Zakupy nie reprezentują już odpowiednio swoich atrybutów, dlatego powinniśmy zmienić je na Nagłówek zamówienia zakupu `Purchase Order Header`.

Szerokie klucze podstawowe są złe. Nie będę tu wchodził w szczegóły, bo będziemy o tym mówić przy następnym błędzie, ale obecnie wszystkie nasze klucze są kluczami naturalnymi, czyli kluczami mającymi znaczenie biznesowe. Tam, gdzie mamy klucze złożone, powinniśmy zastąpić je kluczami sztucznymi, czyli kluczami przechowującymi dowolną wartość bez znaczenia biznesowego. Zwykle liczba rosnąca. Powinniśmy także zastąpić klucze tekstowe kluczami sztucznymi. Ponownie sprawia to, że klucze są węższe.

Mając to na uwadze, wprowadzimy następujące zmiany:

Sales Person otrzyma nowy klucz o nazwie Sales Person ID
Sales Areas otrzymają nowy klucz o nazwie Sales Area ID
Customers otrzymają nowy klucz o nazwie Customer ID
Customer Contacts otrzymają nowy klucz o nazwie Customer Contact ID
Sales Order Details otrzymają nowy klucz o nazwie Sales Order Details ID
Addresses otrzymają nowy klucz o nazwie Address ID
Suppliers otrzymają nowy klucz o nazwie Supplier ID
Supplier Contacts otrzymają nowy klucz o nazwie Supplier Contact ID
Purchase Order Details otrzymają nowy klucz o nazwie Purchase Order Details ID
Products otrzymają nowy klucz o nazwie Product ID
Product Types otrzymają nowy klucz o nazwie Product Type ID
Product Subcategories otrzymają nowy klucz o nazwie Product Subcategory ID
Product Categories otrzymają nowy klucz o nazwie Product Category ID

Na koniec powinniśmy usunąć wszystkie spacje z nazw naszych relacji i atrybutów. Ma to na celu uczynienie ich przyjaznymi dla baz danych podczas tworzenia tabel i kolumn w naszej bazie danych. Jeśli w nazwach relacji i atrybutów mamy jakieś znaki specjalne lub słowa zastrzeżone, czyli słowa używane przez SQL Server, takie jak „SELECT” lub „table”, powinniśmy rozważyć ich zmianę. Gdybyśmy nie wykonali tego kroku, musielibyśmy używać rozdzielanych identyfikatorów podczas odwoływania się do obiektów w SQL Server. Oznacza to umieszczenie nazwy w nawiasach kwadratowych. Na przykład polecenie SELECT MyColumn FROM MyTable zmieni się w SELECT [MojaKolumna] FROM [MojaTabela] Identyfikatory rozdzielane są uważane za złą praktykę, ponieważ zaśmiecają nasz kod i ponieważ w przypadku konieczności ich użycia nie przestrzegaliśmy standardowych zasad i konwencji dotyczących nazewnictwa identyfikatorów .

Skończyliśmy już projekt, ale jak możemy go przetestować? Nie ma na to jednoznacznej i szybkiej odpowiedzi, ale będziemy mieli dobry pomysł, jeśli stworzymy ERD. Badając ERD, powinniśmy zauważyć, że wszystkie podmioty, które wymagają połączenia, są łączone za pomocą typu złączenia 1:many. Jeśli znajdziemy jakiekolwiek złączenia wiele:many lub 1:1, oznacza to, że w modelu prawdopodobnie wystąpił błąd. Zobaczmy więc, jak będzie wyglądał nasz model, jeśli zbudujemy ERD. Ten ERD można zobaczyć na rysunku 4.5.

##### Figure 4.5 ERD from normalized design

![img](https://drek4537l1klr.cloudfront.net/carter/v-1/Figures/04__image005.png)

Naszym ostatnim zadaniem jest napisanie skryptu, który utworzy obiekty naszej bazy danych. Skrypt z Listingu 4.1 wygeneruje te obiekty, z których część wykorzystamy w późniejszych błędach.

> NOTATKA
>
> W scenariuszu wkradł się zamierzony błąd. Tabela BuyOrderDetails zawiera kolumnę BuyOrderNumber, ale nie utworzono klucza obcego, który umożliwiłby jej ponowne połączenie z tabelą BuyOrderHeaders. Konsekwencje tego przeanalizujemy w dalszej części tego rozdziału.

```sql
CREATE DATABASE MagicChoc ;
GO
 
USE MagicChoc 
GO
 
CREATE TABLE dbo.BackOrders (
    BackOrderManufacturingID          INT    NOT NULL    IDENTITY    PRIMARY KEY,
    ProductNextManufactureDate        DATE   NOT NULL,
    ProductNextManufactureQuantity    INT    NOT NULL
) ;
 
CREATE TABLE dbo.ProductCategories (
    ProductCategoryID             SMALLINT         NOT NULL    IDENTITY    PRIMARY KEY,
    ProductCategoryName           NVARCHAR(64)     NOT NULL,
    ProductCategoryDescription    NVARCHAR(256)    NULL
) ;
 
CREATE TABLE dbo.ProductSubcategories (
    ProductSubcategoryID             SMALLINT         NOT NULL    IDENTITY    PRIMARY KEY,
    ProductCategoryID                SMALLINT         NOT NULL    REFERENCES dbo.ProductCategories(ProductCategoryID),
    ProductSubcategoryName           NVARCHAR(64)     NOT NULL,
    ProductSubcategoryDescription    NVARCHAR(256)    NULL
) ;
 
CREATE TABLE dbo.ProductTypes (
    ProductTypeID             SMALLINT        NOT NULL    IDENTITY    PRIMARY KEY,
    ProductTypeName           NVARCHAR(64)    NOT NULL,
    ProductTypeDescription    NVARCHAR(256)   NOT NULL
) ;
 
CREATE TABLE dbo.Products (
    ProductID                   INT             NOT NULL    IDENTITY    PRIMARY KEY,
    ProductName                 NVARCHAR(64)    NOT NULL,
    ProductStockLevel           INT             NOT NULL,
    BackOrderManufacturingID    INT             NULL    REFERENCES dbo.BackOrders(BackOrderManufacturingID),
    ProductTypeID               SMALLINT        NOT NULL    REFERENCES dbo.ProductTypes(ProductTypeID),
    ProductSubcategoryID        SMALLINT        NOT NULL    REFERENCES dbo.ProductSubcategories(ProductSubcategoryID)
) ;
 
CREATE TABLE dbo.Addresses (
    AddressID    INT              NOT NULL    IDENTITY    PRIMARY KEY,
    Street       NVARCHAR(128)    NOT NULL,
    Area         NVARCHAR(64)     NULL,
    City         NVARCHAR(64)     NOT NULL,
    ZipCode      NVARCHAR(10)     NOT NULL
) ;
 
CREATE TABLE dbo.SupplierContacts (
    SupplierContactID           INT              NOT NULL    IDENTITY    PRIMARY KEY,
    SupplierContactFirstName    NVARCHAR(32)     NOT NULL,
    SupplierContactLastName     NVARCHAR(32)     NOT NULL,
    SupplierContactEmail        NVARCHAR(256)    NOT NULL
) ;
 
CREATE TABLE dbo.Suppliers (
    SupplierID           INT             NOT NULL    IDENTITY    PRIMARY KEY,
    SupplierName         NVARCHAR(32)    NOT NULL,
    SupplierContactID    INT             NOT NULL    REFERENCES dbo.SupplierContacts(SupplierContactID),
    SupplierAddressID    INT             NOT NULL    REFERENCES dbo.Addresses(AddressID)
) ;
 
CREATE TABLE dbo.PurchaseOrderHeaders (
    PurchaseOrderNumber    INT     NOT NULL    PRIMARY KEY,
    SupplierID             INT     NOT NULL    REFERENCES dbo.Suppliers(SupplierID),
    PurchaseOrderDate      DATE    NOT NULL
) ;
 
CREATE TABLE dbo.PurchaseOrderDetails (
    PurchaseOrderDetailsID    INT    NOT NULL    IDENTITY    PRIMARY KEY,
    ProductID                 INT    NOT NULL    REFERENCES dbo.Products(ProductID),
    Quantity                  INT    NOT NULL,
    PurchaseOrderNumber       INT    NOT NULL
) ;
 
CREATE TABLE dbo.CustomerContacts (
    CustomerContactID           INT              NOT NULL    IDENTITY    PRIMARY KEY,
    CustomerContactFirstName    NVARCHAR(32)     NOT NULL,
    CustomerContactLastName     NVARCHAR(32)     NOT NULL,
    CustomerContactEmail        NVARCHAR(256)    NOT NULL
) ;
 
CREATE TABLE dbo.Customers (
    CustomerID             INT             NOT NULL    IDENTITY    PRIMARY KEY,
    CustomerCompanyName    NVARCHAR(32)    NOT NULL,
    CustomerContactID      INT             NOT NULL    REFERENCES dbo.CustomerContacts(CustomerContactID),
    InvoiceAddressID       INT             NOT NULL    REFERENCES dbo.Addresses(AddressID),
    DeliveryAddressID      INT             NOT NULL    REFERENCES dbo.Addresses(AddressID)
) ;
 
CREATE TABLE dbo.SalesAreas (
    SalesAreaID                  SMALLINT        NOT NULL    IDENTITY    PRIMARY KEY,
    SalesAreaName                NVARCHAR(32)    NOT NULL,
    SalesAreaManagerFirstName    NVARCHAR(32)    NOT NULL,
    SalesAreaManagerLastName     NVARCHAR(32)    NOT NULL
) ;
 
CREATE TABLE dbo.SalesPersons (
    SalesPersonID           SMALLINT         NOT NULL    IDENTITY    PRIMARY KEY,
    SalesPersonFirstName    NVARCHAR(32)     NOT NULL,
    SalesPersonLastName     NVARCHAR(32)     NOT NULL,
    SalesPersonEmail        NVARCHAR(256)    NOT NULL
) ;
 
CREATE TABLE dbo.SalesOrderHeaders (
    SalesOrderNumber               NCHAR(12)       NOT NULL    PRIMARY KEY,
    SalesOrderDate                 DATE            NOT NULL,
    SalesPersonID                  SMALLINT        NOT NULL    REFERENCES dbo.SalesPersons(SalesPersonID),
    SalesAreaID                    SMALLINT        NOT NULL    REFERENCES dbo.SalesAreas(SalesAreaID),
    CustomerID                     INT             NOT NULL    REFERENCES dbo.Customers(CustomerID),
    SalesOrderDeliveryDueDate      DATE            NOT NULL,
    SalesOrderDeliveryActualDate   DATE            NULL,
    CurrierUsedforDelivery         NVARCHAR(32)    NOT NULL
) ;
 
CREATE TABLE dbo.SalesOrderDetails (
    SalesOrderDetailsID    INT          NOT NULL    IDENTITY    PRIMARY KEY,
    ProductID              INT          NOT NULL    REFERENCES dbo.Products(ProductID),
    Quantity               INT          NOT NULL,
    SalesOrderNumber       NCHAR(12)    NOT NULL    REFERENCES dbo.SalesOrderHeaders(SalesOrderNumber)
) ;
GO
```

Modelowanie danych za pomocą normalizacji może początkowo wydawać się skomplikowane, ale po kilku próbach zaczyna przychodzić naturalnie. Z pewnością ma wiele zalet w porównaniu z podejściem opartym na modelowaniu ocen i jest znacznie mniej podatny na błędy. Pozwala nam zastosować metodologię, która dobrze i naprawdę przetrwała próbę czasu.





### 4.2 #12 Używanie szerokiego klucza podstawowego
Jak wspomniano w poprzedniej sekcji, jako klucz podstawowy tabeli SalesOrderHeaders wybraliśmy SalesOrderNumber, ale format numerów zamówień to ABC1234D-E12. Oznacza to, że nasza kolumna ma typ danych NCHAR(12). Oznacza to, że dla każdego wiersza kolumna zajmuje 24 bajty miejsca, w przeciwieństwie do 4 bajtów miejsca, które zajmowałby INT. Nawet BIGINT zajmie tylko 8 bajtów miejsca.

Mówiliśmy o korzyściach płynących z używania najmniejszych możliwych typów danych w rozdziale 3, ale te dane zawierają litery. W zależności od reguł biznesowych możemy użyć CHAR(12), ale nawet to zajmie 12 bajtów pamięci na wiersz, więc co możemy zrobić? A jakie jest znaczenie tego, że jest to kolumna klucza podstawowego?

Aby odpowiedzieć na te pytania, zastanówmy się nad naszą strategią indeksowania tabeli SalesOrderHeaders. Domyślnie kolumna klucza podstawowego będzie miała indeks klastrowy. Indeks klastrowy organizuje dane w drzewo B. Jest to struktura składająca się z wielu warstw stron indeksowych, które dostarczają wskaźników do stron indeksowych znajdujących się na niższych poziomach struktury. Zawsze istnieje jeden poziom główny, składający się z jednej strony, zwany mapą alokacji indeksu (IAM). Istnieje wówczas zero lub więcej poziomów pośrednich, w zależności od wielkości stołu. Dolny poziom (liść) indeksu klastrowego to rzeczywiste strony danych tabeli.

Ponieważ dane w indeksie są uporządkowane, rekordy można znaleźć bardzo szybko, korzystając z operacji zwanej wyszukiwaniem indeksu klastrowego. Ta operacja przechodzi przez poziomy indeksu w celu zlokalizowania rekordu. Jest to przeciwieństwo skanowania indeksu klastrowego, które jest operacją, podczas której należy przeszukać każdą stronę na poziomie liścia indeksu, aż do znalezienia wymaganego rekordu. Operacje te, wraz z przykładem struktury drzewa B, pokazano na rysunku 4.6. Zauważysz, że za każdym razem, gdy przeszukiwany jest pojedynczy wiersz, maksymalna liczba stron odczytanych w operacji wyszukiwania jest równa liczbie poziomów drzewa B. Maksymalna liczba stron odczytanych podczas skanowania indeksu jest jednak równa liczbie stron danych, które składają się na poziom liścia indeksu, a tym samym tabelę. Jeśli operacja może zwrócić wiele wierszy, wówczas stosuje się wyszukiwanie w celu znalezienia punktu początkowego odczytu poziomu liścia.

> WSKAZÓWKA
>
> Ponieważ indeks klastrowy porządkuje strony danych samej tabeli, w tabeli może zawsze znajdować się tylko jeden indeks klastrowy. Tabela, która nie ma indeksu klastrowego, nazywana jest stertą. Na stercie rekordy są przechowywane w dowolnej kolejności. Sterty są często używane do celów takich jak tabele pomostowe, które zawierają duże wstawki danych efemerycznych, gdzie kolejność nie ma znaczenia, ponieważ może to zoptymalizować wydajność.

##### Figure 4.6 Clustered Index seeks and scans against a B-tree

![img](https://drek4537l1klr.cloudfront.net/carter/v-1/Figures/04__image006.png)





Firma powiedziała nam, że często wyszukuje zamówienia sprzedaży, filtrując datę zamówienia. Dlatego chcemy zoptymalizować te zapytania, tworząc indeks nieklastrowany w kolumnie SalesOrderDate. Indeks nieklastrowany tworzy strukturę B-Tree, która jest bardzo podobna do indeksu klastrowego. Różnica polega na tym, że zamiast poziomu liścia indeksu zawierającego rzeczywiste dane, zawiera on wskaźniki do danych w ramach indeksu klastrowego lub sterty. Ponieważ poziom liścia to tylko wskaźniki, oznacza to, że rzeczywiste dane nie są uporządkowane. Oznacza to, że w jednej tabeli możemy mieć wiele indeksów nieklastrowych.

> NOTATKA
>
> Jeśli indeks nieklastrowany zawiera kolumny WŁĄCZONE, wówczas rzeczywiste wartości danych tych kolumn zostaną zapisane w indeksie wraz ze wskaźnikiem do indeksu lub sterty.

Ponadto firma poinformowała nas, że zapytania dotyczące zamówień sprzedaży zazwyczaj obejmują informacje o obszarze sprzedaży i informacje o klientach. Dane te są przechowywane w różnych tabelach, dlatego będziemy chcieli utworzyć indeksy nieklastrowe również w kolumnach SalesAreaID i CustomerID, które są kluczami obcymi dla tych tabel. Tworząc indeksy nieklastrowe na kluczach obcych, będzie to oznaczać, że klucze są uporządkowane w taki sam sposób, jak ich odpowiedniki klucza podstawowego w tabelach, do których się łączymy. Może to pozwolić na bardziej wydajne operacje łączenia.

Aby zobaczyć konsekwencje posiadania szerokiego klucza podstawowego, dodajmy najpierw trochę danych do odpowiednich tabel. Skrypt z Listingu 4.2 dodaje niewielką ilość danych do odpowiednich tabel. Następnie tworzy indeksy nieklastrowane w tabeli SalesOrderHeaders. Nie ma potrzeby tworzenia indeksu klastrowego, ponieważ został on utworzony automatycznie, kiedy utworzyliśmy klucz podstawowy w tabeli.

```sql
INSERT INTO dbo.SalesPersons (SalesPersonFirstName, SalesPersonLastName, SalesPersonEmail)
VALUES 
    ('Robin', 'Wells', 'robin.wells@magicchoc.com'),
    ('Jack', 'Jones', 'jack.jones@magicchoc.com'),
    ('Jane', 'Smith', 'jane.smith@magicchoc.com') ;
 
INSERT INTO dbo.SalesAreas (SalesAreaName, SalesAreaManagerFirstName, SalesAreaManagerLastName)
VALUES
    ('US', 'Lucy', 'Sykes'),
    ('Euro', 'Ashwin', 'Kumar'),
    ('APAC', 'Emma', 'Roberts') ;
 
INSERT INTO dbo.Addresses (Street, Area, City, ZipCode)
VALUES 
    ('744 Saxon Rd', NULL, 'Crawfordsville', '47933'),
    ('267 Old York Ave.', NULL, 'Reno', '89523'),
    ('923 Taylor Ave.', NULL, 'Charlotte', '28205'),
    ('942 Cactus Street', NULL, 'Albany', '12203'),
    ('32 Selby Drive', NULL, 'Pittsfield', '01201'),
    ('65 New Street', 'Landford', 'Salisbury', 'SP5 2QP') ;
 
INSERT INTO dbo.CustomerContacts (CustomerContactFirstName, CustomerContactLastName, CustomerContactEmail)
VALUES
    ('Ralphie', 'Buchanan', 'Ralphie.Buchanan@Pitt.com'),
    ('Bettie', 'Peters', 'bpeters@cookingschmooking.co.uk'),
    ('Zackery', 'McEachern', 'Zackery.McEachern@wilsonindustries.com') ;
 
 
INSERT INTO dbo.Customers (CustomerCompanyName, CustomerContactID, InvoiceAddressID, DeliveryAddressID)
VALUES
    ('Pitt and Co', 1, 1, 2),
    ('Cooking Schmooking', 2, 3, 4),
    ('Wilson Industries', 3, 6, 6) ;
 
 
INSERT INTO dbo.SalesOrderHeaders (SalesOrderNumber, SalesOrderDate, SalesPersonID, SalesAreaID, CustomerID, SalesOrderDeliveryDueDate, SalesOrderDeliveryActualDate, CurrierUsedforDelivery)
VALUES 
    ('COO1634D-U06', '20230501', 1, 1, 1, '20230503', '20230503', 'LHD'),
    ('WIL1635D-E16', '20230616', 2, 2, 3, '20230630', NULL, 'GoodSpeed International'),
    ('PIT1636D-U04', '20230616', 3, 1, 1, '20230706', NULL, 'LHD') ;
GO
 
CREATE NONCLUSTERED INDEX NI_SalesOrderHeaders_OrderDate ON SalesOrderHeaders(SalesOrderDate) ;
CREATE NONCLUSTERED INDEX NI_SalesOrderHeaders_SalesAreaID ON SalesOrderHeaders(SalesAreaID) ;
CREATE NONCLUSTERED INDEX NI_SalesOrderHeaders_CustomerID ON SalesOrderHeaders(CustomerID) ;
```

Następnie, aby zademonstrować problem, musimy zobaczyć, co jest przechowywane na stronach indeksowych jednego z indeksów, które właśnie utworzyliśmy. Użyjmy tutaj indeksu NI_SalesOrderHeaders_OrderDate, ale możemy wybrać dowolny indeks, który nam się podoba.

Aby przeglądać strony indeksu, wymagane są trzy kroki. Pierwszym krokiem jest ustalenie ID indeksu naszego indeksu. To nie jest identyfikator obiektu, który jest unikalny w całej instancji. Zamiast tego jest to identyfikator indeksu w tabeli. Możemy wylistować indeks dla danego indeksu, przeszukując sys.indexes i filtrując według nazwy indeksu, jak pokazano na Listingu 4.3 poniżej.

Listing 4.3 Określ identyfikator indeksu

```swift
SELECT
    name
  , index_id
  , type_desc
FROM sys.indexes
WHERE name = 'NI_SalesOrderHeaders_OrderDate' ;
```

Identyfikator indeksu, który posiadam, to 2, ale może się okazać, że identyfikator jest inny, jeśli indeksy utworzono w innej kolejności. Teraz, gdy mamy już identyfikator indeksu, następnym krokiem jest użycie polecenia DBCC IND w celu wylistowania wszystkich stron tworzących indeks.

Polecenie wyświetla łańcuch IAM indeksu, od poziomu głównego do poziomu liścia. Dla każdej strony w indeksie zwracany jest jeden wiersz, który zawiera numery stron, poziom indeksu i wskaźniki stron umożliwiające poruszanie się po indeksie, a także inne przydatne informacje.

Polecenie akceptuje trzy parametry, mianowicie nazwę bazy danych (lub identyfikator bazy danych), nazwę tabeli i identyfikator indeksu. Listing 4.4 pokazuje, jak używać tego polecenia.

Listing 4.4 Użyj DBCC IND do odnalezienia numerów stron indeksowych

```swift
DBCC IND ('MagicChoc', 'SalesOrderHeaders', 2) ;
```

> UWAGA
>
> Moje wyniki pokazano poniżej, ale pamiętaj, że prawie na pewno zauważysz, że numery stron się różnią. Upewnij się, że w poniższych przykładach używasz własnych numerów stron.
>

Wyniki przedstawiono poniżej na rysunku 4.7. Zauważysz, że są tylko dwie strony. Pierwsza strona jest stroną główną indeksu. Zawsze będzie dokładnie jedna strona główna, niezależnie od rozmiaru indeksu. Nie ma poziomów pośrednich wskaźnika, ponieważ jest on za mały. Poziom liścia indeksu składa się z dokładnie jednej strony, ponieważ w tabeli znajdują się tylko trzy rekordy, więc wszystko mieści się wygodnie na jednej stronie.

##### Figure 4.7 Results of DBCC IND

![img](https://drek4537l1klr.cloudfront.net/carter/v-1/Figures/04__image007.png)

Ostatnim krokiem jest użycie innego polecenia DBCC, zwanego DBCC PAGE. Tego polecenia można użyć do wyświetlenia zawartości strony z danymi lub indeksu. Polecenie przyjmuje cztery parametry, mianowicie nazwę bazy danych, identyfikator pliku, identyfikator strony (pobrany z DBCC IND) oraz parametr określający opcję formatowania. Należy jednak pamiętać, że przed uruchomieniem DBCC PAGE musimy upewnić się, że flaga śledzenia 3604 jest włączona. Flagi śledzenia służą do włączania i wyłączania funkcji. Niektóre z nich omówimy w rozdziale 10. Ta konkretna flaga wysyła dane wyjściowe DBCC do konsoli. Listing 4.5 pokazuje, jak używać tego polecenia.

Listing 4.5 Użyj DBCC PAGE, aby wyświetlić zawartość strony

```swift
DBCC TRACEON(3604) ;
DBCC PAGE('MagicChoc', 1, 600, 3) ;
```

Wyniki mojego uruchomienia tego polecenia pokazane są na rysunku 4.8. Zgodnie z oczekiwaniami wartość klucza indeksu zobaczysz w kolumnie SalesOrderDate (Key). Ale spójrz na kolumnę obok niej, kolumnę SalesOrderNumber (klucz). Wartość klucza podstawowego jest przechowywana w każdym wierszu. Zapewnia to wskaźnik z powrotem do danych w indeksie klastrowym.

Dlatego klucz podstawowy zajmuje nie tylko 12 bajtów na wiersz tabeli, ale także 12 bajtów na każdy wiersz w każdym pojedynczym indeksie nieklastrowym w tabeli. Co gorsza, nasze indeksy nieklastrowe nie są unikalne. Jeśli indeks nieklastrowany jest unikalny, wartość klucza klastrowego jest przechowywana tylko na poziomie liścia indeksu. Jednak w przypadku nieunikalnych indeksów nieklastrowanych wartość klucza klastrowego jest przechowywana w każdym wierszu na wszystkich poziomach indeksu.

##### Figure 4.8 Results of DBCC PAGE

![img](https://drek4537l1klr.cloudfront.net/carter/v-1/Figures/04__image008.png)

Ten szeroki klucz nie tylko zużywa dodatkowe miejsce na dysku i pamięć, ale także pogarsza wydajność indeksów. Pamiętaj, że strona może przechowywać tylko 8000 bajtów danych. Dlatego im szerszy klucz, tym więcej stron indeksu potrzeba. Z kolei SQL Server musi następnie przeczytać więcej stron, aby pobrać wymagane dane.

W tym scenariuszu, w którym albo kluczem podstawowym jest szeroka kolumna, albo okazuje się, że nasz klucz podstawowy składa się z wielu kolumn, radzę utworzyć sztuczny klucz podstawowy i automatycznie wypełnić go `IDENTITY`. Jeśli musimy wymusić unikalność naszego klucza naturalnego, nadal możemy to osiągnąć, bez powiększania naszych indeksów klastrowych i nieklastrowych, budując unikalny indeks nieklastrowany w kolumnie.

> GUID JAKO KLUCZE PODSTAWOWE
>
> Jednym z przykładów szerokich kluczy podstawowych, jakie widziałem, jest użycie identyfikatorów GUID (globalnie unikalnych identyfikatorów).
>
> Identyfikatory GUID można wygenerować za pomocą funkcji NEWID() i zapewniają mechanizm generowania wartości unikalnych w całym serwerze. Czasami jednak musimy używać identyfikatorów GUID jako kluczy podstawowych. Na przykład istnieją produkty COTS (Commercial Off The Shelf), które wyraźnie tego wymagają.
>
> Istnieją dwie kwestie związane z używaniem identyfikatorów GUID jako klucza podstawowego. Pierwszym z nich jest rozmiar. Identyfikatory GUID mają szerokość 16 bajtów, więc musimy zadać sobie pytanie, czy naprawdę potrzebujemy wartości, która jest unikalna w skali globalnej, czy też potrzebujemy jej tylko unikalnej w kontekście tabeli? Jeśli musi być unikalny w całej bazie danych, mogą istnieć inne metody, takie jak funkcja SEQUENCE, które mogą osiągnąć to w bardziej odpowiedni sposób.
>
> Inną kwestią jest losowy charakter identyfikatora GUID. Indeks klastrowy z definicji porządkuje dane w tabeli według wartości indeksu klastrowego, który zwykle opiera się na kluczu podstawowym. Jeśli te wartości są losowe, SQL Server będzie musiał ciągle zmieniać kolejność stron danych. Oznacza to, że w indeksie może występować problem zwany podziałem strony.
>
> Kiedy nastąpi podział strony, dane zostaną przeniesione pomiędzy stronami, aby zapewnić zachowanie porządku. Doprowadzi to do zwiększenia operacji we/wy i zmniejszenia wydajności operacji UPDATE i INSERT. Dodatkowo doprowadzi to do fragmentacji indeksu, co omówimy w rozdziale 11. Fragmentacja zmniejszy wydajność operacji SELECT do czasu odbudowania indeksu.
>
> Ten problem można rozwiązać, stosując niski współczynnik wypełnienia indeksu klastrowego i odbudowując indeksy przy niskim współczynniku fragmentacji, ale ogólnie zalecam utworzenie nieklastrowanego klucza podstawowego, a następnie wygenerowanie indeksu klastrowego w kolumnie typu danych całkowitych. Następnie możemy utworzyć klucz podstawowy w kolumnie zawierającej identyfikatory GUID.



### 4.3 #13 Nieużywanie kluczy obcych
Brak ograniczeń klucza obcego to błąd, który widziałem u mniej doświadczonych programistów, zwykle podając powód, dla którego muszą szybko wstawić dane do tabeli, a wtedy ograniczenia sprawiają, że jest to zbyt czasochłonne. Najczęściej spotykałem się z takim poglądem w sytuacjach takich jak nasza, gdy klucz jest kluczem naturalnym, a zatem ma znaczenie biznesowe i jest kontrolowany przez aplikację frontendową.

Wyzwanie związane z tym tokiem myślenia polega na tym, że aplikacja frontendowa po prostu nie może zagwarantować integralności danych w taki sam sposób, jak ograniczenia SQL Server. Aby omówić tę kwestię bardziej szczegółowo, skorzystajmy z przykładu naszych tabel BuyOrderHeaders i BuyOrderDetails.

Być może pamiętasz, że w naszym skrypcie tworzenia bazy danych na liście 4.1 wystąpił błąd, w wyniku czego kolumna BuyOrderNumber istnieje w tabeli BuyOrderDetails, ale ponieważ nie utworzono klucza obcego, nie ma możliwości ponownego złączenia z tabelą BuyOrderHeaders.

Zanim przejdziemy dalej, uruchommy skrypt z Listingu 4.6, aby wstawić dane do odpowiednich tabel.

```sql
INSERT INTO dbo.ProductCategories (ProductCategoryName, ProductCategoryDescription)
VALUES
    ('Raw Ingridience', NULL),
    ('Machine Parts', 'Parts used by manufacturing for machine maintenance'),
    ('Misc', 'Office supplies and other miscelaneous stock items'),
    ('Services', 'Non-stock purchases, such as transport'),
    ('Confectionary Products', NULL),
    ('Non-confectionary Products', NULL) ;
 
INSERT INTO dbo.ProductSubcategories (ProductCategoryID, ProductSubcategoryName, ProductSubcategoryDescription)
VALUES
    (1, 'Chilled Ingredience', 'Ingredience that must be kept between 1C and 5C'),
    (1, 'Frozen Ingredience', 'Ingredience must be kept below -18C'),
    (1, 'Ambient Ingredience', 'Ingredience that should be kept in cool, dry storage'),
    (2, 'Line 1 Components', 'Components required for manufacturing line 1'),
    (2, 'Line 2 Components', 'Components required for manufacturing line 1'),
    (2, 'Line 3 Components', 'Components required for manufacturing line 1'),
    (2, 'Line 4 Components', 'Components required for manufacturing line 1'),
    (3, 'Office Supplies', 'Stationary, etc'),
    (3, 'Misc', NULL),
    (4, 'Curriers', NULL),
    (4, 'Building Maintenance', NULL),
    (5, 'Boxes of chocolates', NULL),
    (5, 'Sweets', NULL),
    (5, 'Chocolate Bars', NULL),
    (6, 'Packaging', 'Product Packaging'),
    (6, 'Merchandise', 'Non-core items which are procured and sold, such as mugs and branded gifts') ;
 
INSERT INTO dbo.ProductTypes (ProductTypeName, ProductTypeDescription)
VALUES
    ('Purchased Product', 'Products which are purchased'),
    ('Sold products', 'Products which are sold'),
    ('Traded Products', 'Products which are both bought and sold') ;
 
INSERT INTO dbo.SupplierContacts (SupplierContactFirstName, SupplierContactLastName, SupplierContactEmail)
VALUES
    ('John', 'Smith', 'john.smith@smithfields.com'),
    ('John', 'Doe', 'john.doe@unknownengineering.com'),
    ('Michael', 'Knight', 'mknight@knightridercurriers.com') ;
 
INSERT INTO dbo.Addresses (Street, City, ZipCode)   --x3 suppliers
VALUES
    ('8648 Columbia Street', 'Beachwood', '44122'),
    ('83 Addison Dr.', 'Westerville', '43081'),
    ('508 Mill Pond Street', 'Clinton Township', '48035') ;
 
INSERT INTO dbo.Suppliers (SupplierName, SupplierContactID, SupplierAddressID)
VALUES
    (‘Smithfeilds’, 1, 7),
    (‘Unknown Engineering’, 2, 8),
    (‘Knight Rider Curriers’, 3, 9) ;
 
INSERT INTO dbo.Products (ProductName, ProductStockLevel, ProductTypeID, ProductSubcategoryID)
VALUES
    ('Large head sprocket', 3, 1, 4),
    ('Long weight', 6, 1, 4),
    ('Staples', 8900, 1, 8),
    ('Magic Mug', 38, 3, 16),
    ('Massive Magic Box', 18, 2, 12),
    ('Delivery', -1, 3, 10) ;
```

Zasymulujemy teraz naszą aplikację frontendową wstawiającą zamówienia do tabeli, korzystając ze skryptu z Listingu 4.7. Aplikacja włącza XACT_ABORT i pakuje obie instrukcje INSERT w transakcję. To dobrze. Oznacza to, że jeśli jedno stwierdzenie zawiedzie, drugie również zawiedzie, co pomoże zachować spójność danych.

```sql
SET XACT_ABORT ON
 
BEGIN TRANSACTION
    SELECT @@TRANCOUNT
        INSERT INTO dbo.PurchaseOrderHeaders  (PurchaseOrderNumber, SupplierID, PurchaseOrderDate)
        VALUES 
            (6826, 2, '20230601'),
            (6827, 2, '20230617') ;
 
    INSERT INTO dbo.PurchaseOrderDetails (ProductID, Quantity, PurchaseOrderNumber)
    VALUES
          (4, 3, 6826),
        (5, 4, 6827),
        (4, 1, 6827) ;
COMMIT
```

Ale pomyślmy o  tym, że możemy potrzebować możliwości szybkiego i łatwego aktualizowania tabeli. Rozważmy więc następujący scenariusz. Mamy telefon od szefa zakupów, który mówi: „Wystąpił problem z zamówieniem. Musisz to naprawić w backendzie, ponieważ aplikacja nie pozwala mi sobie z tym poradzić. Proszę o przeniesienie zamówionych artykułów z zamówienia nr 6827 na zamówienie nr 6828 – i to szybko!”

Jesteśmy pod presją, więc zupełnie racjonalnie robimy dokładnie to, o co nas poprosił szef zakupów, uruchamiając kod z listingu 4.8.

```swift
UPDATE dbo.PurchaseOrderDetails
SET PurchaseOrderNumber = 6828
WHERE PurchaseOrderNumber = 6827 ;
```

Niestety, szef działu zakupów tak naprawdę chciał, abyśmy „utworzyli nowe zamówienie o numerze zamówienia 6828, przenieśli zamówione pozycje między sobą, a następnie usunęli zamówienie o numerze 6827”.

Mamy więc teraz sytuację, w której nasze dane są w niespójnym stanie. Gdybyśmy uruchomili zapytanie, takie jak zapytanie z aukcji 4.9, które zwraca szczegóły zamówień zakupu z dwóch tabel, wówczas nie zostanie zwrócone ani zamówienie zakupu 6827, ani zamówienie zakupu 6828.

Gdybyśmy utworzyli ograniczenie klucza obcego, nie byłoby to możliwe, ponieważ wymuszona zostałaby integralność referencyjna i nie bylibyśmy w stanie zaktualizować `BuyOrderNumber` do wartości, która nie istnieje w tabeli `BuyOrderHeaders`.



```sql
SELECT
      poh.PurchaseOrderNumber
    , poh.PurchaseOrderDate
    , pod.ProductID
    , pod.Quantity
FROM dbo.PurchaseOrderHeaders poh
INNER JOIN dbo.PurchaseOrderDetails pod
    ON poh.PurchaseOrderNumber = pod.PurchaseOrderNumber ;
```

Aby uniknąć tej pułapki, upewnij się, że zawsze używasz ograniczeń klucza obcego. Błąd w bazie MagicChoc możemy naprawić uruchamiając skrypt z Listingu 4.10. Przed utworzeniem ograniczenia naprawiamy wartości danych, w przeciwnym razie utworzenie ograniczenia nie powiedzie się.



```sql
UPDATE dbo.PurchaseOrderDetails
SET PurchaseOrderNumber = 6827
WHERE PurchaseOrderNumber = 6828 ;
 
ALTER TABLE dbo.PurchaseOrderDetails ADD CONSTRAINT
    FK_PurchaseOrderDetails_PurchaseOrderHeaders FOREIGN KEY (PurchaseOrderNumber) REFERENCES dbo.PurchaseOrderHeaders(PurchaseOrderNumber) ;
```

Kiedy tabele są ze sobą powiązane, dobrą praktyką jest tworzenie między nimi relacji klucza podstawowego i obcego. Nawet jeśli w aplikację wbudowana jest logika zapobiegająca niespójności danych, logika ta nie będzie w stanie poradzić sobie z błędami, które zdarzają się poza aplikacją.



### 4.4 Podsumowanie
• Aby uniknąć błędów, należy zawsze normalizować dane, zamiast próbować projektować schemat na podstawie osądu.
• Tam, gdzie to konieczne, rozważ użycie generalizacji danych do tworzenia nadtypów i podtypów, ponieważ może to ulepszyć projekt bazy danych.
• Użyj diagramu relacji encji (ERD), aby zweryfikować i udokumentować swój projekt. Ten diagram przedstawia encje z ich atrybutami, a także kluczami podstawowymi i obcymi, wraz z relacjami między nimi.
• Unikaj używania szerokich kolumn lub wielu kolumn jako klucza podstawowego, ponieważ zostanie on zreplikowany w indeksach nieklastrowych i może prowadzić do pogorszenia wydajności.
• Zawsze używaj ograniczenia klucza obcego, gdy tabele są ze sobą powiązane, aby uniknąć problemów ze spójnością danych.
