Zaczynamy tutaj
## Oto jak działa Git dla tworzenia aplikacji iOS

Kurs podzielony jest na moduły, z których każdy zawiera dwa rodzaje lekcji:

    Lekcje wideo na temat teoretycznych koncepcji, wyjaśniające podstawy Gita. Ważne jest, aby najpierw obejrzeć te materiały wideo, zanim przejdziesz do lekcji praktycznych. Wszystkie filmy są dostępne do pobrania w formie slajdów i transkryptów, co ułatwia śledzenie ich treści.
    Lekcje praktyczne, które pokazują zastosowanie procesów i najlepszych praktyk na przykładzie przykładowej aplikacji. Istnieje również pełny projekt Xcode do pobrania, zawierający wszystkie wykonane w kursie commity.

Jak w pełni wykorzystać ten kurs

Podobnie jak w przypadku wszystkich działań związanych z tworzeniem aplikacji iOS, Git jest narzędziem praktycznym, dlatego zalecam praktyczne stosowanie tego, czego się uczysz, zamiast traktowania kursu jako czystą rozrywkę intelektualną. Możesz to zrobić na dwa sposoby:

    Powtórzanie krok po kroku tego, co jest wyjaśniane w lekcjach praktycznych; lub
    Zastosowanie omawianych koncepcji do projektu, nad którym aktualnie pracujesz.

Wiele kursów związanych z Gitem najpierw pokazuje podstawowe komendy, a następnie wraca do omówienia każdej koncepcji. Uważam, że taka metoda wprowadza te koncepcje w niewłaściwej kolejności. Na przykład, często na początku musisz sklonować całe repozytorium, gdy jeszcze nie znasz zbyt dobrze działania Gita.

W tym kursie stosuję inne podejście.

Najpierw uczę, jak pracować z Gitem lokalnie na pojedynczej linii czasu. Pozwala mi to natychmiast omówić komendy i procesy, które możesz od razu zastosować. Ponadto, ułatwią one pracę z gałęziami i zdalnymi repozytoriami.

Mam nadzieję, że ten kurs pomoże Ci wykorzystać Gita do wsparcia Twojej codziennej pracy. Uważam, że poprawne korzystanie z Gita pozwoli Ci szybciej rozwijać swoje aplikacje.

Jeśli masz trudności z zrozumieniem jakiejkolwiek części kursu, daj mi znać. Jestem tutaj, aby ulepszyć kurs dzięki Twojej pomocy.



## Uczenie się drobnych szczegółów każdej komendy Git jest bezużyteczne bez dobrych praktyk.

Git to potężne narzędzie, a Xcode posiada przydatną integrację, która często ułatwia wykonywanie zadań. W każdej lekcji skupię się najpierw na tej integracji. Ponieważ jako deweloper iOS często będziesz używać Gita wewnątrz Xcode.

Mimo to, Git wyraża swoją pełną moc tylko w wierszu poleceń.

Ale zapamiętywanie każdej komendy Git miałoby niewielki sens. Git jest narzędziem, które ma ułatwić Ci życie podczas wykonywania konkretnych zadań. Git nie jest celem samym w sobie, ale środkiem.

To oznacza, że najpierw musisz nauczyć się przypadków użycia, procesów pracy i dobrych praktyk związanych z Gitem. O to właśnie chodzi w tym kursie. Będziemy omawiać Gita, analizując, jak wykonywać konkretne zadania w cyklu rozwoju aplikacji iOS.

Tylko wtedy komendy Gita nabiorą sensu.

Gdy Git stanie się częścią Twojego codziennego procesu pracy, możesz chcieć go używać do bardziej konkretnych funkcji. Wtedy przyda się odniesienie do Gita. Możesz z niego skorzystać, aby poznać mnóstwo opcji, jakie oferuje każda komenda Git, aby uzyskać oczekiwany wynik.
Wybór odpowiedniego klienta interfejsu graficznego Git do Twoich potrzeb.

Mimo że Git jest potężnym narzędziem, jego wizualne wyjście oparte na tekście może czasami być trudne do analizy wizualnej. Jak mówi przysłowie, obraz jest wart tysiąca słów.

Integracja Xcode oferuje pewne wsparcie wizualne, które ułatwia wykonywanie konkretnych zadań. Jednak jest dość ograniczona i wyświetla tylko ułamek tego, czego Git może dokonać.

Na rynku dostępnych jest kilka klientów graficznych (GUI) dla Gita, zarówno darmowych, jak i płatnych. Każdy z nich ma inne podejście. Dlatego zalecam, abyś najpierw nauczył się korzystać z funkcji Gita z wiersza poleceń. Dopiero wtedy będziesz wiedział, jak Git integruje się z Twoimi codziennymi procesami pracy, i będziesz w najlepszej pozycji, aby zdecydować, który klient GUI najlepiej odpowiada Twoim potrzebom.

## Jednorazowa konfiguracja Gita: Ustawienie tożsamości i preferencji edytora

Aby korzystać z Gita, najpierw musisz go zainstalować na swoim komputerze i skonfigurować kilka podstawowych parametrów.

Na szczęście musisz to zrobić tylko raz. Jest to dość proste, więc możesz to zrobić szybko i zapomnieć o tym.

Spis treści

