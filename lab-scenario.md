# Lab Scenario

Scenariusz tego laboratorium zakłada przyswojenie najważniejszych funkcjonalności GitHub.

## Cel

Celem jest przyswojenie wiedzy i zwiększenie umiejętności korzystania z GitHub, w szczególności:

- pracy z forkami i branchami,
- tworzenia Pull Requestów,
- wykonywania code review,
- promowania zmian pomiędzy branchami `dev`, `uat` i `main`,
- tworzenia tagów wersji,
- publikowania GitHub Releases,
- aktualizowania ruchomych tagów środowiskowych.

## Założenia

Repozytorium jest publiczne i należy do **Użytkownika A**.

**Użytkownik B** wykonuje fork repozytorium i przygotowuje zmiany we własnej kopii.

Główne repozytorium posiada trzy branche:

| Branch | Przeznaczenie |
|---|---|
| `dev` | rozwój i integracja nowych zmian |
| `uat` | testy akceptacyjne |
| `main` | stabilna wersja produkcyjna |

Przepływ zmian:

```text
fork Użytkownika B
        ↓
feature branch
        ↓ Pull Request
dev
        ↓ Pull Request
uat
        ↓ Pull Request
main
        ↓
tag wersji i GitHub Release
````

## Model tagów

W laboratorium używane są dwa rodzaje tagów.

### Niezmienne tagi wersji

Tagi wersji wskazują dokładny commit konkretnego wydania:

```text
v1.0.0
v1.0.1
v1.1.0
```

Po opublikowaniu wersji nie należy przesuwać takiego taga na inny commit.

Przykład:

```text
v1.0.0 → commit pierwszego stabilnego wydania
v1.0.1 → commit zawierający późniejszą poprawkę
```

### Ruchome tagi środowiskowe

Tagi `latest-*` wskazują aktualną wersję znajdującą się na określonym środowisku:

| Tag           | Znaczenie                                     |
| ------------- | --------------------------------------------- |
| `latest-dev`  | najnowszy commit przekazany do środowiska DEV |
| `latest-uat`  | najnowszy commit przekazany do środowiska UAT |
| `latest-prod` | commit aktualnej wersji produkcyjnej          |

Tagi te są celowo przesuwane po każdej aktualizacji danego środowiska.

```text
latest-dev  → aktualny commit brancha dev
latest-uat  → aktualny commit brancha uat
latest-prod → commit aktualnego wydania produkcyjnego
```

Tag `latest-prod` nie jest tym samym co oznaczenie **Latest release** widoczne w GitHub Releases. Jest to zwykły, samodzielnie utworzony tag Git.

## Konfiguracja repozytorium

### Ochrona branchy

Dla branchy `dev`, `uat` i `main` należy skonfigurować ochronę:

* Require a pull request before merging,
* Require approvals,
* Required approvals: `1`,
* Require conversation resolution before merging,
* blokada bezpośrednich zmian na chronionych branchach.

### Ochrona tagów wersji

Opcjonalnie należy utworzyć tag ruleset dla wzorca:

```text
v*
```

Ruleset powinien ograniczać:

* aktualizowanie istniejących tagów wersji,
* usuwanie tagów wersji.

Tagi:

```text
latest-dev
latest-uat
latest-prod
```

muszą pozostać możliwe do aktualizowania przez właściciela repozytorium albo osobę odpowiedzialną za wydania.

## Konfiguracja początkowych tagów `latest-*`

GitHub Web UI nie oferuje wygodnej operacji przesunięcia istniejącego taga. Dlatego do zarządzania ruchomymi tagami używany jest Git.

Użytkownik A klonuje główne repozytorium:

```bash
git clone https://github.com/USER_A/REPOSITORY.git
cd REPOSITORY
git fetch origin --prune --tags
```

Następnie tworzy początkowe tagi:

```bash
git tag -f latest-dev origin/dev
git push origin refs/tags/latest-dev --force

git tag -f latest-uat origin/uat
git push origin refs/tags/latest-uat --force

