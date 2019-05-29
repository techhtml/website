---
title: 레이아웃 튜토리얼
short-title: 튜토리얼
description: 레이아웃을 만드는 방법을 배웁니다.
diff2html: true
---

{% assign api = '{{site.api}}/flutter' -%}
{% capture examples -%} {{site.repo.this}}/tree/{{site.branch}}/examples {%- endcapture -%}
{% assign rawExFile = 'https://raw.githubusercontent.com/flutter/website/master/examples' -%}
{% capture demo -%} {{site.repo.flutter}}/tree/{{site.branch}}/examples/flutter_gallery/lib/demo {%- endcapture -%}

<style>dl, dd { margin-bottom: 0; }</style>

{{site.alert.secondary}}
  <h4 class="no_toc">다음을 배우게 됩니다.</h4>

  * Flutter의 레이아웃 메커니즘이 동작하는 방식.
  * 위젯을 수평과 수직으로 배치하는 방법.
  * Flutter 레이아웃을 만드는 방법.
{{site.alert.end}}

이 문서는 Flutter에서 레이아웃을 작성하는 방법을 소개합니다.
You'll build the layout for the following app.

{% include app-figure.md img-class="site-mobile-screenshot border"
    image="ui/layout/lakes.jpg" caption="The finished app" %}

Flutter의 레이아웃을 다루기 전에,
화면 위에 하나의 위젯을 배치하는 방법을 살펴보도록 하겠습니다.
위젯을 수평, 수직으로 배치하는 방법을 논의한 다음에는 가장 일반적인 레이아웃 위젯도 다뤄보겠습니다.

