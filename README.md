# ZUM-projekt

### Szczegółowa interpretacja projektu
Projekt polega na analizie modeli klasyfikujących, pozwalających wykrywać ataki na sieć pośród normalnego ruchu. Na podstawie danych dostarczonych z KDD Cup 1999 oraz NSL-KDD trenowane zostaną modele stworzone za pomocą algorytmów nadzorowanego i nie nadzorowanego uczenia. Następnie ich skuteczność zostanie zbadana na zbiorze testującym i porównane między sobą.

### Charakterystyka zbiorów danych
Do realizacji projektu zamierzamy wykorzystać zbiór danych KDD CUP 99 i jego poprawioną wersję z 2009 roku. W poniższym opisie  skupimy się na scharakteryzowaniu pierwszego z wymienionych. Zbiór treningowy zawiera około 5 milionów zapisów połączeń przechwyconych przez analizator ruchu sieciowego. Milion przypadków zostało w nim sklasyfikowanych jako nieszkodliwy ruch normalny. Pozostałym przyporządkowano konkretny atak sieciowego. Ataki te można dodatkowo podzielić pomiędzy 4 kategorie ataków.
+ DoS (Denial of Service) – jest to kategoria ataków, których celem jest zakłócenia działania atakowanego systemu. Mogą one polegać na:
  - wysyłaniu ogromnej liczby odpowiednio spreparowanych pakietów ICMP (Smurf);
  - rozpoczęcia ogromnej liczby połączeń wyczerpujących zasoby serwera (Neptune);
  - wysyłaniu zniekształconych pakietów, których serwer nie jest w stanie poprawnie odczytać (Teardrop, Pod – Ping of death, Back);
  - wywołaniu nieskończonej pętli na podatnej maszynie (Land – Local area network denial).
+ R2L (Remote to Local) – celem tej kategorii ataków jest uzyskanie nieuprawnionego dostępu do serwera. Mogą one polegać na:
  - próbie zgadnięcia loginu i hasła użytkownika (guess_password, Imap);
  - wykorzystanie podatności w systemie przesyłania plików (warezmaster, warezclient, Ftp_write);
  - wykorzystania podatności w interfejsie CGI (phf)
+ U2R (User to Root) – ataki te mają na celu eskalację uprawnień użytkownika na atakowanym systemie przy założeniu, że atakujący uzyskał już lokalny dostęp do systemu. Mogą one polegać na:
  - przepełnieniu bufora i wykonaniu arbitralnego kodu na serwerze (Buffer overflow)
  - wykorzystaniu podatności sterowników (Loadmodule)
  - wykorzystaniu innych podatności występującej w systemie operacyjnym (Rootkit)
  - wykorzystaniu podatności występującej w zainstalowanych programach (Perl)

+ Probe - nie są to ataki jako takie, lecz jest to ruch pochodzący ze skanów mających ujawnić dodatkowe informacje o systemie, mogące być przydatne potencjalnemu atakującemu do przeprowadzenia bardziej wyrafinowanych ataków. Przykładami takiego ruchu sieciowego są IP sweep, nmap, portsweep i satan.

W zbiorze testowym jest dodatkowo 14 przypadków ataków, które nie występują w zbiorze treningowym. Z tego powodu uznaliśmy, że jest to zbiór danych odpowiedni do zadania związanego z detekcją anomalii. Odpowiednio wytrenowany model powinien być w stanie nie tylko wykrywać ataki ze zbioru treningowego, ale również ich nowe rodzaje.
W projekcie chcielibyśmy skoncentrować się na wykrywaniu ataków typu U2R i R2L. Ze względu na swoją charakterystykę ataki typu Denial of Service są bardzo liczne w zbiorze danych. W przeciwieństwie do powyższych typów ataków ataki U2R i R2L są bardzo nieliczne – niektóre ataki występują zaledwie kilka razy, zaś w sumie liczba wszystkich anomalnych połączeń jest rzędu kilku tysięcy. Dlatego lepiej nadają się one do zgłębiania zagadnienia detekcji anomalii.
W pojedynczym wektorze danych można wyróżnić 3 kategorie atrybutów:
+ podstawowe atrybuty połączenia, takie jak użyty protokół i usługa czy rozmiar danych przesłanych w trakcie połączenia.
+ atrybuty dotyczące przesyłanej zawartości z punktu widzenia bezpieczeństwa, czyli przykładowo liczba nieudanych logowań, czy informacja o wykorzystaniu użytkownika z uprawnieniami administratora systemu. Te atrybuty będą szczególnie ważne do rozpoznawania ataków U2R i R2L.
+ atrybuty połączenia, przykładowo liczba połączeń do tego samego serwera w ciągu ostatnich dwóch sekund, czy procent prób połączeń, które zakończyły się błędem. Spodziewamy się, że te atrybuty mają największą wagę przy wykrywaniu skanów i ataków typu Denial of Service.


