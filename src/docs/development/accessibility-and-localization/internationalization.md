https://github.com/techhtml/website/edit/flutter-dev-ko/src/docs/development/accessibility-and-localization/internationalization.md

---
title: 플러터 앱 국제화
short-title: i18n
description: 플러터 앱을 국제화 하는 방법
---

{{site.alert.secondary}}
  <h4 class="no_toc">다음을 배우게 됩니다.</h4>

  * 장치의 Locale(사용자가 선호하는 언어) 추적하기.
  * Locale에 따라 달라지는 값을 관리하는 방법.
  * 앱에서 지원하는 Locale을 정의하는 방법.
{{site.alert.end}}

다른 언어를 사용하는 사용자에게 앱을 배포할 수 있는 경우 앱을 국제화해야 합니다.
즉 앱이 지원하는 "지역(Locale)"이나 언어 별로 텍스트, 레이아웃 같은 값들이 
"현지화(Localize)" 될 수 있는 방법으로 앱을 개발해야 합니다. 플러터는 국제화를
지원하는 클래스나 위젯들을 제공하고 있으며 플러터 라이브러리들은 자체적으로 국제화되어 있습니다.

아래 튜토리얼들은 대부분의 플러터 앱에서 사용되는 `MaterialApp` 클래스로 작성되어 있습니다.
더 하위 레벨인 `WidgetsApp` 클래스로 작성된 앱에서도 동일한 클래스와 로직을 사용하여
국제화를 적용할 수 있습니다.

