# VIPER dla SwiftUI? Proszę. Nie.

Albo dlaczego nie powinieneś głaskać tego konkretnego węża.

Wszyscy uwielbiają pisać o tym, jak wykorzystują swoją ulubioną architekturę UIKit w SwiftUI. Jednym z bardziej znanych wzorców projektowych/architektur jest VIPER.

View-Interactor-Presenter-Entity-Router.

Nie zrozum mnie źle. VIPER to świetna architektura oprogramowania, zaprojektowana w celu rozwiązania wielu klasycznych i powszechnych problemów występujących podczas opracowywania i tworzenia aplikacji opartych na UIKit i UIViewController.

Ale nie jest to najlepsza architektura dla SwiftUI. Ani drugą najlepszą. W rzeczywistości jest jedną z najgorszych i w tym artykule mam nadzieję pokazać dlaczego.

Ale zanim to zrobię, najpierw musimy wiedzieć dwie rzeczy.

1. Czym jest VIPER?
2. Jakie problemy rozwiązuje?

Zagłębmy się w temat.

## Co to jest VIPER?

Jak wspomniano powyżej, VIPER to akronim oznaczający View-Interactor-Presenter-Entity-Router.
Jeśli jesteś tutaj i czytasz ten tekst, prawdopodobnie znasz już VIPER. Zamiast więc poświęcać czas na samodzielne wyjaśnianie każdego komponentu, pójdę na skróty i zapożyczę trochę z bardzo dobrze napisanego artykułu na ten temat autorstwa Mahdiego Chtioui.

https://medium.com/@mahdichtioui/implementing-viper-archticture-pattern-for-ios-d24a6def8ba2

Oto składniki modułu VIPER.

**Widok**: Warstwa widoku, która jest w zasadzie UIViewController i dowolnym innym typem widoku. Warstwa ta zawiera logikę interfejsu użytkownika (wyświetlanie, aktualizacja, animacja...) i jest odpowiedzialna za przechwytywanie akcji użytkownika i wysyłanie ich do prezentera. Co najważniejsze, nie posiada logiki biznesowej.

**Interaktor**: Możemy o nim myśleć jako o "menedżerze sieci": odpowiada za pobieranie danych z usług (sieć, baza danych, czujniki...) na żądanie prezentera. Interaktor jest odpowiedzialny za zarządzanie danymi z warstwy modelu (zauważ, że model nie jest częścią architektury VIPER, możesz go zaimplementować lub nie, ale na pewno sprawi, że nasza aplikacja będzie bardziej zwięzła).

**Prezenter**: Lubię myśleć o nim jak o płycie głównej, która łączy wszystkie warstwy razem. Prezenter jest jedyną warstwą, która komunikuje się z widokiem (pozostałe warstwy komunikują się z prezenterem). Zasadniczo jest to warstwa odpowiedzialna za podejmowanie decyzji w oparciu o działania użytkownika wysyłane przez Widok.

**Podmiot**: Podmioty to po prostu nasze modele, które są używane przez interaktora. Najlepiej jest umieścić je poza modułami VIPER, przynajmniej z mojego punktu widzenia, ponieważ te encje są zwykle współdzielone w całym systemie przez różne moduły.

**Router**: Ktoś nazywa to "Wireframe". Ta warstwa jest odpowiedzialna za obsługę logiki nawigacji: Pushing, Popping, Presenting UIViewControllers.

VIPER jest jedną z najpopularniejszych tzw. architektur CLEAN. Ale po co w ogóle jej używać?

## Jaki problem rozwiązuje?

VIPER jest jednym z wielu rozwiązań jednego z największych problemów w tradycyjnym tworzeniu oprogramowania UIKit MVC (Model-View-Controller).

Problem jest tak powszechny, że skrót MVC szybko zyskał inną, równie znaną definicję.
MVC: Massive-View-Controller.

Znasz ten problem. Zbyt często w MVC umieszczaliśmy cały nasz kod dla danego ekranu w kontrolerze widoku tego ekranu. Etykiety, pola tekstowe i kontrolery widoku. Przycisk i obsługa akcji. Delegacja i obsługa zdarzeń. Logika biznesowa. Walidacja. Żądania sieciowe i obsługa błędów. Routing i kontrola nawigacji.

To był bałagan.

VIPER został stworzony, aby rozwiązać ten problem i zrobił to poprzez skupienie się na jednej podstawowej koncepcji, zasadzie pojedynczej odpowiedzialności.
Cały kod zawarty wcześniej w kontrolerze widoku został rozbity, a jego fragmenty przeniesione do konkretnej, dobrze zdefiniowanej struktury. Każdy komponent ma określoną odpowiedzialność i rolę do odegrania w procesie.

Widok rozmawia z prezenterem. Prezenter rozmawia z Interaktorem. Interactor zwraca Entities do Presentera, który dzieli wszystko na łatwo przyswajalne fragmenty, które mogą zostać skonsumowane przez View. Po zakończeniu Presenter rozmawia z Routerem i przechodzi do kolejnego modułu VIPER w łańcuchu.

Przepłucz. Powtórz w razie potrzeby.

Jak widać, VIPER tworzy wiele małych ruchomych części i niestety znaczny procent kodu napisanego w aplikacji VIPER jest napisany tylko po to, aby zarządzać VIPER i połączyć wszystkie te elementy z powrotem.
W rzeczywistości jeden z artykułów, który przeczytałem na ten temat, sugerował, że VIPER tworzy i wykorzystuje tak wiele standardowego kodu, że należy polegać na generatorze kodu, aby stworzyć wszystko, co potrzebne.

https://theswiftdev.com/the-ultimate-viper-architecture-tutorial/

Ale... wszystko to zostało powiedziane... to działa.
Działa... ale nie nadaje się do SwiftUI.
Dlaczego? Cóż, na początek, został zaprojektowany na inny czas. I na inny język. I inne SDK.

## Delegacja

VIPER wywodzi się z tradycyjnego programowania opartego na Objective-C i wczesnym Swift UIKit i to widać.
VIPER działa głównie poprzez delegowanie. Każdy komponent w systemie ma interfejs i aby komponent mógł rozmawiać z innym komponentem, potrzebuje odniesienia do tego komponentu.
Utwórz i zaprezentuj nowy UIViewController i Presenter, a wszystko inne musi zostać utworzone, połączone i powiązane ze sobą w formalną, sztywną strukturę.
Kontroler widoku musi wiedzieć o Presenterze, więc ma do niego silne odniesienie. Ale Presenter musi również wiedzieć o kontrolerze widoku, aby mógł sterować zmianami interfejsu użytkownika. W związku z tym Presenter potrzebuje słabego powiązania delegata z powrotem do kontrolera widoku (należy obserwować te cykle referencyjne).
I tak jest z resztą komponentów w VIPER.
Poproszony o stworzenie takiego systemu dzisiaj, zamiast przewodowej delegacji, można by skupić się bardziej na funkcjach z zamknięciami wywołania zwrotnego, a nawet użyć RxSwift lub Combine do komunikacji komponentów i routingu wiadomości.
Narzędzia te nie były jednak dostępne ani powszechnie używane. Niewiele osób znało lub używało Rx, a składnia bloków w Objective-C była tak zła, że powstało wiele stron wyjaśniających, jak je definiować, tworzyć i używać.
Tak więc delegacja została wybrana jako główny mechanizm komunikacji między warstwami. Okej. To trochę niezgrabne, ale w porządku. Powinniśmy być w stanie z tym żyć.

## Widok nie istnieje

Z wyjątkiem SwiftUI widok nie jest widokiem. W UIKit UIView lub UIViewController jest typem referencyjnym. Mogę do niego linkować i do niego delgować.
W SwiftUI "widoki" są lekkimi strukturami, które istnieją w celu zdefiniowania naszej aplikacji. Nie są one typami referencyjnymi, są typami wartości i jako takie Presenter nie może przechowywać odniesienia do jednego z nich.
W SwiftUI widoki nie są sterowane przez imperatywne wywołania z prezentera do widoku w celu aktualizacji interfejsu użytkownika. Są one sterowane przez system, w oparciu o zmiany w obserwowalnych danych.
W VIPER routery ukrywały złożoność tworzenia i przesuwania widoków oraz nawigowania między nimi. Ale w SwiftUI nawigacja jest tak prosta, jak zawijanie widoku w NavigationLink lub dodawanie wartości do NavigationStack.
I tak, gdy każde behawioralne domino upada, cała struktura i racjonalność, od której zależy VIPER, zaczyna się rozpadać. Dosłownie spełnia swoją nazwę i staje się gniazdem węży, a im głębiej w nie sięgasz, tym bardziej prawdopodobne jest, że zostaniesz ugryziony.
Niedawno widziałem kilka przykładów z jednej bazy kodu, w której programista został zmuszony do dodania ViewModel do swojego kodu VIPER tylko po to, aby mieć miejsce na umieszczenie obserwowalnych wartości. VIPERVM?
Ale na razie zignorujmy to wszystko, ponieważ VIPER ma większy problem.
VIPER był w dużej mierze odpowiedzią... na niewłaściwe pytanie.

## Jak naprawić kontrolery Massive-View?

Zapytany w ten sposób, VIPER wydaje się logiczną odpowiedzią. Wyodrębnić cały kod, który możemy i przenieść go gdzie indziej.
Aby być uczciwym, niektóre z tych kodów chcą być gdzie indziej, bez względu na to, co zrobimy. Na przykład kod sieciowy.
Ale ta prosta odpowiedź omija prawdziwe pytanie.
P: Dlaczego nasz kontroler widoku jest tak duży... w pierwszej kolejności?
Chcesz poznać odpowiedź? Oto ona.
A: Ponieważ UIKit sprawia, że kompozycja jest trudna.
I to jest sedno sprawy.
Rozbijanie UIView i UIViewControllers na mniejsze komponenty jest trudne. Tak trudne, że w zasadzie tego nie robimy. I tak często kończyliśmy z pojedynczym UIViewController, który sterował wszystkim, co mogliśmy zobaczyć na jednym ekranie.
Potrzebujesz przykładów?

### Komponenty UIKit

Chcesz dodać oddzielny kontroler UIViewController do ekranu? Czy można po prostu dodać widok? Po prostu dodać kontroler widoku jako podwidok? Nie. Musisz zrobić coś takiego.

