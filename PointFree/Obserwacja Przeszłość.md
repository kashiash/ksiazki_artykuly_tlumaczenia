Swift’s new observation tools

# Obserwacja: Przeszłość

Odcinek #252 - 9 października 2023 r. - tylko dla subskrybentów

https://www.pointfree.co/episodes/ep252-observation-the-past

Nadszedł czas, aby zagłębić się w nowe narzędzia obserwacyjne Swifta. Ale na początek przyjrzymy się narzędziom, które SwiftUI dostarczał w przeszłości, w tym wrapperom właściwości @State i @ObservedObject, jak się zachowują i gdzie mają braki, abyśmy mogli porównać je z nowym makrem @Observable.



## Wprowadzenie
00:05
Brandon: Jedną z bardziej ekscytujących rzeczy ogłoszonych na WWDC w tym roku były nowe narzędzia do "obserwacji", które dają nam możliwość obserwowania, jak zmieniają się określone części systemu. Było to ekscytujące z kilku powodów:

00:18
Po pierwsze, ta maszyneria znacznie poprawia jeden z największych bolączek SwiftUI, a mianowicie zrozumienie, w jaki sposób zmiany w modelu powodują, że widoki ponownie obliczają swoje ciała. Wcześniej to użytkownik musiał upewnić się, że pola używane w widoku zostały oznaczone za pomocą wrappera właściwości `@Published`. Jeśli tego nie zrobiłeś, możliwa była zmiana stanu w modelu bez aktualizacji widoku, co jest złym błędem. Lub jeśli oznaczysz zbyt wiele rzeczy za pomocą `@Published`, ryzykujesz zbyt częste ponowne obliczanie widoków.

00:47
Stephen: Po drugie, ten nowy framework Observation jest wbudowany bezpośrednio w otwarty język Swift, a nie jako zastrzeżony framework Apple. Oznacza to, że teoretycznie można wykorzystać moc obserwacji do wielu rzeczy poza SwiftUI, a nawet poza ekosystemem Apple, takich jak Linux, Windows i inne.

01:06
Brandon: I po trzecie, co jest naprawdę ekscytujące dla nas osobiście, nowy framework Observation pozwolił nam, to znaczy mnie i Stephenowi, na ponowną ocenę prawie każdego założenia, które przyjęliśmy, gdy pierwotnie budowaliśmy Composable Architecture. Daje nam to doskonałą okazję do znacznego uproszczenia i odchudzenia podstawowej biblioteki. Jesteśmy w stanie całkowicie pozbyć się pojęć takich jak "view store" i możemy pozbyć się zoo niestandardowych typów widoków i modyfikatorów, takich jak ForEachStore, IfLetStore, SwitchStore, sheet(store:) i innych.

01:46
Stephen: To naprawdę ekscytujące rzeczy, ale zanim będziemy mogli zagłębić się w to, jak to wszystko wpływa na Composable Architecture, musimy cofnąć się o krok i naprawdę zrozumieć framework Observation. Chcemy poświęcić kilka odcinków, aby naprawdę zagłębić się w ten temat:

01:58
Zaczniemy od przypomnienia sobie, jakie są narzędzia obserwacyjne dostarczone nam w pre-iOS 17, takie jak wrappery właściwości `@State` i `@ObservedObject`, i omówimy ich wady.

02:10
Brandon: Następnie pokażemy, w jaki sposób nowy framework `Observation` poprawia prawie każdy aspekt tych starych narzędzi, a nawet zagłębimy się w rzeczywisty kod w repozytorium Swift o otwartym kodzie źródłowym, aby zrozumieć, jak działają te narzędzia.

02:23
Stephen: Następnie pokażemy, że chociaż nowy framework `Observation` jest całkiem fantastyczny, to ma pewne problemy czające się w cieniu. Jeśli nie jesteś dogłębnie zaznajomiony z działaniem narzędzi, możesz łatwo zaobserwować znacznie więcej stanów, niż się spodziewasz.

