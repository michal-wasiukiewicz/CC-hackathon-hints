# Claude Code — skróty klawiszowe

> Plik dla nowych użytkowników. Skróty z ★ warto nauczyć się w pierwszej kolejności.

---

## Tryby pracy

| Skrót | Opis |
|---|---|
| `Shift` + `Tab` ★ | **Przełącza tryb pracy.** Naciśnij raz → tryb auto-accept (Claude wykonuje zmiany bez pytania). Naciśnij dwa razy → plan mode (Claude opisuje co zrobi, ale nic nie rusza). **Dla nowych użytkowników:** zawsze zacznij od plan mode przy nieznanym zadaniu — zobaczysz intencję Claude zanim cokolwiek się zmieni w plikach. |
| `Alt` + `T` | **Extended thinking.** Włącza głębszy tryb rozumowania — Claude myśli dłużej przed odpowiedzią. Przydatne przy skomplikowanych błędach, architekturze lub refaktoringu wielu plików. Kosztuje więcej tokenów. |
| `Alt` + `P` | **Zmiana modelu w locie.** Możesz przełączyć się na Opus gdy utknąłeś na trudnym problemie, a wrócić na Sonnet do zwykłych zadań. |

---

## Kontrola sesji

| Skrót | Opis |
|---|---|
| `Ctrl` + `C` | **Przerywa bieżącą operację.** Używaj gdy Claude gdzieś utknął lub idzie w złym kierunku. Nie cofa już wykonanych zmian w plikach. |
| `Esc` `Esc` ★ | **Rewind menu — cofnij zmiany.** Naciśnij Esc dwa razy szybko. Otwiera menu z listą ostatnich kroków — możesz wybrać do którego punktu wrócić. **Dla nowych użytkowników:** to Twoja sieć bezpieczeństwa. Jeśli Claude zrobił coś nieoczekiwanego, nie panikuj — Esc+Esc cofnie zmiany bez ruszania git. Znacznie bezpieczniejsze niż Ctrl+C, które tylko przerywa bez cofania. |
| `Ctrl` + `D` | **Wyjście z Claude Code.** Czyste zamknięcie sesji. |
| `Ctrl` + `F` ★ | **Wyszukiwanie w historii konwersacji.** Naciśnij dwa razy szybko. Przydatne gdy szukasz konkretnej decyzji lub wyjaśnienia z wcześniejszego etapu sesji — Claude często tłumaczy architekturę przy pierwszych krokach, a warto do tego wrócić. |

---

## Edycja wejścia

| Skrót | Opis |
|---|---|
| `Shift` + `Enter` | **Nowa linia bez wysyłania.** Pozwala pisać wieloliniowe prompty bezpośrednio w terminalu. W niektórych terminalach wymaga dodatkowej konfiguracji — alternatywą jest Ctrl+G. |
| `Ctrl` + `G` ★ | **Otwórz prompt w zewnętrznym edytorze.** Otwiera Twój domyślny edytor (np. Vim, Nano, VS Code). Po zapisaniu i zamknięciu pliku — prompt trafia do Claude. **Dla nowych użytkowników:** przy długich, złożonych instrukcjach to ogromna ulga. Możesz spokojnie napisać 10-zdaniowy opis zadania z przykładami, bez klepania w jednej linii terminala. Ustaw edytor: `export EDITOR="code --wait"` w `.bashrc`/`.zshrc`. |
| `Tab` | **Autouzupełnianie.** Uzupełnia ścieżki plików, nazwy katalogów i komendy slash. Działa tak samo jak w zwykłym terminalu. |
| `↑` / `↓` | **Historia poleceń.** Przewija wcześniej wysłane prompty — przydatne do ponowienia lub lekkiej modyfikacji poprzedniego polecenia. |
| `Alt` + `B` | **Cofnij o jedno słowo.** Szybsze niż trzymanie Backspace przy dłuższym prompcie. |
| `Alt` + `F` | **Przejdź o jedno słowo do przodu.** Para do Alt+B. |

> **Uwaga:** Skróty `Alt` mogą wymagać konfiguracji terminala.
> W iTerm2: Settings → Profiles → Keys → ustaw Left/Right Option jako `Esc+`.
> W Windows Terminal: Settings → Actions → sprawdź mapowanie Alt.

---

## Komendy slash

