# 🏥 Projekt zaliczeniowy – ASP.NET Core  
## **System zarządzania przychodnią medyczną 2.0**

---

## 🧠 **Cel projektu**

Zaprojektuj i zaimplementuj aplikację webową do obsługi przychodni medycznej. Aplikacja powinna umożliwiać:

- zarządzanie pacjentami i ich dokumentacją medyczną,
- rejestrację wizyt z procedurami i lekami,
- przypisywanie wizyt lekarzom,
- prowadzenie notatek klinicznych do wizyt,
- generowanie raportów PDF (np. karta wizyty, recepta),
- dodawanie zdjęć/skanów dokumentów do kartoteki pacjenta,
- filtrowanie i raportowanie udzielonych świadczeń.

Projekt powinien mieć przejrzystą strukturę, modularność, oraz używać nowoczesnych narzędzi: **EF Core**, **Dependency Injection**, **Mapperly**, **Identity**, **OpenAPI**, **Razor Pages, MVC (lub frontend SPA)**.

---

## 🧑‍🤝‍🧑 Zespół
- Zespół: **2 osoby**
- Praca nad repozytorium GitHub (wymagana historia commitów)

---

## ✅ **Wymagania funkcjonalne**

| Moduł                 | Wymagania                                                                 |
|-----------------------|---------------------------------------------------------------------------|
| 🔐 Uwierzytelnianie   | Rejestracja, logowanie (ASP.NET Identity), role: `Admin`, `Lekarz`, `Rejestratorka` |
| 👤 Pacjenci           | CRUD pacjentów, wyszukiwanie (po nazwisku/PESEL), lista wizyt pacjenta   |
| 🪪 Kartoteka          | CRUD kartoteki, upload skanu/zdjęcia dokumentu (np. skierowanie), PESEL, nr ubezpieczenia |
| 📅 Wizyty             | Tworzenie wizyt, statusy (zaplanowana/w trakcie/zakończona/anulowana), przypisywanie lekarza |
| 🩺 Procedury          | Lista procedur medycznych: opis + koszt świadczenia                      |
| 💊 Leki / recepty     | Wybór leków z katalogu, dawkowanie, ilość, koszt                         |
| 📝 Notatki kliniczne  | Notatki wewnętrzne do wizyty (wywiad, rozpoznanie, zalecenia)            |
| 📦 Katalog leków      | CRUD leków, tylko dla `Admin` / `Rejestratorka`                          |
| 📈 Raporty            | Koszt świadczeń danego pacjenta / lekarza / miesiąca + eksport do PDF    |

---

## ✅ **Pozostałe wymagania**

### 🧩 **1. Indeksy – optymalizacja zapytań**

#### 📌 Zadanie:
- Zidentyfikuj **co najmniej dwa zapytania SELECT**, które są często wykonywane i mają **WHERE** lub **JOIN** po kolumnie niekluczowej (np. wyszukiwanie pacjenta po PESEL, lista wizyt lekarza w danym dniu).
- Dodaj **indeksy nieklastrowane (non-clustered)** do wybranych kolumn.
- Zrób analizę wydajności:
  - **Zrzut planu zapytania (Query Plan)** przed i po dodaniu indeksu.
  - Krótkie porównanie (np. liczba odczytów, operacje przeszukiwania vs seek).
  - Umieść to w **raporcie PDF** z opisem + screenshotami.

#### 📎 Plik: `raport-indeksy.pdf`

---

### 📡 **2. SQL Profiler – nasłuch endpointu**

#### 📌 Zadanie:
- Uruchom **SQL Server Profiler (lub EF Core Logging)**.
- Wybierz konkretny **endpoint API** (np. `GET /api/visits/today`).
- Uruchom aplikację → wywołaj endpoint → zrób screenshot z Profilerem pokazującym zapytanie.
- Dodaj screenshoty + opis działania zapytania + krótki komentarz.

#### 📎 Plik: `raport-sql-profiler.pdf`

---

### ⚙️ **3. GitHub Actions – CI/CD**

#### 📌 Zadanie:
- Skonfiguruj workflow z następującymi krokami:
  - build (`dotnet build`)
  - test (`dotnet test`)
  - opcjonalnie: build obrazu Docker
  - opcjonalnie: push do DockerHub (wymaga tokenu)

#### 📎 Plik: `README.md` → opis działania CI/CD  
#### 📎 Plik: `dotnet-ci.yml` w repozytorium

---

### 📝 **4. Logowanie błędów – NLog**

