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

**Hasła.** In your app's `build.gradle` you will need to define the `signingConfigs` for the release build. Here is what you should avoid:
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

Plik ten jest automatycznie importowany przez Gradle, więc możesz go użyć w `build.gradle` jak poniżej:a`

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