1. Instalowanie Gita na komputerze Mac, aby go używać w terminalu
2. Konfigurowanie swojej tożsamości, aby podpisywać swoje zmiany, gdy pracujesz zespołowo lub publikujesz kod open-source
3. Zmiana domyślnego edytora tekstu w Gicie, aby uniknąć utknięcia w Vimie

### Aby zainstalować Gita na komputerze Mac i móc go używać w terminalu, wykonaj następujące kroki:

1. Git jest już wbudowany w Xcode. Masz już wszystko, czego potrzebujesz, jeśli korzystasz z Gita tylko poprzez integrację z Xcode. Jednak wcześniej zaleciłem zapoznanie się z Gitem w wierszu poleceń od samego początku, a nie dopiero w późniejszym czasie. Zacznijmy to robić w tej lekcji.

2. Przy okazji, chociaż słowo "terminal" wskazuje na fizyczną maszynę, obecnie jest również synonimem wiersza poleceń. Będę używał tych dwóch terminów zamiennie w trakcie kursu.

3. Aby używać Gita w terminalu, musisz zainstalować narzędzia wiersza poleceń dla Xcode. Możesz pobrać pakiet .dmg zawierający te narzędzia ze strony internetowej Apple. Kliknij przycisk "Pobierz" w prawym górnym rogu strony.

Wykonaj te kroki, aby zainstalować Gita na swoim komputerze Mac i móc go używać w terminalu.

https://developer.apple.com/xcode/



Następnie przewiń stronę w dół aż znajdziesz odpowiedni pakiet dla swojej wersji Xcode lub wpisz wyszukiwaną frazę w polu wyszukiwania, aby przefiltrować wyniki na stronie.

Możesz również zainstalować narzędzia wiersza poleceń dla Xcode za pomocą terminala. Jest to szczególnie przydatne na maszynach, do których można się połączyć tylko zdalnie za pośrednictwem wiersza poleceń i na których nie ma dostępu do interfejsu graficznego.

Otwórz aplikację Terminal i wpisz:

xcode-select --install

Nie musisz instalować Xcode'a przed wykonaniem tego polecenia. Jest ono dostępne nawet na świeżo zainstalowanym systemie macOS.



Konfiguracja tożsamości jest kolejnym krokiem, który musisz wykonać, aby podpisywać swoją pracę, gdy pracujesz w zespole lub publikujesz kod jako open-source.

Przejdź do menu Xcode -> Preferencje i następnie przejdź do zakładki "Source Control".

W sekcji "Ogólne" sprawdź opcję "Włącz kontrolę wersji", jeśli nie jest już zaznaczona, oraz zaznacz wszystkie dostępne opcje. Dodatkowo, zaznacz opcję "Pokaż zmiany w kontroli wersji", która okaże się przydatna w przyszłości.

