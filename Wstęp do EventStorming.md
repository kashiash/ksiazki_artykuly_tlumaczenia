#Wstęp do EventStorming

Akt celowego zbiorowego uczenia się

Alberto Brandoliniego

## Wstęp

Zacząłem pisać tę książkę kilka lat temu. Dziwnie jest odłożyć to zdanie, ponieważ w tamtym czasie nie mogłem sobie wyobrazić, co się wydarzy. EventStorming to było kilka postów na blogu i prezentacji, mała społeczność praktyków i plastyczne narzędzie, które wciąż było pełne niespodzianek.

Przyjąłem "filozofię LeanPub", publikując przed zakończeniem książki i rozpocząłem ciekawą przejażdżkę: posty na blogu i prezentacje mogą być celowo efemeryczne, ale książka ma być tutaj, aby pozostać i zapewnić wiarygodne treści dla czytelników, którzy mają przyjść, i to nie tylko dla obecnych słuchaczy.

W tym samym czasie wciąż zgłębiałem technikę i odkrywałem nowe pomysły i format, a nowe pomysły również zasługiwały na miejsce w książce! Informacje rosły jako dziennik podróży, podczas gdy ja musiałem zachować spójność całości. Pisanie było łatwe, ale przepisywanie to zupełnie inna sprawa.

W międzyczasie EventStorming rozrósł się i doprowadził mnie do ćwiczenia i odkrywania EventStorming w wielu różnych domenach na całym świecie. Oczywiście, była za to cena do zapłacenia: dostępny czas Za pisanie skurczył się między podróżami służbowymi, warsztatami, zajęciami szkoleniowymi, konferencjami i spotkaniami. Moje aktualizacje dotyczące rękopisu książki stały się coraz częstsze, podczas gdy liczba czytelników stale rosła.

Z drugiej strony praktyka stopniowo się konsolidowała: niektóre techniki eksperymentalne okazały się teraz skuteczne, a książka może być nieco mniej naiwna niż początkowe pisanie. To po prostu wymagało dobrej jakości czasu, aby być skończonym. A mi tak myślałem.

Pandemia w 2020 roku zadała poważny cios mojemu zamiarowi, aby być odpornym na przyszłość. Każda strona, którą napisałem przed marcem 2020 r., obejmowała ideę, że EventStorming jest przede wszystkim działalnością ludzką, którą należy ćwiczyć, umieszczając wszystkich w tym samym pokoju. Ale w latach 2020-2021 pojęcie wypełniania zamkniętej przestrzeni ludźmi zaczęło mieć złowrogi dźwięk w świecie, w którym dystans społeczny stał się normą.

Musiałem więc zmierzyć się z nowym dylematem: „Czy powinienem spieszyć się, aby dokończyć książkę, proponując narzędzie, które teraz wygląda jak stary świat, czy zamiast tego powinienem zmodernizować treść, aby pasowała do zmienionej rzeczywistości?”

Wybrałem drugą opcję, nawet jeśli oznaczało to boleśnie rozpoczęcie recenzowania i przepisywania dużej ilości treści. Idealnie byłoby, gdyby to dało ci książkę z dobrymi pomysłami na którykolwiek z możliwych skutków pandemii: Powrót do normalności, w tym szaleństwo podróży i logistyki, lub utknął w niekończącym się łańcuchu wideokonferencji podczas pracy w domu. Nie mam kryształowej kuli, ale postaram się dać ci opcje, kiedy będę mógł.

Ogólnie rzecz biorąc, formaty na dużą skalę, takie jak Big Picture, najbardziej korzystają z interakcji międzyludzkich, podczas gdy mniejsze modelowanie procesów i projektowanie oprogramowania lepiej tolerują ograniczenia zdalnego połączenia.

### Dla kogo jest ta książka

EventStorming stał się narzędziem o wiele bardziej ogólnego przeznaczenia, niż się spodziewałem, od rozwoju oprogramowania po projektowanie organizacji.

Moje doświadczenie to rozwój oprogramowania, przejście przez zwinne doradztwo, a następnie stanie się przedsiębiorcą. I idealnie, to jest publiczność, z którą rozmawiam: ludzie biznesu, którzy chcą wykorzystać potencjał technologii i eksperci, którzy chcą być zaangażowani w znaczące projekty.

### Notacja

Będę surowy w byciu zgodnym z moją notacją, używając sztywnego schematu kolorów (pomarańczowy dla zdarzeń domeny `DomainEvents`, magenta dla `HotSpaces` gorących miejsc, niebieski dla poleceń i tak dalej). Rodzi się na dostępnych kolorach dla Karteczki samoprzylepne i ciekawie cyfrowe platformy do modelowania postanowiły być kompatybilne z paletami kolorów na bazie papieru, więc niewiele się zmieniłem.

Podziękowania

[FIXME: to nadal będzie wielki bałagan. Tak wielu ludzi, aby podziękować...]

Jak czytać tę książkę

Obecny stan jest następujący: w książce jest mnóstwo wartościowych treści, ale także dużo pracy w toku.

* niedokończone rozdziały;

* ukończone rozdziały, które teraz wyglądają na trochę przestarzałe w czasach pandemii;

* nadmiarowe treści;

* Nadal nie wiem, jak zakończyć tę książkę.

Licznik postępów na górze rozdziałów

Dodałem wskaźnik postępu w nagłówkach rozdziałów, aby zapewnić poczucie statusu ukończenia przed jego przeczytaniem. Licznik jest arbitralny, ale niektórzy czytelnicy uznali go za przydatny.