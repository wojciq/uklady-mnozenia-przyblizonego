# Układy mnożenia przybliżonego dla akceleratorów sztucznej inteligencji

Implementacja sprzętowa oraz weryfikacja 8-bitowych przybliżonych układów mnożących bez znaku, bazujących na **drzewie redukcji Dadda** oraz **kompresorach 4:2**. Projekt zrealizowany w środowisku **Logisim-Evolution**.

## Wprowadzenie i cel projektu

W klasycznym projektowaniu układów arytmetycznych przyjmuje się zazwyczaj równomierny rozkład sygnałów wejściowych. Sytuacja wygląda jednak inaczej w sprzętowych akceleratorach sztucznej inteligencji (np. sieciach neuronowych) oraz cyfrowym przetwarzaniu sygnałów (DSP), gdzie wagi charakteryzują się **rozkładem normalnym (Gaussa)**. Oznacza to, że większość wartości skupiona jest blisko środka przedziału, a najistotniejsze informacje przenoszone są przez bity o niższych wagach.

Projekt realizuje trzy architektury przybliżonych układów mnożących (**PAM1**, **PAM2** oraz **RAM1**), które wykorzystują tę właściwość statystyczną, aby znacząco zredukować złożoność sprzętową, opóźnienie ścieżki krytycznej oraz pobór mocy, przy zachowaniu akceptowalnej dokładności dla obliczeń AI.

## Zaimplementowane architektury

Proces mnożenia został podzielony na trzy hierarchiczne etapy: generowanie macierzy iloczynów częściowych, redukcję iloczynów częściowych oraz końcową akumulację.

1. **RAM1:** Całkowicie odrzuca kolumny iloczynów częściowych o najwyższych wagach w celu maksymalnej minimalizacji liczby bramek logicznych i zajmowanej powierzchni układu.
2. **PAM1:** Zachowuje wszystkie bity wejściowe. Kolumny o niższych wagach są redukowane za pomocą dokładnych kompresorów 4:2, natomiast w kolumnach o wyższych wagach zastosowano kompresory przybliżone, celowo ignorujące sygnały przeniesień międzykolumnowych.
3. **PAM2:** Maksymalizuje wykorzystanie przybliżonych kompresorów 4:2 w drzewie redukcji, aby jeszcze bardziej skrócić ścieżkę krytyczną i uprościć strukturę układu.

*Uwaga projektowa: Podczas odwzorowywania drzew redukcji z publikacji źródłowej zidentyfikowano błąd logiczno-matematyczny w kolumnie $K_1$ (brak elementu kompresującego dla dwóch iloczynów częściowych). W projekcie wprowadzono korektę w postaci dodatkowego modułu półsumatora, przywracając pełną spójność łańcuchów przeniesień.*

## Weryfikacja i analiza błędów

Układy zostały zweryfikowane w dedykowanym środowisku testowym w programie Logisim, porównującym w czasie rzeczywistym wyniki układów przybliżonych z dokładnym 16-bitowym mnożnikiem. Testy przeprowadzono na wektorach dobranych pod kątem rozkładu Gaussa z punktem centralnym wokół wartości 128 dla 8-bitowych liczb bez znaku.

### Kluczowe wnioski

* **Bezbłędność dla potęg liczby 2 PAM1/PAM2:** Mnożenie przez dowolną potęgę dwójki generuje tylko jeden wiersz jedynek w macierzy iloczynów częściowych. Brak nakładających się bitów eliminuje konieczność propagacji przeniesień, dzięki czemu układy zwracają wynik w 100% dokładny.
* **Kompromis oszczędności energii:** Zgodnie z analizą układów referencyjnych, zastąpienie dokładnych kompresorów ich przybliżonymi odpowiednikami pozwala na średnie **skrócenie opóźnienia ścieżki krytycznej o 17%**, **obniżenie poboru mocy o 26%** oraz **poprawę iloczynu opóźnienia i mocy (PDP) o 29%**.
* **Przypadki skrajne:** Dla liczb o dużym zagęszczeniu jedynek w zapisie binarnym (np. `255 x 255`) układy kumulują znaczne błędy przez odrzucone linie przeniesień. Potwierdza to ich ścisłą specjalizację pod kątem danych o rozkładzie normalnym, a nie arytmetyki ogólnego przeznaczenia.

## Struktura repozytorium

* `/circuits` – Pliki projektu z środowiska Logisim-Evolution (`.circ`) zawierające hierarchiczne podukłady - półsumatory, pełne sumatory, dokładne i przybliżone kompresory 4:2 oraz pełne architektury PAM1, PAM2 i RAM1.
* `/docs` – Pełne sprawozdanie techniczne wraz ze schematami drzew redukcji.

## Literatura źródłowa

Projekt bazuje na metodologii opisanej w publikacji:  
*F. Baraati, M. T. Nasab, A. Amirany, K. Jafari, R. Ghaderi, "Efficient and high accuracy approximate multiplier design approach for AI application", Circuits, Systems, and Signal Processing, vol. 44, pp. 8635–8656, 2025.*
