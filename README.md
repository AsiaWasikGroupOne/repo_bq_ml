# repo_bq_ml


------------------------ Typy modeli ------------------------------------------

1. **Linear Regression** (Regresja liniowa)Czym się zajmuje: Przewidywaniem konkretnej liczby (wartości ciągłej). Szuka linii prostej, która najlepiej oddaje zależność między danymi (np. "im większy metraż, tym wyższa cena").W skrócie: Łączy kropki linią prostą, aby przewidzieć przyszłe wartości.Przykłady zastosowania:Przewidywanie ceny mieszkania na podstawie liczby pokoi i lokalizacji.Szacowanie przychodów sklepu na podstawie budżetu wydanego na reklamy.

2. **Logistic Regression** (Regresja logistyczna)Czym się zajmuje: Przypisywaniem danych do jednej z kategorii (klasyfikacja) – najczęściej binarnej (Tak/Nie, 1/0). Mimo słowa "regresja" w nazwie, to model do klasyfikacji! Zamiast linii prostej używa krzywej w kształcie litery "S" (funkcji logistycznej), która wskazuje prawdopodobieństwo od 0 do 1.W skrócie: Oblicza szansę (w procentach) na to, czy coś się wydarzy.Przykłady zastosowania:Czy transakcja kartą to oszustwo? (Tak / Nie).Czy klient zrezygnuje z subskrypcji w tym miesiącu? (Tak / Nie).

3.** K-means** (K-średnich)Czym się zajmuje: Grupowaniem (segmentacją) danych, które nie mają gotowych etykiet (tzw. uczenie nienadzorowane). Algorytm analizuje cechy obiektów i sam "wrzuca" je do $K$ podobnych do siebie worków (klastrów).W skrócie: Szuka ukrytych wzorców i dzieli dane na grupy "podobne do siebie".Przykłady zastosowania:Podział klientów e-sklepu na grupy (np. "łowcy okazji", "klienci premium", "przypadkowi kupujący") do kampanii marketingowych.Grupowanie artykułów newsowych na kategorie tematyczne bez ręcznego ich tagowania.

4. **Boosted Trees** (Drzewa wzmacniane / np. XGBoost)Czym się zajmuje: Wysoce dokładną klasyfikacją lub regresją na danych tabelarycznych. Łączy setki małych, prostych drzew decyzyjnych w jeden potężny łańcuch, gdzie każde kolejne drzewo poprawia błędy swoich poprzedników.W skrócie: Najmocniejszy algorytm do klasycznych baz danych i Excela; potrafi wyłapać bardzo skomplikowane zależności.Przykłady zastosowania:Ocena ryzyka kredytowego w banku (bardzo precyzyjna decyzja: przyznać kredyt czy odrzucić?).Przewidywanie awarii maszyn w fabryce na podstawie setek czujników.

5. **Deep Neural Network** (DNN - Głębokie sieci neuronowe)Czym się zajmuje: Przetwarzaniem danych o ogromnym stopniu skomplikowania, szczególnie danych nieustrukturyzowanych (obrazy, dźwięk, tekst). Składa się z wielu warstw sztucznych neuronów, które potrafią same uczyć się cech obiektów (np. rozpoznawania krawędzi, kształtów, aż po całą twarz).W skrócie: Naśladuje ludzki mózg do zadań, z którymi tradycyjne tabele sobie nie radzą.Przykłady zastosowania:Rozpoznawanie twarzy na zdjęciach lub obiektów na drodze przez autonomiczne samochody.Tłumaczenie tekstów w czasie rzeczywistym (np. Google Translate) czy generowanie tekstu (LLM).

6. **PCA** (Principal Component Analysis / Analiza głównych składowych)Czym się zajmuje: Redukcją wymiarowości. Kiedy masz tabelę, która ma np. 500 kolumn (cech), model staje się zbyt ciężki i wolny. PCA "kompresuje" te 500 kolumn do np. 5 najważniejszych składowych, starając się zachować jak najwięcej kluczowych informacji z oryginalnego zbioru.W skrócie: To "Zipped/RAR" dla Twoich danych – zmniejsza wagę i skomplikowanie tabeli przed właściwym treningiem.Przykłady zastosowania:Zmniejszanie rozdzielczości zdjęć (z zachowaniem widocznych kształtów) przed wrzuceniem ich do sieci neuronowej.Upraszczanie gigantycznych kwestionariuszy medycznych zawierających setki pytań do kilku głównych czynników zdrowotnych.