git tag -f latest-prod origin/main
git push origin refs/tags/latest-prod --force
```

Początkowy stan:

```text
latest-dev  → HEAD brancha dev
latest-uat  → HEAD brancha uat
latest-prod → HEAD brancha main
```

---

## Scenariusz 1 – Pull Request z forka i promocja zmian

### Cel ćwiczenia

Przećwiczenie pełnego przepływu:

```text
fork → feature branch → Pull Request → review → dev → uat → main
```

### Etap 1 – Utworzenie zadania

1. Użytkownik A tworzy Issue:

   ```text
   Add Linux security recommendations
   ```

2. Acceptance criteria:

   * utworzyć plik `LINUX_SECURITY.md`,
   * opisać aktualizację systemu,
   * opisać konfigurację firewalla,
   * opisać podstawowe zabezpieczenia SSH,
   * dodać link do dokumentu w `README.md`.

3. Issue otrzymuje label:

   ```text
   documentation
   ```

### Etap 2 – Fork i branch roboczy

1. Użytkownik B wykonuje fork głównego repozytorium.

2. W swoim forku synchronizuje branch `dev` z repozytorium źródłowym.

3. Na podstawie `dev` tworzy branch:

   ```text
   feature/add-linux-security-guide
   ```

4. Użytkownik B upewnia się, że edytuje branch w swoim forku, a nie główne repozytorium.

### Etap 3 – Wprowadzenie zmian

1. Użytkownik B tworzy plik:

   ```text
   LINUX_SECURITY.md
   ```

2. Zapisuje pierwszy commit:

   ```text
   Add Linux security recommendations
   ```

3. Edytuje `README.md` i dodaje link:

   ```md
   ## Documentation

   - [Linux Security Recommendations](LINUX_SECURITY.md)
   ```

4. Zapisuje drugi commit:

   ```text
   Link Linux security guide from README
   ```

### Etap 4 – Pull Request do `dev`

Użytkownik B tworzy Pull Request:

```text
USER_B/REPOSITORY:feature/add-linux-security-guide
                         ↓
USER_A/REPOSITORY:dev
```

Opis Pull Requesta:

```md
## Summary

Added Linux security recommendations covering:

- system updates,
- firewall configuration,
- SSH hardening.

The document was linked from README.md.

Closes #1
```

### Etap 5 – Code review

1. Użytkownik A przechodzi do zakładki **Files changed**.

2. Dodaje komentarz:

   ```text
   Please add a command for validating the SSH configuration.
   ```

3. Wybiera:

   ```text
   Request changes
   ```

4. Użytkownik B poprawia plik na tym samym branchu i dodaje:

   ```bash
   sudo sshd -t
   ```

5. Tworzy kolejny commit:

   ```text
   Add SSH configuration validation command
   ```

6. Istniejący Pull Request aktualizuje się automatycznie.

7. Użytkownik B odpowiada na komentarz:

   ```text
   Added the requested SSH configuration validation command.
   ```

8. Dyskusja zostaje oznaczona jako rozwiązana.

9. Użytkownik A wybiera:

   ```text
   Approve
   ```

10. Pull Request zostaje zmergowany do `dev`.

### Etap 6 – Aktualizacja `latest-dev`

Po merge Użytkownik A aktualizuje lokalne dane:

```bash
git fetch origin --prune --tags
```

Następnie przesuwa tag `latest-dev`:

```bash
git tag -f latest-dev origin/dev
git push origin refs/tags/latest-dev --force
```

Rezultat:

```text
latest-dev → najnowszy commit brancha dev
```

### Etap 7 – Promocja do UAT

1. Użytkownik A tworzy Pull Request:

   ```text
   dev → uat
   ```

2. Tytuł:

   ```text
   Promote Linux security documentation to UAT
   ```

3. Po review i testach Pull Request zostaje zmergowany.

4. Użytkownik A aktualizuje tag:

   ```bash
   git fetch origin --prune --tags
   git tag -f latest-uat origin/uat
   git push origin refs/tags/latest-uat --force
   ```

Rezultat:

```text
latest-uat → najnowszy commit brancha uat
```

### Etap 8 – Promocja do produkcji

1. Po pozytywnych testach UAT Użytkownik A tworzy Pull Request:

   ```text
   uat → main
   ```

2. Tytuł:

   ```text
   Release Linux security documentation to production
   ```

3. Po zatwierdzeniu Pull Request zostaje zmergowany.

4. Na tym etapie nie należy jeszcze przesuwać `latest-prod`.

Tag `latest-prod` zostanie zaktualizowany po utworzeniu oficjalnego Release.

### Oczekiwany rezultat

| Element                   | Rezultat                                     |
| ------------------------- | -------------------------------------------- |
| Pull Request z forka      | `Merged` do `dev`                            |
| Pull Request `dev → uat`  | `Merged`                                     |
| Pull Request `uat → main` | `Merged`                                     |
| `latest-dev`              | wskazuje aktualny `dev`                      |
| `latest-uat`              | wskazuje aktualny `uat`                      |
| `latest-prod`             | nadal wskazuje poprzednią wersję produkcyjną |

---

## Scenariusz 2 – Pierwszy Release i aktualizacja `latest-prod`

### Cel ćwiczenia

Przećwiczenie utworzenia trwałego taga wersji i opublikowania GitHub Release.

### Etap 1 – Utworzenie Release

Użytkownik A przechodzi do:

```text
Releases → Draft a new release
```

Ustawienia:

| Pole           | Wartość                               |
| -------------- | ------------------------------------- |
| Tag            | `v1.0.0`                              |
| Target         | `main`                                |
| Release title  | `Linux Security Documentation v1.0.0` |
| Pre-release    | wyłączone                             |
| Latest release | włączone                              |

Release notes:

```md
## Version 1.0.0