02:37
Brandon: Zakończymy tę serię, przesuwając framework `Observation` do granic możliwości. Podczas gdy framework jest przeznaczony głównie do użytku z typami referencyjnymi, zbadamy, co oznaczałoby zastosowanie maszyn do typów wartości. Wiąże się to z pewnymi komplikacjami, ale warto to zrobić, ponieważ będzie to niezwykle ważne, gdy zintegrujemy framework Observation z naszą popularną biblioteką Composable Architecture, ponieważ jedną z jego najbardziej znanych cech jest to, że pozwala budować unikasz typów referencyjnych i budujesz swoje funkcje za pomocą prostych typów wartości.

03:10
Mamy wiele do omówienia, więc zaczynajmy!

## Obserwacja z `@State`
03:15
Nowy framework Observation można uznać za narzędzie ogólnego przeznaczenia do obserwowania zmian w systemie. Ale nie oszukujmy się. Jego stworzenie było prawie całkowicie motywowane przez SwiftUI, a jego początkowe wcielenie jest w większości użyteczne tylko dla SwiftUI. Podczas wczesnych etapów prac nad obserwacją zaproponowano wiele narzędzi, które sprawiłyby, że narzędzia te miałyby znacznie większe zastosowanie poza SwiftUI. Jednak narzędzia te zostały w większości odrzucone do czasu zaakceptowania propozycji, ponieważ wymagały nieco więcej pracy.

03:45
Z tego powodu użyjemy SwiftUI jako głównego przykładu, aby zrozumieć, jaki problem próbuje rozwiązać Observation. Utworzyliśmy nowy projekt, który zaczyna się od prawie pustego widoku i chcę zacząć dodawać dane do tego interfejsu użytkownika, który ma również pewną logikę i zachowanie.

04:05
Zdecydowanie najłatwiejszym sposobem na wprowadzenie stanu do widoku jest wrapper właściwości `@State`. Na przykład, możemy wprowadzić licznik do naszego głównego widoku zawartości i mieć przyciski do jego zwiększania i zmniejszania:

```swift
import SwiftUI

struct CounterView: View {
  @State var count = 0

  var body: some View {
    Formularz {
      Sekcja {
        Text(self.count.description)
        Button("Decrement") { self.count -= 1 }
        Button("Increment") { self.count += 1 }
      } header: {
        Text("Counter")
      }
    }
  }
}

#Preview {
  CounterView()
}
```

04:25
Będziemy musieli zmienić nazwę pliku i zaktualizować punkt wejścia aplikacji.

04:35
Właściwość `@State` wrapper tworzy fragment stanu, który jest w całości własnością widoku. Będzie on istniał tak długo, jak widok będzie wyświetlany na ekranie i nie zostanie zresetowany, nawet jeśli reprezentacja strukturalna widoku może zostać odtworzona dziesiątki lub setki razy.

04:53
Na początku brzmi to dość nieintuicyjnie, w końcu struktury mają być prostymi, lekkimi reprezentacjami danych, wolnymi od zachowań. A jednak w jakiś sposób ten `@State` utrzymuje się podczas wielokrotnego tworzenia struktury.

05:05
Cóż, to wszystko dlatego, że potajemnie za kulisami SwiftUI śledzi cień świata typów referencyjnych, który reprezentuje prawdziwą hierarchię widoku na ekranie. A te typy referencyjne są trwałe, nawet jeśli odpowiadająca im wersja struktury była wielokrotnie odtwarzana i odrzucana. I to właśnie w tych typach referencyjnych wartość `@State` jest naprawdę przechowywana i w ten sposób może przetrwać wewnątrz tej struktury.

05:36
Jeśli uruchomimy podgląd, zobaczymy, że działa zgodnie z naszymi oczekiwaniami. Co więcej, jeśli oprzyrządujemy właściwość body za pomocą _printChanges:

```swift
var body: some View {
  let _ = Self._printChanges()
  ...
}
```

05:55
...i ponownie uruchomić podgląd, zobaczymy, że coś jest drukowane do konsoli za każdym razem, gdy stan jest mutowany i widok jest ponownie renderowany.