```swift
let controller = AccountDetailsViewController()
controller.accounts = accounts
addChild(controller)
controller.view.translatesAutoresizingMaskIntoConstraints = false
container.addSubview(controller.view)
NSLayoutConstraint.activate([
    controller.view.leadingAnchor.constraint(equalTo: container.leadingAnchor, constant: 0),
    controller.view.trailingAnchor.constraint(equalTo: container.trailingAnchor, constant: 0),
    controller.view.topAnchor.constraint(equalTo: container.topAnchor, constant: 0),
    controller.view.bottomAnchor.constraint(equalTo: container.bottomAnchor, constant: 0)
])
controller.didMove(toParent: self)
```

Chcesz użyć elementu interfejsu utworzonego w XIB? O rany. Najpierw musimy sprawić, by system stworzył go dla nas.

```swift
let cardView = NSBundle.mainBundle("CardView").loadNibNamed("", owner: nil, options: nil)[0] as! CardView
```

I to przy założeniu, że poprawnie zaimplementowałeś wymaganą metodę init?(coder aDecoder: NSCoder) w klasieCardView.

UIKit zachęca do pisania kodu do tworzenia widoków, pisania XIB do tworzenia widoków, pisania Storyboardów do tworzenia widoków. Ale czy chcesz używać ich razem? Osadzić jeden w drugim? Tutaj zaczyna się ból.

UIKit dodatkowo komplikuje sprawę, wymagając specjalnych widoków na specjalne okoliczności. Potrzebujesz nagłówka dla widoku tabeli? Cóż, to nie jest UIView, to jest UITableViewHeaderFooterView. Chcesz wyświetlić widok na liście? Oczywiście UITableViewCell. A może UIContainerViewCell, jeśli idziesz w tę stronę. Ale hej! Chcesz użyć tego widoku komórki, który właśnie napisałeś w innym widoku? Nie.
UIKit aktywnie działa przeciwko kompozycji widoków.

A kiedy system aktywnie działa przeciwko rozwiązaniu... czy jest niespodzianką, że robimy go mniej?
Ale gdyby to nie było takie uciążliwe. Gdyby rozbijanie naszych widoków i kontrolerów widoku na mniejsze widoki i kontrolery widoku było łatwiejsze...

Czy nasze masywne kontrolery widoków byłyby w ogóle tak masywne?

Skład
Pisałem o tym wielokrotnie.

https://medium.com/swlh/structural-decomposition-in-swiftui-8892e512b18e

https://medium.com/swlh/deep-inside-views-state-and-performance-in-swiftui-d23a3a44b79

 Jedną z podstawowych zasad, na których opiera się SwiftUI, jest kompozycja.
Chcesz osadzić CardView utworzony gdzie indziej w naszym widoku?

```swift
VStack {
    ...
    CardView(title: "Account", item: account)
    ...
}
```

Bum. To takie proste. Chcesz użyć tego na liście?

```swift
List {
    ForEach(accounts) { account in
        CardView(title: "Account", item: account)
    }
}
```

Zrobione



Jak wskazałem w artykule Najlepsze praktyki w kompozycji SwiftUI, jeśli jedna rzecz była powtarzana w kółko podczas sesji SwiftUI na WWDC, to fakt, że widoki SwiftUI są niezwykle lekkie, a ich tworzenie wiąże się z niewielkim lub zerowym spadkiem wydajności.
Chcesz podzielić część widoku na mniejszy widok podrzędny? To szybkie i łatwe. Heck, w Xcode możesz po prostu kliknąć polecenie na części kodu i wybrać opcję "Extract Subview".
Być może teraz widzisz, dokąd to zmierza...

### View-Interactor-Presenter-Entity-Router

Jeśli tworzenie i używanie małych, dedykowanych widoków i komponentów specjalnego przeznaczenia jest łatwe w SwiftUI... to dlaczego miałbym chcieć korzystać z architektury, która mnie do tego zniechęca?

Architektury, która wymaga ode mnie przeniesienia kodu widoku... i powiązanego kodu prezentera... i odpowiedniego kodu interaktora... i wszystkiego innego, co jest potrzebne.

Zauważ, że wspomniana wcześniej próbka SwiftUI VIPERVM miała ponad 200 linii kodu szablonu VIPER... wszystko do obsługi jednego 20-liniowego widoku.

Dwieście linii kodu.

Więc jeśli każdy nowy widok wymaga również mnóstwa dodatkowych klas i komponentów utworzonych w celu jego obsługi... czy zamierzam tworzyć wiele prostych podwidoków w razie potrzeby? Czy też zamierzam uniknąć kłopotów i nadal konsolidować wszystko w coraz większe widoki SwiftUI?

Czy zamierzam zastąpić Massive-View-Controllers przez Massive-View-Bodies?

Mamy VIPER, ponieważ UIKit zniechęcał do kompozycji widoków.
Ale jeśli VIPER działa również przeciwko nam...

## SwiftUI

Jak podsumowałem w SwiftUI: Wybór architektury aplikacji:

https://betterprogramming.pub/swiftui-choosing-an-application-architecture-6ec9289f8e8f

SwiftUI wprowadza proste deklaratywne podejście do pisania aplikacji. Co więcej, jest ono zwięzłe. Można wykonać wiele pracy przy użyciu bardzo małej ilości kodu.
W związku z tym szkoda byłoby obciążać siebie i nasze aplikacje zbyt formalnymi architekturami, które po raz kolejny dramatycznie zwiększają ilość kodu, który musimy napisać.
I z pewnością nie potrzebujemy architektur zaprojektowanych dla innego SDK, zbudowanych i zaprojektowanych do obejścia i przezwyciężenia ograniczeń innego języka i innego środowiska.
Potrzebujemy architektury, która pozwoli SwiftUI... być SwiftUI.

## Blok zakończenia

To by było na tyle. Moim zdaniem VIPER, choć pozostaje dobrym wyborem dla rozwoju UIKit, jest wyjątkowo kiepskim wyborem dla SwiftUI.
Ale czego użyć zamiast tego? Cóż, możesz zobaczyć moje obecne przemyślenia w SwiftUI: Wybór architektury aplikacji.
Mam jeszcze kilka przemyśleń na ten temat.
Ale to już inny artykuł. Na inny dzień.
Do tego czasu daj mi znać, co myślisz. Czy mam rację? Czy się mylę?



# SwiftUI: Wybór architektury aplikacji

Trudno jest podjąć decyzję, jeśli nie wiesz, czego potrzebujesz.

Deklaratywne podejście SwiftUI do tworzenia oprogramowania znacznie ułatwia pisanie nowoczesnych aplikacji na iOS, Maca i resztę ekosystemu Apple.

SwiftUI nie jest jednak pozbawione własnych wyzwań i pytań, a jednym z nich jest to, jakiego rodzaju architektury powinniśmy używać w naszych nowych aplikacjach SwiftUI?

Jest to często zadawane pytanie i - być może nikogo to nie dziwi - często się na nie odpowiada. W rzeczywistości istnieje wiele, wiele, wiele artykułów, książek i prezentacji na ten temat.

Niestety, wiele z nich zostało napisanych przez ludzi, którzy chcą wprowadzić swoją ulubioną architekturę do SwiftUI. I tak mamy do czynienia z przytłaczającym potokiem artykułów mówiących nam, dlaczego powinniśmy używać MVVM, React/Redux, Clean, VIPER lub TCA. Albo nawet wcale.

Przypuszczam, że jestem winny w tym względzie jak każdy, biorąc pod uwagę artykuły, które sam napisałem na ten temat.

Niemniej jednak pytanie pozostaje: Który z nich powinniśmy wybrać?

Jak podejmujemy decyzję?

Jakich kryteriów używamy, by dokonać wyboru?

Jakimi kryteriami?

Widzisz, wraz z tym ostatnim pytaniem na horyzoncie zaczyna pojawiać się promyk światła. Może, tylko może, łatwiej będzie odpowiedzieć na nasze pytanie o architekturę, jeśli poznamy odpowiedź na inne pytanie:

Jakie problemy ma rozwiązywać nasza architektura?

## Definiowanie problemu

Fizycy mają stare powiedzenie: czas jest tym, co sprawia, że wszystko nie dzieje się jednocześnie.

A ponieważ czas istnieje, ponieważ wszystko nie dzieje się od razu, my jako rasa ludzka zdecydowaliśmy się stworzyć kilka jednostek miary, aby lepiej wszystko zorganizować.

Są to minuty i sekundy. Odrębne okresy czasu, które następnie agregujemy i grupujemy w godziny i dni, miesiące i lata, i tak dalej. Wspólne jednostki, wspólne ramy, których możemy używać do omawiania czasu. Aby nim zarządzać. Aby go zrozumieć.

Podobnie, chociaż moglibyśmy napisać nasze aplikacje jako jeden ogromny blok kodu, my jako programiści zdecydowaliśmy się tego nie robić. Zamiast tego staramy się podzielić nasze aplikacje na mniejsze jednostki, które są łatwiejsze do zrozumienia. Staramy się pisać klasy, struktury i funkcje z dobrze zdefiniowanymi rolami, zachowaniami i obowiązkami. To jest widok. To jest Model. Ten kod tutaj to Usługa, która dostarcza Modele. I tak dalej.

Architektura aplikacji zasadniczo sprowadza się do zasad, których używamy do decydowania o tym, jak dzielimy nasz kod na wszystkie te poszczególne komponenty. Dlaczego ta część idzie tam, a ta tutaj.
Przez lata opracowaliśmy kilka wytycznych, które nam w tym pomagają. Pomysły takie jak *zasada pojedynczej odpowiedzialności*, zasada *otwartego-zamkniętego*, zasada *inwersji zależności* i *separacja odpowiedzialności*.

A gdyby tego było mało, dodaliśmy jeszcze kilka koncepcji wysokiego poziomu, takich jak *pojedyncze źródło prawdy, programowanie funkcjonalne i jednokierunkowy przepływ danych*.