First stable release of the Linux security documentation.

### Added

- Linux system update recommendations
- Firewall verification guidance
- SSH hardening recommendations
- SSH configuration validation command
- Documentation link in README
```

Następnie Użytkownik A wybiera:

```text
Publish release
```

Powstaje trwały tag:

```text
v1.0.0
```

### Etap 2 – Aktualizacja `latest-prod`

Po opublikowaniu Release Użytkownik A pobiera nowy tag:

```bash
git fetch origin --prune --tags
```

Tag `latest-prod` powinien wskazywać dokładnie ten sam commit co `v1.0.0`:

```bash
git tag -f latest-prod v1.0.0
git push origin refs/tags/latest-prod --force
```

Rezultat:

```text
v1.0.0     ─┐
             ├── ten sam commit produkcyjny
latest-prod ─┘
```

### Etap 3 – Weryfikacja

W GitHub UI należy przejść do:

```text
Releases → Tags
```

Następnie sprawdzić, czy:

* `v1.0.0` istnieje,
* `latest-prod` istnieje,
* oba tagi pokazują ten sam commit SHA,
* `latest-dev` wskazuje aktualny commit `dev`,
* `latest-uat` wskazuje aktualny commit `uat`.

Można również wykonać lokalną weryfikację:

```bash
git rev-parse v1.0.0
git rev-parse latest-prod
```

Oba polecenia powinny zwrócić ten sam pełny SHA.

### Oczekiwany rezultat

| Tag           | Wskazywany stan                 |
| ------------- | ------------------------------- |
| `v1.0.0`      | niezmienny commit wydania 1.0.0 |
| `latest-prod` | ten sam commit co `v1.0.0`      |
| `latest-uat`  | aktualny commit środowiska UAT  |
| `latest-dev`  | aktualny commit środowiska DEV  |

---

## Scenariusz 3 – Nowa wersja i przesunięcie tagów `latest-*`

### Cel ćwiczenia

Przećwiczenie wydania poprawki `v1.0.1` oraz przesunięcia tagów środowiskowych na nowsze commity.

### Etap 1 – Nowa poprawka

1. Użytkownik A tworzy Issue:

   ```text
   Fix SSH service restart instructions
   ```

2. Użytkownik B synchronizuje fork.

3. Na podstawie aktualnego `dev` tworzy branch:

   ```text
   fix/ssh-restart-instructions
   ```

4. Wprowadza poprawkę.

5. Tworzy commit:

   ```text
   Fix SSH restart instructions
   ```

6. Otwiera Pull Request:

   ```text
   USER_B:fix/ssh-restart-instructions → USER_A:dev
   ```

7. Pull Request przechodzi review i zostaje zmergowany.

### Etap 2 – Przesunięcie `latest-dev`

Po merge do `dev`:

```bash
git fetch origin --prune --tags
git tag -f latest-dev origin/dev
git push origin refs/tags/latest-dev --force
```

Nowy stan:

```text
latest-dev → commit zawierający poprawkę
```

Pozostałe tagi jeszcze się nie zmieniają:

```text
latest-uat  → poprzednia wersja na UAT
latest-prod → v1.0.0
```

### Etap 3 – Promocja poprawki do UAT

1. Użytkownik A tworzy Pull Request:

   ```text
   dev → uat
   ```

2. Po review, merge i testach aktualizuje tag:

   ```bash
   git fetch origin --prune --tags
   git tag -f latest-uat origin/uat
   git push origin refs/tags/latest-uat --force
   ```

Nowy stan:

```text
latest-dev → nowa poprawka
latest-uat → nowa poprawka po testach
latest-prod → nadal v1.0.0
```

### Etap 4 – Promocja poprawki do produkcji

1. Użytkownik A tworzy Pull Request:

   ```text
   uat → main
   ```

2. Po zatwierdzeniu wykonuje merge do `main`.

3. Tworzy nowy Release:

   ```text
   Tag: v1.0.1
   Target: main
   ```

4. Release title:

   ```text
   Linux Security Documentation v1.0.1
   ```

5. Release notes:

   ```md
   ## Version 1.0.1

   ### Fixed

   - Corrected SSH service validation and restart instructions.
   ```

6. Publikuje Release.

### Etap 5 – Przesunięcie `latest-prod`

Po opublikowaniu wersji:

```bash
git fetch origin --prune --tags
git tag -f latest-prod v1.0.1
git push origin refs/tags/latest-prod --force
```

Nowy stan:

```text
v1.0.0 → stary commit wydania 1.0.0
v1.0.1 → nowy commit wydania 1.0.1

