Intro do pubgruba - https://nex3.medium.com/pubgrub-2fb6470504f
Bardziej in-depth - https://github.com/dart-lang/pub/blob/master/doc/solver.md

Z PubGruba korzysta [`uv`](https://github.com/astral-sh/uv/), alternatywa do pipa, konkretnie w [tym crate](https://github.com/astral-sh/uv/tree/main/crates/uv-resolver/src)

Kolejny link do przeanalizowania (wzięty z issue scarbowego) - https://github.com/njsmith/posy/blob/main/src/resolve.rs

---

Dlaczego nie da się wg Marka wykorzystać gotowej paczki `pubgrub-rs`?

1. Nie dało się jej zintegrować z semver, bo brakowało jakiś funkcjonalności w algorytmie co umożliwiłyby wsparcie dla pre-releasów.
2. Nie wiem jak lockfile tam zrobić w tym.
3. Nasz resolver jest asynchroniczny, bo on leniwie może ściągać paczki/summary z rejestru i tam się robiła rzeź żeby tą logikę zintegrować z tą libką, bo ona jest synchroniczna. Tzn. jedynie rozwiązanie to odpalić algorytm w osobnym wątku i mpsc channelami komunikować go z wątkiem gdzie IO by się działo. Wykonalne, ale przez poprzednie punkty trochę nie widziałem opłacalności tej zabawy wtedy.

ad. 1
pubgrub-rs
- [docsy](https://pubgrub-rs-guide.netlify.app/)
- Istnieje już crate [semver-pubgrub](https://github.com/pubgrub-rs/semver-pubgrub), który (chyba) rozwiązuje ten problem
- Problemy z pre-releasami [tutaj](https://pubgrub-rs-guide.netlify.app/limitations/prerelease_versions) (docsy crate `pubgrub-rs`)


