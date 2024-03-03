# Rozdział 7 - Synchronizacja aplikacji ze zdalnym źródłem danych

Powszechnym zastosowaniem magazynów Core Data/Swift Data jest użycie ich jako środka do obsługi funkcji offline dla aplikacji. Celowo unikam nazywania tego pamięcią podręczną, ponieważ sposób, w jaki Core Data jest używany do funkcji offline, jest często bardziej złożony niż typowa pamięć podręczna.
Na przykład, gdy używasz Core Data jako kopii offline zdalnego źródła danych, będziesz mógł wypełnić wiele ekranów danymi przechowywanymi lokalnie w magazynie danych. Można wysyłać zapytania do magazynu danych w zależności od potrzeb użytkownika, a użytkownik może być w stanie dokonywać modyfikacji rekordów w magazynie, które są wysyłane z powrotem do zdalnego, gdy jest to możliwe. Te możliwości sprawiają, że Core Data jest czymś więcej niż tylko pamięcią podręczną i bardziej przypomina lokalny magazyn, który jest synchronizowany ze zdalnym źródłem.
W tym rozdziale dowiesz się o synchronizacji magazynu Core Data ze zdalnym źródłem w dwóch krokach:



1. Używanie zdalnego źródła do wypełniania lokalnego magazynu 

2. Synchronizacja zmian lokalnych do zdalnego magazynu

  

Strategia synchronizacji zawsze będzie zależała od tego, jak wygląda zdalna usługa, z którą się synchronizujesz. Z tego powodu rozwiązania, które przedstawiam w tym rozdziale, są tak ogólne, jak to tylko możliwe, ale mogą nie być odpowiednie dla każdej sytuacji. 

Na każdym kroku dowiesz się, dlaczego podjąłem określoną decyzję, jakie alternatywy możesz rozważyć i kiedy mają one zastosowanie. Chcę, abyś w tym rozdziale skupił się bardziej na uzasadnieniu niż na dosłownym kodzie, ponieważ nie mogę podać przykładów dla każdego możliwego rozwiązania zdalnego, z którym możesz chcieć się zsynchronizować; jest po prostu zbyt wiele zmiennych.

Chociaż każdy back-end jest inny i być może będziesz musiał wprowadzić pewne modyfikacje do kodu w tym rozdziale, większość strategii synchronizacji ma wiele wspólnych wzorców. Bez względu na to, jak wygląda twój back-end, powinieneś być w stanie wyciągnąć kilka cennych lekcji na temat synchronizacji magazynu Core Data z tego rozdziału.
Należy pamiętać, że pakiet kodu tego rozdziału zawiera tylko projekt SwiftUI. Większość pracy w tym rozdziale jest wykonywana we frameworku z rozdziału 7, który znajduje się w obszarze roboczym dla tego rozdziału, więc nie ma dużej wartości w dostarczaniu dwóch przykładowych aplikacji dla tego rozdziału. Aby uruchomić próbkę i pobawić się importerem, upewnij się, że otworzyłeś plik .xcworkspace w pakiecie kodu.

### Używanie zdalnego źródła danych do wypełniania lokalnego sklepu

Zapełnianie lokalnego sklepu zdalnymi danymi samo w sobie nie jest najbardziej skomplikowanym zadaniem, jakie przychodzi mi do głowy. Zwykle wiąże się to z wykonaniem połączenia sieciowego w celu pobrania danych, konwersji tych danych na modele Codable przy użyciu dekodera JSON, a następnie utrwalenia danych lokalnie. Może to być w magazynie Core Data, ale może też być gdzieś indziej.
W najprostszej formie nie trzeba nawet konwertować danych na modele. Dane są po prostu zapisywane w katalogu Documents aplikacji. Gdy nadejdzie czas na wykorzystanie danych, ładujesz plik, konwertujesz dane z pliku na modele Codable i gotowe.
Prosta strategia wykorzystująca katalog dokumentów jest idealna, gdy wszystko, czego potrzebujesz, to prosta pamięć podręczna dla prostej struktury danych.
	Gdy potrzeby w zakresie przechowywania danych są bardziej złożone, takie podejście szybko staje się nieporęczne i prawdopodobnie napotkasz problemy. Wyobraźmy sobie na przykład aplikację, która wyświetla informacje o interesujących miejscach w pobliżu bieżącej lokalizacji. Aplikacja ta byłaby prawdopodobnie używana przez turystów, co może oznaczać, że dane komórkowe są bardzo drogie dla użytkownika. Aby uniknąć naliczania ogromnych rachunków za roaming, piszesz aplikację tak, aby buforowała dane pobierane z serwera.
	Gdy użytkownik jest w hotelu, może korzystać z danych, wyszukiwać interesujące rzeczy do zrobienia, a użytkownik może oznaczyć je jako ulubione, aby później łatwo uzyskać dostęp do ulubionych lokalizacji. Ponieważ chcesz, aby doświadczenie użytkownika było wspaniałe, przechowujesz wszystkie ulubione miejsca użytkownika lokalnie wraz ze wszystkimi załadowanymi punktami zainteresowania. Oznacza to, że można zgromadzić całkiem sporą liczbę interesujących miejsc.
	W zależności od struktury zaplecza, może być konieczne wielokrotne wywoływanie serwera w celu pobrania każdego punktu zainteresowania wyświetlanego w interfejsie użytkownika. Jest to szczególnie ważne, jeśli użytkownik wyszukuje na przykład interesujące miejsca w kilku miastach.
Jestem pewien, że możesz sobie wyobrazić, że taka aplikacja będzie pobierać sporo interesujących miejsc podczas jednej sesji. Kiedy zapisywałbyś każdą odpowiedź API do katalogu documents jako Data skończyłbyś z nieporęczną pamięcią podręczną, która jest trudna w użyciu i nie może być łatwo przeszukiwana przez użytkownika.
	Na przykład, aby znaleźć wszystkie ulubione pliki użytkownika, musisz albo zduplikować dane między plikiem przechowującym ulubione, albo możesz przechowywać listę identyfikatorów, załadować wszystkie utrwalone pliki i przefiltrować je pod kątem identyfikatorów, które użytkownik oznaczył jako ulubione.
	Co gorsza, może się zdarzyć, że wiele utrwalonych odpowiedzi będzie zawierać zduplikowane punkty zainteresowania. A w dużym mieście możemy mówić o setkach interesujących miejsc, które możemy chcieć pokazać na mapie.
	W przypadku scenariusza takiego jak ten, który właśnie opisałem, sensowne jest przechowywanie każdego punktu zainteresowania jako encji Core Data. Pozwoli nam to na odpytywanie bazy danych o wszystkie istotne punkty zainteresowania i możemy łatwo śledzić, które punkty zainteresowania zostały oznaczone jako ulubione przez użytkownika. Znacznie łatwiej jest również uniknąć przechowywania zduplikowanych danych przy użyciu Core Data.

### Używanie odpowiedzi JSON do wypełniania magazynu Core Data

​	Zazwyczaj podczas pobierania danych z serwera dane te są wysyłane w formacie JSON. Istnieją inne formaty, takie jak XML, ale są one bardzo rzadko używane, więc na razie skupię się tylko na danych JSON.
Pierwszym krokiem do zapełnienia magazynu Core Data danymi JSON jest przemyślenie projektu, który zapewni solidne API dla importera danych. Zwykle lubię projektować moje API importera w sposób, który obsługuje dwa różne podejścia:

1. Podejście pasywne; pobieranie danych z sieci w normalny sposób i przechowywanie pobranych obiektów w Core Data.
2. Podejście aktywne; uruchamianie importera w celu pobrania danych i przechowywania ich w CoreData.



​	Scenariusz, który nakreśliłem we wstępie do tej sekcji, najlepiej pasuje do pierwszego podejścia. Będziemy chcieli pobrać interesujące nas punkty, pokazać je użytkownikowi i zapisać je w Core Data do późniejszego wykorzystania. Aktywne podejście sprawdziłoby się najlepiej w przypadku aplikacji, która ma mały lub średni zestaw danych, który można zsynchronizować z urządzeniem, aby ograniczyć ilość ruchu sieciowego potrzebnego do działania aplikacji. Nadaje się również dobrze do sytuacji, w których dokładnie wiesz, które dane będą potrzebne, a back-end obsługuje pobieranie tych danych za jednym razem.



​	Niezależnie od podejścia, które najlepiej pasuje do danego przypadku użycia, logika importu nie ulega zmianie. Przynajmniej nie powinna.
Ponieważ oczekuje się, że oba podejścia będą pobierać dane i utrwalać je w Core Data, sensowne jest napisanie importera, który wykonuje następujące czynności:

- Akceptuje dane jako dane wejściowe
- Dekoduje dane do odpowiedniego typu modelu 
- Utrwalanie modelu w CoreData

​	Dzięki takiemu podejściu będziesz mógł podłączyć importera w dowolnym miejscu, ponieważ pobiera on dane pobrane z sieci. Aby zdekodować dane na prawidłowy typ modelu, importer będzie musiał użyć generics, a do utrwalenia w Core Data będziemy potrzebować obiektów zarządzanych.
Możemy to wszystko uchwycić w poniższym szkielecie klasy `DataImporter`:



```swift
public class DataImporter {
  private let context: NSManagedObjectContext
  init(context: NSManagedObjectContext) {
    self.context = context
  }
  public func importData<T: NSManagedObject & Decodable>(_ data:
             Data, as model: T.Type) {
                                                                                 		}
}
```

