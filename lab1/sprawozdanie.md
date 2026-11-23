# Sprawozdanie — Laboratorium 1 (First run: SpamAssassin messages)

## Metadane

- **Aplikacja agentowa:** Antigravity
- **Model:** `Gemini Neon`
- **Poziom rozumowania:** `High`
- **Repozytorium GitHub:** [StarTouch7/TDST](https://github.com/StarTouch7/TDST)
- **Bezpośredni link do końcowego raportu (`exercise-summary.json`):** [lab1/exercise-summary.json](https://github.com/StarTouch7/TDST/blob/main/lab1/exercise-summary.json)
- **Środowisko wykonania:** lokalny komputer z systemem Windows (powłoka PowerShell), Python `3.14.4` (`C:\Users\Maria\AppData\Local\Python\pythoncore-3.14-64\python.exe`), katalog projektu `C:\Users\Maria\Desktop\Studia\Magister\Semestr 2\Tdst\lab-1-project\project`

---

## 1. Co zleciłam agentowi i jakie ustaliłam ograniczenia

Pracę z agentem podzieliłam na kolejne kroki i przed każdym z nich jasno określałam zasady działania:

1. **Zapoznanie z projektem bez zmian i bez uruchamiania:** najpierw zleciłam odczytanie `README.md`, wyjaśnienie celu projektu, opisanie działania skryptu `describe_data.py` oraz pokazanie sposobu jego uruchomienia — z zakazem modyfikowania plików i uruchamiania czegokolwiek.
2. **Analiza danych i kodu:** poprosiłam o pokazanie jednej przykładowej wiadomości z `data/exercise.csv` (`exercise-001`), wyjaśnienie znaczenia kolumn `message_id`, `text`, `label` oraz różnicy między `spam` i `ham`, a następnie o omówienie fragmentu kodu zliczającego wiadomości i przygotowanie małego przykładu do samodzielnego przewidzenia wyniku — nadal bez zmian w projekcie.
3. **Uruchomienie skryptu i testu pod kontrolą:** zanim agent wykonał jakiekolwiek polecenia, poprosiłam o potwierdzenie zrozumienia planu, wskazanie środowiska wykonania i zaczekanie na moją zgodę. Ustaliłam ograniczenia:
   - nie zmieniać kodu, danych ani testów,
   - wygenerować raport dla `data/exercise.csv` i zapisać go w `results/exercise-summary.json`,
   - pokazać dokładne polecenia, wyniki ich wykonania oraz pełną zawartość wygenerowanego raportu,
   - uruchomić gotowy test zgodnie z `README.md`, niczego w testach nie poprawiając ani nie zmieniając.
4. **Kontrolowana zmiana jednej etykiety i ponowne uruchomienie:** zleciłam zmianę wyłącznie jednej etykiety (`ham` → `spam`) w `data/exercise.csv` bez ingerencji w treść wiadomości ani inne dane oraz bez natychmiastowego ponownego uruchamiania programu czy edycji raportu, abym mogła najpierw przewidzieć wynik. Po mojej prognozie zleciłam ponowne uruchomienie skryptu z zastrzeżeniem, aby nie edytować raportu ręcznie i nie cofać zmienionej etykiety.

---

## 2. Jak uruchomiłam program, z jakich danych korzystałam i jaki dostałam raport

### Sprawdzenie środowiska
Przed uruchomieniem skryptu sprawdziłam katalog roboczy narzędzia terminalowego oraz wersję Pythona:

- `Get-Location` (kod wyjścia `0`): domyślny katalog `C:\Users\Maria\AppData\Local\Programs\antigravity`, dlatego polecenia uruchamiające projekt poprzedzano przejściem do katalogu `project` (`Set-Location`).
- `py -3 --version` (kod wyjścia `0`): `Python 3.14.4` (spełnia wymaganie Python 3.10+; skrypt korzysta wyłącznie ze standardowej biblioteki Pythona).

### Dane wejściowe
Korzystałam z przygotowanego pliku ćwiczeniowego `data/exercise.csv` (wiadomości z publicznego korpusu Apache SpamAssassin w formacie CSV o kolumnach `message_id`, `text`, `label`). Treść w kolumnie `text` (temat i ciało wiadomości) jest wieloliniowa, dlatego plik ma 203 fizyczne linie, ale reprezentuje 8 pojedynczych wiadomości e-mail. Jako przykład obejrzałam pierwszy rekord (`exercise-001`), który mimo słowa „Spam” w temacie i reklamowej stopki listy mailingowej miał etykietę `ham` (zwykła wiadomość z listy dyskusyjnej).

### Polecenie i wynik pierwszego uruchomienia
Zgodnie z instrukcją w `README.md` (na Windows z `py -3`) uruchomiłam:

```powershell
Set-Location "c:\Users\Maria\Desktop\Studia\Magister\Semestr 2\Tdst\lab-1-project\project"; py -3 describe_data.py --data data/exercise.csv --out results/exercise-summary.json
```

Wynik wykonania (kod wyjścia `0`):

```text
Report saved: C:\Users\Maria\Desktop\Studia\Magister\Semestr 2\Tdst\lab-1-project\project\results\exercise-summary.json
Messages: 8; labels: {'ham': 5, 'spam': 3}
```

### Raport po pierwszym uruchomieniu (`results/exercise-summary.json`)

```json
{
  "input_file": "C:\\Users\\Maria\\Desktop\\Studia\\Magister\\Semestr 2\\Tdst\\lab-1-project\\project\\data\\exercise.csv",
  "input_sha256": "95e9049fe9f1283ba65514c7060499d175deaa88cd8c582097ced4cceed6ed07",
  "total_messages": 8,
  "label_counts": {
    "ham": 5,
    "spam": 3
  },
  "runtime": {
    "python_executable": "C:\\Users\\Maria\\AppData\\Local\\Python\\pythoncore-3.14-64\\python.exe",
    "python_version": "3.14.4",
    "working_directory": "C:\\Users\\Maria\\Desktop\\Studia\\Magister\\Semestr 2\\Tdst\\lab-1-project\\project"
  }
}
```

Potwierdziłam, że pole `input_file` wskazuje dokładnie na `data/exercise.csv`, raport zapisał się w `results/exercise-summary.json`, a zbiór zawierał pierwotnie **8 wiadomości**: **5 `ham`** oraz **3 `spam`**.

---

## 3. Jaki fragment kodu analizowałam, jak go zrozumiałam i jak sprawdziłam swoje przewidywanie na przykładzie

### Analizowany fragment kodu
Analizowałam pętlę zliczającą oraz konstrukcję wyniku w funkcji `describe()` w pliku `describe_data.py` (linie 19–38):

```python
    counts = {"ham": 0, "spam": 0}
    seen_ids = set()
    for number, row in enumerate(rows, start=1):
        if None in row or any(not value or not value.strip() for value in row.values()):
            raise ValueError(f"Record {number}: missing or extra field.")
        if row["message_id"] in seen_ids:
            raise ValueError(f"Record {number}: duplicate message_id.")
        if row["label"] not in counts:
            raise ValueError(f"Record {number}: label must be ham or spam.")
        seen_ids.add(row["message_id"])
        counts[row["label"]] += 1
```

oraz zwracane podsumowanie:

```python
        "total_messages": sum(counts.values()),
        "label_counts": counts,
```

### Jak zrozumiałam działanie kodu
- Skrypt przygotowuje słownik liczników `counts` (osobno dla `ham` i `spam`, oba startują od `0`) oraz pusty zbiór `seen_ids` do wyłapywania powtarzających się identyfikatorów.
- Przechodzi po kolejnych **rekordach CSV** odczytywanych przez `csv.DictReader`, dzięki czemu wieloliniowy e-mail jest liczony jako jedna wiadomość, a nie jako wiele fizycznych linii pliku.
- Dla każdej wiadomości sprawdza trzy warunki: czy wszystkie pola są obecne i niepuste (oraz czy nie ma nadmiarowych kolumn), czy `message_id` nie pojawiło się już wcześniej oraz czy `label` ma wartość wyłącznie `ham` albo `spam`. Błąd w którymkolwiek warunku przerywa wykonanie wyjątkiem `ValueError`.
- Jeśli rekord jest poprawny, dopisuje `message_id` do `seen_ids` i zwiększa odpowiedni licznik `counts[row["label"]] += 1`.
- Łączną liczbę wiadomości (`total_messages`) wylicza jako sumę wartości ze słownika `counts` (`sum(counts.values())`).

### Sprawdzenie przewidywania na przykładzie
Poprosiłam agenta o przygotowanie małego, przykładowego zbioru CSV bez podawania odpowiedzi. Otrzymałam zestaw 5 wieloliniowych wiadomości (`msg-01` do `msg-05`), w tym wiadomość z etykietą `ham` zawierającą w temacie słowo „Promocja” (`msg-04`).

Przewidziałam, że:
- skrypt zakończy się sukcesem, ponieważ wszystkie rekordy mają unikalne identyfikatory, uzupełnione pola i poprawne etykiety,
- wynik wyniesie **łącznie 5 wiadomości**, w tym **2 `ham`** i **3 `spam`**.

Agent potwierdził, że przewidywanie było w 100% poprawne — potwierdziło to, że program zlicza rekordy wyłącznie na podstawie kolumny `label` (a nie słów z treści czy liczby fizycznych linii).

---

## 4. Jak uruchomiłam test, jaki był jego wynik i co dokładnie sprawdzał

### Polecenie i wynik wykonania
Test uruchomiłam zgodnie z `README.md` z katalogu `project`:

```powershell
Set-Location "c:\Users\Maria\Desktop\Studia\Magister\Semestr 2\Tdst\lab-1-project\project"; py -3 -m unittest discover -s tests -v
```

Wynik (kod wyjścia `0`):

```text
test_cli_writes_expected_counts (test_report.ReportTest.test_cli_writes_expected_counts)
The CLI saves a report for the intended input with counts 3 / 2 / 1. ... ok

----------------------------------------------------------------------
Ran 1 test in 0.328s

OK
```

### Co dokładnie sprawdzał test
Test zdefiniowany w `tests/test_report.py` (`ReportTest.test_cli_writes_expected_counts`):
- korzysta z 3 **syntetycznych**, wieloliniowych wiadomości z pliku `tests/fixtures/messages.csv` (`test-001` — `ham` z przecinkiem w treści, `test-002` — `spam`, `test-003` — `ham`), a nie z danych SpamAssassin ani z `data/exercise.csv`,
- uruchamia `describe_data.py` jako osobny podproces (`subprocess.run`) z parametrami `--data tests/fixtures/messages.csv` oraz `--out <katalog_tymczasowy>/summary.json`,
- sprawdza, czy:
  1. program zakończył się kodem wyjścia `0`,
  2. plik raportu został utworzony i daje się wczytać jako poprawny JSON,
  3. pole `input_file` wskazuje dokładnie na `tests/fixtures/messages.csv`,
  4. `total_messages` wynosi dokładnie `3`,
  5. `label_counts` wynosi dokładnie `{"ham": 2, "spam": 1}` (wartości oczekiwane są wpisane w teście na sztywno).

Test zapisuje wynik w tymczasowym katalogu (`tempfile.TemporaryDirectory`) i po zakończeniu automatycznie go usuwa — **nie korzysta z mojego `results/exercise-summary.json` ani go nie nadpisuje**. Jego przejście wspiera poprawność działania skryptu dla tego małego przykładu ze znanym wynikiem, ale nie gwarantuje poprawności dla każdego innego zestawu danych ani jakości przyszłego modelu klasyfikacji.

---

## 5. Zmiana etykiety `exercise-001` z `ham` na `spam`, przewidywanie i wynik po ponownym uruchomieniu

### Wykonana zmiana
W pliku `data/exercise.csv` (linia 24) zmieniłam wyłącznie etykietę wiadomości `exercise-001` (*Subject: [SAdev] Interesting approach to Spam handling..*):
- **stara etykieta:** `ham`
- **nowa etykieta:** `spam`

Treść wiadomości ani żadne inne pola w pliku nie zostały zmienione.

### Przewidywanie przed ponownym uruchomieniem
Ponieważ raport nie aktualizuje się automatycznie po zmianie pliku CSV, należało ponownie uruchomić `describe_data.py`, aby nadpisać `results/exercise-summary.json`. Przed uruchomieniem przewidziałam, że:
- łączna liczba wiadomości pozostanie bez zmian: **`8`**,
- liczba `ham` zmaleje o 1: z `5` do **`4`**,
- liczba `spam` wzrośnie o 1: z `3` do **`4`**.

### Ponowne uruchomienie i końcowy raport
Uruchomiłam ponownie to samo polecenie:

```powershell
Set-Location "c:\Users\Maria\Desktop\Studia\Magister\Semestr 2\Tdst\lab-1-project\project"; py -3 describe_data.py --data data/exercise.csv --out results/exercise-summary.json
```

Wynik (kod wyjścia `0`):

```text
Report saved: C:\Users\Maria\Desktop\Studia\Magister\Semestr 2\Tdst\lab-1-project\project\results\exercise-summary.json
Messages: 8; labels: {'ham': 4, 'spam': 4}
```

Pełna zawartość nowego pliku `results/exercise-summary.json`:

```json
{
  "input_file": "C:\\Users\\Maria\\Desktop\\Studia\\Magister\\Semestr 2\\Tdst\\lab-1-project\\project\\data\\exercise.csv",
  "input_sha256": "7dc8080df480e67e5361218ae0e75f26c963ede9d72788985b25cedb818b8152",
  "total_messages": 8,
  "label_counts": {
    "ham": 4,
    "spam": 4
  },
  "runtime": {
    "python_executable": "C:\\Users\\Maria\\AppData\\Local\\Python\\pythoncore-3.14-64\\python.exe",
    "python_version": "3.14.4",
    "working_directory": "C:\\Users\\Maria\\Desktop\\Studia\\Magister\\Semestr 2\\Tdst\\lab-1-project\\project"
  }
}
```

### Porównanie raportu przed i po zmianie

| Pole raportu | Przed zmianą | Po zmianie | Zgodność z przewidywaniem |
|---|---:|---:|---|
| `total_messages` | `8` | **`8`** | Zgadza się (nadal 8 wiadomości) |
| `label_counts.ham` | `5` | **`4`** | Zgadza się (`5 → 4`) |
| `label_counts.spam` | `3` | **`4`** | Zgadza się (`3 → 4`) |
| `input_sha256` | `95e9049fe9f1283b…ed07` | `7dc8080df480e67e…8152` | Zmieniła się, bo zmieniły się bajty `data/exercise.csv` |

Moje przewidywanie sprawdziło się dokładnie. Raport został wygenerowany wyłącznie przez skrypt (nie był edytowany ręcznie), a zmieniona etykieta `exercise-001` (`spam`) oraz końcowy raport `results/exercise-summary.json` zostały zachowane.