#### 📌 Zadanie:
- Skonfiguruj **NLog** do logowania wyjątków i zdarzeń:
  - logi zapisywane do pliku (np. `/logs/errors.log`)
  - logowanie błędów kontrolerów i serwisów
  - obsługa logowania przez DI (`ILogger<T>`)

---

### 📤 **5. BackgroundService – raport e-mail**

#### 📌 Zadanie:
- Zaimplementuj usługę w tle (`BackgroundService`), która:
  - raz dziennie (lub co 1–2 minuty dla testów) generuje raport z wizyt zaplanowanych na kolejny dzień
  - zapisuje go jako PDF (np. `upcoming_visits.pdf`)
  - wysyła jako załącznik na e-mail administratora przychodni (np. za pomocą SMTP)

#### 📎 Plik: `raport-nadchodzace-wizyty.pdf`  
#### 📎 Klasa: `UpcomingVisitsReportBackgroundService.cs`

---

### 🚀 **6. NBomber – testy wydajności**

#### 📌 Zadanie:
- **Dodaj dodatkowy endpoint API** dedykowany do testów wydajnościowych (np. `GET /api/visits/active`, `GET /api/patients/search?query=...` lub inny endpoint zwracający dane z bazy z JOIN-ami / filtrowaniem).
- Endpoint powinien być udokumentowany w OpenAPI i zwracać realistyczne dane (np. listę aktywnych wizyt z danymi pacjenta i lekarza).
- Skonfiguruj **NBomber** do przetestowania **właśnie tego endpointu**.
- Uruchom test z 50 równoległymi użytkownikami, 100 żądaniami.
- Zapisz **raport PDF z wynikami testu** (czas odpowiedzi, throughput, błędy).

#### 📎 Plik: `nbomber-report.pdf`  
#### 📎 Kod testu: np. `PerformanceTests/VisitsLoadTest.cs`  
#### 📎 Kod endpointu: np. `Controllers/VisitsController.cs`

---

## 🧱 **Modele danych (przykładowe)**

```csharp
class Patient { string Pesel; string InsuranceNumber; ... }
class MedicalRecord { string DocumentScanUrl; ... }
class Visit { Status, AssignedDoctor, List<ProcedurePerformed>, List<ClinicalNote> }
class ProcedurePerformed { Description, ServiceCost, List<PrescribedMedication> }
class Medication { Name, UnitPrice }
class PrescribedMedication { Medication, Dosage, Quantity }
class ClinicalNote { Author, Content, Timestamp }
```

---

## 🛠️ **Wymagania techniczne**

| Obszar                  | Szczegóły                                                                 |
|-------------------------|---------------------------------------------------------------------------|
| **ASP.NET Core**        | Wersja 10 (.NET 10)                                                       |
| **EF Core**             | Code First + migracje - SQL Server                                        |
| **Identity**            | Logowanie, role, autoryzacja                                              |
| **Mapperly**            | Mapowanie DTO ↔️ encje np. Mapperly                                       |
| **DI**                  | Serwisy biznesowe (`IPatientService`, `IVisitService`, ...)               |
| **OpenAPI**             | Dokumentacja API                                                          |
| **Upload plików**       | Skany dokumentów (np. do `/wwwroot/uploads`)                              |
| **PDF**                 | Generowanie raportów jako PDF                                             |
| **Frontend**            | Razor Pages + Bootstrap (opcjonalnie SPA: React/Blazor/Angular)           |
| **Testy**               | testy jednostkowe (xUnit/NUnit)                                           |

---

## 🗂️ **Struktura projektu**

```
/ClinicManager
├── Controllers/
├── DTOs/
├── Models/
├── Services/
├── Mappers/             // Mapperly mappery
├── Views/
├── wwwroot/
│   └── uploads/         // skany dokumentów medycznych
├── Data/
├── Program.cs
```

## ✅ Co należy oddać?

- Repozytorium GitHub z historią commitów
- Działająca aplikacja ASP.NET Core
- Migracje + seed danych (lub dump bazy)
- `README.md` z opisem projektu, logowania, rolami

---

## 📌 Wskazówki

- Wszystkie dane domenowe mapuj za pomocą **Mapperly**
- Używaj **DataAnnotations** do walidacji
- Dbaj o **separację warstw**: logika w serwisach, nie w kontrolerach
- Pamiętaj, że dane medyczne to dane wrażliwe – w komentarzach do projektu warto wspomnieć o **RODO** (np. logowanie dostępu do kartoteki, brak twardego usuwania pacjentów – soft delete)
