# Przewodnik po uwierzytelnianiu za pomocą tokenów internetowych JSON (JWT) w systemie iOS



oryginalnmy artykuł: https://azamsharp.com/2023/11/07/complete-guide-jwt-authentication.html

Uwierzytelnianie jest podstawą tworzenia aplikacji i zabezpiecza interakcje użytkowników zarówno w aplikacjach klienckich, jak i serwerowych. W świecie, w którym JSON króluje jako podstawowy środek wymiany danych, uwierzytelnianie JSON Web Token (JWT) stało się standardem branżowym, zapewniającym bezpieczeństwo i integralność danych.

W tym obszernym przewodniku wyruszamy w podróż przez zawiłości związane z konfiguracją uwierzytelniania JWT na serwerze i bezproblemowym wdrażaniem tego środka bezpieczeństwa po stronie klienta za pomocą SwiftUI. Ta wiedza wyposaży Cię w niezbędne umiejętności potrzebne do wzmocnienia mechanizmów uwierzytelniania aplikacji, zapewniając niezawodność i solidność w dzisiejszym, stale rozwijającym się krajobrazie cyfrowym.

W tym artykule moc SwiftUI została wykorzystana w interfejsie klienta, podczas gdy ExpressJS stanowi szkielet operacji po stronie serwera. Chociaż ten wybór może wzbudzić ciekawość programistów iOS, jest on oparty na moim rozległym doświadczeniu i zaufaniu do ExpressJS, solidnego i niezawodnego narzędzia do tworzenia serwerów.

## Rejestracja użytkownika (serwer) ExpressJS

Jak wspomniano wcześniej, serwer jest zbudowany w oparciu o framework ExpressJS. Do przechowywania danych wykorzystywany jest PostgreSQL wraz z Sequelize, frameworkiem ORM (Object Relational Mapping). Architektura serwera jest zgodna ze wzorcem projektowym MVC (Model View Controller), zapewniając ustrukturyzowane i zorganizowane podejście do tworzenia aplikacji. Poniżej przedstawiamy niektóre z głównych komponentów wykorzystywanych po stronie serwera:

• Routery: te komponenty efektywnie zarządzają trasami aplikacji, reprezentując różne punkty końcowe, za pośrednictwem których aplikacja współdziała.
• Kontrolery: Kontrolery są odpowiedzialne za zarządzanie żądaniami pochodzącymi z routerów. Pełnią rolę pośredników, ułatwiając komunikację z warstwą dostępu do danych w celu wykonywania operacji na modelach aplikacji.
• Modele: Modele definiują podstawowe elementy, wokół których obraca się aplikacja, takie jak użytkownik, film, przepis i inne. Służą jako plan struktur danych używanych w aplikacji.
• Oprogramowanie pośredniczące: Komponenty oprogramowania pośredniczącego stanowią integralną część przepływu wykonywania aplikacji. Wykonują określoną logikę kodu, zanim żądanie osiągnie zamierzoną akcję. W naszym przypadku oprogramowanie pośrednie odgrywa kluczową rolę w egzekwowaniu środków uwierzytelniania na wybranych trasach i punktach końcowych, zapewniając bezpieczeństwo aplikacji.
Przed włączeniem uwierzytelniania użytkownika należy koniecznie upewnić się, że użytkownicy mogą sprawnie ukończyć proces rejestracji. Aby zarejestrować nowego użytkownika, użytkownik musi podać ważny adres e-mail i hasło. Odpowiedzialność za zarządzanie rejestracją użytkowników jest delegowana na authController. Komponent ten odgrywa kluczową rolę w nadzorowaniu procesu rejestracji użytkownika i związanych z nim funkcjonalności.

Implementację pokazano poniżej:

```javascript
exports.register = async (req, res) => {

    const { email, password } = req.body

    const errors = validationResult(req)
    if (!errors.isEmpty()) {
        res.status(400).json({ success: false, message: errors.array().map(error => error.msg).join(' ') })
        return
    }

    // check if the user is already registered 
    const user = await models.User.findOne({
        where: {
            email: email
        }
    })

    if (user) {
        // already exists 
        res.status(409).json({ success: false, message: 'Email is already registered.' })
        return
    }

    try {

        // hash the password 
        const hashedPassword = await bcrypt.hash(password, 10)

        // register new user 
        const newUser = await models.User.create({
            email: email,
            password: hashedPassword 
        })

        // 201 for created and send the new user back 
        res.status(201).json({ success: true })

    } catch (error) {
        console.error(error)
        res.status(500).json({ success: false, message: 'Internal server error' })
    }
}

```



