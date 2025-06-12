# f1_wdc_pred

## 1. Określenie tematu i analiza wymagań

-   **Temat projektu:** Opracowanie modelu regresyjnego do przewidywania końcowej liczby punktów, jaką zdobędzie każdy kierowca Formuły 1 w danym sezonie (WDC - World Drivers' Championship).
-   **Analiza wymagań funkcjonalnych:**
    -   Model musi być zdolny do generowania **predykcji przedsezonowych**, bazując na danych historycznych i składach zespołów na nadchodzący sezon.
    -   Model musi umożliwiać **dynamiczną aktualizację prognoz w trakcie trwania sezonu**, uwzględniając wyniki z każdego kolejnego wyścigu.
    -   System musi być w stanie przetwarzać dane z różnych źródeł i o różnej strukturze (wyniki wyścigów, klasyfikacje generalne, dane historyczne).

## 2. Zbiór danych i ich przygotowanie

-   **Źródła danych:**
    -   **FastF1 API:** Wykorzystane do pobierania szczegółowych danych z sesji wyścigowych, w tym czasów okrążeń i informacji o zdarzeniach.
    -   **Ergast API:** Użyte do pozyskania kompleksowych danych historycznych dotyczących wyników wyścigów oraz klasyfikacji kierowców i konstruktorów z poprzednich sezonów.
-   **Przygotowanie danych (Inżynieria Cech):**
    -   Surowe dane zostały przetworzone w celu stworzenia informatywnych cech dla modelu. Proces ten obejmował:
        -   **Cechy kumulatywne:** Agregujące wyniki w trakcie sezonu (np. `CumulativePoints`, `AvgFinishPos`).
        -   **Cechy kroczące:** Obliczane na podstawie ostatnich 5 wyścigów w celu uchwycenia bieżącej formy kierowcy (np. `PointsLast5`, `FinishRateLast5`).
        -   **Cechy opóźnione:** Przenoszące kluczowe informacje z poprzedniego sezonu (np. `PrevSeasonRank`, `PrevSeasonPoints`).
    -   Przeprowadzono czyszczenie danych, w tym obsługę brakujących wartości (imputację).
    -   Wynikowy, wzbogacony zbiór danych został zapisany do pliku `f1_engineered_features.csv` do dalszego wykorzystania.

## 3. Implementacja modelu AI

-   **Wybór algorytmu:** Zdecydowano się na implementację modelu `XGBoost Regressor` ze względu na jego wysoką skuteczność, szybkość działania oraz zdolność do modelowania złożonych, nieliniowych zależności w danych tabelarycznych.
-   **Przygotowanie danych do modelowania:**
    -   Zastosowano `ColumnTransformer` do rozdzielenia procesu preprocessingu na cechy numeryczne i kategoryczne.
    -   **Cechy numeryczne** zostały przeskalowane za pomocą `MinMaxScaler` do zakresu [0, 1].
    -   **Cechy kategoryczne** (w tym `TeamId`) zostały zakodowane przy użyciu `OneHotEncoder`.
-   **Podział danych:** W procesie walidacji i strojenia wykorzystano `TimeSeriesSplit`, aby zachować chronologiczny porządek danych i zapobiec "wyciekowi informacji z przyszłości", co jest kluczowe w problemach opartych na szeregach czasowych.

## 4. Ocena wyników modelu i optymalizacja

-   **Optymalizacja hiperparametrów:**
    -   Do znalezienia optymalnego zestawu hiperparametrów dla modelu XGBoost wykorzystano mechanizm `RandomizedSearchCV`.
    -   Przeszukiwano przestrzeń kluczowych parametrów, takich jak `learning_rate`, `max_depth`, `subsample`, `colsample_bytree` oraz parametry regularyzacyjne `reg_alpha` i `reg_lambda`.
-   **Metryka oceny:** Proces optymalizacji był ukierunkowany na minimalizację błędu, wykorzystując metrykę `neg_mean_squared_error` (ujemny błąd średniokwadratowy).
-   **Ocena modelu:** Najlepszy model, znaleziony w procesie `RandomizedSearchCV`, został wytrenowany na pełnym zbiorze danych historycznych. Jego skuteczność została zweryfikowana na danych walidacyjnych w ramach walidacji krzyżowej (`TimeSeriesSplit`), co potwierdziło zdolność modelu do generalizacji.

## 5. Zapisanie modelu do pliku i wczytanie go

-   **Serializacja (zapis):**
    -   Wytrenowany i zoptymalizowany model XGBoost został zapisany do pliku `trained_f1_model_xgb_final.joblib`.
    -   Dopasowany obiekt preprocesora (`ColumnTransformer`) został również zapisany do osobnego pliku `fitted_f1_preprocessor_xgb_final.joblib`. Zapisanie preprocesora jest kluczowe dla zapewnienia, że nowe dane będą transformowane w identyczny sposób jak dane treningowe.
-   **Deserializacja (wczytanie):**
    -   Zapisane artefakty (model i preprocessor) są wczytywane w skryptach predykcyjnych.
