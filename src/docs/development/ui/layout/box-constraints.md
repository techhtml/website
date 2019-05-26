---
title: Box Constraint 다루기
short-title: Box constraints
---

{{site.alert.note}}
  프레임워크에서 box constraint 관련 문제를 발견하면 이 페이지를 참고해주세요.
{{site.alert.end}}

Flutter에서, 위젯은 기본 [`RenderBox`]({{site.api}}/flutter/rendering/RenderBox-class.html) 객체에 의해 렌더링 됩니다.
렌더링 박스는 부모에 의해 제약 사항이 주어지며, 해당 조건 내에서 크기가 조정됩니다.
제약 사항은 최소 너비와 최대 너비 그리고 높이로 구성되며, 크기는 특정 넓이와 높이로 구성됩니다.

일반적으로 제약 사항을 조정하는 방법에는 세 종류의 박스가 있는데:

- 최대한 크게 하려는 경우.
  예를 들어, [`Center`]({{site.api}}/flutter/widgets/Center-class.html) 및
  [`ListView`]({{site.api}}/flutter/widgets/ListView-class.html)에 사용하는 박스들이 있습니다.
- 하위 요소들과 같은 크기로 만들려는 경우.
  예를 들어, [`Transform`]({{site.api}}/flutter/widgets/Transform-class.html) 및
  [`Opacity`]({{site.api}}/flutter/widgets/Opacity-class.html)에 사용하는 박스들이 있습니다.
- 개별적인 크기를 갖게 하려는 경우.
  예를 들어, [`Image`]({{site.api}}/flutter/dart-ui/Image-class.html) 및
  [`Text`]({{site.api}}/flutter/widgets/Text-class.html) 에 사용하는 박스들이 있습니다.

[`Container`]({{site.api}}/flutter/widgets/Container-class.html)와 같은 일부 위젯들은 생성자의 인자에 따라 종류별로 달라집니다.
[`Container`]({{site.api}}/flutter/widgets/Container-class.html)의 경우,
최대한 크게 하는 것이 기본 값이지만, `넓이`를 줄 경우 해당 값을 우선적으로 가지게 됩니다.

다른 경우를 예로 들면 [`Row`]({{site.api}}/flutter/widgets/Row-class.html)와
[`Column`]({{site.api}}/flutter/widgets/Column-class.html)은
아래 "Flex" 섹션에서 설명한대로, 주어진 제약 사항에 따라 달라집니다.

때때로 "엄격한" 제약 사항은 렌더링 박스가 크기를 결정할 수 있는 여지를 남기지 않는데요.
(e.g. 최소 너비와 최대 너비가 같을 때, 엄격한 넓이를 가진다고 말합니다)
그 대표적인 예가 [`RenderView`]({{site.api}}/flutter/rendering/RenderView-class.html) 클래스가 포함된 `App` 위젯인데:
어플리케이션의 [`build`]({{site.api}}/flutter/widgets/State/build.html) 함수에 의해
반한되는 자식에 사용하는 박스에는 컨텐츠 영역(일반적으로 전체 화면)을 전부 채우도록 하는 제약 사항이 주어집니다.

Flutter에 있는 많은 상자들 중 하나의 자식만을 가지는 박스는 자식에게 제약 사항을 전달하는데요.
말인즉슨 어플리케이션 렌더 트리의 루트에 여러 박스들을 배치하면, 이런 엄격한 제약 사항들에 의해 서로 정확히 들어맞을 겁니다.

일부 박스들은 제약 사항을 _느슨하게_ 하는데, 이는 최대치는 유지되지만 최소치는 제거된다는 걸 뜻합니다.
예로는 [`Center`]({{site.api}}/flutter/widgets/Center-class.html)가 있습니다.

Unbounded constraints
---------------------

특정한 상황에서, 박스에 주어지는 제약 사항은 _제한되지 않거나_ 무한한데요.
이 말은 최대 너비 혹은 최대 높이가 `double.INFINITY`로 설정된다는 걸 뜻합니다.

가능한 만큼 커지는 박스는 제한되지 않은 제약 사항이 주어질 때 적합하지 않고,
디버그 모드에서 이렇게 조합하면 해당 파일을 가르키는 예외를 던집니다.

제한되지 않은 제약 사항을 가지는 렌더링 박스를 보게 되는 일반적인 경우는
플렉스 박스([`Row`]({{site.api}}/flutter/widgets/Row-class.html)
및 [`Column`]({{site.api}}/flutter/widgets/Column-class.html))와
**스크롤 가능 영역**([`ListView`]({{site.api}}/flutter/widgets/ListView-class.html)
및 다른 [`ScrollView`]({{site.api}}/flutter/widgets/ScrollView-class.html) 하위 클래스) 내부인데요.

특히, [`ListView`]({{site.api}}/flutter/widgets/ListView-class.html)는 사용할 수 있는
공간에 맞게 가로 방향으로 확장됩니다. (예를 들어, 세로 스크롤 블록의 경우 부모와 같은 크기로 넓어집니다.)
만약 가로 스크롤 [`ListView`]({{site.api}}/flutter/widgets/ListView-class.html) 안에
세로 스크롤 [`ListView`]({{site.api}}/flutter/widgets/ListView-class.html)를 배치하면
내부에 있는 뷰는 가능한 만큼 넓어지는데, 이는 외부 뷰에서 해당 방향으로 스크롤할 수 있기 때문입니다.

Flex
----

Flex boxes themselves
([`Row`]({{site.api}}/flutter/widgets/Row-class.html)
and [`Column`]({{site.api}}/flutter/widgets/Column-class.html))
behave differently based on
whether they are in a bounded constraints or unbounded constraints in
their given direction.

In bounded constraints, they try to be as big as possible in that
direction.

In unbounded constraints, they try to fit their children in that
direction. In this case, you cannot set `flex` on the children to
anything other than 0 (the default). In the widget library, this
means that you cannot use [`Expanded`]({{site.api}}/flutter/widgets/Expanded-class.html)
when the flex box is inside
another flex box or inside a scrollable. If you do, you'll get an
exception message pointing you at this document.

In the _cross_ direction, i.e. in their width for
[`Column`]({{site.api}}/flutter/widgets/Column-class.html)
(vertical flex) and in their height for
[`Row`]({{site.api}}/flutter/widgets/Row-class.html)
(horizontal flex), they must never be unbounded,
otherwise they would not be able to reasonably align their children.