Walidacją treści zajmuje się funkcja valid, zdefiniowana w następujący sposób:



```javascript
exports.validate = (method) => {

    switch (method) {
        case 'register': {
            return [
                body('email', 'Email cannot be empty.').exists().isEmail(),
                body('password', 'Password must be at least 6 characters long').exists().isLength({ min: 6 }), 
                body('roleId', 'Role must be provided.').exists() 
            ]
        }
    }
}
```

Akcja rejestru `register` używa narzędzia `express-validator` do sprawdzania przychodzących danych. Po zatwierdzeniu treści sprawdzamy, czy adres e-mail jest unikalny, czy nie. Jeśli adres e-mail jest już zarejestrowany, zwracany jest kod statusu 409. Jeśli adres e-mail jest unikalny, hashujemy hasło i utrwalamy je w bazie danych. Hasło jest szyfrowane przy użyciu pakietu o nazwie `bcryptjs`.
`AuthController` musi zostać zarejestrowany przez router. Dzięki temu wszystkie wyznaczone żądania zostaną przekazane do `authController`. Rejestracja ta odbywa się wewnątrz routera indeksującego. Implementację pokazano poniżej:

```javascript
const express = require('express')
const router = express.Router() 
const authController = require('../controllers/authController') 

router.post('/register', authController.validate('register'), authController.register)

module.exports = router 
```

I na koniec, router indeksowy musi być ustawiony jako oprogramowanie pośredniczące w głównym pliku app.js.

```javascript
const express = require('express')
const app = express() 
const cors = require('cors')
const indexRouter = require('./routes/index') 

app.use(cors())
app.use(express.json())

app.use('/api', indexRouter)

app.listen(8080, () => {
    console.log('Server is running...')
})
```

Nasza akcja rejestracji użytkownika została zakończona. Teraz musimy skupić się na kliencie SwiftUI, aby przeprowadzić rejestrację.

> Ważne jest, aby upewnić się, że działanie rejestru działa zgodnie z oczekiwaniami. Można to wykonać pisząc test na serwerze lub testy integracyjne na kliencie. Użyj przynajmniej Postmana do przetestowania punktów końcowych.



## Rejestracja użytkownika (klient)

Po sprawdzeniu funkcjonalności naszego punktu końcowego rejestracji naszym następnym krokiem będzie przeniesienie uwagi na stronę klienta. Zamiast tworzyć odrębne funkcje sieciowe dla różnych żądań, takich jak logowanie, rejestracja, pobieranie kursów itp., usprawnimy nasze podejście, wprowadzając klienta HTTP JSON. Ten klient HTTP przejmie odpowiedzialność za obsługę żądań sieciowych i dostarczanie danych modelu komunikującemu się z serwerem . Szczegóły implementacji klienta HTTP można znaleźć poniżej:

```swift
struct HTTPClient {
    
    static let shared = HTTPClient()
    private let session: URLSession
    
    private init() {
        let configuration = URLSessionConfiguration.default
         // add the default header
        configuration.httpAdditionalHeaders = ["Content-Type": "application/json"]
        self.session = URLSession(configuration: configuration)
    }
    
    func load<T: Codable>(_ resource: Resource<T>) async throws -> T {
        
        var request = URLRequest(url: resource.url)
        
        switch resource.method {
            case .get(let queryItems):
                var components = URLComponents(url: resource.url, resolvingAgainstBaseURL: false)
                components?.queryItems = queryItems
                guard let url = components?.url else {
                        throw NetworkError.badRequest
                }
                
                request = URLRequest(url: url)
                
            case .post(let data):
                request.httpMethod = resource.method.name
                request.httpBody = data
            
            case .delete:
                request.httpMethod = resource.method.name
        }
        
        let (data, response) = try await session.data(for: request)
        
        if let httpResponse = response as? HTTPURLResponse {
            
            switch httpResponse.statusCode {
                case 401:
                    throw NetworkError.unauthorized
                // handle other cases here ...
                default: break
            }
            
        }
        
        do {
            let result = try JSONDecoder().decode(resource.modelType, from: data)
            return result
        } catch {
            throw NetworkError.decodingError(error)
        }
    }
}
```

