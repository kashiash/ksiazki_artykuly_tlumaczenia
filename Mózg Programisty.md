# Mózg Programisty:

To, co każdy programista powinien wiedzieć o procesie poznawania



![image-20231119160926177](/Users/uta/Library/Application%20Support/typora-user-images/image-20231119160926177.png)

**Przedmowa**

Spędziłem dużo czasu swojego życia myśląc o programowaniu, i jeśli czytasz tę książkę, prawdopodobnie ty również. Niemalże nie poświęciłem jednak tyle samo czasu na zastanawianie się nad samym procesem myślowym. Koncepcja naszych procesów myślowych i tego, w jaki sposób jako ludzie oddziałujemy z kodem, jest dla mnie istotna, choć nie została poparta badaniami naukowymi. Pozwól, że przedstawię ci trzy przykłady.

Jestem głównym współtwórcą projektu .NET o nazwie Noda Time, dostarczającego alternatywne typy daty i czasu w stosunku do tych wbudowanych w .NET. To dla mnie świetne środowisko do projektowania interfejsu API, zwłaszcza jeśli chodzi o nazewnictwo. Patrząc na problemy spowodowane nazwami, które sugerują, że zmieniają istniejącą wartość, ale w rzeczywistości zwracają nową wartość, starałem się używać nazw, które sprawiają, że błędny kod brzmi nieprawidłowo podczas jego czytania. Na przykład typ LocalDate ma metodę PlusDays zamiast AddDays. Mam nadzieję, że ten kod wygląda nieprawidłowo dla większości programistów w języku C#.

```swift
date.PlusDays(1);
```

Wydaje się, że ta linijka kodu dodaje jeden dzień do zmiennej "date". Jest to jednak mylące, ponieważ sugeruje, że modyfikuje istniejącą wartość, podczas gdy w rzeczywistości tworzy nową wartość reprezentującą dzień następny po "date".

**Natomiast wydaje się bardziej sensowne:**

```csharp
tomorrow = today.PlusDays(1);
```

W tym przypadku używamy bardziej intuicyjnej nazwy zmiennej "tomorrow", co sprawia, że kod staje się bardziej zrozumiały. Metoda "PlusDays(1)" dodaje jeden dzień do wartości "today", a wynik przypisuje się do zmiennej "tomorrow".

Porównaj to z metodą AddDays w typie DateTime w .NET:

```csharp
date.AddDays(1);
```

To wygląda tak, jakby po prostu modyfikowało "date" i nie było błędem, mimo że jest równie niepoprawne co pierwsze przykłady.

Drugi przykład również pochodzi z Noda Time, ale nie jest tak specyficzny. Tam, gdzie wiele bibliotek próbuje (z dobrych powodów) wykonywać całą trudną pracę, nie wymagając od programistów zbyt dużego myślenia, my jawnie chcemy, aby użytkownicy Noda Time zastanawiali się nad swoim kodem daty i czasu na wstępie. Staraliśmy się zmusić użytkowników do przemyślenia, co tak naprawdę próbują osiągnąć, bez żadnych niejasności, a następnie staraliśmy się sprawić, aby było łatwo to jasno wyrazić w kodzie.

Na koniec jest przykład koncepcyjny dotyczący tego, jakie wartości przechowują zmienne w językach Java i C#, oraz co się dzieje, gdy przekazujesz argument do metody. Wydaje mi się, że przez większość mojego życia starałem się przeciwstawić przekonaniu, że obiekty są przekazywane przez referencję w Javie, i gdy przeliczam to matematycznie, prawdopodobnie tak właśnie było. podejrzewam, że przez około 25 lat próbowałem pomagać innym programistom dostosować swoje modele mentalne.

Okazuje się, że to, w jaki sposób programiści myślą, było dla mnie ważne przez długi czas, ale bez naukowego wsparcia, tylko domysły i zdobyte trudem doświadczenie. Ta książka pomaga to zmienić, chociaż nie jest to całkowity początek tego procesu dla mnie.

Pierwszy raz spotkałem się z Felienne Hermans na konferencji NDC w Oslo w 2017 roku, kiedy wygłosiła prezentację "Programming Is Writing Is Programming". Moja reakcja na Twitterze mówi wszystko: "Potrzebuję dużo czasu, aby wszystko przyswoić. Ale wow. Wow." Oglądałem tę prezentację Felienne (ewoluującą oczywiście z czasem) przynajmniej trzy razy i za każdym razem wynosiłem z niej coś nowego. Wreszcie pojawiły się pewne kognitywne wyjaśnienia dla rzeczy, które próbowałem robić, a także kilka niespodzianek, które zmusiły mnie do dostosowania swojego podejścia.

Zmienne reakcje "Ah, teraz to ma sens!" i "O, nie pomyślałem o tym!" były tłem rytmu podczas czytania tej książki. Oprócz kilku bezpośrednich praktycznych sugestii, takich jak korzystanie z fiszek, podejrzewam, że wpływ tej książki będzie bardziej subtelny. Być może to trochę więcej rozważań o tym, kiedy wstawiać pustą linię w kodzie. Może to zmiana zadań, które dajemy nowym członkom zespołu, albo po prostu zmiana czasu ich wykonywania. Może to sposób, w jaki wyjaśniamy koncepcje na Stack Overflow.

Nie ważne, jaki będzie wpływ, Felienne dostarczyła skarbnicę pomysłów do przemyślenia i przetworzenia w pamięci operacyjnej, a następnie przeniesienia do pamięci długotrwałej. Myślenie o myśleniu jest uzależniające!

—Jon Skeet



**Przedmowa**