Jak można się spodziewać, importer danych jest inicjowany z kontekstem obiektu zarządzanego, który powinien być użyty do uruchomienia importu. Importer danych ma jedną metodę o nazwie `import- Data(_:as:)`, której możemy użyć do zaimportowania danych i przekonwertowania ich na instancje obiektów zarządzanych, o ile obiekt zarządzany jest zgodny z protokołem Decodable. Pozwoli nam to użyć `JSONDecodera` do dekodowania danych bezpośrednio na obiekty zarządzane bez żadnej dodatkowej pracy.

Całkiem zgrabne, prawda?

Zanim zdefiniujemy NSManagedObject zgodny z Decodable, możemy napisać treść metody dla `importData(_:as:)`:

```swift
    public func importData<T: NSManagedObject & Decodable>(_ data: Data,  as model: T.Type) { 
        context.perform { [unowned self] in
            let decoder = JSONDecoder() decoder.userInfo[.managedObjectContext] = self.context
            do {
                _ = try decoder.decode(model, from: data) try self.context.save()
            } catch {
                if self.context.hasChanges {
                self.context.rollback() }
                print("Failed to insert models")
                print(error)
            }
        }
    }
```

Ten kod prawdopodobnie zawiera co najmniej jedną linię, której wcześniej nie widziałeś:

```swift
decoder.userInfo[.managedObjectContext] = self.context
```



Za chwilę wyjaśnię, co robi ta linia. Spójrzmy najpierw na `importData(_:as:)` z wysokiego poziomu.
poziomu.

Ponieważ celem `importData(_:as:)` jest interakcja z obiektami zarządzanymi i właściwością kontekstu importera, używam metody perform do uruchomienia mojego kodu na kolejce kontekstu. W ciele zamknięcia mojego perform tworzę instancję JSONDecoder i kojarzę zarządzany kontekst obiektu z jego słownikiem UserInfo. Następnie dekoduję dane do typu modelu T.Type, który został przekazany do `importData(_:as:)`. Jeśli to się powiedzie, wywołuję save() na kontekście i import jest zakończony.

Jeśli dekodowanie lub zapisanie nie powiedzie się, sprawdzam, czy kontekst obiektu zarządzanego ma jakiekolwiek oczekujące zmiany przy użyciu jego właściwości `hasChanges`. Jeśli istnieją niezapisane zmiany, są one wycofywane. W ten sposób upewniam się, że wszystkie obiekty, których nie udało się zapisać z powodu błędów walidacji, zostaną usunięte z mojego kontekstu, aby nie uniemożliwiały pomyślnego zapisania kolejnych obiektów.
Ta metoda jest bardzo prosta, prawda? Prawdopodobnie o wiele prostsza niż się spodziewałeś.
I chociaż nie zamierzamy uczynić `importData(_:as:)` bardziej złożonym niż jest teraz, jeszcze nie skończyliśmy.
Jak wiadomo, obiekty zarządzane są powiązane z kontekstem obiektu zarządzanego podczas ich inicjalizacji. Obecnie nic nie ustanawia tej relacji, ponieważ metoda `init(context:)` nie jest wywoływana, gdy próbujemy zdekodować dane za pomocą JSONDecoder. Zamiast tego wywoływana jest metoda init(from:). Będziemy musieli zaimplementować własną metodę `init(from:)` na wszystkich zarządzanych obiektach, które chcemy dostosować do `Decodable`.
W pozostałej części tego rozdziału będę używał następujących danych JSON jako reprezentacji dla moich danych:

```json
[
{
  "address":"Address 0",
  "city":"Amsterdam",
  "information":"Some information about POI 0",
  "longitude":8.9876003265380859,
  "country":"The Netherlands",
  "latitude":10.12339973449707,
  "identifier":"1975DB51-5A45-4FD6-AC54-46CA6042C710",
  "name":"POI 0"
},
// etc...
]
```

Rozważmy następującą podklasę obiektu zarządzanego, która reprezentuje bardzo podstawowy przykład możliwego punktu zainteresowania, z którego może korzystać aplikacja:

```swift
    class PointOfInterest: NSManagedObject {
        @NSManaged var address: String
        @NSManaged var city: String
        @NSManaged var country: String 
        @NSManaged var identifier: UUID
        @NSManaged var information: String
        @NSManaged var latitude: Float
        @NSManaged var longitude: Float
        @NSManaged var name: String
    }
```

Zauważ, że właściwości tego modelu mają typy, których możesz się nie spodziewać. Na przykład żadna z właściwości nie jest opcjonalna w mojej podklasie.
Powodem tego jest fakt, że zdefiniowałem te podklasy obiektów zarządzanych ręcznie, zamiast pozwolić Xcode wygenerować je za mnie. Jest to doskonały przykład tego, jak Core Data wykonuje pomost między typami Swift a wewnętrznie używanymi typami Objective-C.
Zdefiniowałem mój model ręcznie, aby można go było zdekodować. Aby logika dekodowania naszego obiektu zarządzanego działała poprawnie, musimy dodać inicjalizator `init(decoder:)` do podklasy obiektu zarządzanego. Niestety, nie można tego zrobić w rozszerzeniu. Można użyć typu generowania kodu Category/Extension dla obiektów zarządzanych, ponieważ daje to również kontrolę nad inicjalizatorem. Pomyślałem jednak, że jest to dobry sposób na zaprezentowanie ręcznego definiowania obiektu zarządzanego od podstaw.
Aby obiekt zarządzany, który pokazałem wcześniej, był zgodny z Decodable, jego deklaracja musi zostać zaktualizowana do klasy `PointOfInterest`: `NSManagedObject`, `Decodable` i trzeba zaimplementować metodę `init(decoder:)`.
Ponieważ implementujemy niestandardowy inicjalizator, Swift nie wygeneruje dla nas żadnego kodu Decodable, więc oprócz init musisz zdefiniować wyliczenie CodingKeys.
Zobaczmy, jak to wygląda dla `PointOfInterest`:



```swift
public class PointOfInterest: NSManagedObject, Decodable { 
    enum CodingKeys: CodingKey {
    case address, city, country, identifier,
         information, latitude, longitude, name
}

    @NSManaged public var address: String
    @NSManaged public var city: String
    @NSManaged public var country: String
    @NSManaged public var identifier: UUID
    @NSManaged public var information: String
    @NSManaged public var latitude: Float
    @NSManaged public var longitude: Float
    @NSManaged public var name: String

    public required convenience init(from decoder: Decoder) throws {
        guard let context = decoder.userInfo[.managedObjectContext] as? NSManagedObjectContext else {
            fatalError("Attempt to decode managed object with misconfigured decoder.")
        }
        self.init(context: context)

        let container = try decoder.container(keyedBy: CodingKeys.self)
        self.address = try container.decode(String.self, forKey: .address)
        self.city = try container.decode(String.self, forKey: .city) self.country = try container.decode(String.self, forKey:.country)
        self.identifier = try container.decode(UUID.self, forKey: .identifier)
        self.information = try container.decode(String.self, forKey:.information)
        self.latitude = try container.decode(Float.self, forKey:.latitude)
        self.longitude = try container.decode(Float.self, forKey: .longitude)

        self.name = try container.decode(String.self, forKey: .name)
    }
}
```

Ta niestandardowa logika dekodowania jest dość standardową implementacją niestandardowego `init(decoder:)`. Najbardziej interesująca część tego kodu jest następująca:

```swift
guard let context = decoder.userInfo[.managedObjectContext] as?  NSManagedObjectContext else {
	fatalError("Attempt to decode managed object with misconfigured decoder.")
}
self.init(context: context)
```

Ten kod wyodrębnia kontekst obiektu zarządzanego z userInfo dekodera. Pamiętasz, że wcześniej dodaliśmy kontekst obiektu zarządzanego do dekodera JSON? Oto dlaczego.
Ponieważ możemy wyodrębnić kontekst obiektu zarządzanego z dekodera, możemy użyć tego kontekstu do wywołania metody `init(context:)` naszego obiektu zarządzanego. Następnie możemy rozpocząć ciężką pracę dekodowania wszystkich właściwości, które musimy przypisać do naszego obiektu zarządzanego.
Wymaga to sporo ręcznej pracy, ale uważam, że takie podejście do importowania danych jest dość eleganckie i elastyczne.
Niestety, ma ono również pewne wady.
Podejście to tworzy instancję obiektu zarządzanego dla każdego elementu, który chcemy zaimportować i kojarzy tę instancję z kontekstem obiektu zarządzanego. Oznacza to, że kontekst będzie przechowywał jak najwięcej tych obiektów w pamięci, co może spowodować znaczny narzut podczas importowania dużych ilości danych.
Podczas pracy nad tym rozdziałem użyłem przykładowego zestawu danych, który zawiera 10 000 punktów zainteresowania. To dużo i dobrze pokazuje wpływ naszego obecnego importera danych na pamięć:

A gdybym powiedział, że możemy zmniejszyć zużycie pamięci o około 10 MB?
Zobaczmy jak to zrobić.



### Ulepszanie importu danych za pomocą wstawiania wsadowego

W rozdziale 6 - Udostępnianie magazynu Core Data aplikacjom i rozszerzeniom, zobaczyłeś przykład żądania wstawiania wsadowego. Wspomniałem już, że wstawianie wsadowe może pomóc w utrzymaniu niewielkiej ilości pamięci podczas importowania ogromnych ilości danych. Nie powinno być zaskoczeniem, że wstawianie wsadowe może pomóc zmniejszyć zużycie pamięci przez nasz importer danych.
Niestety, podejście Decodable, którego używaliśmy wcześniej, nie jest do końca kompatybilne z dodawanie wsadowymi. Nie zalecałbym używania Decodable z zarządzanymi obiektami, ponieważ jest to wygodne i łatwe w użyciu, ale gdy liczy się wydajność i masz do czynienia z większymi importami, warto pójść nieco inną drogą.
Na szczęście przejście na import wsadowy nie jest zbyt skomplikowane. W idealnym świecie moglibyśmy napisać nową metodę importu, która pobiera dane, konwertuje je na tablicę słowników i używa ich w dodawaniu wsadowym.
Oto jak wyglądałby kod do tego celu:

```swift
    func optimizedImportData(_ data: Data, as entity: NSEntityDescription) { context.perform { [unowned self] in
        do {
            let entries = try JSONSerialization.jsonObject(with: data, options: []) as! [[String: Any]]
            let batchImport = NSBatchInsertRequest(entity: entity, objects: entries)
            try self.context.execute(batchImport)
        } catch {
            print("Failed to batch import") print(error)
        } }
    }
```

Jeśli dane JSON zawierają wartości, które można automatycznie przekonwertować na odpowiadające im wartości w podklasach obiektów zarządzanych, podejście to będzie działać jak urok.
Zauważ, że nie używam tutaj `JSONDecoder`. Powód jest prosty. Chcę dekodować do `[[String: Any]]`, więc mogę mieć słownik z kluczami String i wszystkim jako wartością. Mój słownik może zawierać inne słowniki, liczby całkowite, ciągi, pływaki i wszystko inne, co jest ważne w JSON. Niestety nie można tego przedstawić w sposób wykorzystujący Decodable.
Niestety, JSON, który pokazałem wcześniej, zawiera pole identyfikatora, które przechowuje String. Nasz model ma właściwość identyfikatora typu UUID. Podczas pracy z Decodable wartość String może zostać zdekodowana do UUID. Niestety, nie jest to możliwe w przypadku żądania wsadowego. Oznacza to, że będziemy musieli wykonać trochę ręcznej pracy przy użyciu wersji żądania wstawiania wsadowego, którą pokazałem wcześniej. To podejście wykorzystuje zamknięcie do wypełnienia zmiennego słownika dla każdego elementu, który powinien zostać zaimportowany:

```swift
    func importPointsOfInterest(_ data: Data) { context.perform { [unowned self] in
        do {
            var entries = try JSONSerialization.jsonObject(with: data,
                                                           options: []) as! [[String: Any]]
            let batchImport = NSBatchInsertRequest(entity: PointOfInterest.entity(),
                                                   dictionaryHandler: { dictionary in
                let entry = entries.removeFirst()
                dictionary["address"] = entry["address"] as! String
                dictionary["city"] = entry["city"] as! String
                dictionary["country"] = entry["country"] as! String
                dictionary["information"] = entry["information"] as! String
                dictionary["latitude"] = entry["latitude"] as! Float
                dictionary["longitude"] = entry["longitude"] as! Float
                dictionary["name"] = entry["name"] as! String
                let uuid = UUID(uuidString: entry["identifier"] as! String)!
                dictionary["identifier"] = uuid
                return entries.isEmpty
            })

            try self.context.execute(batchImport) } catch {
                print("Failed to batch import")
                print(error)
            }
    }
    }
```

Ta metoda wyodrębnia tablicę słowników z otrzymanych danych. Następnie tworzę instancję NSBatchInsertRequest.
W zamknięciu usuwam pierwszy słownik z mojej tablicy słowników. Spowoduje to modyfikację oryginalnej właściwości `entries`, usuwając pierwszy element tablicy i zwracając go.
Następnie ręcznie kopiuję i rzutuję każdą wartość słownika. Zauważ, że nie muszę tego robić. Jest to jednak dobry sposób na spowodowanie awarii aplikacji, jeśli otrzymamy nieprawidłowe dane, co na razie jest całkowicie w porządku. W środowisku produkcyjnym możesz chcieć zawieść z większą gracją i gdzieś zarejestrować błąd.
Należy pamiętać, że prawidłowe wyodrębnienie właściwości identyfikatora wymaga dodatkowej pracy:

```swift
let uuid = UUID(uuidString: entry["identifier"] as! String)! 
dictionary["identifier"] = uuid
```

Ponieważ String nie może być automatycznie przekonwertowany na UUID, należy to zrobić ręcznie. Te dwie linie są głównym powodem, dla którego nie mogliśmy użyć wersji `NSBatchInsertRequest`, która pobiera tablicę słowników.
Import wsadowy kończy się, gdy wszystkie wpisy w tablicy wpisów zostaną przetworzone. Z tego powodu zwracam entries.isEmpty z zamknięcia, ponieważ żądanie wstawiania będzie kontynuować wywoływanie tego zamknięcia, dopóki nie zwrócę wartości true.
Gdy import dla 10 000 punktów zainteresowania działa przy użyciu żądania wstawiania wsadowego, użycie pamięci aplikacji wygląda następująco:

Udało nam się zmniejszyć zużycie pamięci o prawie 10 MB, używając importu wsadowego opartego na słownikach zamiast importu, który ładuje wszystkie zarządzane obiekty w pamięci. Chociaż przykład 10 000 rekordów do zaimportowania jest nieco ekstremalny, ta poprawa jest ogromna i pokazuje, jak można dostroić aplikację Core Data, jeśli wiesz, jakie narzędzia są dla Ciebie dostępne. W rozdziale **10 - Debugowanie i profilowanie implementacji Core Data** pokażę ci więcej narzędzi, których możesz użyć do ulepszenia swoich aplikacji.
Chociaż to niesamowite, że możemy tak bardzo poprawić wykorzystanie pamięci przez importera, istnieje jedna poważna wada wstawek wsadowych, o której wspomniałem już w rozdziale 6 - Udostępnianie magazynu Core Data aplikacjom i rozszerzeniom. Możemy używać wstawiania wsadowego tylko dla obiektów, które nie mają relacji.
W prawdziwym życiu PointOfInterest najprawdopodobniej ma jedną lub więcej relacji, więc nie będzie można zaimportować ich jako JSON za pomocą wstawiania wsadowego. Podejście Decodable z poprzedniej sekcji działa dobrze w tym celu, mimo że jest mniej wydajne.
Odkryłem, że wywołanie funkcji reset() w kontekście obiektu zarządzanego importu po jego zapisaniu pomaga zmniejszyć zużycie pamięci podczas importowania danych przy użyciu metody Decodable, ponieważ spowoduje to wyczyszczenie pamięci podręcznej kontekstu obiektu zarządzanego i sprawi, że kontekst obiektu zarządzanego "zapomni" o wszystkich pobranych obiektach zarządzanych. W moich testach spowodowało to obniżenie zużycia pamięci po imporcie do poziomu podobnego do wstawiania wsadowego, jednak szczytowe zużycie pamięci było nadal wyższe.

### Wykonywanie aktualizacji przyrostowych na podstawie danych zdalnych

Importowanie danych ze zdalnego źródła to świetny sposób na zapewnienie użytkownikom środowiska offline. Nierzadko jednak dane są dynamiczne. Niektóre dane zmieniają się w bardzo regularnych odstępach czasu. Na przykład raz dziennie. Pozwala to na wdrożenie bardzo przewidywalnej strategii synchronizacji w aplikacji.
W innych przypadkach częstotliwość zmian danych zdalnych jest nieregularna, a lokalna kopia powinna być aktualizowana równie często. Na przykład możesz chcieć aktualizować dane lokalne za każdym razem, gdy uruchamiasz aplikację.
Niezależnie od interwału, w jakim zmieniają się dane i sposobu, w jaki back-end informuje aplikację o najbardziej aktualnym stanie rzeczy, będziesz chciał przetwarzać przychodzące zmiany tak wydajnie, jak to tylko możliwe.
Najprostszym sposobem aktualizacji lokalnej kopii Core Data byłoby usunięcie całego lokalnego magazynu, a następnie ponowne wstawienie wszystkich obiektów otrzymanych z serwera. I choć jest to najprostszy sposób, jest to również najgorsza strategia. Aby odbudować cały lokalny magazyn, należałoby zażądać wszystkich danych z serwera. Może to oznaczać, że przesyłasz ogromne ilości danych, co jest powolne i może kosztować użytkownika dużo pieniędzy, jeśli nie ma on nieograniczonego planu transmisji danych.
O wiele częściej zdarza się, że serwer wysyła przyrostową aktualizację danych, którymi jesteś zainteresowany. Na przykład pracowałem z back-endami, które zawierały znacznik czasu lub token w swoich odpowiedziach w celu synchronizacji danych. Gdy chcesz pobrać nowe dane, po prostu przekazujesz znacznik czasu lub token do serwera jako parametr zapytania w adresie URL, a serwer dokładnie wie, które dane Cię interesują. Następnie serwer konstruuje odpowiedź, która zawiera tylko nowe i zmodyfikowane rekordy wraz z nowym znacznikiem czasu lub tokenem, którego można użyć następnym razem, gdy chcesz zsynchronizować się z serwerem.
Aktualizacje przyrostowe są fantastyczne, ale wiążą się również z interesującym wyzwaniem podczas importowania danych do magazynu Core Data.
Mówiąc konkretnie, chcemy z wdziękiem radzić sobie z zapobieganiem duplikacji danych w magazynie Core Data.

