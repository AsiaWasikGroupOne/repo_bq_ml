# repo_bq_ml

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

------------------------ gcloud -------------------------------

bq ls --models regerssion
bq ls -m format= pretty regression -> pokaże linear i logistic
bq show -m --format=pretty regression.house_proces2
bq show -m --format=prettyjson regression.house_proces2 - pokaże więcej danych o modelu
bq update --model --description 'shell update' regression.house_proces2 - aktualizacja opisu
bq updete --model --set_label source:shell regression.house_proces2
bq update --model --expiration 432000 regression.house_proces2 (to jest w sekundach)
bq cp --location=US -n dataset1.house_proces2 dataset2.house_proces2 
bq extract --model dataset1.house_proces2 "gs://models_export/from_shell/"