7. **ARIMA** (Autoregressive Integrated Moving Average)Czym się zajmuje: Prognozowaniem szeregów czasowych (Time Series). Analizuje wyłącznie dane ułożone chronologicznie krok po kroku i przewiduje, co wydarzy się w przyszłości, biorąc pod uwagę trendy, sezonowość (np. wzrosty co weekend) oraz historyczne wahania.W skrócie: Patrzy w kalendarz i zegarek, żeby przewidzieć przyszłość na osi czasu.Przykłady zastosowania:Prognozowanie pogody na najbliższy tydzień.Przewidywanie, ile sztuk towaru (np. lodów) magazyn musi zamówić na kolejny miesiąc na podstawie sprzedaży z ostatnich 5 lat.
   

----------------------- Kroli tworzenia modelu ------------------------------------------------------------------------------

1. TWORZENIE MODELU (CREATE MODEL)
Ta komenda uruchamia proces trenowania. Kluczowe jest ustawienie model_type='logistic_reg' oraz wskazanie kolumny, którą model ma przewidzieć (input_label_cols).

SQL
**CREATE OR REPLACE MODEL** `moj_projekt.moj_dataset.model_regresji_logistycznej`
OPTIONS(
  model_type='logistic_reg',        -- Definiuje regresję logistyczną
  input_label_cols=['czy_zakup']    -- Nazwa kolumny z etykietą (0 lub 1 / TAK lub NIE)
) AS
SELECT 
  wiek, 
  zarobki, 
  kraj, 
  czy_zakup 
FROM 
  `moj_projekt.moj_dataset.dane_treningowe`;
2. OCENA JAKOŚCI MODELU (ML.EVALUATE)
Po zakończeniu treningu musisz sprawdzić wskaźniki (te, o które pytałaś wcześniej: Precision, Recall, F1 score). Używasz do tego funkcji ML.EVALUATE, przekazując jej model oraz dane testowe.

SQL
SELECT 
  * FROM 
  **ML.EVALUATE**(
    MODEL `moj_projekt.moj_dataset.model_regresji_logistycznej`,
    (
      SELECT wiek, zarobki, kraj, czy_zakup 
      FROM `moj_projekt.moj_dataset.dane_testowe`
    )
  );
Wskazówka: Jeśli nie podasz podzapytania z nowymi danymi, BQML automatycznie oceni model na podstawie danych, które odłożył sobie podczas treningu (tzw. holdout data).

3. PREDYKCJA / PROGNOZOWANIE (ML.PREDICT)
Gdy model jest już gotowy i sprawdzony, używasz go do przewidywania zachowań nowych klientów. ML.PREDICT zwróci Twoje oryginalne kolumny oraz nowe, wygenerowane przez model (m.in. predicted_czy_zakup oraz prawdopodobieństwo).

SQL
`SELECT 
  * FROM 
  **ML.PREDICT**(
    MODEL `moj_projekt.moj_dataset.model_regresji_logistycznej`,
    (
      SELECT wiek, zarobki, kraj 
      FROM `moj_projekt.moj_dataset.nowi_klienci`
    )
  );`
💡 Ważne szczegóły na egzamin (Data Engineer):
Automatyczne kodowanie tekstów: Regresja logistyczna wymaga liczb. Jeśli w danych masz kolumnę tekstową (np. kraj='Polska'), BQML automatycznie zrobi dla niej tzw. One-Hot Encoding (zamieni tekst na wektory liczbowe). Nie musisz robić tego ręcznie w SQL!

Wersjonowanie: Użycie CREATE OR REPLACE MODEL nadpisze stary model. Jeśli chcesz zachować historię, musisz zmieniać nazwę modelu (np. dodając datę na końcu).


--------------------- Najważniejsze wskaźniki Evaluate -------------------------------------------------------------------------

