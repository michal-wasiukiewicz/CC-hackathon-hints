# Claude Code — instalacja CLI i plugin JetBrains

> Wymagania wstępne: konto Claude Pro ($20/mies.) lub wyższe. Plan Free nie obejmuje Claude Code.

---

## Część 1 — Instalacja CLI

### Wymagania systemowe

| System | Minimalna wersja |
|---|---|
| macOS | 13.0 (Ventura) lub nowszy |
| Windows | Windows 10 build 1809+ |
| Linux | Ubuntu 20.04+, Debian 10+ |

---

### macOS i Linux — natywny installer (zalecane)

Natywny installer nie wymaga Node.js. Instaluje binarny plik wykonywalny i automatycznie aktualizuje się w tle.

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

Po instalacji otwórz **nowe okno terminala** (konieczne, żeby PATH się odświeżył), a następnie sprawdź instalację:

```bash
claude --version
claude doctor
```

`claude doctor` to wbudowana diagnostyka — jeśli coś poszło nie tak, wypisze co konkretnie naprawić.

#### Alternatywa: Homebrew (macOS)

```bash
brew install claude-code
```

> **Uwaga:** Homebrew nie aktualizuje się automatycznie. Pamiętaj o `brew upgrade claude-code` co jakiś czas.

---

### Windows — natywny installer

#### Opcja 1: PowerShell (najszybsza)

```powershell
irm https://claude.ai/install.ps1 | iex
```

> Nie uruchamiaj PowerShell jako administrator — nie jest to wymagane i może powodować problemy z uprawnieniami.

#### Opcja 2: WinGet

```powershell
winget install Anthropic.ClaudeCode
```

> WinGet nie aktualizuje się automatycznie. Uruchamiaj `winget upgrade Anthropic.ClaudeCode` co jakiś czas.

#### Wymaganie: Git for Windows

Claude Code używa Git Bash wewnętrznie do uruchamiania komend. Zainstaluj go przed CLI jeśli jeszcze nie masz:
- Pobierz z: https://git-scm.com/downloads/win
- Podczas instalacji zaznacz „Add Git Bash to PATH"

Jeśli Claude Code nie może znaleźć Git Bash po instalacji, ustaw ścieżkę ręcznie w pliku `%USERPROFILE%\.claude\settings.json`:

```json
{
  "env": {
    "CLAUDE_CODE_GIT_BASH_PATH": "C:\\Program Files\\Git\\bin\\bash.exe"
  }
}
```

#### Opcja 3: WSL (Windows Subsystem for Linux)

Jeśli wolisz środowisko linuxowe na Windowsie:

```powershell
# W PowerShell jako administrator (tylko przy pierwszej instalacji WSL)
wsl --install -d Ubuntu
```

Następnie w terminalu Ubuntu:

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

> WSL 2 obsługuje sandboxing dla lepszego bezpieczeństwa. WSL 1 — nie.

---

### Alternatywa: instalacja przez npm

Metoda npm jest starsza i wymaga Node.js 18+, ale wciąż działa. Przydatna gdy chcesz przypiąć konkretną wersję.

```bash
npm install -g @anthropic-ai/claude-code
```

> **Nigdy nie używaj `sudo npm install -g`** — powoduje problemy z uprawnieniami i bezpieczeństwem. Jeśli pojawi się błąd EACCES, skonfiguruj katalog npm w przestrzeni użytkownika lub użyj nvm.

---

## Część 2 — Pierwsze uruchomienie i logowanie

Po instalacji przejdź do katalogu swojego projektu i uruchom:

```bash
cd /twoj/projekt
claude
```

Przy pierwszym uruchomieniu:
1. Otwiera się przeglądarka z ekranem logowania Anthropic
2. Logujesz się kontem (Claude Pro / Max / Teams / Enterprise)
3. Autoryzujesz CLI — token jest zapisywany lokalnie w `~/.claude/`
4. Przy kolejnych uruchomieniach logowanie nie jest wymagane

Po zalogowaniu Claude zapyta o preferencje (motyw, ustawienia). Po ich ustawieniu jesteś gotowy.

---

## Część 3 — Plugin JetBrains

> Plugin wymaga wcześniej zainstalowanego CLI (Część 1). Plugin nie działa samodzielnie — jest nakładką na terminal, nie zastępuje CLI.

### Obsługiwane IDE

Plugin działa we wszystkich głównych IDE JetBrains, m.in.:
- **PyCharm** (Community i Professional)
- **WebStorm**
- IntelliJ IDEA
- Android Studio
- PhpStorm, GoLand i inne