```
CounterView: @self, @identity, _count changed.
CounterView: _count changed.
```

06:08
OK, więc jeszcze nic zaskakującego, ale poprawmy zachowanie tego widoku. Dodajmy teraz przyciski, które mogą uruchamiać i zatrzymywać timer, który tyka co sekundę. Z każdym tyknięciem timera będziemy inkrementować oddzielny element stanu, a nie licznik.

06:23
Dodamy nowy stan do widoku:

```swift
struct CounterView: View {
  @State var secondsElapsed = 0
  ...
}
```

06:25
Utworzymy nową sekcję w formularzu dla timera:

```swift
Section {

} header: {
  Text("Timer")
}
```

06:31
W sekcji timera dodamy przycisk uruchamiający timer:

```swift
Button("Start timer") {

}
```

06:35
Po naciśnięciu tego przycisku musimy uruchomić timer. Najprostszym sposobem na to jest utworzenie nieustrukturyzowanego zadania, które da nam dostęp do kontekstu asynchronicznego, a następnie możemy po prostu wykonać nieskończoną pętlę z uśpieniem:

```swift
Button("Start timer") {
  self.secondsElapsed = 0
  Task {
    while true {
      try await Task.sleep(for: .seconds(1))
      self.secondsElapsed += 1
    }
  }
}
```

06:54
Jest to najprostszy sposób na utworzenie timera, ale nie jest to najbardziej niezawodny sposób. Gdyby zależało nam na dokładnych zegarach, musielibyśmy wykonać więcej pracy, aby uwzględnić dryf w czasie snu, a gdyby zależało nam na testowalności, chcielibyśmy wstrzyknąć zegar do tego widoku. Ale na razie to wystarczy.

07:14
Mamy teraz przycisk do uruchamiania timera, ale jak możemy go zatrzymać? Cóż, aby zatrzymać asynchroniczną jednostkę pracy, musimy mieć do niej uchwyt, aby móc ją anulować. Odbywa się to poprzez trzymanie się zadania w widoku jako lokalnej właściwości stanu:

```swift
struct CounterView: View {
  @State var timerTask: Task<Void, Error>?
  ...
}
```

07:39
Teraz, gdy timer zostanie uruchomiony, możemy upewnić się, że anulujemy każdy aktualnie działający timer, a następnie uruchomimy nowy timer i będziemy śledzić zadanie:

```swift
Button("Start timer") {
  self.secondsElapsed = 0
  self.timerTask?.cancel()
  self.timerTask = Task {
    ...
  }
}
```

07:52
Następnie możemy sprawdzić, czy timer jest obecnie w locie, abyśmy wiedzieli, który przycisk wyświetlić. Pokażemy nawet mały wskaźnik postępu obok przycisku zatrzymania, aby było jasne, że timer jest w toku:

```swift
if self.timerTask == nil {
  Button("Start timer") {
    ...
  }
} else {
  Button {
    
  } label: {
    HStack {
      Text("Stop timer")
      Spacer()
      ProgressView().id(UUID())
    }
  }
}
```

08:39
Przycisk zatrzymania może teraz zaimplementować swoją logikę, anulując dowolne bieżące zadanie, a ponadto musi upewnić się, że stan jest zerowy, aby przycisk "Uruchom timer" powrócił do interfejsu użytkownika:

```swift
Button {
  self.timerTask?.cancel()
  self.timerTask = nil
} label: {
  ...
}
```

08:54
Dzięki temu możemy uruchomić podgląd i zaobserwować kilka interesujących rzeczy. Po pierwsze, możemy uruchomić timer i zobaczyć, że przycisk "Stop timer" pojawia się wraz ze wskaźnikiem postępu. A następnie dotknięcie przycisku "Stop timer" powoduje przełączenie przycisku z powrotem na "Start timer".

09:09
Jednak celowo nie dodaliśmy jeszcze secondsElapsed do widoku, więc nie mieliśmy wizualnej reprezentacji, że stan faktycznie się aktualizuje. Jeśli spojrzymy na logi, zobaczymy, co następuje:

```
CounterView: @self, @identity, _count, _secondsElapsed, _timerTask changed.
CounterView: _timerTask changed.
CounterView: _timerTask changed.
```

09:24
Widok został wyrenderowany tylko 3 razy, mimo że licznik odmierzał czas wielokrotnie. Nastąpił jeden render dla stanu początkowego, następnie kolejny render, gdy timer został uruchomiony w celu wyświetlenia przycisku "Stop timer", a następnie kolejny render, gdy timer został zatrzymany w celu wyświetlenia przycisku "Start timer".

09:32
Stan secondsElapsed był zdecydowanie mutowany, ale w jakiś sposób jego mutacja nie spowodowała ponownego obliczenia ciała widoku. Możemy nawet umieścić instrukcję drukowania w timerze:

```swift
while true {
  try await Task.sleep(for: .seconds(1))
  self.secondsElapsed += 1
  print("secondsElapsed", self.secondsElapsed)
}
```

09:41
I ponownie uruchom podgląd, uruchamiając i zatrzymując timer, a zobaczymy w dziennikach:

```
CounterView: @self, @identity, _count, _secondsElapsed, _timerTask changed.
CounterView: _timerTask changed.
secondsElapsed 1
secondsElapsed 2
secondsElapsed 3
CounterView: _timerTask changed.
```

Tak więc rzeczywiście właściwość `@State` jest zdecydowanie modyfikowana, ale jej mutacje nie doprowadziły do ponownego obliczenia widoku.

09:47
Jest to całkiem fajna funkcja opakowywania właściwości `@State`, która zachowuje się w ten sposób od samego początku SwiftUI. Jeśli użyjesz `@State` do przechwycenia lokalnego, zmiennego stanu dla widoku, ale nigdy nie użyjesz go w widoku, to jego mutacje nie spowodują ponownego renderowania widoku.

Jeśli zaczniemy używać stanu secondsElapsed w widoku:

Section {
  Text("Upłynęły sekundy: \(self.secondsElapsed)")
  ...
} header: {
  Text("Timer")
}
10:24
Po ponownym uruchomieniu podglądu i 3-krotnym odliczeniu czasu, zobaczymy zupełnie inne logi:

CounterView: @self, @identity, _count, _secondsElapsed, _timerTask changed.
CounterView: _timerTask changed.
secondsElapsed 1
CounterView: _secondsElapsed zmieniono.
secondsElapsed 2
CounterView: _secondsElapsed zmieniono.
secondsElapsed 3
CounterView: _secondsElapsed zmieniono.
CounterView: _timerTask zmieniono.
10:28
Teraz każda zmiana stanu powoduje ponowne renderowanie widoku, ponieważ wie, że widok używa stanu secondsElapsed.

10:42
Więc to jest całkiem fajna rzecz. W jakiś sposób wrapper właściwości @State jest sprytny, aby wiedzieć, kiedy jest używany w widoku, a jeśli jest, może upewnić się, że wszelkie mutacje spowodują ponowne wyrenderowanie widoku. Natomiast jeśli nie jest używana, to wie, że nie ma potrzeby ponownego renderowania, gdy się zmieni.

10:56
Zobaczmy jednak, jak sprytny jest wrapper właściwości @State. Co jeśli warunkowo pokażemy pole secondsElapsed w widoku w czasie wykonywania. Czy widok pominie renderowanie, gdy dane nie są wyświetlane, a następnie rozpocznie renderowanie ponownie, gdy dane zostaną umieszczone w widoku?

11:13
Aby to zbadać, dodamy nowy stan do widoku:

```swift
@State var isDisplayingSecondsElapsed = true
```

11:22
I sprawdzimy ten stan przed wyświetleniem sekund, które upłynęły:

```swift
if self.isDisplayingSecondsElapsed {
  Text("Sekundy, które upłynęły: \(self.secondsElapsed)")
}
```

