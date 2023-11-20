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

Aby lepiej zrozumieć zakres funkcji SQL Servera, wyobraźmy sobie scenariusz użycia firmy cukierniczej o nazwie MagicChoc. Posiadają oni stronę internetową, hostowaną w chmurze publicznej, która sprzedaje ich czekoladowe dobroci. Posiadają także niewielkie centrum danych w swojej fabryce, które obsługuje aplikację produkcyjną, aplikację do kontroli zapasów oraz rozwiązanie do raportowania. Ilustruje to diagram krajobrazu systemowego przedstawiony na rysunku 1.1.





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

*PORADA*

*Rekordy dziennika zawsze są zapisywane na dysku przed zatwierdzeniem transakcji, chyba że jest używana opóźniona trwałość (Delayed Durability).*

Pamięć Buforowa przechowuje strony danych, które zostały odczytane z dysku. Warto zauważyć, że zapytanie zawsze jest zaspokajane z pamięci podręcznej i nigdy bezpośrednio z danych przechowywanych na dysku. Nawet jeśli wymagane strony danych nie znajdują się w pamięci podręcznej, zostaną odczytane z dysku do pamięci podręcznej, a następnie zapytanie zostanie zaspokojone z danych w pamięci podręcznej. Powszechnym błędem jest zwracanie w zapytaniu więcej danych, niż jest to potrzebne. Jeśli to zrobisz, pamięć buforowa szybciej się zapełni niż powinna. Skutkuje to wcześniejszym zwalnianiem starszych danych z pamięci podręcznej. Z kolei może to prowadzić do słabej wydajności, ponieważ dane muszą być czytane z dysku częściej. Omówimy to bardziej szczegółowo w rozdziale 4.

#### 1.2.2 Platformy Heterogeniczne
Ważne jest, aby pamiętać, że SQL Server to już nie tylko "baza danych na systemie Windows". Zamiast tego jest obsługiwany na wielu różnych platformach. Po pierwsze, rozważmy systemy operacyjne, na których jest obsługiwany SQL Server 2022. Te systemy operacyjne są przedstawione na Rysunku 1.4.