Kiedy zacząłem nauczać dzieci programowania około 10 lat temu, szybko zdałem sobie sprawę, że nie mam zielonego pojęcia, jak ludzie używają swoich mózgów do cokolwiek, zwłaszcza do programowania. Chociaż nauczyłem się wielu rzeczy związanych z programowaniem na studiach, żaden kurs w mojej edukacji informatycznej nie przygotował mnie do myślenia o myśleniu w kontekście programowania.

Jeśli podążałeś za programem studiów informatycznych, tak jak ja, lub jeśli nauczyłeś się programowania samodzielnie, najprawdopodobniej nie dowiedziałeś się nic o funkcjach poznawczych mózgu. Dlatego być może nie wiesz również, jak poprawić funkcjonowanie swojego mózgu, aby lepiej czytać i pisać kod. Ja z pewnością nie wiedziałem, ale ucząc dzieci programowania, zdałem sobie sprawę, że potrzebuję głębszego zrozumienia procesu poznawczego. Wtedy postanowiłem dowiedzieć się więcej o to, jak myślimy i jak się uczymy. Ta książka jest wynikiem kilku ostatnich lat, które spędziłem na czytaniu książek, rozmowach z ludźmi i uczestnictwie w wykładach i konferencjach na temat nauki i myślenia.

Zrozumienie, jak działa twój mózg, jest interesujące samo w sobie, ale ma to także znaczenie dla programowania. Programowanie uważane jest za jedną z najbardziej wymagających aktywności poznawczych: rozwiązujesz problem w sposób abstrakcyjny i manipulujesz programem, co wymaga uwagi na poziomie, który nie przychodzi naturalnie większości ludzi. Brak spacji? Błąd. Źle obliczone miejsce indeksowania tablicy? Błąd. Źle zrozumiana precyzyjna praca już istniejącego kodu? Błąd.

Istnieje wiele sposobów, w jakie możesz sobie zaszkodzić, programując. Jak zobaczysz w tej książce, wiele błędów, które popełniasz, ma swoje źródło w problemach poznawczych. Na przykład brak spacji może oznaczać, że nie opanowałeś składni języka programowania w wystarczającym stopniu. Błędne obliczenie, gdzie zaindeksować tablicę, może wskazywać na błędne założenia dotyczące kodu. Niezrozumienie istniejącego kodu może wynikać z braku umiejętności czytania kodu.

Celem tej książki jest przede wszystkim pomóc ci zrozumieć, w jaki sposób twój mózg przetwarza kod. Zrozumienie, co robi twój mózg, gdy jest mu prezentowana nowa informacja, pomoże ci być lepszym programistą, ponieważ zawodowi programiści często mają do czynienia z nowymi informacjami. Gdy już będziemy wiedzieć, w jaki sposób kod wpływa na mózg, omówimy metody poprawy twoich umiejętności przetwarzania kodu.

**Podziękowania**

Doskonale zdaję sobie sprawę, jak ogromnie mam szczęście, że mogę ukończyć książkę na temat, który kocham. W moim życiu zdarzyło się tyle rzeczy, które stały się dokładnie w odpowiedni sposób w dokładnie odpowiednim momencie, bez których moje życie byłoby zupełnie inne, a ta książka z pewnością nie zostałaby napisana. Dziesiątki małych i dużych spotkań z niesamowitymi ludźmi przyczyniły się do powstania tej książki i do mojej kariery. Chcę wymienić kilka bardzo ważnych.

Marlies Aldewereld wprowadziła mnie po raz pierwszy na ścieżkę programowania i nauki języków. Marileen Smit nauczyła mnie wystarczająco dużo psychologii, abym mógł napisać tę książkę. Greg Wilson uczynił temat nauczania programowania ponownie popularnym. Peter Nabbe i Rob Hoogerwoord stanowią światowy wzór, jak być doskonałym nauczycielem. Stefan Hanenberg udzielił mi rad, które ukształtowały trajektorię moich badań. Katja Mordaunt zainicjowała pierwszy klub czytania kodu. Myślenie Llewellyna Falco na temat kōanów głęboko wpłynęło na moje rozumienie procesu nauki. A Rico Huijbers jest moim światłem w każdej burzy.

**Podziękowania**

Oprócz tych osób, oczywiście, muszę podziękować ludziom z Manning— Marjan Bace, Mike Stephens, Tricia Louvar, Bert Bates, Mihaela Batinić, Becky Reinhart, Melissa Ice, Jennifer Houle, Paul Wells, Jerry Kuch, Rachel Head, Sébastien Portebois, Candace Gillhoolley, Chris Kaufmann, Matko Hrvatin, Ivan Martinović, Branko Latinčić, oraz Andrej Hofšuster za podjęcie się tej nieokreślonej koncepcji książki i przekształcenie jej w coś czytelnego i sensownego.

Dla wszystkich recenzentów: Adam Kaczmarek, Adriaan Beiertz, Alex Rios, Ariel Gamiño, Ben McNamara, Bill Mitchell, Billy O’Callaghan, Bruno Sonnino, Charles Lam, Claudia Maderthaner, Clifford Thurber, Daniela Zapata Riesco, Emanuele Origgi, George Onofrei, George Thomas, Gilberto Taccari, Haim Raman, Jaume Lopez, Joseph Perenia, Kent Spillner, Kimberly Winston-Jackson, Manuel Gonzalez, Marcin Sęk, Mark Harris, Martin Knudsen, Mike Hewitson, Mike Taylor, Orlando Méndez Morales, Pedro Seromenho, Peter Morgan, Samantha Berk, Sebastian Felling, Sébastien Portebois, Simon Tschöke, Stefano Ongarello, Thomas Overby Hansen, Tim van Deurzen, Tuomo Kalliokoski, Unnikrishnan Kumar, Vasile Boris, Viktor Bek, Zachery Beyel, i Zhijun Liu, wasze sugestie pomogły uczynić tę książkę lepszą.