### Instalacja pluginu

1. Otwórz IDE (PyCharm lub WebStorm)
2. Wejdź w: `Settings` → `Plugins` → zakładka `Marketplace`
3. Wyszukaj: `Claude Code`
4. Znajdź plugin o nazwie **Claude Code [Beta]** — wydawca: Anthropic
5. Kliknij `Install`
6. **Zrestartuj IDE** (wymagane — plugin nie aktywuje się bez restartu)

Alternatywnie — instalacja przez JetBrains Marketplace w przeglądarce:
- https://plugins.jetbrains.com/plugin/27310-claude-code-beta-

### Pierwsze uruchomienie pluginu

Po restarcie IDE:
1. Otwórz **wbudowany terminal** w IDE (`View` → `Tool Windows` → `Terminal`)
2. Przejdź do katalogu projektu (powinno być automatyczne)
3. Wpisz `claude` i naciśnij Enter

Wszystkie funkcje integracji aktywują się automatycznie gdy Claude Code jest uruchomiony w wbudowanym terminalu IDE.

---

## Część 4 — Co daje plugin vs czysty terminal

| Funkcja | Czysty terminal | Plugin JetBrains |
|---|---|---|
| Podgląd zmian w diff viewerze IDE | ✗ (zmiany w terminalu) | ✅ natywny diff viewer |
| Automatyczny kontekst z aktywnego pliku | ✗ | ✅ Claude widzi co masz otwarte |
| Automatyczny kontekst z zaznaczenia | ✗ | ✅ zaznaczony kod trafia do Claude |
| Błędy lint/syntax automatycznie do Claude | ✗ | ✅ diagnostyka IDE przesyłana na bieżąco |
| Szybkie otwieranie z IDE | ✗ | ✅ `Ctrl+Esc` (Win/Linux) / `Cmd+Esc` (Mac) |
| Wstawianie referencji do pliku | ✗ | ✅ `Alt+Ctrl+K` → `@Plik#L1-99` |

### Skróty pluginu JetBrains

| Skrót | Działanie |
|---|---|
| `Ctrl+Esc` (Win/Linux) / `Cmd+Esc` (Mac) | Otwórz Claude Code z aktualnym kontekstem pliku |
| `Alt+Ctrl+K` (Win/Linux) / `Cmd+Option+K` (Mac) | Wstaw referencję do pliku (`@Plik#L1-99`) |

---

## Część 5 — Rozwiązywanie problemów

### `claude: command not found` po instalacji

Terminal nie odświeżył PATH. Rozwiązania:
1. Otwórz **nowe** okno terminala
2. Jeśli nie pomaga — sprawdź czy PATH został dodany:
   ```bash
   # Dla zsh (domyślny na macOS)
   echo $PATH | grep claude
   # Jeśli brak — dodaj ręcznie do ~/.zshrc lub ~/.bashrc
   ```

### IDE nie widzi komendy `claude`

Częsty problem gdy używasz nvm, asdf lub mise — IDE nie dziedziczy PATH z powłoki.

Rozwiązanie: ustaw pełną ścieżkę do binarki w ustawieniach pluginu:
`Settings` → `Tools` → `Claude Code` → pole `Claude command`

Przykładowe ścieżki:
```
# macOS/Linux — native install
~/.local/bin/claude

# Windows — native install
%USERPROFILE%\.local\bin\claude.exe

# WSL w JetBrains
wsl -d Ubuntu -- bash -lic "claude"
```

### Klawisz ESC nie przerywa operacji w terminalu JetBrains

W ustawieniach IDE wejdź w:
`Settings` → `Tools` → `Terminal`

Znajdź opcję dotyczącą ESC jako znaku sterującego i zastosuj zmiany.

### JetBrains Remote Development

Przy pracy zdalnej (SSH, Dev Containers) plugin musi być zainstalowany na **hoście zdalnym**, nie na lokalnej maszynie:
`Settings` → `Plugin (Host)` → zainstaluj Claude Code

---

## Część 6 — Pierwsze kroki po instalacji

```bash
# 1. Przejdź do projektu
cd /twoj/projekt

# 2. Uruchom Claude Code
claude

# 3. Wygeneruj CLAUDE.md — rób to w każdym nowym projekcie
/init

# 4. Przetestuj integrację
@src/main.py wyjaśnij strukturę tego pliku

# 5. Sprawdź koszty sesji
/cost
```

> Po wygenerowaniu CLAUDE.md — przejrzyj go i uzupełnij o specyfikę projektu, np. które foldery omijać, jakie konwencje stosować, jakiego stacku używasz.