1. Wiesz już, że Core Data nie jest bazą danych SQLite. Oznacza to, że nie używasz kluczy podstawowych, kluczy obcych ani unikalnych ograniczeń w sposób, w jaki można ich używać w bazie danych SQLite. Podczas wstawiania danych do bazy danych SQLite można wykonać tak zwany UPSERT, aby uniknąć duplikacji danych. Upsert w SQLite działa w następujący sposób:

  - Próba wykonania zapytania INSERT w celu wstawienia nowych danych.
  - Jeśli to się nie powiedzie z powodu błędu unikalności, zaktualizuj istniejący rekord danymi z zapytania
    INSERT.

  W tym przypadku błąd unikalności oznacza, że wstawiony obiekt zawiera jedno lub więcej pól, które są oznaczone jako unikalne w tabeli. Oznacza to, że każdy rekord w tabeli musi mieć unikalną wartość dla tego konkretnego pola. Dobrym tego przykładem jest pole id lub identyfikator, które służy do unikalnej identyfikacji rekordu w tabeli.
  Chociaż Core Data nie jest opakowaniem SQLite i nie powinno być traktowane jako takie, czasami chcemy polegać na niektórych mechanizmach SQLite. Możliwość wykonania upsert jest jednym z tych mechanizmów, które będziemy chcieli wykorzystać w naszej bazie danych.
  Istnieją dwa podejścia, które możemy zastosować, aby wykonać upsert w Core Data:

  1. Wykonać całą pracę ręcznie
  2. Niech CoreData zajmie się upsertem

  W większości przypadków najlepiej jest pozwolić Core Data obsługiwać upsert. Oszczędza to kłopotów i jest znacznie bardziej wydajne niż ręczne wykonywanie pracy.

  Wyjaśnię więcej po tym, jak zademonstruję, jak skonfigurować unikalne ograniczenie i zasady scalania. Najpierw dokonamy niewielkiej modyfikacji modelu PointOfInterest, abyśmy mieli nieco bardziej złożony model do pracy. Dodałem następującą właściwość do mojego modelu:

  ```swift
  @NSManaged var category: PoiCategory
  ```

  Ta właściwość reprezentuje nieopcjonalną relację do encji PoiCategory. Zanim to pokażę, implementacja init(decoder:) musi zostać zaktualizowana o następującą linię:

  ```swift
  self.category = try container.decode(PoiCategory.self, forKey:  .category)
  ```

  

Na koniec należy zdefiniować model PoiCategory:

```swift
    class PoiCategory: NSManagedObject, Decodable {
        enum CodingKeys: CodingKey {
            case name }
        
        @NSManaged var name: String
        @NSManaged var pointsOfInterest: Set<PointOfInterest>
        
        required convenience init(from decoder: Decoder) throws {
            guard let context = decoder.userInfo[.managedObjectContext] as? NSManagedObjectContext else {
                
            }
            fatalError("Attempt to decode managed object with misconfigured  decoder.")
        }
        self.init(context: context)
        let container = try decoder.container(keyedBy: CodingKeys.self)
        self.name = try container.decode(String.self, forKey: .name) self.pointsOfInterest = []
    }
```





Zauważ, że używam `Set<PointOfInterest>` zamiast `NSSet` dla mojej relacji `pointsOfInterest`. Ponieważ definiuję moje modele ręcznie, mogę użyć typu Swift, który jest automatycznie łączony z Objective-C.
Jeśli podążasz za mną, nie zapomnij zaktualizować pliku modelu danych. Jestem pewien, że poradzisz sobie z tym samodzielnie. Właściwość name w PoiCategory powinna być oznaczona jako nieopcjonalna.

JSON, którego od teraz będę używał w logice importu, będzie wyglądał następująco:

```swift
[
{
  "address":"Address 0",
  "city":"Amsterdam",
  "information":"Some information about POI 0",
  "longitude":8.9876003265380859,
  "country":"The Netherlands",
  "latitude":10.12339973449707,
  "identifier":"1975DB51-5A45-4FD6-AC54-46CA6042C710",
  "name":"POI 0",
  "category": {
    "name": "Category name"
  }
},
// etc...
]
```

Są dwie rzeczy, które chcę zapewnić w moim magazynie danych.

1. Chcę się upewnić, że przechowywany jest tylko jeden punkt zainteresowania na unikalny identyfikator.  
2. Chcę się upewnić, że dla każdej nazwy przechowywana jest tylko jedna kategoria.

Najprostszym sposobem na skonfigurowanie tego jest zdefiniowanie nazwy kategorii i identyfikatora punktu zainteresowania jako unikalnych właściwości w edytorze modelu.
Można to zrobić, wybierając jednostkę, do której chcesz dodać unikalne ograniczenie w edytorze modelu i dodając właściwość, którą chcesz uczynić unikalną, do sekcji `Constraints`, jak pokazano na poniższym obrazku:



Po dodaniu unikalnego ograniczenia dla identyfikatora `PointOfInterest`, upewnij się, że dodałeś również ograniczenie dla nazwy `PoiCategory`.
Uwaga: Jeśli postępujesz zgodnie z krokami opisanymi w rozdziale, aplikacja ulegnie awarii, jeśli spróbujesz ją uruchomić po dodaniu `PoiCategory` i skonfigurowaniu unikalnych ograniczeń. Dzieje się tak, ponieważ magazyn danych musi migrować ze starego modelu danych do nowego modelu, ale nie może tego zrobić automatycznie, ponieważ dodano nieopcjonalną relację do PointOfInterest. W **rozdziale 9 - Aktualizacja modelu danych i wykonywanie migracji** dowiesz się więcej na temat migracji bazy danych i wersjonowania modelu. Na razie możesz usunąć aplikację i zainstalować ją ponownie, aby pozbyć się starego magazynu danych i utworzyć nowy. Nie jest to eleganckie rozwiązanie, ale podczas rozwoju jest to często najłatwiejsze rozwiązanie, ponieważ model danych może się zmienić kilka razy przed wysłaniem aplikacji lub aktualizacją dla użytkowników, którzy oczekują płynnej migracji ze starego modelu danych aplikacji do nowego modelu danych.
Jeśli uruchomisz importera z tymi ograniczeniami, szybko przekonasz się, że Core

Dane zaczynają zawodzić przy wywołaniach save(). Powodem tego jest importowanie wielu obiektów kategorii o tej samej nazwie. Błąd, który zobaczysz, powinien wyglądać jak poniżej:

```swift
Error Domain=NSCocoaErrorDomain Code=133021 "(null)" UserInfo={NSExceptionOmitCallstacks=true, conflictList=(
    "NSConstraintConflict (0x600003134080) for constraint (\n name\n): database: (null), conflictedObjects: (\n\"0x600002435be0
... etc ...
```

Oznacza to, że funkcja save() nie powiodła się z powodu błędu ograniczeń, a Core Data nie wiedziała, jak automatycznie rozwiązać ten konflikt. Oprócz ręcznego rozwiązania tego konfliktu, możemy wybrać jeden z następujących algorytmów automatycznego rozwiązywania konfliktów:

- NSMergePolicy.mergeByPropertyStoreTrump: Odrzuca zmiany w pamięci dla obiektu będącego w konflikcie na korzyść wersji w magazynie trwałym.
- NSMergePolicy.mergeByPropertyObjectTrump: Nadpisuje obiekt w magazynie trwałym wersją, która jest w pamięci (w wielu przypadkach będzie działać tak samo jak nadpisywanie).
- NSMergePolicy.overwrite: Nadpisuje wpis w magazynie trwałym wersją w pamięci.
- NSMergePolicy.rollback: Odrzuca wszystkie zmiany w pamięci na rzecz obiektów magazynu.
- NSMergePolicy.error: Jest to domyślna polityka, która rzuca błąd, jeśli wystąpi konflikt scalania.

W naszym przypadku chcemy zaktualizować lokalnie przechowywaną kopię rekordu o nowo pobrane dane. Oznacza to, że szukamy polityki scalania `mergeByPropertyObjectTrump`. Tak się składa, że jest to polityka scalania zalecana przez Apple, gdy chcesz wykonać upsert.
Zasady scalania są konfigurowane na poziomie kontekstu zarządzanego obiektu. Oznacza to, że powinieneś ustawić właściwość mergePolicy na zarządzanym kontekście obiektu, którego planujesz użyć do importu. Przypisanie mergePolicy można wykonać w następujący sposób:

```swift
persistentContainer.viewContext.mergePolicy =  NSMergePolicy.mergeByPropertyObjectTrump
```

W tym przypadku ustawiam zasady scalania w moim `viewContext`, ponieważ tam planuję uruchomić import. Używam polityki scalania mergeByPropertyObjectTrump, więc wszelkie zdalne zmiany są używane do nadpisywania mojego lokalnego magazynu. Uruchamianie importu działa teraz pięknie, nie pojawiają się żadne błędy, a gdy jakiekolwiek dane zostały zmienione na serwerze, zmiany te nadpisują istniejące dane w sklepie bazowym.

​	Zasady scalania są bardzo potężnym narzędziem dla każdego programisty pracującego z Core Data, zwłaszcza jeśli używasz Core Data do tworzenia magazynu danych offline, który pomaga zapewnić użytkownikom doskonałe wrażenia offline. Gorąco polecam eksperymentowanie z przykładowym projektem dla tego rozdziału w pakiecie kodu, aby zobaczyć, jak zachowują się różne zasady scalania. Byłoby jeszcze lepiej, gdybyś miał własną aplikację, w której możesz eksperymentować z zasadami scalania, ponieważ pozwoliłoby to od razu ulepszyć aplikację.

