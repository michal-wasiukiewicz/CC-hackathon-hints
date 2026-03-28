# 🚀 Pierwsze Kroki z Claude Code + ECC
### Instrukcja dla całego zespołu · Przykład: Formularz Logowania

> Zakładamy: instalacja zakończona, VM gotowa, `claude --version` działa.
> Nie zakładamy: żadnej wcześniejszej pracy z Claude Code lub ECC.

---

## Spis treści

1. [Co masz przed sobą — model mentalny dla nowego użytkownika](#1-co-masz-przed-sobą--model-mentalny-dla-nowego-użytkownika)
2. [Pierwsze uruchomienie — orientacja w terminalu](#2-pierwsze-uruchomienie--orientacja-w-terminalu)
3. [CLAUDE.md — ustaw kontekst zanim cokolwiek zlecisz](#3-claudemd--ustaw-kontekst-zanim-cokolwiek-zlecisz)
4. [Przykładowe zadanie: Formularz Logowania](#4-przykładowe-zadanie-formularz-logowania)
   - 4a. Opis zadania i zakres
   - 4b. Planowanie: `/plan`
   - 4c. Zatwierdzanie akcji Claude
5. [Realizacja — BE: Backend Spring Boot](#5-realizacja--be-backend-spring-boot)
6. [Realizacja — DB: Baza Danych SQLite](#6-realizacja--db-baza-danych-sqlite)
7. [Realizacja — FE: Frontend React](#7-realizacja--fe-frontend-react)
8. [Realizacja — QA: Testy](#8-realizacja--qa-testy)
9. [Weryfikacja całości: `/verify`](#9-weryfikacja-całości-verify)
10. [Wywołanie pierwszego agenta: `@java-reviewer`](#10-wywołanie-pierwszego-agenta-java-reviewer)
11. [Zarządzanie sesją: `/cost`, `/compact`, `/clear`](#11-zarządzanie-sesją-cost-compact-clear)
12. [Typowe błędy pierwszego dnia i jak z nich wyjść](#12-typowe-błędy-pierwszego-dnia-i-jak-z-nich-wyjść)
13. [Workflow na pierwsze 3 godziny hackathonu](#13-workflow-na-pierwsze-3-godziny-hackathonu)

---

## 1. Co masz przed sobą — model mentalny dla nowego użytkownika

### Claude Code to nie chatbot

Kiedy piszesz do Claude na stronie claude.ai — rozmawiasz z AI przez przeglądarkę. Claude widzi tylko to co wkleisz w okno czatu. Nie ma dostępu do twoich plików.

Claude Code to coś fundamentalnie innego. Działa w twoim terminalu i ma bezpośredni dostęp do systemu plików, może uruchamiać komendy (`mvn`, `npm`, `docker`), czytać i pisać pliki w twoim projekcie, otwierać wiele plików jednocześnie i śledzić zmiany.

Właściwa analogia: Claude Code to **junior developer siedzący przy twoim komputerze**, który ma dostęp do całego projektu i może sam uruchamiać build. Ty jesteś tech leadem — dajesz zadania, zatwierdzasz decyzje, pilnujesz kierunku.

### ECC to "zestaw narzędzi" dla tego developera

Bez ECC Claude Code wie jak kodować, ale nie zna waszego projektu, nie ma gotowych procedur dla Spring Boot, nie wie jak robić TDD w Javie.

ECC instaluje:
- **Rules** — zasady których Claude zawsze przestrzega (np. "zawsze pisz testy przed kodem")
- **Skills** — gotowe playbooki workflow (np. "jak implementować w Spring Boot")
- **Agents** — wyspecjalizowani eksperci do których Claude deleguje (np. `@java-reviewer`)
- **Commands** — komendy slash które uruchamiasz sam (`/plan`, `/verify`, `/tdd`)

### Co Claude widzi gdy uruchamiasz go w projekcie

```
cd C:\projects\hackathon-app
claude
```

W tym momencie Claude automatycznie czyta:
- `CLAUDE.md` w bieżącym katalogu — briefing projektu (najważniejszy)
- `~/.claude/settings.json` — twoje ustawienia modelu
- `~/.claude/rules/` — zawsze-aktywne zasady (z ECC)
- `~/.claude/agents/` — dostępni eksperci
- `~/.claude/skills/` — dostępne playbooki

**Jeśli nie ma `CLAUDE.md` — Claude pracuje "na ślepo"** i będzie zgadywał jak wygląda projekt.

---

## 2. Pierwsze uruchomienie — orientacja w terminalu

### Uruchomienie

Otwórz Windows Terminal (nie PowerShell z menu Start — Windows Terminal który zainstalowałeś). Nawiguj do katalogu projektu i uruchom:

```powershell
cd C:\projects\hackathon-app
claude
```

### Co zobaczysz

```
╭─────────────────────────────────────────────────╮
│ ✻ Welcome to Claude Code!                       │
│                                                 │
│   /help for help                                │
│   /status for account info                      │
│                                                 │
│ cwd: C:\projects\hackathon-app                  │
│ model: claude-sonnet-4                          │
╰─────────────────────────────────────────────────╯

>
```

Znak `>` to prompt — tutaj piszesz. Claude czeka na twoje polecenie.

### Pierwsze komendy żeby się oswoić

Wpisz każdą i naciśnij Enter. Obserwuj co się dzieje:

```
> /help
```
Wyświetla listę wszystkich dostępnych komend. Użyj strzałek do scrollowania.

```
> /status
```
Pokazuje na jakim koncie jesteś zalogowany i jaki model jest aktywny.

```
> hello, what can you do?
```
Możesz pisać po angielsku lub polsku. Claude odpowie i opisze czym jest.

### Jak wyjść z Claude Code

```
> /exit
```
lub naciśnij `Ctrl+D`.

### Skróty klawiszowe które musisz znać

```
Ctrl+C          przerwij bieżące działanie (Claude przestaje generować/wykonywać)
                NIE zamyka programu — Claude czeka na następne polecenie

Ctrl+D          wyjście z Claude Code

↑ / ↓           historia poprzednich wpisanych komend (jak w terminalu)

/               wpisz sam slash — pojawi się lista komend do wyboru
Tab             autocomplete po wpisaniu początku komendy (np. "/pl" → "/plan")
```

---

## 3. CLAUDE.md — ustaw kontekst zanim cokolwiek zlecisz

### Dlaczego to pierwsza rzecz do zrobienia

Wyobraź sobie że zatrudniasz developera i pierwszego dnia mówisz mu tylko "napisz feature". Nie wie jak wygląda projekt, jakiego stylu używacie, co jest już zrobione. Będzie zgadywał.

`CLAUDE.md` to briefing dla Claude — odpowiednik onboarding dokumentu nowego developera. Claude czyta go przy każdym starcie sesji.

### Stwórz CLAUDE.md przed pierwszym zadaniem

W katalogu głównym projektu stwórz plik `CLAUDE.md`. Możesz to zrobić przez VS Code lub przez Claude Code samego:

```
> Stwórz plik CLAUDE.md w bieżącym katalogu z następującą zawartością:
```

Lub stwórz go ręcznie w VS Code i wklej ten szablon:

```markdown
# HackApp — Instrukcja projektu dla Claude Code

## O projekcie
[Opisz w 2-3 zdaniach co robi aplikacja i kto jej używa]

## Stack technologiczny
- Backend: Java 21 + Spring Boot 3.x + Maven
- Baza danych: SQLite (via Spring Data JPA)
  - Dialekt JPA: org.hibernate.community.dialect.SQLiteDialect
  - WAŻNE: używamy INTEGER PRIMARY KEY AUTOINCREMENT, nigdy SEQUENCE
- Frontend: React 18 + TypeScript + Vite + axios
- Konteneryzacja: Docker + Docker Compose
- Testy BE: JUnit 5, MockMvc, @SpringBootTest
- Testy FE/E2E: Playwright (TypeScript)

## Struktura katalogów
```
/
├── backend/
│   └── src/
│       ├── main/java/com/hackapp/
│       │   ├── controller/    REST controllers
│       │   ├── service/       logika biznesowa
│       │   ├── repository/    dostęp do danych
│       │   ├── model/         encje JPA
│       │   ├── dto/           obiekty transferu danych
│       │   └── config/        konfiguracja Spring
│       └── test/
├── frontend/
│   └── src/
│       ├── components/
│       ├── pages/
│       ├── api/               axios client
│       └── hooks/
├── db/
│   ├── schema.sql
│   └── seed.sql
└── docker-compose.yml
```

## Zasady kodowania (ZAWSZE przestrzegaj)
- TDD: najpierw test, potem implementacja
- Klasy Java: max 200 linii, metody: max 30 linii
- REST: prefiks /api/v1/, odpowiedzi JSON z polem status i data lub error
- Lombok: @Data, @Builder, @RequiredArgsConstructor
- Wyjątki: własne klasy exception, handler @ControllerAdvice
- Brak hardcoded wartości — wszystko w application.properties
- Git commit format: feat:, fix:, test:, refactor:

## API Contract (aktualizuj na bieżąco)
[Tu będziemy dodawać endpointy w trakcie hackathonu]

## MVP — co musi działać
[Tu wpisz 3 główne user flows]
```

### Zatwierdź że Claude go widzi

```
> Przeczytaj CLAUDE.md i potwierdź że rozumiesz stack projektu
```

Claude powinien odpowiedzieć podsumowaniem — jeśli odpowiedź jest spójna z tym co wpisałeś, kontekst zadziałał.

---

## 4. Przykładowe zadanie: Formularz Logowania

### 4a. Opis zadania i zakres

Zadanie: Zaimplementuj kompletny **moduł logowania** dla aplikacji webowej.

Zakres modułu (co musi działać):
- Użytkownik wypełnia formularz login + hasło w przeglądarce
- Frontend wysyła dane do REST API
- Backend weryfikuje credentials z bazą danych
- Jeśli poprawne — zwraca token sesji (cookie lub JWT)
- Jeśli niepoprawne — zwraca błąd 401
- Frontend pokazuje odpowiedni komunikat

Granice (czego NIE robimy w tym zadaniu):
- Rejestracja nowych użytkowników — osobny moduł
- Reset hasła — osobny moduł
- OAuth / SSO — osobny moduł

**Podział na warstwy:**

```
DB Dev:     tabela users (id, username, password_hash, created_at)
            seed data z 2-3 testowymi użytkownikami

BE Dev:     endpoint POST /api/v1/auth/login
            UserRepository, AuthService, AuthController
            hashowanie haseł (BCrypt)
            odpowiedź: token sesji lub błąd

FE Dev:     komponent LoginPage z formularzem
            hook useAuth do komunikacji z API
            obsługa błędów i loading state
            przekierowanie po udanym logowaniu

QA:         test integracyjny happy path (login OK)
            test integracyjny error path (złe hasło → 401)
            smoke test docker-compose
```

---

### 4b. Planowanie: `/plan`

Komenda `/plan` to pierwszy krok dla każdego zadania. Nigdy nie zaczynajcie implementować bez planu — Claude może pojechać w złym kierunku i stracicie czas.

**Jak `/plan` działa:**
1. Opisujesz zadanie
2. Claude analizuje projekt (czyta CLAUDE.md i istniejące pliki)
3. Claude pokazuje szczegółowy plan — lista plików, kolejność, zależności
4. **Czeka na twoje zatwierdzenie zanim dotknie kodu**
5. Dopiero po zatwierdzeniu implementuje

**Ważne:** Każda rola uruchamia `/plan` osobno, dla swojej warstwy.

---

### 4c. Zatwierdzanie akcji Claude

Gdy Claude chce zmodyfikować plik, zobaczysz prompt:

```
Claude wants to:
  Create file: backend/src/main/java/com/hackapp/model/User.java

[y] Yes  [n] No  [a] Always allow this type of action  [d] Don't allow  [e] Edit first
Your choice (y/n/a/d/e):
```

**Co oznacza każda opcja:**

```
y — zatwierdź tę jedną akcję, następna znowu zapyta
    → używaj gdy chcesz kontrolować każdy krok

a — zatwierdź TEN TYP akcji na całą sesję
    → np. "a" przy tworzeniu pliku Java = Claude może tworzyć
      wszystkie pliki Java bez pytania
    → BEZPIECZNE dla tworzenia i edycji plików projektu
    → NIE używaj przy komendach bash które mogą być destrukcyjne

n — odmów tej akcji
    → Claude spróbuje innego podejścia lub zapyta jak kontynuować

e — edytuj zanim wykona
    → Claude pokazuje co planuje zapisać, możesz zmienić

d — zablokuj ten typ akcji na sesję
    → używaj jeśli Claude próbuje czegoś czego nie chcesz (np. rm -rf)
```

**Rekomendacja na hackathon:**
- Przy tworzeniu plików: wciskaj `a` — oszczędza czas
- Przy `bash` komendach: czytaj co Claude chce uruchomić, potem `y` lub `n`
- Nigdy `a` dla komend bash bez przeczytania

---

## 5. Realizacja — BE: Backend Spring Boot

### Kto to robi
BE Dev #1 (Tech Lead) lub BE Dev #2 — w zależności od podziału.

### Krok 1: Uruchom Claude Code w katalogu projektu

```powershell
cd C:\projects\hackathon-app
claude
```

### Krok 2: Zaplanuj zadanie BE

Wpisz dokładnie to (lub dostosuj do swojego projektu):

```
/plan "Zaimplementuj warstwę backend dla modułu logowania.

Zakres:
- Encja User (id Long, username String UNIQUE, passwordHash String, createdAt LocalDateTime)
- UserRepository extends JpaRepository<User, Long> z metodą findByUsername
- AuthService z metodą login(username, password):
    - pobiera usera z repo
    - weryfikuje BCrypt hash
    - jeśli OK: zwraca LoginResponseDto z tokenem sesji (UUID jako sessionToken)
    - jeśli błąd: rzuca InvalidCredentialsException (→ 401)
- AuthController z endpointem POST /api/v1/auth/login
    - przyjmuje LoginRequestDto (username, password)
    - zwraca LoginResponseDto (sessionToken, username) lub błąd
- GlobalExceptionHandler (@ControllerAdvice) obsługujący InvalidCredentialsException
- LoginRequestDto, LoginResponseDto

Ograniczenia:
- SQLite — użyj INTEGER PRIMARY KEY AUTOINCREMENT
- Lombok na wszystkich klasach
- Bean Validation na DTO (@NotBlank)
- NIE implementuj JWT — prosty UUID session token wystarczy na hackathon
- TDD: zacznij od testów"
```

### Co zobaczysz po `/plan`

Claude wygeneruje plan podobny do tego:

```
I'll implement the backend authentication layer. Here's my plan:

Files to CREATE:
1. backend/src/main/java/com/hackapp/model/User.java
2. backend/src/main/java/com/hackapp/repository/UserRepository.java
3. backend/src/main/java/com/hackapp/dto/LoginRequestDto.java
4. backend/src/main/java/com/hackapp/dto/LoginResponseDto.java
5. backend/src/main/java/com/hackapp/exception/InvalidCredentialsException.java
6. backend/src/main/java/com/hackapp/service/AuthService.java
7. backend/src/main/java/com/hackapp/controller/AuthController.java
8. backend/src/main/java/com/hackapp/config/GlobalExceptionHandler.java
9. backend/src/test/java/com/hackapp/controller/AuthControllerTest.java

Files to MODIFY:
10. backend/pom.xml — dodanie spring-security-crypto (BCrypt)

Implementation order:
  Step 1: Exceptions → Step 2: Model → Step 3: DTO →
  Step 4: Repository → Step 5: Service → Step 6: Controller →
  Step 7: ExceptionHandler → Step 8: Tests

Shall I proceed?
```

### Krok 3: Oceń plan i zatwierdź lub skoryguj

Przejrzyj plan. Jeśli coś jest nie tak, popraw TERAZ — zanim Claude zacznie pisać:

```
# Przykład korekty:
"Nie dodawaj spring-security-crypto do pom.xml —
mamy już spring-boot-starter-security. Kontynuuj z tym założeniem."

# Przykład zatwierdzenia:
"Plan wygląda dobrze. Wykonaj."
```

### Krok 4: Obserwuj implementację

Claude zacznie tworzyć pliki jeden po drugim. Przy każdym nowym pliku zobaczysz prompt zatwierdzający. Wciśnij `a` przy pierwszym pytaniu o stworzenie pliku Java — wtedy nie będzie pytać przy kolejnych.

Podczas implementacji możesz:
- Czytać generowany kod na bieżąco
- Przerwać `Ctrl+C` i dodać komentarz jeśli coś jest nie tak
- Nie przerywać i dać Claude dokończyć — potem zrobić review

### Krok 5: Sprawdź rezultat

Po zakończeniu:

```
> /diff
```

Zobaczysz wszystkie zmiany. Przewiń i przeczytaj kluczowe pliki — szczególnie `AuthService.java` i `AuthController.java`.

```
> mvn compile
```

Wpisz to bezpośrednio (Claude może uruchomić to za ciebie, albo możesz sam). Powinno być `BUILD SUCCESS`.

### Krok 6: Code review przez agenta

```
> @java-reviewer Przejrzyj AuthService.java i AuthController.java.
Szczególnie sprawdź: bezpieczeństwo hashowania haseł,
obsługę wyjątków, czy @Transactional jest użyte poprawnie"
```

Agent `@java-reviewer` przejrzy kod i da ci listę uwag z numerami linii.

---

## 6. Realizacja — DB: Baza Danych SQLite

### Kto to robi
DB Dev — ta osoba powinna skończyć swoją część **jako pierwsza**, bo BE nie może pisać bez schematu.

### Krok 1: Uruchom Claude Code

```powershell
cd C:\projects\hackathon-app
claude
```

### Krok 2: Zaplanuj zadanie DB

```
/plan "Przygotuj schemat bazy danych dla modułu logowania.

Zakres:
- Tabela users:
    id INTEGER PRIMARY KEY AUTOINCREMENT
    username TEXT NOT NULL UNIQUE
    password_hash TEXT NOT NULL
    created_at TEXT NOT NULL DEFAULT (datetime('now'))
- Plik db/schema.sql — definicja tabeli
- Plik db/seed.sql — 3 testowych użytkowników z hashami BCrypt
  (użyj realnych BCrypt hashów dla haseł: admin123, user123, test123)
- Plik db/cleanup-test.sql — DELETE FROM users; (do czyszczenia między testami)

Ograniczenia:
- Czysty SQLite — bez PostgreSQL-owych konstrukcji
- Żadnych SEQUENCE, żadnych SERIAL
- TEXT zamiast VARCHAR (SQLite nie rozróżnia)
- Indeks na username (dla szybkiego wyszukiwania przy logowaniu)"
```

### Co zobaczysz po `/plan`

```
Files to CREATE:
1. db/schema.sql
2. db/seed.sql
3. db/cleanup-test.sql

Content preview:

schema.sql:
  CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    username TEXT NOT NULL UNIQUE,
    password_hash TEXT NOT NULL,
    created_at TEXT NOT NULL DEFAULT (datetime('now'))
  );
  CREATE INDEX IF NOT EXISTS idx_users_username ON users(username);

seed.sql:
  INSERT OR IGNORE INTO users (username, password_hash) VALUES
    ('admin', '$2a$10$...'),   -- admin123
    ('user1', '$2a$10$...'),   -- user123
    ('tester', '$2a$10$...');  -- test123

Shall I proceed?
```

### Krok 3: Zatwierdź i poczekaj na pliki

```
"Zatwierdź. Upewnij się że hasze BCrypt są prawdziwe i poprawnie wygenerowane
(powinny zaczynać się od $2a$10$)"
```

### Krok 4: Weryfikacja schematu przez agenta

```
> "@database-reviewer Sprawdź db/schema.sql pod kątem:
SQLite best practices, poprawności typów danych,
czy indeks na username jest potrzebny i prawidłowy"
```

### Krok 5: Zakomunikuj zespołowi

Po zatwierdzeniu schematu — **powiedz o tym reszcie zespołu**. BE Dev może teraz zacząć pisać encje i repository. To jest krytyczna zależność.

Wpisz w CLAUDE.md (lub TASKS.md) że schemat jest gotowy:

```
> "Dodaj do sekcji 'API Contract' w CLAUDE.md informację:
Tabela users: id, username, password_hash, created_at — schema w db/schema.sql, GOTOWE"
```

---

## 7. Realizacja — FE: Frontend React

### Kto to robi
BE Dev #2 lub osoba odpowiedzialna za frontend.

### Kiedy zaczynać
Frontend może zacząć równolegle z backendem — wystarczy że znasz kontrakt API (endpoint i format requestu/response). Nie musisz czekać aż BE skończy implementację.

**Kontrakt API który musisz znać zanim zaczniesz:**
```
POST /api/v1/auth/login
Request:  { "username": "string", "password": "string" }
Response OK (200):    { "status": "success", "data": { "sessionToken": "uuid", "username": "string" } }
Response ERR (401):   { "status": "error", "error": "Invalid credentials" }
```

### Krok 1: Uruchom Claude Code

```powershell
cd C:\projects\hackathon-app
claude
```

### Krok 2: Zaplanuj zadanie FE

```
/plan "Zaimplementuj frontend dla modułu logowania w React + TypeScript.

Zakres:
- src/api/auth.ts
    Funkcja loginUser(username, password): Promise<LoginResponse>
    axios POST do /api/v1/auth/login
    Obsługa błędów sieciowych i 401
    
- src/hooks/useAuth.ts
    Hook zarządzający stanem logowania
    Stan: { isLoading, error, user }
    Akcja: login(username, password)
    Po udanym logowaniu: zapisuje sessionToken w localStorage
    
- src/pages/LoginPage.tsx
    Formularz: dwa pola (username, password), przycisk Login
    Stany UI: loading spinner podczas wysyłania, komunikat błędu przy 401
    Po sukcesie: przekierowanie na /dashboard (react-router-dom)
    Prosty wygląd — nie trać czasu na CSS, ważna funkcjonalność

- src/types/auth.ts
    Interfejsy TypeScript: LoginRequest, LoginResponse, AuthUser

Ograniczenia:
- NIE używaj any w TypeScript
- Obsługa błędów przez try/catch — nie pozwól żeby błąd API crashował aplikację
- axios baseURL: http://localhost:8080 (backend Docker)
- Formularz: kontrolowane inputy (React state), nie uncontrolled
- Brak bibliotek UI (Material, Tailwind etc.) — czysty HTML + minimal CSS"
```

### Krok 3: Oceń plan i zatwierdź

Przejrzyj — szczególnie sprawdź czy Claude zaplanował obsługę błędów i loading state.

```
"Plan zatwierdzam. Zacznij od auth.ts i typów, potem hook, na końcu komponent"
```

### Krok 4: Obserwuj i weryfikuj

Po implementacji:

```
> cd frontend && npm run dev
```

Otwórz przeglądarkę na `http://localhost:3000` — powinieneś zobaczyć formularz logowania.

Jeśli backend jeszcze nie działa — to normalne. Formularz powinien się wyrenderować i pokazać błąd sieciowy (nie crash aplikacji) gdy klikniesz Login.

### Krok 5: Code review

```
> "@code-reviewer Przejrzyj src/hooks/useAuth.ts i src/pages/LoginPage.tsx.
Sprawdź: obsługę błędów, poprawność TypeScript typów,
czy loading state jest obsługiwany we wszystkich przypadkach"
```

---

## 8. Realizacja — QA: Testy

### Kto to robi
QA Tester — zaczyna pisać testy zaraz po tym jak BE ma zaimplementowany endpoint (lub nawet wcześniej, w trybie TDD).

### Podejście: najpierw happy path, potem edge cases

Na hackathonie priorytet jest jasny:
1. Test że logowanie działa (happy path) — **OBOWIĄZKOWE**
2. Test że złe hasło daje 401 — **OBOWIĄZKOWE**
3. Test że brak pola daje 400 — ważne
4. Test że puste pole daje 400 — jeśli czas

### Krok 1: Uruchom Claude Code

```powershell
cd C:\projects\hackathon-app
claude
```

### Krok 2: Zaplanuj testy (tryb TDD)

```
/tdd

"Napisz testy integracyjne Spring Boot dla modułu logowania.

Zakres testów:
1. AuthControllerTest — @SpringBootTest + MockMvc:
   - loginHappyPath: POST /api/v1/auth/login z poprawnymi danymi
     → oczekuj: 200, body.status == 'success', body.data.sessionToken nie null
   - loginWrongPassword: POST /api/v1/auth/login ze złym hasłem
     → oczekuj: 401, body.status == 'error'
   - loginUserNotFound: POST /api/v1/auth/login z nieistniejącym username
     → oczekuj: 401 (nie 404 — nie ujawniamy czy user istnieje)
   - loginMissingUsername: POST bez pola username
     → oczekuj: 400

Wymagania techniczne:
- @SpringBootTest(webEnvironment = RANDOM_PORT)
- @AutoConfigureMockMvc
- @Sql(scripts='/db/schema.sql') przed każdym testem — ładuje schemat
- @Sql(scripts='/db/seed.sql') przed każdym testem — dane testowe
- @Sql(scripts='/db/cleanup-test.sql', executionPhase=AFTER_TEST_METHOD)
- ObjectMapper do serializacji request body
- Każdy test niezależny — czysci po sobie
- Testy mają opisowe nazwy (co testujesz + oczekiwany rezultat)"
```

### Co zobaczysz

Claude napisze testy, a następnie (w trybie TDD) sprawdzi czy przechodzą:

```
[TDD — RED phase]
Writing failing tests first...

AuthControllerTest.java created.

Running tests to confirm they fail (expected):
  mvn test -pl backend -Dtest=AuthControllerTest
  
  RESULTS:
  ✗ loginHappyPath — FAIL (AuthController not yet implemented)
  ✗ loginWrongPassword — FAIL
  ✗ loginUserNotFound — FAIL
  ✗ loginMissingUsername — FAIL

Tests failing as expected. Ready for GREEN phase when BE implements endpoint.
```

To jest prawidłowe zachowanie w TDD. Testy powinny "czerwone" zanim BE zaimplementuje kod.

### Krok 3: Koordynacja z BE

Gdy BE skończy implementację:

```
> "BE skończyło implementację AuthController.
Uruchom testy: mvn test -pl backend -Dtest=AuthControllerTest"
```

Claude uruchomi testy i pokaże wyniki. Jeśli coś nie przechodzi, naprowadzi na przyczynę.

### Krok 4: Smoke test dla Docker

```
> "Napisz skrypt PowerShell check-stack.ps1 który:
1. Uruchamia docker-compose up -d --build
2. Czeka max 30 sekund na health check backendu (GET /actuator/health)
3. Wykonuje POST /api/v1/auth/login z danymi: admin / admin123
4. Sprawdza że odpowiedź ma status 200 i pole sessionToken
5. Zatrzymuje kontenery: docker-compose down
6. Wypisuje PASS lub FAIL z podsumowaniem"
```

Ten skrypt uruchamiajcie przed każdym synciem zespołu i przed demo.

### Krok 5: Walidacja całego przepływu

Po tym jak BE i FE są gotowe:

```
> /e2e "Napisz test Playwright (TypeScript) dla przepływu logowania:
1. Otwórz http://localhost:3000/login
2. Wpisz username: admin, password: admin123
3. Kliknij przycisk Login
4. Oczekuj przekierowania na /dashboard (url powinien się zmienić)
5. Oczekuj że nie ma komunikatu błędu

Drugi test:
1. Otwórz /login
2. Wpisz username: admin, password: blednehaslo
3. Kliknij Login
4. Oczekuj że url NADAL jest /login
5. Oczekuj że jest widoczny tekst błędu (np. 'Invalid credentials')"
```

---

## 9. Weryfikacja całości: `/verify`

Po tym jak wszystkie warstwy są gotowe — łączycie je razem i weryfikujecie jako całość.

### Kto to robi
BE Lead — ma największy wgląd w całość.

### Krok 1: Uruchom pełną weryfikację

```
> /verify
```

Claude uruchomi sekwencję:

```
[1/5] Compiling backend...
  → mvn compile -pl backend
  ✓ BUILD SUCCESS (0 errors)

[2/5] Running unit tests...
  → mvn test -pl backend
  ✓ 4/4 tests passing

[3/5] Building Docker images...
  → docker-compose build
  ✓ backend image built
  ✓ frontend image built

[4/5] Starting stack...
  → docker-compose up -d
  ✓ backend healthy (GET /actuator/health → 200)
  ✓ frontend serving (GET http://localhost:3000 → 200)

[5/5] Integration smoke test...
  → POST /api/v1/auth/login {username:admin, password:admin123}
  ✓ Response: 200, sessionToken present

Verification PASSED ✓
```

Jeśli któryś krok się posypie, Claude automatycznie próbuje naprawić problem i restartuje weryfikację.

### Krok 2: Jeśli `/verify` znajdzie problem

Claude powie co jest nie tak. Możesz:

```
# Dać mu naprawić:
"Napraw ten błąd i uruchom verify ponownie"

# Lub sam zdiagnozować:
"Nie naprawiaj — wyjaśnij mi co jest przyczyną tego błędu"
```

---

## 10. Wywołanie pierwszego agenta: `@java-reviewer`

### Czym jest agent vs komenda

Komenda (`/plan`, `/verify`) to ty inicjujesz konkretny workflow.

Agent (`@java-reviewer`, `@architect`) to wyspecjalizowany ekspert — wysyłasz do niego konkretne pytanie lub prośbę o analizę.

### Jak wywołać agenta

```
> "@java-reviewer [twoje polecenie]"
```

Ważne: cudzysłów nie jest wymagany, ale nazwę agenta poprzedź zawsze `@`.

### Przykłady wywołania dla modułu logowania

```
# Po napisaniu AuthService:
> @java-reviewer Przejrzyj AuthService.java.
Sprawdź czy BCrypt jest używany prawidłowo i
czy exception handling jest kompletny

# Po skończeniu całego BE:
> @java-reviewer Przejrzyj cały pakiet com.hackapp.
Szukaj: N+1 queries, brakujących @Transactional,
potencjalnych NullPointerException

# Gdy masz wątpliwości architektoniczne:
> @architect Czy lepiej trzymać session token w tabeli SQL
czy in-memory (np. ConcurrentHashMap)?
Mamy SQLite i jeden serwer — co rekomendujesz dla hackathonu?

# Gdy testy padają i nie wiesz czemu:
> @java-build-resolver Testy padają z tym błędem:
[wklej pełny stacktrace]
Co jest przyczyną i jak naprawić?
```

### Co dostaniesz od agenta

Agent `@java-reviewer` daje output w formacie:

```
Code Review — AuthService.java

🔴 CRITICAL (napraw przed commitem):
  Line 34: Plaintext password logged in catch block — security risk
  → Remove: log.debug("Login failed for: {}", password)
  → Keep only: log.debug("Login failed for user: {}", username)

🟡 WARNING (napraw jeśli czas):
  Line 28: Missing @Transactional(readOnly = true) on findByUsername query
  → Add annotation to improve SQLite performance

🟢 INFO (rozważ):
  Line 15: Consider extracting BCrypt rounds to application.properties
  → spring.security.bcrypt.strength=10
```

Wróć do kodu, nanieś poprawki, możesz ponownie wywołać agenta żeby potwierdzić.

---

## 11. Zarządzanie sesją: `/cost`, `/compact`, `/clear`

### Dlaczego to ważne

Claude Code ma okno kontekstowe — jak pamięć krótkotrwała. Im dłużej pracujesz bez resetu, tym więcej "pamięci" zużywa sesja. Przy pełnym oknie: Claude zaczyna "zapominać" wcześniejszy kontekst, odpowiedzi są mniej spójne, koszty rosną.

### `/cost` — sprawdź zużycie

```
> /cost
```

Output:
```
Session cost: $0.34
Input tokens:  28,450
Output tokens:  9,120
Context usage: 47%
Cache hits: 81%
```

**Reguła:** Sprawdzaj `/cost` co 60-90 minut. Jeśli Context usage > 70% — czas na `/compact`.

### `/compact` — skompresuj i kontynuuj

```
> /compact
```

Lub z wskazówką co jest ważne:

```
> /compact "Zachowaj kontekst: AuthService, AuthController, schemat bazy, kontrakt API"
```

**Kiedy używać `/compact`:**
- Po skończeniu warstwy (np. BE), przed przejściem do następnej (FE)
- Gdy context usage > 70%
- Gdy sesja trwa >2 godziny
- Przed nowym dużym zadaniem

**Co `/compact` zachowuje:** Kluczowe decyzje techniczne, nazwy plików i klas, aktualny stan implementacji, błędy które już naprawiono.

**Co `/compact` usuwa:** Długie wymiany pytań i odpowiedzi, wygenerowany kod (jest w plikach, nie musi być w pamięci), powtórzone informacje.

### `/clear` — zacznij od zera

```
> /clear
```

Czyści całą historię sesji. Claude nie pamięta nic z poprzedniej rozmowy. Pliki projektu zostają — zmieniane tylko historia konwersacji.

**Kiedy używać `/clear`:**
- Zmieniasz temat kompletnie (z backendu na zupełnie inny moduł)
- Claude zaczyna dawać coraz gorsze odpowiedzi (zagubił kontekst)
- Po długim debugowaniu które poszło w złym kierunku — świeży start
- Przed demo — czyste środowisko

**Ryzyko `/clear`:** Tracisz kontekst sesji. Dlatego PRZED `/clear`:

```
> "Podaj podsumowanie co zrobiliśmy:
1. Jakie pliki zostały stworzone/zmodyfikowane
2. Jakie decyzje techniczne podjęliśmy  
3. Co jest todo
4. Aktualny status testów"
```

Skopiuj odpowiedź do `CLAUDE.md` lub `TASKS.md`. Po `/clear` wklej to jako pierwszy prompt nowej sesji.

### Strategia dla hackathonu

```
H0:00 — Start sesji: czytasz CLAUDE.md, zaczynasz zadanie
H1:30 — /cost → jeśli >60%, /compact z kontekstem
H3:00 — koniec zadania: /compact lub /clear przed nowym modułem
H5:00 — sync zespołu: każdy sprawdza /cost
H7:00 — przed demo: /clear na wszystkich maszynach, świeże sesje
```

---

## 12. Typowe błędy pierwszego dnia i jak z nich wyjść

### Błąd #1: "Claude generuje kod którego nie chcę"

**Objawy:** Claude zaczyna implementować coś innego niż prosiłeś, dodaje niepotrzebne rzeczy, zmienia istniejący kod którego nie miał dotykać.

**Przyczyna:** Za mało precyzyjny prompt, brakuje ograniczeń (`NIE rób X`).

**Rozwiązanie:**
```
Ctrl+C  ← natychmiast przerwij

> "Stop. Cofnij ostatnie zmiany. Nie edytuj więcej plików
bez mojego zatwierdzenia.
[opisz dokładnie co chciałeś]"
```

Jeśli Claude już zapisał pliki:
```
> /diff   ← sprawdź co zmienił
> "Cofnij zmiany w [nazwa pliku] — wróć do poprzedniej wersji"
```

Lub przez git:
```powershell
git diff          # co się zmieniło
git checkout -- backend/src/main/java/com/hackapp/service/AuthService.java
```

---

### Błąd #2: "Testy padają i Claude nie potrafi ich naprawić"

**Objawy:** Claude naprawia jeden test → psuje drugi → naprawia ten → psuje poprzedni. Koło się kręci.

**Przyczyna:** Claude stracił kontekst dlaczego testy padają, próbuje naprawić symptomy zamiast przyczyny.

**Rozwiązanie:**
```
> /clear

> "@java-build-resolver Testy padają z tym błędem:
[pełny stacktrace z mvn test]

Kontekst projektu:
- Spring Boot 3.x + SQLite
- AuthController używa AuthService
- Testy używają @SpringBootTest i @Sql dla seedowania

Zdiagnozuj root cause — nie naprawiaj nic zanim nie wytłumaczysz mi co jest przyczyną"
```

---

### Błąd #3: "Claude ignoruje CLAUDE.md"

**Objawy:** Claude generuje kod w złym stylu, używa PostgreSQL-specyficznych konstrukcji, nie stosuje Lombok, robi rzeczy których zabroniliście.

**Przyczyna:** Długa sesja — CLAUDE.md jest daleko w historii kontekstu i Claude "zapomniał".

**Rozwiązanie:**
```
> /compact "Przypomnij sobie CLAUDE.md — jesteśmy na Spring Boot + SQLite,
używamy Lombok, INTEGER PRIMARY KEY AUTOINCREMENT, TDD"
```

Lub:
```
> "Przeczytaj ponownie CLAUDE.md i potwierdź jakich zasad przestrzegasz"
```

---

### Błąd #4: "Claude Code uruchomił się ale `/plan` nie działa"

**Objawy:** Wpisujesz `/plan` i nie dzieje się nic, lub widzisz "command not found".

**Przyczyna:** ECC nie jest zainstalowany lub plugin nie jest aktywny.

**Rozwiązanie:**
```
> /help
# Sprawdź czy widzisz plan w liście komend

> /plugin list everything-claude-code@everything-claude-code
# Powinno pokazać listę komend ECC

# Jeśli plugin nie jest zainstalowany:
> /plugin marketplace add affaan-m/everything-claude-code
> /plugin install everything-claude-code@everything-claude-code
```

Jeśli polecenia plugin nie działają:
```powershell
# Sprawdź czy commands są w katalogu:
ls $env:USERPROFILE\.claude\commands\
# Powinny być pliki: plan.md, tdd.md, verify.md etc.

# Jeśli brak — wróć do VM Setup Guide i powtórz Krok D
```

---

### Błąd #5: "Docker Compose nie wstaje po zmianach kodu"

**Objawy:** `docker-compose up` startuje ale backend jest `unhealthy`, albo aplikacja zachowuje się jak stary kod.

**Przyczyna:** Docker używa zbuforowanego obrazu — nie przebudował po zmianie kodu.

**Rozwiązanie:**
```
> "Zatrzymaj Docker Compose, przebuduj obrazy i uruchom ponownie.
Użyj --no-cache żeby wymusić pełny rebuild"
```

Lub ręcznie:
```powershell
docker-compose down
docker-compose build --no-cache
docker-compose up -d
```

---

## 13. Workflow na pierwsze 3 godziny hackathonu

To jest dokładna sekwencja kroków dla każdej roli. Nie zaczynajcie "na czuja" — trzymajcie się tej kolejności.

### T+0:00 — Kickoff (wszyscy razem, 30 minut)

```
1. Ustalcie Product Brief (co robicie, kto używa, 3 główne user flows)
2. Zapiszcie API contract na kartce/tablicy:
   - lista endpointów: metoda + ścieżka + request/response
3. Podzielcie TASKS.md na konkretne zadania per osoba
4. Każdy: git pull na świeżym main
```

### T+0:30 — Każdy odpala Claude Code (równolegle)

**Wszyscy wykonują:**
```powershell
cd C:\projects\hackathon-app
claude
```

Pierwsze polecenie KAŻDEGO:
```
> Przeczytaj CLAUDE.md i powiedz mi w 3 zdaniach co to za projekt i jaki jest stack
```

Jeśli Claude odpowiada spójnie z briefingiem — jesteś gotowy. Jeśli nie — zaktualizuj CLAUDE.md.

---

**DB Dev — sekwencja:**
```
T+0:30  /plan "Schemat tabeli users + seed data + cleanup script"
T+0:35  Review planu → zatwierdź
T+0:55  Implementacja gotowa → commit: "feat: add users schema and seed data"
T+1:00  Ogłoś zespołowi: "Schemat gotowy, możecie zaczynać BE"
T+1:05  @database-reviewer sprawdź schema.sql
T+1:15  Zacznij schemat dla następnej encji z domeny biznesowej
```

**BE Lead — sekwencja:**
```
T+0:30  /plan "Spring Boot init + docker-compose skeleton"
T+0:40  Review → zatwierdź → poczekaj na implementację
T+1:10  mvn compile → powinno BUILD SUCCESS
T+1:15  Ogłoś: "Backend startuje, health check działa"
T+1:20  /plan "AuthController + AuthService + User entity"
         (czekasz na info od DB że schema gotowa)
T+1:30  Review planu → zatwierdź
T+2:00  /verify → sprawdź czy testy przechodzą
T+2:05  @java-reviewer przejrzyj AuthService.java
T+2:15  git commit: "feat: implement login endpoint"
```

**BE Dev #2 — sekwencja:**
```
T+0:30  /plan "React app init + Vite + routing skeleton + axios config"
T+0:40  Review → zatwierdź
T+1:10  npm run dev → frontend startuje na :3000
T+1:15  Ogłoś: "Frontend skeleton gotowy"
T+1:20  /plan "LoginPage + useAuth hook + auth API client"
         (nie musisz czekać na BE — masz kontrakt API)
T+1:30  Review planu → zatwierdź
T+2:00  npm run dev → przetestuj formularz manualnie
T+2:10  @code-reviewer sprawdź LoginPage.tsx i useAuth.ts
T+2:20  git commit: "feat: implement login page"
```

**QA — sekwencja:**
```
T+0:30  /plan "Test harness: konfiguracja @SpringBootTest + @Sql"
T+0:45  Napisz szkielet klasy AuthControllerTest (puste metody testowe)
T+1:00  /tdd — zacznij pisać testy które celowo padają (RED phase)
T+1:30  Gdy BE ogłosi że endpoint gotowy:
         mvn test -Dtest=AuthControllerTest
T+1:40  Sprawdź wyniki, raportuj BE co nie przechodzi
T+2:00  /e2e — wygeneruj smoke test dla docker-compose
T+2:15  Uruchom check-stack.ps1
T+2:20  Raportuj status: "X/4 testy zielone, docker stack PASS/FAIL"
```

### T+2:00 — SYNC #1 (wszyscy razem, 15 minut)

Każda osoba odpowiada na 3 pytania:
1. Co zrobiłem w ostatnich 90 minutach?
2. Co planuję w następnych 90 minutach?
3. Czy jestem zablokowany przez kogoś?

Stan po sync #1 dla przykładu logowania:
```
✓ DB: schemat gotowy, seed data gotowe
✓ BE: health check działa, endpoint login zaimplementowany
✓ FE: formularz logowania wyrenderowany, komunikuje z BE
✓ QA: 3/4 testy zielone, 1 test na obsługę 400 w toku
Blocker: QA czeka na potwierdzenie formatu error response od BE
```

### T+2:15 — Każdy kontynuuje swój następny task

Po sync każdy wraca do Claude Code i kontynuuje według TASKS.md. Jeśli sesja trwała ponad 90 minut:

```
> /compact "Podsumuj co zrobiliśmy: [główne punkty z sync]"
```

---

## Szybka ściąga — sekwencja dla jednego zadania

```
┌─────────────────────────────────────────────────────────────┐
│ SEKWENCJA DLA KAŻDEGO ZADANIA                               │
├─────────────────────────────────────────────────────────────┤
│ 1. Upewnij się że CLAUDE.md jest aktualny                   │
│ 2. /plan "opis zadania z zakresem i ograniczeniami"         │
│ 3. Przeczytaj plan uważnie — popraw jeśli trzeba            │
│ 4. "Zatwierdź i wykonaj" / "Zmień X i wykonaj"              │
│ 5. Obserwuj implementację — Ctrl+C jeśli coś jest nie tak   │
│ 6. /diff — przejrzyj wszystkie zmiany                       │
│ 7. mvn compile / npm run dev — sprawdź że działa            │
│ 8. @java-reviewer / @code-reviewer — code review            │
│ 9. /verify — pełna weryfikacja                              │
│ 10. git add . && git commit -m "feat: opis"                 │
│ 11. /cost — sprawdź zużycie                                 │
│ 12. /compact jeśli >60% lub /clear przed nowym modułem      │
└─────────────────────────────────────────────────────────────┘
```

---

*Wersja: Hackathon 2026 · Login Module Example · Claude Code + ECC v1.9.0*
