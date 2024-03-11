1. Dodaj konflikt na twojej paczce (tzn. tej, którą piszesz) oznaczającą, że dana wersja musi zostać wybrana (np. `{not my-package 1.0.0}`). Zauważ, że pomimo tego, że mamy właściwie tylko jedną wersję swojej paczki (obecną), to jest to konflikt, a nie przypisanie wersji.
2. Przypisz swoją paczkę do `next`
3. W pętli:
- [Unit propagation](działanie.md#unit-propagation) na `next`, żeby znaleźć kolejne konflikty
	- Jeśli to spowoduje, że jakaś [niezgodność zostanie spełniona](definicje.md#L84), mamy konflikt. Unit propagation spróbuje go rozwiązać - jeśli się to nie uda, zostaje zwrócony error.
- Gdy nie ma już żadnych wyprowadzeń, [podejmij decyzję](działanie.md#decision-making) i ustaw `next` na nazwę paczki, która wybrana przez *decision making*. Weź pod uwagę, że pierwsza decyzja zawsze wybierze jedyną możliwą wersję twojej paczki
	- Podejmowanie decyzji może stwierdzić, że nie ma więcej pracy do wykonania - wtedy częściowe rozwiązanie jest końcowym rozwiązaniem - kończymy algorytm


### Unit propagation

Łączy [partial solution](definicje.md#częściowe-rozwiązanie-partial-solution) ze znanymi [niezgodnościami (incompatibilities)](definicje.md#niezgodność-incompatibility), żeby wyciągnąć nowe [przypisania (assignments)](definicje.md#L91). 
Biorąc jakąś niezgodność `t`, która dla jednego termu jest [nierozstrzygalna (inconclusive)](definicje.md#L32) w całym rozwiązaniu częściowym, oznacza to, że musimy zaprzeczyć `t`. Dodajemy zatem `not t` do częściowego rozwiązania.



Gdy szukamy niezgodności z jednym nierozstrzygalnym termem, możemy natrafić na niezgodność, która jest przez dane częściowe rozwiązanie. 
>[!NOTE]
> Przykład
>==TODO dodać przykład na to==


Jeśli tak się stanie, to wiemy, że obecne częściowe rozwiązanie nie jest w stanie wyprodukować dobrego ogólnego rozwiązania (z [definicji niezgodności (incompatibility)](definicje.md#niezgodność-incompatibility))


Jeśli podczas szukania niezgodności z nierozstrzygalnym termem trafimy na taką, która jest [spełniana](definicje.md#L84) przez obecne częściowe rozwiązanie, to wiemy, że jest złe i robimy [conflict resolution](#conflict-resolution). Zwraca error albo cofa się w częściowym rozwiązaniu i zwraca inną niezgodność, która reprezentuje oryginalną przyczynę konfliktu, np.:

- mamy `{a ^1.0.0, b ^2.0.0}`
- wybieramy w trakcie *unit propagation* `a 1.2.3` i `b 2.3.4` 
- mamy konflikt
- wracamy do `{a ^1.0.0, b ^2.0.0}` i sprawdzamy, co było przyczyną stworzenia takiej niekompatybilności
- ✅ jeśli da się ją zmienić na coś innego, co zaakceptuje wersję `a 1.2.3` i `b 2.3.4`, to zmieniamy
- ❌ jeśli nie, to rzucamy error

Niezgodności są indeksowane po nazwach paczek, do których się odnoszą i iterujemy tylko po tych, które odnoszą się do ostatnio dodanej wersji paczki lub wnioskowań które zostały dodane przy obecnej iteracji unit propagation(?) (*current propagation session*), tzn.
- mamy powiedzmy 3 niezgodności
1. `{a ^1.0.0, b ^2.0.0}`,
2. `{c 1.2.3, d ^5.0.0}`,
3. `{a ^1.4.0, c ^0.5.0}`
- dodajemy wersję paczki `a 1.5.0`
- do paczki `a` odnoszą się niezgodności 1. i 3., więc tylko je rozważamy

**Algorytm *unit propagation*:**
- Przypisz zbiór zawierający nazwę przetwarzanej paczki do 🧺`changed` 
- Dopóki 🧺`changed` nie jest puste:
- Usuń element z 🧺`changed`, przypisz do 📦`package`
- Dla każdego ⚡️`incompatibility`, która odnosi się do 📦`package` od najnowszej do najstarszej (bo [conflict resolution](#conflict-resolution) przeważnie produkuje więcej ogólnych niezgodności później):
	- Jeśli ⚡️`incompatibility` jest [spełnione (satisfies)](definicje.md#L76) przez częściowe rozwiązanie:
		- [conflict resolution](#conflict-resolution) na ⚡️`incompatibility`. Jeśli się uda, zwraca niezgodność, która na pewno jest [prawie spełnialna (almost satisfied)](definicje.md#L80) - nazwijmy ją `term`
		- dodajemy `not term` do częściowego rozwiązania z ⚡️`incompatibility` jako jego przyczynę
		- zamieniamy 🧺`changed` na zbiór zawierający tylko nazwę paczki `term`
	- jeśli ⚡️`incompatibility` jest [prawie spełnione (almost satisfies)](definicje.md#L80):
		- przypisz ten jeden niespełniony term jako `term`
		- dodaj `not term` do częściowego rozwiązania z ⚡️`incompatibility` jako przyczynę
		- dodaj nazwę paczki `term` do 🧺`changed`

### Conflict resolution

Kiedy niezgodność [jest spełniona (satisfies)](definicje.md#L76) przez częściowe rozwiązanie, to oznacza, że obecne częściowe rozwiązanie nie jest podzbiorem głównego rozwiązania - czyli po prostu nie działa. Proces cofania się z tego stanu nazwany jest *conflict resolution (rozwiązywanie konfliktów)*.

Główna zasada w rozwiązywaniu konfliktów to [*rezolucja*](https://pl.wikipedia.org/wiki/Rezolucja_(matematyka)), która w tym przypadku sprowadza się do:

- `a or b` i `not a or c` daje `b or c`

co w przypadku niezgodności to: 
- `{t, q}` i `{not t, r}` -> `{q, r}`^conflict-resolution-rule


Można to zgeneralizować: ^generalized-resolution
- weź dwie niezgodności `{t1, q}` i `{t2, r}` 
- albo `t1` albo `t2` jest spełnione w każdym rozwiązaniu, w którym `t1 ∪ t2` jest spełnione 
- można wywnioskować  z tego `{q, r, t1 ∪ t2}`

Redukujemy to do `{q, r}`w każdym przypadku, gdzie `not t2 ⊆ t1` (to jest, gdzie `not t2` spełnia `t1`), wliczając przypadek gdzie `t1 = t` i `t2 = not t`[(czyli oryginalny przypadek)](działanie.md#L66).

Służy to opisaniu *wcześniejszej przyczyny* - niezgodności o krok bliżej pierwotnej przyczyny. Znajdujemy ją poprzez znalezienie najwcześniejszego przypisania `x`, które w pełni spełnia niezgodność, z którą mamy konflikt. Na `x` oraz na jej przyczynie wykonujemy [[#^generalized-resolution|zgeneralizowaną rezolucję]]. Z tego dostajemy nową niezgodność, która jest naszą *wcześniejszą przyczyną*.

> [!NOTE] Przykład
> ==TODO==

W ten sposób jesteśmy w stanie znaleźć główną (*root*) przyczynę wykonując tę procedurę aż:
- satysfakcjonujące przypisanie jest decyzją
- jedynym przypisaniem na swoim poziomie decyzji, który jest istotny dla tego konfliktu

==o co tu chodzi==
W pierwszym przypadku nie ma przyczyny, w drugim cofnęliśmy się na tyle daleko, że możemy backtrakować częściowe rozwiązanie i być pewni, że wyprowadzimy nowe przypisania.

Algorytm (w pętli):
- if niezgodność ❌`incompatibility`:
	- nie zawiera żadnych termów
	- zawiera pojedynczy term, który odnosi się do wersji głównej paczki
	- rzuć error z ❌`incompatibility` jako główną niezgodność
- znajdź najwcześniejsze przypisanie w częściowym rozwiązaniu takie, że ❌`incompatibility` jest spełnione przez częściowe rozwiązanie aż do tego przypisania (je wliczając). Przypisz je do `satisfier` i nazwij term w ❌`incompatibility`, który się do niego odnosi jako `term`
- znajdź najwcześniejsze przypisanie w częściowym rozwiązaniu **przed** `satisfier` takie, że ❌`incompatibility` jest spełnione przez częściowe rozwiązanie aż do tego przypisania (je wliczajac) oraz `satisfier`. Przypisz do `previousSatisfier`
> [!NOTE] Uwaga
> `satisfier` może sam nie spełnić `term`, np. jeśli `term` to `foo >=1.0.0 <2.0.0`, to może być spełniony przez `{foo >=1.0.0, foo <2.0.0}`, ale nie przez każde przypisanie osobno. W takim przypadku `previousSatisfier` może odnosić się do tej samej paczki co `satisfier`.

- `previousSatisfierLevel` = poziom decyzji `satisfierLevel` lub 1, jeśli nie ma `previousSatisfier`
> [!NOTE] Uwaga
> Poziom decyzji 1 to poziom, na którym główna (*root*) paczka została wybrana. Bezpieczniej wrócić do poziomu 0, ale zatrzymywanie się na poziomie 1 zwykle generuje lepsze error message, bo referencje do głównej paczki są bliżej wniosku, że nie istnieje żadne rozwiązanie.

- if `satisfier` jest decyzją || `previousSatisfierLevel` jest inny, niż poziom decyzji `satisfier`:
	- jeśli `incompatibility` jest inne, niż początkowy input - dodaj je do *solver's incompatibility set* (jeśli konfliktująca niezgodność była dodana leniwie podczas [podejmowania decyzji](#decision-making), może nie mieć wyraźnej/jednoznacznej pierwotnej przyczyny)
	- backtrack - usuwaj wszystkie przypisania, których *decision level* jest większy, niż `previousSatisfierLevel` z częściowego rozwiązania
	- zwróć `incompatibility`
- else, `priorCause` to suma mnogościowa termów w niezgodności i termów w przyczynie `satisfier` minut termy odnoszące się do paczki `satisfier`
>[!NOTE] Uwaga
Odpowiada to wywnioskowanej niezgodności `{q, r}` z [przykładu wyżej](działanie.md#L66)
- if `satisfier` nie spełnia `term`, dodaj `not (satisfier \ term)` do `priorCause`
> [!NOTE] Uwaga
> `not (satisfier \ term)` odpowiada `t1 ∪ t2` w [zgereralizowanej zasadzie](działanie.md#L69), gdzie `term = t1` i `satisfier = not t2`, z własności `(Sᶜ \ T)ᶜ = S ∪ T`.
- przypisz `incompatibility` do `priorCause`

### Decision making
Proces wybierania wersji paczki licząc na to, że będzie częścią głównego  rozwiązania oraz zapewniając, że jej dependencje będą dobrze obsłużone. Jest trochę elastyczności pod tym względem, wersja, która spełnia poniższe kryteria jest dobra:
- częściowe rozwiązanie ma pozytywne wyprowadzenie dla tej paczki
- częściowe rozwiązanie nie zawiera decyzji dla tej paczki (czyli jej po prostu jeszcze nie wybrano)
- wersja paczki spełnia wszystkie przypisania w częściowym rozwiązaniu
<br>

Algorytm zaczyna wybierać najnowsze wersje paczek, które dla danych ograniczeń (czyli obecnych niezgodności) mają najmniej możliwych dostępnych wersji, np. 
- paczka A ma dla danych ograniczeń 3 możliwe wersje do wyboru
- paczka B ma 6 wersji do wyboru

Algorytm w takim wypadku wybierze najnowszą wersję paczki A. [^1]
<br>
Częścią procesu jest zamiana dependencji paczki na niezgodności - robione to jest leniwie gdy każda wersja paczki jest dodana [^2]
<br>
Jeśli `foo 1.0.0 depends on bar ^1.0.0` i `foo 1.1.0 depends on bar ^1.0.0`, to są one zbijane do jednej niezgodności - `{foo ^1.0.0, not bar ^1.0.0}` [^3]
<br>
Przedziały wersji paczek zależnych od czegoś (`foo` w przypadku wyżej) są zawsze inkluzywne od dołu na pierwszą wersję paczki, która **ma** daną depkę i eksluzywne na pierwszą wersję paczki, która **nie ma** tej dependencji. [^4]
- jeśli najnowsza wersja ma tę depkę, górna granica jest pomijana (np. `foo >=1.0.0`)
- jeśli pierwsza wersja jej nie ma, dolna granica jest pomijana (np. `foo <2.0.0`)
<br>

Jeśli wersja paczki nie może być w ogóle wybrana (bo np. jest niezgodna z obecną wersją używanego języka progamowania), to nie dodajemy w ogóle jej dependencji. Dodajemy natomiast niezgodność symbolizującą, że dana wersja paczki (oraz każde wersje obok, które nie spełniają założeń) nie powinny być wybrane. [^5]
<br>

Algos: 
- Przypisz do 📦`package` paczkę z pozytywnym wyprowadzeniem, ale bez decyzji w obecnym częściowym rozwiązaniu
- Przypisz do `term` przecięcie (*intersection*, `∩`) wszystkich przypisań w częściowym rozwiązaniu odnoszących się do 📦`package`
- Przypisz do `version` wersję 📦`package`, która spełnia `term`
- Jeśli nie ma takiego `version`:
	- dodaj niezgodność `{term}` do zbioru niezgodności i zwróć nazwę paczki 📦`package` (to daje informację algorytmowi do unikania tego zakresu wersji w przyszłości)
- Dodaj każde `incompatibility` z dependencji `version` do zbioru niezgodności jeśli ich jeszcze tam nie ma
- Dodaj `version` do częściowego rozwiązania jako *decyzję*, chyba, że wyprodukuje to konflikt z jakąś z nowych niezgodności
- Zwróć nazwę paczki 📦`package`

### Error reporting 
Zwracanie błędów dlaczego się nie udało trudne z tego samego powodu, dla którego komputer nie da rady łatwo stwierdzić, czy *version solving* się uda, czy nie.

Struktura algorytmu ułatwia wyjaśnianie najbardziej zawiłych przypadków dzięki śledzeniu głównych przyczyn niezgodności (*root-cause tracking*) - **ponieważ wyprowadza nowe niezgodności za każdym razem, gdy natrafi na konflikt, to automatycznie generuje łańcuch wyprowadzeń, który ostatecznie wyprowadza to, że rozwiązanie nie istnieje.**

Gdy [conflict resolution](#conflict-resolution) zawodzi, tworzy niezgodność z jednym pozytywnym termem - główną paczką - główna paczka nie jest częścią rozwiązania - nie ma rozwiązania. Do pokazania dlaczego stosujemy [graf wnioskowania](definicje.md#graf-wnioskowania-derivation-graph).

Można łatwo z niego wyciągnąć przyczyny każdej niezgodności, ale może to prowadzić do zbyt rozwlekłych informacji, np.:
> ... And, because `root` depends on `foo ^1.0.0`, `root` requires `baz ^3.0.0`. So, because `root` depends on `baz ^1.0.0`, `root` isn't valid and version solving has failed.

można zamienić na:

> ... So, because `root` depends on both `foo ^1.0.0` and `baz ^3.0.0`, `root` isn't valid and version solving has failed.

Dodatkowo, grafy mogą być bardziej skomplikowane, np. wywnioskowana niezgodność może być spowodowana wieloma niezgodnościami, które też były wywnioskowane itd., np.:
```
┌───┐ ┌───┐ ┌───┐ ┌───┐
│   │ │   │ │   │ │   │
└─┬─┘ └─┬─┘ └─┬─┘ └─┬─┘
  └▶┐ ┌◀┘     └▶┐ ┌◀┘
   ┌┴─┴┐       ┌┴─┴┐
   │   │       │   │
   └─┬─┘       └─┬─┘
     └──▶─┐ ┌─◀──┘
         ┌┴─┴┐
         │   │
         └───┘
```

albo

```
    ┌───┐ ┌───┐
    │   │ │   │
    └─┬─┘ └─┬─┘
      └▶┐ ┌◀┘
┌───┐  ┌┴─┴┐  ┌───┐
│   │  │   │  │   │
└─┬─┘  └┬─┬┘  └─┬─┘
  └▶┐ ┌◀┘ └▶┐ ┌◀┘
   ┌┴─┴┐   ┌┴─┴┐
   │   │   │   │
   └─┬─┘   └─┬─┘
     └─▶┐ ┌◀─┘
       ┌┴─┴┐
       │   │
       └───┘
```
[^6]

**Przed odpaleniem algorytmu przejdź po grafie wyprowadzeń i zapisz ile wychodzących krawędzi ma każda z niezgodności - czyli ile różnych niezgodności powoduje**

Algorytm bierze jako input wyprowadzone `incompatibility` i wypisuje linie outputu (które mogą mieć przypisane sobie numerki), np.:
- *Because `foo <1.1.0` depends on `a ^1.0.0` which depends on `b ^2.0.0`, `foo <1.1.0` requires `b ^2.0.0`.*
- *So, because `foo <1.1.0` depends on `b ^1.0.0`, `foo <1.1.0` is forbidden. (1)* - numerek
- *And because `foo <1.1.0` is forbidden (1), `foo` is forbidden.* - odniesienie do numerka

Każda linia opisuje *jedną wywnioskowaną niezgodność* i tłumaczy, dlaczego zachodzi.
<br>

Działanie algorytmu:
1. Jeśli `incompatibility` spowodowana przez dwie inne wywnioskowane niezgodności `cause1`, `cause2`:
	- 1️⃣ *if* obydwie przyczyny mają przypisane sobie numery linii:
		- Wypisz *Because `cause1` (`cause1.line`) and `cause2` (`cause2.line`), `incompatibility`.*
	- 2️⃣ *else if* tylko jedna przyczyna ma przypisany numer linii:
		- Rekurencyjnie wywołaj [Error reporting](działanie.md#error-reporting-) (czyli ten algorytm) na przyczynie bez numeru
		- przyczyna z numerem = `cause`
		- Wypisz "*And because `cause` (`cause.line`), `incompatibility`.*"
	- 3️⃣ *else* (`cause1` i `cause2` nie mają numerów linii):
		- 🅰️ *if* przynajmniej jedna z niezgodności jest spowodowana dwoma [zewnętrznymi (external) niezgodnościami](definicje.md#L66):
		- 🅱️


[^1]: Zwykle pomaga to znaleźć konflikty wcześniej, bo takie paczki szybciej wyczerpią swoje możliwe wersje
[^2]: Żeby zapobiec zalania solvera niezgodnościami, które prawdopodobnie będą nieistotne.
[^3]: To znacznie zmniejsza całkowitą liczbę niezgodności i mocno ułatwia algorytmowi rozważanie wielu wersji pakietów na raz.
[^4]: Expanding the version range in this way makes it more closely match the format users tend to use when authoring dependencies, which makes it easier for Pubgrub to reason efficiently about the relationship between dependers and the packages they depend on.
[^5]: Więcej info tutaj: https://github.com/dart-lang/pub/blob/master/lib/src/solver/package_lister.dart
[^6]: In these cases, a naïvely linear explanation won't be clear. We need to refer to previous derivations that may not be physically nearby. We use line numbers to do this, but we only number incompatibilities that we _know_ will need to be referred to later on. In the simple linear case, we don't include line numbers at all.
