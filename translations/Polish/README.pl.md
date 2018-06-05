# Najlepsze praktyki tworzenia aplikacji Android

Unikaj ponownego odkrywania koła, postępując zgodnie z tymi wytycznymi. Wnioski wyciągnięte przez programistów Androida w [Futurice](http://www.futurice.com). Jeśli jesteś zainteresowany rozwijaniem aplikacji iOS lub Windows Phone, sprawdź również nasze [**iOS Dobre Praktyki**](https://github.com/futurice/ios-good-practices) i [**Windows Rozwijanie Aplikacji Najlepsze praktyki**](https://github.com/futurice/windows-app-development-best-practices) dokumenty.

[![Android Arsenal](https://img.shields.io/badge/Android%20Arsenal-android--best--practices-brightgreen.svg?style=flat)](https://android-arsenal.com/details/1/1091) 

## Podsumowanie

#### [Używaj Gradle, to jest domyślna struktura projektu](#zbuduj-system)
#### [Umieść hasła i dane poufne w gradle.properties](#konfiguracja-gradle)
#### [Użyj biblioteki Jackson do parsowania danych JSON](#biblioteki)
#### [Nie pisz własnego klienta HTTP, korzystaj z bibliotek OkHttp](#networklibs)
#### [Unikaj Guava i używaj tylko kilku bibliotek ze względu na limit *65k metod](#methodlimitation)
#### [Wybieraj ostrożnie pomiędzy Aktywnościami i Fragmentami](#aktywnosci-i-fragments)
#### [Layout'y XML to kod, dobrze je uporządkuj](#resources)
#### [Użyj stylów, aby uniknąć duplikowania atrybutów w układach XML](#style)
#### [Użyj wielu plików stylu, aby uniknąć pojedynczego dużego](#splitstyles)
#### [Zachowaj plik colors.xml krótki i DRY, po prostu zdefiniuj paletę](#colorsxml)
#### [Zachowaj także plik dimens.xml DRY, zdefiniuj stałe ogólne](#dimensxml)
#### [Nie twórz głębokiej hierarchii ViewGroups](#deephierarchy)
#### [Unikaj przetwarzania po stronie klienta w przypadku użycia WebViews i uważaj na wycieki](#webview)
#### [Użyj JUnit do testów jednostkowych, Espresso do testów połączonych (UI) i AssertJ-Android dla łatwiejszych asercji w testach Androida](#test-frameworks)
#### [Zawsze używaj ProGuard lub DexGuard](#proguard-configuration)
#### [Użyj SharedPreferences dla prostej zapisów, dla pozostałych ContentProviders](#przechowywanie-danych)
#### [Użyj Stetho do debugowania aplikacji](#uzyj-stetho)
#### [Użyj Leak Canary, aby znaleźć wycieki pamięci](#use-leakcanary)
#### [Użyj Continuous Integration](#uzyj-continuous-integration)

----------

### Android SDK

Umieść swój [Android SDK](https://developer.android.com/sdk/installing/index.html?pkg=tools) gdzieś w swoim katalogu domowym lub w innej lokalizacji niezależnej od aplikacji. Niektóre IDE pobierają zestaw SDK przy instalacji i mogą umieszczać go w tym samym katalogu, co IDE. Może to być złe, gdy musisz zaktualizować (lub ponownie zainstalować) IDE. Przez to możesz utracić SDK i być zmuszonym do długiego i żmudnego ponownego pobierania.

Należy również unikać umieszczania zestawu SDK w katalogu na poziomie systemu, który może wymagać uprawnień użytkownika root.

### Zbuduj system

Domyślnie powinienes używać [Gradle](https://gradle.org) (https://gradle.org) przy pomocy wtyczki [Android Gradle plugin](https://developer.android.com/studio/build/index.html).

Ważne jest, aby proces tworzenia aplikacji był zdefiniowany przez pliki Gradle, a nie zależny od konfiguracji określonych przez IDE. Pozwala to na spójne kompilacje między narzędziami i lepszą obsługę systemu Continuous Integration.

### Struktura projektu

Mimo że Gradle oferuje dużą elastyczność w strukturze projektu. Powinieneś zaakceptować jego [domyślną strukturę](https://developer.android.com/studio/build/index.html#sourcesets), ponieważ upraszcza to Twoje skrypty budowania, chyba że masz do tego nieodparty powód, aby zrobić inaczej.

### Konfiguracja Gradle

**Struktura ogólna.** Śledź [Przewodnik Google na temat Gradle dla Androida](https://developer.android.com/studio/build/index.html).

**minSdkVersion: 21** Przed zdefiniowaniem minimalnego wymaganego interfejsu API zapoznaj się z [Tablą użycia wersji Androida](https://developer.android.com/about/dashboards/index.html#Platform). Pamiętaj, że podane statystyki są globalnymi statystykami i mogą się różnić w przypadku kierowania na konkretny rynek regionalny/demograficzny. Warto wspomnieć, że niektóre funkcje Material Design są dostępne tylko w systemie Android 5.0 (poziom API 21) i nowszych wersjach. A także, z API 21, biblioteka pomocy multidex nie jest już potrzebna.

**Małe zadania.** Zamiast skryptów (powłoki, Python, Perl itp.), możesz wykonywać zadania w Gradle. Wystarczy zapoznać się z [Dokumentacją Gradle'a](http://www.gradle.org/docs/current/userguide/userguide_single.html#N10CBF), aby uzyskać więcej informacji. Google zapewnia także pomocne [przepisy Gradle'a](https://developer.android.com/studio/build/gradle-tips.html), specyficzne dla Androida.

**Hasła.** W pliku `build.gradle` Twojej aplikacji będziesz musiał zdefiniować `signingConfigs` dla wersji release. Oto, czego należy unikać:

_Nie rób tego!_. Pojawi się to w systemie kontroli wersji.

```groovy
signingConfigs {
    release {
        // NIE RÓB TEGO!!
        storeFile file("myapp.keystore")
        storePassword "password123"
        keyAlias "thekey"
        keyPassword "password789"
    }
}
```

Zamiast tego należy utworzyć plik `gradle.properties`, którego nie należy dodawać do systemu kontroli wersji:

```
KEYSTORE_PASSWORD=password123
KEY_PASSWORD=password789
```

Plik ten jest automatycznie importowany przez Gradle, więc możesz go użyć w `build.gradle` jak poniżej:

```groovy
signingConfigs {
    release {
        try {
            storeFile file("myapp.keystore")
            storePassword KEYSTORE_PASSWORD
            keyAlias "thekey"
            keyPassword KEY_PASSWORD
        }
        catch (ex) {
            throw new InvalidUserDataException("You should define KEYSTORE_PASSWORD and KEY_PASSWORD in gradle.properties.")
        }
    }
}
```

**Preferuj rozwiązywanie zależności Maven'a, aby zaimportować pliki jar.** Jeśli jawnie umieścisz pliki jar w projekcie, będą one specyficzną, zamrożoną wersją, na przykład `2.1.1`. Pobieranie jar'ów i aktualizacja są uciążliwe. Są także problematyczne, natomiast Maven już rozwiązuje je poprawnie. Tam, gdzie to możliwe, powinieneś spróbować użyć Maven'a do rozwiązania twoich zależności, na przykład:

```groovy
dependencies {
    implementation 'com.squareup.okhttp3:okhttp:3.10.0'
}
```    

**Unikaj dynamicznego rozwiązywania zależności Maven'a**
Unikaj używania dynamicznych wersji zależności, takich jak `2.1. +`, ponieważ może to powodować różne i niestabilne kompilacje lub subtelne, nieśledzone różnice w zachowaniu między kompilacjami. Korzystanie z wersji statycznych, takich jak `2.1.1` pomaga stworzyć bardziej stabilne, przewidywalne i powtarzalne środowisko programistyczne.

**Użyj innej nazwy pakietu dla kompilacji innych niż wydania**
Użyj `applicationIdSuffix` dla *debug* [typu budowania](http://tools.android.com/tech-docs/new-build-system/user-guide#TOC-Build-Types), aby móc zainstalować oba typy *debug* i *release* apk na tym samym urządzeniu (zrób to również dla niestandardowych typów kompilacji, jeśli ich potrzebujesz). Będzie to szczególnie cenne po opublikowaniu Twojej aplikacji.

```groovy
android {
    buildTypes {
        debug {
            applicationIdSuffix '.debug'
            versionNameSuffix '-DEBUG'
        }

        release {
            // ...
        }
    }
}
```

Użyj różnych ikon, aby odróżnić kompilacje zainstalowane na urządzeniu - na przykład z różnymi kolorami lub nałożoną etykietą "debugowanie". Gradle czyni to bardzo łatwo: z domyślną strukturą projektu wystarczy umieścić ikonę *debug* w `app/src/debug/res` i *release* w `app/src/release/res`. Możesz również [zmienić nazwę aplikacji](http://stackoverflow.com/questions/24785270/how-to-change-app-name-per-gradle-build-type) na typ kompilacji, a także `versionName` (jak w powyższym przykładzie).

**Udostępnij plik magazynu kluczy aplikacji debugowania**
Udostępnianie pliku magazynu kluczy APK debugowania za pośrednictwem repozytorium aplikacji oszczędza czas podczas testowania na wspólnych urządzeniach i pozwala uniknąć odinstalowania/ponownego instalowania aplikacji. Upraszcza również przetwarzanie pracy z niektórymi pakietami Android SDK, takimi jak Facebook, które wymagają rejestracji pojedynczego hash'a magazynu kluczy. W przeciwieństwie do pliku klucza wersji, plik klucza debugowania można bezpiecznie dodać do repozytorium.

**Udostępnij definicje formatowania stylu kodu**
Udostępnianie stylu kodu i definicji formatowania za pośrednictwem repozytorium aplikacji zapewnia spójny wizualnie kod i ułatwia zrozumienie kodu oraz przeglądanie.

### Android Studio Twoim głównym IDE

Zalecanym środowiskiem developerskim dla Androida jest [Android Studio](https://developer.android.com/sdk/installing/studio.html) ponieważ jest rozwijane i aktualizowane przez Google, posiada dobre wsparcie dla Gradle, posiada szereg przydatnych narzędzi do monitorowania i analiz oraz jest w pełni dostosowany do developmentu Androida.

Unikaj dodawania specyficznych plików konfiguracyjnych Android Studio, takich jak pliki `.iml` do systemu kontroli wersji, ponieważ często zawierają one konfiguracje specyficzne dla twojego komputera lokalnego, które nie będą działać u Twoich współpracowników.

### Biblioteki

- **[Jackson](http://wiki.fasterxml.com/JacksonHome)** to biblioteka Java do serializacji i deserializacji JSON, ma szeroki zakres i wszechstronny interfejs API, obsługujący różne sposoby przetwarzania JSON: streaming, model drzewa w pamięci i tradycyjne JSON-POJO powiązanie danych. 

- [Gson](https://code.google.com/p/google-gson/) to kolejny popularny wybór, będący mniejszą biblioteką niż Jackson. Może wybierzesz go, aby uniknąć przekroczenia limitu 65k metod. Ponadto, jeśli używasz...
- [Moshi](https://github.com/square/moshi), jednej z open source'owych bibliotek [Square](https://github.com/square), opiera się na nauce płynącej z rozwoju Gsona, a także dobrze integruje się z Kotlinem.

<a name="networklibs"></a>
**Praca w sieci, buforowanie i obrazy.** Istnieje kilka sprawdzonych w praktyce rozwiązań do wysyłania żądań do serwerów, których powinieneś używać zamiast implementować własnego klienta. Zalecamy oparcie twojego stosu wokół [OkHttp](http://square.github.io/okhttp/) dla wydajnych żądań HTTP i użycie [Retrofit](http://square.github.io/retrofit/) w celu zapewnienia bezpieczeństwa. Jeśli wybierzesz Retrofit, rozważ [Picasso](http://square.github.io/picasso/) do ładowania i buforowania obrazów.

Retrofit, Picasso i OkHttp są tworzone przez tę samą firmę, więc dobrze się uzupełniają, a problemy ze zgodnością są rzadkością.

[Glide](https://github.com/bumptech/glide) to kolejna opcja do ładowania i buforowania obrazów. Obsługuje animowane GIF-y, okrągłe obrazy i twierdzi, że ma lepszą wydajność niż Picasso, ale także większą liczbę metod.

**RxJava** jest biblioteką do programowania reaktywnego, innymi słowy, obsługującą zdarzenia asynchroniczne. Jest to potężny paradygmat, ale ma również stromą krzywą uczenia się. Zalecamy zachować ostrożność przed użyciem tej biblioteki do zaprojektowania całej aplikacji. Napisaliśmy na niej kilka postów na blogu: [[1]](http://blog.futurice.com/tech-pick-of-the-week-rx-for-net-and-rxjava-for-android), [[2]](http://blog.futurice.com/top-7-tips-for-rxjava-on-android), [[3]](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754), [[4]](http://blog.futurice.com/android-development-has-its-own-swift). W przypadku aplikacji referencyjnej nasza aplikacja Open Source [Freesound Android](https://github.com/futurice/freesound-android) szeroko wykorzystuje RxJava 2.

Jeśli nie masz wcześniejszego doświadczenia z Rx, zacznij od stosowania go tylko w odpowiedziach z API aplikacji. Możesz też rozpocząć od zastosowania do prostej obsługi zdarzeń interfejsu, takich jak kliknięcia lub wpisywanie zdarzeń w polu wyszukiwania. Jeśli jesteś pewny swoich umiejętności Rx i chcesz zastosować go do całej architektury, to napisz dokumentację na temat wszystkich trudnych części. Należy pamiętać, że inni programiści nieznający RxJava mogą mieć trudności z utrzymaniem projektu. Staraj się, aby pomóc im zrozumieć kod, a także Rx.

Używaj [RxAndroid](https://github.com/ReactiveX/RxAndroid) dla obsługi wątków Androida i [RxBinding](https://github.com/JakeWharton/RxBinding) dla łatwego tworzenia Observables z istniejących komponentów Androida.

**[Retrolambda](https://github.com/evant/gradle-retrolambda)** jest biblioteką Java do używania składni wyrażeń Lambda w systemie Android i innych platformach przed JDK8. Pomaga to w utrzymaniu kodu w czysty i czytelny sposób, zwłaszcza jeśli używasz stylu funkcjonalnego, takiego jak RxJava.

Android Studio oferuje obsługę wspomagania kodu dla lambdamy w języku Java 8. Jeśli jesteś nowy w lambdach, po prostu wykonaj następujące czynności, aby rozpocząć:

- Każdy interfejs posiadający tylko jedną metodę jest "przyjazny dla lambda" i może być złożony w bardziej zwartej składni
- Jeśli masz wątpliwości co do parametrów itp., napisz normalną anonimową klasę wewnętrzną, a następnie pozwól Android Studio spasować ją w lambda.

Zwróć uwagę, że z wersji Android Studio 3.0, [Retrolambda nie jest już wymagana](https://developer.android.com/studio/preview/features/java8-support.html).

<a name="methodlimitation"></a>
**Uważaj na ograniczenie metody dex i unikaj używania wielu bibliotek.** Aplikacje na Androida, gdy są spakowane jako plik dex, mają surowe ograniczenie użytych 65536 metod [[1]](https://medium.com/@rotxed/dex-skys-the-limit-no-65k-methods-is-28e6cb40cf71) [[2]](http://blog.persistent.info/2014/05/per-package-method-counts-for-androids.html) [[3]](http://jakewharton.com/play-services-is-a-monolith/). Jeśli przekroczysz limit, zobaczysz błąd krytyczny kompilacji. Z tego powodu użyj minimalnej ilości bibliotek i użyj narzędzia [dex-method-counts](https://github.com/mihaip/dex-method-counts), aby określić, który zestaw bibliotek może być użyty, aby pozostać poniżej limitu. W szczególności unikaj używania biblioteki Guava, ponieważ zawiera ona ponad 13 tys. Metod.

### Aktywności i Fragmenty

Nie ma zgody wśród społeczności ani twórców Futurice, jak najlepiej zorganizować architekturę Androida za pomocą Fragmentów i Aktywności. Square ma nawet [bibliotekę do budowania architektury głównie za pomocą Views](https://github.com/square/mortar), pomijając potrzebę Fragmentów, ale nadal nie jest to powszechnie zalecana praktyka w społeczności.

Z powodu historii interfejsu API Androida możesz swobodnie traktować fragmenty jako kawałki interfejsu użytkownika ekranu. Innymi słowy, Fragmenty są zwykle związane z interfejsem użytkownika. Działania mogą być luźno uważane za kontrolery, są szczególnie ważne dla ich cyklu życia i zarządzania stanem. Prawdopodobnie zauważysz różnice w tych rolach: czynności mogą przenosić role interfejsu użytkownika ([dostarczanie przejść między ekranami](https://developer.android.com/about/versions/lollipop.html)) i [fragmenty mogą być używane wyłącznie jako kontrolery](http://developer.android.com/guide/components/fragments.html#AddingWithoutUI). Proponujemy używać ostrożnie, podejmując świadome decyzje, ponieważ istnieją wady przy wyborze architektury zorientowanej wyłącznie na: fragmenty, aktywności lub widoki. Oto kilka rad, na co należy uważać, ale potraktuj je z przymrużeniem oka:

- Unikaj używania [fragmentów zagnieżdżonych](https://developer.android.com/about/versions/android-4.2.html#NestedFragments) zbyt rozlegle, ponieważ mogą wystąpić [błędy matryoshki](http://delyan.me/android-s-matryoshka-problem/). Użyj zagnieżdżonych fragmentów tylko wtedy, gdy ma to sens (na przykład fragmenty w przesuwanym poziomo widoku ViewPager wewnątrz fragmentu podobnego do ekranu (screen-like fragment)) lub jeśli jest to świadoma decyzja.
- Unikaj umieszczania zbyt dużej ilości kodu w Activity. O ile to możliwe, przechowuj je w postaci lekkich kontenerów, istniejących w aplikacji głównie w cyklu życia i innych ważnych interfejsach Androida (Android-interfacing APIs). Preferuj Single-Fragment Activities zamiast zwykłych Activities - umieść kod UI w Fragmencie. To sprawia, że można go ponownie użyć na wypadek, gdyby trzeba było go zmienić w układzie kart lub na ekranie z wieloma fragmentami tabletu. Unikaj aktywności bez odpowiedniego fragmentu, chyba że podejmiesz świadomą decyzję.

### Struktura pakietów Java

Zalecamy użycie struktury pakietu *opartej na funkcjach* dla Twojego kodu. Ma to następujące zalety:

- Wyraźniejsza zależność funkcji i granic interfejsu.
- Promuje enkapsulację.
- Łatwiejsze zrozumienie komponentów definiujących tę funkcję.
- Zmniejsza ryzyko nieświadomej modyfikacji niepowiązanego lub współdzielonego kodu.
- Prostsza nawigacja: większość powiązanych klas będzie w jednym pakiecie.
- Łatwiejsze usuwanie funkcji.
- Upraszcza przejście do struktury budowania opartej na modułach (lepsze czasy budowy i obsługa aplikacji błyskawicznych {Instant Apps})


Alternatywne podejście do definiowania pakietów poprzez *jak* funkcja jest budowana (poprzez umieszczanie powiązanych działań, fragmentów, adapterów itp. w oddzielnych pakietach) może prowadzić do fragmentacji kodu z mniejszą elastycznością implementacji. Co najważniejsze, utrudnia to zrozumienie kodu w zakresie jego podstawowej roli: zapewnienie funkcji dla aplikacji.

