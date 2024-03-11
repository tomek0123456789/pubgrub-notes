##### term

Stwierdzenie dotyczące paczki (z którym możemy się spotkać w [`Cargo.toml`](https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html#specifying-dependencies-from-cratesio) czy [`Scarb.toml`](https://docs.swmansion.com/scarb/docs/reference/specifying-dependencies.html#version-requirements)), które może być **spełnione lub nie** przez jakąś wybraną wersję paczki.

Przykłady `term`ów:
1. `bar ^1.0.0`
2. `baz >= 1.0.0, < 2.0.0`
3. `not bar ^1.0.0`


Dla przykładu pierwszego (`bar ^1.0.0`, po rozwinięciu semantyki `^` `bar >=1.0.0, <2.0.0`)
- spełnieniem `term`u jest wybranie wersji paczki `bar 1.2.3` (bo pomiędzy dolną i górną granicą),
- niespełnieniem jest `bar 2.3.4` (bo `2.3.4` nie spełnia `<2.0.0`)
Dla przykładu drugiego (`not bar ^1.0.0`)
- spełnieniem `term`u jest wybranie wersji paczki `bar 2.3.4` lub ==**niewybranie żadnej wersji paczki**==[^1]
- niespełnieniem jest `bar 1.2.3`

Dodatkowo, mamy 3 zasady:

1. Zbiór termów $S$ **spełnia** (satisfies) term $t$ gdy $t$ jest spełnione gdy każdy term w $S$ jest spełniony. 
>[!EXAMPLE] Przykład 
   > zbiór $S$ `{foo >=1.0.0, foo <2.0.0}` spełnia term $t$ `foo ^1.0.0` ^term-satisfies

2. Zbiór termów $S$ **nie spełnia termu** (contradicts) $t$, gdy każdy term w $S$ jest spełniony, a $t$ jest niespełnione ^term-contradicts
>[!EXAMPLE] Przykład 
   > 
   > zbiór $S$ `foo ^1.5.0` (inaczej `{foo ^1.5.0}`) nie spełnia termu $t$ `not foo ^1.0.0` 
   > 
    Każda wersja spełniająca $S$, czyli wersje `{foo >=1.5.0, <2.0.0}` są sprzeczne z wersjami $t$ (`foo <1.0.0` lub niewybranie żadnej wersji paczki))
    
3. Gdy 1. i 2. nie zachodzą, $S$ jest **nierozstrzygalny** (inconclusive) dla termu $t$ ^term-inconclusive
>[!EXAMPLE] Przykład 
> zbiór $S$ `foo ^1.0.0` jest nierozstrzygalny dla $t$ `foo ^1.5.0` 
> 
Wersje `>=1.0.0, <1.5.0` spełniają $S$, nie spełniają $t$; wersje `>=1.5.0, <2.0.0` spełniają zarówno $S$, jak i $t$ 

Termy mogą być traktowane jako zbiory dozwolonych wersji, gdzie ich negacja to dopełnienie tego zbioru. Działania i relacje na zbiorach też mogą być zdefiniowane, np.:
- `foo ^1.0.0 ∪ foo ^2.0.0` to `foo >=1.0.0 <3.0.0`.
>[!EXAMPLE] Przykład 
> `foo ^1.0.0 -> {foo ^1.0.0} -> {foo >=1.0.0, <2.0.0}`
> `foo ^2.0.0 -> {foo ^2.0.0} -> {foo >=2.0.0, <3.0.0}`
> suma daje ciągłość w `2.0.0`, stąd `>=1.0.0, <3.0.0`
- `foo >=1.0.0 ∩ not foo >=2.0.0` to `foo ^1.0.0`.
>[!EXAMPLE] Przykład 
> `not foo >=2.0.0 -> foo <2.0.0`
> część wspólna daje `foo ^1.0.0`
- `foo ^1.0.0 \ foo ^1.5.0` to `foo >=1.0.0 <1.5.0`.
>[!EXAMPLE] Przykład 
> usunięcie `foo >=1.5.0` z `foo ^1.0.0`

Z tego mamy dwa wnioski:
1. $S$ spełnia $t$ $\iff$ $\bigcap S \subseteq t$
>[!EXAMPLE] Przykład 
   >==TODO dodać przykład==
2. $S$ nie spełnia $t \iff \bigcap S$ jest rozłączne z $t$
>[!EXAMPLE] Przykład 
   >==TODO dodać przykład==
   
##### niezgodność (*incompatibility*)

Zbiór termów, w którym nie wszystkie termy mogą być spełnione - tzn., że co najmniej jeden z termów w konflikcie musi być fałszywy, żeby ogólne rozwiązanie mogło być prawdziwe. 

>[!EXAMPLE] Przykład 
niezgodność `{foo ^1.0.0, bar ^2.0.0}` oznacza, że jeśli wybierzemy np. `foo 1.2.3`, to nie będziemy mogli wybrać w tym samym czasie `bar 2.3.4`, bo ogólne rozwiązanie byłoby fałszywe

Dwa typy niezgodności:
1. *external* (zewnętrzne) - zależności paczki od innej paczki, np. paczka `foo ^1.0.0` zależy od `bar ^2.0.0`, to niezgodność z tego wynikająca to `{foo ^1.0.0, not bar ^2.0.0}` ^external
2. *derived* (wywnioskowane) - podczas [rozwiązywania konfliktów](/pubgrub/działanie) [[rozwiązywanie konfliktów|rozwiązywania konfliktów]], powstają z dwóch wcześniej istniejących niezgodności (przyczyn). Zapisywane, żeby nie sprawdzać kilka razy tego samego ^derived

Niezgodności są normalizowane, żeby w jednej niezgodności nie znajdowały się wielokrotne odniesienia do tej samej paczki, np.:
- `{foo >=1.0.0, foo <2.0.0}` normalizowane do `foo ^1.0.0`
Wywnioskowane niezgodności też normalizowane, żeby usunąć prawdziwe określenia do głównej (root) paczki, bo te będą zawsze spełnione


==TODO zmienić na listę i dodać przykłady==
>We say that a set of terms `S` satisfies an incompatibility `I` if `S` satisfies every term in `I`. ^incompatibility-satisfies

>We say that `S` contradicts `I` if `S` contradicts at least one term in `I`. ^incompatibility-contradicts

>If `S` satisfies all but one of `I`'s terms and is inconclusive for the remaining term, we say `S` "almost satisfies" `I` and we call the remaining term the "unsatisfied term". ^incompatibility-almost-satisfies
##### częściowe rozwiązanie (*partial solution*)

Uporządkowana lista termów zwana "przypisaniami" (przydziałami?, *assignments*). Reprezentuje obecnie najlepsze przypuszczenie dotyczące wersji paczek, które będą wykorzystywane. ^assignment

Mamy dwie kategorie przypisań:
1. **decyzje** (decision) - indywidualna wersja paczki, która może zadziałać (`PackageId`)
2. **wyprowadzenia** (derivation) - z reguły zakres wersji (`PackageRange`), który reprezentuje termy, które muszą być prawdziwe wywnioskowane na podstawie niezgodności i poprzednich przypisań. Zapisują one przyczynę swojej niezgodności.

Każde przypisanie ma powiązany ze sobą "decision level", inta > 0 reprezentującą liczbę decyzji, na której jest albo przed nią w 
Służy do określenia, jak daleko mamy się cofnąć, żeby znaleźć główną przyczynę przy [[rozwiązywanie konfliktów|rozwiązywaniu konfliktów]] i jak daleko mamy się cofnąć, gdy konflikt zostanie znaleziony. 

##### graf wnioskowania (*derivation graph*)

Skierowany, acykliczny graf binarny (czyli właściwie drzewo, tylko odwrócone). 
Liście (wierzchołki, do których nie wchodzą krawędzie) to [[#^external|zewnętrzne (external) niezgodności]], wierzchołki wewnętrzne (czyli pozostałe) to [[#^derived|wywnioskowane (derived) niezgodności]].

Liście: 1, 2, 4, 6
Wewnętrzne: 3, 5, 7
```
┌1───────────────────────────┐ ┌2───────────────────────────┐
│{foo ^1.0.0, not bar ^2.0.0}│ │{bar ^2.0.0, not baz ^3.0.0}│
└─────────────┬──────────────┘ └──────────────┬─────────────┘
              │      ┌────────────────────────┘
              ▼      ▼
┌3────────────┴──────┴───────┐ ┌4───────────────────────────┐
│{foo ^1.0.0, not baz ^3.0.0}│ │{root 1.0.0, not foo ^1.0.0}│
└─────────────┬──────────────┘ └──────────────┬─────────────┘
              │      ┌────────────────────────┘
              ▼      ▼
 ┌5───────────┴──────┴──────┐  ┌6───────────────────────────┐
 │{root any, not baz ^3.0.0}│  │{root 1.0.0, not baz ^1.0.0}│
 └────────────┬─────────────┘  └──────────────┬─────────────┘
              │   ┌───────────────────────────┘
              ▼   ▼
        ┌7────┴───┴──┐
        │{root 1.0.0}│
        └────────────┘
``` 

Tworzony dzięki temu, że każda wywnioskowana (derived) niezgodność przechowuje informację o swojej przyczynie. 
Gdy algorytm nie znajdzie żadnego rozwiązania, wykorzystuje graf dla niezgodności `root any`, żeby pokazać, dlaczego żadna wersja paczki `root` nie może zostać wybrana (czyli, że dependencje się nie zgadzają)

1. Because `foo ^1.0.0` depends on `bar ^2.0.0`
2. and `bar ^2.0.0` depends on `baz ^3.0.0`,
3. `foo ^1.0.0` requires `baz ^3.0.0`.
4. And, because `root` depends on `foo ^1.0.0`,
5. `root` requires `baz ^3.0.0`.
6. So, because `root` depends on `baz ^1.0.0`,
7. `root` isn't valid and version solving has failed.


[^1]: Trochę sprzeczne z traktowaniem termów jako zbiorów dozwolonych wersji i ich negowaniem, ale słaby z logiki jestem i może czegoś nie łapię