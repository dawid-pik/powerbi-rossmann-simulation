# Rossmann Store Sales — Przykładowy Dashboard Analityczno-Symulacyjny (Projekt Portfolio)

Projekt pokazowy typu **Proof of Concept (PoC)**, którego głównym celem jest zaprezentowanie praktycznych umiejętności z zakresu inżynierii danych oraz Business Intelligence. Repozytorium demonstruje czystą architektonicznie ścieżkę wdrożenia rozwiązania analitycznego: od przygotowania i próbkowania danych w środowisku Python, przez budowę zdenormalizowanego modelu w Power BI, aż po zaawansowane analizy scenariuszowe w języku DAX.

---

##  Tech Stack & Wykorzystane Umiejętności

* **Python (Pandas / NumPy):** Proces ETL — ekstrakcja surowych danych (Kaggle), czyszczenie bazy, integracja danych do postaci płaskiej tabeli (flat table) oraz wyznaczenie dni anomalii rynkowych (`Is_Demand_Shock`) za pomocą losowego doboru próby (`np.random.choice`).
* **Power BI Desktop:** Praca na płaskim modelu danych (gwarantującym maksymalną wydajność odświeżania bez obciążania silnika relacjami), optymalizacja pamięciowa oraz stworzenie interaktywnego interfejsu (UI/UX) w czytelnej, menedżerskiej kolorystyce.
* **DAX (Data Analysis Expressions):** Konstrukcja zaawansowanych miar biznesowych (KPI) oraz wdrożenie dynamicznego silnika symulacyjnego typu *What-If*, generującego zmienność losową w czasie rzeczywistym.

---

##  Architektura i Struktura Dashboardu (3 Strony)

Raport został podzielony na trzy funkcjonalne ekrany, prowadzące użytkownika od ogólnego spojrzenia na biznes po szczegółowe modelowanie przyszłości:

### Strona 1: Dashboard Otwierający
* **Cel:** Codzienne monitorowanie kluczowych wskaźników kondycji sieci handlowej.
* **Implementacja:** Zestaw głównych kart KPI (Suma Sprzedaży, Średni Koszyk, Liczba Klientów).
* **Wizualizacje:** Wykresy trendów ukazujące całkowitą sprzedaż w ujęciu miesięcznym (dane historyczne 2013-2015) oraz średnią dzienną sprzedaż per sklep. Dodatkowo wdrożono analizę sezonowości tygodniowej (Suma sprzedaży w podziale na dni tygodnia).

### Strona 2: Top/Flop 10 i Promocja
* **Cel:** Diagnostyka efektywności poszczególnych placówek oraz badanie opłacalności akcji marketingowych.
* **Implementacja:** Rozbudowany panel filtrów pozwalający na izolowanie danych dla konkretnego formatu sklepu (`a`, `b`, `c`, `d`) oraz poziomu asortymentu (Podstawowy, Rozszerzony, Pełny).
* **Wizualizacje:** Bezpośrednie porównanie wpływu aktywnej promocji na Średni Koszyk i Średnią Sprzedaż (wykresy kolumnowe). Szczegółowe tabele rankingowe identyfikujące 10 najlepszych (Top 10) oraz 10 najsłabszych (Flop 10) sklepów pod kątem wolumenu sprzedaży.

### Strona 3: Symulacja Dodatkowe Popytu
* **Cel:** Modelowanie scenariuszowe (Stress Testing) pozwalające na ocenę wpływu skoków popytu na poszczególne formaty sklepów.
* **Implementacja:** Zastosowanie natywnego parametru What-If (suwak `Symulacja_Popytu`), generującego interaktywne przeliczenia. Karty KPI wskazują bazę, sumę symulowaną oraz czysty przyrost (Dodatkowy Przychód).
* **Wizualizacje:** Wykres liniowy śledzący dzienny trend i rozbieżność między sprzedażą faktyczną a symulowaną, zestawienie słupkowe oraz macierz podsumowująca wyniki dla każdego formatu.

---

##  Zaawansowana Implementacja DAX (Silnik Symulacji)

Aby uniknąć błędu podwójnego liczenia (double-counting), środowisko Python posłużyło wyłącznie do losowego oflagowania historycznych dni anomalii (`Is_Demand_Shock = True`). Cała logika matematyczna i stochastyczna została przeniesiona do silnika DAX w Power BI. 

Zamiast płaskich mnożników, miara wprowadza losowy szum rynkowy obliczany w locie:

```dax
Symulowana_Sprzedaż = 
VAR WartośćSuwaka = SELECTEDVALUE('Symulacja_Popytu'[Symulacja_Popytu], 0)
VAR MnożnikPopytu = 1 + WartośćSuwaka
RETURN
    SUMX(
        'rossmann_decision_data (1)',
        IF(
            'rossmann_decision_data (1)'[Is_Demand_Shock] = TRUE(),
            'rossmann_decision_data (1)'[Sales] * MnożnikPopytu * (1 + RAND() * 0.05),
            'rossmann_decision_data (1)'[Sales]
        )
    )
