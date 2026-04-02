# 🧠 ECC — Kompletna Ściąga Użytkownika
### Everything Claude Code · Claude Max · Claude Code CLI

> Dokument dla zielonego zespołu. Zero założeń o wcześniejszej wiedzy.
> Każda komenda ma: co robi, kiedy użyć, przykład użycia, czego oczekiwać.

---

## Spis treści

1. [Jak działa Claude Code — model mentalny](#1-jak-działa-claude-code--model-mentalny)
2. [Uruchamianie i podstawowa nawigacja](#2-uruchamianie-i-podstawowa-nawigacja)
3. [Komendy wbudowane Claude Code](#3-komendy-wbudowane-claude-code)
4. [ECC: Slash Commands — planowanie i feature development](#4-ecc-slash-commands--planowanie-i-feature-development)
5. [ECC: Slash Commands — jakość, testy, bezpieczeństwo](#5-ecc-slash-commands--jakość-testy-bezpieczeństwo)
6. [ECC: Slash Commands — naprawa błędów](#6-ecc-slash-commands--naprawa-błędów)
7. [ECC: Slash Commands — multi-agent i orkiestracja](#7-ecc-slash-commands--multi-agent-i-orkiestracja)
8. [ECC: Slash Commands — zarządzanie sesją i pamięcią](#8-ecc-slash-commands--zarządzanie-sesją-i-pamięcią)
9. [ECC: Slash Commands — model routing i harness](#9-ecc-slash-commands--model-routing-i-harness)
10. [ECC: Agenci — pełna lista i kiedy używać](#10-ecc-agenci--pełna-lista-i-kiedy-używać)
11. [ECC: Skills — jak działają i jak aktywować](#11-ecc-skills--jak-działają-i-jak-aktywować)
12. [ECC: Hooks — co się dzieje automatycznie](#12-ecc-hooks--co-się-dzieje-automatycznie)
13. [ECC: Rules — zasady które Claude zawsze stosuje](#13-ecc-rules--zasady-które-claude-zawsze-stosuje)
14. [Zarządzanie modelami i tokenami](#14-zarządzanie-modelami-i-tokenami)
15. [Wzorce promptowania — jak pisać skuteczne polecenia](#15-wzorce-promptowania--jak-pisać-skuteczne-polecenia)
16. [Typowe scenariusze hackathonowe — gotowe sekwencje](#16-typowe-scenariusze-hackathonowe--gotowe-sekwencje)
17. [Troubleshooting — co gdy coś nie działa](#17-troubleshooting--co-gdy-coś-nie-działa)

---

## 1. Jak działa Claude Code — model mentalny

### Czym jest Claude Code vs zwykły Claude (claude.ai)

```
Zwykły Claude (claude.ai / ta strona):
  - Chatbot w przeglądarce
  - Widzi tylko to co mu wklejasz
  - Nie ma dostępu do twoich plików
  - Nie może uruchamiać kodu

Claude Code (terminal):
  - Agent działający w twoim terminalu
  - Ma bezpośredni dostęp do systemu plików
  - Może czytać, pisać, tworzyć pliki
  - Może uruchamiać komendy bash/mvn/npm
  - Może edytować wiele plików jednocześnie
  - Pamięta kontekst całej sesji (do momentu /clear)
```

### ECC jako "upgrade pack" do Claude Code

```
Claude Code (bez ECC):
  Inteligentny asystent który koduje na życzenie
  
Claude Code + ECC:
  Inteligentny asystent który:
  - Ma gotowe procedury (skills) dla Javy, Spring Boot, Docker, TDD
  - Ma specjalistyczne subagenty (28 sztuk) do których może delegować
  - Reaguje automatycznie na zdarzenia (hooks)
  - Przestrzega stałych reguł (rules) niezależnie od promptu
  - Potrafi orkiestrować wiele równoległych zadań (multi-agent)
```

### Hierarchia: Rules → Skills → Agents → Commands

```
RULES (zawsze aktywne, niewidoczne)
  ↓ nakładają zasady na każdą odpowiedź
SKILLS (playbooki uruchamiane gdy pasują do zadania)
  ↓ dają Claudowi konkretne instrukcje workflow
AGENTS (wyspecjalizowane "osobowości" do delegowania)
  ↓ samodzielnie wykonują odizolowane zadania
COMMANDS (ty inicjujesz komendą /xxx)
  ↓ uruchamiają konkretny workflow
TY (piszesz prompty lub komendy)
```

### Co Claude Code "widzi" w twoim projekcie

Gdy uruchamiasz `claude` w folderze projektu, agent automatycznie czyta:
- `CLAUDE.md` — główny briefing projektu (twój główny plik konfiguracyjny)
- `AGENTS.md` — definicje agentów (z ECC)
- `.claude/settings.json` — lokalne ustawienia projektu
- `~/.claude/settings.json` — globalne ustawienia użytkownika
- `~/.claude/rules/` — zawsze-aktywne reguły (zainstalowane przez ECC)
- `~/.claude/agents/` — definicje subagentów
- `~/.claude/skills/` — dostępne playbooki

---

## 2. Uruchamianie i podstawowa nawigacja

### Uruchamianie Claude Code

```bash
# ZAWSZE uruchamiaj z katalogu projektu!
cd /ścieżka/do/projektu
claude

# Jeśli uruchomisz z innego katalogu — Claude nie znajdzie CLAUDE.md
# i będzie pracował "w ciemno" bez kontekstu projektu
```

### Pierwsze wrażenie — co zobaczysz

```
╭────────────────────────────────────╮
│ Claude Code v2.x.x                 │
│ Model: claude-sonnet-4             │
│ Project: hackathon-app             │
╰────────────────────────────────────╯
> 
```

Znak `>` to prompt — tutaj piszesz komendy i pytania.

### Podstawowe gesty

```
[Enter]          — wyślij wiadomość
[Ctrl+C]         — przerwij bieżące działanie Claude (nie wyjście z programu)
[Ctrl+D]         — wyjście z Claude Code
[↑] [↓]         — historia komend (jak w terminalu)
/                — lista dostępnych komend (wpisz sam slash)
Tab              — autocomplete komend po "/"
```

### Jak zatwierdzać akcje Claude

Claude pyta o zgodę przed wykonaniem potencjalnie destrukcyjnych operacji:

```
Claude wants to: Edit file src/main/java/OrderService.java
[y] Yes  [n] No  [a] Always allow  [d] Don't allow  [e] Edit

# y — zatwierdź tę jedną akcję
# a — zatwierdź wszystkie podobne akcje w tej sesji (bezpieczne przy zaufanych zadaniach)
# n — odmów
# e — edytuj co Claude planuje zrobić zanim wykona
```

> ⚠️ Na hackathonie: używajcie `a` (always allow) dla operacji na plikach projektu — oszczędza czas. Nigdy nie używajcie `a` dla `rm -rf` lub operacji na bazach danych produkcyjnych.

---

## 3. Komendy wbudowane Claude Code

To komendy które istnieją w Claude Code BEZ ECC. Są fundamentem — musicie je znać.

### `/help`

```
> /help
```

Wyświetla listę wszystkich dostępnych komend (wbudowanych + zainstalowanych przez ECC).
**Używajcie gdy nie pamiętacie komendy** — szybsze niż szukanie w dokumentacji.

---

### `/clear`

```
> /clear
```

**Co robi:** Całkowicie czyści historię konwersacji. Claude "zapomina" wszystko z bieżącej sesji. Nie usuwa żadnych plików projektu.

**Kiedy używać:**
- Przełączasz się na zupełnie inny temat (np. z backendu na frontend)
- Sesja trwa długo i Claude zaczyna dawać niespójne odpowiedzi
- Skończyłeś zadanie i zaczynasz nowe
- Po błędzie który wprowadził Claude w "zły tok myślenia"

**⚠️ Uwaga:** Po `/clear` Claude nie pamięta nic z poprzedniej rozmowy. Jeśli ważne decyzje były tylko w historii chatu — są stracone. Dlatego kluczowe decyzje zapisujcie w `CLAUDE.md`.

---

### `/compact`

```
> /compact
> /compact "Pracujemy nad OrderService — zachowaj kontekst o strukturze bazy"
```

**Co robi:** Kompresuje historię sesji zachowując najważniejsze informacje. Okno kontekstu jest "odchudzane" ale Claude nie zapomina kluczowego kontekstu. Możesz podać wskazówkę co jest ważne.

**Kiedy używać:**
- Po zakończeniu jednego feature, przed następnym (w tym samym obszarze kodu)
- Gdy sesja trwa >2h bez resetu
- Gdy widzisz że Claude zaczyna "zapominać" wcześniej ustalony styl
- Przed dużą implementacją — chcesz świeże okno ale pamiętasz architekturę

**Różnica między `/clear` a `/compact`:**
```
/clear   = wymaż tablicę, zacznij od zera
/compact = sfotografuj tablicę, wymaż, przyklej zdjęcie
```

---

### `/cost`

```
> /cost
```

**Co robi:** Pokazuje ile tokenów/USD zużyłeś w bieżącej sesji.

**Kiedy używać:** Regularnie podczas hackathonu żeby nie przekroczyć limitu Claude Max. Jeśli widzisz że sesja kosztuje dużo — czas na `/compact` lub `/clear`.

Przykładowy output:
```
Session cost: $0.47
Input tokens: 45,230
Output tokens: 12,890
Cache hits: 78%
```

---

### `/diff`

```
> /diff
```

**Co robi:** Otwiera interaktywny podgląd wszystkich zmian które Claude zrobił w tej sesji — jak `git diff` ale w ładnym widoku.

**Kiedy używać:** Przed commitem — sprawdź co Claude zmieniło. Dobry habit: zawsze `/diff` → review → `git commit`.

---

### `/model`

```
> /model sonnet
> /model opus
> /model haiku
```

**Co robi:** Przełącza model AI w bieżącej sesji.

**Kiedy używać:**
```
sonnet  → domyślny, 90% zadań (najlepszy balans jakości i kosztu)
opus    → skomplikowana architektura, trudny debug, decyzje strategiczne
haiku   → proste, powtarzalne zadania (najszybszy, najtańszy)
```

---

### `/skills`

```
> /skills
```

**Co robi:** Wyświetla listę wszystkich zainstalowanych skills (playbooki ECC). 

---

### `/batch` (wbudowany w Claude Code)

```
> /batch "Dodaj error handling do wszystkich endpointów w katalogu controller/"
```

**Co robi:** Rozkłada duże zadanie powtarzalne na 5-30 niezależnych jednostek i wykonuje je równolegle w izolowanych worktrees. Każdy agent pracuje na swojej kopii repo, implementuje swoją część i może otworzyć PR.

**Kiedy używać:** Dla zmian które aplikujesz do wielu plików według tego samego wzorca. Np. dodanie walidacji do 20 endpointów, dodanie logowania do 15 serwisów, refaktoryzacja importów.

**Kiedy NIE używać:** Dla złożonych zmian wymagających spójnego planu — np. przepisanie architektury. To nadaje się dla `/plan` + ręczna implementacja.

---

### `/loop` (wbudowany)

```
> /loop "co 30 minut uruchom mvn test i podsumuj wyniki" 30m
```

**Co robi:** Uruchamia prompt cyklicznie co podany interwał, dopóki sesja jest otwarta.

**Kiedy używać na hackathonie:** Zostawcie uruchomione w tle żeby co pół godziny weryfikowało czy testy przechodzą.

---

## 4. ECC: Slash Commands — planowanie i feature development

### `/plan`

```
> /plan "Dodaj moduł zarządzania produktami"
> /plan "Zaimplementuj autoryzację JWT"
> /plan "Stwórz endpoint do eksportu raportów PDF"
```

**Co robi:** Analizuje wymaganie, pyta o potwierdzenie, tworzy szczegółowy plan implementacji. Pokazuje listę plików do stworzenia/modyfikacji, zależności, kolejność kroków. **CZEKA na twój CONFIRM zanim dotknie kodu.**

**Dlaczego to ważne:** Bez `/plan` Claude zaczyna kodować natychmiast i może pojechać w złym kierunku. Z `/plan` widzisz co zamierza zrobić, możesz skorygować, a dopiero potem powiedzieć "ok, rób".

**Typowy flow:**
```
1. Wpiszesz: /plan "Dodaj produkt"
2. Claude wyświetla plan:
   - OrderController.java (modyfikacja)
   - ProductService.java (nowy)
   - ProductRepository.java (nowy)
   - ProductDto.java (nowy)
   - 3 testy integracyjne
3. Ty: "OK ale nie chcę DTO, używamy encji bezpośrednio"
4. Claude: aktualizuje plan
5. Ty: "Zatwierdź"
6. Claude: implementuje
```

---

### `/tdd`

```
> /tdd
> /tdd "Zaimplementuj OrderService.createOrder()"
```

**Co robi:** Przełącza Claude w tryb Test-Driven Development. Wymusza kolejność: RED (napisz test który nie przechodzi) → GREEN (napisz minimalny kod żeby test przeszedł) → REFACTOR (ulepsz kod).

**Dlaczego to ważne:** Bez `/tdd` Claude domyślnie pisze kod bez testów lub pisze testy po fakcie (co jest anty-patternem). `/tdd` wymusza właściwy workflow.

**Co zobaczysz:**
```
[RED] Writing failing test for OrderService.createOrder()...
  ✗ OrderServiceTest.createOrder_shouldReturnOrderWithId — FAIL (expected)
[GREEN] Implementing minimal code to pass test...
  ✓ OrderServiceTest.createOrder_shouldReturnOrderWithId — PASS
[REFACTOR] Improving implementation...
```

---

### `/checkpoint`

```
> /checkpoint
> /checkpoint "Przed implementacją płatności"
```

**Co robi:** Zapisuje aktualny stan weryfikacji projektu — wyniki testów, status build, coktóre pliki zmodyfikowano. Tworzy punkt powrotu jeśli następna zmiana coś zepsuje.

**Kiedy używać:** Przed każdą większą zmianą. Jak "save game" w grze.

---

## 5. ECC: Slash Commands — jakość, testy, bezpieczeństwo

### `/code-review`

```
> /code-review
> /code-review "OrderService.java"
> /code-review "Sprawdź szczególnie obsługę transakcji"
```

**Co robi:** Uruchamia agenta `code-reviewer` który analizuje ostatnie zmiany (lub wskazany plik) pod kątem: jakości kodu, potencjalnych bugów, wzorców projektowych, problemów z bezpieczeństwem, testowalności.

**Co dostaniesz:** Lista konkretnych problemów z numerami linii i propozycjami naprawy. Nie tylko "to jest złe" ale "zmień linię 47 z X na Y bo..."

**Przykładowe znaleziska:**
```
🔴 CRITICAL: OrderService.java:89 — SQL injection risk, use parameterized query
🟡 WARNING: OrderController.java:34 — Missing @Transactional on createOrder()
🟢 INFO: ProductDto.java:12 — Consider adding @NotNull validation
```

---

### `/verify`

```
> /verify
```

**Co robi:** Uruchamia pełny verification loop: kompilacja → testy → linting → typecheck → security check. Jeśli cokolwiek nie przechodzi, Claude próbuje to naprawić automatycznie.

**To jest komenda którą uruchamiacie NAJCZĘŚCIEJ podczas hackathonu** — po każdym większym bloku kodu.

**Przykładowy output:**
```
[1/5] Build... ✓ (mvn compile: 0 errors)
[2/5] Tests... ✓ (47/47 passing)
[3/5] Lint... ✗ (3 checkstyle warnings)
  → Auto-fixing checkstyle issues...
  ✓ Fixed
[4/5] Security... ✓ (no critical issues)
[5/5] Docker... ✓ (health check passing)
Verification complete ✓
```

---

### `/e2e`

```
> /e2e
> /e2e "Scenariusz: użytkownik tworzy zamówienie i sprawdza jego status"
```

**Co robi:** Generuje testy End-to-End przez agenta `e2e-runner`. Dla Spring Boot tworzy testy integracyjne z `@SpringBootTest`. Może też generować testy Playwright dla frontendu.

**Wygeneruje dla was:**
- Testy z `MockMvc` dla REST endpoints
- Setup/teardown bazy testowej
- Scenariusze happy path i error path
- Testy dla krytycznych user flows

---

### `/test-coverage`

```
> /test-coverage
> /test-coverage "backend/src/main/java/com/app/service/"
```

**Co robi:** Analizuje pokrycie testami — pokazuje które klasy i metody nie mają testów, oblicza % pokrycia, sugeruje jakie testy dopisać.

---

### `/security-scan`

```
> /security-scan
```

**Co robi:** Uruchamia AgentShield — system bezpieczeństwa z 102 regułami statycznej analizy. Skanuje: CLAUDE.md, settings.json, MCP configs, hooki, agentów. Szuka luk, misconfiguracji, wstrzyknięć.

**Dla hackathonu:** Uruchomcie raz na początku po skonfigurowaniu ECC żeby sprawdzić czy nic nie jest źle ustawione.

---

### `/quality-gate`

```
> /quality-gate
> /quality-gate "backend/src/"
```

**Co robi:** Uruchamia pipeline jakości ECC na wskazanym zakresie kodu lub całym projekcie. Bardziej kompleksowe niż `/verify` — sprawdza też metryki kodu, duplikacje, złożoność cyklomatyczną.

---

### `/eval`

```
> /eval "Sprawdź czy OrderService spełnia kryteria: transakcyjność, walidacja inputów, logowanie błędów"
```

**Co robi:** Ocenia kod względem konkretnych kryteriów które podajesz. Przydatne gdy chcecie sprawdzić czy implementacja spełnia wymagania biznesowe.

---

## 6. ECC: Slash Commands — naprawa błędów

### `/build-fix`

```
> /build-fix
```

**Co robi:** Analizuje błędy kompilacji (Maven/Gradle), identyfikuje ich przyczynę, naprawia je minimalną zmianą. Deleguje do agenta `java-build-resolver`.

**Scenariusz użycia:**
```
1. Wpisujesz: mvn compile
2. Widzisz: 15 błędów kompilacji
3. Wpisujesz: /build-fix
4. Claude: czyta błędy, naprawia jeden po drugim
5. Wynik: mvn compile → BUILD SUCCESS
```

**Ważne:** `/build-fix` robi minimalne zmiany — nie refaktoryzuje, nie "przy okazji" zmienia innych rzeczy. To chirurgiczne narzędzie.

---

### `/refactor-clean`

```
> /refactor-clean
> /refactor-clean "OrderService.java"
```

**Co robi:** Bezpiecznie identyfikuje i usuwa martwy kod. Używa agenta `refactor-cleaner`. Weryfikuje testy po każdym kroku usuwania — jeśli test się posypie, cofnie zmianę.

**Kiedy używać:** Po szybkiej hackathonowej implementacji, gdy chcecie posprzątać przed demo.

---

## 7. ECC: Slash Commands — multi-agent i orkiestracja

To są najbardziej zaawansowane komendy ECC. Uruchamiają **wiele agentów jednocześnie** pracujących równolegle.

### Kiedy UŻYWAĆ multi-agent vs zwykłego promptu

```
Zwykły prompt — gdy zadanie jest:
  - Proste (1-2 pliki)
  - Sekwencyjne (krok B zależy od kroku A)
  - Szybkie (<15 min implementacji)

Multi-agent — gdy zadanie jest:
  - Złożone (5+ plików)
  - Równoległe (moduły niezależne od siebie)
  - Wymaga specjalizacji (np. DB + BE + FE naraz)
```

---

### `/multi-plan`

```
> /multi-plan "Zaimplementuj pełny moduł zamówień: model, serwis, controller, testy, frontend"
```

**Co robi:** Rozkłada złożone zadanie na niezależne jednostki pracy dla wielu agentów. Tworzy plan orkiestracji — który agent robi co, w jakiej kolejności, jakie są zależności.

**Zawsze używajcie `/multi-plan` przed `/multi-execute`** — to jest "planowanie" a execute to "egzekucja".

---

### `/multi-execute`

```
> /multi-execute
```

**Co robi:** Wykonuje plan stworzony przez `/multi-plan`. Odpala agentów równolegle, monitoruje postęp, zbiera wyniki.

**Typowy flow:**
```
/multi-plan "Moduł płatności"
  → Plan: Agent A (model+repo) | Agent B (service) | Agent C (controller) | Agent D (testy)
/multi-execute
  → [Agent A] Creating Payment.java, PaymentRepository.java...
  → [Agent B] Creating PaymentService.java...
  → [Agent C] Creating PaymentController.java...
  → [Agent D] Creating PaymentControllerTest.java...
  → Merge: integruje wyniki wszystkich agentów
```

---

### `/multi-backend`

```
> /multi-backend "Zaimplementuj obsługę faktur: CRUD + walidacja + testy"
```

**Co robi:** Skrót dla typowego backendowego zadania. Automatycznie orkiestruje: model/entity, repository, service, controller, DTO, testy integracyjne — wszystko równolegle. Idealny dla nowych modułów CRUD.

**To jest wasza "killer feature" na hackathonie** dla BE devów. Jeden prompt generuje cały stos.

---

### `/multi-frontend`

```
> /multi-frontend "Stwórz widok listy faktur z filtrowaniem i paginacją"
```

**Co robi:** Orkiestruje równoległe tworzenie komponentów React: strona główna, komponenty UI, hooki, typy TypeScript, API client.

---

### `/multi-workflow`

```
> /multi-workflow "Dodaj notyfikacje email: model, service, controller, React UI, testy"
```

**Co robi:** Najbardziej rozbudowany workflow — łączy backend i frontend w jednym zadaniu. Orkiestruje agentów BE i FE jednocześnie.

---

### `/orchestrate`

```
> /orchestrate "Przeprowadź pełną integrację modułu zamówień z modułem faktur"
```

**Co robi:** Ogólny orchestrator dla złożonych wieloetapowych zadań. Bardziej elastyczny niż `multi-*` — możesz opisać dowolną sekwencję zdarzeń.

---

### `/pm2`

```
> /pm2
```

**Co robi:** Analizuje projekt i generuje konfigurację PM2 do zarządzania procesami Node.js. Przydatne jeśli macie jakikolwiek serwis Node w stacku (np. frontend dev server, proxy).

---

## 8. ECC: Slash Commands — zarządzanie sesją i pamięcią

### `/sessions`

```
> /sessions
```

**Co robi:** Pokazuje historię poprzednich sesji Claude Code. Możesz zobaczyć co robiłeś w poprzedniej sesji (nawet jeśli zamknąłeś terminal).

---

### `/learn`

```
> /learn
```

**Co robi:** W połowie sesji analizuje wzorce które się pojawiły i wyciąga z nich "instinkty" — małe reguły które Claude będzie pamiętał w przyszłości. Np. "w tym projekcie zawsze używamy Builder pattern".

**Kiedy używać:** Po dłuższej sesji gdzie Claude nauczył się specyfiki waszego projektu. Przed `/clear` — żeby nie stracić wyuczonej wiedzy.

---

### `/learn-eval`

```
> /learn-eval
```

**Co robi:** Jak `/learn` ale z oceną jakości wyciąganych wzorców. Zanim zapisze instynkt, sprawdza czy jest wystarczająco użyteczny.

---

### `/instinct-status`

```
> /instinct-status
```

**Co robi:** Pokazuje wszystkie wyuczone instynkty (wzorce) z ich confidence score. Możesz zobaczyć co Claude "zapamiętał" o projekcie.

Przykładowy output:
```
Instinct #1 [confidence: 0.89]: Always use @Transactional on service methods that modify data
Instinct #2 [confidence: 0.76]: Error responses use standard ApiErrorResponse class
Instinct #3 [confidence: 0.91]: SQLite — use INTEGER PRIMARY KEY, never SEQUENCE
```

---

### `/instinct-export` i `/instinct-import`

```
> /instinct-export
> /instinct-import instincts-backup.json
```

**Co robią:** Eksportują/importują wyuczone instynkty. Przydatne gdy chcecie współdzielić wiedzę między developerami w zespole.

**Scenariusz hackathonowy:** BE Lead uczy Claude specyfiki projektu przez 2h → `/instinct-export` → reszta zespołu `/instinct-import` — wszyscy mają ten sam kontekst.

---

### `/evolve`

```
> /evolve
```

**Co robi:** Grupuje podobne instynkty w formalne Skills. Jeśli 5 instynktów dotyczy obsługi błędów — `/evolve` tworzy z nich skill "error-handling-patterns".

---

### `/prune`

```
> /prune
```

**Co robi:** Usuwa instynkty starsze niż 30 dni lub o niskim confidence score. "Porządkowanie" wiedzy Claude.

---

## 9. ECC: Slash Commands — model routing i harness

### `/harness-audit`

```
> /harness-audit
```

**Co robi:** Audyt niezawodności harnessu — sprawdza konfigurację ECC, hooki, agenty, reguły. Daje ocenę gotowości systemu i listę problemów do naprawy.

**Uruchomcie raz na początku hackathonu** żeby upewnić się że ECC jest poprawnie zainstalowane.

---

### `/model-route`

```
> /model-route "Zaimplementuj algorytm wyceny produktów"
```

**Co robi:** Analizuje złożoność zadania i automatycznie wybiera optymalny model (haiku/sonnet/opus) bazując na wymaganiach i budżecie tokenowym. 

**Przydatne gdy nie wiecie jakiego modelu użyć** — Claude zdecyduje za was.

---

### `/loop-start` i `/loop-status`

```
> /loop-start "Co 60 minut: mvn test + jeśli testy padają, napraw je"
> /loop-status
```

**Co robią:** `/loop-start` uruchamia autonomiczną pętlę zadań. `/loop-status` sprawdza status uruchomionej pętli.

**Scenariusz:** Zostawcie uruchomione na QA komputerze — autonomicznie pilnuje żeby testy zawsze przechodziły.

---

### `/update-docs`

```
> /update-docs
> /update-docs "OrderService.java"
```

**Co robi:** Aktualizuje dokumentację (Javadoc, README, API docs) żeby była spójna z aktualnym kodem. Deleguje do agenta `doc-updater`.

---

### `/update-codemaps`

```
> /update-codemaps
```

**Co robi:** Aktualizuje "mapę kodu" projektu — plik opisujący strukturę i zależności modułów. Pomaga Claude orientować się w projekcie przy następnych sesjach.

---

## 10. ECC: Agenci — pełna lista i kiedy używać

Agenci to wyspecjalizowane "osobowości" AI. Możesz je wywołać bezpośrednio przez `@nazwa-agenta` w swoim prompcie.

### Jak wywoływać agentów

```
# Składnia:
"@nazwa-agenta [twoje polecenie]"

# Przykłady:
"@java-reviewer sprawdź OrderService.java"
"@architect zaproponuj strukturę dla modułu notyfikacji"
"@tdd-guide poprowadź implementację przez TDD"
```

Gdy Claude sam uzna że potrzebuje specjalisty, sam deleguje do odpowiedniego agenta. Ale możesz też wywołać ich ręcznie.

---

### Agenci dla waszego stacku (Java/Spring/Docker)

#### `@java-reviewer`

**Co robi:** Code review kodu Java/Spring Boot pod kątem: wzorców projektowych, SOLID principles, Spring best practices, wydajności, testowalności.

**Wywołaj gdy:**
- Skończyłeś implementację serwisu lub kontrolera
- Chcesz się upewnić że Spring annotations są użyte poprawnie
- Masz wątpliwości co do architektury modułu

```
"@java-reviewer Przejrzyj PaymentService.java. 
Szczególnie sprawdź: transakcje, obsługę wyjątków, czy nie ma wycieków zasobów"
```

---

#### `@java-build-resolver`

**Co robi:** Specjalista od naprawiania błędów budowania Java/Maven/Gradle. Analizuje stacktrace, identyfikuje root cause, naprawia minimalną zmianą.

**Wywołaj gdy:**
- `mvn compile` lub `mvn test` wyrzuca błędy których nie rozumiesz
- Są dependency conflicts w pom.xml
- Błędy są kaskadowe (jeden błąd powoduje 50 innych)

```
"@java-build-resolver mvn test wyrzuca: 
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-surefire-plugin...
[pełny stacktrace]"
```

---

#### `@database-reviewer`

**Co robi:** Ekspert od baz danych i JPA/Hibernate. Sprawdza: zapytania JPQL/HQL, N+1 problem, konfigurację relacji, migracje, indeksy.

**Wywołaj gdy:**
- Masz wolne zapytania
- JPA generuje dziwny SQL
- Nie jesteś pewny mappingów @OneToMany/@ManyToOne
- SQLite zachowuje się inaczej niż oczekujesz

```
"@database-reviewer Sprawdź OrderRepository.findByUserWithProducts().
Podejrzewam N+1. Aktualny kod: [wklej kod]"
```

---

#### `@tdd-guide`

**Co robi:** Mentor TDD. Prowadzi przez cały cykl RED-GREEN-REFACTOR, sugeruje co testować, pomaga pisać asercje, wskazuje kiedy test jest zbyt szeroki/wąski.

**Wywołaj gdy:**
- Nie wiesz od czego zacząć pisanie testów
- Twoje testy przechodzą ale nie testują tego co powinny
- Chcesz nauczyć się TDD w praktyce

```
"@tdd-guide Chcę zaimplementować metodę obliczania rabatu w CartService.
Poprowadź mnie przez TDD krok po kroku"
```

---

#### `@architect`

**Co robi:** Senior architect AI. Projektuje struktury systemów, ocenia decyzje techniczne, proponuje wzorce dla złożonych problemów.

**Wywołaj gdy:**
- Stoisz przed decyzją architektoniczną (np. jak zorganizować moduły)
- Planujesz integrację Kafka/external API
- Masz problem z wydajnością systemu jako całości

```
"@architect Mamy Spring Boot monolith z SQLite. 
Chcemy dodać async processing dla heavy operacji.
Rekomenduj podejście: Spring @Async vs Kafka vs osobny serwis"
```

---

#### `@planner`

**Co robi:** Planowanie implementacji. Dostaje wymaganie biznesowe i tworzy szczegółowy plan techniczny — listę plików, kolejność implementacji, zależności.

**Wywołaj gdy:**
- Dostajesz nowe wymaganie i nie wiesz od czego zacząć
- Zadanie jest złożone i chcesz upewnić się że nic nie przegapisz

```
"@planner Zaimplementuj eksport danych do CSV.
Wymagania: user może wybrać zakres dat, eksport zamówień, asynchroniczny"
```

---

#### `@code-reviewer`

**Co robi:** Ogólny code reviewer (język-agnostyczny). Skupia się na jakości, czytelności, security, testowalności.

```
"@code-reviewer Zrób review ostatnich zmian w katalogu backend/"
```

---

#### `@security-reviewer`

**Co robi:** Specjalista bezpieczeństwa. Szuka: SQL injection, XSS, insecure deserialization, broken auth, exposed secrets, OWASP Top 10.

**Wywołaj gdy:**
- Implementujesz autoryzację/autentykację
- Piszesz kod który przyjmuje input od użytkownika
- Przed demo — ostateczny security check

```
"@security-reviewer Sprawdź AuthController.java i JWT konfigurację.
Szukaj: token validation, expiry handling, secret management"
```

---

#### `@e2e-runner`

**Co robi:** Generuje i uruchamia testy E2E. Rozumie Playwright i Spring Boot `@SpringBootTest`.

```
"@e2e-runner Napisz testy E2E dla przepływu: login → create order → view order history"
```

---

#### `@refactor-cleaner`

**Co robi:** Bezpieczna eliminacja martwego kodu. Po każdym usunięciu uruchamia testy — jeśli padną, cofa zmianę.

```
"@refactor-cleaner Posprzątaj OrderService.java — usuń nieużywane metody"
```

---

#### `@doc-updater`

**Co robi:** Aktualizuje dokumentację. Synchronizuje Javadoc z kodem, aktualizuje README, generuje API docs.

```
"@doc-updater Zaktualizuj Javadoc w OrderController.java po ostatnich zmianach"
```

---

## 11. ECC: Skills — jak działają i jak aktywować

### Co to jest Skill

Skill to plik `SKILL.md` z instrukcjami (playbook) dla Claude. Gdy Claude dostaje zadanie pasujące do skilla — automatycznie ładuje odpowiedni playbook i pracuje według niego.

Przykład: gdy poprosisz o implementację w Spring Boot, Claude ładuje skill `springboot-patterns` który zawiera best practices, wzorce, jak budować w waszym projekcie.

### Dwa sposoby aktywacji skilla

```
# 1. Automatyczny — Claude sam wykrywa pasujący skill
"Zaimplementuj Spring Security z JWT"
→ Claude automatycznie ładuje springboot-security

# 2. Ręczny — ty decydujesz
"/springboot-tdd Zaimplementuj CartService"
→ Claude wymusza użycie TDD workflow dla Spring Boot
```

### Skills zainstalowane przez ECC które wam się przydadzą

#### `springboot-patterns`
Best practices architektoniczne dla Spring Boot: jak organizować kod, wzorce Repository, Service, Controller, obsługa błędów, konfiguracja.

**Aktywuje się automatycznie** gdy piszesz kod Spring Boot.

Zawiera wzorce takie jak:
```java
// Claude będzie generował kod w tym stylu:
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class OrderService {
    private final OrderRepository orderRepository;
    
    public OrderResponseDto findById(Long id) {
        return orderRepository.findById(id)
            .map(OrderMapper::toDto)
            .orElseThrow(() -> new ResourceNotFoundException("Order", id));
    }
}
```

---

#### `springboot-tdd`
Workflow TDD specyficzny dla Spring Boot: jak pisać testy przed kodem, jak używać MockMvc, jak mockować zależności, jakie asercje stosować.

```
/springboot-tdd "Zaimplementuj endpoint do aktualizacji statusu zamówienia"
```

---

#### `springboot-security`
Security best practices dla Spring: konfiguracja Spring Security, JWT, CORS, walidacja inputów, ochrona przed OWASP Top 10.

```
/springboot-security "Skonfiguruj JWT auth"
```

---

#### `springboot-verification`
Verification loops dla Spring Boot: sekwencja kroków weryfikacji po każdej zmianie (compile → test → integration test → docker).

---

#### `docker-patterns`
Best practices Docker: multi-stage builds, sieciowanie kontenerów, wolumeny, health checks, bezpieczeństwo obrazów.

**Aktywuje się automatycznie** gdy edytujesz Dockerfile lub docker-compose.yml.

---

#### `api-design`
REST API design: nazewnictwo endpointów, kody odpowiedzi, paginacja, obsługa błędów, wersjonowanie API.

---

#### `database-migrations`
Wzorce migracji bazy danych: Liquibase, plain SQL, rollback strategy, seed data.

---

#### `e2e-testing`
Playwright E2E patterns: Page Object Model, selektory, obsługa async, CI integration.

---

### Jak sprawdzić dostępne skills

```
> /skills
```

Wylistuje wszystkie zainstalowane skills z opisem.

---

## 12. ECC: Hooks — co się dzieje automatycznie

Hooks to automatyzacje które włączają się bez waszego udziału gdy Claude wykonuje określone akcje.

### Czego się spodziewać (co się włącza samo)

#### Po każdej edycji pliku Java:
- Auto-format kodu (jeśli skonfigurowany)
- Sprawdzenie czy dodano `console.log` lub `System.out.println` (przypomnienie o usunięciu)
- Typecheck/kompilacja sprawdzana inkrementalnie

#### Przy starcie sesji (SessionStart hook):
- Ładowanie kontekstu z poprzedniej sesji jeśli był zapisany
- Odczytanie `CLAUDE.md` i aktualizacja "pamięci" Claude

#### Przy zakończeniu sesji (Stop/SessionEnd hook):
- Zapisanie podsumowania sesji do `~/.claude/session-data/`
- Extractowanie wzorców do instynktów (jeśli skonfigurowane)

#### Przy próbie commitu (pre-commit hook):
- Sprawdzenie czy nie ma hardcoded secrets (sk-, ghp_, AKIA)
- Ostrzeżenie jeśli commit zawiera pliki .env

### Jak sprawdzić jakie hooks są aktywne

```bash
cat ~/.claude/settings.json | grep -A 50 "hooks"
# lub
cat hooks/hooks.json  # w katalogu ECC
```

### Wyłączanie hooków (jeśli coś przeszkadza)

```bash
# W terminalu przed uruchomieniem claude:
export ECC_HOOK_PROFILE=minimal     # tylko krytyczne hooki
export ECC_HOOK_PROFILE=standard    # domyślne (zalecane)
export ECC_HOOK_PROFILE=strict      # wszystkie hooki

# Wyłączenie konkretnego hooku:
export ECC_DISABLED_HOOKS="pre:bash:tmux-reminder,post:edit:typecheck"
```

---

## 13. ECC: Rules — zasady które Claude zawsze stosuje

Rules to pliki w `~/.claude/rules/` które Claude zawsze bierze pod uwagę, niezależnie od promptu. Nie musisz ich wywoływać — działają cały czas w tle.

### Co zawierają zainstalowane rules

#### `common/coding-style.md`
- Tworzenie nowych obiektów zamiast mutowania (immutability)
- Małe pliki: 200-400 linii typowo, max 800
- Organizacja po feature/domenie, nie po typie
- Spójne nazewnictwo

#### `common/git-workflow.md`
- Format commitów: `feat:`, `fix:`, `test:`, `refactor:`
- Zakaz `git push --force` bez przeglądu
- PR process

#### `common/testing.md`
- TDD jako domyślny workflow
- Minimum 80% pokrycia dla nowego kodu
- Testy niezależne od siebie (każdy czyści po sobie)

#### `common/security.md`
- Nigdy nie hardkoduj secretów
- Walidacja wszystkich inputów
- Mandatory security checklist dla nowych endpointów
- Jeśli znajdziesz security issue: STOP → security-reviewer → napraw

#### `common/performance.md`
- Wybór modelu AI (sonnet domyślnie, opus dla złożonych)
- Zarządzanie kontekstem sesji

---

## 14. Zarządzanie modelami i tokenami

### Modele i ich przeznaczenie

```
haiku   → Proste zadania, szybkie odpowiedzi, najtańszy
          Np: formatowanie, proste refaktoring, generowanie DTO

sonnet  → 90% zadań, najlepszy balans jakości/kosztu [DEFAULT]
          Np: implementacja feature, testy, code review

opus    → Złożona architektura, trudny debug, strategiczne decyzje
          Np: projektowanie systemu, rozwiązywanie nietrywialnych bugów
          ⚠️ Drogi — używajcie tylko gdy sonnet "nie daje rady"
```

### Konfiguracja w `~/.claude/settings.json`

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

Wyjaśnienie każdego ustawienia:

```
"model": "sonnet"
  → domyślny model dla wszystkich zadań

"MAX_THINKING_TOKENS": "10000"
  → Claude może "myśleć" przed odpowiedzią (ukryte rozumowanie)
  → domyślnie: 31999 tokenów (!), 10000 to ~70% redukcja kosztów
  → dla hackathonu: 10000 wystarczy w 95% przypadków

"CLAUDE_AUTOCOMPACT_PCT_OVERRIDE": "50"
  → auto-compact sesji gdy zużycie okna kontekstu osiągnie 50%
  → domyślnie: 95% (bardzo późno!)
  → przy 50%: lepsze odpowiedzi bo mniej "zaśmieconego" kontekstu

"CLAUDE_CODE_SUBAGENT_MODEL": "haiku"
  → subagenci (spawnovani przez Claude) używają haiku
  → oszczędność bez utraty jakości dla prostszych podtasków
```

### Strategia tokeny w hackathonie

```
H0-H3: wydawajcie śmiało — to czas na fundamenty
H3-H6: ostrożniej — główne features
H6-H8: ekonomicznie — polish i demo
```

Regularnie sprawdzajcie `/cost`. Jeśli sesja kosztuje >$1 — czas na `/compact`.

### Przełączanie modelu w locie

```
> /model opus
"Zaprojektuj architekturę systemu notyfikacji dla naszego Spring Boot monolith.
Mamy ograniczenia: SQLite, max 1000 req/s, Docker env."

(po decyzji architektonicznej)

> /model sonnet
"OK, zaimplementuj to co zaplanowaliśmy"
```

---

## 15. Wzorce promptowania — jak pisać skuteczne polecenia

### Zasada CCCE: Cel, Kontekst, Constrainty, Efekt

```
Cel:        Co chcesz osiągnąć (czasownik + rezultat)
Kontekst:   Relevantne informacje (pliki, zależności, aktualny stan)
Constrainty: Ograniczenia (technologie, wzorce, czego nie robić)
Efekt:      Jakiego formatu wyjścia oczekujesz
```

**Przykład bez CCCE (złe):**
```
"Napisz endpoint"
```

**Przykład z CCCE (dobre):**
```
Cel:       "Zaimplementuj endpoint do pobierania historii zamówień użytkownika"
Kontekst:  "Mamy Order entity (id, userId, status, createdAt), OrderRepository extends JpaRepository"
Constrainty: "Paginacja wymagana, SQLite backend, nie używaj native queries"
Efekt:     "Napisz najpierw test w TDD, potem implementację, zwróć Page<OrderResponseDto>"
```

---

### Wzorzec: "Najpierw przeczytaj, potem pisz"

Dla zadań które dotykają istniejącego kodu:

```
"Zanim zaczniesz pisać kod:
1. Przeczytaj OrderService.java
2. Przeczytaj OrderRepository.java  
3. Przeczytaj schemat bazy db/schema.sql
Potem zaimplementuj [zadanie] spójnie ze stylem który widzisz"
```

---

### Wzorzec: Atomic task

Jeden prompt = jedno jasno określone zadanie. Nie:
```
"Napisz serwis, controller, testy, DTO i zaktualizuj dokumentację"
```
Zamiast tego seria promptów:
```
"Napisz OrderService z metodami: findById, create, updateStatus"
(po implementacji i review)
"Napisz OrderController używający OrderService który właśnie stworzyliśmy"
(po implementacji)
"Napisz testy integracyjne dla OrderController"
```

---

### Wzorzec: Contextual reminder dla SQLite

Zawsze gdy piszecie coś związanego z bazą danych:

```
"Pamiętaj: nasza baza to SQLite, nie PostgreSQL.
Oznacza to: INTEGER PRIMARY KEY AUTOINCREMENT (nie SEQUENCE),
dialekt: org.hibernate.community.dialect.SQLiteDialect,
brak JSONB/ARRAY/UUID natywnych typów.
Z tym ograniczeniem: [twoje zadanie]"
```

---

### Wzorzec: Explicit output format

Gdy chcecie konkretny format kodu:

```
"Wygeneruj klasę Java zgodnie z tym wzorcem:
- Lombok @Data, @Builder, @NoArgsConstructor, @AllArgsConstructor
- Pola private final (immutable gdzie możliwe)
- Brak logiki w DTO
- Walidacja Bean Validation (@NotNull, @Size etc.)
Dla encji: OrderCreateRequestDto"
```

---

### Wzorzec: Resume after context loss

Gdy Claude "zapomniał" co robiliście:

```
"Kontynuuj pracę nad modułem zamówień.
Dotychczas zrobiliśmy:
- Order entity + OrderRepository
- OrderService z metodami CRUD
Do zrobienia teraz:
- OrderController z endpointami REST
- Testy integracyjne
Aktualny status testów: 23/23 zielone
Kontynuuj od OrderController."
```

---

### Anty-wzorce — czego unikać

```
❌ "Napraw to" — co? jak? gdzie?
❌ "Zrób lepiej" — według jakich kryteriów?
❌ "Napisz całą aplikację" — za duże, Claude wyprodukuje bałagan
❌ "Jak myślisz co powinienem zrobić?" — to nie chatbot do rozmyślań, to narzędzie do dostarczania
❌ Wklejanie 500 linii kodu bez kontekstu co z tym zrobić
```

---

## 16. Typowe scenariusze hackathonowe — gotowe sekwencje

### Scenariusz A: Inicjalizacja projektu Spring Boot od zera

```bash
cd hackathon-app
claude

> /plan "Stwórz Spring Boot 3.x projekt od podstaw:
  - Java 21, Maven
  - Dependencies: spring-web, spring-data-jpa, xerial/sqlite-jdbc, lombok, spring-boot-test
  - application.properties z SQLite config
  - HealthController na /actuator/health
  - Dockerfile multi-stage (build + runtime)
  - docker-compose.yml z backend service
  Nie piszemy żadnej logiki biznesowej, tylko fundament"
  
> (review planu)
> "Zatwierdź i wykonaj"
> /verify
```

---

### Scenariusz B: Nowy moduł CRUD end-to-end

```bash
> /multi-backend "Zaimplementuj pełny moduł Product:
  Encja: Product (id Long, name String, price BigDecimal, stock Integer, createdAt LocalDateTime)
  
  Wymagane:
  - Product.java (@Entity, Lombok, walidacja)
  - ProductRepository extends JpaRepository
  - ProductService (findAll, findById, create, update, delete)
  - ProductController (/api/v1/products, pełne CRUD)
  - ProductRequestDto, ProductResponseDto
  - Testy integracyjne dla każdego endpointu
  
  Constraints: SQLite, TDD, error handling z custom exceptions"

> /verify
> /code-review
```

---

### Scenariusz C: Debugowanie błędu build

```bash
# Po błędzie mvn test:
> /build-fix
# lub jeśli złożony:
> "@java-build-resolver Mam taki błąd: [wklej pełny stacktrace Maven]
  Napraw minimalną zmianą, nie refaktoryzuj"
```

---

### Scenariusz D: Szybki React CRUD frontend

```bash
> /multi-frontend "Stwórz widok Products:
  - ProductList.tsx: tabela z produktami, kolumny: name, price, stock, akcje (edit/delete)
  - ProductForm.tsx: formularz tworzenia/edycji produktu
  - useProducts.ts: hook zarządzający stanem (fetch, create, update, delete)
  - api/products.ts: axios client dla /api/v1/products
  
  TypeScript, obsługa loading/error state, responsywne"
```

---

### Scenariusz E: Przed demo — weryfikacja całości

```bash
> /verify                        # czy kod się kompiluje i testy przechodzą?
> /security-scan                 # czy nie ma oczywistych luk?
> /e2e "Napisz smoke test który:
  1. Tworzy produkt (POST /api/v1/products)
  2. Pobiera go (GET /api/v1/products/{id})
  3. Aktualizuje (PUT /api/v1/products/{id})
  4. Usuwa (DELETE /api/v1/products/{id})
  Weryfikuje każdy krok przez status code i response body"
  
> "@security-reviewer Ostateczny przegląd przed demo:
  sprawdź ProductController.java i AuthConfig.java"
```

---

### Scenariusz F: Ratowanie przed końcem hackathonu

Zostało 90 minut, coś nie działa:

```bash
# 1. Oceń sytuację
> "/harness-audit co jest broken?"

# 2. Jeśli build nie chodzi:
> /build-fix

# 3. Jeśli testy padają:
> "@java-build-resolver Testy w OrderServiceTest padają z tym błędem: [stacktrace]
  Napraw TYLKO ten test, nie rusz innych plików"

# 4. Jeśli Docker nie wstaje:
> "@architect Docker health check nie przechodzi, błąd: [logi].
  Podaj szybkie rozwiązanie, priorytet: działające demo za 90 minut"

# 5. Jeśli nie zdążycie zrobić feature:
> "Upewnij się że to co mamy DZIAŁA stabilnie.
  Usuń wszystko co może się posypać podczas demo.
  Priorytet: stabilność nad completeness"
```

---

## 17. Troubleshooting — co gdy coś nie działa

### Problem: Komendy ECC nie działają (`/plan`, `/tdd` etc. nie są rozpoznawane)

```bash
# Sprawdź czy ECC jest zainstalowane:
ls ~/.claude/commands/
# Powinien być tam plan.md, tdd.md, build-fix.md etc.

# Jeśli brak:
cd everything-claude-code
./install.sh java

# Sprawdź plugin:
# W sesji claude:
/plugin list everything-claude-code@everything-claude-code
```

---

### Problem: Claude nie widzi CLAUDE.md

```bash
# Sprawdź czy jesteś w właściwym katalogu:
pwd  # powinno być /ścieżka/do/projektu
ls CLAUDE.md  # plik musi istnieć

# Zawsze uruchamiaj claude z katalogu projektu:
cd /ścieżka/do/projektu && claude
```

---

### Problem: "Duplicate hooks file detected"

```
Duplicate hooks file detected: ./hooks/hooks.json resolves to already-loaded file
```

To znany bug ECC. Rozwiązanie:

```bash
# Sprawdź settings.json czy nie ma explicitnie "hooks" field:
cat ~/.claude/settings.json | grep hooks

# Jeśli widzisz: "hooks": "hooks/hooks.json"
# Usuń tę linię - Claude Code v2.1+ ładuje hooks.json automatycznie
```

---

### Problem: Claude "głupieje" po długiej sesji

Objawy: generuje niespójny kod, "zapomina" wcześniej ustalony styl, robi dziwne rzeczy.

```bash
# Rozwiązanie 1: compact
> /compact "Zachowaj: architekturę projektu, nazwy klas, wzorzec obsługi błędów"

# Rozwiązanie 2: clear + context dump
> "Przed resetem: podaj 5-zdaniowe podsumowanie aktualnego stanu projektu"
# (zanotuj odpowiedź)
> /clear
# (wklej podsumowanie jako pierwszy prompt w nowej sesji)
```

---

### Problem: Zbyt dużo tokenów / limity Claude Max

```bash
# Sprawdź zużycie:
> /cost

# Jeśli wysoko:
> /compact     # zmniejsza okno

# Jeśli sesja jest "ciężka":
> /clear       # reset + zacznij od świeżego kontekstu
```

---

### Problem: Hooks blokują Claude (nie może edytować pliku)

```bash
# Tymczasowe wyłączenie hooków:
export ECC_HOOK_PROFILE=minimal
claude

# Po naprawieniu wrócić do normalnego:
export ECC_HOOK_PROFILE=standard
```

---

### Problem: `/verify` ciągle się sypie

```bash
# Izoluj który krok pada:
> "Uruchom tylko mvn compile i powiedz co widzisz"
> "Uruchom tylko mvn test -pl backend i powiedz co widzisz"
> "Sprawdź czy docker-compose up działa bez --build"

# Jeśli test konkretny pada:
> "@java-build-resolver Ten test ciągle pada: [nazwa testu + błąd].
  Nie zmieniaj nic poza tym co bezpośrednio powoduje błąd"
```

---

### Problem: Agent nie odpowiada / timeout

```bash
# Jeśli subagent się zawiesił:
[Ctrl+C]  # przerwij bieżącą operację

# Spróbuj prostszego podejścia:
> /model sonnet  # upewnij się że nie jesteś na opus (wolniejszy)
> "Wykonaj samo [konkretne małe zadanie] - nic więcej"
```

---

## Szybka ściąga — "cheat sheet na ścianę"

```
┌─────────────────────────────────────────────────────────┐
│              CLAUDE CODE + ECC — ŚCIĄGA                 │
├─────────────────┬───────────────────────────────────────┤
│ PLANOWANIE      │ /plan "opis"                          │
│                 │ /multi-plan "złożone zadanie"         │
├─────────────────┼───────────────────────────────────────┤
│ IMPLEMENTACJA   │ /tdd [opis]                           │
│                 │ /multi-backend "pełny moduł CRUD"     │
│                 │ /multi-frontend "widok React"         │
│                 │ /multi-workflow "BE+FE razem"         │
├─────────────────┼───────────────────────────────────────┤
│ JAKOŚĆ          │ /verify          ← używajcie często!  │
│                 │ /code-review                          │
│                 │ /e2e                                  │
│                 │ /test-coverage                        │
├─────────────────┼───────────────────────────────────────┤
│ NAPRAWA         │ /build-fix                            │
│                 │ /refactor-clean                       │
│                 │ @java-build-resolver [błąd]           │
├─────────────────┼───────────────────────────────────────┤
│ SESJA           │ /cost  ← sprawdzaj regularnie         │
│                 │ /compact [wskazówka]                  │
│                 │ /clear                                │
│                 │ /diff  ← przed każdym commitem        │
├─────────────────┼───────────────────────────────────────┤
│ MODEL           │ /model sonnet  ← domyślny             │
│                 │ /model opus    ← architektura/debug   │
│                 │ /model haiku   ← proste zadania       │
├─────────────────┼───────────────────────────────────────┤
│ AGENCI          │ @java-reviewer                        │
│                 │ @java-build-resolver                  │
│                 │ @database-reviewer                    │
│                 │ @architect                            │
│                 │ @tdd-guide                            │
│                 │ @security-reviewer                    │
├─────────────────┼───────────────────────────────────────┤
│ DIAGNOSTYKA     │ /harness-audit  ← na start dnia       │
│                 │ /security-scan  ← przed demo          │
│                 │ /instinct-status                      │
└─────────────────┴───────────────────────────────────────┘
```

---

*Wersja: Hackathon 2026 · Stack: Java Spring + SQLite + React + Docker · ECC v1.9.0*