​	Jeśli dostarczone zasady scalania nie odpowiadają Twoim potrzebom, możesz utworzyć podklasę `NSMergePolicy` i napisać niestandardową logikę, aby poradzić sobie z konfliktami scalania. W przypadku moich projektów nie znalazłem jeszcze potrzeby stosowania niestandardowych zasad scalania i nie uważałbym pisania niestandardowych zasad scalania za coś, co prawdopodobnie skończysz robić. Chciałem jednak upewnić się, że jesteś świadomy możliwości podklasy `NSMergePolicy` i zaimplementowania niestandardowego rozwiązywania konfliktów, abyś wiedział, że jeśli zajdzie taka potrzeba, istnieje narzędzie, które możesz chcieć wykorzystać.

## Synchronizacja lokalnych zmian ze zdalnym magazynem

​	Pobieranie danych ze zdalnego zasobu i przechowywanie ich lokalnie w celu zapewnienia doświadczenia offline może zwiększyć zaangażowanie aplikacji, a co ważniejsze, może pomóc użytkownikom zaoszczędzić cenne dane, ponieważ wystarczy zażądać danych tylko raz. Niektóre aplikacje pozwalają jednak użytkownikom generować i modyfikować dane lokalnie na ich urządzeniu. Modyfikacje te powinny ostatecznie zostać zsynchronizowane z powrotem do `BackEndu`, aby mogły zostać przetworzone na serwerze i udostępnione na innych urządzeniach użytkownika. Istnieje wiele innych powodów, dla których aplikacja potrzebuje synchronizacji i jeśli czytasz tę sekcję, jestem pewien, że masz pomysł na dane, które chciałbyś zsynchronizować między swoją aplikacją a serwerem.

​	Wdrożenie takiej dwukierunkowej synchronizacji może wymagać sporo wysiłku. Zwłaszcza jeśli chcesz umożliwić użytkownikowi wprowadzanie zmian, gdy jest offline, i przetwarzanie wszystkich zmian po ponownym nawiązaniu połączenia z Internetem.
​	W tej sekcji przedstawię strategię synchronizacji, która może być przydatna. Podobnie jak w poprzedniej sekcji, nie mogę zdecydować, jak będzie lub powinien wyglądać twój back-end. Z tego powodu przedstawię przykład, który jest tak ogólny i elastyczny, jak to tylko możliwe, umożliwiając dostosowanie go w razie potrzeby, aby można go było zintegrować z aplikacją.

### Wybór strategii synchronizacji

​	Najważniejszą częścią każdej dwukierunkowej strategii synchronizacji jest określenie sposobu wysyłania aktualizacji do zaplecza. Czy zmiany będą synchronizowane z serwerem w momencie ich wystąpienia? Czy będziesz zbierać zmiany i wysyłać je na serwer dopiero po osiągnięciu określonej liczby zmian? A może dobrym pomysłem jest zbieranie zmian przez określony czas i wysyłanie wszystkich zebranych zmian na serwer w określonych odstępach czasu? A co zrobić, gdy nie można wysłać lokalnych danych na serwer? Na przykład, jeśli użytkownik nie ma połączenia z Internetem.
Niezależnie od przyjętej strategii, wszystkie one mają ze sobą coś wspólnego.
Będziesz chciał śledzić stan synchronizacji dla każdego lokalnie zmodyfikowanego rekordu. Najprostszym sposobem na to jest włączenie właściwości do encji Core Data, która może reprezentować trzy stany:



- zsynchronizowany
- oczekuje na synchronizację 
- niezsynchronizowany

Liczba całkowita powinna działać dobrze. Możemy zmapować tę liczbę całkowitą na enum w Swift, aby mieć wygodny dostęp do stanu synchronizacji obiektu.

Następnie powinniśmy zdecydować, w jaki sposób poradzimy sobie z łączeniem naszych lokalnych danych z danymi zdalnymi. W poprzedniej sekcji mogliśmy zawsze zakładać, że przychodzący obiekt zawiera najbardziej aktualne informacje. W końcu użytkownik nie miał możliwości lokalnej modyfikacji rekordów. 

Z tego powodu mogliśmy użyć wbudowanej w Core Data obsługi konfliktów scalania, aby zastąpić dowolne właściwości w w magazynie Core Data z wartościami w pamięci w przypadku konfliktu unikalności. Nie będziemy mogli użyć tego podejścia w dwukierunkowej synchronizacji, ponieważ odpowiednia strategia scalania zależy od tego, czy zarządzany obiekt jest zsynchronizowany z serwerem, czy nie.

​	Alternatywnie, możesz upewnić się, że zawsze najpierw synchronizujesz swoje lokalne zmiany z serwerem, aby serwer mógł scalić twoje lokalne dane ze swoimi danymi. Po wysłaniu wszystkich lokalnych modyfikacji do serwera można pobrać dane ze zdalnego obiektu i mieć pewność, że wszystkie dane otrzymane ze zdalnego obiektu są aktualne.

​	Jak to często bywa w programowaniu, najlepsze rozwiązanie dla danej aplikacji zależy od wielu czynników. W tym przypadku rozwiązanie będzie prawdopodobnie zależeć również od tego, co jest najlepsze dla zespołu serwerowego. Podczas pracy nad funkcjami takimi jak ta, zdecydowanie zalecam usiąść z zespołem serwera i omówić strategię synchronizacji, która będzie łatwa w zarządzaniu w aplikacji i mam nadzieję, że będzie dobrze działać dla zespołu pracującego na serwerze.
Strategia synchronizacji, którą chciałbym pokazać w tej sekcji, będzie działać w następujący sposób:

- Zmiany będą gromadzone i przetwarzane co 20 sekund.
- Użyjemy wyszukiwarki, aby zatwierdzić zarządzane obiekty.
- Lokalne zmiany będą zawsze przerywane, jeśli lokalny obiekt nie jest w pełni zsynchronizowany.
zsynchronizowany
- Dla wygody przyjmiemy, że serwer odpowiada najnowszej wersji obiektów, które właśnie zsynchronizowaliśmy.
które właśnie zsynchronizowaliśmy, aby uwzględnić wszelkie scalenia, które miały miejsce na serwerze
- Zarządzane obiekty będą miały zsynchronizowane i zaktualizowane pole At, które może być używane przez serwer do określenia
przez serwer w celu określenia, kiedy obiekt był ostatnio modyfikowany

Jest to dość obszerna lista funkcji i zasad, które weźmiemy pod uwagę w tej dwukierunkowej strategii synchronizacji. Z mojego doświadczenia wynika, że strategia synchronizacji, która obejmuje wszystkie wymienione tutaj funkcje, powinna zapewnić solidne wrażenia i uwzględniać niektóre z bardziej złożonych scenariuszy. W zależności od potrzeb, możesz potencjalnie użyć znacznie prostszej strategii. Ponownie, zależy to w dużej mierze od aplikacji i wymagań zespołu serwera. Rozwiązanie, które zamierzam ci pokazać, jest jedynie sugestią opartą na wcześniejszych doświadczeniach.



### Wdrażanie strategii synchronizacji
Aby zaimplementować naszą strategię synchronizacji, będziemy musieli zdefiniować nowy obiekt Synchronizer, który zajmie się planowaniem i przetwarzaniem całego procesu synchronizacji.



Będziemy również musieli zaimplementować nową strategię importowania danych, aby zaimportować dane i upewnić się, że nie nadpisujemy lokalnie zmodyfikowanych danych, a także będziemy potrzebować mechanizmu pobierania lub wstawiania dla zarządzanych obiektów.
Będziemy pracować od krawędzi funkcji synchronizacji. Oznacza to, że najpierw dodamy nowe pola do `PointOfInterest`. Następnie zaimplementujemy mechanizm wyszukiwania lub wstawiania, a następnie zaimplementujemy nowy importer danych i na koniec utworzymy obiekt `Synchronizer`, który obsługuje synchronizację lokalnie zmodyfikowanych elementów.
Ponieważ zmiana jest dość prosta, oto nowe właściwości, które dodałem do `PointOfInterest`:

```swift
    // sync related properties
    @NSManaged public var synchronized: Int 
    @NSManaged public var updatedAt: Date

    public var synchronizationState: SynchronizationState {
        get {
            SynchronizationState(rawValue: synchronized) ?? .notSynchronized
        }
        set {
            synchronized = newValue.rawValue
        }
    }
```

Obie nowe zarządzane właściwości nie są opcjonalne. Właściwość synchronized będzie domyślnie miała wartość 0, co oznacza, że obiekt nie jest jeszcze zsynchronizowany. Dodałem wygodną właściwość o nazwie synchronizationState. Jej typ to enum, a jej wartość jest w całości oparta na właściwości synchronized, która jest Int.
Oto jak wygląda SynchronizationState:

```swift
public enum SynchronizationState: Int {
case notSynchronized = 0, synchronizationPending, synchronized
}
```

Jest to bardzo proste wyliczenie, a jego jedynym celem jest uczynienie naszego kodu nieco czystszym.
Kiedy dodajesz nowe właściwości do zarządzanego obiektu, nie zapomnij zaktualizować metody init(from:), aby zdekodować wszystkie właściwości, które spodziewasz się otrzymać z serwera. W tym przypadku właściwość updatedAt powinna zostać dodana tutaj.

#### Dodanie mechanizmu wyszukiwania lub wstawiania dla zarządzanych obiektów 