1. Threshold (Próg decyzyjny) = 0.5000
Model rzadko mówi "to na 100% jest spam". Zazwyczaj wylicza prawdopodobieństwo, np. 65%.

Próg 0.5 oznacza granicę: jeśli model jest pewien na ponad 50%, że mail to spam, oznacza go jako spam. Jeśli poniżej 50% – przepuszcza go do skrzynki odbiorczej.

2. Precision (Precyzja) = 0.9618 (96.18%)
Odpowiada na pytanie: "Skoro model oznaczył maile jako spam, to ile z nich faktycznie nim było?".

Twój wynik jest świetny. Oznacza to, że jeśli model wrzuci 100 maili do folderu SPAM, to około 96 z nich to realny spam, a tylko 4 to ważne wiadomości (tzw. fałszywe alarmy / False Positives), które tam utknęły.

3. Recall (Czułość) = 1.0000 (100%)
Odpowiada na pytanie: "Ile z całego spamu, który w ogóle do nas szedł, model zdołał wyłapać?".

Wynik 1.0 (100%) oznacza absolutną perfekcję w tej dziedzinie. Model wyłapał każdy możliwy spam, który zmierzał do skrzynki. Żaden śmieć nie przedostał się do wiadomości odebranych.

4. Accuracy (Dokładność ogólna) = 0.9618 (96.18%)
Odpowiada na pytanie: "Ile ogólnie decyzji modelu (zarówno 'to spam', jak i 'to nie spam') było prawidłowych?".

Model ma rację w 96.18% wszystkich przypadków.

Uwaga egzaminacyjna (Data Engineer): Ta metryka bywa zdradliwa przy danych niezbalansowanych. Gdybyś miała 96 spamu i 4 dobre maile, model, który zawsze mówi "to spam", miałby 96% Accuracy, będąc bezużytecznym.

5. F1 Score = 0.9805
To średnia harmoniczna z Precision i Recall. Łączy obie te metryki w jedną liczbę.

Często między Precision a Recall trwa przeciąganie liny (gdy jedno rośnie, drugie spada). F1 Score daje Ci znać, czy model zachowuje zdrowy balans. Wynik 0.98 jest potężny i oznacza, że model jest niezwykle stabilny i dokładny.

6. Log Loss = 0.1623
Mierzy "pewność siebie" modelu w jego błędach. Tutaj im mniejsza liczba (bliższa 0), tym lepiej.

Jeśli model pomyli się, będąc pewnym swojej decyzji na 99%, Log Loss mocno go ukarze. Wynik 0.16 jest bardzo niski, co oznacza, że model rzadko zalicza tak potężne, "pewne siebie" wpadki.

7. ROC AUC = 0.0000
Mierzy ogólną zdolność modelu do rozróżniania klas (spamu od nie-spamu) przy różnych progach decyzyjnych (nie tylko 0.5). Idealny model ma AUC = 1.0, a losowy rzut monetą to 0.5.

ALERT: Wynik 0.0000 w prawdziwym życiu oznacza błąd! Świadczy to o tym, że albo model został doskonale odwrócony (przewiduje dokładnie na odwrót – klasyfikuje spam jako dobre maile, a dobre jako spam), albo w danych testowych/walidacyjnych zabrakło jednej z klas (np. przekazałaś do ewaluacji same maile ze spamem, przez co BQML nie był w stanie wyliczyć krzywej ROC).

------------------------------------------------ gcloud ---------------------------------------------------------------------------

bq ls --models regerssion
bq ls -m format= pretty regression -> pokaże linear i logistic
bq show -m --format=pretty regression.house_proces2
bq show -m --format=prettyjson regression.house_proces2 - pokaże więcej danych o modelu
bq update --model --description 'shell update' regression.house_proces2 - aktualizacja opisu
bq updete --model --set_label source:shell regression.house_proces2
bq update --model --expiration 432000 regression.house_proces2 (to jest w sekundach)
bq cp --location=US -n dataset1.house_proces2 dataset2.house_proces2 
bq extract --model dataset1.house_proces2 "gs://models_export/from_shell/"
bq rm -f --model dataset1.house_proces2