**O tej Książce**

"Mózg Programisty" to publikacja skierowana do programistów wszystkich poziomów, którzy pragną uzyskać głębsze zrozumienie tego, w jaki sposób ich mózgi funkcjonują oraz jak mogą poprawić swoje umiejętności i nawyki programistyczne. Przedstawione zostaną przykłady kodu w różnych językach, w tym JavaScript, Python i Java. Jednak nie wymaga się głębokiej wiedzy z żadnego z tych języków, o ile potrafisz czytać kod źródłowy w językach programowania, które mogą być dla Ciebie nowością.

Aby jak najlepiej wykorzystać lekturę tej książki, zaleca się posiadanie doświadczenia w pracy zespołowej deweloperskiej lub nad większymi systemami oprogramowania oraz w procesie wprowadzania nowych członków do zespołu. Często będziemy się odnosić do tego typu sytuacji, a Twoje zrozumienie zyska na głębokości, jeśli będziesz mógł łączyć je z własnymi doświadczeniami. W rzeczywistości proces uczenia się staje się bardziej efektywny, gdy można łączyć nowe informacje z istniejącą wiedzą i doświadczeniami, co obejmuje prezentowana w tej książce tematykę.

Chociaż książka ta porusza wiele zagadnień z zakresu nauk poznawczych, jest w gruncie rzeczy dedykowana programistom. Zawsze będziemy kontekstualizować funkcjonowanie mózgu na podstawie wyników badań nad programowaniem i językami programowania.

**Jak ta książka jest zorganizowana: **

Ta książka składa się z 13 rozdziałów podzielonych na 4 części. Rozdziały powinny być czytane w określonej kolejności, ponieważ budują na sobie nawzajem. Każdy rozdział oferuje zastosowania i ćwiczenia, które pomogą ci przetworzyć materiał i zrozumieć go głębiej. W niektórych przypadkach poproszę cię o znalezienie bazy kodu, na której będziesz mógł wykonywać ćwiczenia, aby zapewnić najlepszy kontekst dla ciebie.

Twoja codzienna praktyka programistyczna to również miejsce, w którym wiedza powinna być stosowana. Wyobrażam sobie, że możesz czytać tę książkę przez długi okres i stosować nauki z jednego rozdziału do praktyki programistycznej, zanim przejdziesz do kolejnych rozdziałów.

**Rozdział 1** analizuje trzy procesy poznawcze, które odgrywają rolę podczas programowania i jak każdy z nich związany jest z własnym rodzajem dezorientacji.

**Rozdział 2** omawia, jak szybko czytać kod i uzyskiwać wyobrażenie o jego działaniu.

**Rozdział 3** uczy, jak lepiej i łatwiej uczyć się składni i koncepcji programowania.

**Rozdział 4** pomaga w czytaniu złożonego kodu.

**Rozdział 5** prezentuje techniki, które pomagają osiągnąć głębsze zrozumienie nieznajomego kodu.

**Rozdział 6** obejmuje techniki poprawiające umiejętność rozwiązywania problemów programistycznych.

**Rozdział 7** pomaga unikać błędów zarówno w kodzie, jak i w myśleniu.

**Rozdział 8** omawia wybór czytelnych nazw zmiennych, zwłaszcza w kontekście kodu źródłowego.

**Rozdział 9** skupia się na "zapachach kodu" (code smells) oraz zasadach poznawczych stojących za nimi.

**Rozdział 10** dyskutuje o bardziej zaawansowanych technikach rozwiązywania skomplikowanych problemów.

**Rozdział 11** obejmuje samo akt kodowania i eksploruje różnorodność zadań w programowaniu.

**Rozdział 12** uczy, w jaki sposób poprawić duże bazy kodu.

**Rozdział 13** pomaga uczynić proces wprowadzania nowych programistów mniej bolesnym.

Książka zawiera wiele przykładów kodu źródłowego zarówno w numerowanych listach, jak i w tekście. W obu przypadkach kod źródłowy jest sformatowany stałą czcionką o stałej szerokości, aby oddzielić go od zwykłego tekstu. Czasami kod jest również pogrubiony, aby wyróżnić kod, który zmienił się w stosunku do poprzednich kroków w rozdziale, na przykład gdy nowa funkcja dodaje się do istniejącej linii kodu.



W wielu przypadkach oryginalny kod źródłowy został sformatowany ponownie; dodaliśmy złamania linii i dostosowaliśmy wcięcia, aby zmieścić się w dostępnej przestrzeni na stronie książki. W rzadkich przypadkach nawet to nie wystarczyło, i listy zawierają znaczniki kontynuacji linii (➥). Dodatkowo, komentarze w kodzie źródłowym zostały często usunięte z listy, gdy kod jest opisany w tekście. Do wielu list dodane są adnotacje kodu, podkreślające istotne koncepcje.

### Dyskusyjne forum liveBook
Zakup książki "The Programmer’s Brain" obejmuje darmowy dostęp do prywatnego forum internetowego prowadzonego przez Manning Publications, gdzie możesz zamieszczać komentarze dotyczące książki, zadawać pytania techniczne i otrzymywać pomoc od autora oraz innych użytkowników. Aby uzyskać dostęp do forum, odwiedź https://livebook.manning.com/book/the-programmers-brain/discussion. Możesz także dowiedzieć się więcej o forach Manning i zasadach ich korzystania pod adresem https://livebook.manning.com/#!/discussion.

Zobowiązaniem Manning wobec czytelników jest zapewnienie miejsca, gdzie może się odbywać znaczący dialog między poszczególnymi czytelnikami a także między czytelnikami a autorem. Nie jest to zobowiązanie do określonej ilości udziału ze strony autora, którego wkład w forum pozostaje dobrowolny (i niepłatny). Sugerujemy, abyś próbował zadawać autorowi trudne pytania, by utrzymać jego zainteresowanie! Forum oraz archiwa poprzednich dyskusji będą dostępne na stronie wydawcy tak długo, jak książka będzie dostępna w druku.

