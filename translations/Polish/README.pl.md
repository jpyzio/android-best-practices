# Najlepsze praktyki tworzenia aplikacji Android

Unikaj ponownego odkrywania koła, postępując zgodnie z tymi wytycznymi. Wnioski wyciągnięte przez programistów Androida w [Futurice](http://www.futurice.com). Jeśli jesteś zainteresowany rozwijaniem aplikacji iOS lub Windows Phone, sprawdź również nasze [**iOS Dobre Praktyki**](https://github.com/futurice/ios-good-practices) i [**Windows Rozwijanie Aplikacji Najlepsze praktyki**](https://github.com/futurice/windows-app-development-best-practices) dokumenty.

[![Android Arsenal](https://img.shields.io/badge/Android%20Arsenal-android--best--practices-brightgreen.svg?style=flat)](https://android-arsenal.com/details/1/1091) 

## Podsumowanie

#### [Używaj Gradle, to jest domyślna struktura projektu] (#istniejący-system)
#### [Umieść hasła i dane poufne w gradle.properties] (#gradle-konfiguracja)
#### [Użyj biblioteki Jackson do parsowania danych JSON] (#biblioteki)
#### [Nie pisz własnego klienta HTTP, korzystaj z bibliotek OkHttp] (#networklibs)
#### [Unikaj Guava i używaj tylko kilku bibliotek ze względu na limit *65k metod] (#methodlimitation)
#### [Wybieraj ostrożnie pomiędzy Aktywnościami i Fragmentami] (#aktywnosci-i-fragments)
#### [Layout'y XML to kod, dobrze je uporządkuj] (#resources)
#### [Użyj stylów, aby uniknąć duplikowania atrybutów w układach XML] (#style)
#### [Użyj wielu plików stylu, aby uniknąć pojedynczego dużego] (#splitstyles)
#### [Zachowaj plik colors.xml krótki i DRY, po prostu zdefiniuj paletę] (#colorsxml)
#### [Zachowaj także plik dimens.xml DRY, zdefiniuj stałe ogólne] (#dimensxml)
#### [Nie twórz głębokiej hierarchii ViewGroups] (#deephierarchy)
#### [Unikaj przetwarzania po stronie klienta w przypadku użycia WebViews i uważaj na wycieki] (#webview)
#### [Użyj JUnit do testów jednostkowych, Espresso do testów połączonych (UI) i AssertJ-Android dla łatwiejszych asercji w testach Androida] (#test-frameworks)
#### [Zawsze używaj ProGuard lub DexGuard] (# proguard-configuration)
#### [Użyj SharedPreferences dla prostej zapisów, dla pozostałych ContentProviders] (#przechowywanie-danych)
#### [Użyj Stetho do debugowania aplikacji] (#uzyj-stetho)
#### [Użyj Leak Canary, aby znaleźć wycieki pamięci] (#use-leakcanary)
#### [Użyj Continuous Integration] (#uzyj-continuous-integration)

----------
