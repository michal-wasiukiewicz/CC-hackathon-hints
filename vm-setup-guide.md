# 🖥️ Przygotowanie VM — Hackathon Setup Guide
### Windows na Azure · Java 21 · Node.js · Docker · Git · Claude Code + ECC

> Dokument dla 4 osób. Każda przechodzi przez ten sam proces na swojej VM.
> Czas wykonania: ~45-60 minut na VM. Rób to D-2 przed hackathon, nie w dniu.

---

## Spis treści

1. [Azure VM — wybór rozmiaru i tworzenie](#1-azure-vm--wybór-rozmiaru-i-tworzenie)
2. [Krok 0 — pierwsze uruchomienie VM po RDP](#2-krok-0--pierwsze-uruchomienie-vm-po-rdp)
3. [Krok 1 — WSL2 (wymagane dla Docker)](#3-krok-1--wsl2-wymagane-dla-docker)
4. [Krok 2 — Narzędzia developerskie (Git, Node, Java, Maven)](#4-krok-2--narzędzia-developerskie-git-node-java-maven)
5. [Krok 3 — Docker](#5-krok-3--docker)
6. [Krok 4 — IDE i edytory](#6-krok-4--ide-i-edytory)
7. [Krok 5 — Claude Code CLI](#7-krok-5--claude-code-cli)
8. [Krok 6 — Everything Claude Code (ECC)](#8-krok-6--everything-claude-code-ecc)
9. [Krok 7 — Git konfiguracja i shared repo](#9-krok-7--git-konfiguracja-i-shared-repo)
10. [Krok 8 — Weryfikacja całości](#10-krok-8--weryfikacja-całości)
11. [Skrypt automatyczny — one-shot setup](#11-skrypt-automatyczny--one-shot-setup)
12. [Troubleshooting](#12-troubleshooting)
13. [Checklista dla każdej osoby](#13-checklista-dla-każdej-osoby)

---

## 1. Azure VM — wybór rozmiaru i tworzenie

### ⚠️ KRYTYCZNE: rozmiar VM a Docker

Docker Desktop na Windows wymaga **nested virtualization** (wirtualizacja w wirtualizacji). W Azure obsługują to tylko konkretne serie VM. Zły rozmiar = Docker nie ruszy.

**Nie działa (brak nested virtualization):**
- Seria B (B2s, B4ms, B8ms) — popularne, tanie, ale NIE
- Seria A (A2, A4)
- Seria F (F4s)

**Działa (wspierają nested virtualization):**
- **Standard_D4s_v5** — 4 vCPU, 16 GB RAM — **REKOMENDOWANY minimum**
- Standard_D8s_v5 — 8 vCPU, 32 GB RAM — komfortowy
- Standard_D4s_v4 — 4 vCPU, 16 GB RAM — alternatywa
- Standard_E4s_v5 — 4 vCPU, 32 GB RAM — więcej RAM-u

> **Dla hackathonu:** Standard_D4s_v5 wystarczy dla jednej osoby z całym stackiem. Jeśli budżet pozwala — D8s_v5 da wam zapas.

### Tworzenie VM w Azure Portal

1. Otwórz [portal.azure.com](https://portal.azure.com)
2. `Utwórz zasób` → `Maszyna wirtualna`
3. Wypełnij:

```
Podstawy:
  Subskrypcja: [wasza]
  Grupa zasobów: hackathon-rg (stwórz nową)
  Nazwa VM: hackathon-dev-01  (i -02, -03, -04 dla kolejnych)
  Region: West Europe lub North Europe (blisko Polski)
  Obraz: Windows Server 2022 Datacenter - x64 Gen2
  Rozmiar: Standard_D4s_v5  ← WAŻNE
  
Konto administratora:
  Nazwa: hackuser
  Hasło: [minimum 12 znaków, cyfry + duże + małe + specjalne]
  
Reguły portów wejściowych:
  Zezwól na wybrane porty: TAK
  Wybierz porty: RDP (3389)
```

4. Dysk OS: Premium SSD, 128 GB minimum (zmień jeśli domyślnie mniej)
5. Sieć: zostaw domyślne
6. Kliknij `Przejrzyj i utwórz` → `Utwórz`

### Połączenie RDP

```
Windows:
  Start → mstsc → wpisz IP maszyny → Połącz
  Login: hackuser
  Hasło: [ustawione przy tworzeniu]

macOS:
  Zainstaluj "Microsoft Remote Desktop" z App Store
  Dodaj nowe połączenie → IP maszyny

Linux:
  sudo apt install remmina
  Nowe połączenie RDP → IP maszyny
```

---

## 2. Krok 0 — pierwsze uruchomienie VM po RDP

Po połączeniu przez RDP, zanim zaczniesz instalować cokolwiek:

### Otwórz PowerShell jako Administrator

```
Kliknij prawym na Start → "Windows PowerShell (Administrator)"
lub:
Kliknij Start → wpisz "PowerShell" → Kliknij prawym → "Uruchom jako Administrator"
```

**Wszystkie komendy w tym dokumencie uruchamiasz w PowerShell (Admin).**

### Zaktualizuj Windows + winget

```powershell
# Zaktualizuj winget do najnowszej wersji
winget upgrade winget

# Zaakceptuj warunki winget przy pierwszym użyciu (może wyskoczyć prompt)
winget list
# Jeśli pyta o akceptację warunków — wpisz Y i Enter
```

### Włącz Developer Mode (opcjonalnie, ułatwia pracę)

```powershell
# Włącz tryb developerski (nie wymagany, ale przydatny)
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\AppModelUnlock" /t REG_DWORD /f /v "AllowDevelopmentWithoutDevLicense" /d "1"
```

### Zainstaluj Windows Terminal (znacznie lepszy niż domyślny)

```powershell
winget install --id Microsoft.WindowsTerminal --source winget -e --silent --accept-package-agreements --accept-source-agreements
```

> Po instalacji: otwórz Windows Terminal zamiast starego PowerShell. Wszystkie kolejne komendy uruchamiaj tam jako Admin.

---

## 3. Krok 1 — WSL2 (wymagane dla Docker)

WSL2 = Windows Subsystem for Linux 2. Docker Desktop na Windows używa WSL2 jako silnika. Bez WSL2 → Docker nie działa.

### Instalacja WSL2

```powershell
# Włącz WSL i pobierz kernel linuxa (jednolinijkowiec)
wsl --install

# Jeśli powyższe nie zadziała (starszy Windows):
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

> **Restart wymagany.** Po restarcie wróć do PowerShell (Admin) i kontynuuj.

```powershell
# Po restarcie — ustaw WSL2 jako domyślną wersję
wsl --set-default-version 2

# Zainstaluj Ubuntu (potrzebne jako dystrybucja dla WSL2)
wsl --install -d Ubuntu-22.04
```

Przy pierwszym uruchomieniu Ubuntu zostaniesz poproszony o:
```
Enter new UNIX username: hackuser
New password: [ustaw proste hasło, używasz go tylko lokalnie]
Retype new password: [powtórz]
```

### Weryfikacja WSL2

```powershell
# Sprawdź czy Ubuntu jest w wersji 2
wsl -l -v
```

Oczekiwany output:
```
  NAME            STATE           VERSION
* Ubuntu-22.04    Running         2
```

Jeśli VERSION = 1, wykonaj:
```powershell
wsl --set-version Ubuntu-22.04 2
```

---

## 4. Krok 2 — Narzędzia developerskie

Wszystko przez `winget` — jeden package manager, zero klikania przez instalatory.

### Git

```powershell
winget install --id Git.Git --source winget -e --silent --accept-package-agreements --accept-source-agreements
```

Weryfikacja (otwórz NOWY terminal po instalacji):
```powershell
git --version
# Oczekiwane: git version 2.x.x.windows.x
```

---

### Node.js 22 LTS

Node.js jest wymagany przez Claude Code CLI. Wersja 22 LTS to aktualny long-term support.

```powershell
winget install --id OpenJS.NodeJS.LTS --source winget -e --silent --accept-package-agreements --accept-source-agreements
```

Weryfikacja (NOWY terminal):
```powershell
node --version
# Oczekiwane: v22.x.x

npm --version
# Oczekiwane: 10.x.x
```

---

### Java 21 JDK (Microsoft OpenJDK)

Microsoft Build of OpenJDK 21 — darmowy, no-cost, długoterminowe wsparcie. Identyczny z Oracle JDK dla development.

```powershell
winget install --id Microsoft.OpenJDK.21 --source winget -e --silent --accept-package-agreements --accept-source-agreements
```

Weryfikacja (NOWY terminal):
```powershell
java -version
# Oczekiwane: openjdk version "21.x.x" ... Microsoft

javac -version
# Oczekiwane: javac 21.x.x
```

Jeśli java nie jest widoczna po nowym terminalu:
```powershell
# Sprawdź czy JAVA_HOME jest ustawione
$env:JAVA_HOME

# Jeśli puste — ustaw ręcznie (zmień ścieżkę na faktyczną z instalacji)
[System.Environment]::SetEnvironmentVariable("JAVA_HOME", "C:\Program Files\Microsoft\jdk-21.0.x.x-hotspot", "Machine")
[System.Environment]::SetEnvironmentVariable("PATH", $env:PATH + ";C:\Program Files\Microsoft\jdk-21.0.x.x-hotspot\bin", "Machine")

# Zamknij i otwórz nowy terminal — weryfikuj ponownie
```

---

### Maven 3.9

Maven to build tool dla projektów Java/Spring Boot.

```powershell
winget install --id Apache.Maven --source winget -e --silent --accept-package-agreements --accept-source-agreements
```

Weryfikacja (NOWY terminal):
```powershell
mvn --version
# Oczekiwane: Apache Maven 3.9.x
# Oczekiwane: Java version: 21.x.x (musi pasować do zainstalowanej Javy!)
```

> **Ważne:** Linia `Java version` w output `mvn --version` musi pokazywać Java 21. Jeśli pokazuje inną — Maven używa złego JDK. Sprawdź JAVA_HOME.

---

## 5. Krok 3 — Docker

### ⚠️ Kwestia licencji Docker Desktop na Azure VM

Docker Desktop wymaga licencji **Docker Business** gdy używany wewnątrz VM (Azure, VMware etc.) zgodnie z ich Terms of Service dla środowisk enterprise.

**Opcje dla hackathonu:**

**Opcja A: Rancher Desktop (zalecana dla hackathonu)**
Darmowa alternatywa Docker Desktop. Używa tego samego Docker Engine. Komendy `docker` i `docker compose` działają identycznie. Brak ograniczeń licencyjnych.

**Opcja B: Docker Desktop z licencją**
Jeśli wasza organizacja ma Docker Business — użyjcie Docker Desktop. Poniżej instrukcja dla obu.

---

### Opcja A: Rancher Desktop (FREE, rekomendowane)

```powershell
winget install --id SUSE.RancherDesktop --source winget -e --accept-package-agreements --accept-source-agreements
```

Po instalacji:
1. Uruchom Rancher Desktop z menu Start
2. Przy pierwszym uruchomieniu wybierz:
   - **Container Engine: dockerd (moby)** ← ważne, nie containerd
   - Kubernetes: możesz wyłączyć (Enable Kubernetes: OFF) — nie potrzebujecie
3. Poczekaj aż status w tray będzie zielony (~3-5 minut)

Weryfikacja:
```powershell
docker --version
# Oczekiwane: Docker version 27.x.x

docker compose version
# Oczekiwane: Docker Compose version v2.x.x

docker run hello-world
# Oczekiwane: "Hello from Docker!" bez błędów
```

---

### Opcja B: Docker Desktop (jeśli macie licencję Business)

```powershell
winget install --id Docker.DockerDesktop --source winget -e --accept-package-agreements --accept-source-agreements
```

Po instalacji:
1. Uruchom Docker Desktop
2. Konfiguracja przy pierwszym uruchomieniu:
   - Use WSL 2 instead of Hyper-V: **TAK** (zaznaczone)
   - Add shortcut to desktop: opcjonalne
3. Sign in z Docker ID (jeśli macie konto z licencją Business)
4. Poczekaj aż tray icon zmieni kolor na zielony

Weryfikacja (jak w Opcji A).

---

### Konfiguracja Docker — limity zasobów WSL2

Na Azure D4s_v5 (16GB RAM) warto ograniczyć ile Docker może zjeść:

```powershell
# Stwórz plik konfiguracyjny WSL2
$wslConfig = @"
[wsl2]
memory=10GB
processors=3
swap=2GB
localhostForwarding=true
"@

$wslConfig | Out-File -FilePath "$env:USERPROFILE\.wslconfig" -Encoding UTF8

# Restart WSL2 żeby zaaplikować
wsl --shutdown
# Poczekaj 10 sekund, Docker/Rancher Desktop sam się zrestartuje
```

> Zostawia 6GB dla Windows i IDE. Dostosujcie jeśli macie inny rozmiar VM.

---

## 6. Krok 4 — IDE i edytory

### IntelliJ IDEA Community (dla BE devów i DB deva)

```powershell
winget install --id JetBrains.IntelliJIDEA.Community --source winget -e --silent --accept-package-agreements --accept-source-agreements
```

**Pierwsze uruchomienie IntelliJ:**
1. Start → IntelliJ IDEA Community Edition
2. Accept UI License
3. Customize: Dark theme (Darcula) — polecane przy długich sesjach
4. Plugins: zainstaluj `Lombok` (Settings → Plugins → szukaj Lombok → Install)
5. Plugins: opcjonalnie `Docker` plugin (użyteczny do docker-compose)

**Konfiguracja JDK w IntelliJ:**
```
File → Project Structure → Platform Settings → SDKs
→ kliknij + → Add JDK
→ wskaż: C:\Program Files\Microsoft\jdk-21.x.x.x-hotspot
→ OK
```

---

### VS Code (dla wszystkich, szczególnie FE i QA)

```powershell
winget install --id Microsoft.VisualStudioCode --source winget -e --silent --accept-package-agreements --accept-source-agreements
```

**Rozszerzenia VS Code — zainstaluj po uruchomieniu:**

Otwórz VS Code → `Ctrl+P` → wpisz każdą komendę po kolei:

```
# Dla wszystkich:
ext install ms-vscode.vscode-typescript-next
ext install dbaeumer.vscode-eslint
ext install esbenp.prettier-vscode
ext install ms-azuretools.vscode-docker

# Dla BE devów (Java):
ext install redhat.java
ext install vscjava.vscode-maven
ext install vscjava.vscode-java-test

# Dla QA:
ext install ms-playwright.playwright
```

---

## 7. Krok 5 — Claude Code CLI

### Instalacja

```powershell
# Zainstaluj globalnie przez npm
npm install -g @anthropic-ai/claude-code

# Weryfikacja
claude --version
# Oczekiwane: Claude Code v2.x.x
```

### Logowanie do Claude Max

```powershell
claude login
```

To otworzy przeglądarkę. Zaloguj się na konto Anthropic z aktywną subskrypcją **Claude Max**.

> **Każda osoba w zespole musi mieć własne konto Claude Max.** Claude Max jest per-użytkownik, nie per-zespół.

Weryfikacja logowania:
```powershell
# Uruchom w katalogu testowym
mkdir C:\test-claude
cd C:\test-claude
claude
# Powinno uruchomić sesję bez błędów
# Wpisz: hello
# Ctrl+D lub /exit żeby zamknąć
```

---

## 8. Krok 6 — Everything Claude Code (ECC)

### Krok A: Sklonuj repo ECC

```powershell
# Sklonuj do katalogu domowego
cd $env:USERPROFILE
git clone https://github.com/affaan-m/everything-claude-code.git
cd everything-claude-code
```

### Krok B: Zainstaluj zależności npm

```powershell
npm install
```

### Krok C: Uruchom instalator dla Javy

```powershell
# Windows PowerShell
.\install.ps1 java

# Akceptuj wszystkie pytania (Y)
```

Jeśli skrypt nie działa z powodu ExecutionPolicy:
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
.\install.ps1 java
```

### Krok D: Kopiuj skills Spring Boot manualnie

Instalator może nie mieć wszystkich potrzebnych skills. Skopiuj je ręcznie:

```powershell
# Stwórz katalogi jeśli nie istnieją
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\skills"
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\agents"
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\commands"
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\rules"

# Kopiuj rules (OBOWIĄZKOWE)
Copy-Item -Recurse -Force "$env:USERPROFILE\everything-claude-code\rules\common\*" "$env:USERPROFILE\.claude\rules\"

# Kopiuj skills Spring Boot
$skills = @(
    "springboot-patterns",
    "springboot-tdd",
    "springboot-security",
    "springboot-verification",
    "docker-patterns",
    "api-design",
    "database-migrations",
    "deployment-patterns",
    "e2e-testing",
    "tdd-workflow",
    "security-review",
    "backend-patterns"
)

foreach ($skill in $skills) {
    $src = "$env:USERPROFILE\everything-claude-code\skills\$skill"
    if (Test-Path $src) {
        Copy-Item -Recurse -Force $src "$env:USERPROFILE\.claude\skills\"
        Write-Host "✓ Copied skill: $skill"
    } else {
        Write-Host "⚠ Skill not found: $skill (skip)"
    }
}

# Kopiuj agentów Java
$agents = @(
    "java-reviewer.md",
    "java-build-resolver.md",
    "database-reviewer.md",
    "tdd-guide.md",
    "planner.md",
    "architect.md",
    "code-reviewer.md",
    "security-reviewer.md",
    "e2e-runner.md",
    "refactor-cleaner.md",
    "doc-updater.md"
)

foreach ($agent in $agents) {
    $src = "$env:USERPROFILE\everything-claude-code\agents\$agent"
    if (Test-Path $src) {
        Copy-Item -Force $src "$env:USERPROFILE\.claude\agents\"
        Write-Host "✓ Copied agent: $agent"
    } else {
        Write-Host "⚠ Agent not found: $agent (skip)"
    }
}

# Kopiuj commands
Copy-Item -Recurse -Force "$env:USERPROFILE\everything-claude-code\commands\*" "$env:USERPROFILE\.claude\commands\"
Write-Host "✓ Commands copied"
```

### Krok E: Skonfiguruj ~/.claude/settings.json

```powershell
# Stwórz plik konfiguracyjny Claude Code
$settings = @{
    model = "sonnet"
    env = @{
        MAX_THINKING_TOKENS = "10000"
        CLAUDE_AUTOCOMPACT_PCT_OVERRIDE = "50"
        CLAUDE_CODE_SUBAGENT_MODEL = "haiku"
    }
} | ConvertTo-Json -Depth 3

$settingsPath = "$env:USERPROFILE\.claude\settings.json"
$settings | Out-File -FilePath $settingsPath -Encoding UTF8
Write-Host "✓ Settings saved to $settingsPath"
```

Weryfikacja że plik wygląda prawidłowo:
```powershell
Get-Content "$env:USERPROFILE\.claude\settings.json"
```

Oczekiwany output:
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

### Krok F: Zainstaluj plugin ECC w Claude Code

```powershell
# Stwórz projekt testowy
mkdir C:\hackathon-setup-test
cd C:\hackathon-setup-test

# Uruchom Claude Code
claude
```

W sesji Claude Code wpisz:
```
/plugin marketplace add affaan-m/everything-claude-code
```
(poczekaj na potwierdzenie)
```
/plugin install everything-claude-code@everything-claude-code
```
(poczekaj na potwierdzenie)
```
/plugin list everything-claude-code@everything-claude-code
```
(powinna pojawić się lista komend)
```
/exit
```

---

## 9. Krok 7 — Git konfiguracja i shared repo

### Konfiguracja Git (każda osoba indywidualnie)

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

> `core.autocrlf input` — krytyczne przy pracy team Windows+Linux. Bez tego Git będzie dodawał Windows-style line endings do plików i będziecie mieli noise w diffach.

### SSH klucze dla GitHub/GitLab (jeśli używacie)

```powershell
# Wygeneruj klucz SSH
ssh-keygen -t ed25519 -C "email@firma.pl"
# Naciśnij Enter na wszystkie pytania (domyślne ścieżki, bez passphrase dla hackathonu)

# Wyświetl klucz publiczny
Get-Content "$env:USERPROFILE\.ssh\id_ed25519.pub"
# Skopiuj ten klucz i dodaj w GitHub/GitLab → Settings → SSH Keys
```

### Klonowanie shared repo (dla każdej osoby)

```powershell
# Stwórz katalog roboczy
mkdir C:\projects
cd C:\projects

# Sklonuj projekt (zastąp URL waszym repo)
git clone git@github.com:wasz-org/hackathon-app.git
cd hackathon-app

# Sprawdź że repo się sklonowało
git log --oneline -5
git branch -a
```

---

## 10. Krok 8 — Weryfikacja całości

Uruchom ten skrypt weryfikacyjny — każda linia powinna dać zielone ✓:

```powershell
# ============================================
# VERIFICATION SCRIPT — uruchom po instalacji
# ============================================

$errors = 0

function Check($name, $cmd, $expectedPattern) {
    try {
        $result = Invoke-Expression $cmd 2>&1 | Out-String
        if ($result -match $expectedPattern) {
            Write-Host "✓ $name" -ForegroundColor Green
        } else {
            Write-Host "✗ $name — got: $result" -ForegroundColor Red
            $script:errors++
        }
    } catch {
        Write-Host "✗ $name — error: $_" -ForegroundColor Red
        $script:errors++
    }
}

Write-Host "`n=== HACKATHON VM VERIFICATION ===" -ForegroundColor Cyan

Check "Git"        "git --version"      "git version"
Check "Node.js"    "node --version"     "v2[2-9]|v[3-9][0-9]"
Check "npm"        "npm --version"      "[0-9]+\.[0-9]+"
Check "Java 21"    "java -version 2>&1" "21\."
Check "javac 21"   "javac -version"     "21\."
Check "Maven"      "mvn --version"      "Apache Maven 3\."
Check "Docker"     "docker --version"   "Docker version"
Check "Docker Compose" "docker compose version" "Docker Compose"
Check "Docker Running"  "docker run --rm hello-world 2>&1" "Hello from Docker"
Check "Claude Code" "claude --version"  "Claude Code"
Check "WSL2"       "wsl -l -v"          "2"

# Sprawdź pliki ECC
$claudeDir = "$env:USERPROFILE\.claude"
if (Test-Path "$claudeDir\settings.json") {
    Write-Host "✓ Claude settings.json" -ForegroundColor Green
} else {
    Write-Host "✗ Claude settings.json MISSING" -ForegroundColor Red
    $errors++
}

if ((Get-ChildItem "$claudeDir\commands" -ErrorAction SilentlyContinue).Count -gt 10) {
    Write-Host "✓ ECC commands installed" -ForegroundColor Green
} else {
    Write-Host "✗ ECC commands missing or incomplete" -ForegroundColor Red
    $errors++
}

if ((Get-ChildItem "$claudeDir\agents" -ErrorAction SilentlyContinue).Count -gt 5) {
    Write-Host "✓ ECC agents installed" -ForegroundColor Green
} else {
    Write-Host "✗ ECC agents missing" -ForegroundColor Red
    $errors++
}

Write-Host "`n=== RESULT ===" -ForegroundColor Cyan
if ($errors -eq 0) {
    Write-Host "✅ ALL CHECKS PASSED — VM ready for hackathon!" -ForegroundColor Green
} else {
    Write-Host "❌ $errors check(s) FAILED — fix before hackathon" -ForegroundColor Red
}
```

---

## 11. Skrypt automatyczny — one-shot setup

Jeśli wolisz uruchomić wszystko jednym skryptem zamiast krok po kroku.

**UWAGA:** Ten skrypt instaluje Rancher Desktop (nie Docker Desktop). Wymaga restartu po WSL2.

Zapisz jako `C:\setup-hackathon.ps1` i uruchom jako Admin:

```powershell
# ============================================================
# HACKATHON VM SETUP SCRIPT
# Uruchom jako Administrator w PowerShell
# Wymaga restartu po Kroku 1 (WSL2)
# ============================================================

param(
    [switch]$SkipWSL,
    [switch]$SkipRestart
)

Set-ExecutionPolicy RemoteSigned -Scope CurrentUser -Force

Write-Host "🚀 Hackathon VM Setup Starting..." -ForegroundColor Cyan

# ---- KROK 0: winget update ----
Write-Host "`n[0/7] Updating winget..." -ForegroundColor Yellow
winget upgrade winget --accept-package-agreements --accept-source-agreements 2>$null

# ---- KROK 1: WSL2 ----
if (-not $SkipWSL) {
    Write-Host "`n[1/7] Installing WSL2..." -ForegroundColor Yellow
    wsl --install --no-distribution
    Write-Host "`n⚠️  RESTART REQUIRED after WSL2 install!" -ForegroundColor Red
    Write-Host "After restart, run: .\setup-hackathon.ps1 -SkipWSL" -ForegroundColor Yellow
    if (-not $SkipRestart) {
        Write-Host "Restarting in 15 seconds... (Ctrl+C to cancel)" -ForegroundColor Yellow
        Start-Sleep 15
        Restart-Computer -Force
    }
    return
}

# ---- KROK 2: WSL2 post-restart config ----
Write-Host "`n[2/7] Configuring WSL2..." -ForegroundColor Yellow
wsl --set-default-version 2
wsl --install -d Ubuntu-22.04

# WSL2 resource limits
$wslConfig = "[wsl2]`nmemory=10GB`nprocessors=3`nswap=2GB`nlocalhostForwarding=true"
$wslConfig | Out-File "$env:USERPROFILE\.wslconfig" -Encoding UTF8
Write-Host "✓ WSL2 configured" -ForegroundColor Green

# ---- KROK 3: Windows Terminal ----
Write-Host "`n[3/7] Installing Windows Terminal..." -ForegroundColor Yellow
winget install --id Microsoft.WindowsTerminal -e --silent --accept-package-agreements --accept-source-agreements

# ---- KROK 4: Dev Tools ----
Write-Host "`n[4/7] Installing dev tools (Git, Node, Java, Maven)..." -ForegroundColor Yellow

$packages = @(
    @{id="Git.Git"; name="Git"},
    @{id="OpenJS.NodeJS.LTS"; name="Node.js LTS"},
    @{id="Microsoft.OpenJDK.21"; name="Java 21 JDK"},
    @{id="Apache.Maven"; name="Maven"},
    @{id="Microsoft.VisualStudioCode"; name="VS Code"}
)

foreach ($pkg in $packages) {
    Write-Host "  Installing $($pkg.name)..." -NoNewline
    winget install --id $pkg.id -e --silent --accept-package-agreements --accept-source-agreements 2>$null
    Write-Host " ✓" -ForegroundColor Green
}

# ---- KROK 5: Rancher Desktop (Docker) ----
Write-Host "`n[5/7] Installing Rancher Desktop (Docker)..." -ForegroundColor Yellow
winget install --id SUSE.RancherDesktop -e --accept-package-agreements --accept-source-agreements
Write-Host "✓ Rancher Desktop installed — launch it manually and set container engine to 'dockerd'" -ForegroundColor Green

# ---- KROK 6: Refresh PATH ----
Write-Host "`n[6/7] Refreshing environment variables..." -ForegroundColor Yellow
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")

# ---- KROK 7: Claude Code + ECC ----
Write-Host "`n[7/7] Installing Claude Code + ECC..." -ForegroundColor Yellow

# Claude Code
npm install -g @anthropic-ai/claude-code
Write-Host "✓ Claude Code CLI installed" -ForegroundColor Green

# ECC
cd $env:USERPROFILE
if (-not (Test-Path "everything-claude-code")) {
    git clone https://github.com/affaan-m/everything-claude-code.git
}
cd everything-claude-code
npm install

# Katalogi
$claudeDir = "$env:USERPROFILE\.claude"
@("skills","agents","commands","rules") | ForEach-Object {
    New-Item -ItemType Directory -Force -Path "$claudeDir\$_" | Out-Null
}

# Rules
Copy-Item -Recurse -Force "rules\common\*" "$claudeDir\rules\" 2>$null

# Skills
@("springboot-patterns","springboot-tdd","springboot-security","springboot-verification",
  "docker-patterns","api-design","database-migrations","e2e-testing","tdd-workflow","backend-patterns") | ForEach-Object {
    if (Test-Path "skills\$_") {
        Copy-Item -Recurse -Force "skills\$_" "$claudeDir\skills\" 2>$null
    }
}

# Agents
@("java-reviewer.md","java-build-resolver.md","database-reviewer.md",
  "tdd-guide.md","planner.md","architect.md","code-reviewer.md","security-reviewer.md","e2e-runner.md") | ForEach-Object {
    if (Test-Path "agents\$_") {
        Copy-Item -Force "agents\$_" "$claudeDir\agents\" 2>$null
    }
}

# Commands
Copy-Item -Recurse -Force "commands\*" "$claudeDir\commands\" 2>$null

# Settings
@{
    model = "sonnet"
    env = @{
        MAX_THINKING_TOKENS = "10000"
        CLAUDE_AUTOCOMPACT_PCT_OVERRIDE = "50"
        CLAUDE_CODE_SUBAGENT_MODEL = "haiku"
    }
} | ConvertTo-Json -Depth 3 | Out-File "$claudeDir\settings.json" -Encoding UTF8

Write-Host "✓ ECC installed" -ForegroundColor Green

Write-Host "`n============================================" -ForegroundColor Cyan
Write-Host "✅ SETUP COMPLETE" -ForegroundColor Green
Write-Host "============================================" -ForegroundColor Cyan
Write-Host ""
Write-Host "NEXT STEPS:" -ForegroundColor Yellow
Write-Host "1. Launch Rancher Desktop → set engine to 'dockerd (moby)'"
Write-Host "2. Open new terminal and run: claude login"
Write-Host "3. Run verification: cd C:\projects && claude"
Write-Host "   Then: /plugin marketplace add affaan-m/everything-claude-code"
Write-Host "   Then: /plugin install everything-claude-code@everything-claude-code"
```

Uruchamianie:
```powershell
# Krok 1 (przed restartem):
.\setup-hackathon.ps1

# Po restarcie (krok 2+):
.\setup-hackathon.ps1 -SkipWSL
```

---

## 12. Troubleshooting

### Problem: `winget` nie jest rozpoznany

```powershell
# Windows Server 2022 może nie mieć winget domyślnie
# Zainstaluj App Installer z Microsoft Store lub ręcznie:
$url = "https://aka.ms/getwinget"
Invoke-WebRequest -Uri $url -OutFile "$env:TEMP\AppInstaller.msixbundle"
Add-AppxPackage "$env:TEMP\AppInstaller.msixbundle"
```

---

### Problem: `java` nie jest widoczna po instalacji

```powershell
# Odświeź zmienne środowiskowe w bieżącej sesji
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")

# Lub po prostu otwórz NOWĄ sesję PowerShell
# Instalacje winget wymagają nowego terminalu żeby PATH był zaktualizowany
```

---

### Problem: Docker Desktop wymaga licencji Business

Użyjcie Rancher Desktop (Krok 3, Opcja A) — identyczna funkcjonalność, zero kosztów, zero ograniczeń licencyjnych dla środowisk VM.

---

### Problem: WSL2 nie może włączyć nested virtualization

```
Error: 0x80370102 — The virtual machine could not be started because a required feature is not installed.
```

Przyczyna: VM w Azure ma zły rozmiar (B-series, A-series).

Rozwiązanie: Zmień rozmiar VM w Azure Portal:
```
VM → Rozmiar → zmień na Standard_D4s_v5 lub D8s_v5
(VM musi być zatrzymana żeby zmienić rozmiar)
```

---

### Problem: `docker run hello-world` zawiesza się

```powershell
# 1. Sprawdź czy Rancher Desktop / Docker Desktop jest uruchomiony
# (ikona w tray systemowym powinna być zielona)

# 2. Jeśli zielona a docker nie odpowiada:
wsl --shutdown
# Poczekaj 10 sekund, Rancher Desktop sam się zrestartuje

# 3. Sprawdź log Docker daemon w WSL2:
wsl -d rancher-desktop -- journalctl -u docker --since "5 minutes ago"
```

---

### Problem: `claude login` nie otwiera przeglądarki

```powershell
# Azure VMs domyślnie nie mają przeglądarki graficznej
# Skopiuj URL który Claude wyświetli w terminalu
# i otwórz go w przeglądarce na swoim lokalnym komputerze (przez którego RDP)
claude login
# Skopiuj wyświetlony URL → otwórz w lokalnej przeglądarce → zaloguj → skopiuj kod → wklej z powrotem w terminal
```

---

### Problem: `./install.sh` zamiast `install.ps1` (zły terminal)

ECC ma dwa skrypty: `install.sh` (dla Linux/macOS) i `install.ps1` (dla Windows). Upewnij się że uruchamiasz PowerShell, nie bash/WSL.

---

### Problem: `Set-ExecutionPolicy` blokuje skrypt

```powershell
# Uruchom skrypt z explicitną polityką:
powershell.exe -ExecutionPolicy Bypass -File .\setup-hackathon.ps1 -SkipWSL
```

---

### Problem: Maven używa złej wersji Javy (`mvn --version` pokazuje Java 11/17)

```powershell
# Sprawdź co jest w JAVA_HOME
echo $env:JAVA_HOME

# Sprawdź wszystkie zainstalowane JDK
Get-Command java | Select-Object -ExpandProperty Source

# Wymuś JAVA_HOME na Java 21
$java21Path = (Get-ChildItem "C:\Program Files\Microsoft" -Filter "jdk-21*" | Select-Object -First 1).FullName
[System.Environment]::SetEnvironmentVariable("JAVA_HOME", $java21Path, "Machine")

# Otwórz nowy terminal i sprawdź
mvn --version  # powinno pokazać Java 21
```

---

## 13. Checklista dla każdej osoby

Każdy member zespołu potwierdza przed hackathon:

```
AZURE VM:
[ ] VM rozmiar: Standard_D4s_v5 lub wyższy (NIE B-series!)
[ ] Połączenie RDP działa
[ ] PowerShell (Admin) dostępny

SYSTEM:
[ ] WSL2 zainstalowany i Ubuntu-22.04 w wersji 2 (wsl -l -v)
[ ] Windows Terminal zainstalowany

NARZĘDZIA DEV:
[ ] git --version        → git version 2.x.x
[ ] node --version       → v22.x.x
[ ] npm --version        → 10.x.x
[ ] java -version        → openjdk 21.x.x
[ ] javac -version       → javac 21.x.x
[ ] mvn --version        → Apache Maven 3.9.x + Java version: 21
[ ] docker --version     → Docker version 27.x.x
[ ] docker compose version → v2.x.x
[ ] docker run hello-world → "Hello from Docker!" (bez błędów)

CLAUDE:
[ ] claude --version     → Claude Code v2.x.x
[ ] claude login         → zalogowany (konto z Claude Max)
[ ] ~/.claude/settings.json istnieje z model: sonnet
[ ] ~/.claude/commands/ zawiera plan.md, tdd.md etc. (>10 plików)
[ ] ~/.claude/agents/ zawiera java-reviewer.md etc. (>5 plików)
[ ] Plugin ECC zainstalowany (w sesji claude: /plan działa)

GIT:
[ ] git config --global user.name ustawione
[ ] git config --global user.email ustawione
[ ] Shared repo sklonowane do C:\projects\hackathon-app
[ ] git status w repo działa bez błędów

IDE:
[ ] IntelliJ IDEA uruchomia się (opcjonalnie: plugin Lombok zainstalowany)
[ ] VS Code uruchomia się
```

**Gdy każda osoba ma wszystkie checkboxy — jesteście gotowi.**

---

## Podsumowanie narzędzi i wersji

| Narzędzie | Wersja | Do czego |
|---|---|---|
| Windows Server | 2022 Datacenter Gen2 | OS na Azure VM |
| WSL2 + Ubuntu 22.04 | 2.x | Silnik Dockera |
| Git | 2.x.x | Kontrola wersji |
| Node.js | 22.x.x (LTS) | Wymagany przez Claude Code |
| npm | 10.x.x | Package manager dla Node |
| Microsoft OpenJDK | 21.x.x | Java runtime + kompilator |
| Apache Maven | 3.9.x | Build system dla Spring Boot |
| Rancher Desktop | najnowszy | Docker Engine (bezpłatna alternatywa Docker Desktop) |
| Docker Compose | v2.x.x | Orchestracja kontenerów (included w Rancher) |
| VS Code | najnowszy | Edytor (FE, QA) |
| IntelliJ IDEA Community | najnowszy | IDE Java (BE, DB) |
| Claude Code CLI | v2.x.x | Agent AI |
| Everything Claude Code | v1.9.0 | Plugin + skills + agenci |

---

*Dokument przygotowany dla Azure Windows VM D4s_v5 · Windows Server 2022 · Hackathon 2026*