<aside class="alert alert-info" markdown="1">
  <h4 class="no_toc">국제화 앱 예제</h4>

  국제화된 플러터앱 코드를 먼저 살펴보고 싶으신 분들을 위해 2가지 예제를 제공합니다.
  첫번째는 가능한한 단순하게 구현된 예제이며, 두번째는 [intl]({{site.pub-pkg}}/intl) 패키지에 의해 제공되는
  API와 도구를 사용한 예제입니다.
  다트 Intl 패키지를 처음 사용하시는 분들은 [Dart intl 도구 사용하기](#dart-tools)를 확인해주세요.

  * [최소화된 국제화 예제]({{site.github}}/flutter/website/tree/master/examples/internationalization/minimal)
  * [`intl` 패키지 기반 국제화 예제]({{site.github}}/flutter/website/tree/master/examples/internationalization/intl_example)
</aside>

## 국제화된 앱 설정: flutter<wbr>_localizations 패키지 {#setting-up}

기본적으로 플러터는 US English 지역화만 제공합니다.
다른 언어에 대한 지원을 추가하려면, `flutter_localizations` 패키지를 추가하고
`MaterialApp`에 관련 속성들을 정의해야합니다. 2020년 2월 기준으로 이 패키지는
77개 언어를 지원하고 있습니다.

flutter_localizations를 사용하려면 `pubspec.yaml`을 열고
아래와 같이 패키지를 추가해주세요:

{% prettify yaml %}
dependencies:
  flutter:
    sdk: flutter
  flutter_localizations:
    sdk: flutter
{% endprettify %}

다음으로 flutter_localizations에 대한 import 구문을 추가하고
`MaterialApp`의 `localizationsDelegates`와 `supportedLocales` 속성을
정의해야합니다:

{% prettify dart %}
import 'package:flutter_localizations/flutter_localizations.dart';

MaterialApp(
 localizationsDelegates: [
   // ... 앱 별 Localization Delegate를 여기에 정의
   GlobalMaterialLocalizations.delegate,
   GlobalWidgetsLocalizations.delegate,
   GlobalCupertinoLocalizations.delegate,
 ],
 supportedLocales: [
    const Locale('en', ''), // 영어
    const Locale('he', ''), // 히브리어
    const Locale.fromSubtags(languageCode: 'zh'), // 중국어
    // ... 앱이 지원하는 Locale 정보
  ],
  // ...
)
{% endprettify %}

`WidgetsApp`를 기반으로 작성된 앱의 경우 대부분 동일하게 설정하면 되지만
`GlobalMaterialLocalizations.delegate`는 필요하지 않습니다.

`Locale` 기본 생성자도 충분하지만, scriptCode에 대한 지원이 필요하다면
`Locale.fromSubtags` 생성자를 사용할 수 있습니다.

`localizationsDelegates` 리스트의 인자들은 팩토리 클래스이며,
지역화된 값들을 포함하고 있는 컬렉션을 생성하는 역할을 합니다.
`GlobalMaterialLocalizations.delegate`는 Material Components 라이브러리를 위한
지역화된 문자열을 제공합니다. `GlobalWidgetsLocalizations.delegate`는 위젯 라이브러리가
텍스트를 나열하는 방향(왼쪽에서 오른쪽 또는 오른쪽에서 왼쪽으로 텍스트를 나열하는 설정)에 대한 기본 값을 정의하고 있습니다. 

이러한 설정에 대한 자세한 정보나 관련된 Type 정보, 그리고 국제회된
플러터 앱이 일반적으로 어떻게 구성되는지에 대한 정보는 아래에서 살펴볼 수 있습니다.

<a name="advanced-locale"></a>
## Locale 정의에 대한 상세정보

여러 변형이 존재하는 일부 언어에 대해서는 언어 코드 외에 더 많은 정보를 제공해야
적절하게 지역화될 수 있습니다.

예를들어, 중국어의 다양한 변형을 모두 지원하기 위해서는 언어코드(languageCode), 문자코드(scriptCode),
국가코드(countryCode)를 모두 정의해야 합니다. 왜냐하면 문자유형의 차이(간자체, 번자체)도 존재하며
동일한 문자유형 일지라도 지역에 따라 문자가 쓰이는 방법이 다르기 때문입니다.

국가코드 `CN`, `TW`, `HK`에 대한 모든 중국어 변형을 표현하기 위해서는
다음 Locale에 대한 지원을 모두 포함해야합니다.

{% prettify dart %}
// CN, TW, HK에 대한 모든 중국어 지원
supportedLocales: [
  const Locale.fromSubtags(languageCode: 'zh'), // 일반적인 중국어 'zh'
  const Locale.fromSubtags(languageCode: 'zh', scriptCode: 'Hans'), // 일반적인 중국어 간자체 'zh_Hans'
  const Locale.fromSubtags(languageCode: 'zh', scriptCode: 'Hant'), // 일반적인 중국어 번자체 'zh_Hant'
  const Locale.fromSubtags(languageCode: 'zh', scriptCode: 'Hans', countryCode: 'CN'), // 'zh_Hans_CN'
  const Locale.fromSubtags(languageCode: 'zh', scriptCode: 'Hant', countryCode: 'TW'), // 'zh_Hant_TW'
  const Locale.fromSubtags(languageCode: 'zh', scriptCode: 'Hant', countryCode: 'HK'), // 'zh_Hant_HK'
],
{% endprettify %}

이러한 완전하면서 구체적인 정의는 여러 분의 앱이 
제공된 국가코드의 모든 조합을 구별할 수 있게 해주며
섬세하면서도 완전히 현지화된 컨텐트를 제공할 수 있게 해줍니다.
만약 사용자가 선호하는 Locale이 정의되지 않으면 가장 가까운 Locale 값이 대신 사용되지만,
이러한 동작은 사용자가 기대하는 것과 다를 수 있습니다.
플러터는 `supportedLocales`에 정의되어 있는 값만 사용할 것입니다.
플러터는 공통적으로 사용되는 언어(languageCode)에 대해서 문자코드(scriptCode) 별로 
차별화된 지역화 컨텐트를 제공합니다. 앱에서 지원되는 Locale과 사용자가 선호하는 Locale이
어떻게 처리되는지 자세한 정보를 원하시면 
[`Localizations`]({{site.api}}/flutter/widgets/Localizations-class.html)를 확인해보세요.

중국어에 대한 예제를 주로 다루었지만, 프랑스어(fr_FR, fr_CA)와 같은 다른 언어들도
섬세한 지역화를 위해 완전히 세분화된 정보를 제공해야합니다.

<a name="tracking-locale"></a>
## Locale 추적: Locale 클래스 및 Localizations 위젯

[`Locale`]({{site.api}}/flutter/dart-ui/Locale-class.html) 클래스는 사용자의 언어를 식별하는데 사용됩니다.
모바일 장치는 일반적으로 시스템 설정 메뉴를 통해 모든 어플리케이션에 대한
Locale 설정을 지원합니다. 국제화가 진행된 앱이라면 Locale 별로 정의된 값들을 출력할 것입니다.
예를들어 사용자가 장치의 Locale을 영어에서 프랑스어로 전환하면 "Hello World"를 표시하던
`Text` 위젯은 "Bonjour le monde"를 출력하도록 다시 빌드됩니다.

[`Localizations`]({{site.api}}/flutter/widgets/Localizations-class.html) 
위젯은 자식 위젯을 위한 Locale과 자식이 의존하는 지역화된 리소스를 정의합니다.
[WidgetsApp]({{site.api}}/flutter/widgets/WidgetsApp-class.html)
위젯은 `Localizations` 위젯을 생성하고 시스템의 Locale이 변경되면 이를 재구축합니다.

`Localizations.localeOf()`를 사용하여 언제든지 앱의
현재 Locale을 조회할 수 있습니다:


{% prettify dart %}
Locale myLocale = Localizations.localeOf(context);
{% endprettify %}

<a name="loading-and-retrieving"></a>
## 지역화된 값 로드 및 검색

`Localizations` 위젯은 지역화된 값 컬렉션을 포함하는 객체를 로드하고 검색하는데 사용됩니다. 
앱에서는 [`Localizations.of(context,type)`]({{site.api}}/flutter/widgets/Localizations/of.html)을 
사용하여 로드된 객체를 참조하게 됩니다.
만약 장치의 Locale이 변경되면 `Localizations` 위젯은 새 Locale에
해당하는 값을 자동으로 로드하고 이 값을 사용하는 위젯들을 재구성합니다.
이러한 동작은 `Localizations` 위젯이 
[InheritedWidget]({{site.api}}/flutter/widgets/InheritedWidget-class.html) 
처럼 작동 하기 때문에 발생합니다.
build 함수에서 Inherited 위젯을 참조하면 해당 Inherited 위젯에 대한
암시적인 의존성이 생성됩니다. Inherited 위젯이 변경되면(`Localizations` 위젯의
Locale이 변경될 때) 이 의존성 컨텍스트도 재구성됩니다.

지역화된 값들은 `Localizations` 위젯의 [LocalizationsDelegate]({{site.api}}/flutter/widgets/LocalizationsDelegate-class.html) 
리스트에 의해 로드됩니다.
리스트에 포함된 각 Delegate는 비동기 [`load()`]({{site.api}}/flutter/widgets/LocalizationsDelegate/load.html)
메서드를 정의하고 있습니다.
이 메서드는 지역화된 값 컬렉션을 캡슐화하는 객체를 생성하여 리턴합니다. 일반적으로 이러한 객체들은 
지역화된 값 마다 하나의 메서드를 정의하고 있습니다.

대형 앱에서는 다양한 모듈과 패키지들이 자신만의 Localizations과 함께 포함되어 있습니다.
이것이 바로 `Localizations` 위젯이 `LocalizationsDelegate` 당 하나씩 
객체 테이블을 관리하는 이유입니다.
`LocalizationsDelegate`의 `load` 메서드 중 하나가 생성한 특정 객체를 검색하려면
`BuildContext`와 객체 유형을 지정하면 됩니다.

예를들어, Material Components 위젯의 지역화된 문자열들은 
[MaterialLocalizations]({{site.api}}/flutter/material/MaterialLocalizations-class.html)
클래스에 정의되어 있습니다.
이 클래스의 인스턴스는 [MaterialApp]({{site.api}}/flutter/material/MaterialApp-class.html)에서 
제공하는 `LocalizationDelegate`에 의해 생성되며, `Localizations.of()`을 사용하여 검색 할 수 있습니다.

{% prettify dart %}
Localizations.of<MaterialLocalizations>(context, MaterialLocalizations);
{% endprettify %}

이러한 특정 `Localizations.of()` 표현식은 자주 사용되므로
`MaterialLocalizations` 클래스는 편리한 단축표현식을 제공합니다.

{% prettify dart %}
static MaterialLocalizations of(BuildContext context) {
  return Localizations.of<MaterialLocalizations>(context, MaterialLocalizations);
}

/// MaterialLocalizations에 정의된 지역화된 값에 대한 참조는
/// 일반적으로 다음과 같이 작성됩니다.:

tooltip: MaterialLocalizations.of(context).backButtonTooltip,
{% endprettify %}

<a name="using-bundles">
## 번들된 Localizations&shy;Delegates 사용하기

가능한 작고 단순하게 유지하기 위해, 플러터 패키지에는 
US English에 대한 값을 제공하는
`MaterialLocalizations`와 `WidgetsLocalizations` 인터페이스에 대한
구현이 포함되고 있습니다.
이러한 구현 클래스들은 각각 `DefaultMaterialLocalizations`와
`DefaultWidgetsLocalizations`로 불립니다.
동일한 기본 유형의 다른 구현 Delegate가 `localizationsDelegates`에
지정되지 않는한 자동으로 포함됩니다.

`flutter_localizations` 패키지에는 Localizations 인터페이스의 다국어 
구현이 포함되어 있습니다. 이는 
[GlobalMaterialLocalizations]({{site.api}}/flutter/flutter_localizations/GlobalMaterialLocalizations-class.html)와 
[GlobalWidgetsLocalizations]({{site.api}}/flutter/flutter_localizations/GlobalWidgetsLocalizations-class.html)로 
불립니다. [국제화된 앱 설정](#setting-up)에서 설명한대로 국제화된 앱은 
이러한 클래스들에 대한 Localization Delegates를 반드시 지정해야합니다.

{% prettify dart %}
import 'package:flutter_localizations/flutter_localizations.dart';

MaterialApp(
 localizationsDelegates: [
   // ... 앱 별 Localization Delegate를 여기에 정의
   GlobalMaterialLocalizations.delegate,
   GlobalWidgetsLocalizations.delegate,
 ],
 supportedLocales: [
    const Locale('en', ''), // 영어, no country code
    const Locale('he', ''), // 히브리어, no country code
    const Locale('zh', ''), // 중국어, no country code
    // ... 앱이 지원하는 Locale 정보
  ],
  // ...
)
{% endprettify %}

전역 Localization Delegates는 해당하는 클래스의 인스턴스를 Locale 별로 생성합니다.
예를들어, `GlobalMaterialLocalizations.delegate`는 
`GlobalMaterialLocalizations` 인스턴스를 생성하는 `LocalizationsDelegate`입니다.

2020년 2월까지 전역 Localization 클래스는 [77개 언어]({{site.github}}/flutter/flutter/tree/master/packages/flutter_localizations/lib/src/l10n)를 
지원합니다.

<a name="defining-class"></a>
## 앱의 지역화된 리소스에 대한 클래스 정의

국제화된 앱을 위해 이 모든 것들을 통합하는 것은 
일반적으로 지역화된 값들을 캡슐화하는 클래스와 함께 시작됩니다.
다음 예제는 이러한 클래스들의 일반적인 형태를 보여줍니다.

예제 앱의 [전체 소스코드]({{site.github}}/flutter/website/tree/master/examples/internationalization/intl_example).

이 예제는 [intl]({{site.pub-pkg}}/intl) 패키지에서 제공되는 API 및 도구를 기반으로 합니다.

[지역화된 리소스에 대한 다른 예제 클래스](#alternative-class)는 `intl`패키지에 
의존하지 않는 [예제]({{site.github}}/flutter/website/tree/master/examples/internationalization/minimal)를
보여주고 있습니다.

`DemoLocalizations` 클래스는 앱에서 지원할 Locale로 번역된 문자열을 제공합니다. 
(여기에서는 예시를 위해 title 하나만 정의함)
이 클래스는 번역된 문자열을 로드하기 위해 [intl 패키지]({{site.pub-pkg}}/intl)에서 생성된
`initializeMessages()` 함수를 호출하고 있으며 지역화된 값을 검색하기 위해 
[`Intl.message()`]({{site.pub-api}}/intl/latest/intl/Intl/message.html)를
사용하고 있습니다.

{% prettify dart %}
class DemoLocalizations {
  DemoLocalizations(this.localeName);

  static Future<DemoLocalizations> load(Locale locale) {
    final String name = locale.countryCode.isEmpty ? locale.languageCode : locale.toString();
    final String localeName = Intl.canonicalizedLocale(name);
    return initializeMessages(localeName).then((_) {
      return DemoLocalizations(localeName);
    });
  }

  static DemoLocalizations of(BuildContext context) {
    return Localizations.of<DemoLocalizations>(context, DemoLocalizations);
  }

  final String localeName;

  String get title {
    return Intl.message(
      'Hello World',
      name: 'title',
      desc: 'Title for the Demo application',
      locale: localeName,
    );
  }
}
{% endprettify %}

`intl` 패키지를 기반으로 하는 클래스는 [`intl` 도구](#dart-tools)에 의해 생성된 Message 카탈로그를 import 해야합니다.
[`intl` 도구](#dart-tools)는 `Intl.message()`호출이 포함된 클래스의 소스코드를 분석하고
Message 카탈로그를 생생해 줍니다.
이렇게 생성된 Message 카탈로그는 `initializeMessages()`함수를 포함하고 있으며
`Intl.message()`에 대한 Locale 별 저장소도 제공합니다.
`DemoLocalizations` 클래스는 `Intl.message()` 호출을 포함하고 있으므로
`DemoLocalizations`를 바탕으로 `intl` 도구를 사용하여 Message 카탈로그를 생성할 수 있습니다.

<a name="specifying-supportedlocales"></a>
## supported&shy;Locales 매개변수 정의

플러터 flutter_localizations 라이브러리가 77개의 언어를 지원하지만,
오직 영어만 기본적으로 사용이 가능합니다. 앱에서 지원하는 Locale 외에
다른 Locale을 라이브러리가 지원하는 것은 합리적이지 않기 때문에, 
어떤 언어를 지원할 지 결정하는 것은 개발자에게 달려있습니다.

`MaterialApp`의 [`supportedLocales`]({{site.api}}/flutter/material/MaterialApp/supportedLocales.html)
매개변수는 Locale의 변경을 제한합니다. 사용자가 장치의 Locale 설정을 변경했을 때,
해당 Locale이 이 매개변수에 존재해야만 `Localizations` 위젯은 이를 반영합니다.
장치의 Locale과 일치하는 값을 찾을 수 없으면 사용되는 
[`languageCode`]({{site.api}}/flutter/dart-ui/Locale/languageCode.html)가
일치하는 첫번째 `supportedLocales` 값이 사용됩니다.
이마저도 검색에 실패하면 `supportedLocales` 내역의 첫번째 항목이 사용됩니다.

이전의 DemoApp 예제는 오직 US English 또는 French Canadian Locale만 지원하며
그 외의 Locale에 대해서는 US English(리스트의 첫번째 Locale)이 대신 사용됩니다.

이와 다른 Locale 검색 알고리즘을 원하는 경우에는
[`localeResolutionCallback`.]({{site.api}}/flutter/widgets/LocaleResolutionCallback.html)을
사용할 수 있습니다. 예를들어, 사용자가 선택한 Locale을 무조건 수락하는 앱은
다음과 같이 작성합니다:

{% prettify dart %}
class DemoApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
       localeResolutionCallback: (Locale locale, Iterable<Locale> supportedLocales) {
         return locale;
       }
       // ...
    );
  }
}
{% endprettify %}

<a name="alternative-class"></a>
## 앱의 지역화된 리소스에 대한 대체 클래스

이전의 DemoApp 예제는 Dart `intl` 패키지를 사용하는 형태로 정의되었습니다.
개발자는 단순함을 위해 혹은 다른 i18n 프레임워크와 통합하기 위한 목적으로
지역화된 리소스를 관리하는 고유의 접근법을 선택할 수 있습니다.

예제 App [소스 코드]({{site.github}}/flutter/website/tree/master/examples/internationalization/minimal)

이번 DemoApp 버전은 DemoLocalizations 클래스에서 지역화를 다루며,
Map에 언어 별로 직접 번역된 모든 정보를 저장하고 있습니다.


{% prettify dart %}
class DemoLocalizations {
  DemoLocalizations(this.locale);

  final Locale locale;

  static DemoLocalizations of(BuildContext context) {
    return Localizations.of<DemoLocalizations>(context, DemoLocalizations);
  }

  static Map<String, Map<String, String>> _localizedValues = {
    'en': {
      'title': 'Hello World',
    },
    'es': {
      'title': 'Hola Mundo',
    },
  };

  String get title {
    return _localizedValues[locale.languageCode]['title'];
  }
}
{% endprettify %}

`DemoLocalizationsDelegate` 클래스도 살짝 다릅니다.
사용되고 있는 비동기 로딩이 존재하지 않기 때문에 `load` 함수는 
[SynchronousFuture]({{site.api}}/flutter/foundation/SynchronousFuture-class.html)를
반환하고 있습니다.

{% prettify dart %}
class DemoLocalizationsDelegate extends LocalizationsDelegate<DemoLocalizations> {
  const DemoLocalizationsDelegate();

  @override
  bool isSupported(Locale locale) => ['en', 'es'].contains(locale.languageCode);

  @override
  Future<DemoLocalizations> load(Locale locale) {
    return SynchronousFuture<DemoLocalizations>(DemoLocalizations(locale));
  }

  @override
  bool shouldReload(DemoLocalizationsDelegate old) => false;
}
{% endprettify %}

<a name="adding-language"></a>
## 새 언어에 대한 지원 추가

[GlobalMaterialLocalizations]({{site.api}}/flutter/flutter_localizations/GlobalMaterialLocalizations-class.html)에
포함되지 않는 언어를 지원해야하는 경우에는 추가적인 작업을 진행해야만 합니다:
`GlobalMaterialLocalizations`는 단어나 문장에 대한 약 70개의 번역("지역화")을 제공하고 있습니다.

벨라루스 언어 지원을 추가하기 위한 예제를 살펴봅시다.

새로운 `GlobalMaterialLocalizations` 하위클래스는 Material 라이브러리가
의존하는 지역화 정보들을 정의합니다. 새로운 `LocalizationsDelegate` 하위클래스는 
`GlobalMaterialLocalizations`를 위한 팩토리로서 정의되어야 합니다.

[`add_language` 예제 코드]({{site.github}}/flutter/website/tree/master/examples/internationalization/add_language/lib/main.dart)는
벨라루스 번역을 제외한 소스코드를 제공합니다.

벨라루스 Locale에 대한 `GlobalMaterialLocalizations` 구현은 `BeMaterialLocalizations` 클래스로
작성되었습니다. `LocalizationsDelegate`에 대한 구현은 `_BeMaterialLocalizationsDelegate`로 작성되었습니다.
`BeMaterialLocalizations.delegate`의 값은 `_BeMaterialLocalizationsDelegate`의 인스턴스이며,
이것이 지역화를 위해 필요한 모든 것입니다.

Delegate 클래스에는 기본적인 숫자와 날짜 포멧 지역화도 포함하고 있습니다 
이를 제외한 다른 모든 지역화는 `BeMaterialLocalizations` 내에서
`String` 타입의 Getter 속성으로 정의되어 있습니다:

{% prettify dart %}
@override
String get backButtonTooltip => r'Back';

@override
String get cancelButtonLabel => r'CANCEL';

@override
String get closeButtonLabel => r'CLOSE';

// etc..
{% endprettify %}


예제 소스코드에서는 모두 영어로 작성되어 있습니다.
작업을 마무리하기 위해 필요한 작업은 각각의 Getter 속성을 
적합한 벨라루스 언어로 변경하는 것입니다.

Getter는 `r'About $applicationName'`과 같이 r 접두사가 있는
"Raw" String 값을 반환합니다. 왜냐하면 때때로 문자열에 `$` 접두사와 함께
변수가 포함될 수 있기 때문입니다.
이 변수들은 매개변수 있는 지역화 메서드의 형태로 전달받을 수 있습니다:
{% prettify dart %}
@override
String get aboutListTileTitleRaw => r'About $applicationName';

@override
String aboutListTileTitle(String applicationName) {
  final String text = aboutListTileTitleRaw;
  return text.replaceFirst(r'$applicationName', applicationName);
}

{% endprettify %}

지역화 문자열에 대한 자세한 정보를 원하시면 [flutter_localizations README]({{site.github}}/flutter/flutter/blob/master/packages/flutter_localizations/lib/src/l10n/README.md)를 
확인해주세요.

원하는 언어를 구현하는 
`GlobalMaterialLocalizations`와 `LocalizationsDelegate`의 하위클래스를 
구현했다면, 이제 필요한건 앱에 언어와 Delegate 인스턴스를 추가하는 것입니다.
아래는 앱에 벨라루스 언어를 추가하고 `BeMaterialLocalizations` Delegate 인스턴스를
`localizationsDelegates` 리스트에 추가한 코드입니다:

{% prettify dart %}
MaterialApp(
  localizationsDelegates: [
    GlobalWidgetsLocalizations.delegate,
    GlobalMaterialLocalizations.delegate,
    BeMaterialLocalizations.delegate,
  ],
  supportedLocales: [
    const Locale('be', 'BY')
  ],
  home: ...
)
{% endprettify %}

<a name="dart-tools"></a>
## 부록: Dart intl 도구 사용

[`intl`]({{site.pub-pkg}}/intl) 패키지를 사용하는 API를 작성하기 전에
먼저 `intl` 패키지 문서를 읽고 필요한 사전준비 작업을 확인해주세요.
여기에서는 `intl` 패키지에 의존하는 지역화 앱을 위해 필요한 절차를 간단히 안내합니다.

데모 앱에서는 `l10n/messages_all.dart` 소스파일을 참조하고 있습니다.
이 파일은 자동생성된 파일로서 앱에서 사용되는 모든 지역화 문자열을 포함하고 있습니다.

`l10n/messages_all.dart`를 다시 생성하려면 2가지 작업을 진행해야 합니다.

 1. 터미널에서 앱의 루트 디렉터리로 이동한 후 다음 명령어를 입력하면,
    `lib/main.dart` 파일을 분석한 후 `l10n/intl_messages.arb`파일을 생성해줍니다:

    ```terminal
    $ flutter pub run intl_translation:extract_to_arb --output-dir=lib/l10n lib/main.dart
    ```

    `intl_messages.arb` 파일은 JSON 포멧으로 작성되어 있으며
	`main.dart`에 정의되어 있는	`Intl.message()` 함수 호출과 각각 매핑되어 있습니다.
	이 파일은 번역을 위한 템플릿으로 제공되는 것입니다. 만약 영어와 스페인어로 번역하고자 한다면
	이 파일을 복사하여 `intl_en.arb`와 `intl_es.arb`을 생성하신 후 번역 정보를 입력해주세요.

 2. 터미널에서 앱의 루트 디렉터리에서 다음 명령어를 입력해주세요.
    그러면 위에서 작성한 `intl_<locale>.arb` 파일들을
	`intl_messages_<locale>.dart` 파일로 변환해주며
	모든 언어를 포함하는 `intl_messages_all.dart`도 생성됩니다:

    ```terminal
    $ flutter pub run intl_translation:generate_from_arb \
        --output-dir=lib/l10n --no-use-deferred-loading \
        lib/main.dart lib/l10n/intl_*.arb
    ```
    
    ***윈도우 운영체제는 파일명에 대한 와일드카드를 지원하지 않습니다.***
    대신 `intl_translation:extract_to_arb` 명령에 의해 생성된 .arb 파일을 나열해주세요.
    ```terminal
    $ flutter pub run intl_translation:generate_from_arb \
        --output-dir=lib/l10n --no-use-deferred-loading \
        lib/main.dart \
        lib/l10n/intl_en.arb lib/l10n/intl_fr.arb lib/l10n/intl_messages.arb
    ```

    `DemoLocalizations` 클래스에서는 지역화된 문자열들을 로드하기 위해
    `intl_messages_all.dart` 파일에 정의된 `initializeMessages()` 함수를 호출합니다.
	`Intl.message()` 함수들은 해당하는 지역화된 문자열 값을 검색하여 리턴합니다.


<a name="ios-specifics"></a>
## 부록: iOS 앱 번들 업데이트

iOS 애플리케이션은 `Info.plist`파일에 메타데이터 Key를 정의합니다.
이 메타데이터에는 지원하는 언어에 대한 설정을 포함하고 있습니다.
앱에서 지원하는 Locale를 설정하려면 이 파일을 수정해야 합니다.

먼저 프로젝트의 `ios/Runner.xcworkspace` Xcode 워크스페이스 파일을 열어주세요.
그리고 **Project Navigator**에서 `Runner` 프로젝트의 `Runner` 폴더에 존재하는
`Info.plist` 파일을 열어주세요.

다음으로 **Information Property List**를 선택하고
**Editor** 메뉴에서 **Add Item**을 선택해주세요. 
팝업 메뉴에서 **Localizations**을 선택하세요.

새롭게 생성된 `Localizations` 항목을 선택하고 확장해주세요.
앱이 지원해야하는 Locale 값을 추가하기 위해
새로운 항목을 추가하고 **Value** 필드의 팝업 메뉴에서 
Locale을 선택해주세요.
여기서 설정한 내역은 플러터의 [supportedLocales](#specifying-supportedlocales) 속성과
동일해야 합니다.

모든 Locale이 추가되었으면 파일을 저장해주세요.