latest-prod → ten sam commit co v1.0.1
```

Tag `v1.0.0` pozostaje bez zmian.

### Etap 6 – Weryfikacja końcowa

Należy potwierdzić:

| Tag           | Oczekiwany rezultat                       |
| ------------- | ----------------------------------------- |
| `v1.0.0`      | nadal wskazuje pierwsze wydanie           |
| `v1.0.1`      | wskazuje nowe wydanie                     |
| `latest-prod` | wskazuje ten sam commit co `v1.0.1`       |
| `latest-uat`  | wskazuje najnowszy commit wdrożony na UAT |
| `latest-dev`  | wskazuje najnowszy commit wdrożony na DEV |

Lokalna weryfikacja:

```bash
git fetch origin --prune --tags

git rev-parse v1.0.0
git rev-parse v1.0.1
git rev-parse latest-prod
git rev-parse latest-uat
git rev-parse latest-dev
```

Wartości:

```text
git rev-parse v1.0.1
git rev-parse latest-prod
```

powinny być identyczne.

## Ważna zasada dotycząca środowisk

Nie należy automatycznie przestawiać wszystkich tagów `latest-*` na commit produkcyjny.

Każdy tag opisuje własne środowisko:

```text
latest-dev  → to, co aktualnie znajduje się na DEV
latest-uat  → to, co aktualnie znajduje się na UAT
latest-prod → to, co aktualnie znajduje się na produkcji
```

Jeżeli po wydaniu `v1.0.1` na `dev` rozpoczęto już prace nad wersją `v1.1.0`, wtedy:

```text
latest-dev  → może wskazywać nowszy commit niż v1.0.1
latest-uat  → może nadal wskazywać v1.0.1 albo nowszego kandydata
latest-prod → powinien wskazywać v1.0.1
```

## Podsumowanie przepływu

```text
zmiana w forku
      ↓
Pull Request do dev
      ↓
merge
      ↓
przesunięcie latest-dev
      ↓
Pull Request dev → uat
      ↓
testy i merge
      ↓
przesunięcie latest-uat
      ↓
Pull Request uat → main
      ↓
merge
      ↓
tag wersji v1.0.1
      ↓
GitHub Release v1.0.1
      ↓
przesunięcie latest-prod na v1.0.1
```

## Najważniejsze zasady

| Zasada                                    | Wyjaśnienie                                            |
| ----------------------------------------- | ------------------------------------------------------ |
| Nie przesuwaj `v1.0.0`                    | Tag wersji musi stale wskazywać oryginalne wydanie     |
| Twórz nowy tag dla nowej wersji           | Poprawka otrzymuje np. `v1.0.1`                        |
| Przesuwaj `latest-dev` po zmianie DEV     | Tag wskazuje aktualny stan DEV                         |
| Przesuwaj `latest-uat` po zmianie UAT     | Tag wskazuje aktualny stan UAT                         |
| Przesuwaj `latest-prod` po Release        | Tag wskazuje aktualną wersję produkcyjną               |
| Nie twórz Release z `latest-*`            | Release powinien opierać się na trwałym tagu wersji    |
| Aktualizacja `latest-*` wymaga force push | Istniejący tag musi zostać przestawiony na nowy commit |

```
