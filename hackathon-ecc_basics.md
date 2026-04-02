# 🏆 Hackathon — Przewodnik Profesjonalnego Zespołu
### Java Spring · SQLite · React · Docker · Claude Max + Everything Claude Code

---

## Spis treści

1. [Filozofia podejścia](#1-filozofia-podejścia)
2. [Skład zespołu i podział ról](#2-skład-zespołu-i-podział-ról)
3. [Przygotowanie PRZED hackathon (D-2 do D-1)](#3-przygotowanie-przed-hackathon)
4. [Instalacja i konfiguracja Claude Code + ECC](#4-instalacja-i-konfiguracja-claude-code--ecc)
5. [Struktura projektu i CLAUDE.md](#5-struktura-projektu-i-claudemd)
6. [Planowanie pracy podczas hackathonu](#6-planowanie-pracy-podczas-hackathonu)
7. [Efektywne użycie Claude Code w trakcie](#7-efektywne-użycie-claude-code-w-trakcie)
8. [Workflow i komendy ECC dla waszego stacku](#8-workflow-i-komendy-ecc-dla-waszego-stacku)
9. [Tester automatyczny — maksymalne wykorzystanie](#9-tester-automatyczny--maksymalne-wykorzystanie)
10. [Zarządzanie kontekstem i tokenami](#10-zarządzanie-kontekstem-i-tokenami)
11. [Opcjonalne rozszerzenia — API / Kafka](#11-opcjonalne-rozszerzenia--api--kafka)
12. [Timeline hackathonu — godzina po godzinie](#12-timeline-hackathonu--godzina-po-godzinie)
13. [Najczęstsze pułapki i jak ich unikać](#13-najczęstsze-pułapki-i-jak-ich-unikać)
14. [Checklista startowa](#14-checklista-startowa)

---

## 1. Filozofia podejścia

### AI jako junior developer na sterydach

Claude Code z ECC to nie autocompletion — to **autonomiczny agent**, który może:
- Samodzielnie zaimplementować całe moduły po jednym prompcie
- Naprawiać błędy budowania bez twojego udziału
- Generować testy, dokumentację, migracje bazy równolegle
- Reviewować kod zgodnie z ustalonymi regułami projektowymi

Kluczowa zmiana mentalna: **nie piszecie kodu — zarządzacie agentem piszącym kod.**

### Zasady złotej trójki hackathonu

1. **Scope first** — Zdecydujcie co robicie ZANIM odpaliecie IDE. Zakreślenie MVP jasno to różnica między dostarczeniem a chaosem.
2. **Każdy na swoim torze** — Każdy czlonek zespołu ma własną instancję Claude Code, własny branch, jasno zdefiniowane granice.
3. **Integracja co 90 minut** — Nie po 6 godzinach. Wczesna integracja to wczesne wykrycie konfliktu architektury.

---

## 2. Skład zespołu i podział ról

### Mapowanie ról na hackathon

| Osoba | Rola oficjalna | Rola hackathonowa | Odpowiada za |
|---|---|---|---|
| BE Dev #1 | Backend | **Tech Lead / Architekt** | Decyzje architektoniczne, API contract, integracja modułów |
| BE Dev #2 | Backend | **Feature Developer** | Implementacja endpointów, logika biznesowa |
| DB Dev | Database | **Data Engineer** | Schema, migracje, SQLite optymalizacja, seed data |
| QA Tester | Tester Automatyczny | **Quality Gate** | Testy integracyjne, E2E, scenariusze |

> ⚠️ **Frontend?** Nie ma dedykowanego FE deva — Claude Code napisze React. Tech Lead nadzoruje, BE Dev #2 integruje z API.

### Zasady rozgraniczenia pracy (DRI — Directly Responsible Individual)

```
BE Dev #1 (Tech Lead):
  - Definicja API contract (OpenAPI / endpoint lista)
  - Konfiguracja Docker Compose
  - Merge PR-ów do main
  - Decyzja "robimy / nie robimy" dla opcjonalnych featury

BE Dev #2 (Feature Dev):
  - Implementacja service/controller warstwy
  - Integracja frontendu z backendem
  - Hotfixy podczas demo

DB Dev:
  - schema.sql + migracje (Liquibase lub plain SQL)
  - Seed data dla demo
  - Zapytania i indeksy
  - Odpowiada za żeby baza była gotowa ZANIM BE zacznie pisać repositories

QA Tester:
  - Testy integracyjne Spring
  - Scenariusze happy path (priorytet)
  - E2E przez Playwright lub RestAssured
  - Walidacja że docker-compose up daje działający system
```

---

## 3. Przygotowanie PRZED hackathon

### D-2: Środowisko

**Każdy developer instaluje lokalnie:**

```bash
# 1. Node.js (wymagane przez Claude Code)
# Minimum v18, najlepiej v22 LTS
node --version

# 2. Claude Code CLI
npm install -g @anthropic-ai/claude-code
claude --version  # sprawdź czy działa

# 3. Zaloguj się do Claude Max
claude login
# Otwiera przeglądarkę — zaloguj konto z subskrypcją Max

# 4. Docker + Docker Compose
docker --version
docker compose version

# 5. Java 21+ JDK
java -version

# 6. Verify całość
claude --version && docker --version && java -version
```

**Przygotujcie wspólne repo:**

```bash
git init hackathon-app
cd hackathon-app
git checkout -b main

# Stwórzcie strukturę katalogów od razu
mkdir -p backend/src frontend db docker docs
```

### D-2: Decyzja domeny i MVP

Zanim cokolwiek instalujecie, odpowiedzcie na te pytania jako zespół (max 30 minut):

```
1. Co KONKRETNIE robi nasza aplikacja? (1-2 zdania)
2. Kto jest użytkownikiem końcowym?
3. Jakie są 3 najważniejsze przepływy użytkownika (user flows)?
4. Co MUSI działać na demo? (MVC minimum)
5. Co chcielibyśmy ale możemy wyciąć? (nice-to-have)
6. Jak będzie wyglądać 5-minutowe demo?
```

Wynik: **Product Brief** — jeden dokument markdown opisujący powyższe. Ten dokument wejdzie do `CLAUDE.md`.

### D-1: Instalacja Everything Claude Code

Szczegóły w sekcji 4 poniżej. Każdy developer instaluje ECC na swoim środowisku tego samego dnia — żeby nie tracić czasu hackathonu.

---

## 4. Instalacja i konfiguracja Claude Code + ECC

### Krok 1: Zainstaluj Claude Code (jeśli nie zrobione)

```bash
npm install -g @anthropic-ai/claude-code
claude login
```

### Krok 2: Zainstaluj Everything Claude Code

```bash
# Sklonuj repo
git clone https://github.com/affaan-m/everything-claude-code.git
cd everything-claude-code

# Zainstaluj zależności
npm install

# Uruchom install dla Javy i wspólnych reguł
./install.sh java          # na macOS/Linux
# lub:
.\install.ps1 java         # na Windows PowerShell
```

> **Co instaluje `install.sh java`?**
> Kopiuje do `~/.claude/`:
> - `rules/common/` — zasady stylu, git workflow, TDD, security
> - `rules/java/` (jeśli dostępne) — standardy Java/Spring
> - `agents/` — 28 wyspecjalizowanych subagentów
> - `commands/` — komendy slash do szybkiego uruchamiania
> - `skills/` — opisy workflow (springboot-patterns, springboot-tdd, docker-patterns itd.)

### Krok 3: Zainstaluj plugin bezpośrednio w Claude Code

```
# W sesji Claude Code (wpisz w terminalu):
claude

# Wewnątrz sesji:
/plugin marketplace add affaan-m/everything-claude-code
/plugin install everything-claude-code@everything-claude-code
```

### Krok 4: Skonfiguruj `~/.claude/settings.json`

Otwórz lub stwórz plik `~/.claude/settings.json`:

```json
{
  "model": "sonnet",
  "env": {
    "MAX_THINKING_TOKENS": "10000",
    "CLAUDE_AUTOCOMPACT_PCT_OVERRIDE": "50",
    "CLAUDE_CODE_SUBAGENT_MODEL": "haiku"
  }
}
```

> **Dlaczego `sonnet` a nie `opus`?**
> Sonnet obsługuje 80%+ zadań kodowania i jest ~60% tańszy tokenowo. Opus używacie tylko dla decyzji architektonicznych lub trudnych bugów. Na hackathonie limit tokenów może być wąskim gardłem — to ustawienie daje wam 2x więcej zasobów.

### Krok 5: Zainstaluj reguły Java/Spring manualnie

```bash
# Z katalogu everything-claude-code
# Reguły wspólne (OBOWIĄZKOWE)
cp -r rules/common/* ~/.claude/rules/

# Jeśli istnieją reguły Java:
cp -r rules/java/* ~/.claude/rules/ 2>/dev/null || true

# Skills Spring Boot (KLUCZOWE dla waszego stacku)
mkdir -p ~/.claude/skills
cp -r skills/springboot-patterns ~/.claude/skills/
cp -r skills/springboot-tdd ~/.claude/skills/
cp -r skills/springboot-security ~/.claude/skills/
cp -r skills/springboot-verification ~/.claude/skills/
cp -r skills/docker-patterns ~/.claude/skills/
cp -r skills/api-design ~/.claude/skills/
cp -r skills/database-migrations ~/.claude/skills/
cp -r skills/deployment-patterns ~/.claude/skills/
cp -r skills/e2e-testing ~/.claude/skills/

# Agents Java
cp agents/java-reviewer.md ~/.claude/agents/
cp agents/java-build-resolver.md ~/.claude/agents/
cp agents/database-reviewer.md ~/.claude/agents/
cp agents/tdd-guide.md ~/.claude/agents/
cp agents/planner.md ~/.claude/agents/
cp agents/architect.md ~/.claude/agents/
```

### Weryfikacja instalacji

```bash
# Uruchom Claude Code w folderze projektu
cd hackathon-app
claude

# Sprawdź dostępne komendy
/plugin list everything-claude-code@everything-claude-code

# Powinieneś zobaczyć listę 60+ komend i agentów
```

---

## 5. Struktura projektu i CLAUDE.md

### CLAUDE.md — najważniejszy plik projektu

`CLAUDE.md` to plik który Claude Code czyta **na początku każdej sesji**. To wasz "briefing dla agenta". Umieśćcie go w katalogu głównym projektu.

**Szablon CLAUDE.md dla waszego hackathonu:**

```markdown
# HackApp — Instrukcja dla Claude Code

## Kontekst biznesowy
[Wklejcie tutaj Product Brief: co robi aplikacja, kto używa, główne przepływy]

## Stack technologiczny
- Backend: Java 21 + Spring Boot 3.x
- Baza danych: SQLite (via Spring Data JPA + sqlite-dialect)
- Frontend: React 18 + TypeScript + Vite
- Konteneryzacja: Docker + Docker Compose
- Testy: JUnit 5, MockMvc, Playwright (E2E)
- Build: Maven
```
## Struktura katalogów

```
├── backend/          # Spring Boot app
│   ├── src/main/java/com/hackapp/
│   │   ├── controller/
│   │   ├── service/
│   │   ├── repository/
│   │   ├── model/
│   │   └── config/
│   └── src/test/
├── frontend/         # React app
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── api/      # API client (axios)
│   │   └── hooks/
├── db/
│   ├── schema.sql
│   └── seed.sql
└── docker-compose.yml
```

## Zasady kodowania
- TDD: pisz testy PRZED implementacją
- Pokrycie testami minimum 70% dla nowych serwisów
- REST endpoints: prefix /api/v1/
- Wszystkie odpowiedzi JSON z polem `status` i `data`/`error`
- SQLite: NIE używaj sekwencji, używaj autoincrement INTEGER PRIMARY KEY
- Brak hardcodowanych credentiali — używaj application.properties
- Każda klasa max 200 linii, metoda max 30 linii

## Nazewnictwo
- Branch: feature/nazwa-funkcji, fix/opis-bugu
- Commit: feat: opis, fix: opis, test: opis
- Klasy Java: PascalCase, testy: NazwaKlasyTest
- React komponenty: PascalCase, hooki: use+NazwaCamelCase

## API Contract (uzupełniać na bieżąco)
[Tu dodawajcie endpointy w trakcie hackathonu]
GET  /api/v1/...
POST /api/v1/...

## MVP — co MUSI działać
1. [Flow #1]
2. [Flow #2]
3. [Flow #3]

## Nice-to-have (tylko jeśli zostanie czas)
- Integracja przez REST API z zewnętrznym systemem
- Kafka messaging


> **Pro tip:** Aktualizujcie CLAUDE.md w trakcie hackathonu gdy dodajecie endpointy i decyzje techniczne. To "żywy dokument" projektu.

## 6. Planowanie pracy podczas hackathonu
### Model planowania: Epiki → Taski → Prompty
Nie używajcie Jiry. Zamiast tego: **prosty plik `TASKS.md`** w repo.


# TASKS.md

## EPIK 1: Fundament (h1-h3)
- [ ] DB: Schema i seed data @db-dev
- [ ] BE: Spring Boot init + Docker @be-lead
- [ ] BE: Auth (jeśli potrzebny) @be-lead
- [ ] FE: Szkielet React + routing @be-dev2

## EPIK 2: Główna funkcjonalność (h3-h6)
- [ ] BE: Endpoint A - GET /api/v1/X @be-dev2
- [ ] BE: Endpoint B - POST /api/v1/Y @be-dev2
- [ ] FE: Widok Z @be-dev2
- [ ] QA: Testy integracyjne @qa

## EPIK 3: Polish + Demo (h6-h8)
- [ ] FE: UI polish @be-dev2
- [ ] QA: Scenariusze demo @qa
- [ ] BE: Docker Compose finalizacja @be-lead
- [ ] Dokumentacja demo @cały zespół

### Zasada równoległości

```
T=0h          T=1h          T=2h          T=3h
DB Dev:   [Schema] ─────── [Seed] ─── [Optymalizacja]
BE Lead:  [Init]   ──── [Arch] ────── [Integracja] ───
BE Dev2:  [Init]   ──── [Service] ─── [Controller] ───
QA:       [Setup] ────── [Testy] ──── [E2E]  ─────────
                                     ▲ sync #1   ▲ sync #2
```

**Kluczowe zależności które musicie rozwiązać na początku:**
1. DB Dev publikuje `schema.sql` w pierwszej godzinie → odblokowanie BE
2. BE Lead definiuje API contract (lista endpointów) → odblokowanie FE i QA
3. Docker Compose skeleton gotowy H1 → każdy może uruchomić stack lokalnie

---

## 7. Efektywne użycie Claude Code w trakcie

### Uruchamianie Claude Code

```bash
# W katalogu projektu (ZAWSZE — żeby Claude widział CLAUDE.md)
cd hackathon-app
claude
```

Claude Code automatycznie czyta `CLAUDE.md` przy starcie sesji.

### Anatomia dobrego promptu do Claude Code

Struktura: **Cel → Kontekst → Ograniczenia → Format wyjścia**

```
# ZŁY prompt:
"Napisz endpoint do pobierania użytkowników"

# DOBRY prompt:
"Zaimplementuj endpoint GET /api/v1/users/{id} w UserController.
Endpoint ma:
- Pobierać użytkownika po ID z UserRepository
- Rzucać UserNotFoundException jeśli nie znaleziono (404)
- Zwracać UserResponseDto (bez pola password)
Napisz najpierw test w UserControllerTest korzystając z MockMvc,
potem implementację. Użyj wzorców z skills/springboot-patterns."
```

### Komendy ECC które będziecie używać najczęściej

```bash
# W sesji claude:

/plan "Zaimplementuj moduł zarządzania zamówieniami"
# → Tworzy szczegółowy plan implementacji z listą plików do stworzenia

/tdd
# → Przełącza Claude w tryb TDD: najpierw test, potem kod

/code-review
# → Code review ostatnich zmian pod kątem jakości i security

/build-fix
# → Naprawia błędy kompilacji Maven/Java automatycznie

/e2e
# → Generuje testy E2E dla krytycznych przepływów

/multi-backend "Zaimplementuj obsługę zamówień: model, repo, service, controller"
# → Orkiestruje wieloagentowy workflow dla złożonego zadania

/verify
# → Uruchamia pełny verification loop: build + test + lint
```

### Technika "Plan then Execute"

Przed każdym większym zadaniem:

```
# Krok 1: Planowanie (5 min)
/plan "Dodaj system powiadomień push: model Notification, 
service NotificationService, endpoint POST /api/v1/notifications"

# → Claude generuje plan z listą plików, zależności, krokami

# Krok 2: Review planu
# Przejrzyjcie plan, poprawcie jeśli coś jest nie tak

# Krok 3: Implementacja
"OK, wykonaj plan powyżej. Zacznij od modelu i repozytorium."
```

### Delegowanie do subagentów (zabójcza funkcja)

```bash
# Kiedy macie złożony task, delegujcie do wyspecjalizowanych agentów:

"@java-reviewer przejrzyj OrderService.java pod kątem 
wzorców transakcji i potencjalnych problemów z SQLite"

"@tdd-guide poprowadź mnie przez implementację PaymentService metodą TDD"

"@database-reviewer sprawdź czy nasze zapytania w OrderRepository 
są zoptymalizowane dla SQLite"

"@architect zaproponuj strukturę dla integracji Kafka — 
consumer/producer pattern dla naszego stacku Spring Boot"
```

---

## 8. Workflow i komendy ECC dla waszego stacku

### BE Dev #1 (Tech Lead) — typowe komendy

```bash
# Inicjalizacja projektu Spring Boot
"Stwórz Spring Boot 3.x projekt z Maven, Java 21. 
Dependencies: spring-web, spring-data-jpa, sqlite-jdbc, 
spring-boot-test, lombok. Stwórz docker-compose.yml z serwisem backend.
Skonfiguruj application.properties dla SQLite."

# Docker Compose
"Napisz docker-compose.yml z serwisami: backend (port 8080), 
frontend (port 3000). Backend ma zmienną DATABASE_PATH dla ścieżki SQLite.
Dodaj health check dla backendu."

# Review architektoniczne
/code-review
"@architect oceń naszą strukturę pakietów i zaproponuj 
usprawnienia dla skalowalności"
```

### BE Dev #2 (Feature Dev) — typowe komendy

```bash
# Implementacja feature end-to-end
/tdd
"Zaimplementuj CRUD dla encji Order:
- Model: Order (id, userId, status, totalAmount, createdAt)
- Repository: OrderRepository extends JpaRepository
- Service: OrderService (findById, create, updateStatus)  
- Controller: OrderController (/api/v1/orders)
- DTO: OrderRequestDto, OrderResponseDto
Zacznij od testów w TDD."

# Integracja frontendu
"Stwórz React komponent OrderList który:
- Pobiera dane z GET /api/v1/orders przez axios
- Wyświetla tabelę z zamówieniami
- Ma loading state i error handling
- Używa TypeScript
Stwórz też hook useOrders.ts"

# Naprawa błędów build
/build-fix
# automatycznie analizuje błędy Maven i naprawia
```

### DB Dev — typowe komendy

```bash
# Schema
"Zaprojektuj schema SQLite dla aplikacji [opis domeny].
Encje: [lista]. Uwzględnij:
- INTEGER PRIMARY KEY AUTOINCREMENT (nie SEQUENCE)
- indeksy na polach filtrowania
- relacje przez klucze obce
- pola created_at, updated_at
Wygeneruj schema.sql i seed.sql z danymi demo."

# Migracje
"@database-reviewer sprawdź schema.sql pod kątem:
- wydajności zapytań w SQLite
- poprawności typów danych
- potencjalnych problemów z blokadami przy concurrent access"

# JPA mapping
"Napisz klasy @Entity dla tabel z naszego schema.sql.
Użyj Lombok @Data, @Builder. Skonfiguruj relacje @OneToMany, @ManyToOne."
```

### QA Tester — typowe komendy

```bash
# Testy integracyjne
/e2e
"Napisz testy integracyjne Spring Boot dla OrderController:
- Test happy path: create order, get order, update status
- Test error paths: 404, 400 validation error
- Użyj @SpringBootTest, MockMvc, @Sql dla setup bazy
- Testy mają być niezależne — czyszczą dane po sobie"

# Testy E2E Playwright
"Napisz testy Playwright (TypeScript) dla głównych przepływów:
1. Użytkownik tworzy zamówienie
2. Użytkownik przegląda historię zamówień
Aplikacja działa na localhost:3000"

# Smoke test docker
"Napisz skrypt bash który:
1. Uruchamia docker-compose up -d
2. Czeka na health check backendu (max 30s)  
3. Wykonuje serię curl żądań do kluczowych endpointów
4. Zwraca 0 jeśli wszystko OK, 1 jeśli coś nie działa"
```

---

## 9. Tester automatyczny — maksymalne wykorzystanie

QA w hackathonie to nie "tester na końcu" — to **ciągły safety net** który daje zespołowi pewność że refaktory nie psują działającego kodu.

### Priorytetyzacja testów hackathonowych

```
PRIORYTET 1 — Happy Path (zawsze):
  Każdy główny przepływ użytkownika musi mieć 1 test integracyjny.
  Cel: jeśli to działa, demo przejdzie.

PRIORYTET 2 — Build gate (zawsze):
  docker-compose up + curl /health zwraca 200.
  Cel: deploy na demo nie wysypie się.

PRIORYTET 3 — Edge cases (jeśli czas):
  Walidacja inputów, błędy 4xx.

PRIORYTET 4 — E2E Playwright (nice-to-have):
  Tylko dla 1-2 kluczowych user stories.
```

### Szablon testu integracyjnego Spring

```java
// Szablon do przekazania Claude Code
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc
@Sql(scripts = {"/db/schema.sql", "/db/test-seed.sql"}, 
     executionPhase = BEFORE_TEST_METHOD)
@Sql(scripts = "/db/cleanup.sql", 
     executionPhase = AFTER_TEST_METHOD)
class OrderControllerIT {
    
    @Autowired MockMvc mockMvc;
    @Autowired ObjectMapper objectMapper;
    
    @Test
    void createOrder_happyPath_returns201() throws Exception {
        // given
        var request = new OrderRequestDto(...);
        
        // when + then
        mockMvc.perform(post("/api/v1/orders")
                .contentType(APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.data.id").exists())
            .andExpect(jsonPath("$.status").value("success"));
    }
}
```

Dajcie ten szablon Claude Code przy prośbie o testy — dostaniecie spójny styl.

### Automatyczny verification loop

```bash
# Dodajcie do projektu skrypt check.sh:
#!/bin/bash
set -e
cd backend
mvn clean test -q
echo "✅ Tests passed"
mvn spring-boot:build-image -q 2>/dev/null || true
echo "✅ Build OK"
cd ../
docker compose up -d --build
sleep 10
curl -sf http://localhost:8080/actuator/health | grep -q '"status":"UP"'
echo "✅ Stack healthy"
```

```bash
# W Claude Code uruchamiajcie po każdym większym feature:
/verify
# lub ręcznie:
"Uruchom ./check.sh i napraw wszystkie błędy które znajdziesz"
```

---

## 10. Zarządzanie kontekstem i tokenami

Claude Code działa na oknie kontekstowym — przy długich sesjach zaczyna "zapominać" wcześniejszy kod.

### Zasady zarządzania sesją

```bash
# Między niezwiązanymi zadaniami — zawsze:
/clear
# Zeruje kontekst, nie kosztuje nic, natychmiastowy reset

# Po zakończeniu dużego feature / przed nowym:
/compact  
# Kompresuje historię sesji, zachowując kluczowe informacje

# Sprawdź zużycie tokenów:
/cost
```

### Kiedy compact, kiedy clear

| Sytuacja | Komenda |
|---|---|
| Zmieniasz temat (z backendu na frontend) | `/clear` |
| Skończyłeś feature, zaczynasz nowy w tym samym obszarze | `/compact` |
| Sesja trwa >2h bez reset | `/compact` |
| Bug goni bug i coraz wolniej | `/clear` — zacznij świeżą sesję |
| Przed ważną decyzją architektoniczną | `/model opus` + nowa sesja |

### Strategia context dump

Przed `/clear`, "dumpnijcie" kontekst:

```bash
"Przed zakończeniem sesji: podsumuj w 200 słowach:
1. Co zaimplementowaliśmy
2. Kluczowe decyzje techniczne  
3. Znane problemy / TODO
4. Stan testów"
```

Wklejcie output do `TASKS.md` lub `CLAUDE.md` — następna sesja zaczyna od tej wiedzy.

---

## 11. Opcjonalne rozszerzenia — API / Kafka

Uruchamiajcie te moduły TYLKO gdy MVP jest gotowe i działa. Szacowane czasy:

### REST API Integration (~2h)

```bash
"Zaimplementuj integrację z zewnętrznym API [nazwa/opis].
Użyj Spring WebClient (nie RestTemplate).
Dodaj:
- Client class z @Service
- Retry logic (@Retryable) dla 5xx
- Timeout konfiguracja (connectTimeout: 5s, readTimeout: 10s)
- Fallback wartość jeśli API niedostępne
- Test z WireMock"
```

### Kafka Integration (~3h)

```bash
# Krok 1: Docker Compose rozszerzenie
"Dodaj do docker-compose.yml serwisy Kafka i Zookeeper (image: confluentinc/cp-kafka)"

# Krok 2: Producer
"Dodaj Spring Kafka producer w OrderService.
Po każdym POST /orders emituj event OrderCreatedEvent na topic 'orders.created'.
Event ma: orderId, userId, amount, timestamp."

# Krok 3: Consumer
"Zaimplementuj @KafkaListener dla topic 'orders.created' 
w NotificationService który loguje event i (opcjonalnie) 
tworzy rekord w tabeli notifications."

# Agent Kafka
"@architect zaprojektuj topic naming convention i 
partitioning strategy dla naszej domeny"
```

> **Decyzja go/no-go dla Kafka:** Jeśli do H6 hackathonu MVP nie ma 2+ działających endpointów z testami — wycinajcie Kafkę i koncentrujcie się na polerowaniu tego co jest.

---

## 12. Timeline hackathonu — godzina po godzinie

### Zakładając hackathon 8-godzinny

```
H0:00 — KICKOFF (30 min)
├── Wszyscy razem: finalizacja Product Brief
├── Ustalenie API contract (lista endpointów na kartce/tablicy)
├── Podział tasków w TASKS.md  
└── Każdy odpala claude w swoim katalogu roboczym

H0:30 — FUNDAMENTY (do H2:00)
├── DB Dev:    schema.sql + seed.sql → commit na main
├── BE Lead:   Spring Boot init + docker-compose skeleton  
├── BE Dev2:   React app init + Vite + routing skeleton
└── QA:        Test harness setup, szkielet testów integracyjnych

H2:00 — SYNC #1 (15 min, OBOWIĄZKOWY)
├── DB Dev: pokazuje schema — czy wszyscy rozumieją?
├── BE Lead: pokazuje działający /health endpoint w Docker  
├── Każdy: "co zrobię w następnych 2h"
└── Blocker check: czy ktoś jest zablokowany?

H2:15 — CORE FEATURES (do H5:00)
├── DB Dev:    dane produkcyjne seed, optymalizacja zapytań
├── BE Lead:   autoryzacja (jeśli), konfiguracje, integracja modułów
├── BE Dev2:   główne endpointy + React widoki
└── QA:        testy dla endpointów które już istnieją

H5:00 — SYNC #2 (20 min, KRYTYCZNY)
├── Demo wewnętrzny: każdy pokazuje co działa
├── Ocena: czy MVP jest zamknięte? TAK/NIE
├── Jeśli TAK: przejście do polishingu
├── Jeśli NIE: triage — co wycinamy, co naprawiamy
└── DECYZJA go/no-go dla opcjonalnych features (Kafka/API)

H5:20 — INTEGRATION & POLISH (do H7:00)
├── BE Lead:   integracja wszystkich modułów, docker-compose final
├── BE Dev2:   UI polish, error handling na frontendzie
├── QA:        scenariusz demo smoke test, E2E dla happy path
└── Opcjonalnie: 1 osoba na Kafka/API jeśli decyzja GO

H7:00 — DEMO PREPARATION (do H7:45)
├── Cały zespół: dry run demo
├── QA:         uruchamia ./check.sh — czy stack jest zdrowy?
├── Dokumentacja: README ze screen/gif jeśli czas
└── FREEZE kodu: zero nowych commitów po H7:30

H7:45 — BUFFER
└── Tylko hotfixy krytycznych bugów

H8:00 — PREZENTACJA
```

---

## 13. Najczęstsze pułapki i jak ich unikać

### Pułapka #1: "Napiszemy najpierw, planujemy potem"

**Problem:** Każdy pisze w innym stylu, konflikty przy integracji.
**Rozwiązanie:** 30 minut planowania na początku > 3 godziny naprawiania konfliktów.

### Pułapka #2: "Claude napisał kod, więc musi być dobry"

**Problem:** Claude Code może wygenerować kod który kompiluje się ale ma błędy logiczne lub nie pasuje do reszty projektu.
**Rozwiązanie:** Zawsze uruchamiajcie `/verify` lub skrypt testowy po każdym większym bloku kodu. Nie commitujcie nie sprawdzonego kodu.

### Pułapka #3: SQLite + Spring Data JPA specyfika

**Problem:** SQLite nie obsługuje wszystkich funkcji które JPA zakłada (np. SEQUENCE, niektóre typy).
**Rozwiązanie:** Wklejcie do promptu:
```
"Pamiętaj: używamy SQLite, nie PostgreSQL.
Użyj INTEGER PRIMARY KEY AUTOINCREMENT zamiast SEQUENCE.
Dialekt: org.hibernate.community.dialect.SQLiteDialect.
Unikaj typów danych nieobsługiwanych przez SQLite (np. JSONB, ARRAY)."
```

### Pułapka #4: "Mamy 8 godzin, zdążymy wszystko"

**Problem:** Scope creep. Każda "mała" feature to minimum 45 minut.
**Rozwiązanie:** W SYNC #2 (H5:00) bądźcie bezlitośni w wycinaniu. Demo 3 działające features > demo 8 niedokończonych.

### Pułapka #5: Zbyt długie sesje Claude Code bez resetu

**Problem:** Po 2h bez `/clear` lub `/compact` Claude "zapomina" wcześniejszy kontekst i zaczyna generować niespójny kod.
**Rozwiązanie:** Reset sesji między featurami. Kluczowe informacje trzymajcie w `CLAUDE.md`, nie w historii chatu.

### Pułapka #6: Merge hell na koniec

**Problem:** Wszyscy commitują do main przez 8 godzin bez mergeowania.
**Rozwiązanie:** 
- Każdy pracuje na własnym branchu (feature/xxx)
- BE Lead merguje do main co 2 godziny
- Po każdym merge: `git pull --rebase origin main`

### Pułapka #7: Docker na demie nie działa

**Problem:** "U mnie działa" → na demie odpala się na innym komputerze i nie wstaje.
**Rozwiązanie:** QA odpowiada za smoke test na fresh `docker-compose up` minimum 1h przed demo. Skrypt `./check.sh` uruchamiany po każdym merge do main.

---

## 14. Checklista startowa

### D-1 (dzień przed)

- [ ] Każdy zainstalował Node.js, Claude Code CLI i zalogował się
- [ ] Każdy zainstalował ECC (`./install.sh java`)
- [ ] `~/.claude/settings.json` skonfigurowany z `model: sonnet`
- [ ] Shared Git repo stworzony z `main` branchem
- [ ] Product Brief napisany (co robimy, MVP, 3 user flows)
- [ ] Podstawowa struktura katalogów (`backend/`, `frontend/`, `db/`, `docker/`)

### H0:00 (start hackathonu)

- [ ] `CLAUDE.md` stworzony z Product Brief i zasadami kodowania
- [ ] `TASKS.md` stworzony z podziałem pracy
- [ ] API contract (lista endpointów) uzgodniona i zapisana
- [ ] Każdy jest na swoim branchu: `feature/db-schema`, `feature/be-core`, `feature/fe-skeleton`, `feature/test-harness`
- [ ] Każdy uruchomił `claude` w swoim katalogu roboczym

### H2:00 (SYNC #1)

- [ ] `schema.sql` jest w repo i każdy go zna
- [ ] `docker-compose.yml` startuje bez błędów
- [ ] `/actuator/health` zwraca 200
- [ ] Frontend startuje na `:3000` (nawet jako skeleton)
- [ ] Test harness gotowy — można uruchomić `mvn test`

### H5:00 (SYNC #2)

- [ ] Każdy główny endpoint z MVP jest zaimplementowany (choćby częściowo)
- [ ] Przynajmniej 1 test integracyjny przechodzi
- [ ] `docker-compose up` z frontem i backiem działa razem
- [ ] Decyzja: Kafka/API — go/no-go
- [ ] TASKS.md zaktualizowany

### H7:00 (przed demo)

- [ ] Dry run demo — cały flow z perspektywy użytkownika
- [ ] `./check.sh` zwraca success na fresh `docker-compose up`
- [ ] README z instrukcją uruchomienia
- [ ] Code freeze — tylko hotfixy

---

## Szybka ściąga komend ECC

```bash
# Planowanie
/plan "opis zadania"          # Plan implementacji
/multi-backend "opis"         # Złożony backend task z orkiestracją

# Implementacja  
/tdd                          # Tryb TDD
/build-fix                    # Napraw błędy build

# Jakość
/code-review                  # Review kodu
/verify                       # Pełny verification loop
/e2e                          # Generuj testy E2E

# Zarządzanie sesją
/cost                         # Ile tokenów zużyłem
/compact                      # Skompresuj historię
/clear                        # Wyczyść kontekst
/model opus                   # Przełącz na Opus (decyzje arch)
/model sonnet                 # Wróć na Sonnet (domyślny)

# Agenci
@java-reviewer                # Review Java/Spring
@java-build-resolver          # Napraw build Java
@database-reviewer            # Review DB/JPA
@tdd-guide                    # Prowadzenie TDD
@architect                    # Decyzje architektoniczne
@planner                      # Planowanie feature
```

---

## Bonus: Sprawdzone prompty dla waszego stacku

```bash
# Inicjalizacja Spring Boot z SQLite
"Stwórz Spring Boot 3.x projekt. Stack: Java 21, Maven, 
spring-boot-starter-web, spring-boot-starter-data-jpa, 
xerial/sqlite-jdbc 3.x, Lombok, spring-boot-starter-test.
Konfiguracja: spring.datasource.url=jdbc:sqlite:./data/app.db,
spring.jpa.database-platform=org.hibernate.community.dialect.SQLiteDialect,
spring.jpa.hibernate.ddl-auto=none.
Wygeneruj Application.java, application.properties i pusty HealthController."

# React + Vite + TypeScript init
"Stwórz React 18 + TypeScript + Vite aplikację.
Dodaj: axios dla HTTP, react-router-dom v6 dla routingu.
Stwórz: src/api/client.ts (axios instancja z baseURL http://localhost:8080),
src/pages/Home.tsx (placeholder), src/App.tsx z routingiem."

# Docker Compose kompletny
"Napisz docker-compose.yml dla:
- backend: build z ./backend/Dockerfile, port 8080:8080,
  volume ./data:/app/data dla SQLite, health check GET /actuator/health
- frontend: build z ./frontend/Dockerfile, port 3000:3000,
  depends_on backend
Dodaj Dockerfile dla backendu (multi-stage: Maven build + JRE runtime)
i dla frontendu (Node build + nginx serve)."
```

---

*Powodzenia na hackathonie! Pamiętajcie: wygrywa ten kto dostarcza działające demo, nie ten kto napisał najpiękniejszy kod.* 🚀