Niektóre z tych pomysłów dobrze ze sobą współgrają. Inne mogą dodawać dodatkowej złożoności i sprawić, że nawet prosty program będzie znacznie trudniejszy do napisania, zrozumienia i utrzymania. A niektóre z nich są niczym więcej, jak tylko pełnymi dobrych intencji próbami nałożenia na Swift i iOS ograniczeń i problemów występujących w innych językach i na innych platformach.
Architektura aplikacji - każda architektura aplikacji - próbuje skodyfikować i zrównoważyć wszystkie te zasady i wytyczne w zestaw najlepszych praktyk, których możemy używać i przestrzegać.
A jeśli odniesiemy sukces w naszych wysiłkach, jeśli dobrze wybierzemy nasze komponenty i naszą architekturę, otrzymamy funkcjonalny, elegancki i dobrze zrozumiany kawałek oprogramowania.
Jeśli więc mielibyśmy zdefiniować idealną architekturę dla SwiftUI... czego byśmy od niej oczekiwali?

## Wydajność i kompatybilność

Gdyby ktoś zadał mi to pytanie, jednym z moich pierwszych kryteriów byłoby to, że musi dobrze współpracować z samym SwiftUI.

Oprócz definiowania naszych układów za pomocą prostego, deklaratywnego interfejsu, SwiftUI wprowadziło jeszcze jedną ważną koncepcję: Stan i ideę pojedynczego źródła prawdy dla fragmentu danych.

Zaktualizuj ten stan, a program automatycznie wyświetli zmianę, rekonfigurując i animując się w tym procesie.

Niektórzy ludzie doprowadzają jednak pojedyncze źródło prawdy do skrajności. Jeśli powinno istnieć jedno źródło prawdy dla danego fragmentu danych, pytają, to dlaczego nie mieć jednego źródła prawdy dla całej aplikacji?

Jest to podstawa architektur opartych na koncepcjach stojących za React/Redux i chociaż koncepcja ta wydaje się logiczna na początku, nie jest pozbawiona wad w zastosowaniu do SwiftUI.

Pisałem o tym dość obszernie w artykule Deep Inside Views, State and Performance in SwiftUI, ale najważniejsze jest to, że SwiftUI może być bardzo skuteczny w określaniu konsekwencji zmiany dowolnego fragmentu danych i przerysuje tylko tę część interfejsu, na którą ta zmiana ma wpływ.

Utworzenie pojedynczego stanu globalnego może jednak spowodować problemy z wydajnością w większych aplikacjach, w których zmiana tego stanu wymaga od SwiftUI sprawdzenia i/lub przebudowania każdej zależności w całym drzewie widoku (tj. całej aplikacji), za każdym razem, gdy nastąpi jakakolwiek zmiana.

Alternatywą, jak wskazano podczas prezentacji Apple Data Flow Through SwiftUI, jest powiązanie stanu tak nisko w hierarchii widoku, jak to możliwe.

Kiedy wiążemy się nisko w hierarchii, radykalnie minimalizujemy liczbę potrzebnych aktualizacji interfejsu i renderowania, ponieważ tylko małe części naszego drzewa widoku są dotknięte przez daną zmianę stanu.

Inną wadą dużego stanu globalnego jest to, że jeśli zaimportujesz ten stan do pojedynczego widoku, wszystko będzie widoczne dla wszystkich. W związku z tym, jak można być pewnym, do jakich informacji dany widok może uzyskiwać dostęp lub manipulować nimi bez przechodzenia przez każdą linię kodu w widoku?

Niedobrze.

Możesz napisać dodatkowy kod, aby przefiltrować stan w coś odpowiedniego dla danego widoku... ale wtedy w zasadzie piszesz więcej kodu, aby rozwiązać problem, który sam stworzyłeś.

Sytuacja, która jest nieco mniej niż optymalna.

Gorąco polecam przeczytanie artykułu Deep Inside Views, State and Performance in SwiftUI, jeśli jesteś zainteresowany tym, co dzieje się za kulisami SwiftUI, ale w międzyczasie myślę, że możemy wykorzystać powyższe kwestie, aby wykluczyć tego rodzaju architektury.

Co dalej?

## Prostota

SwiftUI oferuje proste, deklaratywne podejście do pisania aplikacji. Co więcej, jest ono zwięzłe. Można wykonać dużo pracy z bardzo małą ilością kodu.

W związku z tym szkoda byłoby obciążać siebie i nasze aplikacje zbyt formalnymi architekturami, które po raz kolejny dramatycznie zwiększają ilość kodu, który musimy napisać.

Zwłaszcza kodu standardowego. Może to tylko osobiste preferencje, ale nienawidzę szablonowego kodu. Co więcej, jestem również zdania, że więcej kodu prowadzi do większej liczby błędów, ponieważ więcej kodu daje małym frajerom więcej miejsc do ukrycia się. A ponieważ szablonowy kod jest, no cóż, szablonowy, ma tendencję do częstego kopiowania i wklejania, aby uniknąć przepisywania wszystkiego od nowa... co z kolei może prowadzić do podstępnych małych błędów kopiowania/wklejania w kodzie.

To, według mnie, zwykle wyklucza zbyt formalne architektury, takie jak VIPER, z jego naciskiem na rozbicie każdej części naszej aplikacji na widoki, interaktorów, prezenterów, jednostki i routery. Architektura VIPER jest silnie oparta na zasadzie pojedynczej odpowiedzialności z SOLID, ale z mojej perspektywy ma tendencję do przenoszenia tej idei do skrajności.

VIPER był tradycyjnie łączony z aplikacjami opartymi na UIKit, w których mieliśmy niefortunną tendencję do tworzenia pojedynczego dużego kontrolera UIViewController dla każdego ekranu w naszej aplikacji. Tworzenie odrębnych widoków i XIB oraz zagnieżdżanie kontrolerów widoku było często skomplikowaną pracą, więc zbyt często nie zawracaliśmy sobie tym głowy.

W związku z tym szukaliśmy sposobu na przeniesienie jak największej części logiki i interakcji użytkownika poza kontroler widoku... i zrobiliśmy to, każdy z nich w swoją własną, małą, indywidualną część układanki.
VIPER tworzy wiele małych ruchomych części, więc znaczny procent kodu napisanego do zarządzania aplikacją VIPER jest napisany do zarządzania samym VIPER.

Więcej na ten temat napisałem w artykule VIPER For SwiftUI? Proszę. Nie.

Ok, więc co powiesz na użycie innej architektury wspólnej dla UIKit, takiej jak MVP?

## MVP: Model-View-Presenter.

MVP to klasyczny wzorzec architektoniczny używany w wielu aplikacjach UIKit.

Tutaj większość logiki jest przenoszona z ViewController do obiektu znanego jako Presenter. Większość logiki biznesowej dla danego ekranu jest tam wykonywana, a gdy potrzebna jest zmiana, prezenter wysyła wiadomość do danego widoku, informując go o konieczności zmiany i przerysowania.

Jest to klasyczne podejście obiektowe polegające na przekazywaniu komunikatów... które ma kilka wad, gdy próbujesz użyć go ze SwiftUI.

Pierwszą i najważniejszą jest to, że w przeciwieństwie do UIKit, widok SwiftUI nie jest widokiem. To po prostu definicja naszego interfejsu. Definicja ta może wygenerować widok lub po prostu utworzyć warstwę na widoku. Może też być czymś innym.

Na przykład, widzisz HStack i VStack w kodzie i możesz myśleć, że tworzą one UIStackView... ale tak nie jest. W SwiftUI "widoki" HStack i VStack działają głównie jako instrukcje dla silnika układu. Nie są to widoki stosu.
Tak więc przekazywanie komunikatów z prezentera do określonego widoku nie jest możliwe. Docelowy "obiekt" nie jest tak naprawdę obiektem, prezenter nie może przechowywać odniesienia do niego, a cały schemat szybko się rozpada.

Nie. MVP też nie jest dobrym rozwiązaniem.

Więc co jest?

## Kompozycja widoków

Jeśli jedna rzecz była powtarzana w kółko podczas różnych sesji SwiftUI na WWDC, to to, że widoki SwiftUI są niezwykle lekkie, a ich tworzenie wiąże się z niewielkim lub żadnym spadkiem wydajności.

Tak więc w SwiftUI tworzenie tak wielu odrębnych i specjalnych widoków, jakich może wymagać aplikacja, jest bardzo korzystne.

Pisałem obszernie na temat tej koncepcji, w Najlepsze praktyki w kompozycji SwiftUI 

https://betterprogramming.pub/best-practices-in-swiftui-composition-282b02772a24

i ponownie w Kompozycja widoku w SwiftUI, 

https://medium.com/swlh/structural-decomposition-in-swiftui-8892e512b18e

więc jeśli chcesz dowiedzieć się więcej, sugeruję dodanie tych artykułów do listy lektur.

Kluczowym wnioskiem jest jednak to, że jeśli budujemy naszą aplikację z mniejszymi widokami i komponentami widoków, to po prostu nie potrzebujemy dodatkowej złożoności generowanej zwykle przez rozwiązania takie jak VIPER, a nawet MVP.

## Testowanie

Możliwe jest jednak zbytnie wychylenie wahadła i podjęcie decyzji w oparciu o wszystko, co zostało powiedziane do tej pory, że w ogóle nie potrzebujemy żadnej architektury. Wystarczy upchnąć cały nasz kod w kilka małych widoków i mieć to z głowy.

Rozważmy następujący widok.

```swift
struct OrderDetailsRowView: View {
    var item: OrderItem
    var body: some View {
        HStack {
            if item.quantity == 1 {
                Text(item.name)
            } else {
                Text("\(item.name) $(\(item.quantity, specifier: "%.2f") @ $\(item.price, specifier: "%.2f")")
            }
            Spacer()
            Text("$(\(item.total, specifier: "%.2f")")
        }
    }
}
```

To świetny przykład małego, dedykowanego widoku w SwiftUI. Cóż, być może świetny, z wyjątkiem faktu, że cała logika i formatowanie są upchnięte wewnątrz treści widoku.

Co sprawia, że niezwykle trudno jest nam napisać przypadki testowe, które zapewnią, że dane wyjściowe tego widoku są poprawne.

## Separacja odpowiedzialności

Wiodącą architekturą często proponowaną jest MVVM (Model-View-View Model).

To powiedziawszy, tak naprawdę powinno być napisane jako Model - Model Widoku - Widok (MVMV), ponieważ Model Widoku istnieje po to, aby pośredniczyć między danymi aplikacji (model) a wymaganiami widoku (układ).