### Dwie klasy labeli oryginalnych:
+ Nie atak:
  + normal
+ Atak:
  + buffer_overflow
  + loadmodule
  + perl
  + neptune
  + smurf
  + guess_passwd
  + podteardrop
  + portsweep
  + ipsweep
  + land
  + ftp_write
  + back
  + imap
  + satan
  + phf
  + nmap
  + multihop
  + warezmaster
  + warezclient
  + spy
  + rootkit


Dane rozkład częstotliwościowy we wszystkich danych:

|1|back.|2203|
|2|buffer_overflow.|30|
|3|ftp_write.|8|
|4|guess_passwd.|53|
|5|imap.|12|
|6|ipsweep.|12481|
|7|land.|21|
|8|loadmodule.|9|
|9|multihop.|7|
|10|neptune.|1072017|
|11|nmap.|2316|
|12|normal.|972781|
|13|perl.|3|
|14|phf.|4|
|15|pod.|264|
|16|portsweep.|10413|
|17|rootkit.|10|
|18|satan.|15892|
|19|smurf.|2807886|
|20|spy.|2|
|21|teardrop.|979|
|22|warezclient.|1020|
|23|warezmaster.|20|

Dane rozkład częstotliwościowy w zbiorze posiadającym 10% danych:

|1|back.|2203|
|2|buffer_overflow.|30|
|3|ftp_write.|8|
|4|guess_passwd.|53|
|5|imap.|12|
|6|ipsweep.|1247|
|7|land.|21|
|8|loadmodule.|9|
|9|multihop.|7|
|10|neptune.|107201|
|11|nmap.|231|
|12|normal.|97278|
|13|perl.|3|
|14|phf.|4|
|15|pod.|264|
|16|portsweep.|1040|
|17|rootkit.|10|
|18|satan.|1589|
|19|smurf.|280790|
|20|spy.|2|
|21|teardrop.|979|
|22|warezclient.|1020|
|23|warezmaster.|20|


### Opis algorytmów, które będą wykorzystywane do badań
+ Pierwsza kategoria nadzorowanych danych:
  + Drzewa decyzyjne
  + Naiwny klasyfikator Bayesowski
  + Las losowy
+ Druga kategoria nienadzorowanych danych:
  + KNN - algorytm centroidalny
  + DBSCAN - algorytm gęstościowy

### Parametry algorytmów, których wpływ na wyniki będzie badany
+ Las losowy:
  + kryterium stopu
  + liczba drzew
+ Drzewo decyzyjne:
  + kryterium stopu
  + głębokość drzewa
+ Naiwny klasyfikator bayesowski:
  + brak konieczności strojenia parametrów
+ KNN:
  + liczba sąsiedztw
+ DBSCAN:
  + epsilon - wielkość okolicy, którą uznajemy za sąsiedztwo
  + minPts - minimalna ilość punktów do uznania okolicy za grupę
  + borderPoints - czy bierzemy pod uwagę punkty graniczne

### Miary oceny oparte o tablice pomyłek:
+ Precyzja precision
+ Odzysk - recall
+ miara F
+ czułość - sensitivity
+ specyficzność - specivity
+ Procedury oceny modeli: Analiza ROC

### Cel poszczególnych eksperymentów:
Eksperymenty mają za zadanie analizę porównawczą modeli uczenia nadzorowanego i nienadzorowanego na podstawie danych dotyczących ataków sieciowych z lat 1999 oraz 2009. Celem analizy jest określenie najlepszego modelu dla zadania detekcji anomalii, pozwalającego wykryć zarówno typy ataków przedstawione w danych treningowych oraz te nie spotkane przez model, które są dostępne tylko w danych testowych.