![img](https://drek4537l1klr.cloudfront.net/carter/v-1/Figures/01__image004.png)

UWAGA

Edycja SQL Server Express i Standard może również być instalowana na systemach Windows 10 i Windows 11.

Zauważysz, że SQL Server można instalować nie tylko na systemie Windows Server Core, który jest wersją systemu Windows obsługiwaną wyłącznie za pomocą PowerShell i nie posiada interfejsu graficznego, ale również na trzech różnych wersjach systemu Linux. Oznacza to, że specjalista od baz danych może pracować nie tylko z interfejsem graficznym, ale także z PowerShell i Bash.

Warto również omówić hosty, na których można zainstalować SQL Server 2022, co przedstawiono na Rysunku 1.5.

![img](https://drek4537l1klr.cloudfront.net/carter/v-1/Figures/01__image005.png)

Officialne wsparcie dla SQL Servera na platformie VMWare istnieje od pewnego czasu. To, co może być bardziej zaskakujące, to fakt, że najnowsze wersje SQL Servera są również obsługiwane w kontenerach. Oznacza to, że niektórzy specjaliści od baz danych mogą potrzebować zapoznać się z technologiami takimi jak Docker i Kubernetes.

UWAGA

W chwili pisania tego tekstu SQL Server jest obsługiwany tylko na kontenerach Linux. SQL Server na kontenerach Windows był dostępny w wersji Beta, ale ten program został odwołany. Nic nie stoi na przeszkodzie, abyś samodzielnie tworzył własne kontenery Windows z zainstalowanym SQL Serverem, opierając się na podstawowym obrazie kontenerowym Windows Core. Należy jednak pamiętać, że ponieważ nie jest to obsługiwane, nigdy nie powinieneś tego robić w środowisku produkcyjnym.

Należy również rozważyć obsługę chmury dla SQL Servera. SQL Server jest obsługiwany na maszynach wirtualnych w ramach infrastruktury jako usługi (IaaS) w chmurze. W Azure i GCP są nazywane maszynami wirtualnymi Azure, a w AWS - instancjami EC2. Istnieją także oferty platformy jako usługi (PaaS) od tych dostawców.

W AWS dostępne są obrazy maszynowe aplikacji (AMIs) z zainstalowanym SQL Serverem. W zależności od użytego AMI, możesz zakupić SQL Servera wraz z instancją EC2 na podstawie umowy licencyjnej z dostawcą usług (SPLA). W tym modelu licencja jest uwzględniona w koszcie godzinowym instancji EC2. Alternatywnie możesz używać swojej licencji (pod warunkiem, że jest to zgodne z umową licencyjną z Microsoftem).

Istnieje również opcja korzystania z RDS, czyli oferty bazy danych jako usługi (DBaaS). Podstawowy serwer i instancję SQL Servera zarządza AWS, a ty jesteś odpowiedzialny tylko za zarządzanie bazami danych hostowanymi w obrębie tego środowiska.

Oficjalne wsparcie dla SQL Server na platformie VMWare istnieje od pewnego czasu. To, co może być bardziej zaskakujące, to fakt, że najnowsze wersje SQL Servera są również obsługiwane w kontenerach. Oznacza to, że niektórzy specjaliści od baz danych mogą potrzebować zapoznać się z technologiami takimi jak Docker i Kubernetes.

UWAGA

W chwili pisania tego tekstu SQL Server jest obsługiwany tylko na kontenerach Linux. SQL Server na kontenerach Windows był dostępny w wersji Beta, ale ten program został odwołany. Nic nie stoi na przeszkodzie, abyś samodzielnie tworzył własne kontenery Windows z zainstalowanym SQL Serverem, opierając się na podstawowym obrazie kontenerowym Windows Core. Należy jednak pamiętać, że ponieważ nie jest to obsługiwane, nigdy nie powinieneś tego robić w środowisku produkcyjnym.

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

FIZYCZNI ADMINISTRATORZY BAZ DANYCH W PORÓWANIU DO ADMINISTRATORÓW BAZ DANYCH ODPOWIEDZIALNYCH ZA STRUKTURĘ

Termin "fizyczny administrator baz danych" odnosi się do administratorów baz danych (DBA), którzy posiadają dobre umiejętności zarządzania instancjami SQL Server, ale nie są specjalistami w zakresie struktury bazy danych. Odpowiednikiem są tutaj "administratorzy baz danych odpowiedzialni za strukturę" (ang. Logical DBAs), którzy są specjalistami w zakresie tworzenia i optymalizacji baz danych, ale nie mają doświadczenia w zarządzaniu instancjami.

W moim doświadczeniu widziałem firmy, które organizują swoje zespoły baz danych w ten sposób, zwłaszcza gdy mają dostawcę usług zarządzania infrastrukturą, który buduje i obsługuje instancje SQL Server, dba o ich dostępność. Często odpowiedzialność za bazy danych przechodzi następnie na wewnętrzny zespół wsparcia aplikacji, który odpowiada za obsługę i optymalizację konstrukcji na poziomie bazy danych.

W praktyce niemal zawsze zachodzi konieczność utrzymania niektórych baz danych poza ofertą DBaaS (ang. Database as a Service). Jednym z powodów jest wsparcie dostawcy. Jeśli korzystasz z komercyjnego produktu (Commercial Off The Shelf - COTS), który ma bazę danych SQL Server, jesteś zależny od zgody dostawcy na wsparcie produktu, jeśli baza danych zostanie przeniesiona do usługi DBaaS.

Kolejnym powodem jest to, że wiele firm ma duże, złożone aplikacje z warstwą danych, czasem z setkami tysięcy linii kodu osadzonych w komponentach, takich jak SQL Server Integration Services. Często te aplikacje nie mogą być po prostu przeniesione bez żadnej transformacji czy nowoczesnej aktualizacji. Wymagają one raczej przebudowy, co oznacza znaczne inwestycje zarówno czasowe, jak i finansowe, które często są trudne do uzasadnienia.



Dlaczego ważne jest właściwe skonfigurowanie SQL Servera?

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

Pierwotną przyczyną błędów związanych z SQL Serverem jest błędne założenie, że bazy danych są proste i łatwe w obsłudze. SQL Server to popularny System Zarządzania Bazą Danych Relacyjnych (RDBMS). Posiada obszerny i złożony ekosystem składający się z wielu głównych komponentów. Głównym elementem SQL Servera jest Silnik Bazy Danych, który obejmuje komponenty takie jak Silnik Relacyjny i Silnik Składowania. SQL Server jest kompatybilny zarówno z systemami operacyjnymi Windows, jak i Linux. Można go uruchamiać na fizycznych serwerach, maszynach wirtualnych oraz kontenerach Linux. W chmurze publicznej SQL Server jest dostępny zarówno w formie maszyn wirtualnych (IaaS), jak i ofert Platform as a Service (PaaS). Ważne jest właściwe wdrożenie SQL Servera, aby uniknąć problemów, takich jak słaba wydajność, utrata danych i naruszenia bezpieczeństwa.

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

W tej sytuacji otrzymujesz zgłoszenie od zespołu aplikacyjnego odpowiedzialnego za aplikację o nazwie TimeChewer, informujące o wolno działających zapytaniach. Wspominają, że baza danych aplikacji jest hostowana na serwerze o nazwie sql-shared-app-server. Po połączeniu się z serwerem, odkrywasz instancje o nazwach SQL01 i SQL02, bez jasnych wskazówek, która instancja zawiera bazę danych aplikacji. Po zbadaniu każdej instancji znajdujesz bazy danych o nazwach DB01, DB02, DB03 i DB04.

Po skontaktowaniu się z zespołem aplikacyjnym udzielają oni informacji, że TimeChewer łączy się z DB03 na SQL01. Następnie przystępujesz do analizy jednej ze słabo działających procedur składowanych, a poniżej znajduje się definicja tej procedury:

```sql
ALTER PROCEDURE dbo.proc01
AS
BEGIN
    ;WITH t3 AS (SELECT col1, col2, col3 FROM tbl03)
    SELECT t1.col1, t2.col2, t1.col2, t1.col3, t3.col1, t3.col2
    FROM dbo.tbl01 t1
    INNER JOIN tbl02 t2 ON t1.col1 = t2.col1 AND t1.col3 < 55
    INNER JOIN t3 ON t3.col1 = t1.col1
    UNION
    SELECT col1, col2, col3, NULL, NULL, '0' FROM dbo.tbl04;
END
```

Ta procedura składowana o nazwie `proc01` używa wspólnej tabeli o nazwie `t3` i łączy dane z wielu tabel za pomocą operacji INNER JOIN. Operator UNION jest używany do połączenia wyników dwóch instrukcji SELECT. Jeśli ta procedura składowana została zidentyfikowana jako źródło problemów wydajności, można rozważyć optymalizację logiki zapytania, utworzenie indeksów na odpowiednich kolumnach lub zastosowanie innych technik optymalizacji, w zależności od konkretnej struktury bazy danych i obciążenia.

Wyobraź sobie, że właśnie próbujesz przyjrzeć się procedurze składowanej, starając się zrozumieć, co właściwie robi. W skrócie, poświęciłeś 30 minut na próby zidentyfikowania, która baza danych sprawia problemy, i teraz będziesz musiał poświęcić kolejne 15 minut na sformatowanie kodu tak, aby był czytelny, oraz zrozumienie, co właściwie robi ta bardzo prosta procedura składowana. To 45 minut marnowane, zanim w ogóle zaczniesz badać problem.

Choć to jest skrajny przykład, ilustruje, dlaczego konwencje nazewnictwa i standardy kodowania są tak istotne, oraz dlaczego nieprawidłowe postępowanie w tej dziedzinie będzie sprawiać frustrację tobie i innym, którzy próbują zrozumieć kod.

### 2.1 #1 Niewłaściwe nazwy obiektów

Choć przykład użyty w wprowadzeniu do tego rozdziału dotyczący nieopisowych nazw baz danych był dość ekstremalny, widziałem przypadki, gdzie ludzie stosowali takie standardowe nazwy instancji jak SQL01, SQL02 itp., co naprawdę sprawia kłopoty. Jednak bardziej powszechne stosowanie nieopisowych nazw występuje na poziomie kodu, i to na tym aspekcie skupimy się w tym rozdziale.

PORADA

Więcej rozmawiamy o konkretnych problemach spowodowanych nieprawidłowym nazewnictwem instancji w rozdziale 8.

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

Teraz więc wyobraź sobie, że ustaliłeś, że procedura składowana orders to pierwsza procedura składowana w tym łańcuchu. Możesz znaleźć definicję tej procedury składowanej za pomocą SSMS lub wykonując zapytanie z listingu 2.2.

**Listing 2.2 Pobierz definicję procedury składowanej**

```sql
SELECT s.definition
FROM sys.objects o 
INNER JOIN sys.sql_modules s
    ON o.object_id = s.object_id
WHERE o.name = 'sp_orders' ;
```

**Wskazówka**

Osobiście wolę generować definicję procedury składowanej z Eksploratora obiektów w SSMS, zamiast pobierać definicję z katalogowego widoku sys.sql_modules. Wynika to z tego, że pobranie z sys.sql_modules nie zachowa formatowania.

Definicję procedury składowanej sp_orders można znaleźć w listingu 2.3. Parametr @AddressID jest bezsensowny. Czy to jest adres rozliczeniowy czy adres dostawy? Parametr @Address jest zarówno bezsensowny, jak i mylący. Nazwa parametru sugeruje rzeczywisty adres, podczas gdy jest to przeznaczone do przechowywania identyfikatora adresu. Ponadto, odnosi się on do któregoś z adresów? Kolejną bezsensowną nazwą parametru jest @date. Czy to jest data zamówienia? Data dostawy? Może nawet sugerować znacznik czasowy, kiedy rekord został wstawiony do tabeli! Wewnątrz ciała procedury mamy również mylące nazwy zmiennych. Zmienna @stock ma przechowywać ilość zamówionych produktów, a @product przechowuje identyfikator produktu. Te nazwy po prostu nie są jasne, i trzeba sięgnąć do kodu, aby zrozumieć ich cel.

```sql
Listing 2.3 sp_orders procedure definition

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

**Wskazówka**

Jeśli nie jesteś zaznajomiony z tym, jak korzystać z XML w SQL Server, to nie jesteś sam. Faktycznie, niewłaściwe wykorzystanie XML to kolejny powszechny błąd, który omówimy w rozdziale 3.

Czytając definicję tej procedury składowanej, możemy już zauważyć, że bezsensowne nazwy parametrów i zmiennych sprawiają nam kłopoty. Jeśli będziemy musieli rozszerzać lub debugować funkcjonalność związana z adresami lub czasem, to będziemy musieli poświęcić czas na zrozumienie, co oznaczają dane.

Na razie jednak musimy naprawić problem z aktualizacją zapasu, a ta definicja procedury składowanej pozwoliła nam ustalić, że procedura sp_stockUpdate prawdopodobnie zawiera kod, który musimy zdebugować. Spójrzmy więc na procedurę w listingu 2.4.

Widzimy, że sp_stockUpdate to prosta procedura składowana, która aktualizuje poziom zapasów w SalesDB, a następnie za pomocą Linked Server aktualizuje poziom zapasów w systemie Inventory. Widzimy również, że przyczyną błędu jest używanie niekonsekwentnych nazw, co wprowadza programistę w błąd i powoduje przekazanie identyfikatora produktu do obliczeń aktualizujących ilość pozostałą na magazynie, jednocześnie próbując filtrować identyfikator produktu w oparciu o ilość zamówionego towaru, zamiast faktycznego identyfikatora produktu.

**Definicja procedury sp_stockUpdate**

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

**PRZYKŁAD Z RZECZYWISTEGO ŚWIATA**

Ten przykład jest dość prosty, służy tylko celom ilustracyjnym, ale możesz sobie wyobrazić, że przy skomplikowanym kodzie te wyzwania szybko zamienią się w koszmar. Przykład w tej sekcji luźno oparty jest na rzeczywistym przykładzie, z którym się spotkałem. Programista napisał bardzo złożony proces, z pięcioma warstwami procedur składowanych i wieloma punktami wejścia z zewnętrznych aplikacji. Wszystkich procedur było około 2000 linii kodu. Po jego odejściu odkryto błąd, i próbując go rozwiązać, stwierdziliśmy, że programista używał zmiennych, nazw parametrów i nazw kolumn zamiennie. Sam błąd był prosty i powinien zostać rozwiązany w ciągu kilku godzin, ale z powodu plątaniny nazw zajęło to trzy dni wysiłku, aby rozwiązać problem. Następnie zajęło kolejne dwie kolejne tygodnie, aby zaktualizować wszystkie nazwy, by były spójne, i przeprowadzić testy regresji.



**PORADA**

Pozytywnym przykładem dobrego nazewnictwa w naszym przykładzie są kolumny klucza głównego tabel. Zauważ, że tabela tbl_products ma ProductID, tabela tbl_orders ma OrderID, a tabela tbl_addresses ma AddressID. Powszechnym błędem jest używanie ID jako samodzielnej nazwy kolumny dla kolumn klucza głównego we wszystkich tabelach, co oznacza, że kolumna klucza głównego we wszystkich tabelach ma tę samą nazwę. Pewnie możesz sobie wyobrazić, jak łatwo mogłoby to stać się mylące w kontekście skomplikowanej procedury składowanej.

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

**PORADA**

W stylu mojego ulubionego prywatnego detektywa z lat 80. - "Wiem, co myślisz, i masz rację. Z pewnością powinniśmy także zmienić nazwę procedury, prawda?" Jednak na razie zostawiłem ją bez zmian, ponieważ będziemy o niej rozmawiać bardziej szczegółowo w Błędzie #2.

Proszę również zwrócić uwagę na nową definicję procedury sp_stockUpdate w listingu 2.6. Zauważ, że nazwa procedury została zmieniona na sp_updateProductStockLevel, co jest znacznie bardziej sensowną nazwą dla procedury. Dzięki tej nazwie nie mielibyśmy problemu z jej odnalezieniem na początku tego przykładu.

**Listing 2.6 Nowa definicja procedury sp_updateProductStockLevel**

```sql
CREATE PROCEDURE sp_updateProductStockLevel
    @OrderQty INT,
    @ProductID INT
AS
BEGIN
    UPDATE tbl_products
    SET StockQty = StockQty - @OrderQty
    WHERE ProductID = @ProductID ;
 
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



POWODY ISTNIENIA WIDOKÓW, KTÓRE DOKŁADNIE ODPOWIADAJĄ TABELI

Istnieją dwie przyczyny, dla których niektórzy ludzie mogą mieć widoki, które zwracają wszystkie kolumny z pojedynczej tabeli. Pierwszym jest sposób blokowania tabel, aby uniknąć przypadkowej zmiany schematu tabeli. Jeśli utworzysz widok Z WIĄZANIEM SCHEMY, nie można zmienić definicji tabeli bez wcześniejszego usunięcia (lub usunięcia WIĄZANIA SCHEMY) z widoku. Nie polecam tego podejścia. Dużo lepiej i bardziej zrozumiale jest po prostu ograniczenie dostępu, aby uniemożliwić modyfikacje schematu przy użyciu odpowiedniej strategii zabezpieczeń. Istnieje zasada projektowania zwana Zasadą Najmniejszego Zaskoczenia. Podpisuję się pod tą teorią.

Drugim powodem, dla którego możesz zobaczyć widok zwracający wszystkie kolumny z jednej tabeli, jest rygorystyczna zasada, zgodnie z którą wszystkie aplikacje muszą uzyskiwać dostęp do danych za pomocą warstwy abstrakcji. Zdecydowanie popieram zasadę leżącą u podstaw tego podejścia, którą jest utrzymanie złożoności na poziomie serwera SQL i umożliwienie programistom aplikacji prostego pobierania danych z widoków i przechowywanych procedur. Jednak gdy jest to przesadzone, dodaje się tylko dodatkowe obiekty bez żadnych korzyści. Po prostu masz więcej obiektów do utrzymania.

The script in listing 2.8 will drop the prefix from the tables and view.

##### Listing 2.8 Drop the prefix

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

The application wydaje się generować dziwny błąd. Aby spróbować zdiagnozować problem, możemy zasymulować wywołanie procedury składowanej przez aplikację za pomocą skryptu w Listingu 2.10.

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

UWAGA

Te listy mają na celu dać smak rozważaniom, które wpływają na standardy kodowania. W żaden sposób nie mają być wyczerpujące.

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

Przepraszam za nieporozumienie. Rzeczywiście, w kontekście baz danych termin "kolumna ordinalna" nie jest powszechnie używany. Możliwe jest, że autor użył tego terminu w sposób nieprecyzyjny.

W kontekście SQL Servera, klauzula ORDER BY zazwyczaj używa nazw kolumn lub wyrażeń, a nie ich pozycji ordinalnej. Wydaje się, że autor mógł pomylić się, chcąc powiedzieć, że można sortować po kolumnie poprzez podanie jej numeru w klauzuli ORDER BY.

Oto poprawiona wersja tłumaczenia:

5. Używanie numeru pozycji kolumny
SQL Server obsługuje użycie numerów pozycji kolumn w klauzuli ORDER BY. Oznacza to, że zamiast sortować według nazwy kolumny, można sortować według jej numeru pozycji w klauzuli SELECT. Na przykład, rozważ zapytanie z listingu 2.15.

Listing 2.15 Sortowanie Według Numeru Pozycji Kolumny

```sql
SELECT * 
FROM SYS.databases 
ORDER BY 54;
```

Ok, więc według której kolumny posortowaliśmy zapytanie? Odpowiedź brzmi - według kolumny log_reuse_wait_desc. Jednak jedynymi sposobami ustalenia tego faktu są albo policzenie kolumn w zbiorze wynikowym, aż dojdziesz do 54. kolumny, albo uruchomienie zapytania z listingu 2.16, które pobiera nazwę kolumny z metadanych przechowywanych w widokach katalogu.

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

PORADA

Jeśli obiektem, który otrzymujemy w wyniku zapytania, byłby obiekt użytkownika, a nie obiekt systemowy, to zapytanie również zadziałałoby, gdybyśmy dołączyli sys.objects do sys.columns.

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

MagicChoc zdecydowało, że potrzebuje nowej aplikacji obsługującej Zasoby Ludzkie. Skrypt w listingu 3.1 tworzy bazę danych HumanResources, a następnie tworzy pierwszą tabelę - dbo.employees. Zauważysz, że ta tabela została utworzona, używając typu danych NVARCHAR(MAX) dla każdej kolumny. Wygląda to dziwnie, prawda? Ale dlaczego? Wszystkie dane, które chcemy przechowywać, mogą być wprowadzane do tego rozległego typu danych. Czy to naprawdę ma znaczenie? Będziemy zgłębiać ten przykład przez cały ten rozdział.

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