Pełną implementację klienta HTTP można znaleźć [tutaj](https://gist.github.com/azamsharp/91919badd2c70c75732ad715b1c3e01e).

Być może rozważasz podzielenie funkcji ładowania na mniejsze podfunkcje, być może w oczekiwaniu na ponowne wykorzystanie tych komponentów w całym kodzie. Radziłbym jednak na tym etapie powstrzymać się od takiej abstrakcji. Głównym uzasadnieniem tego zalecenia jest to, że logika funkcji obciążenia dotyczy obecnie wyłącznie tej funkcji. Obecnie nie ma zapotrzebowania ani konieczności samodzielnego stosowania izolowanych segmentów funkcji obciążenia. Jeśli w przyszłości pojawi się takie wymaganie, zawsze możemy wyodrębnić i zmodularyzować określone funkcje w oparciu o istniejącą implementację funkcji obciążenia.

*Kod, który pasuje do siebie, pozostaje razem*

Istnieją różne sposoby wywoływania warstwy HTTPClient z widoku. Możesz udostępnić klienta HTTPClient jako wartość Environment i uzyskać do niego bezpośredni dostęp z poziomu widoku lub wprowadzić obiekt ObservableObject, taki jak `Account`.

Każde podejście ma swoje zalety. Podejście do wartości środowiska jest znacznie prostsze i pozwala na bezpośredni dostęp do warstwy sieciowej w widoku. Jest to idealne rozwiązanie w scenariuszach, w których interfejs API sieci Web jest tylko do odczytu i nie pozwala widokowi na mutację danych.

W SwiftUI widok jest modelem widoku. Jeśli chcesz przeczytać więcej o architekturze SwiftUI, zapoznaj się z moim artykułem [Tworzenie dużych aplikacji przy użyciu SwiftUI](https://azamsharp.com/2023/02/28/building-large-scale-apps-swiftui.html).

Podejście ObservableObject jest używane w scenariuszach, w których możesz chcieć ponownie wykorzystać wyniki żądania w innych widokach, a także wykonać pewne logiczne operacje na danych.
W tym artykule będę korzystał z podejścia ObservableObject, ponieważ pozwoli nam to na utrwalenie stanu w przyszłości, gdy będziemy mieli do czynienia z logowaniem i uwierzytelnianiem.
Implementacja klasy Account pokazana jest poniżej:

```swift
@Observable
class Account {
    
    private var httpClient: HTTPClient
    
    init(httpClient: HTTPClient) {
        self.httpClient = httpClient
    }
    
    func register(email: String, password: String) async throws -> RegistrationResponse {
        
        let params = [JSON.Keys.email: email, JSON.Keys.password: password]
        let body = try JSONEncoder().encode(params)
        
        let resource = Resource(url: API.endpointURL(for: .register), method: .post(body), modelType: RegistrationResponse.self)
        let response = try await httpClient.load(resource)
        return response
    }
    
}
```

`Account` jest zależne od klienta HTTP. Należy zauważyć, że nie używamy abstrakcji dla klienta HTTP, ale bezpośrednio używamy konkretnej implementacji. Głównym powodem jest to, że `HTTPClient` jest zależnością zarządzaną. Zależności zarządzane to zależności, nad którymi masz całkowitą kontrolę. Świat zewnętrzny nie ma do nich dostępu. Obejmuje to system plików, bazy danych itp. Podczas pisania testów zachowań wchodzących w interakcję z zarządzanymi zależnościami zaleca się użycie prawdziwej implementacji zamiast prób. W przypadku niezarządzanych zależności należy używać tzw `Mocków`. Typowymi przykładami niezarządzanej zależności są bramki płatnicze, lokalne usługi wymiany walut itp.

> Ważną rzeczą, o której należy pamiętać podczas korzystania z konkretnych implementacji, jest zawsze czyszczenie danych po teście. Oznacza to, że jeśli podczas testu dodałeś użytkownika do bazy danych, pamiętaj o usunięciu go po zakończeniu testu. Zwykle wykonuje się to w ramach funkcji rozrywania w środowisku XCTest.

Account jest wstrzykiwane jako obiekt środowiska do widoku głównego aplikacji. Implementację pokazano poniżej:

```swift
@main
struct ExamPrepApp: App {
    
    @State private var account = Account(httpClient: HTTPClient.shared)
    
    var body: some Scene {
        WindowGroup {
            NavigationStack {
                ContentView()
                    .environment(account)
            }
        }
    }
}
```

Jak wspomniano wcześniej, ponieważ klient HTTPClient jest zależnością zarządzaną, używamy konkretnej implementacji zamiast Mocka .

Widok może uzyskać dostęp do instancji `Account` za pomocą opakowania właściwości `@EnvironmentObject`. Gdy widok uzyska dostęp do konta, może skorzystać z funkcji rejestracji, jak pokazano poniżej:

```swift
 private func register() async {
        do {
            let response = try await account.register(email: email, password: password)
            if response.success {
                // navigate to the login screen
                navigate(.login)
            } else {
                message = response.message ?? ""
            }
        } catch {
            message = error.localizedDescription 
        }
    }
```

Jedną z rzeczy, na które warto zwrócić uwagę w związku z naszą interakcją z serwerem, jest to, że serwer zwraca wiadomości do klienta. Komunikaty te wskazują powód awarii, która wystąpiła na serwerze.

Jeśli chcesz dowiedzieć się, jak zbudować system nawigacji w SwiftUI i globalnie obsługiwać komunikaty, sprawdź poniższe zasoby.

[Navigacja](https://azamsharp.com/2023/02/28/building-large-scale-apps-swiftui.html#navigation)

[Wyświetlanie błedów](https://azamsharp.com/2023/02/28/building-large-scale-apps-swiftui.html#displaying-errors)

Po pomyślnej rejestracji użytkownik zostanie przekierowany do ekranu logowania. W tym momencie musimy wrócić do serwera i zaimplementować punkt końcowy logowania.

## Logowanie  (serwer)

Jednym z najkorzystniejszych aspektów nabywania biegłości w uwierzytelnianiu JSON Web Token (JWT) jest to, że po pomyślnym zaimplementowaniu go w jednym frameworku i języku programowania stosunkowo łatwo będzie zastosować je w dowolnym innym frameworku. Osobiście stosowałem JWT w różnych kontekstach, w tym w React, Flutter i iOS, i odkryłem, że poza składnią specyficzną dla języka, podstawowe zasady pozostają spójne.

Proces logowania uwierzytelniającego JWT składa się z następujących kroków:

1. Sprawdź parametry podane w treści żądania.
2. Pobierz użytkownika na podstawie jego nazwy użytkownika lub adresu e-mail.
3. Zweryfikuj przesłane hasło z hasłem przechowywanym w bazie danych.
4. Po pomyślnej weryfikacji wygeneruj token sieciowy JSON, podpisz go identyfikatorem użytkownika i ustaw datę ważności.
5. W odpowiedzi przekaż użytkownikowi token.
Oto implementacja trasy logowania.

```swift
exports.login = async (req, res) => {

    const { email, password } = req.body
     
    // perform validation on email and password  

    // check if the user exists 
    const user = await models.User.findOne({
        where: {
            email: email
        }
    })

    if (user) {
        // check the password 
        let result = await bcrypt.compare(password, user.password)
        if (result) {
            // generate the expiration time = 1 hour 
            const expirationTime = Math.floor(Date.now() / 1000) + 60 * 60 * 24 * 60; // 60 days in seconds.. you can change that :) 
            // generate the jwt token 
            const token = jwt.sign({ userId: user.id, exp: expirationTime }, process.env.JWT_PRIVATE_KEY)
            res.json({ success: true, token: token, exp: expirationTime, roleId: user.roleId })

        } else {
            res.status(400).json({ success: false, message: 'Incorrect password' })
        }

    } else {
        res.status(400).json({ success: false, message: 'User not found' })
        return
    }

}
```



> Bardzo ważne jest, aby unikać przechowywania klucza prywatnego JWT bezpośrednio w kodzie. Jednym z kluczowych powodów jest zapobieganie widocznemu kluczowi prywatnemu w systemach kontroli źródła, takich jak GitHub. Bezpieczniejsze podejście polega na użyciu zmiennych środowiskowych za pośrednictwem pliku .env i zapewnieniu jego poufności poprzez dodanie pliku do .gitignore, aby utrzymać go poza kontrolą wersji.

Po raz kolejny konieczne jest dokładne przetestowanie akcji logowania. Masz możliwość pisania testów dla swoich kontrolerów, możesz także tworzyć testy integracyjne, które obejmują zarówno Twojego klienta (SwiftUI), jak i serwer (ExpressJS). W kontekście testów integracyjnych krytyczna jest ocena najdłuższych szczęśliwych ścieżek i zbadanie przypadków brzegowych, aby zapewnić solidne i kompleksowe testowanie.

> Najdłuższa szczęśliwa ścieżka oznacza pomyślną realizację scenariusza biznesowego.

Następnym krokiem jest wdrożenie klienta. Obejmuje to wywołanie akcji logowania w celu uwierzytelnienia użytkownika.

## Logowanie (Klient)

Początkowe zadanie polega na utworzeniu obiektu `DTO` odpowiedzialnego za mapowanie odpowiedzi serwera. Oprócz oczekiwanych właściwości, takich jak „sukces” i „wiadomość”, serwer udostępnia również „token”, który musi być przechowywany po stronie klienta. Poniżej znajdziesz implementację obiektu „LoginResponse”:

```swift
struct LoginResponse: Codable {
    let success: Bool
    let message: String?
    let token: String?
}
```

> LoginResponse nie zawiera właściwości userId. Głównym powodem jest to, że userId jest już zalogowany do samego tokena, więc serwer będzie mógł go wyodrębnić w razie potrzeby.

```swift
  func login(email: String, password: String) async throws {
        
        let params = [JSON.Keys.email: email, JSON.Keys.password: password]
        let body = try JSONEncoder().encode(params)
        
        let resource = Resource(url: API.endpointURL(for: .login), method: .post(body), modelType: LoginResponse.self)
        let response = try await httpClient.load(resource)
        
        // Check if the login was successful
        guard response.success, let token = response.token else {
            throw LoginError.loginFailed
        }
        
        // Store the JWT token in the Keychain
        guard Keychain.set(token, forKey: "jwttoken") else {
            throw LoginError.keychainError
        }
        
        isLoggedIn = true
    }
```

Krytycznym aspektem funkcji logowania jest wykorzystanie pęku kluczy do przechowywania tokenów. Keychain oferuje wysoce bezpieczne rozwiązanie do przechowywania danych, zaprojektowane specjalnie do ochrony wrażliwych danych, w tym haseł, tokenów i kluczy kryptograficznych. Dzięki szyfrowaniu i środkom ochronnym jest to idealny wybór do ochrony poufnych informacji.

> UserDefaults przedstawia alternatywę do przechowywania par klucz/wartość; należy jednak pamiętać, że UserDefaults nie jest szyfrowany i nie należy go używać do utrwalania wrażliwych lub poufnych danych.

Problem z powyższą implementacją polega na tym, że gdy użytkownik ponownie uruchomi aplikację, właściwość `isLoggedIn` jest ustawiana na wartość false, nawet jeśli użytkownik pomyślnie się zalogował. Możemy rozwiązać ten problem, ustawiając wartość isLoggedIn na podstawie tokena przechowywanego w pęku kluczy. Jest to zaimplementowane poniżej:

```swift
 init(httpClient: HTTPClient) {
        self.httpClient = httpClient
        // set the isLoggedIn to be the value from the keychain
        isLoggedIn = Keychain<String>.get("jwttoken") != nil
    }
```

Teraz, gdy uruchomisz aplikację od zera, właściwość isLoggedIn zostanie zainicjowana w oparciu o token JWT.

Teraz możesz użyć klasy Account na ekranie logowania, jak pokazano poniżej:

```swift
struct LoginScreen: View {
    
    @Environment(Account.self) private var account
    
    @State private var email: String = ""
    @State private var password: String = ""
    
    @State private var authenticating: Bool = false
    
    private func login() async {
        do {
            try await account.login(email: email, password: password)
            // navigate to the dashboard screen
            navigate(.dashboard)
        } catch {
            print(error)
        }
    }
    
    var body: some View {
        Form {
            TextField("Email", text: $email)
                .textInputAutocapitalization(.never)
            SecureField("Password", text: $password)
            Button("Login") {
                authenticating = true
            }.task(id: authenticating) {
                if authenticating {
                    await login()
                    authenticating = false
                }
            }
        }.navigationTitle("Login")
    }
}
```

Po pomyślnym zalogowaniu użytkownik zostaje przeniesiony do ekranu panelu kontrolnego.

Jeśli chcesz dowiedzieć się, jak skonfigurować nawigację w SwiftUI, przeczytaj mój artykuł tutaj.
W kolejnej części dowiesz się jak tworzyć chronione trasy na serwerze, co wymaga zalogowania użytkownika.

## Implementacja tras ochronnych na serwerze

Termin chronione trasy oznacza, że użytkownik musi zostać uwierzytelniony, aby móc uzyskać dostęp do żądanych zasobów.

W `FacultyController` implementujemy następującą funkcję, aby pobrać wszystkie kursy oferowane przez członka wydziału.

```swift
exports.getCourses = async (req, res) => {
    
    // fetch the courses from the database

    // return courses 
    res.json(courses)
}
```

Wspomniana powyżej funkcja „getCourses” pełni funkcję chronionego zasobu, a jednym z warunków uzyskania dostępu do niej jest uwierzytelnienie użytkownika. Chociaż jednym z podejść jest osadzenie logiki uwierzytelniania bezpośrednio w funkcji „getCourses”, zaleca się zaimplementowanie tej funkcjonalności jako oprogramowania pośredniczącego. W ten sposób możemy scentralizować logikę uwierzytelniania i udostępnić ją do wykorzystania w innych działaniach i trasach, zwiększając możliwość ponownego użycia i łatwość konserwacji.

Funkcja uwierzytelniania jest zaimplementowana w authMiddleware.js, jak pokazano poniżej:

```swift
// authenticate middleware 
exports.authenticate = async (req, res, next) => {

    const authHeader = req.headers['authorization']
    if(authHeader) {
        // get the token out of the header 
        const token = authHeader.split(' ')[1]
        console.log(token)
        if(token) {
            // decode the token 
            try {
                const decodedToken = jwt.verify(token, process.env.JWT_PRIVATE_KEY)
                const user = await models.User.findByPk(decodedToken.userId)
                if(!user) {
                    res.status(401).json({success: false, message: 'User not found'})
                }
                // store the userId in the request
                req.userId = user.id 
                next() 
            } catch(error) {
                if(error instanceof jwt.JsonWebTokenError) {
                    res.status(401).json({success: false, message: 'Token expired'})
                } else {
                    res.status(500).json({success: false, message: 'Internal server error'})
                }
            }
        } else {
            res.status(401).json({success: false, message: 'Bearer token is missing'})
        }

    } else {
        res.status(401).json({success: false, message: 'Required headers are missing'})
    }
}
```

Nasz pierwszy krok polega na uzyskaniu nagłówka autoryzacyjnego z żądania. Po uzyskaniu nagłówka autoryzacyjnego przystępujemy do wyodrębnienia zawartego w nim tokena. Token ten jest następnie poddawany weryfikacji za pomocą funkcji „verify”, będącej komponentem pakietu „jsonwebtoken”. Po pomyślnej weryfikacji przypisujemy wyodrębniony identyfikator użytkownika do żądania, zapewniając jego dostępność do wykorzystania w przyszłych żądaniach.

> Pakiet `jsonwebtoken` dodatkowo obsługuje kontrolę ważności tokenu. Po wygaśnięciu tokenu serwer zgłosi błąd `jwt.JsonWebTokenError`.

Po wdrożeniu implementacji `authMiddleware` kolejnym krokiem jest jego zarejestrowanie na trasach wymagających ochrony. Ten proces rejestracji odbywa się w pliku app.js, jak pokazano poniżej:

```swift
app.use('/api/faculty', authenticate, facultyRouter)
app.use('/api/students', authenticate, studentRouter)
```

> Trasy takie jak logowanie i rejestracja nie muszą być chronione. Nie możesz dodać warunku wstępnego logowania, aby być już zalogowanym.

W następnej sekcji przeniesiemy naszą uwagę na stronę klienta i dowiemy się, jak wyodrębnić token i udostępnić go chronionym żądaniom.

## Dostęp do chronionych tras (klient)

Aby klient mógł uzyskać dostęp do chronionej trasy na serwerze, musi wysłać token w nagłówkach żądania. Tę operację można zaimplementować bezpośrednio w kliencie HTTP, jak pokazano poniżej:

```swift
struct HTTPClient {

    static let shared = HTTPClient() 
    private let session: URLSession
    
    private init() {
        let configuration = URLSessionConfiguration.default
         // add the default header
        configuration.httpAdditionalHeaders = ["Content-Type": "application/json"]
        // get the token from the Keychain
        let token: String? = Keychain.get("jwttoken")
        
        if let token {
 configuration.httpAdditionalHeaders?["Authorization"] = "Bearer \(token)"        }
        
        self.session = URLSession(configuration: configuration)
    }

}

// load function is implemented here. 

```

Po zainicjowaniu klienta HTTP sprawdzamy pęk kluczy pod kątem utrwalonego tokena internetowego JSON. Jeśli token jest obecny, do żądania dodajemy nagłówek Authorization. Dzięki temu token będzie częścią każdego żądania wysyłanego przez klienta.

Teraz, gdy użytkownik uruchomi aplikację, możesz sprawdzić stan właściwości isLoggedIn w klasie Konto i przekierować użytkownika do odpowiedniego ekranu, w oparciu o jego status logowania.

```swift
@main
struct ExamPrepApp: App {
    
    @State private var account = Account(httpClient: HTTPClient.shared)
    
    var body: some Scene {
        WindowGroup {
            NavigationStack {
                if account.isLoggedIn {
                    DashboardScreen() 
                } else {
                    LoginScreen()
                        .environment(account)
                }
            }
        }
    }
}
```

W podobnym scenariuszu, jeśli użytkownik zażąda zasobu, a token wygasł, możesz przekierować użytkownika do ekranu logowania. Pokazano to w poniższej implementacji:

```swift
struct DashboardScreen: View {
    
    @Environment(Faculty.self) private var faculty
    
    var body: some View {
        VStack {
            Text("Dashboard")
        }.task {
            do {
                try await faculty.loadCourses()
            } catch NetworkError.unauthorized {
                 navigate(.login)
            } catch {
                print(error.localizedDescription)
            }
        }
    }
}
```

> Najlepszą praktyką jest sprawdzenie tokena na serwerze i kliencie. Weryfikacja klienta tokena zostanie dodana w dalszej części tego artykułu.



> Ten artykuł NIE dotyczy tokenów odświeżania. Być może jest to coś, co będę mógł dodać w przyszłości. Głównym celem tokenów odświeżania w tokenach sieciowych JSON (JWT) jest przedłużenie czasu życia tokenów dostępu przy jednoczesnym zachowaniu bezpiecznego mechanizmu uwierzytelniania. Tokeny dostępu, które są zazwyczaj krótkotrwałe, służą do weryfikacji tożsamości użytkownika na potrzeby określonych działań w aplikacji. Jednak częste ponowne uwierzytelnianie z powodu wygasających tokenów dostępu może zakłócać wygodę użytkownika i utrudniać użyteczność aplikacji.

## Kod źródłowy

Kod możesz pobrać [stąd](https://github.com/azamsharp/ExamPrepCode).

## Podsumowanie:

Podsumowując, ten obszerny przewodnik zapewnił dogłębną wiedzę na temat uwierzytelniania JSON Web Token (JWT) w kontekście programowania systemu iOS. Uwierzytelnianie jest kluczowym aspektem bezpieczeństwa aplikacji, a JWT jest powszechnie uznawany za standard branżowy służący do zabezpieczania aplikacji klient/serwer, zwłaszcza gdy podstawowym sposobem komunikacji jest JSON.

W tym artykule dowiesz się, jak skonfigurować uwierzytelnianie JSON Web Token na serwerze i przeprowadzić uwierzytelnianie po stronie klienta za pomocą aplikacji SwiftUI. Chociaż konkretne narzędzia i struktury użyte w tym przewodniku to SwiftUI dla klienta i ExpressJS dla serwera, omówione zasady można zastosować na różnych platformach.

Wdrożenie po stronie serwera obejmowało rejestrację i logowanie użytkowników, podkreślając znaczenie środków bezpieczeństwa, takich jak hashowanie haseł. Serwer chronił również niektóre trasy, wdrażając oprogramowanie pośredniczące do uwierzytelniania użytkowników.

Po stronie klienta nauczyłeś się, jak utworzyć klienta HTTP do obsługi żądań sieciowych oraz bezpiecznego przechowywania i pobierania tokenu JWT z pęku kluczy. Kod po stronie klienta pokazał, jak wchodzić w interakcję z serwerem, obsługiwać pomyślne logowanie i nawigować do różnych widoków w aplikacji SwiftUI.

Ogólnie rzecz biorąc, uwierzytelnianie JWT jest cenną umiejętnością dla programistów pracujących na iOS i poza nim. Niezależnie od tego, czy zdecydujesz się zastosować konkretne narzędzia i praktyki opisane w tym przewodniku, czy też dostosujesz je do preferowanego środowiska, zdobytą tutaj wiedzę można zastosować w celu zwiększenia bezpieczeństwa i funkcjonalności aplikacji.