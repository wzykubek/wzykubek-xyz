+++
title = "Bookmarklet - zapomniana funkcja przeglądarek internetowych"
date = 2025-12-06T01:05:32+01:00
draft = false
layout = 'article'
description = 'W tym artykule opisuję bookmarklety, czyli skrypty JS wykonywane po kliknięciu w zakładkę przeglądarki.'
+++

Obecnie internet to straszne miejsce, gdzie "surfowanie" to bardziej pływanie w morzu ~~plastikowych~~ NPM-owych odpadów. Przeważająca część nowych stron internetowych jest tworzona przy użyciu wielkich frameworków, bibliotek i innych w większości przypadków **zbędnych** zależności. Jest wiele zastosowań, gdzie takie rozwiązania są przydatne i wskazane, ale nawet strony, które serwują tylko proste i z założenia statyczne treści zawierają mnóstwo niepotrzebnych skryptów bez żadnego powodu.

Oprócz tego mamy jeszcze całą masę rozszerzeń do przeglądarek. Dodają one przydatne funkcje, ale często spowalniają przeglądarkę i mają dostęp do prywatnych danych, których prawdopodobnie nikt z nas nie chce nikomu udostępniać. Ale co jeśli Ci powiem, że możesz usunąć część ze swoich rozszerzeń bez tracenia funkcjonalności, które oferują? We wszystkich przeglądarkach *głównego nurtu*, takich jak Firefox, Chrom(e)ium, Safari, oraz w ich mobilnych wersjach dostępna jest funkcja zwana [bookmarklet](https://en.wikipedia.org/wiki/Bookmarklet). Ta niesamowita technologia została udostępniona 25 lat temu i jest wspierana do dzisiaj.

## Wyjaśnienie
Bookmarklet to rodzaj zakładki w przeglądarce, która zamiast linku do strony uruchamia kod JavaScript. Inną nazwą dla bookmarkletów jest *favelet*.

## Przykłady 
*Możesz zaznaczyć dowolną linijkę JavaScriptu poniżej, a następnie przeciągnąć ją na pasek zakładek[^1].*

[^1]: Używanie bookmarkletów osób trzecich jest kiepskim pomysłem, ale to samo tyczy się rozszerzeń i innych skryptów znalezionych w internecie. Myślę, że to oczywiste, ale dla zasady o tym wspominam.

### Wayback Machine
Otwieranie *martwej* strony internetowej korzystając z [Archiwum Internetu](https://web.archive.org/), lub zapisywanie stanu *jeszcze* żyjącej strony.

**Otwieranie**:
```javascript
javascript:location.href='https://web.archive.org/web/*/'+document.location.href.replace(/\/$/, '');
```

**Zapisywanie**:
```javascript
javascript:void(window.open('https://web.archive.org/save/'+location.href));
```

Źródło: [Wikipedia](https://en.wikipedia.org/wiki/Help:Using_the_Wayback_Machine#JavaScript_bookmarklet)

### Słowniki i tłumacze
Wysyłanie zaznaczonego tekstu do słownika lub tłumacza online.

```javascript
javascript:void(window.open('https://www.diki.pl/?q='+encodeURIComponent(document.getSelection().toString())));
```

### Inne
Niektóre z serwisów oferują nawet swoje własne ~~*zakładko-skrypty*~~ bookmarklety. Na przykład [Miniflux](https://miniflux.app/) pozwala na dodawanie nowych feedów RSS, a [Linkding](https://linkding.link/) pozwala zapisać obecną stronę jako zakładkę.

Jak możesz zauważyć w moich przykładach, używam tej funkcjonalności głównie do integracji z zewnętrznymi serwisami, ale nic nie stoi na przeszkodzie, żeby wywołać dowolny inny kod... z małymi wyjątkami, o których piszę w sekcji [problemy](#problemy).

## Problemy
Może zastanawiasz się, czemu ta technologia nie jest wykorzystywana częściej, jeśli jest tak prosta i użyteczna. Jest to spowodowane głównie [Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CSP), które zostało wprowadzone do przeglądarek kilka (około 10) lat temu. Powodem jest ochrona przed podatnościami [XSS](https://en.wikipedia.org/wiki/Cross-site_scripting). Niestety wiąże się to również z blokadą wykonywania niektórych skryptów użytkowników na stronach, które korzystają z tego mechanizmu. Na szczęście, nie wszystkie strony są objęte tym problemem, jak również nie każdy bookmarklet. Wszystkie [przykłady](#przykłady), które podałem, powinny bez problemu działać na każdej stronie internetowej.