Podczas korzystania z modelu widoku chcemy przenieść jak najwięcej logiki poza widok, pozostawiając kod, który jest bardzo prosty do zrozumienia.

To dobry przykład separacji zagadnień. Umieszczamy naszą logikę biznesową po jednej stronie ogrodzenia, a całą naszą prezentację i układ widoku po drugiej.

Można to lepiej zilustrować na przykładzie, więc rozważmy inny widok SwiftUI, który jest rodzicem naszego wcześniejszego widoku `OrderDetailsRowView`:

```swift
struct OrderDetailsView: View {
  @StateObject var vm = OrderDetailsViewModel()
  var body: some View {
     Form {
        if vm.message.hasMessage {
            StatusMessageView(type: vm.message)
        }        
        LabelValueRowView(label: "Order", value: vm.dateValue)
          
        ForEach(vm.items) { item in
            OrderDetailsRowView(item: item)
        }        
        if vm.hasDiscount {
            LabelValueRowView(label: "Subtotal", value: vm.subtotal)
            OrderDetailsDiscountView(value: vm.discount)
        }        
        LabelValueRowView(label: vml.totalLabel, value: vm.total)        
        Button("Order Again") {
            self.vm.reorder()
        }
     }
     .onAppear {
        vm.load()
     }
   }
}
```

Zauważ, że wszystko w tym widoku jest sterowane przez model widoku.
Wszystkie wartości warunkowe i obliczane pochodzą z modelu widoku. Istnieje kilka instrukcji if kontrolujących widoczność niektórych elementów, ale logika stojąca za tymi decyzjami jest podejmowana w modelu. Widok po prostu je wykonuje.

Gdy stan ulegnie zmianie, powiedzmy przez naciśnięcie przycisku "Zamów ponownie", widok ponownie zregeneruje się w oparciu o model widoku

Tak więc, jeśli przetestujemy nasz model widoku i zobaczymy pożądane dane wyjściowe, a nasz widok jest poprawnie powiązany z naszym modelem widoku, możemy z dość dużym stopniem pewności stwierdzić, że nasz ekran - i nasz kod - jest poprawny.

## Testowanie dedykowanych widoków

Chociaż osobiście uważam, że MVVM dobrze nadaje się do SwiftUI, możliwe jest, że w niektórych przypadkach nawet to jest przesadą. Nie każdy widok potrzebuje odrębnego modelu widoku.

Możemy na przykład refaktoryzować nasz oryginalny widok wiersza szczegółów, aby wyglądał jak poniżej.

```swift
struct OrderDetailsRowView: View {
    var item: OrderItem
    var body: some View {
        HStack {
            Text(itemDescription)
            Spacer()
            Text(itemTotal)
        }
    }
    var itemDescription: String {
        if item.quantity == 1 {
            return item.name
        } else {
            return "\(item.name) (\(item.formattedQuantity) @ \(item.formattedPrice))"
        }
    }
    var itemTotal: String {
        item.formattedTotal
    }
}
```

Biorąc pod uwagę tę treść widoku, łatwo jest zrozumieć, że wyświetlamy dwa elementy danych i po prostu niewiele może pójść źle z perspektywy logiki biznesowej.
A dzięki logice warunkowej i formatowaniu podzielonym na odrębne zmienne możliwe jest utworzenie instancji samego widoku z jednym elementem i przetestowanie naszej logiki, aby sprawdzić, czy jest poprawna, a następnie utworzenie kolejnego z dwoma elementami i przetestowanie go.

```swift
func testOrderDetailsRowView() {
    let view1 = OrderDetailsRowView(item: OrderItem.mock1)
    XCTAssert(view1.itemDescription == "Soft Drink")
    XCTAssert(view1.itemTotal == "$1.99")
    let view2 = OrderDetailsRowView(item: OrderItem.mock2)
    XCTAssert(view2.itemDescription == "Cheeseburger (2 @ $4.99)")
    XCTAssert(view2.itemTotal == "$9.98")
}
```

Ponownie, musimy po prostu przenieść jak najwięcej logiki poza ciało widoku i umieścić ją w zmiennych i funkcjach, które możemy zobaczyć spoza widoku.

Ponieważ, parafrazując wojskowy aksjomat: Jeśli możemy to zobaczyć, możemy to przetestować.
To powiedziawszy, jeśli dany widok zacząłby stawać się zbyt duży, zacząłbym zastanawiać się, jak albo podzielić go na mniejsze widoki, albo zacząłbym przenosić mój kod warunkowy i formatowanie do dedykowanego modelu widoku.

A jeśli dany widok musiałby obsługiwać żądania API, obsługiwać błędy i komunikaty o błędach, zarządzać przypadkami brzegowymi, takimi jak puste listy i tak dalej, to zdecydowanie przeniósłbym się do dedykowanego modelu widoku.

Więcej informacji na temat prawidłowego konfigurowania maszyn wirtualnych z logiką żądań sieciowych można znaleźć w artykule: Używanie protokołów modelu widoku w SwiftUI? Robisz to źle.

https://betterprogramming.pub/swiftui-view-models-are-not-protocols-8c415c0325b1

## Kryteria wyboru architektury SwiftUI

Podsumowując, moje kryteria wyboru architektury SwiftUI są następujące...

1. Musi być wydajna, bez względu na rozmiar aplikacji.
2. Musi być kompatybilna z zachowaniem i zarządzaniem stanem SwiftUI.
3. Powinna być zwięzła, lekka, adaptowalna i elastyczna.
4. Wspiera kompozycję widoków SwiftUI.
5. Wspiera testowanie.

Innymi słowy, pozwala SwiftUI być SwiftUI.

## Podsumowanie

Może się wydawać, że wziąłem cię za rękę i poprowadziłem ścieżką, abyś stanął przed ołtarzem MVVM (i żeby być uczciwym, dokładnie to zrobiłem). Ale przynajmniej wiemy teraz, dlaczego tam jesteśmy, w oparciu o wybory, których dokonaliśmy.
Czy zgadzasz się z moimi kryteriami? Masz jakieś własne? Zgadzasz się z moimi wnioskami, a może coś przeoczyłem? Tak czy inaczej, chciałbym wiedzieć, więc daj mi znać w sekcji komentarzy poniżej.



# SwiftUI: Zrozumienie programowania deklaratywnego

Zdezorientowany programowaniem deklaratywnym? Nie martw się, nie jest to takie trudne do zrozumienia.

Według Apple, SwiftUI to niesamowity deklaratywny framework programistyczny do tworzenia interfejsów użytkownika na iOS i innych platformach Apple. Tak jest napisane na pudełku.

Ale co oznacza "deklaratywny"?

Cóż, moglibyśmy zacząć od debaty na temat programowania deklaratywnego vs. programowania imperatywnego, ale to po prostu kopie wiadro dalej, ponieważ musimy teraz zdefiniować termin "imperatywny".

Zamiast tego wolałbym po prostu zastąpić słowo w moim oryginalnym zdaniu.

SwiftUI to niesamowite funkcjonalne środowisko programistyczne do tworzenia interfejsów użytkownika.

Przyjrzyjmy się więc tej definicji i zobaczmy, dokąd nas zaprowadzi.

## Programowanie funkcyjne
Niektórzy ludzie widzą słowa "programowanie funkcyjne" i zaczynają się martwić.

Dzieje się tak głównie dlatego, że zbyt wielu programistów funkcyjnych zaczyna rzucać frazami takimi jak czyste funkcje lub terminami takimi jak monady i lambdy, a następnie zaczyna sięgać po tablice, aby opisać matematykę stojącą za tym wszystkim.... co powoduje, że oczy zaczynają się szklić i zaczynasz szukać najbliższego wyjścia.

Nie martw się, nie będziemy tego tutaj robić.

W końcu wiemy już, czym są funkcje. Funkcje to w zasadzie hermetyzowane bloki kodu, które przyjmują wartości i zwracają wyniki. Gotowe.

W SwiftUI funkcje te zwracają elementy i części, które definiują nasz interfejs użytkownika. Innymi słowy, używamy tych funkcji do deklarowania naszego interfejsu.

Zaczynasz rozumieć?

## Widoki
W SwiftUI większość elementów i części składających się na nasz interfejs nazywana jest widokami. Jest jakiś tekst? To jest widok. Obraz? Kolejny widok. Czy jest to lista? Zgadłeś. To widok, który jest listą widoków.
Ale w przeciwieństwie do mocno przeciążonych UIViews w Objective-C, widoki SwiftUI są tak naprawdę tylko małymi fragmentami informacji, które połączone razem opisują nasz interfejs.
A jak je ze sobą połączyć?

## Zagnieżdżone funkcje

Innym aspektem programowania funkcyjnego jest to, że zazwyczaj zagnieżdżamy funkcje wewnątrz funkcji. To samo robimy w SwiftUI.

Nie odchodź jeszcze!

Może się to wydawać dziwne, ale jeśli się nad tym zastanowisz, jesteś już przyzwyczajony do myślenia w tych kategoriach. Nasz UINavigationController zawiera UITableView, którego listy zawierają UITableViewCells, które mogą zawierać UIStackViews, które zawierają UILabels i UIButtons.

Jeden element interfejsu zawiera inne elementy, które zawierają inne elementy, aż do voila! Opisaliśmy nasz interfejs i przechodzimy do następnego ekranu, gdzie robimy to samo.

W poniższym przykładzie SwiftUI interfejs zwracany przez ciało widoku sprowadza się do NavigationView zawierającego listę, której elementami są widoki HStack, które z kolei zawierają obraz i tekst.



```swift
struct ListView : View {
    var list = ["A", "B", "C"]
    var body: some View {
        NavigationView {
            List(list.identified(by: \.self)) { item in
                HStack {
                    Image(systemName: "circle")
                    Text(item)
                    Spacer()
                }
            }
            .navigationBarTitle(Text("Sample List"))
        }
    }
}
```

Składnia może wyglądać nieco dziwnie, ale jest to wspierane przez kilka nowych funkcji w Swift 5.1.
Boom. Właśnie zadeklarowaliśmy nasz pierwszy w pełni funkcjonalny interfejs SwiftUI w zaledwie 15 linijkach kodu.

## Modyfikatory

"Zaczekaj, sportowcu!" mówisz. "Nie tak szybko. Co jeśli chcę, aby mój tekst był czerwony? I potrzebuję tam tytułu, a nie tekstu głównego i...".