**O Autorze**

Dr. Felienne Hermans jest profesorem nadzwyczajnym na Uniwersytecie w Lejdzie w Holandii, gdzie prowadzi badania w dziedzinie nauczania programowania i języków programowania. Jest również edukatorem nauczycieli w Akademii Nauczycielskiej Vrije Universiteit Amsterdam, specjalizując się w dydaktyce informatyki, oraz nauczycielem w szkole średniej Lyceum Kralingen w Rotterdamie.

Felienne jest również twórczynią języka programowania Hedy dla początkujących programistów oraz gospodarzem podcastu Software Engineering Radio, jednego z największych podcastów o oprogramowaniu w sieci.



**O Ilustracji na Okładce**

Postać na okładce książki "The Programmer’s Brain" została podpisana jako "femme Sauvage du Canada", czyli dzika kobieta z Kanady. Ilustracja pochodzi z kolekcji kostiumów z różnych krajów autorstwa Jacques'a Grasset de Saint-Sauveur (1757-1810), zatytułowanej "Costumes civils actuels de tous les peuples connus", opublikowanej we Francji w 1788 roku. Każda ilustracja została starannie ręcznie rysowana i kolorowana. Bogactwo różnorodności kolekcji Grasset de Saint-Sauveur przypomina nam żywo, jak kulturowo odległymi były miasta i regiony świata zaledwie 200 lat temu. Odizolowani od siebie ludzie posługiwali się różnymi dialektami i językami. Na ulicach czy na wsi łatwo było zidentyfikować, skąd pochodzą i jakie zajęcie wykonują, po ich ubiorze.

Sposób, w jaki się ubieramy, zmienił się od tamtej pory, a różnorodność według regionów, tak bogata w tamtym czasie, zanikła. Teraz trudno jest odróżnić mieszkańców różnych kontynentów, nie mówiąc już o różnych miastach, regionach czy krajach. Być może zamieniliśmy różnorodność kulturową na bardziej zróżnicowane życie osobiste, z pewnością na bardziej zróżnicowane i szybkie życie technologiczne.

W czasach, gdy trudno odróżnić jedną książkę komputerową od drugiej, Manning celebruje inwencję i inicjatywę branży komputerowej okładkami opartymi na bogactwie różnorodności życia regionalnego sprzed dwóch wieków, ożywionymi przez obrazy Grasset de Saint-Sauveura.

## Część 1: Jak czytać kod lepiej

Czytanie kodu to kluczowy element programowania, ale jako zawodowy programista może się okazać, że nie wiesz, jak to zrobić. Czytanie kodu rzadko jest nauczane lub praktykowane, a zaznajamianie się z kodem bywa mylące i często wymaga wysiłku. Pierwsze rozdziały tej książki pomogą ci zrozumieć, dlaczego czytanie kodu jest takie trudne i co możesz zrobić, aby stać się w tym lepszym.

### 1. Rozszyfrowywanie swojego zamieszania podczas kodowania

Ten rozdział obejmuje:
- Rozróżnianie różnych sposobów, w jakie możesz być zdezorientowany podczas kodowania
- Porównywanie trzech różnych procesów poznawczych, które odgrywają rolę podczas kodowania
- Zrozumienie, w jaki sposób różne procesy poznawcze uzupełniają się nawzajem

Zamieszanie to integralna część programowania. Kiedy uczysz się nowego języka programowania, pojęcia lub frameworka, nowe pomysły mogą cię przerażać. Czytając nieznany kod lub kod napisany dawno temu, możesz nie zrozumieć, co robi kod lub dlaczego został napisany w ten sposób. Za każdym razem, gdy zaczynasz pracować w nowej dziedzinie biznesowej, nowe terminy i żargon mogą kolidować w twoim umyśle.

Oczywiście, przez pewien czas być zdezorientowanym nie stanowi problemu, ale nie chcesz być zdezorientowany dłużej, niż to konieczne. Ten rozdział uczy cię rozpoznawania i rozszyfrowywania swojego zamieszania. Być może nigdy o tym nie myślałeś, ale istnieje różne sposoby bycia zdezorientowanym. Nieznajomość znaczenia pojęcia z dziedziny to inny rodzaj zamieszania niż próba czytania skomplikowanego algorytmu krok po kroku.

Różne rodzaje zamieszania związane są z różnymi rodzajami procesów poznawczych. Za pomocą różnych przykładów kodu, ten rozdział omówi trzy różne rodzaje zamieszania i wyjaśni, co dzieje się w twoim umyśle.

Pod koniec tego rozdziału będziesz w stanie rozpoznać różne sposoby, w jakie kod może powodować zamieszanie, oraz zrozumiesz procesy poznawcze zachodzące w twoim mózgu w każdym przypadku. Gdy już poznasz trzy różne rodzaje zamieszania i związane z nimi trzy procesy poznawcze, kolejne rozdziały nauczą cię, jak poprawić te procesy poznawcze.

#### 1.1 Różne rodzaje zamieszania w kodzie

Całkowicie nowy kod jest pewnym stopniem zdezorientowania, ale nie wszystki kod jest zdezorientowujący w ten sam sposób. Ilustrujmy to trzema różnymi przykładami kodu. Wszystkie trzy przykłady przekształcają daną liczbę N lub n na postać binarną. Pierwszy program jest napisany w APL, drugi w Javie, a trzeci w BASIC.

