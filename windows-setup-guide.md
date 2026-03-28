# 🖥️ Przygotowanie Środowiska Developerskiego — Windows 10/11
### Java 21 · Node.js · Docker · Git · Claude Code + ECC
### Instalacja na lokalnym Windows (bez Azure VM)

> Dokument dla 4 osób. Każda przechodzi przez ten sam proces na swoim komputerze.
> Czas wykonania: ~45–75 minut. Rób to D-2 przed hackathon, nie w dniu.

---

## Spis treści

1. [Wymagania wstępne — sprawdź zanim zaczniesz](#1-wymagania-wstępne--sprawdź-zanim-zaczniesz)
2. [Krok 0 — PowerShell jako Administrator](#2-krok-0--powershell-jako-administrator)
3. [Krok 1 — Windows Terminal](#3-krok-1--windows-terminal)
4. [Krok 2 — WSL2 (wymagane dla Docker)](#4-krok-2--wsl2-wymagane-dla-docker)
5. [Krok 3 — Git](#5-krok-3--git)
6. [Krok 4 — Node.js 22 LTS](#6-krok-4--nodejs-22-lts)
7. [Krok 5 — Java 21 JDK](#7-krok-5--java-21-jdk)
8. [Krok 6 — Maven](#8-krok-6--maven)
9. [Krok 7 — Docker](#9-krok-7--docker)
10. [Krok 8 — IDE](#10-krok-8--ide)
11. [Krok 9 — Claude Code CLI](#11-krok-9--claude-code-cli)
12. [Krok 10 — Everything Claude Code (ECC)](#12-krok-10--everything-claude-code-ecc)
13. [Krok 11 — Git konfiguracja globalna](#13-krok-11--git-konfiguracja-globalna)
14. [Krok 12 — Weryfikacja całości](#14-krok-12--weryfikacja-całości)
15. [Skrypt automatyczny — one-shot setup](#15-skrypt-automatyczny--one-shot-setup)
16. [Troubleshooting](#16-troubleshooting)
17. [Checklista końcowa](#17-checklista-końcowa)

---

## 1. Wymagania wstępne — sprawdź zanim zaczniesz

### Minimalne wymagania sprzętowe

| Komponent | Minimum | Zalecane |
|---|---|---|
| RAM | 8 GB | 16 GB |
| Dysk wolne miejsce | 20 GB | 40 GB |
| CPU | 4 rdzenie, 64-bit | 8 rdzeni |
| CPU — Wirtualizacja | **Włączona w BIOS** | — |

### Wersja systemu Windows

```powershell
# Sprawdź wersję Windows
winver
```

Wymagane:
- Windows 10 wersja **2004** (build 19041) lub nowszy
- Windows 11 — każda wersja

Jeśli masz starszego Windows 10 — zaktualizuj przez Windows Update przed kontynuowaniem.

### Sprawdź czy wirtualizacja jest włączona

```powershell
# Uruchom w PowerShell (Admin)
(Get-WmiObject Win32_ComputerSystem).HypervisorPresent
```

Jeśli wynik to `False` — wirtualizacja wyłączona w BIOS. Musisz ją włączyć:

1. Uruchom ponownie komputer
2. Wejdź do BIOS/UEFI (klawisz F2, F10, Del, lub Esc — zależy od producenta)
3. Szukaj: `Intel VT-x`, `AMD-V`, `SVM Mode`, lub `Virtualization Technology`
4. Ustaw na `Enabled`
5. Zapisz i uruchom Windows

> **Bez włączonej wirtualizacji → Docker nie zadziała.**

### Sprawdź czy winget jest dostępny

```powershell
winget --version
```

Jeśli nie ma komendy `winget`:
- Windows 11: zainstaluj `App Installer` ze sklepu Microsoft Store
- Windows 10: pobierz ze strony [github.com/microsoft/winget-cli/releases](https://github.com/microsoft/winget-cli/releases) — plik `.msixbundle`

---

## 2. Krok 0 — PowerShell jako Administrator

**Wszystkie komendy instalacyjne uruchamiasz w PowerShell jako Administrator.**

Jak otworzyć:
```
Metoda 1: Kliknij prawym na Start → "Windows PowerShell (Administrator)"
Metoda 2: Start → wpisz "PowerShell" → Kliknij prawym → "Uruchom jako Administrator"
Metoda 3: Win+X → Windows PowerShell (Administrator)
```

Po otwarciu zobaczysz nagłówek z `Administrator: Windows PowerShell`. Jeśli nie ma słowa "Administrator" — zamknij i otwórz ponownie właściwą metodą.

### Zaktualizuj winget

```powershell
winget upgrade winget --accept-package-agreements --accept-source-agreements
```

Jeśli pojawi się pytanie o zaakceptowanie warunków — wpisz `Y` i Enter.

---

## 3. Krok 1 — Windows Terminal

Znacznie lepszy terminal niż domyślny PowerShell. Wspiera karty, lepsze kolory, wygodniejszą pracę.

```powershell
winget install --id Microsoft.WindowsTerminal -e --silent --accept-package-agreements --accept-source-agreements
```

Po instalacji: zamknij obecny PowerShell i otwórz Windows Terminal jako Administrator. **Wszystkie kolejne kroki rób w Windows Terminal.**

Żeby otworzyć Windows Terminal jako Admin:
```
Start → wpisz "Terminal" → Kliknij prawym → "Uruchom jako Administrator"
```

---

## 4. Krok 2 — WSL2

WSL2 = Windows Subsystem for Linux 2. Docker Desktop używa go jako silnika. Bez WSL2 → Docker nie działa.

### Instalacja

```powershell
wsl --install
```

> **Wymagany restart.** Po restarcie Windows Terminal otworzy się ponownie i dokończy instalację Ubuntu. Zostaniesz poproszony o podanie nazwy użytkownika i hasła dla Ubuntu.

```
Enter new UNIX username: twojaimie
New password: [wpisz hasło — możesz dać proste, tylko do użytku lokalnego]
Retype new password: [powtórz]
```

### Po restarcie — weryfikacja i konfiguracja

Otwórz Windows Terminal jako Administrator i wpisz:

```powershell
# Ustaw WSL2 jako domyślną wersję
wsl --set-default-version 2

# Sprawdź wersję Ubuntu
wsl -l -v
```

Oczekiwany output:
```
  NAME            STATE           VERSION
* Ubuntu           Running         2
```

Jeśli VERSION = 1:
```powershell
wsl --set-version Ubuntu 2
```

### Konfiguracja limitów pamięci WSL2

Utwórz plik `.wslconfig` żeby WSL2 nie zjadł całego RAM-u:

```powershell
# Dla komputera z 16 GB RAM:
$wslConfig = "[wsl2]`nmemory=8GB`nprocessors=4`nswap=2GB`nlocalhostForwarding=true"
$wslConfig | Out-File -FilePath "$env:USERPROFILE\.wslconfig" -Encoding UTF8

# Dla komputera z 8 GB RAM — użyj:
# memory=4GB, processors=2
```

Dostosuj wartości do swojego sprzętu. Zostaw przynajmniej 4 GB dla Windows i IDE.

```powershell
# Zastosuj konfigurację
wsl --shutdown
# Poczekaj 10 sekund
```

---

## 5. Krok 3 — Git

```powershell
winget install --id Git.Git -e --silent --accept-package-agreements --accept-source-agreements
```

**Otwórz nowy terminal** (ważne — stary nie ma jeszcze zaktualizowanego PATH):

```powershell
git --version
# Oczekiwane: git version 2.x.x.windows.x
```

---

## 6. Krok 4 — Node.js 22 LTS

Node.js jest wymagany przez Claude Code CLI. Wersja 22 LTS to aktualny long-term support.

```powershell
winget install --id OpenJS.NodeJS.LTS -e --silent --accept-package-agreements --accept-source-agreements
```

Otwórz nowy terminal:

```powershell
node --version
# Oczekiwane: v22.x.x

npm --version
# Oczekiwane: 10.x.x
```

---

## 7. Krok 5 — Java 21 JDK

Microsoft Build of OpenJDK 21 — darmowy, identyczny z Oracle JDK dla development.

```powershell
winget install --id Microsoft.OpenJDK.21 -e --silent --accept-package-agreements --accept-source-agreements
```

Otwórz nowy terminal:

```powershell
java -version
# Oczekiwane: openjdk version "21.x.x" ... Microsoft

javac -version
# Oczekiwane: javac 21.x.x
```

### Jeśli `java` nie jest widoczna po nowym terminalu

```powershell
# Sprawdź aktualny JAVA_HOME
$env:JAVA_HOME

# Znajdź ścieżkę instalacji
Get-ChildItem "C:\Program Files\Microsoft" -Filter "jdk-21*"

# Ustaw JAVA_HOME (zmień ścieżkę na faktyczną)
[System.Environment]::SetEnvironmentVariable("JAVA_HOME", "C:\Program Files\Microsoft\jdk-21.0.x.x-hotspot", "Machine")
[System.Environment]::SetEnvironmentVariable("PATH", $env:PATH + ";C:\Program Files\Microsoft\jdk-21.0.x.x-hotspot\bin", "Machine")

# Otwórz nowy terminal i sprawdź ponownie
```

---

## 8. Krok 6 — Maven

Maven to build tool dla projektów Java/Spring Boot.

```powershell
winget install --id Apache.Maven -e --silent --accept-package-agreements --accept-source-agreements
```

Otwórz nowy terminal:

```powershell
mvn --version
# Oczekiwane:
# Apache Maven 3.9.x
# Java version: 21.x.x  ← MUSI być Java 21!
```

> **Krytyczne:** Jeśli `mvn --version` pokazuje inną wersję Javy niż 21 — Maven używa złego JDK. Sprawdź i popraw `JAVA_HOME` jak opisano w Kroku 5.

---

## 9. Krok 7 — Docker

### Kwestia licencyjna — Docker Desktop vs Rancher Desktop

**Docker Desktop** wymaga płatnej licencji Business dla firm >250 pracowników lub >$10M przychodów. Sprawdź czy wasza organizacja ma licencję.

**Rancher Desktop** jest bezpłatną alternatywą — te same komendy `docker` i `docker compose`, zero ograniczeń licencyjnych.

---

### Opcja A: Rancher Desktop (bezpłatna, rekomendowana)

```powershell
winget install --id SUSE.RancherDesktop -e --accept-package-agreements --accept-source-agreements
```

Po instalacji:
1. Uruchom Rancher Desktop z menu Start
2. Przy pierwszym uruchomieniu:
   - **Container Engine: dockerd (moby)** ← wybierz to, nie containerd
   - Kubernetes: ustaw **OFF** (nie potrzebujecie)
3. Poczekaj aż ikona w zasobniku systemowym zmieni kolor na zielony (~3–5 minut)

---

### Opcja B: Docker Desktop (jeśli macie licencję)

```powershell
winget install --id Docker.DockerDesktop -e --accept-package-agreements --accept-source-agreements
```

Po instalacji uruchom Docker Desktop i w konfiguracji:
- Use WSL 2 instead of Hyper-V: **TAK**
- Zaloguj się do Docker Hub jeśli wymagane przez licencję

---

### Weryfikacja Docker (obie opcje)

Otwórz nowy terminal:

```powershell
docker --version
# Oczekiwane: Docker version 27.x.x

docker compose version
# Oczekiwane: Docker Compose version v2.x.x

docker run hello-world
# Oczekiwane: "Hello from Docker!" — bez błędów
```

---

## 10. Krok 8 — IDE

### IntelliJ IDEA Community (dla BE devów i DB deva)

```powershell
winget install --id JetBrains.IntelliJIDEA.Community -e --silent --accept-package-agreements --accept-source-agreements
```

Po instalacji uruchom IntelliJ i skonfiguruj:

**Motyw:** File → Settings → Appearance → Theme: Darcula (ciemny motyw przy długich sesjach)

**Plugin Lombok** (wymagany):
```
File → Settings → Plugins → Marketplace → szukaj "Lombok" → Install → Restart
```

**JDK w projekcie:**
```
File → Project Structure → Platform Settings → SDKs → + → Add JDK
→ wskaż: C:\Program Files\Microsoft\jdk-21.x.x.x-hotspot
```

---

### VS Code (dla FE i QA, opcjonalnie dla wszystkich)

```powershell
winget install --id Microsoft.VisualStudioCode -e --silent --accept-package-agreements --accept-source-agreements
```

Otwórz VS Code i zainstaluj rozszerzenia przez `Ctrl+P`:

```
# Dla wszystkich:
ext install ms-vscode.vscode-typescript-next
ext install dbaeumer.vscode-eslint
ext install esbenp.prettier-vscode
ext install ms-azuretools.vscode-docker

# Dla Java (BE/DB):
ext install redhat.java
ext install vscjava.vscode-maven
ext install vscjava.vscode-java-test

# Dla QA/E2E:
ext install ms-playwright.playwright
```

---

## 11. Krok 9 — Claude Code CLI

### Instalacja

```powershell
npm install -g @anthropic-ai/claude-code
```

Weryfikacja (nowy terminal):

```powershell
claude --version
# Oczekiwane: Claude Code v2.x.x
```

### Logowanie do Claude Max

```powershell
claude login
```

Otworzy się przeglądarka. Zaloguj się na konto Anthropic z aktywną subskrypcją **Claude Max**.

> Każda osoba w zespole potrzebuje własnego konta Claude Max — licencja jest per-użytkownik.

Weryfikacja:

```powershell
mkdir C:\projects\test-claude
cd C:\projects\test-claude
claude
# Powinno uruchomić sesję bez błędów
# Wpisz: hello
# /exit żeby zamknąć
```

---

## 12. Krok 10 — Everything Claude Code (ECC)

### A: Sklonuj repo

```powershell
cd $env:USERPROFILE
git clone https://github.com/affaan-m/everything-claude-code.git
cd everything-claude-code
npm install
```

### B: Uruchom instalator Windows

```powershell
# Jeśli pojawi się błąd ExecutionPolicy:
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

# Uruchom instalator dla Javy
.\install.ps1 java
```

### C: Kopiuj skills i agentów (manualnie — pewna metoda)

```powershell
# Stwórz katalogi
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\skills"
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\agents"
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\commands"
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\rules"

# Rules (zawsze aktywne — OBOWIĄZKOWE)
Copy-Item -Recurse -Force "$env:USERPROFILE\everything-claude-code\rules\common\*" "$env:USERPROFILE\.claude\rules\"

# Skills Spring Boot
$skills = @(
    "springboot-patterns","springboot-tdd","springboot-security",
    "springboot-verification","docker-patterns","api-design",
    "database-migrations","deployment-patterns","e2e-testing",
    "tdd-workflow","security-review","backend-patterns"
)
foreach ($skill in $skills) {
    $src = "$env:USERPROFILE\everything-claude-code\skills\$skill"
    if (Test-Path $src) {
        Copy-Item -Recurse -Force $src "$env:USERPROFILE\.claude\skills\"
        Write-Host "✓ $skill"
    }
}

# Agenci Java
$agents = @(
    "java-reviewer.md","java-build-resolver.md","database-reviewer.md",
    "tdd-guide.md","planner.md","architect.md","code-reviewer.md",
    "security-reviewer.md","e2e-runner.md","refactor-cleaner.md"
)
foreach ($agent in $agents) {
    $src = "$env:USERPROFILE\everything-claude-code\agents\$agent"
    if (Test-Path $src) {
        Copy-Item -Force $src "$env:USERPROFILE\.claude\agents\"
        Write-Host "✓ $agent"
    }
}

# Commands
Copy-Item -Recurse -Force "$env:USERPROFILE\everything-claude-code\commands\*" "$env:USERPROFILE\.claude\commands\"
Write-Host "✓ Commands copied"
```

### D: Konfiguracja settings.json

```powershell
$settings = @{
    model = "sonnet"
    env   = @{
        MAX_THINKING_TOKENS             = "10000"
        CLAUDE_AUTOCOMPACT_PCT_OVERRIDE = "50"
        CLAUDE_CODE_SUBAGENT_MODEL      = "haiku"
    }
} | ConvertTo-Json -Depth 3

$settings | Out-File -FilePath "$env:USERPROFILE\.claude\settings.json" -Encoding UTF8
Write-Host "✓ Settings saved"

# Weryfikacja:
Get-Content "$env:USERPROFILE\.claude\settings.json"
```

### E: Zainstaluj plugin ECC w Claude Code

```powershell
mkdir C:\projects\hackathon-setup-test -Force
cd C:\projects\hackathon-setup-test
claude
```

W sesji Claude Code:
```
/plugin marketplace add affaan-m/everything-claude-code
/plugin install everything-claude-code@everything-claude-code
/plugin list everything-claude-code@everything-claude-code
/exit
```

---

## 13. Krok 11 — Git konfiguracja globalna

Każda osoba ustawia indywidualnie:

```powershell
git config --global user.name "Imię Nazwisko"
git config --global user.email "email@firma.pl"
git config --global core.autocrlf input
git config --global core.editor "code --wait"
git config --global init.defaultBranch main
git config --global pull.rebase true

# Weryfikacja
git config --list --global
```

> `core.autocrlf input` jest krytyczne w zespole mieszanym Windows/Linux — zapobiega problemom z końcami linii w diff-ach.

### SSH klucze (jeśli używacie GitHub/GitLab przez SSH)

```powershell
ssh-keygen -t ed25519 -C "email@firma.pl"
# Enter na wszystkie pytania (domyślne ścieżki)

# Wyświetl klucz publiczny i dodaj do GitHub/GitLab
Get-Content "$env:USERPROFILE\.ssh\id_ed25519.pub"
```

### Sklonuj shared repo

```powershell
mkdir C:\projects -Force
cd C:\projects
git clone git@github.com:wasza-org/hackathon-app.git
cd hackathon-app
git branch -a
```

---

## 14. Krok 12 — Weryfikacja całości

Uruchom ten skrypt weryfikacyjny — każda pozycja musi dać zielone ✓:

```powershell
$errors = 0

function Check($name, $cmd, $pattern) {
    try {
        $result = Invoke-Expression $cmd 2>&1 | Out-String
        if ($result -match $pattern) {
            Write-Host "  ✓ $name" -ForegroundColor Green
        } else {
            Write-Host "  ✗ $name  →  $($result.Trim())" -ForegroundColor Red
            $script:errors++
        }
    } catch {
        Write-Host "  ✗ $name  →  ERROR: $_" -ForegroundColor Red
        $script:errors++
    }
}

Write-Host "`n══════════════════════════════════════" -ForegroundColor Cyan
Write-Host "  HACKATHON SETUP — WERYFIKACJA" -ForegroundColor Cyan
Write-Host "══════════════════════════════════════`n" -ForegroundColor Cyan

Write-Host "[ Narzędzia deweloperskie ]" -ForegroundColor Yellow
Check "Git"            "git --version"          "git version"
Check "Node.js 22"    "node --version"          "v2[2-9]\.|v[3-9][0-9]\."
Check "npm"           "npm --version"           "[0-9]+\.[0-9]+"
Check "Java 21"       "java -version 2>&1"      "21\."
Check "javac 21"      "javac -version 2>&1"     "21\."
Check "Maven 3.9"     "mvn --version"           "Apache Maven 3\."
Check "Maven → Java 21" "mvn --version"         "Java version: 21"

Write-Host "`n[ Docker ]" -ForegroundColor Yellow
Check "Docker daemon"     "docker --version"          "Docker version"
Check "Docker Compose"    "docker compose version"    "Docker Compose"
Check "Docker run test"   "docker run --rm hello-world 2>&1" "Hello from Docker"

Write-Host "`n[ Claude Code ]" -ForegroundColor Yellow
Check "Claude Code CLI" "claude --version" "Claude Code"

$claudeDir = "$env:USERPROFILE\.claude"
if (Test-Path "$claudeDir\settings.json") {
    Write-Host "  ✓ settings.json" -ForegroundColor Green
} else {
    Write-Host "  ✗ settings.json BRAK" -ForegroundColor Red; $errors++
}

$cmdCount = (Get-ChildItem "$claudeDir\commands" -ErrorAction SilentlyContinue).Count
if ($cmdCount -gt 10) {
    Write-Host "  ✓ ECC commands ($cmdCount plików)" -ForegroundColor Green
} else {
    Write-Host "  ✗ ECC commands brak lub niekompletne ($cmdCount)" -ForegroundColor Red; $errors++
}

$agentCount = (Get-ChildItem "$claudeDir\agents" -ErrorAction SilentlyContinue).Count
if ($agentCount -gt 5) {
    Write-Host "  ✓ ECC agents ($agentCount plików)" -ForegroundColor Green
} else {
    Write-Host "  ✗ ECC agents brak ($agentCount)" -ForegroundColor Red; $errors++
}

Write-Host "`n[ WSL2 ]" -ForegroundColor Yellow
Check "WSL2 zainstalowany" "wsl -l -v" "2"

Write-Host "`n══════════════════════════════════════" -ForegroundColor Cyan
if ($errors -eq 0) {
    Write-Host "  ✅ WSZYSTKO OK — gotowy na hackathon!" -ForegroundColor Green
} else {
    Write-Host "  ❌ $errors błąd(ów) — napraw przed hackathon" -ForegroundColor Red
}
Write-Host "══════════════════════════════════════`n" -ForegroundColor Cyan
```

---

## 15. Skrypt automatyczny — one-shot setup

Zapisz jako `C:\hackathon-setup.ps1` i uruchom jako Admin:

```powershell
# ================================================================
# HACKATHON WINDOWS SETUP SCRIPT — Windows 10/11
# Uruchom jako Administrator w Windows Terminal
# Po Kroku WSL2 wymagany jest restart!
# ================================================================

param([switch]$SkipWSL, [switch]$SkipRestart)

Set-ExecutionPolicy RemoteSigned -Scope CurrentUser -Force

Write-Host "🚀 Hackathon Setup — Windows 10/11" -ForegroundColor Cyan

# --- winget update ---
Write-Host "`n[0] winget..." -ForegroundColor Yellow
winget upgrade winget --accept-package-agreements --accept-source-agreements 2>$null

# --- WSL2 ---
if (-not $SkipWSL) {
    Write-Host "`n[1] WSL2..." -ForegroundColor Yellow
    wsl --install --no-distribution
    Write-Host "`n⚠  RESTART WYMAGANY!" -ForegroundColor Red
    Write-Host "Po restarcie uruchom: .\hackathon-setup.ps1 -SkipWSL" -ForegroundColor Yellow
    if (-not $SkipRestart) { Start-Sleep 10; Restart-Computer -Force }
    return
}

# --- WSL2 post-restart ---
Write-Host "`n[2] WSL2 konfiguracja..." -ForegroundColor Yellow
wsl --set-default-version 2
wsl --install -d Ubuntu-22.04

$wslCfg = "[wsl2]`nmemory=8GB`nprocessors=4`nswap=2GB`nlocalhostForwarding=true"
$wslCfg | Out-File "$env:USERPROFILE\.wslconfig" -Encoding UTF8
Write-Host "✓ WSL2" -ForegroundColor Green

# --- Dev tools ---
Write-Host "`n[3] Narzędzia..." -ForegroundColor Yellow
$pkgs = @(
    @{id="Microsoft.WindowsTerminal"; n="Windows Terminal"},
    @{id="Git.Git";                   n="Git"},
    @{id="OpenJS.NodeJS.LTS";         n="Node.js 22 LTS"},
    @{id="Microsoft.OpenJDK.21";      n="Java 21 JDK"},
    @{id="Apache.Maven";              n="Maven"},
    @{id="Microsoft.VisualStudioCode";n="VS Code"},
    @{id="JetBrains.IntelliJIDEA.Community"; n="IntelliJ IDEA"}
)
foreach ($p in $pkgs) {
    Write-Host "  Installing $($p.n)..." -NoNewline
    winget install --id $p.id -e --silent --accept-package-agreements --accept-source-agreements 2>$null
    Write-Host " ✓" -ForegroundColor Green
}

# --- Rancher Desktop (Docker) ---
Write-Host "`n[4] Rancher Desktop (Docker)..." -ForegroundColor Yellow
winget install --id SUSE.RancherDesktop -e --accept-package-agreements --accept-source-agreements
Write-Host "✓ Rancher Desktop — uruchom ręcznie i wybierz dockerd" -ForegroundColor Green

# --- PATH refresh ---
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")

# --- Claude Code ---
Write-Host "`n[5] Claude Code + ECC..." -ForegroundColor Yellow
npm install -g @anthropic-ai/claude-code
Write-Host "✓ Claude Code CLI" -ForegroundColor Green

cd $env:USERPROFILE
if (-not (Test-Path "everything-claude-code")) {
    git clone https://github.com/affaan-m/everything-claude-code.git
}
cd everything-claude-code
npm install

$cd = "$env:USERPROFILE\.claude"
@("skills","agents","commands","rules") | % { New-Item -Force -ItemType Directory -Path "$cd\$_" | Out-Null }

Copy-Item -Recurse -Force "rules\common\*" "$cd\rules\" 2>$null

@("springboot-patterns","springboot-tdd","springboot-security","springboot-verification",
  "docker-patterns","api-design","database-migrations","e2e-testing","tdd-workflow","backend-patterns") | % {
    if (Test-Path "skills\$_") { Copy-Item -Recurse -Force "skills\$_" "$cd\skills\" 2>$null }
}

@("java-reviewer.md","java-build-resolver.md","database-reviewer.md","tdd-guide.md",
  "planner.md","architect.md","code-reviewer.md","security-reviewer.md","e2e-runner.md") | % {
    if (Test-Path "agents\$_") { Copy-Item -Force "agents\$_" "$cd\agents\" 2>$null }
}

Copy-Item -Recurse -Force "commands\*" "$cd\commands\" 2>$null

@{ model="sonnet"; env=@{MAX_THINKING_TOKENS="10000";CLAUDE_AUTOCOMPACT_PCT_OVERRIDE="50";CLAUDE_CODE_SUBAGENT_MODEL="haiku"} } |
    ConvertTo-Json -Depth 3 | Out-File "$cd\settings.json" -Encoding UTF8

Write-Host "✓ ECC zainstalowane" -ForegroundColor Green

Write-Host "`n════════════════════════════════" -ForegroundColor Cyan
Write-Host "✅ SETUP GOTOWY" -ForegroundColor Green
Write-Host "════════════════════════════════" -ForegroundColor Cyan
Write-Host "Co teraz:"
Write-Host "1. Uruchom Rancher Desktop → wybierz dockerd (moby)"
Write-Host "2. Nowy terminal → claude login"
Write-Host "3. cd C:\projects\hackathon-app && claude"
Write-Host "4. /plugin marketplace add affaan-m/everything-claude-code"
Write-Host "5. /plugin install everything-claude-code@everything-claude-code"
```

Uruchamianie:
```powershell
# Krok 1 (przed restartem):
.\hackathon-setup.ps1

# Po restarcie (krok 2+):
.\hackathon-setup.ps1 -SkipWSL
```

---

## 16. Troubleshooting

### `wsl --install` kończy się błędem "feature not installed"

Włącz WSL ręcznie:
```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
# Restart, potem:
wsl --set-default-version 2
wsl --install -d Ubuntu-22.04
```

---

### Docker: "Hardware assisted virtualization and data execution protection must be enabled"

Wirtualizacja wyłączona w BIOS. Sprawdź sekcję 1 — włącz Intel VT-x lub AMD-V w ustawieniach BIOS.

---

### `mvn --version` pokazuje Java 11 lub Java 17 zamiast 21

```powershell
# Znajdź ścieżkę Java 21
Get-ChildItem "C:\Program Files\Microsoft" -Filter "jdk-21*"

# Wymuś JAVA_HOME
[System.Environment]::SetEnvironmentVariable("JAVA_HOME","C:\Program Files\Microsoft\jdk-21.0.X.X-hotspot","Machine")

# Otwórz nowy terminal i sprawdź
mvn --version
```

---

### `winget` nie jest rozpoznane na Windows 10

```powershell
# Zainstaluj App Installer z Microsoft Store
# lub pobierz ręcznie:
$url = "https://aka.ms/getwinget"
Invoke-WebRequest -Uri $url -OutFile "$env:TEMP\AppInstaller.msixbundle"
Add-AppxPackage "$env:TEMP\AppInstaller.msixbundle"
```

---

### `docker run hello-world` zawiesza się

```powershell
# Sprawdź czy Rancher Desktop/Docker Desktop jest uruchomiony (ikona w zasobniku)
# Jeśli zielona ale docker nie odpowiada:
wsl --shutdown
# Poczekaj 15 sekund — Docker/Rancher sam się zrestartuje
```

---

### `claude login` nie otwiera przeglądarki

```powershell
claude login
# Skopiuj URL który pojawi się w terminalu
# Otwórz URL ręcznie w przeglądarce
# Zaloguj się → skopiuj kod autoryzacyjny → wklej z powrotem w terminal
```

---

### ECC `/plan` nie działa w Claude Code

```powershell
# Sprawdź czy commands są zainstalowane
ls "$env:USERPROFILE\.claude\commands\"

# Jeśli puste — wróć do Kroku 10 Punkt C
# Jeśli pliki są ale /plan nie działa — zainstaluj plugin:
# (w sesji claude)
# /plugin marketplace add affaan-m/everything-claude-code
# /plugin install everything-claude-code@everything-claude-code
```

---

## 17. Checklista końcowa

Każda osoba potwierdza przed hackathon:

```
SYSTEM:
[ ] Windows 10 build 19041+ lub Windows 11
[ ] Wirtualizacja włączona w BIOS (docker run hello-world działa)
[ ] winget dostępny

TERMINAL:
[ ] Windows Terminal zainstalowany i używany
[ ] WSL2 działa: wsl -l -v pokazuje VERSION 2

NARZĘDZIA:
[ ] git --version          → git version 2.x.x
[ ] node --version         → v22.x.x
[ ] npm --version          → 10.x.x
[ ] java -version          → openjdk 21.x.x Microsoft
[ ] javac -version         → javac 21.x.x
[ ] mvn --version          → Maven 3.9.x + Java version: 21
[ ] docker --version       → Docker version 27.x.x
[ ] docker compose version → v2.x.x
[ ] docker run hello-world → "Hello from Docker!" bez błędów

CLAUDE:
[ ] claude --version       → Claude Code v2.x.x
[ ] claude login           → zalogowany (konto z Claude Max)
[ ] ~/.claude/settings.json — model: sonnet
[ ] ~/.claude/commands/    → >10 plików (plan.md, tdd.md etc.)
[ ] ~/.claude/agents/      → >5 plików (java-reviewer.md etc.)
[ ] Plugin ECC aktywny     → /plan działa w sesji claude

GIT:
[ ] git config user.name ustawione
[ ] git config user.email ustawione
[ ] Shared repo sklonowane do C:\projects\hackathon-app

IDE:
[ ] IntelliJ IDEA uruchomia się + plugin Lombok zainstalowany
[ ] VS Code uruchomia się
```

**Gdy wszystkie checkboxy odhaczone — jesteś gotowy do hackathonu. 🚀**

---

## Podsumowanie narzędzi i wersji

| Narzędzie | Wersja | Przeznaczenie |
|---|---|---|
| Windows 10/11 | 21H2+ / dowolna | System operacyjny |
| WSL2 + Ubuntu 22.04 | 2.x | Silnik Dockera |
| Windows Terminal | najnowszy | Wygodna praca w terminalu |
| Git | 2.x.x | Kontrola wersji |
| Node.js LTS | 22.x.x | Wymagany przez Claude Code |
| npm | 10.x.x | Package manager |
| Microsoft OpenJDK | 21.x.x | Java runtime + kompilator |
| Apache Maven | 3.9.x | Build system Spring Boot |
| Rancher Desktop | najnowszy | Docker Engine (bezpłatny) |
| Docker Compose | v2.x.x | Orchestracja kontenerów |
| IntelliJ IDEA Community | najnowszy | IDE Java |
| VS Code | najnowszy | Edytor FE/QA |
| Claude Code CLI | v2.x.x | Agent AI |
| Everything Claude Code | v1.9.0 | Plugin + skills + agenci |

---

*Windows 10/11 Native Setup · Hackathon 2026 · Java Spring + SQLite + React + Docker*