레이아웃 메커니즘을 이해하기 위한 "큰 그림"을 알고싶으신 경우라면,
[Flutter's approach to layout](/docs/development/ui/layout)을 찾아보세요.

## Step 0: 앱 기본 코드 만들기

[set up](/docs/get-started/install) 과정을 통해 개발 환경을 잘 구성하셨다면,
다음의 순서를 따라보세요.

 1. [Create a basic "Hello World" Flutter app][hello-world].
 2. 아래의 코드를 적용해서 app bar title과 app title을 바꿔보세요.

    <?code-excerpt "{codelabs/startup_namer/step1_base,layout/base}/lib/main.dart"?>
    ```diff
    --- codelabs/startup_namer/step1_base/lib/main.dart
    +++ layout/base/lib/main.dart
    @@ -6,10 +6,10 @@
       @override
       Widget build(BuildContext context) {
         return MaterialApp(
    -      title: 'Welcome to Flutter',
    +      title: 'Flutter layout demo',
           home: Scaffold(
             appBar: AppBar(
    -          title: Text('Welcome to Flutter'),
    +          title: Text('Flutter layout demo'),
             ),
             body: Center(
               child: Text('Hello World'),
    ```

[hello-world]: /docs/get-started/codelab#step-1-create-the-starter-flutter-app

## Step 1: 레이아웃 그리기

첫 단계로 레이아웃을 기본 요소들로 나눠보겠습니다.

* Row와 Column 레이아웃을 확인합니다.
* 레이아웃이 그리드를 포함하나요?
* 겹쳐친 요소가 있나요?
* UI에 탭이 필요한가요?
* 정렬, 패딩, 보더가 필요한 영역에 주의합니다.

먼저, 더 큰 요소들을 구분합니다. 이 예제에서는 4개의 요소를 가지고 있는데, 
이미지 하나, 두 개의 row 레이아웃, 텍스트 영역 하나입니다.

{% include app-figure.md img-class="site-mobile-screenshot border"
    image="ui/layout/lakes-column-elts.png" caption="Column elements (circled in red)" %}

다음에는 각 row를 그려봅니다. 첫 번째 row 타이틀 영역으로 
텍스트 column 하나, 별 아이콘 하나, 그리고 숫자를 가지고 있습니다.
첫 번째 자식 요소는 두 줄짜리 텍스트 column입니다.
이 column은 넓은 영역을 차지하고 있기 때문에, 
Expanded 위젯을 사용해야 합니다.

{% include app-figure.md image="ui/layout/title-section-parts.png" alt="Title section" %}

두 번째 row는 세 개의 자식 요소를 가지고 있고, 
각 요소는 아이콘과 텍스트를 가지고 있는 column 입니다.

{% include app-figure.md image="ui/layout/button-section-diagram.png" alt="Button section" %}

레이아웃을 다 그렸으면, 가장 하위 요소부터 구현합니다.
깊이 파묻힌 레이아웃 코드의 시각적 혼란을 
최소화 하기 위해서, 적당히 함수와 변수을 이용해서 구현합니다.

## Step 2: 타이틀 row 구현하기

<?code-excerpt path-base="layout/lakes/step2"?>

먼저, 타이틀 영역의 외쪽 column을 만듭니다.

<?code-excerpt "lib/main.dart (titleSection)" title?>
```dart
Widget titleSection = Container(
  padding: const EdgeInsets.all(32),
  child: Row(
    children: [
      Expanded(
        /*1*/
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            /*2*/
            Container(
              padding: const EdgeInsets.only(bottom: 8),
              child: Text(
                'Oeschinen Lake Campground',
                style: TextStyle(
                  fontWeight: FontWeight.bold,
                ),
              ),
            ),
            Text(
              'Kandersteg, Switzerland',
              style: TextStyle(
                color: Colors.grey[500],
              ),
            ),
          ],
        ),
      ),
      /*3*/
      Icon(
        Icons.star,
        color: Colors.red[500],
      ),
      Text('41'),
    ],
  ),
);
```

{:.numbered-code-notes}
 1. Putting a Column inside an Expanded widget stretches the column to use all
    remaining free space in the row. Setting the `crossAxisAlignment` property to
    `CrossAxisAlignment.start` positions the column at the start of the row.
 2. Putting the first row of text inside a Container enables you to add padding.
    The second child in the Column, also text, displays as grey.
 3. The last two items in the title row are a star icon, painted red,
    and the text "41". The entire row is in a Container and padded
    along each edge by 32 pixels.

Add the title section to the app body like this:

<?code-excerpt path-base="layout/lakes"?>
<?code-excerpt "{../base,step2}/lib/main.dart" from="return MaterialApp"?>
```diff
--- ../base/lib/main.dart
+++ step2/lib/main.dart
@@ -8,11 +46,13 @@
     return MaterialApp(
       title: 'Flutter layout demo',
       home: Scaffold(
         appBar: AppBar(
           title: Text('Flutter layout demo'),
         ),
-        body: Center(
-          child: Text('Hello World'),
+        body: Column(
+          children: [
+            titleSection,
+          ],
         ),
       ),
     );
```

{{site.alert.tip}}
  - When pasting code into your app, indentation can
    become skewed. You can fix this in your Flutter editor
    using the [automatic reformatting support](/docs/development/tools/formatting).
  - For a faster development experience, try Flutter's [hot reload][] feature.
  - If you have problems, compare your code to [lib/main.dart][].

  [hot reload]: /docs/development/tools/hot-reload
  [lib/main.dart]: {{examples}}/layout/lakes/step2/lib/main.dart
{{site.alert.end}}

## Step 3: Implement the button row

<?code-excerpt path-base="layout/lakes/step3"?>

The button section contains 3 columns that use the same layout&mdash;an
icon over a row of text. The columns in this row are evenly spaced,
and the text and icons are painted with the primary color.

Since the code for building each column is almost identical, create a private
helper method named `buildButtonColumn()`, which takes a color, an Icon and
Text, and returns a column with its widgets painted in the given color.

<?code-excerpt "lib/main.dart (_buildButtonColumn)" title?>
```dart
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // ···
  }

  Column _buildButtonColumn(Color color, IconData icon, String label) {
    return Column(
      mainAxisSize: MainAxisSize.min,
      mainAxisAlignment: MainAxisAlignment.center,
      children: [
        Icon(icon, color: color),
        Container(
          margin: const EdgeInsets.only(top: 8),
          child: Text(
            label,
            style: TextStyle(
              fontSize: 12,
              fontWeight: FontWeight.w400,
              color: color,
            ),
          ),
        ),
      ],
    );
  }
}
```

The function adds the icon directly to the column. The text is inside a
Container with a top-only margin, separating the text from the icon.

Build the row containing these columns by calling the function and passing the
color, `Icon`, and text specific to that column. Align the columns along the main axis
using `MainAxisAlignment.spaceEvenly` to arrange the free space evenly before,
between, and after each column. Add the following code just below the
`titleSection` declaration inside the `build()` method:

<?code-excerpt "lib/main.dart (buttonSection)" title?>
```dart
Color color = Theme.of(context).primaryColor;

Widget buttonSection = Container(
  child: Row(
    mainAxisAlignment: MainAxisAlignment.spaceEvenly,
    children: [
      _buildButtonColumn(color, Icons.call, 'CALL'),
      _buildButtonColumn(color, Icons.near_me, 'ROUTE'),
      _buildButtonColumn(color, Icons.share, 'SHARE'),
    ],
  ),
);
```

Add the button section to the body:

<?code-excerpt path-base="layout/lakes"?>
<?code-excerpt "{step2,step3}/lib/main.dart" from="return MaterialApp" to="}"?>
```diff
--- step2/lib/main.dart
+++ step3/lib/main.dart
@@ -46,3 +59,3 @@
     return MaterialApp(
       title: 'Flutter layout demo',
       home: Scaffold(
@@ -52,8 +65,9 @@
         body: Column(
           children: [
             titleSection,
+            buttonSection,
           ],
         ),
       ),
     );
   }
```

## Step 4: Implement the text section

<?code-excerpt path-base="layout/lakes/step4"?>

Define the text section as a variable. Put the text in a Container and add
padding along each edge. Add the following code just below the `buttonSection`
declaration:

<?code-excerpt "lib/main.dart (textSection)" title?>
```dart
Widget textSection = Container(
  padding: const EdgeInsets.all(32),
  child: Text(
    'Lake Oeschinen lies at the foot of the Blüemlisalp in the Bernese '
        'Alps. Situated 1,578 meters above sea level, it is one of the '
        'larger Alpine Lakes. A gondola ride from Kandersteg, followed by a '
        'half-hour walk through pastures and pine forest, leads you to the '
        'lake, which warms to 20 degrees Celsius in the summer. Activities '
        'enjoyed here include rowing, and riding the summer toboggan run.',
    softWrap: true,
  ),
);
```

By setting `softwrap` to true, text lines will fill the column width before
wrapping at a word boundary.

Add the text section to the body:

<?code-excerpt path-base="layout/lakes"?>
<?code-excerpt "{step3,step4}/lib/main.dart" from="return MaterialApp"?>
```diff
--- step3/lib/main.dart
+++ step4/lib/main.dart
@@ -59,3 +72,3 @@
     return MaterialApp(
       title: 'Flutter layout demo',
       home: Scaffold(
@@ -66,6 +79,7 @@
           children: [
             titleSection,
             buttonSection,
+            textSection,
           ],
         ),
       ),
```

## Step 5: Implement the image section

Three of the four column elements are now complete, leaving only the image.
Add the image file to the example:

* Create an `images` directory at the top of the project.
* Add [`lake.jpg`]({{rawExFile}}/layout/lakes/step5/images/lake.jpg).

  {{site.alert.info}}
    Note that `wget` doesn't work for saving this binary file. The original image
    is [available online][] under a Creative Commons license, but it's large and
    slow to fetch.

    [available online]: https://images.unsplash.com/photo-1471115853179-bb1d604434e0?dpr=1&amp;auto=format&amp;fit=crop&amp;w=767&amp;h=583&amp;q=80&amp;cs=tinysrgb&amp;crop=
  {{site.alert.end}}

* Update the `pubspec.yaml` file to include an `assets` tag. This makes the
  image available to your code.

  <?code-excerpt "{step4,step5}/pubspec.yaml"?>
  ```diff
  --- step4/pubspec.yaml
  +++ step5/pubspec.yaml
  @@ -17,3 +17,5 @@

   flutter:
     uses-material-design: true
  +  assets:
  +    - images/lake.jpg
  ```

Now you can reference the image from your code:

<?code-excerpt "{step4,step5}/lib/main.dart"?>
```diff
--- step4/lib/main.dart
+++ step5/lib/main.dart
@@ -77,6 +77,12 @@
         ),
         body: Column(
           children: [
+            Image.asset(
+              'images/lake.jpg',
+              width: 600,
+              height: 240,
+              fit: BoxFit.cover,
+            ),
             titleSection,
             buttonSection,
             textSection,
```

`BoxFit.cover` tells the framework that the image should be as small as
possible but cover its entire render box.

## Step 6: Final touch

In this final step, arrange all of the elements in a `ListView`, rather than a
`Column`, because a `ListView` supports app body scrolling when the app is run
on a small device.

<?code-excerpt "{step5,step6}/lib/main.dart" diff-u="6" from="return MaterialApp"?>
```diff
--- step5/lib/main.dart
+++ step6/lib/main.dart
@@ -72,13 +77,13 @@
     return MaterialApp(
       title: 'Flutter layout demo',
       home: Scaffold(
         appBar: AppBar(
           title: Text('Flutter layout demo'),
         ),
-        body: Column(
+        body: ListView(
           children: [
             Image.asset(
               'images/lake.jpg',
               width: 600,
               height: 240,
               fit: BoxFit.cover,
```

**Dart code:** [main.dart]({{examples}}/layout/lakes/step6/lib/main.dart)<br>
**Image:** [images]({{examples}}/layout/lakes/step6/images)<br>
**Pubspec:** [pubspec.yaml]({{examples}}/layout/lakes/step6/pubspec.yaml)

That's it! When you hot reload the app, you should see the same app layout as
the screenshot at the top of this page.

You can add interactivity to this layout by following [Adding Interactivity to
Your Flutter App](/docs/development/ui/interactive).