Daj sobie kilka minut, aby dokładnie przeanalizować te programy. Na jakiej wiedzy polegasz, czytając je? Jak się to różni dla trzech programów? Być może w tym momencie nie masz jeszcze słów, aby wyrazić, co dzieje się w twoim mózgu, gdy czytasz te programy, ale przypuszczam, że dla każdego z nich będzie to odczuwane inaczej. Pod koniec tego rozdziału będziesz miał słownictwo, aby omawiać różne procesy poznawcze zachodzące, gdy czytasz kod.

Przykład w listingu 1.1 to program przekształcający liczbę N w reprezentację binarną w APL. Chyba że jesteś matematykiem z lat 60., prawdopodobnie nigdy nie korzystałeś z APL (języka programowania). Został on zaprojektowany specjalnie do operacji matematycznych i praktycznie nigdzie nie jest używany dzisiaj. Jak widzisz, ten program jest bardzo zwarty i zupełnie nieintuicyjny dla nie wtajemniczonych.

Listing 1.1 Reprezentacja binarna w APL

```apl
2 2 2 2 2 ⊤ n
```

Drugi przykład to program w języku Java, który również zamienia liczbę `n` na reprezentację binarną. W tym przypadku program używa metody Java `toBinaryString()` do przeprowadzenia transformacji.

Listing 1.2 Reprezentacja binarna w Javie

```java
public class BinaryCalculator { 
   public static void main(Integer n) {
      System.out.println(Integer.toBinaryString(n)); 
   }
}
```

Ostatni przykład to kolejny program, tym razem w języku BASIC, który zamienia liczbę `N` na reprezentację binarną. Ten kod implementuje algorytm obejmujący kilka kroków w celu przekształcenia liczby.

Listing 1.3 Reprezentacja binarna w BASIC

```basic
LET N2 =  ABS (INT (N))
LET B$ = ""
FOR N1 = N2 TO 0 STEP 0
     LET N2 =  INT (N1 / 2)
     LET B$ =  STR$ (N1 - N2 * 2) + B$
     LET N1 = N2
 NEXT N1
 PRINT B$
 RETURN
```

##### 1.1.1 Rodzaj dezorientacji 1: Brak wiedzy

Teraz przyjrzyjmy się, co dzieje się podczas czytania trzech programów. Po pierwsze mamy program APL. Zauważ, jak program używa operatora T. Dezorientacja polega tutaj na tym, że możesz nie wiedzieć, co oznacza T.

Listing 1.4 Reprezentacja binarna w APL

```apl
2 2 2 2 2 ⊤ n
```

Nie możesz zrozumieć tego programu, nie znając operatora T. Stąd dezorientacja wynika tutaj z braku wiedzy.



##### 1.1.2 Rodzaj dezorientacji 2: Brak informacji
Dla drugiego programu, źródło dezorientacji jest inne. Zakładam, że przy pewnym stopniu znajomości programowania, nawet jeśli nie jesteś ekspertem w Javie, twoje umysł może znaleźć odpowiednie części programu Javy. Program opiera się na konkretnej metodzie Javy. Dezorientację można spowodować tutaj, nie znając szczegółów działania toBinaryString().

Listing 1.5 Reprezentacja binarna w Javie

```java
public class BinaryCalculator { 
    public static void main(Integer n) {
       System.out.println(Integer.toBinaryString(n));
    }
}
```

Nawet jeśli zgadniesz funkcjonalność na podstawie nazwy metody, nie będziesz w stanie głęboko zrozumieć, co robi kod, chyba że nawigujesz do definicji toBinaryString() gdzie indziej w kodzie i kontynuujesz czytanie tam. Ponadto, z samego tego listingu nie jest jasne, gdzie dokładnie znajdziesz potrzebną definicję. Stąd problemem tutaj jest brak informacji.

##### 1.1.3 Rodzaj dezorientacji 3: Brak mocy obliczeniowej
W trzecim programie, na podstawie nazw zmiennych i operacji, możesz przypuszczać, co robi kod. Ale jeśli naprawdę chcesz śledzić, nie jesteś w stanie przetworzyć całego wykonania w swoim umyśle. Ten program w BASIC jest dezorientujący, ponieważ nie możesz obejrzeć wszystkich małych kroków wykonywanych w umyśle. Jeśli musisz zrozumieć wszystkie kroki, możesz skorzystać z pomocy pamięci, takiej jak wartości pośrednie zmiennych pokazane na rysunku 1.1.