| Komenda | Opis |
|---|---|
| `/init` ★ | **Wygeneruj CLAUDE.md dla projektu.** Analizuje strukturę projektu i tworzy plik konfiguracyjny z opisem stacku, konwencji i preferencji. Claude czyta ten plik automatycznie przy każdej sesji — dzięki temu nie musisz za każdym razem tłumaczyć co to za projekt. **Dla nowych użytkowników:** uruchom to jako pierwszy krok w każdym nowym projekcie. |
| `/plan` ★ | **Tryb planowania.** Claude opisuje krok po kroku co zamierza zrobić — lista plików do zmiany, podejście, potencjalne ryzyka. Nic nie jest jeszcze wykonywane. Możesz odpowiedzieć "tak, działaj" albo "zmień podejście w kroku 3". **Dla nowych użytkowników:** przy każdym zadaniu dotykającym wielu plików najpierw `/plan`. Oszczędza frustrację gdy Claude idzie w złą stronę. |
| `/compact` | **Skompresuj kontekst.** Gdy okno kontekstu jest zapełnione w >80%, Claude zaczyna "zapominać" wcześniejsze części rozmowy. `/compact` podsumowuje historię do krótszej formy. Możesz dodać wskazówkę: `/compact zachowaj decyzje architektoniczne`. |
| `/clear` | **Czyść kontekst, zacznij od nowa.** Nowa sesja bez historii. Używaj gdy zaczynasz inne zadanie lub gdy kontekst jest zbyt duży i spowalnia odpowiedzi. |
| `/model` | **Zmień model w trakcie sesji.** Np. `/model opus` przy trudnym debugowaniu, `/model sonnet` do zwykłych zadań. Kontekst rozmowy jest zachowany. |
| `/btw` | **Pytanie poza kontekstem.** Zadaj szybkie pytanie do Claude bez wplątywania go w historię sesji. Np. `/btw jak działa useEffect cleanup` — Claude odpowie bez zaśmiecania kontekstu. |
| `/vim` | **Tryb vim do edycji wejścia.** Dla użytkowników vim — włącza nawigację i edycję modalną bezpośrednio w wierszu poleceń. |
| `/cost` | **Sprawdź zużycie tokenów i szacowany koszt sesji.** Przydatne przy pracy na Claude Pro — pozwala ocenić jak intensywna była sesja. |
| `/doctor` | **Diagnostyka instalacji.** Sprawdza czy Claude Code jest poprawnie zainstalowany, skonfigurowany i połączony z kontem. Uruchom jako pierwszą rzecz gdy coś nie działa. |
| `?` | **Lista dostępnych skrótów.** Wyświetla skróty aktywne w Twoim środowisku i terminalu. Różne systemy mogą mieć różne mapowania. |

---

## CLI — flagi startowe

| Komenda | Opis |
|---|---|
| `claude -c` ★ | **Kontynuuj poprzednią sesję.** Wraca do ostatniej rozmowy z zachowanym kontekstem. **Dla nowych użytkowników:** nie musisz od zera tłumaczyć projektu przy każdym uruchomieniu — `-c` kontynuuje tam gdzie skończyłeś. |
| `claude -r "id"` | **Wznów konkretną sesję po ID.** ID sesji możesz znaleźć w `~/.claude/sessions/`. Przydatne gdy wracasz do zadania sprzed kilku dni. |
| `claude --model opus` | **Uruchom z wybranym modelem.** Alternatywa dla zmiany przez `/model` w trakcie sesji. |
| `claude --no-tools` | **Uruchom bez dostępu do narzędzi.** Claude nie będzie mógł pisać/edytować plików — tryb tylko do rozmowy i planowania. |

---

## Referencje w prompcie

| Składnia | Opis |
|---|---|
| `@plik.ts` ★ | **Dołącz plik do kontekstu.** Wpisz `@` i zacznij pisać nazwę pliku — autouzupełnianie pokaże dopasowania. Claude zobaczy zawartość pliku bez konieczności otwierania go. Np. `sprawdź czy @src/api/auth.ts obsługuje błędy sieci`. |
| `@folder/` | **Dołącz cały katalog.** Wszystkie pliki z folderu trafiają do kontekstu. Ostrożnie z dużymi katalogami — zapełnia kontekst. |
| `! polecenie` | **Uruchom polecenie shell.** Wynik trafia bezpośrednio do kontekstu Claude. Np. `! git diff HEAD~1` — Claude zobaczy ostatni commit i może go skomentować lub naprawić. Przydatne też do: `! npm test`, `! cat package.json`. |

---

## Dobre praktyki dla nowych użytkowników

**Zanim zaczniesz nowy projekt:**
1. Uruchom `/init` — wygeneruj CLAUDE.md
2. Przejrzyj i uzupełnij CLAUDE.md o specyfikę projektu (framework, konwencje nazewnictwa, co NIE ruszać)

**Przy każdym większym zadaniu:**
1. Użyj `Shift+Tab` żeby wejść w plan mode
2. Przeczytaj plan Claude — popraw jeśli coś się nie zgadza
3. Dopiero zatwierdź wykonanie

**Gdy coś pójdzie nie tak:**
- `Esc` + `Esc` → rewind do bezpiecznego punktu
- `! git status` → sprawdź co się zmieniło
- `/clear` → zacznij nową sesję z czystym kontekstem