11:28
I wreszcie sprawimy, że stan będzie można przełączać z widoku:

```swift
Toggle(isOn: self.$isDisplayingSecondsElapsed) {
  Text("Wyświetl sekundy")
}
```

11:36
Teraz w podglądzie uruchommy timer i włączmy lub wyłączmy wyświetlanie. Niestety przekonamy się, że SwiftUI nie ma dość sprytu, aby to zrobić:

```
CounterView: @self, @identity, _count, _secondsElapsed, _isDisplayingSecondsElapsed, _timerTask changed.
CounterView: _timerTask changed.
CounterView: _secondsElapsed changed.
CounterView: _isDisplayingSecondsElapsed changed.
CounterView: _secondsElapsed changed.
CounterView: _secondsElapsed changed.
CounterView: _secondsElapsed changed.
CounterView: _timerTask changed.
```

11:51
Nawet jeśli sekundy, które upłynęły, nie były wyświetlane w widoku, w jakiś sposób nadal powodowało to ponowne renderowanie ciała widoku. Domyślam się, że po wykryciu, że dane są używane w widoku, po prostu subskrybuje ich aktualizacje na zawsze, nawet jeśli dane przestaną być używane później.

## Problemy z `@State`
12:33
Brandon: OK, widzieliśmy teraz, że wrapper właściwości `@State` jest jednym z narzędzi do dynamicznego aktualizowania danych w widoku, mimo że widoki są modelowane jako struktury, które nie mają pojęcia czasu życia ani zachowania. I widzieliśmy, że @State ma przyzwoitą ilość sprytnych rozwiązań, które próbują zmniejszyć liczbę ponownych renderowań widoku, gdy zmienia się stan.

12:54
Stephen: Jednak właściwość `@State` ma bardzo specyficzny przypadek użycia i nie jest odpowiednia do używania przez cały czas. Najlepiej jest jej używać dla stanu, który jest całkowicie lokalny dla widoku i nie musi nigdy podlegać wpływom z zewnątrz. Na przykład, jeśli tworzysz komponent UI wielokrotnego użytku, taki jak przycisk lub suwak, i chcesz śledzić jakiś stan wewnętrzny, taki jak to, czy użytkownik aktualnie dotyka komponentu. Jest to stan, na który strona zewnętrzna nie powinna mieć wpływu i o którym tak naprawdę nie musi nawet wiedzieć.

13:19
Brandon: Jednak nie wszystkie stany są tego rodzaju. Wiele stanów musi być przekazywanych do widoków od rodzica, aby rodzic mógł obserwować zmiany, jakie dziecko wprowadza do stanu, i aby rodzic mógł wprowadzać zmiany w stanie i odpowiednio reagować na dziecko. Jest to niezwykle ważne w przypadku głębokiego linkowania, gdzie domena nadrzędna musi być w stanie skonstruować fragment stanu, przekazać go do SwiftUI i pozwolić SwiftUI przywrócić wszystkie widoki podrzędne, w tym arkusze, drążenia i inne.

13:46
Jeśli przechowujesz cały stan aplikacji przy użyciu wrappera właściwości @State, a nawet przy użyciu wrappera właściwości @StateObject, to poważnie ograniczasz możliwość głębokiego linkowania do określonego stanu aplikacji. Ponieważ każdy mały widok zarządza własnym stanem lokalnym, rodzic nie ma możliwości wpływania na ten stan, a tym samym nie ma możliwości głębokiego linkowania.

14:07
Stephen: Jest jeszcze jeden powód, by nie używać @State dla wszystkich stanów widoków. Im bardziej złożone zachowanie ma twój widok, w szczególności im bardziej musi wchodzić w interakcje z zewnętrznymi systemami, takimi jak wysyłanie żądań sieciowych, zarządzanie długotrwałymi efektami lub korzystanie z frameworków Apple, takich jak CoreLocation, tym trudniej jest hermetyzować całą tę logikę w widoku.

14:28
Z tych powodów i nie tylko, istnieje koncepcja ObservableObject, więc szybko przyjrzyjmy się, co ona wnosi do stołu.