![img](https://iosfoundations.com/wp-content/uploads/2022/09/general-section.jpeg)

W sekcji "Git" możesz skonfigurować swoje imię i adres e-mail.

Tutaj również możesz zmienić domyślną nazwę głównej gałęzi (Default Branch Name) na dowolną, co zostanie omówione w późniejszych lekcjach.

Chociaż Xcode domyślnie ustawia nazwę na "main", ja zawsze zmieniam ją z powrotem na "master", który jest standardową nazwą głównej gałęzi w repozytorium Git używaną przez miliony programistów. Możesz przeczytać dlaczego w ostatniej sekcji tej lekcji.

Mimo to, możesz zmienić nazwę głównej gałęzi (master) w repozytorium na cokolwiek innego. Jest to gałąź jak każda inna i nie ma nic specjalnego w jej nazewnictwie.



![img](https://iosfoundations.com/wp-content/uploads/2022/09/git-section.jpeg)



Możesz również skonfigurować swoją tożsamość i domyślną nazwę gałęzi w terminalu. Musisz to zrobić tylko raz, więc jeśli już to zrobiłeś w Xcode, wystarczy.

```bash
git config --global user.name "Matteo Manferdini"
git config --global user.email matteo@purecreek.com
git config --global init.defaultBranch master
```

Zmiana domyślnego edytora tekstu w Git, aby nie utknąć w środku Vim-a.

Podczas korzystania z Git-a z wiersza poleceń, będą sytuacje, w których będziesz musiał edytować pewien tekst. W takich przypadkach Git otworzy domyślny edytor tekstu (na macOS jest to Vim).

Vim to potężny edytor, ale będziesz musiał pokonać stromą krzywą uczenia się, zanim go opanujesz. Jeśli przypadkowo znajdziesz się w Vim-ie i nie wiesz, jak go używać, będziesz mieć trudności z wyjściem z niego.

Ja osobiście korzystam z Vim-a z Git-em, a zrzuty ekranu w tym kursie odzwierciedlają ten wybór.

Masz dwie opcje:
1. Naucz się używać Vim-a.

Nauczenie się podstaw Vim-a nie zajmuje wiele czasu i można znaleźć wiele darmowych samouczków online. Poznanie Vim-a nie będzie przydatne tylko dla Git-a. Będzie to cenna umiejętność, jeśli kiedykolwiek będziesz łączyć się z zdalnym serwerem przez terminal.

2. Zmień domyślny edytor Git-a na TextEdit lub inny łatwy w użyciu edytor.

Git nie jest powiązany z żadnym konkretnym edytorem tekstu, więc możesz użyć dowolnego edytora.

Na macOS popularnym wyborem jest aplikacja TextEdit. Wpisz następującą komendę w terminalu, aby ustawić ją jako domyślny edytor Git-a:

```bash
git config --global core.editor /Applications/TextEdit.app/Contents/MacOS/TextEdit
```

Domyślne zachowanie Git-a polega na czekaniu, aż zakończysz edycję tekstu w edytorze, aby kontynuować wykonanie komendy, która spowodowała jego otwarcie. Oznacza to, że musisz zakończyć działanie aplikacji TextEdit, aby Git kontynuował. Może to być uciążliwe, jeśli masz otwarte inne dokumenty.

Jeśli zamykanie innych dokumentów może być dla Ciebie problematyczne, możesz wciąż ustawić TextEdit jako domyślny edytor bez potrzeby zamykania ich za pomocą tego sztucznego triku:

```bash
git config --global core.editor "open -W -n"
```

Spowoduje to otwarcie nowej instancji TextEdit, nawet jeśli jedna jest już uruchomiona.

 Oznacza to, że zobaczysz dwa ikony TextEdit w macOS Dock. Zamknięcie jednej aplikacji nie będzie miało wpływu na drugą.

Możesz nawet ustawić Xcode jako domyślny edytor tekstowy w Git, ale nie polecam tego. Xcode to dość ciężka aplikacja do otwierania tylko w celu edycji małego pliku tekstowego. Ponadto, często będziesz pracować w Xcode, a zamykanie go w celu wykonania polecenia Git może być zbyt uciążliwe.

Inne ustawienia, które mogą być przydatne dla dewelopera iOS

Wszystkie omówione powyżej ustawienia są tymi, których będziesz potrzebować teraz. Jednak Git ma kilka dodatkowych ustawień dla szczególnych przypadków.

Na przykład, domyślnym narzędziem do scalania (merge) w Git jest aplikacja Filemerge, która jest dostarczana wraz z Xcode. Pokażę Ci, jak z niej skorzystać w późniejszej części kursu, ale jeśli pobierzesz lub kupisz lepsze narzędzie, możesz zmienić konfigurację Git-a, aby je dostosować.

Na tej stronie możesz dowiedzieć się, jak to zrobić, oraz znaleźć inne przydatne ustawienia. Omówię te przydatne dla rozwoju aplikacji iOS, gdy pojawi się odpowiednia okazja w dalszej części kursu.

Standardowa nazwa głównej gałęzi w repozytorium

Historyczną domyślną nazwą głównej gałęzi w repozytorium jest "master".

Była to akceptowana nazwa przez prawie dwie dekady. Używana jest w milionach repozytoriów przez setki milionów programistów i jest domyślną nazwą, którą Git nadaje nowemu repozytorium utworzonemu za pomocą wiersza poleceń (choć możesz to zmienić w konfiguracji Git-a).

Ostatnio Apple i GitHub (należący do Microsoftu) zmieniły ją na "main" ze względów politycznych w swoich domyślnych ustawieniach.

Możesz uznać to za słuszne. To w porządku.

Możesz uważać, że to nie jest słuszne. To też w porządku.

Nie jestem tutaj, aby mówić Ci, co masz myśleć. Na ten temat napisano wiele zarówno po jednej, jak i po drugiej stronie. Jeśli masz zamiar, możesz przeczytać o tym online. Jestem tutaj tylko po to, aby powiedzieć Ci, że tak się stało, abyś mógł sam podjąć decyzję.

Niezależnie od tego, czy zgadzam się z przyczynami zmiany, uważam to za głęboko nieetyczne, żeby wielomiliardowe korporacje jednostronnie narzucały polityczne decyzje

 całej społeczności deweloperów iOS i ustalały standardy akceptowalności mowy za pomocą domyślnych ustawień oprogramowania lub kodeksów postępowania.

Choć możesz się zgodzić z tą konkretą decyzją, prawdopodobnie nie czułbyś tak samo, gdyby nałożyli na Ciebie coś, co stoi w sprzeczności z Twoimi przekonaniami.

Dlatego nazwij gałąź "master" lub "main". Dla Git-a nie ma to znaczenia.

Zawsze możesz zmienić jej nazwę później. Jednak uważaj przy zmianie nazwy istniejącej gałęzi, ponieważ skrypty budowania i systemy CI/CD poza Git-em mogą polegać na konkretnej nazwie.

To niestety muszę wspomnieć o tym w kursie, który powinien dotyczyć tylko tworzenia oprogramowania, ale milczenie na ten temat byłoby przemilczeniem.

Na szczęście jest to pierwszy i jedyny raz, kiedy muszę to zrobić przez prawie dziesięć lat tworzenia kursów. Mam nadzieję, że będzie to ostatni raz.



Tworzenie repozytorium Git: Istotny krok, aby rozpocząć każdy projekt

Pierwszą rzeczą, którą robię przy tworzeniu nowego projektu Xcode, jest utworzenie nowego repozytorium Git.

Nie ma znaczenia, czy to projekt osobisty, projekt dla klienta, przykład tworzony dla moich artykułów i kursów, czy po prostu prosty eksperyment.

W każdym przypadku rzadko zdarza się, aby projekt składał się tylko z kilku linii kodu. Często trzeba napisać znaczną ilość kodu, a przydatne jest śledzenie historii wszystkich zmian.

Wszystkie te przypadki mogą korzystać z gotowego do działania repozytorium Git. Ponadto, jest to niezwykle proste, nie kosztuje nic i nie ma żadnych wad. Nie ma więc powodu, aby tworzyć projekt bez repozytorium Git.

Spis treści

    Tworzenie repozytorium Git dla nowych lub istniejących projektów Xcode
    Znajdowanie i usuwanie repozytorium Git dla projektu Xcode
    Tworzenie repozytorium Git za pomocą wiersza poleceń
    Synchronizacja zmian Xcode z Git za pomocą wiersza poleceń
    Podstawy obsługi wiersza poleceń dla tych, którzy nigdy wcześniej nie korzystali z terminala
        Wiersz poleceń
        Wyświetlanie zawartości katalogu
        Przechodzenie między katalogami
        Otwieranie plików i katalogów
    Otwieranie folderu projektu w terminalu bezpośrednio z Xcode
        Tworzenie skryptu powłoki (shell script)
        Tworzenie niestandardowego zachowania w Xcode

Tworzenie repozytorium Git dla nowych lub istniejących projektów Xcode

Jako przykład roboczy w tym kursie posłużymy się małą aplikacją. Aplikacja będzie wyświetlać profil osoby, tak jak na platformie mediów społecznościowych.

Możemy zacząć od utworzenia projektu Xcode dla tej aplikacji.

![img](https://iosfoundations.com/wp-content/uploads/2022/09/create-project.jpeg)



Po skonfigurowaniu nazwy i opcji projektu, Xcode zapyta, gdzie chcesz go zapisać.

To jest moment, w którym możesz utworzyć repozytorium Git, więc upewnij się, że zaznaczysz pole wyboru Utwórz repozytorium Git na moim Macu.



![img](https://iosfoundations.com/wp-content/uploads/2022/09/create-repository.jpeg)



Aby dodać repozytorium Git do projektu, który jeszcze nie jest pod kontrolą wersji, wybierz Source Control -> New Git Repositories w menu Xcode.

W oknie dialogowym już zostanie wybrany Twój projekt Xcode. Wystarczy kliknąć przycisk Utwórz.

Aby sprawdzić, czy projekt Xcode jest objęty kontrolą wersji, przejdź do nawigatora Source Control znajdującego się po lewej stronie okna Xcode. Tam, w zakładce Repositories, zobaczysz swoje repozytorium i całą jego zawartość.

![img](https://iosfoundations.com/wp-content/uploads/2022/09/repositories.jpeg)



Mieć projekt Xcode objęty kontrolą wersji oznacza, że projekt jest śledzony przez system kontroli wersji, w tym przypadku Git. To oznacza, że wszystkie zmiany w plikach projektu są monitorowane i rejestrowane w repozytorium Git, umożliwiając zarządzanie historią zmian, cofanie się do wcześniejszych wersji projektu oraz pracę w zespołach.

Chociaż samo repozytorium Git jest ukryte i znajduje się w formie ukrytego folderu, można do niego łatwo przejść za pomocą programu Finder. Szybszym sposobem jest kliknięcie prawym przyciskiem myszy na plik projektu Xcode w nawigatorze projektu i wybranie polecenia "Pokaż w Finder" z menu kontekstowego.

![img](https://iosfoundations.com/wp-content/uploads/2022/09/show-in-finder.jpeg)

Domyślnie, w programie Finder nie są widoczne ukryte pliki. Aby przełączyć tę opcję, naciśnij klawisze Shift + Command + kropka (⇧+⌘+.) na klawiaturze. Wówczas pojawi się folder o nazwie .git. To właśnie tam znajduje się repozytorium projektu.

![img](https://iosfoundations.com/wp-content/uploads/2022/09/hidden.jpeg)



Zawartość tego folderu nie powinna Cię interesować, chyba że chcesz tworzyć własne narzędzia Git. Interakcję z Gitem będziesz przeprowadzał tylko za pośrednictwem Xcode lub interfejsu wiersza poleceń.

Nie ma możliwości usunięcia repozytorium Git z poziomu Xcode. Jeśli jednak kiedykolwiek będziesz musiał to zrobić, wystarczy usunąć folder .git. Bądź jednak ostrożny, ponieważ cała historia gita zostanie utracona.
Tworzenie repozytorium Git z wiersza poleceń

Tworzenie repozytorium z wiersza poleceń jest tak łatwe, jak przejście do folderu Twojego projektu i wpisanie polecenia git init.

Pierwsza część może być jednak trudna, jeśli nigdy wcześniej nie korzystałeś z terminala. Jeśli tak jest, zalecam zapoznanie się z podstawowymi komendami powłoki systemu macOS. W kolejnej sekcji tej lekcji przedstawię Ci podstawowe wprowadzenie. Możesz również poszukać tutoriali w Google, aby dowiedzieć się więcej.

Możesz otworzyć dowolny folder w terminalu, korzystając z programu Finder: kliknij prawym przyciskiem myszy na folderze i wybierz Usługi -> Nowy Terminal w folderze w menu kontekstowym.

![img](https://iosfoundations.com/wp-content/uploads/2022/09/new-terminal.jpeg)

Następnie możesz uruchomić polecenie git init.

![img](https://iosfoundations.com/wp-content/uploads/2022/09/git-init.jpeg)

Ostrzegam jednak, że istnieje różnica między inicjalizowaniem nowego repozytorium w Xcode, a za pomocą wiersza poleceń.

W pierwszym przypadku Xcode tworzy również pierwszy migawki (commit), zawierający wszystkie początkowe pliki projektu. Natomiast każde repozytorium rozpoczęte za pomocą wiersza poleceń jest puste. Jeśli chcesz, będziesz musiał samodzielnie utworzyć początkowy commit.

Zwykle lubię mieć ten punkt wyjścia, dlatego zawsze tworzę repozytoria za pomocą Xcode.
Synchronizowanie Xcode z zmianami Git z wiersza poleceń

Xcode zazwyczaj wykrywa zmiany w repozytorium Git, które występują w terminalu. Jednak czasami może ich nie wykrywać i być niezsynchronizowany z aktualnym stanem repozytorium.

Kiedy to się zdarzy, wybierz Source Control -> Odśwież status pliku w menu Xcode.
Podstawy obsługi wiersza poleceń dla tych, którzy nigdy nie korzystali z terminala

Dla tych, którzy nigdy nie korzystali z terminala, oto podsumowanie najważniejszych poleceń, które będą potrzebne nie tylko w Git, ale przez całą karierę jako programista.
Wiersz poleceń

Wiersz poleceń to linia, w której wpisujesz polecenia w terminalu. Pokazuje nazwę użytkownika, a następnie nazwę twojego komputera. W moim przypadku jest to matteo@Nicholas.

Następnie możesz zobaczyć nazwę bieżącego katalogu, czyli bieżącego folderu. Znak ~ oznacza katalog domowy.

Nowa sesja terminala rozpoczyna się w katalogu domowym, który znajduje się w folderze Użytkownicy na dysku twardego twojego Maca.

Aby poznać pełną ścieżkę bieżącego położenia, wpisz polecenie pwd.

![img](https://iosfoundations.com/wp-content/uploads/2022/09/ls.jpeg)

Ta widok nie dostarcza zbyt wielu informacji. Na przykład, trudno odróżnić pliki od katalogów.

Aby wyświetlić więcej szczegółów, wpisz polecenie ls -l.

![img](https://iosfoundations.com/wp-content/uploads/2022/09/ls-l.jpeg)

Jeśli chcesz również wyświetlić ukryte pliki i katalogi, które zaczynają się od kropki (.), wpisz polecenie ls -la.

![img](https://iosfoundations.com/wp-content/uploads/2022/09/ls-la.jpeg)



Aby przejść do katalogu, wpisz cd, a następnie nazwę katalogu. Możesz skorzystać z funkcji automatycznego uzupełniania długich nazw, wpisując pierwsze litery nazwy i naciskając klawisz Tab na klawiaturze.

Aby przejść o jeden poziom wyżej i opuścić bieżący katalog, wpisz cd ...

Za pomocą polecenia cd możesz bezpośrednio przejść do dowolnego katalogu, korzystając z jego pełnej ścieżki niezależnie od Twojego obecnego położenia. Na przykład wpisanie cd / przenosi Cię do korzenia systemu plików.

Aby wrócić do katalogu domowego, wpisz cd bez argumentów.

![img](https://iosfoundations.com/wp-content/uploads/2022/09/cd.jpeg)



Otwieranie plików i katalogów

Możesz otworzyć dowolny plik lub katalog za pomocą polecenia "open".

Plik otworzy się w domyślnym programie dla tego typu pliku. Na przykład plik .txt otworzy się w aplikacji TextEdit, a plik .swift otworzy się w Xcode.

Wreszcie, wpisanie "open ." otworzy bieżący katalog w Finderze.

Istnieje wiele innych poleceń powłoki, które możesz się nauczyć. Jednak wykraczają one poza zakres tego krótkiego wprowadzenia. W miarę postępu w tym kursie nauczymy się wszystkich potrzebnych poleceń powłoki, w tym poleceń dla Git-a.
Otwieranie foldera projektu w terminalu bezpośrednio z Xcode

Jeśli zamierzasz często korzystać z terminala, a w tym kursie na pewno tak będzie, nawigowanie do odpowiedniego foldera może stać się monotonne, niezależnie od używanej metody.

Najbardziej wydajnym i idealnym sposobem na to jest otwieranie terminala we właściwym miejscu bezpośrednio z Xcode.

Możesz to zrobić, tworząc niestandardowe zachowanie w Xcode, które uruchamia skrypt powłoki. Wystarczy kilka kroków, a jest to bardzo wygodne i często używane rozwiązanie.
Tworzenie skryptu powłoki

Powszechnym miejscem dla niestandardowych skryptów powłoki jest katalog "bin" w folderze domowym użytkownika. Jeśli nie istnieje na Twoim komputerze, utwórz go. Następnie utwórz plik tekstowy w katalogu "bin" z następującymi dwoma liniami:

#!/bin/bash
open -a Terminal "`pwd`"

Zapisz ten plik i nazwij go "openterminal", bez rozszerzenia. Następnie przejdź do katalogu "bin" w terminalu i wpisz polecenie "chmod +x openterminal". To polecenie sprawi, że skrypt będzie można wykonywać.
Tworzenie niestandardowego zachowania w Xcode

    Wybierz "Xcode -> Preferences" w menu Xcode.
    Przejdź do karty "Behaviours".
    Kliknij przycisk "+" w lewym dolnym rogu i nazwij nowe zachowanie "Open Terminal".
    Zaznacz pole wyboru "Run", a następnie kliknij "Choose Script..." w menu kontekstowym obok niego.
    Przejdź do katalogu "bin" w katalogu domowym użytkownika i wybierz skrypt "openterminal", który właśnie utworzyłeś.
    Skonfiguruj skrót klawiaturowy dla tego zachowania (kliknij dwukrotnie i wprowadź sekwencję na klawiaturze). Ja używam ctrl+⌘+T, aby nie kolidować z innymi komendami w Xcode.

![img](https://iosfoundations.com/wp-content/uploads/2022/09/open-terminal.jpeg)

Od teraz, za każdym razem, gdy będziesz musiał użyć Git-a z wiersza poleceń, naciśnij ctrl+cmd+T (lub swój własny skrót klawiaturowy), a będziesz gotowy do działania.



##  Tworzenie migawek Twojej pracy: Tworzenie komitów Git w obszarze przygotowania

Gdy już masz repozytorium dla swojego projektu, możesz zacząć rozwijać aplikację i zapisywać swoją pracę stopniowo.

Git przechowuje Twoją pracę w komitach, które są pełnymi migawkami stanu Twojego projektu. Jednak komitowanie pracy do repozytorium nie jest tak proste, jak niektórzy programiści myślą.

Slajd 2
Najczęstszą i najważniejszą czynnością w Gicie jest komitowanie pracy do repozytorium w celu utworzenia migawki historii. Przyjrzymy się krótko, jak działa Git wewnętrznie, aby zrozumieć to lepiej.
Git utrzymuje integralność za pomocą funkcji skrótu. Obecnie Git korzysta z funkcji skrótu SHA-1. Istnieją plany migracji do SHA-256 w nieokreślonym punkcie w przyszłości.
Funkcja skrótu mapuje dane o dowolnym rozmiarze na wartość o stałym rozmiarze, zwanej skrótem lub po prostu haszem. W przypadku SHA-1, hasze mają długość 160 bitów. Zazwyczaj są one przedstawiane jako 40-znakowe liczby szesnastkowe.
Dobra funkcja skrótu jest bardzo szybka do obliczenia. Produkuje bardzo różne hasze dla danych różniących się nawet o jeden bit. Dobra funkcja skrótu minimalizuje również kolizje. Kolizje występują, gdy dwa różne fragmenty danych dają ten sam skrót.
Dzięki funkcji skrótu niemożliwe jest zmienianie zawartości plików lub katalogów bez wiedzy Gita. Nie można stracić informacji w trakcie przesyłania ani spowodować uszkodzenia pliku bez wykrycia tego przez Gita.
Będziesz widział wartości skrótu wszędzie w Gicie, ponieważ przechowuje on swoje treści według ich wartości skrótu.

Slajd 3
Za każdym razem, gdy komitujesz pracę do repozytorium, Git tworzy migawkę wszystkich plików w Twoim projekcie. Komi Git jest identyfikowany przez pojedynczy skrót, który zależy od jego zawartości, autora i daty.
Jedno powszechne nieporozumienie polega na tym, że Git przechowuje tylko zmiany w repozytorium. To nazywane jest kontrolą wersji opartą na delcie. Jest to używane przez wiele innych systemów kontroli wersji, ale nie tak działa Git.
Zamiast tego, Git przechowuje kompletną migawkę tego, jak wyglądają Twoje pliki w danym momencie. Aby być wydajnym, Git nie przechowuje ponownie pliku, jeśli nie został zmieniony. Zamiast tego, zachowuje odnośnik do poprzedniego identycznego już przechowywanego pliku. Komity Git są również ze sobą powiązane. Ważne jest zauważenie kierunku strzałek. Każdy komit wskazuje na poprzedni, tworząc tym samym całą historię Twojego projektu. Dlatego w diagramach Gita strzałki wskazują w przeszłość.

Slajd 4

Tworzenie nowego komitu jest tak łatwe jak wykonanie pojedynczej komendy Git. Ale zanim to zrobisz, musisz zdecydować, jakie zmiany zostaną zapisane w Twojej migawce.
Gdy myślisz o procesie komitowania w Gicie, wyobraź sobie trzy obszary. Pierwszy to repozytorium Git, w którym Twoje pliki są bezpiecznie przechowywane w lokalnej bazie danych. Po zainicjowaniu nowego repozytorium dla projektu jest ono puste.
Drugi obszar to Twoja kopia robocza. Czasami nazywana jest również katalogiem roboczym. To jest miejsce, w którym znajdują się wszystkie Twoje pliki na systemie plików, dzięki czemu możesz do nich uzyskać dostęp i je modyfikować.
Każdy nowo utworzony plik w Twojej kopii roboczej zaczyna się jako nieśledzony. Oznacza to, że Git o nim nie wie i nie śledzi jego zmian.
Git ma trzeci obszar, którym korzystasz do przygotowania zmian, które chcesz skomitować do repozytorium. To jest obszar przygotowania, czasami nazywany również indeksem.
Przed skommitowaniem swojej pracy dodaj wszystkie zmiany, które chcesz uwzględnić w następnym komicie do obszaru przygotowania. Przeniesienie zmian do obszaru przygotowania kopiuje je tam i oznacza jako śledzone zmiany. Oznacza to, że jest trudniej utracić te informacje, nawet jeśli nie zostały jeszcze skomitowane do repozytorium.

Slajd 5

Gdy jesteś zadowolony z zawartości obszaru przygotowania, możesz skomitować swoją pracę do repozytorium. Spowoduje to utworzenie nowej migawki, która zawiera tylko wybrane zmiany.
Każdy plik obecny w tej migawce jest teraz uważany za śledzony. A gdy odpowiadający mu plik w kopii roboczej nie ma zmian, jest uważany za niezmieniony.

Slajd 6

Gdy zmieniasz śledzony plik w kopii roboczej, staje się on zmodyfikowany. Jednak nie oznacza to automatycznego przygotowania zmian, nawet jeśli plik jest śledzony.

Usunięcie pliku obecnego w ostatniej migawce lub obszarze przygotowania również oznacza go jako

 zmodyfikowany. Jednak usunięte pliki nie znikają z repozytorium ani obszaru przygotowania.

Większość operacji w Gicie jest bezpieczna, ponieważ Git zazwyczaj tylko dodaje dane do swojej bazy danych. Dlatego skomitowanie usuniętego pliku usuwa go z Twojego komitu, ale plik wciąż istnieje bezpiecznie w historii Twojego repozytorium.

Wielu programistów obawia się usuwania niepotrzebnego już kodu, ponieważ może być potrzebny w przyszłości. Nie chcą marnować czasu, który zainwestowali w jego napisanie. Dlatego trzymają go w plikach, które nie są używane przez projekt. Lub co gorsza, jako komentarze w kodzie.

Ponieważ Git zachowuje każdą migawkę nietkniętą, możesz bezpiecznie usunąć dowolny kod, którego nie potrzebujesz. A jeśli go ponownie potrzebujesz, możesz go bezpiecznie przywrócić z historii Gita.

Slajd 7

Każda zmiana musi być jawnie dodana do obszaru przygotowania, aby została śledzona przez Gita. Nawet gdy plik jest przygotowany, kolejne zmiany w tym samym pliku nie zostaną automatycznie przygotowane.

Oznacza to, że raz przygotowana zmiana jest przechowywana przez Gita. Jest przechowywana w sposób bardziej nietrwały niż w repozytorium. Możesz nawet wziąć zmiany z obszaru przygotowania i użyć ich do zastąpienia plików w kopii roboczej. Ale to jest rzadko potrzebne.

Slajd 8

Jeśli pominięte zostaną niektóre zmiany podczas komitowania pracy, zmiany te pozostaną w Twojej kopii roboczej. A odpowiadający plik w Twojej kopii roboczej pozostanie w stanie zmodyfikowanym.

To podkreśla istotny punkt dotyczący Gita, który często jest pomijany. Większość programistów myśli o kontroli wersji jako o śledzeniu plików. Ale jeśli chodzi o przygotowywanie zmian, Git śledzi indywidualne zmiany wewnątrz każdego pliku.

Może to wydawać się niepotrzebną funkcją i irytującym dodatkowym krokiem dla kogoś, kto nie jest zaznajomiony z pracą w Git. Programiści często myślą o komitach jako o stopniowych migawkach całej zawartości ich kopii roboczej w określonym momencie.

Ale gdy omawiamy procesy i najlepsze praktyki Gita, stanie się jasne, dlaczego obszar przygotowania jest ważną funkcją.

Git oferuje również opcję całkowitego pominięcia obszaru przygotowania. W tym przypadku zmiany są

 komitowane bezpośrednio z kopii roboczej do repozytorium.

Jest to domyślne zachowanie Xcode'a. Może to upraszczać niektóre procesy pracy, ale niestety oznacza to również, że wielu programistów całkowicie pomija obszar przygotowania.

Różne stany pliku w twojej kopii roboczej

Przydatne może być rozważanie różnych kategorii, do których można przyporządkować pliki w twojej kopii roboczej.

Każdy plik w twojej kopii roboczej może znajdować się w jednym z następujących stanów:

- Śledzony (tracked), gdy Git o nim wie, czy to dlatego, że znajduje się w ostatniej migawce lub został przygotowany.
- Niesledzony (untracked), gdy Git o nim nie wie, co oznacza, że nie znajduje się ani w poprzedniej migawce, ani nie został przygotowany.

Śledzone pliki mogą być:

- Niezmienione (unmodified): śledzony plik, który nie został zmieniony w twojej kopii roboczej.
- Zmienione (modified): śledzony plik, który został zmieniony w twojej kopii roboczej.
- Usunięte (deleted): śledzony plik, który nie istnieje już w twojej kopii roboczej.

Chociaż w podręczniku Gita mówi się o plikach, lepiej myśleć o zmianach. Ten sam plik może mieć trzy różne wersje, po jednej dla każdej części Gita:

- Wersja przechowywana w ostatniej migawce.
- Zmodyfikowana wersja w obszarze przygotowania.
- Zmodyfikowana wersja w kopii roboczej z dodatkowymi zmianami, które nie zostały jeszcze przygotowane.



## Śledzenie i commitowanie: Dodawanie plików do obszaru przygotowania i zapisywanie ich w repozytorium

Aby utworzyć migawki swojej pracy, musisz przejść przez trzy etapy:

1. Dodaj nowe pliki do swojego katalogu roboczego lub zmień istniejące.
2. Wybierz, jakie zmiany chcesz grupować w jednej migawce i dodaj je do obszaru przygotowania w Gicie.
3. Zatwierdź wybrane zmiany w repozytorium projektu.

Choć brzmi to prosto, proces tworzenia aplikacji iOS może być nieco bardziej skomplikowany. Nie wszystkie twoje prace zawsze trafią do repozytorium. Czasami eksperymenty nie będą działać, i będziesz musiał odrzucić wprowadzone zmiany.

Spis treści:

- Tworzenie, modyfikowanie i usuwanie plików w Xcode
- Śledzenie statusu zmodyfikowanych plików w twojej kopii roboczej
- Jak pliki specjalne w Xcode i Git współpracują ze sobą
- Przygotowywanie i commitowanie plików w Xcode
- Zarządzanie obszarem przygotowania w Git z poziomu terminala
- Odrzucanie niektórych lub wszystkich zmian w kopii roboczej
  - W Xcode
  - Z poziomu terminala
- Commitowanie plików z poziomu terminala

Tworzenie, modyfikowanie i usuwanie plików w Xcode

Jako deweloper iOS w zasadzie używasz Xcode'a do dodawania, usuwania i edytowania plików. Pierwszą zaletą jest to, że integracja Gita w Xcode'ie automatycznie dodaje każdą zmianę do obszaru przygotowania Gita. Bez dodatkowej pracy z naszej strony.

Pierwszą częścią naszej przykładowej aplikacji, którą zbudujemy, jest okrągłe zdjęcie do wyświetlenia profilu.

Nawet dodanie małej funkcji do aplikacji prawdopodobnie oznacza, że będziesz musiał zmienić lub dodać więcej niż jeden plik do swojego projektu w Xcode'ie. Na początek potrzebujemy zdjęcia, które będzie podglądem naszego okrągłego obrazka. Możemy dodać je do zasobów podglądu naszego projektu.



![https://iosfoundations.com/wp-content/uploads/2022/09/preview-assets-scaled.jpeg](https://iosfoundations.com/wp-content/uploads/2022/09/preview-assets-scaled.jpeg)

Zazwyczaj dodaję rozszerzenie widoku (View extension) do wszystkich moich projektów. Zawiera ono kilka modyfikatorów widoku, takich jak nazwa i zmiana rozmiaru podglądów.

```swift
import SwiftUI

extension View {
    func previewWithName(_ name: String) -> some View {
        self
            .previewLayout(.sizeThatFits)
            .previewDisplayName(name)
    }

    func namedPreview() -> some View {
        let name = String.name(for: type(of: self))
        return previewWithName(name)
    }
}

extension String {
    static func name<T>(for type: T) -> String {
        String(reflecting: type)
            .components(separatedBy: ".")
            .last ?? ""
    }
}
```

Ten kod nie musi być uwzględniany w finalnej wersji aplikacji przesyłanej do App Store. Dlatego umieszczam go w pliku Previews.swift w grupie zawartości podglądów (Preview Content) projektu w Xcode.

Następnym krokiem jest stworzenie naszego okrągłego widoku.

```swift
import SwiftUI

struct CircularImage: View {
    let image: Image

    var body: some View {
        image
            .resizable()
            .frame(width: 160.0, height: 160.0)
            .clipShape(Circle())
            .overlay(
                Circle().stroke(Color.gray, lineWidth: 5.0)
            )
    }
}

struct CircularImage_Previews: PreviewProvider {
    static var previews: some View {
        CircularImage(image: Image("bearded-man"))
            .namedPreview()
    }
}
```

To jest widok, który prawdopodobnie będę używać ponownie na różnych ekranach aplikacji. Dlatego można go umieścić w osobnym pliku.
Śledzenie statusu zmodyfikowanych plików w working copy

Podczas dokonywania zmian w projekcie, Xcode śledzi je za pomocą obszaru stagingu w Git i podświetla je w nawigatorze projektu.

![img](https://iosfoundations.com/wp-content/uploads/2022/09/tracked-changes.jpeg)

Twoje pliki mogą mieć jedno z następujących stanów:

    A: dodany (nowy i przystawiony plik).
    M: zmodyfikowany (zaangażowany plik z zmianami w working copy).
    R: usunięty (zaangażowany plik usunięty z working copy).
    ?: nieśledzony (nowy, ale nieprzystawiony plik).

Możesz wyświetlić zawartość obszaru stagingu w Git za pomocą polecenia git status.

![img](https://iosfoundations.com/wp-content/uploads/2022/09/git-status.jpeg)



W Git, pliki są podzielone na trzy grupy: przystawione (staged), nieprzystawione (not staged) i nieśledzone (untracked). Na razie nie ma nieśledzonych plików, ale wkrótce się pojawią.

Polecenie git status może być uciążliwe, zwłaszcza gdy dokonałeś wielu zmian. Dodanie opcji `-s' pozwoli na uzyskanie uproszczonych wyników statusu, korzystając z powyższych liter.

![https://iosfoundations.com/wp-content/uploads/2022/09/git-status-s.jpeg](https://iosfoundations.com/wp-content/uploads/2022/09/git-status-s.jpeg)

Jak wspomniałeś, ważne jest, aby pamiętać, że Xcode ukrywa pewne szczegóły, ale nie przed Gitem. Jeśli nie będziesz ostrożny, może to stworzyć problemy.

Po pierwsze, zauważ, że plik projektu Xcode jest oznaczony jako zmodyfikowany, chociaż go nie dotknęliśmy.

Plik projektu Xcode zawiera wiele informacji. Na początek, wszystkie ustawienia projektu potrzebne do zbudowania aplikacji są tam przechowywane.

![https://iosfoundations.com/wp-content/uploads/2022/09/project-file-scaled.jpeg](https://iosfoundations.com/wp-content/uploads/2022/09/project-file-scaled.jpeg)