![CH01_F01_Hermans2](https://drek4537l1klr.cloudfront.net/hermans2/HighResolutionFigures/figure_1-1.png)

Dezorientacja w tym przypadku wynika z braku mocy obliczeniowej. Trudno jest utrzymać w umyśle jednocześnie wszystkie wartości pośrednie zmiennych i odpowiadające im działania. Jeśli naprawdę chcesz mentalnie obliczyć, co robi ten program, prawdopodobnie sięgniesz po długopis i kartkę, by naszkicować kilka wartości pośrednich, lub nawet zapiszesz je obok linii w fragmencie kodu, jak pokazano na tym przykładzie.

W tych trzech programach zauważyliśmy, że dezorientacja, choć zawsze irytująca i niewygodna, może mieć trzy różne źródła. Po pierwsze, dezorientacja może wynikać z braku wiedzy na temat języka programowania, algorytmu lub dziedziny, nad którą pracujesz. Jednak dezorientację może również wywołać brak pełnego dostępu do wszystkich informacji, których potrzebujesz do zrozumienia kodu. Zwłaszcza że współczesny kod często korzysta z różnych bibliotek, modułów i pakietów, zrozumienie kodu może wymagać rozległej nawigacji, podczas której musisz pozyskiwać nowe informacje, jednocześnie pamiętając, co właściwie robisz. Wreszcie czasami kod jest bardziej skomplikowany, niż twój umysł jest w stanie przetworzyć, i to, co cię dezorientuje, to brak mocy obliczeniowej.

Teraz przejdźmy do różnych procesów poznawczych związanych z każdym z tych trzech rodzajów dezorientacji.

#### 1.2 Różne procesy poznawcze wpływające na kodowanie

Przyjrzyjmy się trzem różnym procesom poznawczym, które zachodzą w twoim mózgu, gdy czytasz trzy przykładowe programy. Jak już wspomniano, różne formy dezorientacji są związane z problemami różnych procesów poznawczych, wszystkich związanych z pamięcią. Są one wyjaśnione w dalszej części rozdziału w bardziej szczegółowy sposób.

Brak wiedzy oznacza, że w twojej pamięci długotrwałej (LTM), miejscu, w którym przechowywane są trwale wszystkie twoje wspomnienia, nie ma wystarczającej liczby istotnych faktów. Natomiast brak informacji stanowi wyzwanie dla twojej pamięci krótkotrwałej (STM). Informacje, które pozyskujesz, muszą być przechowywane tymczasowo w STM, ale jeśli musisz szukać w wielu różnych miejscach, możesz zapomnieć o niektórych rzeczach, które już przeczytałeś. Wreszcie, gdy musisz przetwarzać dużo informacji, to wpływa to na pamięć operacyjną, czyli miejsce, gdzie zachodzi twoje myślenie. Te trzy procesy poznawcze nie występują tylko podczas czytania kodu, ale we wszystkich działaniach poznawczych, w tym (w kontekście programowania) pisaniu kodu, projektowaniu architektury systemu czy pisaniu dokumentacji.

##### 1.2.1 Pamięć długotrwała (LTM) a programowanie

Pierwszym procesem poznawczym wykorzystywanym podczas programowania jest pamięć długotrwała (LTM). Może przechowywać twoje wspomnienia przez bardzo długi czas. Większość ludzi potrafi przypomnieć sobie wydarzenia sprzed lat, a nawet dekad. Twoja pamięć długotrwała odgrywa rolę we wszystkim, co robisz, od zawiązywania sznurówek, gdzie twoje mięśnie pamiętają, co robić niemal automatycznie, po napisanie wyszukiwania binarnego, gdzie pamiętasz abstrakcyjny algorytm, składnię języka programowania i sposób pisania na klawiaturze. W rozdziale 3 zostanie szczegółowo omówione wykorzystanie pamięci długotrwałej, obejmując różne formy zapamiętywania i sposoby wzmacniania tego procesu poznawczego.

Twoja pamięć długotrwała przechowuje kilka rodzajów istotnych informacji programistycznych. Na przykład może przechowywać wspomnienia, kiedy skutecznie zastosowałeś określoną technikę, znaczenie słów kluczowych w języku Java, znaczenie słów w języku angielskim czy też fakt, że maxint w Javie to 2147483647.

Pamięć długotrwała można porównać do dysku twardego komputera, przechowującego fakty przez długi czas.

###### Program w APL: LTM

Przy czytaniu programu w APL najbardziej korzystasz z pamięci długotrwałej (LTM). Jeśli znasz znaczenie słowa kluczowego APL - T, odzyskasz to z pamięci długotrwałej, czytając ten program.

Program w APL ilustruje także znaczenie wiedzy o odpowiedniej składni. Jeśli nie wiesz, co oznacza T w APL, będziesz miał bardzo trudności z zrozumieniem programu. Z drugiej strony, jeśli wiesz, że oznacza to dwuargumentową funkcję kodowania, która przekształca wartość na inny sposób liczbowy, czytanie programu jest niemal trywialne. Nie trzeba rozumieć żadnych słów, i nie trzeba krok po kroku rozkładać działania kodu.

##### 1.2.2 STM i programowanie
Drugim procesem kognitywnym związanym z programowaniem jest STM. STM służy do krótkotrwałego przechowywania przychodzących informacji. Na przykład, gdy ktoś odczytuje ci numer telefonu przez telefon, nie trafia on od razu do twojej LTM. Numer telefonu trafia najpierw do STM, która ma ograniczoną pojemność. Szacunki różnią się, ale większość naukowców zgadza się, że do STM mieści się tylko kilka elementów, z pewnością nie więcej niż kilkanaście.

Na przykład, czytając program, słowa kluczowe, nazwy zmiennych i używane struktury danych są tymczasowo przechowywane w STM.

##### Program Java: STM

W przypadku programu Java głównym procesem kognitywnym jest STM. Najpierw przetwarzasz linię 1 z listingu 1.6, która informuje cię, że parametr wejściowy n funkcji to liczba całkowita. W tym momencie nie wiesz jeszcze, co funkcja będzie robić, ale możesz kontynuować czytanie, jednocześnie pamiętając, że n to liczba. Wiedza, że n to liczba całkowita, jest przechowywana w STM przez pewien czas. Następnie przechodzisz do linii 2, gdzie toBinaryString() wskazuje, co funkcja zwróci. Być może nie będziesz pamiętać tej funkcji za dzień, a nawet za godzinę. Gdy twoje mózg rozwiąże problem, w tym przypadku zrozumienie funkcji, STM zostaje opróżniona.

Chociaż STM odgrywa dużą rolę w zrozumieniu tego programu, LTM również jest zaangażowana w czytanie tego programu. W rzeczywistości nasza LTM jest zaangażowana we wszystko, co robimy. Na przykład, jeśli znasz język Java, jak zakładam, że większość czytelników, wiesz, że słowa kluczowe public class i public static void main można zignorować, jeśli zostaniesz poproszony o wyjaśnienie, co funkcja robi. Prawdopodobnie nawet nie zauważyłeś, że metoda nazywa się "mian" a nie "main".

Twój mózg zrobił skrót, zakładając nazwę, pokazując mieszanie dwóch procesów kognitywnych. Zdecydował się użyć "main" na podstawie wcześniejszych doświadczeń przechowywanych w twojej LTM, zamiast używać rzeczywistej nazwy, którą przeczytałeś i którą przechowywano w twoim STM. To pokazuje, że te dwa procesy kognitywne nie są tak odseparowane od siebie, jak przedstawiałem. Jeśli LTM jest jak twardy dysk twojego mózgu, przechowujący wspomnienia na zawsze, możesz myśleć o STM jak o pamięci RAM komputera lub pamięci podręcznej, którą można używać do tymczasowego przechowywania wartości.



**1.2.3 Pamięć operacyjna a programowanie**
Trzecim procesem poznawczym, który odgrywa rolę w programowaniu, jest pamięć operacyjna. STM i LTM są głównie urządzeniami przechowującymi. Przechowują informacje, czy to przez krótki czas po przeczytaniu czy usłyszeniu ich, w przypadku STM, czy przez długi czas, w przypadku LTM. Rzeczywiste myślenie jednak nie zachodzi w LTM ani STM, ale w pamięci operacyjnej. To tutaj powstają nowe myśli, pomysły i rozwiązania. Jeśli myślimy o LTM jak o twardym dysku mózgu, przechowującym wspomnienia na długi okres, a STM jak o RAM, tymczasowo trzymającym najnowsze informacje, to pamięć operacyjna najlepiej porównuje się do procesora mózgu.

###### Program w BASIC: Pamięć operacyjna

Podczas czytania programu w BASIC używasz swojej LTM, na przykład, gdy pamiętasz znaczenie słów kluczowych, takich jak LET czy EXIT. Ponadto korzystasz ze swojej STM, aby przechowywać niektóre informacje, takie jak fakt, że B$ zaczyna się jako pusty ciąg znaków.

Jednak twoje myśli pracują znacznie bardziej, gdy czytasz program w BASIC. Próbujesz mentalnie wykonać kod, zrozumieć, co się dzieje. Ten proces nazywa się śledzeniem - mentalnym kompilowaniem i wykonywaniem kodu. Część mózgu używana do śledzenia i innych złożonych zadań poznawczych to pamięć operacyjna. Możesz ją porównać do procesora komputera, który wykonuje obliczenia.

Podczas śledzenia bardzo skomplikowanych programów możesz odczuwać potrzebę notowania wartości zmiennych, zarówno w kodzie, jak i w osobnej tabeli. Fakt, że twoje myśli czują potrzebę przechowywania informacji na zewnątrz, może być sygnałem, że twoja pamięć operacyjna jest zbyt pełna, aby przetworzyć więcej informacji. Omówimy ten problem przeładowania informacji i jak zapobiegać przeciążeniu mózgu w rozdziale 4.

Oto szybkie podsumowanie, jak różne rodzaje dezorientacji są związane z różnymi procesami poznawczymi:

Brak wiedzy = Problem w LTM
Brak informacji = Problem w STM
Brak mocy obliczeniowej = Problem w pamięci operacyjnej



#### 1.3 Procesy poznawcze w współpracy**

W poprzednim rozdziale szczegółowo opisałem trzy ważne procesy poznawcze, które są istotne dla programowania. W skrócie, twoja LTM przechowuje informacje, które zdobyłeś na przestrzeni długiego czasu, STM tymczasowo przechowuje informacje, które właśnie przeczytałeś lub usłyszałeś, a pamięć operacyjna przetwarza informacje i formuje nowe myśli. Chociaż opisałem je jako oddzielne procesy, te procesy poznawcze mają silne związki między sobą. Porozmawiajmy o tym, jak się wzajemnie oddziałują.

##### 1.3.1 Krótka analiza tego, jak procesy poznawcze oddziaływały**

W rzeczywistości wszystkie trzy procesy poznawcze są aktywowane do pewnego stopnia, gdy myślisz o czymś, jak to zilustrowano na rysunku 1.2. Być może doświadczyłeś wszystkich trzech procesów świadomie, gdy czytałeś wcześniej, w tym rozdziale, fragment kodu Java (listing 1.2). Niektóre informacje zostały przechowane w twojej STM, na przykład, gdy przeczytałeś, że n było liczbą całkowitą. Jednocześnie twój mózg odzyskał znaczenie tego, co to jest liczba całkowita, z twojej LTM, a ty myślałeś o znaczeniu programu, używając swojej pamięci operacyjnej.

Do tej pory w tym rozdziale skupiliśmy się szczególnie na procesach poznawczych, które zachodzą, gdy czytasz kod. Jednak te trzy procesy poznawcze są zaangażowane w wiele innych zadań także związanych z programowaniem.

![CH01_F02_Hermans2](https://drek4537l1klr.cloudfront.net/hermans2/HighResolutionFigures/figure_1-2.png)

##### 1.3.2 Procesy poznawcze dotyczące zadań programistycznych

Na przykład, rozważ sytuację, gdy czytasz raport o błędzie od klienta. Błąd wydaje się być spowodowany błędem o jedno miejsce  (off-by-one error). Raport o błędzie dostaje się do mózgu przez twoje zmysły - twoje oczy, jeśli widzisz, lub uszy, jeśli czytasz za pomocą czytnika ekranowego. Aby rozwiązać problem, musisz ponownie przeczytać kod, który napisałeś kilka miesięcy temu. Podczas ponownego czytania kodu twoje STM przechowuje to, co przeczytałeś, podczas gdy twoja LTM mówi ci o tym, co zaimplementowałeś kilka miesięcy temu - na przykład, że używałeś wtedy modelu aktora. Oprócz wspomnień o twoich doświadczeniach, masz także przechowywane w LTM informacje faktyczne, na przykład jak można rozwiązać błąd o jedno miejsce. Wszystkie te informacje - nowe informacje o raporcie o błędzie z STM, twoje osobiste wspomnienia i istotne fakty dotyczące rozwiązania podobnych błędów z LTM - trafiają do twojej pamięci operacyjnej, gdzie możesz myśleć o problemie.

> Błąd o jedno miejsce (off-by-one error) to rodzaj błędu programistycznego, w którym indeks, licznik lub inny odnośnik jest przesunięty o jedno miejsce od oczekiwanej pozycji. To oznacza, że program używa nieprawidłowego indeksu lub liczby, co prowadzi do błędnego dostępu do pamięci lub innego rodzaju problemów.

###### Ćwiczenie 1.1

Aby praktykować swoje nowo zdobyte zrozumienie trzech procesów poznawczych związanych z programowaniem, przygotowałem trzy programy. Tym razem jednak nie podano wyjaśnień, co robią fragmenty kodu. Musisz zatem przeczytać programy i samodzielnie zdecydować, co robią. Programy są ponownie napisane w APL, Java i BASIC, w tej kolejności. Jednak każdy z programów wykonuje inną operację, więc nie możesz polegać na zrozumieniu pierwszego programu, aby wspierać czytanie pozostałych programów.

Starannie przeczytaj programy i spróbuj określić, co robią. Przy tym zastanów się nad mechanizmami, których używasz. Skorzystaj z pytań zawartych w poniższej tabeli, aby kierować analizą własną.

![img](https://drek4537l1klr.cloudfront.net/hermans2/Figures/EXERCISE_1_1.png)

Kod fragmentu 1: Program w APL

```apl
f • {⍵≤1:⍵ ⋄ (∇ ⍵-1)+∇ ⍵-2}
```

Co robi ten kod? Jakie procesy poznawcze są zaangażowane?

Kod fragmentu 2: Program w Java

```java
public class Luhn {
   public static void main(String[] args) {
       System.out.println(luhnTest("49927398716"));
   }
 
   public static boolean luhnTest(String number){
       int s1 = 0, s2 = 0;
       String reverse = new StringBuffer(number).reverse().toString();
       for(int i = 0 ;i < reverse.length();i++){
           int digit = Character.digit(reverse.charAt(i), 10);
           if(i % 2 == 0){//this is for odd digits
               s1 += digit;
           }else{//add 2 * digit for 0-4, add 2 * digit - 9 for 5-9
               s2 += 2 * digit;
               if(digit >= 5){
                   s2 -= 9;
               }
           }
       }
       return (s1 + s2) % 10 == 0;
   }
}
```

Co robi ten kod? Jakie procesy poznawcze są zaangażowane?

Kod fragmentu 3: Program w BASIC

```basic
100 INPUT PROMPT "String: ":TX$
120 LET RES$=""
130 FOR I=LEN(TX$) TO 1 STEP-1
140 LET RES$=RES$&TX$(I)
150 NEXT
160 PRINT RES$
```

Co robi ten kod? Jakie procesy poznawcze są zaangażowane?

##### Podsumowanie

Aby rozwiązać zagubienie, najpierw musisz zidentyfikować jego źródło. Zagubienie podczas kodowania zazwyczaj wynika z trzech problemów: braku wiedzy, braku łatwo dostępnych informacji lub braku mocy obliczeniowej w mózgu.
Trzy procesy poznawcze są zaangażowane podczas czytania lub pisania kodu: długotrwała pamięć (LTM), krótkotrwała pamięć (STM) i pamięć operacyjna.
LTM przechowuje wiedzę, do której może być potrzebny dostęp po dłuższym czasie. Na przykład w LTM przechowywane są znaczenia słów kluczowych.
STM może tymczasowo przechowywać informacje, takie jak nazwa metody lub zmiennej.
Pamięć operacyjna to miejsce aktywnego przetwarzania. W kontekście kodu może to być zadania takie jak stwierdzenie, że indeks jest o jeden za niski.
Wszystkie trzy procesy poznawcze pracują, gdy czytasz kod, i procesy te się uzupełniają. Na przykład, gdy STM napotyka nazwę zmiennej, takiej jak n, mózg przeszukuje LTM w poszukiwaniu związanych programów, które czytałeś w przeszłości. A gdy czytasz niejednoznaczne słowo, aktywowana jest pamięć operacyjna, a mózg stara się zdecydować, jakie jest właściwe znaczenie w danym kontekście.

### Rozdział 2: Szybkie czytanie kodu

Ten rozdział obejmuje:
- Analizę dlaczego szybkie czytanie kodu jest trudne nawet dla doświadczonego programisty.

- Rozkładanie na czynniki, jak mózg dzieli nowe informacje na rozpoznawalne części.

- Poznanie, jak LTM (długa pamięć) i STM (krótka pamięć) współpracują podczas analizowania informacji, takich jak słowa czy kod.

- Zbadanie roli pamięci ikonicznej podczas przetwarzania kodu.

- Wyjaśnienie, jak zapamiętywanie kodu może być używane jako narzędzie do (auto)diagnozy poziomu umiejętności programistycznych.

  ​	

​	Praktykę w pisaniu kodu, który jest łatwiejszy do przeczytania przez innych.
Rozdział pierwszy przedstawił trzy procesy poznawcze, które odgrywają rolę podczas programowania i czytania kodu. Pierwszym procesem poznawczym był LTM, który można porównać do dysku twardego przechowującego wspomnienia i fakty. Drugim procesem poznawczym był STM, który działa jak pamięć RAM, przechowując informacje na krótki czas. Ostatecznie mamy pamięć operacyjną, która działa trochę jak procesor i przetwarza informacje z LTM i STM, aby dokonywać myślenia.