Mechanizm wyszukiwania lub wstawiania to coś, co prawie zawsze implementuję w każdej aplikacji Core Data, nad którą pracuję. Pozwala mi pobrać istniejący obiekt zarządzany lub utworzyć nowy w zaledwie jednej linii kodu. Mechanizm taki jak ten będzie zwykle działał wraz z polityką scalania w przypadkach takich jak ten, który napotykamy teraz. Czasami będziesz chciał nadpisać lokalną kopię nowymi danymi, innym razem chcesz zachować istniejące dane. Innym razem możesz chcieć nadpisać niektóre części rekordu, ale zachować inne nienaruszone.
Bez względu na powód, znajdź lub wstaw jest przydatnym narzędziem w zestawie narzędzi Core Data.
Mechanizm znajdowania lub zastępowania wymaga, aby zarządzane obiekty mogły być jednoznacznie identyfikowane. Na szczęście model danych, którego używamy w tym rozdziale, ma klasę PointOfInterest, która wykorzystuje właściwość identifier jako swój unikalny identyfikator. Dodamy statyczną metodę do PointOfInterest, która przyjmie UUID i zwróci istniejący PointOfInterest lub wstawi nowy do naszego magazynu Core Data:



```swift
    static func findOrInsert(using identifier: UUID, in context:
                             NSManagedObjectContext) -> PointOfInterest {
        let request = NSFetchRequest<PointOfInterest>(entityName: "PointOfInterest")
        request.predicate = NSPredicate(format: "%K == %@",
                                        #keyPath(PointOfInterest.identifier),
                                        identifier as NSUUID)
        if let poi = try? context.fetch(request).first { return poi
        } else {
            let poi = PointOfInterest(context: context)
            return poi }
    }}
```

Ta metoda powinna już mówić sama za siebie. Tworzymy żądanie pobierania, aby znaleźć punkty zainteresowania pasujące do otrzymanego identyfikatora. Ponieważ to żądanie powinno zwrócić tylko jedną wartość (identyfikator ma przecież unikalne ograniczenie), nie musimy ustawiać limitu pobierania, aby upewnić się, że pobieramy tylko jeden element.
Jednak nadal będziemy otrzymywać tablicę z fetch(_:), więc używamy .first, aby pobrać pierwszy wynik, jeśli taki istnieje, a następnie zwracamy go.
Jeśli nie zostanie znaleziony żaden istniejący rekord, wstawiamy nowy i zwracamy go.
Ten wzorzec jest prosty w implementacji i za chwilę okaże się całkiem skuteczny.

#### Aktualizacja obiektu DataImporter 

Dodamy nową metodę importu do DataImporter, która pobiera nieprzetworzone dane, konwertuje je na tablicę, a następnie przetwarza w celu zaimportowania.
Przejdźmy od razu do implementacji tej nowej metody:



```swift
public func importPointsOfInterestUsingData(_ data: Data) { 
  let entries = try! JSONSerialization.jsonObject(with: data,options: []) as! [[String: Any]] 
  for entry in entries {
    let uuid = UUID(uuidString: entry["identifier"] as! String)! let poi = 	        PointOfInterest.findOrInsert(using: uuid, in:self.context)

    if poi.objectID.isTemporaryID || poi.synchronizationState == .synchronized {
      poi.identifier = uuid
      poi.address = entry["address"] as! String
      poi.city = entry["city"] as! String
      poi.country = entry["country"] as! String
      poi.information = entry["information"] as! String
      poi.latitude = entry["latitude"] as! Float

      poi.longitude = entry["longitude"] as! Float
      poi.name = entry["name"] as! String
      poi.synchronizationState = .synchronized
    } else {
      // We have local changes, don't update
    } 
  }
}
```

Po przekonwertowaniu danych na tablicę typu [[String: Any]] przeglądamy wszystkie próby, które należy zaimportować. Wyodrębniam UUID z wpisu i przekazuję go do findOrIn- sert(using:in:), aby uzyskać PointOfInterest. Następnie sprawdzam, czy identyfikator obiektu dla tego obiektu jest tymczasowy. Jeśli tak, wiem, że ten obiekt jest nowy; nie ma jeszcze stałego identyfikatora obiektu, ponieważ nigdy nie został zapisany. Bezpiecznie jest zaktualizować nowy obiekt danymi ze zdalnego, ponieważ nie mam lokalnej niezsynchronizowanej kopii tego obiektu.
Alternatywnie, jeśli synchronizationState punktu zainteresowania jest zsynchronizowany, wiem, że nie ma on żadnych lokalnych modyfikacji. Oznacza to, że możemy bezpiecznie aktualizować dane lokalne danymi ze zdalnego obiektu. Nie używam tutaj JSONDecodera do wypełniania pól obiektu zarządzanego, ponieważ jego inicjalizacja jest wykonywana w findOrInsert(using:in:). Z tego powodu musimy pobrać wartości ze słownika wyodrębnionego przez JSONSerialization
Jeśli obiekt nie jest nowy, a jego synchronizationState nie jest zsynchronizowany, wiemy, że mamy do czynienia z istniejącym punktem zainteresowania, który ma lokalne modyfikacje, więc nie chcemy jeszcze aktualizować go zdalnymi danymi.
Będziemy chcieli najpierw zsynchronizować obiekt ze zdalnym, a następnie zaktualizować lokalny obiekt za pomocą odpowiedzi serwera.
W tej sekcji zakładam, że serwer odpowie na żądanie POST wysłane w celu synchronizacji danych z najbardziej aktualną wersją każdego obiektu, który został zawarty w POST.
Mówiąc o wykonywaniu żądania POST, jest jeszcze jedna zmiana, którą należy wprowadzić zarówno do PointOfInterest, jak i Category. Musimy dostosować je do protokołu Encodable, abyśmy mogli przekonwertować instancje tych obiektów na Data dla naszego uploadu. Zaktualizuj deklarację dla PointOfInterest, zastępując Decodable przez Codable, ponieważ Codable jest kombinacją Decodable i Encodable:

```swift
class PointOfInterest: NSManagedObject, Codable
```

Zrób to samo dla Category.
Następnie dodaj następującą implementację dla encode(to:) do PointOfInterest:

```swift
    func encode(to encoder: Encoder) throws {
        var container = encoder.container(keyedBy: CodingKeys.self)
        try container.encode(address, forKey: .address)
        try container.encode(city, forKey: .city)
        try container.encode(country, forKey: .country)
        try container.encode(identifier, forKey: .identifier) 
        try container.encode(information, forKey: .information)
        try container.encode(latitude, forKey: .latitude)
        try container.encode(longitude, forKey: .longitude) 
        try container.encode(name, forKey: .name)
        try container.encode(updatedAt, forKey: .updatedAt)
        try container.encode(category, forKey: .category)
    }
```

And add the following to Category:

```swift
func encode(to encoder: Encoder) throws {
var container = encoder.container(keyedBy: CodingKeys.self)
try container.encode(name, forKey: .name) 
}
```

Pozwoli nam to później użyć JSONEncodera do przekształcenia niezsynchronizowanych punktów zainteresowania w dane.

#### Implementacja obiektu Synchronizer 

Pozwoli nam to później użyć JSONEncodera do przekształcenia niezsynchronizowanych punktów zainteresowania w dane.

Implementacja obiektu Synchronizer 

Do tej pory uwzględniliśmy następujące funkcje, o których wspomniałem wcześniej w tej sekcji:

Użyjemy podejścia find lub insert do aktualizacji zarządzanych obiektów
Zmiany lokalne zawsze będą ważniejsze od zmian zdalnych, jeśli obiekt lokalny nie jest w pełni zsynchronizowany.
zsynchronizowany
Obiekty zarządzane będą miały pole updatedAt i synchronized, które mogą być
używane przez serwer do określenia, kiedy obiekt został ostatnio zmodyfikowany

Oznacza to, że mamy jeszcze tylko dwa pola do zaznaczenia:

Zmiany będą grupowane i przetwarzane w ciągu 20 sekund.
Dla wygody założymy, że serwer odpowiada najnowszą wersją obiektów, które właśnie zsynchronizowaliśmy, aby uwzględnić wszelkie scalenia, które miały miejsce na serwerze

Zanim pokażę ci sam synchronizator, zróbmy małą wycieczkę do królestwa sieci.
Nie zamierzam opowiadać się za konkretną architekturą aplikacji, ale jestem pewien, że wszyscy możemy się zgodzić, że rozdzielenie obaw w programie jest dobrym pomysłem. 
Z tego powodu zdecydowanie zalecam, aby nie umieszczać kodu sieciowego w klasie Synchronizer. 
Zamiast tego lepiej jest wstrzyknąć obiekt Networking.
To, jak będzie wyglądał twój stos sieciowy, zależy wyłącznie od ciebie, więc potraktuj obiekt Networking, który zaraz zobaczysz, jako inspirację do stworzenia własnego. Wymyśliłem jedynie obiekt, który jest tak prosty, jak to tylko możliwe, a jednocześnie zapewnia wystarczającą funkcjonalność, aby był użyteczny. Kod, który zaraz zobaczysz, jest kompletną implementacją mojej klasy Networking. Omówię go krok po kroku, więc nic się nie stanie, jeśli nie zrozumiesz wszystkiego za jednym razem:

```swift
public class Networking { 
  public init() {
    // 1
    URLProtocol.registerClass(FakeServer.self)
  }
  func uploadPointsOfInterest(_ pointsOfInterest: [PointOfInterest])  -> AnyPublisher<(data: Data,response: URLResponse), URLError>  {

    // 2
    guard let context = pointsOfInterest.first?.managedObjectContext else {
      fatalError("uploadPointsOfInterest should be called with an array of pois that exist in a managed object context")
    }
    // 3
    context.performAndWait {
      for poi in pointsOfInterest {
        poi.synchronizationState = .synchronizationPending
      }
      try! context.save() }
    // 4
    let encoder = JSONEncoder()
    let data = try! encoder.encode(pointsOfInterest)
    // 5
    let url = URL(string: "http://my-server.com/upload")! 
    var request = URLRequest(url: url)
    request.httpMethod = "POST"
    request.httpBody = data
    // 6
    FakeServer.preparedData = data
    // 7
    return URLSession.shared.dataTaskPublisher(for: request) .eraseToAnyPublisher()
  }
}
```