Rozumiem. Chcesz zmodyfikować domyślny wygląd i styl.

Więc jak to zrobić?

Modyfikatory.

Tak, to ja je skonfigurowałem. Ale to termin używany przez Apple i pasuje, więc będziemy się go trzymać.
Z koncepcyjnego punktu widzenia modyfikator jest operatorem na widoku... który zwraca inny widok...., który można modyfikować w nieskończoność (lub do wyczerpania pamięci 😉).

Z praktycznego punktu widzenia wszystko, co robisz, to używanie widoków i operatorów SwiftUI do opisania interfejsu SwiftUI, w prawie dokładnie takich samych warunkach, jakich użyłbyś do opisania interfejsu innemu programiście.

"Chcę, aby mój tekst stopki znajdował się w tym miejscu, użyj czcionki i rozmiaru przypisu i ustaw go jako kolor drugorzędny, aby działał w trybie ciemnym".

To jeden element interfejsu z dwoma modyfikatorami.

```swift
Text("Your mileage my vary.")
    .font(.footnote)
    .foregroundColor(.secondary)
```

Opisz powyższy interfejs do SwiftUI, a on zrobi resztę, układając tekst z wypełnieniem i odstępami odpowiednimi do rozmiaru czcionki i urządzenia, na którym działa aplikacja.

Automatycznie dba o dynamiczny tekst i dostępność rozmiarów czcionek. Automatycznie zmienia kolor, aby wyglądał poprawnie zarówno w trybie jasnym, jak i ciemnym. Obsługuje internacjonalizację i kierunek tekstu.

Krótko mówiąc, robi wszystko, co w jej mocy, aby zarządzać drobnymi szczegółami i robi to na każdym elemencie i części interfejsu. Dzięki temu programista może skupić się na funkcjach aplikacji, logice biznesowej i danych.

Skoro już o tym mowa...

## Co z naszymi danymi?
Większość aplikacji zawiera dane. Czasami dane są wbudowane w aplikację, jak pokazano w naszym powyższym przykładzie zrzeczenia się odpowiedzialności, ale zazwyczaj nasze dane pochodzą z zewnętrznego źródła, bazy danych, interfejsu API, a czasem nawet z informacji wprowadzonych bezpośrednio do aplikacji.

Kiedy definiujemy element interfejsu użytkownika w SwiftUI, mówimy mu również, gdzie znaleźć potrzebne dane. Czasami informacje te są statyczne, jak ciąg znaków w widoku tekstowym pokazanym powyżej.
Czasami informacje te są generowane przez nasze dane. W poniższym przykładzie zmodyfikowaliśmy nasz widok listy, aby korzystał z danych przekazywanych do niego podczas inicjalizacji w innym miejscu aplikacji.

```swift
struct ListView : View {
    var list: [String]
    var body: some View {
        NavigationView {
            List(list.identified(by: \.self)) { item in
                HStack {
                    Image(systemName: "circle")
                    Text(item)
                    Spacer()
                }
            }
            .navigationBarTitle(Text("Sample List"))
        }
    }
}
```

Pierwszy parametr listy mówi jej dokładnie, gdzie ma znaleźć swoje dane.

List następnie pobierze każdy element z tablicy i przekaże go do zamknięcia, które z kolei użyje go do zbudowania i zwrócenia widoku wyświetlającego ten element.

Wszystko to dzieje się bez żadnych źródeł danych ani delegatów. Powyższy kod może być kompletnym, działającym ekranem w aplikacji.

## Dynamiczne dane
Czasami nasze informacje są dynamiczne, podlegają zmianom, a nasz interfejs powinien się odpowiednio dostosować.

```swift
struct ListView: View {
    @Binding var list: [String]
    @Binding var title: String
    var body: some View {
        NavigationView {
            List(list.identified(by: \.self)) { item in
                HStack {
                    Image(systemName: "circle")
                    Text(item)
                    Spacer()
                }
            }
            .navigationBarTitle(Text(title))
        }
    }
```

Powyższa implementacja jest praktycznie identyczna z pierwszą, ale z główną różnicą - właściwością @Binding dodaną do naszych zmiennych listy i tytułu.

Binding mówi SwiftUI, aby obserwował zmiany w tablicy list. Jeśli jej właściciel zaktualizuje zawartość tablicy, powiadomi nas o tym, a nasz widok listy zaktualizuje się i wyrenderuje - automatycznie.

Warto zauważyć, że List jest niezwykle inteligentnym widokiem. Jeśli właściciel listy wstawi element i usunie inny, po otrzymaniu informacji o zmianie List przeskanuje zaktualizowaną listę, porówna ją z poprzednią listą, a następnie wygeneruje odpowiednie animacje wstawiania i usuwania.

Zwróć też uwagę, że teraz przekazujemy również tytuł, który chcemy umieścić na pasku nawigacyjnym. Zmienna title jest również powiązana, więc za każdym razem, gdy ta część danych ulegnie zmianie, nasz pasek tytułu zostanie automatycznie zaktualizowany.

Powiązania to tylko jeden ze sposobów zarządzania i utrzymywania stanu aplikacji w SwiftUI. Pełne omówienie wszystkich z nich wykracza poza zakres tego artykułu, więc bądź na bieżąco.

## Środowisko
Dodatkowym aspektem, o którym należy wspomnieć, jest środowisko.
W SwiftUI środowisko to globalny zestaw zmiennych, które opisują środowisko, w którym działa aplikacja. Na przykład, czy jesteśmy w trybie jasnym czy ciemnym? Jaka jest bieżąca klasa rozmiaru pionowego? Pozioma?
Zmienne te mogą być sprawdzane w celu określenia aktualnego stanu urządzenia i aplikacji, dzięki czemu można zrobić właściwą rzecz we właściwym czasie.
Możesz także dodać własne informacje do środowiska i uzyskać do nich dostęp później, w dalszej hierarchii widoku.

```swift
@EnvironmentObject var settings: UserSettings
@Environment(\.colorScheme) var colorScheme: ColorScheme
```

Ponownie, wykracza to nieco poza zakres tego artykułu, ale musisz wiedzieć, że tam jest.

## Pod maską
Deklarujesz swój interfejs i wskazujesz SwiftUI swoje dane... a SwiftUI robi resztę.

Każdy widok, stan jego modyfikatorów, jego dane i aktualny stan środowiska są łączone i agresywnie redukowane do zestawu poleceń układu i renderowania. Wynik jest następnie prezentowany użytkownikowi.

Następnie SwiftUI czeka na zmiany danych. W przypadku ich wystąpienia, SwiftUI przechodzi przez dotknięte widoki i buduje nowe drzewo układu, porównuje wszelkie znalezione zmiany z bieżącym drzewem, a następnie renderuje te części widoku, które wymagają aktualizacji.

Sprawia to, że SwiftUI jest niesamowicie szybki w porównaniu do UIKit. W rzeczywistości większość renderowania widoków i animacji jest wykonywana bezpośrednio przez Metal.

## Animacje?
Tak, tworzenie animacji w SwiftUI jest bardzo proste. W rzeczywistości określanie, jakie animacje mogą być potrzebne do przejścia stanu widoku, jest wbudowane bezpośrednio w proces porównywania drzew.

Niektóre animacje, takie jak wstawianie i usuwanie list oraz te odnoszące się do zmian stanu modyfikatorów widoku, są wbudowane bezpośrednio w system.

Wszystkie one mogą być modyfikowane i dostosowywane w razie potrzeby.

## Blok zakończenia
To by było na tyle. Mam nadzieję, że teraz rozumiesz, co Apple ma na myśli, gdy mówi, że SwiftUI to nowy, deklaratywny framework programistyczny do konstruowania interfejsów użytkownika na iOS i innych platformach Apple.

I że ten niesamowity modyfikator [sic] jest w pełni zasłużony.

Jak zawsze, pozostaw wszelkie pytania lub komentarze poniżej, a ja dołożę wszelkich starań, aby na nie odpowiedzieć, bezpośrednio lub w przyszłym artykule.

# Najlepsze praktyki w kompozycji SwiftUI
Kilka przemyśleń na temat kompozycji widoków SwiftUI, czytelności kodu i wydajności aplikacji

SwiftUI zmieni sposób tworzenia przyszłych aplikacji dla systemów iOS, iPadOS, macOS, tvOS i watchOS.
Jednak pełny wpływ SwiftUI nie polega tylko na wyeliminowaniu UIKit i zamianie widoków na UIViews lub list na UITableViews, ani nawet na całkowitym wyeliminowaniu potrzeby stosowania UIConstraints i kotwic UIView.
Nie chodzi też o radykalne uproszczenie naszych aplikacji poprzez wyeliminowanie Storyboardów, IBOutlets, IBActions, Segues i wszystkich innych powiązanych boilerplate'ów, które tak długo ograniczały nasze aplikacje. Nie wspominając już o wyeliminowaniu błędów, które często czają się głęboko w kodzie z powodu przypadkowego odłączenia outletów i akcji.
Nie zrozum mnie źle. SwiftUI da nam wszystkie te korzyści. I jeszcze więcej. Ale z mojej perspektywy główny wpływ SwiftUI nie będzie dotyczył tego, jak budujemy interfejsy naszych aplikacji... ale tego, jak projektujemy nasze aplikacje.
ale w tym, jak projektujemy nasze aplikacje.

## Najlepsze praktyki dotyczące widoku SwiftUI
Mam wiele do powiedzenia na temat modeli widoku i zarządzania stanem widoku. Więcej niż wystarczająco na kilka artykułów.

Dziś jednak skupię się na tym, co uważam za najlepsze praktyki w kodowaniu interfejsów użytkownika, widoków i hierarchii widoków.

- Kompozycja widoku
- Skup się na funkcjonalności, a nie na wyglądzie
- Używaj kolorów semantycznych
- Architekt z myślą o innych platformach
- Pozwól systemowi robić swoje
- Powiąż stan tak nisko w hierarchii, jak to możliwe.

Gotowy? Zaczynajmy...

### Kompozycja widoków
Jeśli jedna rzecz była powtarzana w kółko podczas sesji SwiftUI na WWDC, to fakt, że widoki SwiftUI są niezwykle lekkie, a ich tworzenie wiąże się z niewielkim lub zerowym spadkiem wydajności.