14:38
Przede wszystkim przyjrzyjmy się naszemu widokowi licznika w obecnej formie, aby zobaczyć, jak bardzo się skomplikował. Podstawowym celem widoku jest skonstruowanie hierarchii widoku, która jest rzeczą zwracaną z właściwości body, ale teraz posypaliśmy go wszelkiego rodzaju złożoną i zniuansowaną logiką.

14:52
Miejsca, w których dodaliśmy logikę i zachowanie, znajdują się właśnie w zamknięciach ucieczki, takich jak zamknięcia akcji przycisków. Są to jedyne miejsca, w których można wstawić zachowanie w widoku SwiftUI. Nie można wykonywać efektów ubocznych w żadnym innym miejscu widoku.

15:04
Na przykład, uruchomienie timera bezpośrednio w ciele widoku nie miałoby żadnego sensu:

```swift
var body: some View {
  let _ = self.timerTask = Task {
    while true {
      try await Task.sleep(for: .seconds(1))
      self.secondsElapsed += 1
      print("secondsElapsed", self.secondsElapsed)
    }
  }
  ...
}
```

15:34
Po pierwsze, ciało widoku może być wywoływane wiele, wiele razy. Jako użytkownicy SwiftUI powinniśmy być całkowicie nieświadomi wewnętrznego działania frameworka, w tym tego, kiedy i jak wywoływana jest ta właściwość. Oznacza to, że moglibyśmy przypadkowo uruchomić ten timer wiele razy, kiedy nie spodziewalibyśmy się tego.

15:48
Co więcej, nie jest nawet poprawne mutowanie właściwości @State w trakcie oceny treści widoku. Gdybyś uruchomił to w symulatorze, otrzymałbyś fioletowe ostrzeżenie informujące, że jest to całkowicie niewłaściwe:

Modyfikowanie stanu podczas aktualizacji widoku spowoduje niezdefiniowane zachowanie.

16:04
Możesz mutować @State tylko z zamknięć uciekających, takich jak zamknięcia akcji przycisku.

16:11
Tak więc, używając @State do wszystkiego w tym widoku, połączyliśmy dwie uzupełniające się, ale oddzielne koncepcje: Zachowanie widoku i tworzenie widoku. I naprawdę, ten widok staje się naprawdę trudny do odczytania. W miarę dodawania coraz większej ilości zachowań, trudno będzie zobaczyć, jak naprawdę wyglądają kości widoku. Fakt, że jest to prosty formularz z kilkoma sekcjami i kilkoma przyciskami, zostanie całkowicie przesłonięty, ponieważ zdecydowana większość linii w widoku jest poświęcona wykonywaniu akcji, a nie konstruowaniu hierarchii widoku.

16:44
Tak więc, jeśli zależy nam na oddzieleniu zachowania od tworzenia widoku i zależy nam na przywracaniu stanu i głębokim linkowaniu, a także jeśli zależy nam na innych rzeczach, o których nie mamy czasu mówić, takich jak testowanie, to należy rozważyć przeniesienie stanu i zachowania widoku do zewnętrznego obiektu, który jest wstrzykiwany do widoku.

17:04
Zaczniemy od podejścia do tego w bardzo naiwny sposób. Chcemy zamknąć logikę i zachowanie tej funkcji w jakimś obiekcie, a obiekt ten musi być typem referencyjnym, ponieważ wykonywane są efekty uboczne. A co by było, gdybyśmy nie wiedzieli o pełnym zoo wrapperów właściwości dostarczanych przez SwiftUI i po prostu natywnie zaimplementowaliśmy następującą klasę:

```swift
class CounterModel {
  var count = 0
  var secondsElapsed = 0
  private var timerTask: Task<Void, Error>?
  var isTimerOn: Bool {
    self.timerTask != nil
  }

  func decrementButtonTapped() {
    self.count -= 1
  }
  func incrementButtonTapped() {
    self.count += 1
  }

  func startTimerButtonTapped() {
    self.timerTask?.cancel()
    self.timerTask = Task {
      while true {
        try await Task.sleep(for: .seconds(1))
        self.secondsElapsed += 1
        print("secondsElapsed", self.secondsElapsed)
      }
    }
  }

  func stopTimerButtonTapped() {
    self.timerTask?.cancel()
    self.timerTask = nil
  }
}
```

17:24
Jest to prosta klasa, która opakowuje niektóre zmienne, te same, których używaliśmy w widoku. Następnie udostępniamy metody, które można wywołać z widoku, aby rozpocząć wykonywanie logiki i zachowania funkcji. Lubimy nazywać metody dosłownie po tym, co użytkownik robi w widoku, na przykład dotykając przycisku, a nie po tym, jaką pracę wykona model wewnątrz metody. Uwalnia to metody od robienia czegokolwiek wewnątrz i sprawia, że jest bardzo jasne, które metody powinny być wywoływane z których części widoku.

17:46
I ten obiekt zdecydowanie musi być klasą ze względu na efekty uboczne. Jeśli spróbujemy zmienić go na strukturę, natychmiast zobaczymy, co pójdzie nie tak:

```swift
struct CounterModel {
  ...
}
```

17:54
Otrzymamy kilka błędów dotyczących mutacji, ale można je łatwo naprawić poprzez zmutowanie metod. Prawdziwy problem pojawia się, gdy próbujemy zmutować model po jakiejś asynchronicznej pracy:

🛑 Mutable capture of 'inout' parameter 'self' is not allowed in concurrently-executing code.

18:10
Nie możemy uzyskać dostępu do mutowalnego parametru self wewnątrz wysyłanego zamknięcia Task, co ostatecznie jest wadą próby użycia struktury w tym modelu. Ale to nie szkodzi, klasy są świetne do modelowania domen, które mają zachowanie, tak jak ta.

18:22
Kolejną rzeczą, na którą chcemy zwrócić uwagę, jest to, że wszystko w tej klasie jest w zasadzie tym, co zaśmieciliśmy w naszym widoku, a teraz wszystko zebrane w jednym, łatwym do zrozumienia miejscu. Mamy mutowalny stan dla licznika i sekund, które upłynęły. Co więcej, uczyniliśmy nawet timerTask prywatnym, ponieważ widok nie powinien dbać o ten konkretny szczegół. Wszystko, na czym zależy widokowi, to to, czy timer jest włączony, czy nie, a więc udostępniliśmy obliczoną właściwość dla tego szczegółu.

18:44
Następnie mamy metody, które widok może wywołać, aby wykonać różne akcje, gdy użytkownik coś zrobi. Zawartość tych metod jest dokładnie tym, co robimy teraz w widoku. Możemy nawet uniemożliwić widokowi mutowanie właściwości stanu tego obiektu poza tymi metodami, oznaczając właściwości jako prywatne (set).

19:07
Ta klasa może być nawet testowana, ponieważ została odłączona od widoku. Po prostu utworzylibyśmy instancję obiektu w teście, wywołalibyśmy kilka metod, aby naśladować działanie użytkownika, takie jak uruchamianie i zatrzymywanie timera, a następnie sprawdzilibyśmy, jak zmienia się stan wewnątrz obiektu.

19:22
Co teraz trzeba zrobić, aby użyć tego modelu w naszym widoku? Cóż, podejdźmy do tego w najbardziej naiwny możliwy sposób. Jeśli chciałbym, aby ten model był dostarczany przez domenę nadrzędną, to trzymałbym go jako prosty let:

```swift
struct CounterView: View {
  let model: CounterModel
  ...
}
```

19:34
Oznacza to, że za każdym razem, gdy ktoś tworzy ten widok, musi przekazać mu CounterModel, a tym samym przekazuje odpowiedzialność za utrzymywanie długotrwałego modelu gdzieś w górę łańcucha. Albo domena nadrzędna, albo rodzic rodzica, i tak dalej.

19:45
Alternatywnie