W pierwszym kroku tego kodu robię coś, czego być może nigdy wcześniej nie widziałeś:



```swift
 URLProtocol.registerClass(FakeServer.self)
```

Ten wiersz kodu rejestruje klasę, którą pokażę po wyjaśnieniu mojego obiektu Networking. Rejestrując klasę FakeServer w URLProtocol, mogę przejąć pełną kontrolę nad tym, co dzieje się, gdy próbuję wysłać dane POST do serwera. Oznacza to, że mogę naśladować odpowiedź serwera w aplikacji, co bardzo ułatwia testowanie synchronizatora.
W przypadku kodu produkcyjnego nie powinieneś tego robić, ponieważ przechwytywałbyś wszystkie połączenia sieciowe z serwerem, ale do testów URLProtocol jest świetny. Jest to kolejna funkcja, która nie jest związana z Core Data, ale nadal jest fajną funkcją, którą chciałem ci pokazać. Więcej na temat URLProtocol później. 

Przejdźmy do drugiego kroku w klasie Networking.

```swift
    guard let context = pointsOfInterest.first?.managedObjectContext else {
      fatalError("uploadPointsOfInterest should be called with an array of pois that exist in a managed object context")
    }
```

W tej części kodu pobieram kontekst obiektu zarządzanego z pierwszego POI, który synchronizujemy. Ponieważ wszystkie moje punkty zainteresowania powinny być pobierane przy użyciu zarządzanego kontekstu obiektu, spodziewam się, że każdy punkt zainteresowania, który zamierzamy zsynchronizować, ma zarządzany kontekst obiektu, którego można użyć do wykonania niektórych żądań.

```swift
    context.performAndWait {
      for poi in pointsOfInterest {
        poi.synchronizationState = .synchronizationPending
      }
      try! context.save() }
```

Ponieważ mamy zamiar zsynchronizować otrzymane obiekty z naszym serwerem, powinniśmy zaktualizować ich bieżący stan synchronizacji do synchronizationPending. W razie potrzeby możemy zaktualizować interfejs użytkownika, aby to odzwierciedlić, a to gwarantuje, że te rekordy nie zostaną odebrane, jeśli rozpoczniemy kolejną synchronizację, gdy ta jest jeszcze w toku (na przykład z powodu braku połączenia z Internetem).

```swift
    let encoder = JSONEncoder()
    let data = try! encoder.encode(pointsOfInterest)
```

Czwarty krok mówi sam za siebie.



```swift
    let url = URL(string: "http://my-server.com/upload")! 
    var request = URLRequest(url: url)
    request.httpMethod = "POST"
    request.httpBody = data
```

Zakodowane POI są używane jako httpBody w żądaniu POST. Ponownie, to mówi samo za siebie.

```swift
    let url = URL(string: "http://my-server.com/upload")! 
    var request = URLRequest(url: url)
    request.httpMethod = "POST"
    request.httpBody = data
```

W szóstym kroku przypisuję pewne dane, których użyję jako odpowiedzi, gdy mój FakeServer zostanie poproszony o przetworzenie URLRequest, który został skonfigurowany w poprzednim kroku.

```swift
FakeServer.preparedData = data
```

Wreszcie, po całej tej konfiguracji zwracam wydawcę zadania danych Combine.

Zanim przejdziemy do synchronizatora, rzućmy okiem na FakeServer, aby zrozumieć, co robi:

```swift
class FakeServer: URLProtocol { 
  static var preparedData: Data!
  override class func canInit(with task: URLSessionTask) -> Bool {
    return task.currentRequest?.url?.absoluteString == "https://my-server.com/upload" 
  }
  override class func canonicalRequest(for request: URLRequest) -> URLRequest {
    return request
  }
  override func startLoading() { 
    DispatchQueue.global().async {
      self.client?.urlProtocol(self, didLoad: Self.preparedData)
      self.client?.urlProtocol(self, didReceive: URLResponse(),  
                               cacheStoragePolicy: .notAllowed)
      self.client?.urlProtocolDidFinishLoading(self)
    } }
  override func stopLoading() { }
}
```

Ponieważ zarejestrowałem FakeServer na obiekcie URLProtocol, jego metoda `canInit(with:)` będzie wywoływana dla każdego żądania sieciowego, które wykona aplikacja. Sprawdzam, czy adres URL zadania jest moim fikcyjnym adresem URL przesyłania, a jeśli tak, zwracam wartość true, aby wskazać, że chcę, aby FakeServer obsłużył żądanie zamiast pozwalać mojej sesji URL na jego obsługę.

Następnie jestem proszony o podanie kanonicznego żądania dla URLRequest, które zamierzamy załadować. Po prostu zwracam zadanie, które zostało przekazane jako argument. Implementacja tej metody jest obowiązkowa, ale nie musisz robić nic wymyślnego. Następnie wywoływana jest metoda `startLoading()`. To tutaj mamy obsłużyć żądanie. 
Gdy dane są gotowe, należy przekazać je do obiektu klienta, który dziedziczy po URLProtocol, wywołując `urlProtocol(_:didLoad:)`. Po załadowaniu wszystkich danych należy wywołać `urlProtocol(_:didReceive:cacheStoragePolicy:)`, aby przekazać obiekt `URLResponse` do klienta. Na koniec wywołujemy `urlProtocolDidFinishLoading(_:)`. Powoduje to zakończenie fałszywego połączenia sieciowego i zwykle wywołuje program obsługi zakończenia zadania danych.

 Jeśli używasz Combine, oznacza to, że wydawca zadania danych wyemituje swoje dane wyjściowe i zakończy działanie.
Mam pustą implementację dla `stopLoading()`, ponieważ nie obsługuję anulowania mojego fałszywego żądania sieciowego.
Niestandardowy `URLProtocol` może być bardzo przydatny, gdy chcesz przetestować niektóre funkcje związane z siecią w swojej aplikacji bez konieczności korzystania z serwera. Jest idealny do testowania czegoś takiego jak nasza strategia synchronizacji.
W każdym razie, dygresja. Wracając do tematu synchronizacji! Poniższy kod jest zarysem synchronizatora:

```swift
    public class Synchronizer {
        let context: NSManagedObjectContext
        private var timerCancellable: AnyCancellable?
        let networking: Networking let importer: DataImporter
        public init(context: NSManagedObjectContext, networking: Networking  = .init()) {
            self.context = context
            self.networking = networking
            self.importer = DataImporter(context: context)
        }
        public func start() {
            guard timerCancellable == nil else {
                return
            }
            timerCancellable = Timer.publish(every: 20, on: .current, in: .default)
                .autoconnect()
            /* ... we'll do a bunch of work here */ .sink(receiveValue: { _ in
                print("completed a sync call")
            })
        }
    }
```

Mój synchronizator wymaga do działania kontekstu obiektu zarządzanego i obiektu sieciowego. Kontekst ma być w stanie pobierać niezsynchronizowane obiekty, a obiekt sieciowy jest potrzebny do wykonania faktycznej synchronizacji.
Będę również używał Combine Timer, więc muszę przechowywać odwołanie, które otrzymam, gdy zasubskrybuję ten Timer.
`Synchronizator` będzie również używał `DataImporter` do importowania punktów zainteresowania, które (fałszywy) serwer zwraca w odpowiedzi na nasze żądania synchronizacji.
Nie będę wchodził w szczegóły Combine i tego, jak działa wydawca Timera. Najważniejszą rzeczą do zrozumienia jest to, że timer będzie wysyłał datę co 20 sekund. Możemy mapować, `flatMap` i więcej na Timerze, aby przekształcić tę datę w inne rzeczy. Jak zobaczysz za chwilę, jest to idealne rozwiązanie dla naszego przypadku użycia.
Pierwszym krokiem w naszym potoku synchronizacji jest zareagowanie na emisję wartości Date poprzez pobranie wszystkich niezsynchronizowanych punktów zainteresowania w mapie i zwrócenie ich. Zasadniczo zamieniamy Date na `[PointOfInterest]`, dzięki czemu możemy zastosować nowy operator po mapie.
Oto jak to wygląda:

```swift
    timerCancellable = Timer.publish(every: 10, on: .current, in:  .default)
    .autoconnect()
    .map({ (_) -> [PointOfInterest] in
        let request = PointOfInterest.unsyncedFetchRequest
        let unsynced: [PointOfInterest] = try! self.context.fetch(request)
        return unsynced
        })
        /* ... we'll do a bunch of work here */
        .sink(receiveValue: { _ in print("completed a sync call")
        })
```

Dodałem wygodną właściwość `PointOfInterest`, aby utworzyć żądanie pobierania, które ma predykat wybierający wszystkie niezsynchronizowane punkty zainteresowania.
Następnym krokiem jest dodanie flatMap, która pobiera niezsynchronizowane punkty zainteresowania i przesyła je za pomocą obiektu sieciowego.

```swift
    timerCancellable = Timer.publish(every: 10, on: .current, in: .default)
    .autoconnect()
    .map({ (_) -> [PointOfInterest] in
        let request = PointOfInterest.unsyncedFetchRequest
        let unsynced: [PointOfInterest] = try! self.context.fetch(request) return unsynced
    })
    .flatMap({ unsynced -> AnyPublisher<Void, Never> in
        guard !unsynced.isEmpty else {
            return Empty().eraseToAnyPublisher()
        }
        return self.networking.uploadPointsOfInterest(unsynced) 
              /* We'll do more work here... */
    })
    .sink(receiveCompletion: { _ in }, receiveValue: { _ in
        print("done!")
    })
```