W przeciwieństwie do UIViews w UIKit, większość widoków SwiftUI istnieje jako struktury Swift i jest tworzona, przekazywana i przywoływana jako parametry wartości. Chociaż może to mieć pewne niepożądane konsekwencje w tym momencie, użycie struktur pozwala uniknąć mnóstwa alokacji pamięci i tworzenia wielu mocno podklasowanych i dynamicznych UIView opartych na UIKit.

Ponadto, w przeciwieństwie do UIView, parametry i modyfikatory w zagnieżdżonym widoku SwiftUI są łączone w jedną całość podczas cykli układu i wyświetlania. Co więcej, węzły w drzewie widoku są monitorowane pod kątem zmian stanu i - jeśli pozostają niezmienione - zwykle nie wymagają ponownego renderowania.

Z drugiej strony, każdy UIView jest alokowany i istnieje jako połączony podwidok gdzieś na drzewie renderowania układu i jest aktywną częścią procesu układu.

Wszystko to oznacza, że w SwiftUI korzystne jest tworzenie tylu odrębnych i specjalnych widoków, ile może wymagać aplikacja.

```swift
struct FootnoteText : View {
    let text: String
    var body: some View {
        MultiLineText(text: text, alignment: .center)
            .font(.footnote)
    }
}
struct MultiLineText: View {
    var text: String = ""
    var alignment: HAlignment = .leading
    var body: some View {
        Text(text)
            .lineLimit(nil)
            .multilineTextAlignment(alignment)
    }
}
```

W powyższym przykładzie widok MultiLineText byłby prawdopodobnie często używany w całej aplikacji. Widok Footnote to po prostu wyspecjalizowany MultiLineText z dołączonym określonym modyfikatorem czcionki.
Tak więc w całej aplikacji można po prostu odwoływać się do...

```swift
FootnoteText(text: $model.disclamer)
```

...w razie potrzeby, w przeciwieństwie do rozpraszania następujących elementów w kodzie:

```swift
Text($model.disclamer)
    .lineLimit(nil)
    .multilineTextAlignment(.center)
    .font(.footnote)
```

Prawidłowo nazwany widok, taki jak FootnoteText, bardziej formalnie ogłasza twoje zamiary każdemu, kto później przyjdzie i przeczyta twój kod.

Małe, dobrze zamknięte widoki są również łatwiejsze do zrozumienia i są o wiele mniej podatne na błędy i niezamierzone efekty uboczne.

Weźmy przykład z podręcznika Smalltalk, gdzie wiele funkcji pozornie składa się z linijki lub dwóch kodu przed delegowaniem funkcjonalności do innej funkcji... która z kolei robi to samo.

Podstawową filozofią projektowania UIKit jest dziedziczenie.

SwiftUI to kompozycja.

## Skup się na funkcjonalności, a nie wyglądzie

Pop quiz, w poniższym kodzie, jakiego koloru jest tekst?

```swift
Text($model.disclamer)
    .foregroundColor(.red)
    .foregroundColor(.green)
```

(Dźwięk muzyki Jeopardy w tle...)
Odpowiedź? Tekst ma kolor .red. W tym przypadku najbliższa wartość jest powiązana z widokiem.

Co to oznacza? Cóż, można pokusić się o określenie tekstu przypisu w pierwszym przykładzie jako:

```swift
struct FootnoteText : View {
    let text: String
    var body: some View {
        MultiLineText(text: text, alignment: .center)
            .foregroundColor(.gray)
            .font(.footnote)
    }
}
```

Tam, gdzie określiłeś kolor jako szary, ponieważ jest to kolor, który chciałeś do tej pory. Ale problem polega na tym, że teraz utknąłeś z tym i próbujesz...

```swift
FootnoteText(text: $model.disclamer)
    .foregroundColor(.red)
```

Nadal otrzymujemy szary tekst. Nasz przykład obfituje teraz w inne problemy, takie jak prawdopodobnie spieprzyliśmy automatyczną adaptację trybu ciemnego, określając konkretną wartość pojedynczego koloru tekstu, a także prawdopodobnie spieprzyliśmy możliwość korzystania z naszego widoku FootnoteText na innych platformach z różnymi wyglądami i schematami kolorów.

Wstrzymaj się więc z czysto wizualnymi modyfikatorami wyglądu w komponentach widoku. Zwłaszcza kolor.

### Używaj kolorów semantycznych

Jeśli jednak musisz ustawić kolory, zdecydowanie rozważ użycie kolorów semantycznych.
Color.primary, `Color.secondary` i `Color.accentColor` to tylko przykłady kolorów dostarczanych przez system i środowisko. Nawet kolor taki jak .orange może i będzie prawidłowo dostosowywać się do trybu jasnego i ciemnego, zmieniając się nieznacznie w trakcie tego procesu.
Możesz także zdefiniować własne kolory semantyczne w Xcode. Podobnie jak Apple, można je nawet dostosować do trybu jasnego i ciemnego.

![img](/Users/uta/Desktop/Tlumaczenia%20ksiazek%20i%20artykulow/PointFree/1*dqEjrMe5UkgHhINEgMR6qw.png)

Zaletą tego podejścia jest to, że można łatwo zmienić schematy kolorów i branding całej aplikacji w jednym miejscu. Można nawet modyfikować schematy dla każdej platformy bez zmiany kodu, po prostu dostarczając inny zestaw kolorów w katalogu xcassets tej platformy.

Skoro już o tym mowa...

### Projektuj z myślą o innych platformach
Dzięki SwiftUI łatwiej niż kiedykolwiek jest używać tego samego kodu na różnych platformach. Do pewnego stopnia można było to zrobić wcześniej, przenosząc modele, kod API i logikę biznesową z platformy na platformę, ale teraz jest całkowicie możliwe wykorzystanie wielu elementów interfejsu użytkownika również na różnych platformach.

Było to wyraźnie widoczne w prezentacji Apple SwiftUI na wszystkich urządzeniach podczas WWDC.
Nie będę zagłębiał się w ten temat, ponieważ prezentacja omówiła go tak dobrze, ale należy pamiętać, że wiele z międzyplatformowego udostępniania interfejsu użytkownika między aplikacjami na iOS i iPadOS oraz aplikacjami na macOS i tvOS pochodzi z podłączania tych samych widoków treści, które utworzyłeś na jednej platformie, do innej struktury nawigacji na innej platformie.

Ponownie należy więc rozważyć, gdzie można określić takie elementy, jak czcionki, kolory itp. SwiftUI ma kilka mechanizmów dostarczania lub opóźniania tych specyfikacji.

Powyżej omówiliśmy semantyczne kolory, ale oto kolejny przykład.

```swift
Group {
    MyCustomTextField($model.username)
    MyCustomTextField($model.password)
    }
    .font(.headline)
    .background(Color.white.opacity(0.5))
    .relativeWidth(1)
```

Grupa to potężne narzędzie, które pozwala grupować elementy, którymi należy manipulować razem lub, w tym przypadku, które mają wspólne atrybuty.

W tym przykładzie modyfikatory grupy są stosowane do każdego pola MyCustomTextField, w związku z czym każde z nich otrzyma czcionkę nagłówka, kolor tła i zostanie dopasowane do pełnej szerokości kontenera nadrzędnego.

MyCustomTextField zajmuje się funkcjonalnością. Niech kontekst zajmie się stylem.

## Niech system zrobi swoje
Należy również pamiętać, że SwiftUI automatycznie tłumaczy widoki na elementy interfejsu wizualnego odpowiednie dla danej platformy. Na przykład widok Toogle renderuje się inaczej - i poprawnie - na iOS, macOS, tvOS i watchOS.

![img](/Users/uta/Desktop/Tlumaczenia%20ksiazek%20i%20artykulow/PointFree/1*TLLGVjDqGimtjmidd_wKnA.png)

Co więcej, SwiftUI odpowiednio dostosuje kolory, odstępy, wypełnienie i tym podobne w oparciu o platformę, bieżący rozmiar ekranu i / lub kontenera, stan sterowania. Weźmie również pod uwagę wszelkie zmiany wymagane dla funkcji ułatwień dostępu, które mogą być włączone, zrobi właściwą rzecz dla trybu Light/Dark i nie tylko.

Aby wybrać losowo jeden przykład, wypełnianie tekstu akapitowego wygląda w jeden sposób na iOS, ale zwykle ma więcej miejsca na ekranie iPada... chyba że zawartość jest teraz skompresowana przez pojawienie się w widoku Slide-Over. Wiele kwestii, które, szczerze mówiąc, nie zawsze bierzemy pod uwagę.
W UIKit prawdopodobnie po prostu podłączyliśmy wartość 15 do pola ograniczenia na storyboardzie i przeszliśmy dalej.

Oznacza to dla nas tu i teraz, że musimy się zrelaksować i pozwolić systemowi robić swoje. Unikaj obsesyjnego próbowania dopasowania idealnych pod względem pikseli układów projektowych, często tworzonych dla pojedynczego "najlepszego przypadku" układu na określonym rozmiarze ekranu dla domyślnego trybu dostępności.

Weź przykład z twórców stron internetowych, którzy przeszli od bardzo sztywnych układów projektowych do wysoce responsywnych, dobrze dostosowanych do każdej platformy i potrzeb indywidualnego użytkownika.

Może to oznaczać kilka rozmów z zespołem projektowym UI/UX.
Apple zadało sobie wiele trudu, aby upewnić się, że SwiftUI robi właściwe rzeczy we właściwym czasie, tak długo, jak długo nie szturchamy go łokciem. Więc nie rób tego.
I chcę być tutaj jasny. Nie mówię, że ty też nie możesz. Apple dało nam dużą kontrolę nad prezentacją. Musimy tylko wiedzieć, kiedy odpuścić i zachować specjalne przypadki dla bardzo szczególnych przypadków.

Pozwól systemowi robić swoje, a nie tylko napiszesz mniej kodu, ale jestem gotów się założyć, że będziesz miał również mniej błędów.



## Wiązanie stanu tak nisko w hierarchii, jak to możliwe

Jak wspomniałem wcześniej, zamierzam zachować większość moich przemyśleń na temat modeli widoku i zarządzania stanem na późniejsze artykuły, ale ta koncepcja pasuje do przykładów pokazanych do tej pory, więc zakończę tym.
Zwróć uwagę na nasz wierny widok FootnoteText w poniższym kodzie.

