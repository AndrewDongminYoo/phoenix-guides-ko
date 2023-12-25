# Components and HEEx

> **요구 사항**: 이 가이드는 사용자가 [소개 가이드](installation.html)를 살펴보고 Phoenix 애플리케이션을 [가동 및 실행](up_and_running.html)한 것으로 가정합니다.
>
> **요구 사항**: 이 가이드는 사용자가 [요청 수명 주기 가이드](request_lifecycle.html)를 살펴봤을 것으로 예상합니다.

Phoenix 엔드포인트 파이프라인은 요청을 받아 컨트롤러로 라우팅하고 보기 모듈을 호출하여 템플릿을 렌더링합니다.
컨트롤러의 보기 인터페이스는 간단합니다. 컨트롤러는 연결이 할당된 보기 함수를 호출하고, 이 함수의 작업은 HEEx 템플릿을 반환하는 것입니다.
'assigns' 매개변수를 받아들이고 HEEx 템플릿을 반환하는 함수를 _function component_라고 부릅니다.
함수 컴포넌트는 [`Phoenix.Component`](https://hexdocs.pm/phoenix_live_view/Phoenix.Component.html) 모듈의 도움으로 정의됩니다.

함수 컴포넌트는 Phoenix에서 수행하게 될 모든 종류의 마크업 기반 템플릿 렌더링에 필수적인 구성 요소입니다.
이 컴포넌트는 표준 MVC 컨트롤러 기반 애플리케이션, 라이브뷰 애플리케이션, 레이아웃 및 다른 템플릿 전체에서 사용할 작은 UI 정의에 대한 공유 추상화 역할을 합니다.

이 장에서는 이전 장에서 컴포넌트가 어떻게 사용되었는지 요약하고 새로운 사용 사례를 찾아보겠습니다.

## Function components

요청 수명 주기 챕터의 마지막에 `lib/hello_web/controllers/hello_html/show.html`에 템플릿을 만들었습니다.heex`, let's open it up:

```perl HEEx
<section>
  <h2>Hello World, from <%= @messenger %>!</h2>
</section>
```

이 템플릿은 `lib/hello_web/controllers/hello_html`에서 `HelloHTML`의 일부로 임베드됩니다.ex`:

```perl Elixir
defmodule HelloWeb.HelloHTML do
  use HelloWeb, :html

  embed_templates "hello_html/*"
end
```

충분히 간단합니다.
단지 두 줄, `use HelloWeb, :html`만 있으면 됩니다.
이 줄은 함수 컴포넌트와 템플릿에 대한 기본 임포트와 구성을 설정하는 `HelloWeb`에 정의된 `html/0` 함수를 호출합니다.

모듈에서 만든 모든 임포트와 별칭은 템플릿에서도 사용할 수 있습니다.
템플릿은 해당 모듈 내부의 함수로 효과적으로 컴파일되기 때문입니다.
예를 들어 모듈에서 함수를 정의하면 템플릿에서 바로 함수를 호출할 수 있습니다.
이를 실제로 살펴봅시다.

`show.html.heex`를 리팩터링하여 `<h2>Hello World, from <%= @messenger %>!</h2>`의 렌더링을 자체 함수로 옮기고 싶다고 가정해 보겠습니다.
HelloHTML` 내부의 함수 컴포넌트로 이동해 보겠습니다:

```perl Elixir
defmodule HelloWeb.HelloHTML do
  use HelloWeb, :html

  embed_templates "hello_html/*"

  attr :messenger, :string, required: true

  def greet(assigns) do
    ~H"""
    <h2>Hello World, from <%= @messenger %>!</h2>
    """
  end
end
```

`Phoenix.Component`에서 제공하는 `attr/3` 매크로를 통해 허용하는 어트리뷰트를 선언한 다음, HEEx 템플릿을 반환하는 `greet/1` 함수를 정의했습니다.

다음으로 `show.html.heex`을 업데이트해야 합니다:

```perl Elixir
<section>
  <.greet messenger={@messenger} />
</section>
```

`http://localhost:4000/hello/Frank`를 다시 로드하면 이전과 동일한 내용이 표시될 것입니다.

템플릿이 `HelloHTML` 모듈 안에 내장되어 있기 때문에 `<.greet messenger="..." />`로 간단히 보기 함수를 호출할 수 있었습니다.

컴포넌트가 다른 곳에 정의된 경우 `<HelloWeb.HelloHTML.greet messenger="..." />`를 입력할 수도 있습니다.

어트리뷰트를 필요에 따라 선언하면 어트리뷰트를 전달하지 않고 `<.greet />` 컴포넌트를 호출하면 컴파일 시점에 Phoenix가 경고합니다.
속성이 선택 사항인 경우 값과 함께 `:default` 옵션을 지정할 수 있습니다:

```perl Elixir
attr :messenger, :string, default: nil
```

이것은 간단한 예시이지만, 함수 컴포넌트가 Phoenix에서 수행하는 다양한 역할을 보여줍니다:

- 함수 컴포넌트는 `greet/1`에서와 같이 `assigns`를 인수로 받고 `~H` 시질을 호출하는 함수로 정의할 수 있습니다.

- 함수 컴포넌트는 템플릿 파일에서 임베드할 수 있으며, 이는 `show.html.heex`를 `HelloWeb.HelloHTML`에 로드하는 방식입니다.

- 함수 컴포넌트는 예상되는 어트리뷰트를 선언할 수 있으며, 컴파일 시 유효성이 검사됩니다.

- 함수 컴포넌트는 컨트롤러에서 직접 렌더링할 수 있습니다.

- 함수 컴포넌트는 다른 함수 컴포넌트에서 직접 렌더링할 수 있으며, `<.greet messenger={@messenger} />`를 `show.html.heex`에서 호출한 것처럼 말입니다.

그리고 더 있습니다.
더 깊이 들어가기 전에 HEEx 템플릿 언어의 표현력을 완전히 이해해 보겠습니다.

## HEEx

함수 컴포넌트와 템플릿 파일은 "HTML+EEx"의 약자인 [HEEx 템플릿 언어](https://hexdocs.pm/phoenix_live_view/Phoenix.Component.html#sigil_H/2)로 구동됩니다.
EEx는 `<%= 표현식 %>`을 사용하여 Elixir 표현식을 실행하고 그 결과를 템플릿에 보간하는 Elixir 라이브러리입니다.
이는 `@` 단축키를 통해 설정한 할당을 표시하는 데 자주 사용됩니다.
컨트롤러에서 호출하는 경우

```perl Elixir
  render(conn, :show, username: "joe")
```

그런 다음 템플릿에서 `<%= @사용자 이름 %>`로 해당 사용자 이름에 액세스할 수 있습니다.
어사인과 함수를 표시하는 것 외에도 거의 모든 엘릭서 표현식을 사용할 수 있습니다.
예를 들어, 조건문을 사용하려면

```perl HEEx
<%= if some_condition? do %>
  <p>Some condition is true for user: <%= @username %></p>
<% else %>
  <p>Some condition is false for user: <%= @username %></p>
<% end %>
```

or even loops:

```perl HEEx
<table>
  <tr>
    <th>Number</th>
    <th>Power</th>
  </tr>
  <%= for number <- 1..10 do %>
    <tr>
      <td><%= number %></td>
      <td><%= number * number %></td>
    </tr>
  <% end %>
</table>
```

위에서 `<%= %>`와 `<% %>`를 사용한 것을 눈치채셨나요? 템플릿에 무언가를 출력하는 모든 표현식은 반드시 등호 기호(`=`)를 사용해야 합니다.
이 기호를 포함하지 않으면 코드가 실행되기는 하지만 템플릿에 아무 것도 삽입되지 않습니다.

HEEx에는 다음에 배우게 될 편리한 HTML 확장 기능도 함께 제공됩니다.

### HTML extensions

`.heex` 템플릿은 `<%= %>`를 통해 Elixir 표현식의 보간을 허용하는 것 외에도 HTML 인식 확장을 제공합니다.
예를 들어 "<" 또는 ">"가 포함된 값을 보간하려고 하면 HTML 삽입으로 이어질 수 있으므로 어떤 일이 발생하는지 살펴보겠습니다:

```perl HEEx
<%= "<b>Bold?</b>" %>
```

템플릿을 렌더링하면 페이지에 리터럴 `<b>`가 표시됩니다.
이는 사용자가 페이지에 HTML 콘텐츠를 삽입할 수 없음을 의미합니다.
그렇게 하도록 허용하려면 `raw`를 호출할 수 있지만 매우 신중하게 하세요:

```perl HEEx
<%= raw "<b>Bold?</b>" %>
```

HEEx 템플릿의 또 다른 강점은 HTML의 유효성 검사와 속성의 간결한 보간 구문입니다.
다음처럼 쓸 수 있습니다:

```perl HEEx
<div title="My div" class={@class}>
  <p>Hello <%= @username %></p>
</div>
```

`키={값}`을 간단히 사용할 수 있는 방법을 주목하세요.
HEEx는 `false`와 같은 특수 값을 자동으로 처리하여 속성 또는 클래스 목록을 제거합니다.

키워드 목록 또는 맵에서 동적인 수의 속성을 보간하려면 다음을 수행합니다:

```perl HEEx
<div title="My div" {@many_attributes}>
  <p>Hello <%= @username %></p>
</div>
```

또한 닫는 `</div>`를 제거하거나 `</div-typo>`로 이름을 변경해 보세요.
HEEx 템플릿은 오류에 대해 알려줍니다.

HEEx는 특수 `:if` 및 `:for` 속성을 통해 `if` 및 `for` 표현식에 대한 속기 구문도 지원합니다.
예를 들면 다음과 같습니다:

```perl HEEx
<%= if @some_condition do %>
  <div>...</div>
<% end %>
```

다음처럼 쓸 수 있습니다:

```perl HEEx
<div :if={@some_condition}>...</div>
```

마찬가지로 구문을 위해 다음과 같이 작성할 수 있습니다:

```perl HEEx
<ul>
  <li :for={item <- @items}><%= item.name %></li>
</ul>
```

## Layouts

레이아웃은 함수 컴포넌트일 뿐입니다.
다른 모든 함수 컴포넌트 템플릿과 마찬가지로 모듈에 정의됩니다.
새로 생성된 앱에서는 `lib/hello_web/components/layouts.ex`입니다.
또한 `layouts` 폴더에는 Phoenix에서 생성된 두 개의 기본 제공 레이아웃이 있습니다.
기본 _root 레이아웃_은 `root.html.heex`라고 불리며, 모든 템플릿이 기본적으로 렌더링되는 레이아웃입니다.
두 번째는 `app.html.heex`라고 하는 _app 레이아웃_으로, 루트 레이아웃 내에서 렌더링되며 콘텐츠를 포함합니다.

렌더링된 뷰의 결과 문자열이 어떻게 레이아웃 안에 들어가는지 궁금할 수 있습니다.
좋은 질문입니다! lib/hello_web/components/layouts/root.html.heex`를 보면 `<body>`의 끝부분에 이런 내용이 있습니다.

```perl HEEx
<%= @inner_content %>
```

즉, 페이지를 렌더링한 후 결과가 `@inner_content` 어사인에 배치됩니다.

Phoenix는 어떤 레이아웃을 렌더링할지 제어할 수 있는 모든 종류의 편의를 제공합니다.
예를 들어, `Phoenix.Controller` 모듈은 _루트 레이아웃_을 전환할 수 있는 `put_root_layout/2` 함수를 제공합니다.
이 함수는 첫 번째 인수로 `conn`과 형식 및 해당 레이아웃의 키워드 목록을 받습니다.
레이아웃을 완전히 비활성화하려면 `false`로 설정하면 됩니다.

`lib/hello_web/controllers/hello_controller.ex`에서 `HelloController`의 `index` 액션을 다음과 같이 편집할 수 있습니다.

```perl Elixir
def index(conn, _params) do
  conn
  |> put_root_layout(html: false)
  |> render(:index)
end
```

[http://localhost:4000/hello](http://localhost:4000/hello)를 다시 로드하면 제목이나 CSS 스타일링이 전혀 없는 매우 다른 페이지를 볼 수 있습니다.

애플리케이션 레이아웃을 사용자 정의하기 위해 `put_layout/2`라는 유사한 함수를 호출합니다.
실제로 다른 레이아웃을 만들고 그 안에 인덱스 템플릿을 렌더링해 봅시다.
예를 들어, 애플리케이션의 관리자 섹션에 로고 이미지가 없는 다른 레이아웃이 있다고 가정해 보겠습니다.
이렇게 하려면 기존 `app.html.heex`를 동일한 디렉토리 `lib/hello_web/components/layouts`에 있는 새 파일 `admin.html.heex`에 복사합니다.
그런 다음 새 파일에서 `<header>...</header>` 태그 안의 모든 내용을 제거하거나 원하는 대로 변경합니다.

이제 `lib/hello_web/controllers/hello_controller`의 컨트롤러의 `index` 액션에서.ex`, add the following:

```perl Elixir
def index(conn, _params) do
  conn
  |> put_layout(html: :admin)
  |> render(:index)
end
```

페이지를 로드하면 헤더가 없는 관리자 레이아웃(또는 사용자가 작성한 사용자 정의 레이아웃)이 렌더링되어야 합니다.

이쯤 되면 왜 피닉스에 두 가지 레이아웃이 있는지 궁금할 것입니다.

우선, 유연성을 제공합니다.
실제로는 HTML 헤더만 포함하는 경우가 많기 때문에 루트 레이아웃이 여러 개 있는 경우는 거의 없습니다.
따라서 서로 다른 애플리케이션 레이아웃 간에 변경되는 부분만 가지고 다른 애플리케이션 레이아웃에 집중할 수 있습니다.
둘째, 피닉스는 서버 렌더링 HTML로 풍부한 실시간 사용자 경험을 구축할 수 있는 라이브뷰라는 기능을 제공합니다.
라이브뷰는 페이지의 콘텐츠를 동적으로 변경할 수 있지만 앱 레이아웃만 변경하고 루트 레이아웃은 변경하지 않습니다.
자세한 내용은 [라이브뷰 문서](https://hexdocs.pm/phoenix_live_view)를 참조하세요.

## CoreComponents

새로운 Phoenix 애플리케이션에서는 `components` 폴더 안에 `core_components.ex` 모듈도 찾을 수 있습니다.
이 모듈은 애플리케이션 전체에서 재사용할 함수 컴포넌트를 정의하는 좋은 예입니다.
이를 통해 애플리케이션이 발전함에 따라 컴포넌트가 일관성 있게 보이도록 보장합니다.

`lib/hello_web.ex`에 있는 `HelloWeb`의 `def html` 내부를 살펴보면 `CoreComponents`가 `use HelloWeb, :html`을 통해 모든 HTML 보기로 자동 임포트되는 것을 볼 수 있습니다.
이것이 바로 `CoreComponents` 자체가 맨 위에 `use HelloWeb, :html` 대신 `use Phoenix.Component`를 수행하는 이유이기도 합니다: 후자를 수행하면 `CoreComponents` 자체를 가져오려고 할 때 교착 상태가 발생할 수 있기 때문입니다.

코드 생성기는 애플리케이션을 빠르게 스캐폴딩하기 위해 해당 컴포넌트를 사용할 수 있다고 가정하기 때문에 CoreComponents는 Phoenix 코드 생성기에서도 중요한 역할을 합니다.
이 모든 작품에 대해 더 자세히 알아보고 싶으시다면 다음을 참조하세요:

- 생성된 `CoreComponents` 모듈을 살펴보고 실제 예제를 통해 자세히 알아보세요.

- [`피닉스 컴포넌트` 공식 문서](https://hexdocs.pm/phoenix_live_view/Phoenix.Component.html) 읽기

- [HEEx 및 ~H sigils 공식 문서](https://hexdocs.pm/phoenix_live_view/Phoenix.Component.html#sigil_H/2) 읽기