​	Właśnie dodana flatMap jest używana do zwrócenia nowego wydawcy Combine. Dane wyjściowe tego nowego `publisher` są używane jako dane wejściowe dla `sink`, który znajduje się na końcu. Ponieważ nie chcemy nic robić z wynikiem naszej pracy (za chwilę zobaczysz dlaczego), zamknięcie flatMap zwraca wydawcę typu AnyPublisher<Void, Never>. Innymi słowy, będziemy publikować puste wartości, a wydawca nigdy nie zgłosi błędu.
​	Wewnątrz flatMap sprawdzam, czy mamy jakieś zarządzane obiekty do przetworzenia. Jeśli nie, zwracam pusty publikator. Ten wydawca nie emituje żadnych wartości i po prostu kończy swoją pracę, nie robiąc nic.
​	Jeśli istnieją elementy, które muszą zostać zsynchronizowane, przekazuję je do mojego obiektu networking.
​	Ponieważ funkcja uploadPointsOfInterest(_:) obiektu sieciowego zwraca wydawcę, który wysyła(data: Data, response: URLResponse), będę potrzebował przetworzyć wyemitowane dane, aby zaimportować je do mojego magazynu danych i ustawić synchronizationState każdego zsynchronizowanego rekordu na synchronized.
​	Pierwszą rzeczą, którą chcę tutaj dodać, jest możliwość aktualizacji stanu synchronizacji dla zsynchronizowanych obiektów:

```swift
   timerCancellable = Timer.publish(every: 10, on: .current, in: .default)
        .autoconnect()
        .map({ (_) -> [PointOfInterest] in
            let request = PointOfInterest.unsyncedFetchRequest
            let unsynced: [PointOfInterest] = try! self.context.fetch(request) return unsynced
        })
        .flatMap({ unsynced -> AnyPublisher<Void, Never> in
            guard !unsynced.isEmpty else {
                return Empty().eraseToAnyPublisher()
            }
            return self.networking.uploadPointsOfInterest(unsynced)

                .handleEvents(receiveOutput: { output in
                    self.context.performAndWait {
                        for poi in unsynced {
                            poi.synchronizationState = .synchronized
                        }
                        try! self.context.save()
                        self.importer.importPointsOfInterestUsingData(output.data)
                    }
                }, receiveCompletion: { completion in
                    if case .failure = completion {
                        self.context.performAndWait {
                            for poi in unsynced {
                                poi.synchronizationState = .notSynchronized
                            }
                            try! self.context.save()
                        }
                    }
                })
            // we're not quite there yet

        })
        .sink(receiveCompletion: { _ in }, receiveValue: { _ in
            print("done!")
        })
```

​	Zauważ, że dodałem operator `handleEvents` i przekazałem mu zamknięcie `receiveOutput` i `re-ceiveCompletion`. Zamknięcia te są wywoływane, gdy wydawca zadania danych emituje dane i gdy zadanie kończy działanie. Gdy zadanie wysyła dane, chcemy skorzystać z tej okazji, aby ustawić stan synchronizacji dla wszystkich wcześniej niezsynchronizowanych punktów zainteresowania na zsynchronizowane. Chcemy również przekazać dane, które są częścią danych wyjściowych zadania danych do naszego importera, aby mógł zaktualizować nasze lokalne kopie zsynchronizowanych obiektów o nowe dane.

​	Gdy wydawca zadania danych zakończy działanie, może to zrobić pomyślnie lub z błędem. Gdy zadanie zakończy się pomyślnie, zamknięcie `receiveOutput` zostało wywołane i nie mamy nic do zrobienia. Wszystkie punkty zainteresowania są oznaczone jako zsynchronizowane i zostały zaktualizowane o najnowsze dane z naszego serwera.
Jeśli jednak połączenie sieciowe nie powiedzie się z jakiegokolwiek powodu, będziemy chcieli zresetować stan synchronizacji, aby był niezsynchronizowany. Robię to, sprawdzając obiekt ukończenia, który jest przekazywany do mojego zamknięcia receiveCompletion i aktualizując moje punkty zainteresowania, jeśli ukończenie jest równe niepowodzeniu.
​	Ponieważ `handleEvent` nie modyfikuje danych wyjściowych z wydawcy zadania danych, a jedynie na nie reaguje, nadal musimy wprowadzić pewne modyfikacje. Będziemy chcieli przekształcić dane wyjściowe wydawcy zadania danych w Void i wychwycić wszelkie błędy, aby zwrócić wydawcę, który nigdy nie zawiedzie z naszej `flatMap`.
Oto jak wygląda pełny potok dla wydawcy sieciowego. Poniższy kod powinien znajdować się wewnątrz wcześniejszej `flatMap`. Pominąłem otaczający kod, aby zapobiec wklejaniu tego samego kodu w kółko.



```swift
return self.networking.uploadPointsOfInterest(unsynced)
.handleEvents(receiveOutput: { output in
                              self.context.performAndWait {
                                for poi in unsynced {
                                  print("updating sync status for poi \(poi.identifier)")
                                  poi.synchronizationState = .synchronized
                                }

                                try! self.context.save()
                                self.importer.importPointsOfInterestUsingData(output.data)
                              }
                             }, receiveCompletion: { completion in
                                                    if case .failure = completion {
                                                      self.context.performAndWait {
                                                        for poi in unsynced {
                                                          poi.synchronizationState = .notSynchronized
                                                        }
                                                        try! self.context.save() 
                                                      }
                                                    } 
                                                   })
.map({ _ in return () }) 
.catch({ _ in Empty() })
.eraseToAnyPublisher()
```

Ostatnie trzy operatory w tym łańcuchu zamieniają dane wyjściowe zadania na Void, zastępują wszelkie niepowodzenia funkcją `Empty()`, a wynikowy wydawca jest usuwany jako `AnyPublisher<Void, Never>`.

Z tym kodem synchronizator powinien być gotowy do pracy. Możesz utworzyć instancję obiektu Synchronizer w dogodnym miejscu, takim jak AppDelegate, struktura aplikacji lub nawet w StorageProvider i wywołać na nim start(). Spowoduje to uruchomienie timera, a synchronizator pobierze i prześle wszelkie niezsynchronizowane punkty zainteresowania na serwer.

​	Strategie synchronizacji nie są łatwe ani proste, ale ich implementacje mogą być dość proste. W przykładzie, który pokazałem w tej sekcji, wzięliśmy pięć wymagań, które moim zdaniem każda dobra strategia synchronizacji zaimplementuje w jakiś sposób i pokazałem, jak zaimplementować te wymagania przy stosunkowo niewielkiej ilości kodu.

​	Myślę, że kiedy zaczniesz pracować nad własną strategią synchronizacji, przekonasz się, że ważne jest, aby przemyśleć swoją strategię przed rozpoczęciem kodowania. Pomoże ci to znaleźć najkrótszą i najprostszą ścieżkę kodu dla strategii, która odpowiada twoim potrzebom.
Miejmy nadzieję, że dzięki przykładom z tego rozdziału zyskałeś inspirację i wgląd w to, jak wygląda podstawowa strategia.



## Podsumowanie
W tym rozdziale pokazałem, jak skonfigurować magazyn Core Data, aby stworzyć fantastyczne środowisko offline dla użytkowników.
Po pierwsze, dowiedziałeś się, jak ładować dane z serwera i przechowywać je lokalnie, aby zapisać dane użytkownika i uniknąć konieczności każdorazowego łączenia się z siecią. Dowiedziałeś się, jak ustawić zasady scalania w kontekście obiektu zarządzanego i zobaczyłeś, jak dodać ograniczenia unikalności do obiektów zarządzanych. Pokazałem, w jaki sposób można dostosować obiekty zarządzane do Decodable, aby uzyskać bardzo wygodny mechanizm importu, i pokazałem, jak można użyć żądania wstawiania wsadowego, aby znacznie poprawić wydajność importu, jeśli importujesz zestaw danych, który nie obejmuje relacji.

Następnie przeszliśmy dalej i dowiedziałeś się o dwukierunkowej synchronizacji, w której synchronizujesz lokalne modyfikacje z powrotem na serwer. Wyjaśniłem, w jaki sposób można określić odpowiednią strategię synchronizacji, a następnie pokazałem, jak wdrożyć przykładową strategię. W trakcie tego procesu dowiedziałeś się o udawaniu odpowiedzi sieciowej za pomocą protokołu URLProtocol, a nawet zobaczyłeś trochę kodu Combine.

Użytkownicy oczekują płynnych doświadczeń na swoich urządzeniach i ma dla nich sens, że dane z jednego urządzenia w magiczny sposób pojawią się na innych urządzeniach. Ułatwienie tego nie jest łatwym zadaniem. Ten rozdział jest jednym z najdłuższych w książce i jestem w pełni świadomy, że wciąż ma luki. Powodem tego nie jest to, że nie chciałem o tym pisać. Powodem jest to, że jest po prostu zbyt wiele do omówienia w niestandardowym rozwiązaniu, zwłaszcza jeśli back-end dla każdej aplikacji może być zupełnie inny, co będzie miało ogromny wpływ na kod, który piszesz w swojej aplikacji.

Na szczęście iOS może zaoszczędzić nam wiele bólu głowy dzięki iCloud. W następnym rozdziale pokażę, jak skonfigurować aplikację do opartej na iCloud synchronizacji magazynu danych podstawowych na wszystkich urządzeniach użytkownika za pomocą `NSPersistentCloudKitContainer`.