```swift
struct MyMainView : View {
    @ObservedObject var model: MainViewModel
    @EnvironmentObject var settings: UserSettings
    var body: some View {
        VStack {
            MainContentView(model)
            MainContentButtons(model)
            FootnoteText(text: settings.fullVersionString)
        }
    }
}
```

Zauważ tutaj, że `MyMainView` automatycznie importuje obiekt środowiska o nazwie settings i przekazuje settings.fullVersionString do naszego widoku `FootnoteText`.
Wszystko dobrze... ale dlaczego `MyMainView` w ogóle wie o `UserSettings`? Co by było, gdybyśmy zamiast tego zrobili co następuje?

```swift
struct MyMainView : View {
    @ObservedObject var model: MainViewModel
    var body: some View {
        VStack {
            MainContentView(model)
            MainContentButtons(model)
            ApplicationVersionFootnote()
        }
    }
}
```

I zdefiniowałem ApplicationVersionFootnote w innym miejscu jako...

```swift
struct ApplicationVersionFootnote : View {
    @EnvironmentObject var settings: UserSettings
    var body: some View {
        FootnoteText(text: settings.fullVersionString)
    }
}
```

Tutaj nasza zmienna środowiskowa jest pobierana i używana niżej w hierarchii widoku. MyMainView nie wie nic o UserSettings i nie powinno go to obchodzić. Tak samo jak MainViewModel.

Ten ostatni punkt jest kluczowy. Od pewnego czasu staramy się unikać syndromu Massive-View-Controller, przyjmując jakąś formę struktury modelu widoku, czy to MVVM, VIPER, czy cokolwiek innego.
Jeśli jednak nie byliśmy bardzo ostrożni, wszystko co udało nam się zrobić, to zastąpić nasze masywne kontrolery widoku masywnymi modelami widoku.

W bardziej tradycyjnej implementacji MVVM, nasz obiekt UserSettings prawdopodobnie zostałby wstrzyknięty do naszego MainViewModel, a jakaś funkcja lub zmienna lub wiązanie utworzone w celu ujawnienia fullVersionString w naszym modelu widoku. Komplikuje to nasz model widoku, komplikuje nasze strategie wstrzykiwania i kod inicjalizacyjny oraz przyczynia się do tego, że komponenty naszej aplikacji są ściślej powiązane i sztywne.

Ale w naszym ostatnim przykładzie nasz widok `ApplicationVersionFootnote` jest w rzeczywistości małym, wysoce specyficznym, specjalnym modelem widoku, który łączy dane `UserSettings` naszego środowiska z widokiem FootnoteText.

Widzę duży potencjał dla tego rodzaju rzeczy w przyszłości i bardzo pasuje to do zasady pojedynczej odpowiedzialności SOLID.

# SwiftUI Microservices

Nowoczesne podejście do architektury aplikacji w SwiftUI

SwiftUI wprowadza nowe deklaratywne, oparte na stanach i komponentach podejście do tworzenia aplikacji na iOS, macOS i wszystkie inne urządzenia Apple.
W związku z tym nadszedł czas, aby nasze podejście do architektury aplikacji również poszło do przodu.
Zanim jednak przejdziemy dalej, rzućmy okiem na historię i obecny stan techniki.

## Model View Controller
Klasyczne podejście do tworzenia aplikacji na iOS opiera się na MVC (Model View Controller). W MVC, kontroler przenosi informacje tam i z powrotem pomiędzy modelem i różnymi widokami, które reprezentują nasz interfejs.

![img](/Users/uta/Desktop/Tlumaczenia%20ksiazek%20i%20artykulow/PointFree/1*QA23oAHq1FUnZIhnjAMNWg.png)

W systemie iOS kontroler przejawiał się jako pojedynczy obiekt, UIViewController. Kontroler widoku zarządzał wszystkimi interakcjami użytkownika i zmianami stanu - w tym ładowaniem informacji, manipulowaniem i aktualizowaniem tych danych, a także obsługiwał nawigację użytkownika do i z różnych ekranów i stron w naszej aplikacji.
Takie podejście oznaczało, że kontroler miał zbyt dużą rolę do odegrania w architekturze naszej aplikacji. Jego rola była tak duża, że zaczęto nazywać go MVC (skrót od Massive View Controller).
Nie trzeba dodawać, że to podejście było nieco mniej niż optymalne.

## Model-View-ViewModel

Zaproponowano różne rozwiązania w celu zwalczania masywnych kontrolerów widoku, ale większość z nich sprowadza się do jakiejś formy modelu-widoku-widoku-modelu (MVVM).

Modele i widoki nadal istnieją, podobnie jak kontrolery widoku. Ale wnętrzności aplikacji, manipulacja danymi i logika biznesowa, zostały wyodrębnione z kontrolera widoku i przeniesione do ViewModel.

Dlaczego? Cóż, z jednej strony upraszcza to kontroler widoku, ale głównym celem wyodrębnienia całej tej logiki z kontrolera widoku jest to, że teraz można ją testować. Możemy utworzyć instancję modelu widoku i przekazać mu informacje oraz wywołać jego metody i bezpośrednio obserwować wynikające z tego zmiany stanu, które model widoku przedstawi następnie kontrolerowi widoku.

![img](/Users/uta/Desktop/Tlumaczenia%20ksiazek%20i%20artykulow/PointFree/1*XnXDtt4C7cBltVoVbgCmVA.png)

Ponieważ zadanie kontrolera widoku zostało zredukowane do prostego przekazywania tych zmian stanu do widoków, które tworzą naszą aplikację, możemy być całkiem pewni, że jeśli dane wyjściowe modelu widoku są poprawne, nasza aplikacja również będzie poprawna.

Istnieją wariacje na ten temat: model-widok-prezenter (MVP). VIPER. Clean. Ale wszystkie one opierają się na tych samych podstawowych koncepcjach i różnią się przede wszystkim sposobem podziału odpowiedzialności między zestaw komponentów.

Wszystkie jednak zgadzają się co do jednego - kontroler widoku powinien być tak głupi, jak to tylko możliwe.

## SwiftUI
Apple najwyraźniej się z tym zgadza, ponieważ po WWDC19 Apple wprowadziło SwiftUI, które między innymi całkowicie eliminuje większość kontrolerów zdefiniowanych przez użytkownika i kontrolowanych widoków.

W SwiftUI używasz prostej składni do deklarowania interfejsu użytkownika.

Co więcej, interfejs ten jest całkowicie zależny od stanu aplikacji w danym momencie. Wystarczy zmienić stan aplikacji, a interfejs aplikacji zostanie natychmiast zaktualizowany, aby odzwierciedlić te zmiany.

Apple określa tę koncepcję mianem "pojedynczego źródła prawdy".



Ale tylko dlatego, że powinno istnieć Pojedyncze Źródło Prawdy dla dowolnej części aplikacji, niekoniecznie oznacza, że powinno istnieć Pojedyncze Źródło Prawdy dla całej aplikacji.
Zdezorientowany? Pozwól, że wyjaśnię.



### Kompozycja

Jak pisałem w artykule "Kompozycja widoków w SwiftUI", Apple zachęca do dekomponowania widoków na małe, zwarte, odrębne komponenty, w których każdy widok kontroluje określoną część interfejsu użytkownika.
Przyjrzyjmy się jeszcze raz jednemu komponentowi z tego artykułu, przyciskowi ulubionych, który służy do wskazywania, że dany element powinien zostać zapamiętany i wyświetlony na liście ulubionych w aplikacji.

Kod przycisku ulubionych jest następujący:

```swift
struct FavoritesButton: View {
    let item: MenuItem
    @EnvironmentObject var favorites: FavoritesService
    var imageName: String {
        favorites.isFavorite(item) ? "star.fill" : "star"
    }
    var body: some View {
        Image(systemName: imageName)
            .foregroundColor(.accentColor)
            .scaleEffect(1.2)
            .onTapGesture {
                self.favorites.toggleFavorite(self.item)
            }
    }
}
```

Interfejs i zachowanie przycisku ulubionych są całkowicie niezależne i mogą być używane w dowolnym miejscu w dowolnym widoku w całej naszej aplikacji. Jak pokazano na zrzucie ekranu, możemy nawet upuścić go na pasku nawigacyjnym.

```swift
struct DetailView: View {
    let item: MenuItem
    var body: some View {
        ScrollView(.vertical) {
            VStack {
                ...
            }
        }
        .navigationBarTitle("Details", displayMode: .inline)
        .navigationBarItems(trailing: FavoritesButton(item: item))
    }
}
```

Dotknięcie przycisku ulubionych na pasku nawigacyjnym spowoduje oznaczenie bieżącej pozycji jako ulubionej. Lub usunięty. Niezależnie od tego, DetailView nie ma wiedzy na temat wewnętrznych szczegółów lub implementacji przycisku.

### Usługa obsługi obiektów ulubionych

Podczas gdy kod stojący za interfejsem przycisku ulubionych jest niezależny, wewnętrznie podstawowa funkcjonalność widoku zależy od FavoritesService, obiektu środowiskowego, który został zdefiniowany i wstawiony do hierarchii widoku gdzieś wyżej w łańcuchu.

FavoritesService to obiekt obserwowalny SwiftUI, który udostępnia naszemu widokowi jedną opublikowaną wartość i dwie metody. Jedną z nich jest metoda isFavorite(item), która określa, czy dany element jest już ulubionym, a drugą jest toggleFavorite(item), która odpowiednio przełącza stan elementu.

Zwróć uwagę, że gdy wywoływana jest metoda toggleFavorite(item) - z tego miejsca lub z dowolnego miejsca w aplikacji - nasza lista ulubionych elementów jest aktualizowana i w związku z tym każdy widok zależny od usługi FavoritesService zostanie poproszony o odpowiednią aktualizację prezentacji widoku.

```swift
class FavoritesService: ObservableObject {
    @Published var items: [MenuItem] = []
    func isFavorite(_ menuItem: MenuItem) -> Bool {
        items.firstIndex(where: { $0.id == menuItem.id }) != nil
    }
    func toggleFavorite(_ menuItem: MenuItem) {
        if let index = items.firstIndex(where: { $0.id == menuItem.id }) {
            items.remove(at: index)
        } else {
            items.append(menuItem)
        }
    }
}
```

FavoritesService jest pojedynczym źródłem prawdy dla tego konkretnego widoku. Może być również źródłem prawdy dla innych widoków, ale FavoritesButton nie dba o to.
FavoritesService również przestrzega zasady pojedynczej odpowiedzialności. Jej celem jest zarządzanie listą ulubionych pozycji menu i to wszystko.

### Karty aplikacji

Przyjrzyjmy się kolejnej usłudze, choć niezwykle minimalistycznej.

```swift
enum AppTabs: Int {
    case favorites
    case menu
    case order
}
class AppState: ObservableObject {
    @Published var currentTab = AppTabs.favorites
}
```

Tutaj śledzimy bieżący stan zakładki aplikacji, dzięki czemu możemy programowo przejść do określonej zakładki w razie potrzeby.

```swift
struct AppTabView: View {
    @EnvironmentObject var appState: AppState
    var body: some View {
        TabView(selection: $appState.currentTab) {
            FavoritesView()
                .tabItem {
                    Image(systemName: "star")
                    Text("Favorites")
                    }
                .tag(AppTabs.favorites)
            ...
            }
    }
}
```

### Obsługa zamówień

I jeszcze jedno. Oto usługa OrderService z tej samej aplikacji, używana do śledzenia zamówionych produktów.

```swift
class OrderService: ObservableObject {
    @Published var items = [MenuItem]()
    var total: Int {
        items.reduce(0) { $0 + $1.price }
    }
    func isInCart(_ menuItem: MenuItem) -> Bool {
        items.firstIndex(where: { $0.id == menuItem.id }) != nil
    }
    func add(item: MenuItem) {
        items.append(item)
    }
    func remove(item: MenuItem) {
        if let index = items.firstIndex(of: item) {
            items.remove(at: index)
        }
    }
}
```

### Redux
Ponieważ dla każdego komponentu aplikacji powinno istnieć jedno źródło prawdy, niektórzy zaproponowali, aby SwiftUI przeszedł na model stanów w stylu Redux, z jednym źródłem prawdy dla całej aplikacji.

```swift
class AppState: ObservableObject {
    @Published var currentTab = AppTabs.favorites
    @Published var menuItems: [MenuItem] = []
    @Published var favoriteItems: [MenuItem] = []
    @Published var orderItems: [MenuItem] = []
}
```

Lub jeśli chcesz zachować funkcjonalność między liniami komponentów, być może spróbuj zrobić coś bardziej podobnego:

```swift
class AppState: ObservableObject {
    @Published var currentTab = AppTabs.favorites
    @Published var menu = MenuService()
    @Published var favorites = FavoritesService()
    @Published var order = OrderService()
}
```

Zaimportuj globalny AppState do każdego z widoków, które potrzebują danych i gotowe.

### Zalety i wady pojedynczego stanu globalnego

Zalety pojedynczego AppState polegają głównie na prostocie. Jak wspomniano, wystarczy zaimportować jeden obiekt environmentObject.

Ale wady, według mnie, są liczne.

Przede wszystkim dotyczą one wydajności. Dokonanie pojedynczej zmiany w stanie aplikacji - powiedzmy, oznaczenie pojedynczego elementu jako ulubionego - i teraz każde pojedyncze drzewo widoku w aplikacji musi zostać przejrzane i sprawdzone pod kątem zmian. Dlaczego? Ponieważ pojedynczy obiekt środowiska, od którego zależy każdy widok, zasygnalizował aktualizację.

W przypadku mniejszych aplikacji to uderzenie może być nieistotne. Ale dla większych?

(Należy zauważyć, że jest to znany problem również w przypadku dużych aplikacji internetowych React/Redux).

### Dane globalne
Moje drugie największe zastrzeżenie dotyczy globalnej ekspozycji danych aplikacji.

Zaimportowanie AppState do pojedynczego widoku sprawia, że wszystko jest widoczne dla wszystkich. W takim przypadku, jak można być pewnym, do jakich informacji dany widok może uzyskiwać dostęp lub manipulować nimi bez przechodzenia przez każdą linię kodu w widoku?

Nasz powyższy przycisk FavoritesButton jest świetnym przykładem tego, że tak nie jest. Wystarczy spojrzeć na początek kodu, aby stwierdzić, że jedyną rzeczą, którą ten kod może zobaczyć lub zmienić, jest usługa FavoritesService, ponieważ jest to jedyny obiekt zaimportowany ze środowiska aplikacji.

Co więcej, gdybym chciał użyć FavoritesButton w innej aplikacji, łatwo jest zobaczyć, co jeszcze muszę przenieść do innej aplikacji.

https://betterprogramming.pub/swiftui-microservices-c7002228710

### Testowanie

Mój trzeci zarzut dotyczy testów. Jedną z naszych głównych motywacji w odniesieniu do łamania kodu do modeli i usług jest ułatwienie testowania tego kodu.

W SwiftUI nasza aplikacja jest całkowicie kontrolowana przez jej stan. Więc jeśli umieścimy ten stan w modelu lub usłudze i jeśli ten stan zmieni się w wyniku działań wywołanych przez użytkownika, to w testowaniu możemy uruchomić te akcje i obserwować wynikającą z tego zmianę stanu.

Jeśli stan zaktualizuje się poprawnie dla każdej możliwej zmiany lub działania, możemy mieć dość wysoki poziom pewności, że nasza aplikacja jest poprawna.

Ale umieszczenie całego naszego stanu w jednym pojemniku sprawia, że testowanie naszych poszczególnych modeli lub usług w izolacji jest o wiele, wiele trudniejsze. To działa dość dobrze dla testów integracyjnych, ale nie jest tak dobre dla testów jednostkowych.

I nawet wtedy, w stanie globalnym, potencjał posiadania niesprawdzonych, a zatem nieprzewidzianych skutków ubocznych jest niezwykle wysoki. "Och, nie zdawałem sobie sprawy, że to też zmienia tę zmienną!"

### Późne wiązanie

Jak również wskazano w artykule o kompozycjach widokowych, kolejną najlepszą praktyką SwiftUI jest wiązanie stanu tak nisko, jak to możliwe w hierarchii.

Kiedy wiążemy się nisko w hierarchii, radykalnie minimalizujemy liczbę potrzebnych aktualizacji interfejsu i renderowania, ponieważ tylko małe części naszego drzewa widoku są dotknięte jakąkolwiek aktualizacją i muszą zostać zregenerowane.

Wszystko to znacznie poprawia wydajność aplikacji.

Powyżej ulubiona usługa jest powiązana bezpośrednio z obiektem, który jej potrzebuje. DetailView, który chce wyświetlić przycisk ulubionych, nie wie ani nie przejmuje się tym. Ktoś wyżej w łańcuchu musiał to zapewnić, oczywiście, ale to inna odpowiedzialność za kogoś innego.

### Dlaczego usługi, a nie ViewModels?

Można zapytać, dlaczego nazywamy je usługami, a nie tylko ViewModels.

W tym przypadku kluczowym czynnikiem różnicującym jest to, że modele widoków są zwykle pisane w celu prowadzenia pojedynczego ekranu, strony lub widoku, a ten widok jest właścicielem modelu widoku.

Z drugiej strony usługi są tworzone i współdzielone między wieloma widokami i komponentami w naszej aplikacji SwiftUI poprzez wstrzykiwanie ich do środowiska aplikacji na pewnym poziomie hierarchii widoków do użytku przez elementy na niższym poziomie. Utrzymują się tak długo, jak długo ten poziom się utrzymuje.

W rzeczywistości wiele usług jest zwykle tworzonych i wstrzykiwanych na sam szczyt hierarchii widoku, gdy nasz początkowy widok treści jest tworzony w naszym SceneDelegate.

```
func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {    // Create the SwiftUI view that provides the window contents.
    let contentView = AppTabView()
        .environmentObject(AppState())
        .environmentObject(MenuService())
        .environmentObject(MessageService())
        .environmentObject(FavoritesService())
        .environmentObject(OrderService())
        .environmentObject(RatingsService())    // Use a UIHostingController as window root view controller.
    if let windowScene = scene as? UIWindowScene {
        let window = UIWindow(windowScene: windowScene)
        window.rootViewController = UIHostingController(rootView: contentView)
        self.window = window
        window.makeKeyAndVisible()
    }
}
```



Chociaż lepszym rozwiązaniem może być użycie modyfikatora usług systemowych, jak opisano w "SwiftUI and the Missing Environment Object".



```swift
    let contentView = AppTabView()
        .modifier(SystemServices())
```

Z modyfikatorem usług w następujący sposób:

```swift
struct SystemServices: ViewModifier {
    private static var appState: AppState = AppState()
    private static var menu = MenuService()
    private static var messages = MessageService()
    private static var favorites = FavoritesService()
    private static var ratings = RatingsService()
    private static var order = OrderService()
    func body(content: Content) -> some View {
        content
            // defaults
            .accentColor(.red)
            // messages
            .overlay(MessageOverlayView(), alignment: .top)
            // services
            .environmentObject(Self.appState)
            .environmentObject(Self.menu)
            .environmentObject(Self.messages)
            .environmentObject(Self.favorites)
            .environmentObject(Self.order)
            .environmentObject(Self.ratings)
    }
}
```

Zauważ, że nasz modyfikator SystemServices istnieje po prostu po to, aby w razie potrzeby wstrzyknąć nasze usługi do środowiska SwiftUI (np. gdy przedstawiamy nowy widok modalny lub arkusz akcji). Dlatego jego członkowie są prywatni.

### Mikrousługi w SwiftUI

Architektura mikrousług jest definiowana jako układanie aplikacji w zbiór luźno powiązanych usług. Te usługi są drobnoziarniste, a protokoły między nimi są lekkie.

W architekturze mikrousług usługi można samodzielnie wdrażać. W przypadku FavoritesService powyżej, widzieliśmy, że możemy łatwo ponownie wdrożyć tę usługę i te komponenty interfejsu w innej aplikacji.

Wreszcie, odniesienie się do nich jako do mikrousług dodatkowo wzmacnia ideę, że nasze usługi powinny być małe, dobrze zdefiniowane, a każda z nich powinna być wdrażana z naciskiem na zarządzanie jednym aspektem naszej aplikacji.

Jedno źródło prawdy.