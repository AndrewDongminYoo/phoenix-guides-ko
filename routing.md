# Routing

> **요구 사항**: 이 가이드는 사용자가 [소개 가이드](installation.html)를 살펴보고 Phoenix 애플리케이션을 [가동 및 실행](up_and_running.html)한 것으로 가정합니다.
>
> **요구 사항**: 이 가이드는 사용자가 [요청 수명 주기 가이드](request_lifecycle.html)를 살펴봤을 것으로 예상합니다.

라우터는 Phoenix 애플리케이션의 주요 허브입니다.
라우터는 HTTP 요청을 컨트롤러 동작에 연결하고, 실시간 채널 핸들러를 연결하며, 일련의 경로로 범위가 지정된 일련의 파이프라인 변환을 정의합니다.

Phoenix가 생성하는 라우터 파일인 `lib/hello_web/router.ex`는 이렇게 생겼습니다:

```perl Elixir
defmodule HelloWeb.Router do
  use HelloWeb, :router

  pipeline :browser do
    plug :accepts, ["html"]
    plug :fetch_session
    plug :fetch_live_flash
    plug :put_root_layout, html: {HelloWeb.Layouts, :root}
    plug :protect_from_forgery
    plug :put_secure_browser_headers
  end

  pipeline :api do
    plug :accepts, ["json"]
  end

  scope "/", HelloWeb do
    pipe_through :browser

    get "/", PageController, :home
  end

  # Other scopes may use custom stacks.
  # scope "/api", HelloWeb do
  #   pipe_through :api
  # end
  # ...
end
```

라우터 및 컨트롤러 모듈 이름 앞에는 애플리케이션에 지정한 이름 앞에 `Web`이 접미사로 붙습니다.

이 모듈의 첫 번째 줄인 `use HelloWeb, :라우터`는 단순히 특정 라우터에서 Phoenix 라우터 기능을 사용할 수 있도록 합니다.

범위는 이 가이드에 별도의 섹션이 있으므로 여기서는 `scope "/", HelloWeb do` 블록에 대해 설명하지 않겠습니다.
`pipe_through :browser` 줄은 이 가이드의 "파이프라인" 섹션에서 자세히 다루겠습니다.
지금은 파이프라인을 통해 여러 경로 세트에 플러그 세트를 적용할 수 있다는 점만 알아두면 됩니다.

하지만 스코프 블록 안에는 첫 번째 실제 경로가 있습니다:

```perl Elixir
get "/", PageController, :home
```

`get`은 HTTP 동사 GET에 해당하는 Phoenix 매크로입니다.
다른 HTTP 동사에도 비슷한 매크로가 존재하는데, 여기에는 POST, PUT, PATCH, DELETE, OPTIONS, CONNECT, TRACE, HEAD 등이 있습니다.

## Examining routes

Phoenix는 애플리케이션에서 경로를 조사하기 위한 훌륭한 도구인 `mix phx.routes`를 제공합니다.

어떻게 작동하는지 살펴봅시다.
새로 생성된 Phoenix 애플리케이션의 루트로 이동하여 `mix phx.routes`를 실행합니다.
현재 가지고 있는 모든 경로로 생성된 다음과 같은 내용이 표시될 것입니다:

```shell
$ mix phx.routes
GET  /  HelloWeb.PageController :home
...
```

위의 경로는 애플리케이션의 루트에 대한 모든 HTTP GET 요청이 `HelloWeb.PageController`의 `home` 액션에 의해 처리될 것임을 알려줍니다.

## Resources

라우터는 [`get`](`Phoenix.Router.get/3`), [`post`](`Phoenix.Router.post/3`), [`put`](`Phoenix.Router.put/3`) 같은 HTTP 동사에 대한 매크로 외에 다른 매크로를 지원합니다.
이 중 가장 중요한 것은 [`resources`](`Phoenix.Router.resources/4`)입니다.
이렇게 `lib/hello_web/router.ex` 파일에 리소스를 추가해 봅시다:

```perl Elixir
scope "/", HelloWeb do
  pipe_through :browser

  get "/", PageController, :home
  resources "/users", UserController
  ...
end
```

지금은 실제로 `HelloWeb.UserController`가 없는 것은 중요하지 않습니다.

프로젝트 루트에서 `mix phx.routes`를 다시 한 번 실행합니다.
다음과 같은 내용이 표시될 것입니다:

```shell
...
GET     /users           HelloWeb.UserController :index
GET     /users/:id/edit  HelloWeb.UserController :edit
GET     /users/new       HelloWeb.UserController :new
GET     /users/:id       HelloWeb.UserController :show
POST    /users           HelloWeb.UserController :create
PATCH   /users/:id       HelloWeb.UserController :update
PUT     /users/:id       HelloWeb.UserController :update
DELETE  /users/:id       HelloWeb.UserController :delete
...
```

이것은 HTTP 동사, 경로 및 컨트롤러 동작의 표준 매트릭스입니다.
한동안 이 행렬을 RESTful 라우트라고 불렀지만, 요즘은 대부분 잘못된 명칭으로 간주합니다.
개별적으로 살펴봅시다.

- `users`에 대한 GET 요청은 모든 사용자를 표시하기 위해 `index` 액션을 호출합니다.
- `users/:id/edit`에 대한 GET 요청은 데이터 저장소에서 개별 사용자를 검색하고 편집할 수 있는 양식으로 정보를 표시하기 위해 ID와 함께 `edit` 액션을 호출합니다.
- `users/new`에 대한 GET 요청은 `new` 액션을 호출하여 새 사용자 생성을 위한 양식을 표시합니다.
- `users/:id`에 대한 GET 요청은 아이디와 함께 `show` 액션을 호출하여 해당 아이디로 식별된 개별 사용자를 표시합니다.
- `users`에 대한 POST 요청은 새 사용자를 데이터 저장소에 저장하기 위해 `create` 작업을 호출합니다.
- `users/:id`에 대한 PATCH 요청은 업데이트된 사용자를 데이터 저장소에 저장하기 위해 ID와 함께 `update` 작업을 호출합니다.
- `users/:id`로 PUT 요청을 하면 업데이트된 사용자를 데이터 저장소에 저장하기 위해 ID와 함께 `update` 액션을 호출합니다.
- `users/:id`에 대한 DELETE 요청은 ID와 함께 `delete` 액션을 호출하여 데이터 저장소에서 개별 사용자를 제거합니다.

이러한 경로가 모두 필요하지 않은 경우 `:only` 및 `:except` 옵션을 사용하여 특정 동작을 필터링할 수 있습니다.

읽기 전용 게시물 리소스가 있다고 가정해 보겠습니다.
다음과 같이 정의할 수 있습니다:

```perl Elixir
resources "/posts", PostController, only: [:index, :show]
```

`mix phx.routes`를 실행하면 이제 인덱스에 대한 경로와 표시 작업만 정의되었음을 보여줍니다.

```shell
GET     /posts      HelloWeb.PostController :index
GET     /posts/:id  HelloWeb.PostController :show
```

마찬가지로 댓글 리소스가 있는데 삭제 경로를 제공하지 않으려는 경우 다음과 같이 경로를 정의할 수 있습니다.

```perl Elixir
resources "/comments", CommentController, except: [:delete]
```

이제 `mix phx.routes`를 실행하면 삭제 작업에 대한 DELETE 요청을 제외한 모든 경로가 표시됩니다.

```shell
GET    /comments           HelloWeb.CommentController :index
GET    /comments/:id/edit  HelloWeb.CommentController :edit
GET    /comments/new       HelloWeb.CommentController :new
GET    /comments/:id       HelloWeb.CommentController :show
POST   /comments           HelloWeb.CommentController :create
PATCH  /comments/:id       HelloWeb.CommentController :update
PUT    /comments/:id       HelloWeb.CommentController :update
```

`Phoenix. Router.resources/4` 매크로는 리소스 경로를 사용자 정의하기 위한 추가 옵션을 설명합니다.

## Verified Routes

Phoenix에는 `~p` 시질을 사용하여 라우터에 대한 라우터 경로를 컴파일 타임에 검사하는 `Phoenix.VerifiedRoutes` 모듈이 포함되어 있습니다.
예를 들어 컨트롤러, 테스트, 템플릿에 경로를 작성하면 컴파일러가 해당 경로가 라우터에 정의된 경로와 실제로 일치하는지 확인합니다.

실제로 확인해 봅시다.
프로젝트 루트에서 `iex -S mix`를 실행합니다.
두 개의 `~p` 경로 경로를 빌드하는 일회용 예제 모듈을 정의하겠습니다.

```perl Elixir
iex> defmodule RouteExample do
...>   use HelloWeb, :verified_routes
...>
...>   def example do
...>     ~p"/comments"
...>     ~p"/unknown/123"
...>   end
...> end
warning: no route path for HelloWeb.Router matches "/unknown/123"
  iex:5: RouteExample.example/0

{:module, RouteExample, ...}
iex>
```

기존 경로인 `~p"/comments"`에 대한 첫 번째 호출은 아무런 경고를 표시하지 않지만 잘못된 경로 경로인 `~p"/unknown/123"`은 예상대로 컴파일러 경고를 표시하는 것을 볼 수 있습니다.
이는 애플리케이션에서 하드코딩된 경로를 작성할 수 있고 컴파일러가 잘못된 경로를 작성하거나 라우팅 구조를 변경할 때마다 이를 알려주기 때문에 중요합니다.

Phoenix 프로젝트는 테스트를 포함하여 웹 레이어 전체에서 검증된 경로를 사용할 수 있도록 즉시 설정됩니다.
예를 들어 템플릿에서 `~p` 링크를 렌더링할 수 있습니다:

```perl HEEx
<.link href={~p"/"}>Welcome Page!</.link>
<.link href={~p"/comments"}>View Comments</.link>
```

또는 컨트롤러에서 리디렉션을 발행할 수 있습니다:

```perl Elixir
redirect(conn, to: ~p"/comments/#{comment}")
```

경로 경로에 `~p`를 사용하면 애플리케이션 경로와 URL이 라우터 정의에 따라 최신 상태로 유지됩니다.
컴파일러가 버그를 잡아내고 애플리케이션의 다른 곳에서 참조되는 경로를 변경할 때 알려줍니다.

### More on verified routes

쿼리 문자열이 포함된 경로는 어떻게 하나요? 쿼리 문자열 키 값을 직접 추가하거나 키-값 쌍의 사전을 제공할 수 있습니다:

```perl Elixir
~p"/users/17?admin=true&active=false"
"/users/17?admin=true&active=false"

~p"/users/17?#{[admin: true]}"
"/users/17?admin=true"
```

경로 대신 전체 URL이 필요하다면 어떻게 해야 할까요? 경로를 `~p`를 사용할 수 있는 모든 곳에서 가져오는 `Phoenix.VerifiedRoutes.url/1` 호출로 감싸기만 하면 됩니다:

```perl Elixir
url(~p"/users")
"http://localhost:4000/users"
```

`url` 호출은 각 환경에 대해 설정된 구성 매개변수에서 전체 URL을 구성하는 데 필요한 호스트, 포트, 프록시 포트 및 SSL 정보를 가져옵니다.
구성에 대한 자세한 내용은 자체 가이드에서 설명합니다.
지금은 프로젝트의 `config/dev.exs` 파일을 살펴보고 해당 값을 확인할 수 있습니다.

## Nested resources

Phoenix 라우터에서 리소스를 중첩할 수도 있습니다.
'사용자'와 다대일 관계에 있는 '게시물' 리소스가 있다고 가정해 봅시다.
즉, 한 사용자가 여러 개의 게시물을 만들 수 있고 개별 게시물은 한 사용자에게만 속합니다.
다음과 같이 `lib/hello_web/router.ex`에 중첩 경로를 추가하여 이를 표현할 수 있습니다:

```perl Elixir
resources "/users", UserController do
  resources "/posts", PostController
end
```

이제 `mix phx.routes`를 실행하면 위의 `사용자`에 대한 경로 외에 다음과 같은 경로 집합이 표시됩니다:

```perl Elixir
...
GET     /users/:user_id/posts           HelloWeb.PostController :index
GET     /users/:user_id/posts/:id/edit  HelloWeb.PostController :edit
GET     /users/:user_id/posts/new       HelloWeb.PostController :new
GET     /users/:user_id/posts/:id       HelloWeb.PostController :show
POST    /users/:user_id/posts           HelloWeb.PostController :create
PATCH   /users/:user_id/posts/:id       HelloWeb.PostController :update
PUT     /users/:user_id/posts/:id       HelloWeb.PostController :update
DELETE  /users/:user_id/posts/:id       HelloWeb.PostController :delete
...
```

이러한 각 경로가 사용자 ID로 게시물의 범위를 지정하는 것을 알 수 있습니다.
첫 번째 경로의 경우, `PostController`의 `index` 액션을 호출하되 `user_id`를 전달합니다.
이는 해당 개별 사용자에 대한 모든 글만 표시한다는 것을 의미합니다.
이러한 모든 경로에 동일한 범위가 적용됩니다.

중첩된 경로에 대한 경로를 작성할 때는 경로 정의에서 해당 경로가 속한 ID를 보간해야 합니다.
다음 `show` 경로의 경우 `42`는 `user_id`이고 `17`은 `post_id`입니다.

```perl Elixir
user_id = 42
post_id = 17
~p"/users/#{user_id}/posts/#{post_id}"
"/users/42/posts/17"
```

확인된 경로는 `Phoenix.Param` 프로토콜도 지원하지만, 아직은 엘릭서 프로토콜에 대해 걱정할 필요는 없습니다.
`%User{}` 및 `%Post{}`와 같은 구조체로 애플리케이션을 빌드하기 시작하면 이러한 데이터 구조를 `~p` 경로에 직접 보간할 수 있으며, Phoenix는 경로에 사용할 올바른 필드를 뽑아낼 것입니다.

```perl Elixir
~p"/users/#{user}/posts/#{post}"
"/users/42/posts/17"
```

`user.id` 또는 `post.id`를 보간할 필요가 없는 것이 보이시죠? 이 기능은 나중에 URL을 좀 더 멋지게 만들고 대신 슬러그를 사용하기로 결정할 때 특히 유용합니다.
`~p`를 하나도 변경할 필요가 없습니다!

## Scoped routes

범위는 공통 경로 접두사와 범위가 지정된 플러그 집합으로 경로를 그룹화하는 방법입니다.
관리자 기능, API, 특히 버전이 지정된 API에 이 방법을 사용할 수 있습니다.
사이트에 사용자가 작성한 리뷰가 있고 이러한 리뷰는 먼저 관리자의 승인을 받아야 한다고 가정해 보겠습니다.
이러한 리소스의 의미는 상당히 다르며 동일한 컨트롤러를 공유하지 않을 수도 있습니다.
범위를 사용하면 이러한 경로를 분리할 수 있습니다.

사용자 대상 리뷰 경로는 표준 리소스처럼 보일 것입니다.

```shell
/reviews
/reviews/1234
/reviews/1234/edit
...
```

관리 검토 경로 앞에 `/admin`을 붙일 수 있습니다.

```shell
/admin/reviews
/admin/reviews/1234
/admin/reviews/1234/edit
...
```

이와 같이 경로 옵션을 `/admin`으로 설정하는 범위 지정 경로를 사용하여 이를 수행합니다.
이 범위를 다른 범위 안에 중첩할 수도 있지만, 대신 `lib/hello_web/router.ex`에 다음을 추가하여 루트에서 단독으로 설정해 보겠습니다:

```shell Elixir
scope "/admin", HelloWeb.Admin do
  pipe_through :browser

  resources "/reviews", ReviewController
end
```

모든 경로의 접두사가 `/admin`이고 모든 컨트롤러가 `HelloWeb.Admin` 네임스페이스 아래에 있는 새 스코프를 정의합니다.

이전 경로 집합에 추가하여 `mix phx.routes`를 다시 실행하면 다음과 같은 결과가 나타납니다:

```shell
...
GET     /admin/reviews           HelloWeb.Admin.ReviewController :index
GET     /admin/reviews/:id/edit  HelloWeb.Admin.ReviewController :edit
GET     /admin/reviews/new       HelloWeb.Admin.ReviewController :new
GET     /admin/reviews/:id       HelloWeb.Admin.ReviewController :show
POST    /admin/reviews           HelloWeb.Admin.ReviewController :create
PATCH   /admin/reviews/:id       HelloWeb.Admin.ReviewController :update
PUT     /admin/reviews/:id       HelloWeb.Admin.ReviewController :update
DELETE  /admin/reviews/:id       HelloWeb.Admin.ReviewController :delete
...
```

좋아 보이지만 여기에 문제가 있습니다.
사용자 대상 리뷰 경로인 `/reviews`와 관리자 대상 경로인 `/admin/reviews`가 모두 필요하다는 점을 기억하세요.
이제 라우터에 사용자 대면 리뷰를 이렇게 루트 범위 아래에 포함시키면 됩니다:

```perl Elixir
scope "/", HelloWeb do
  pipe_through :browser

  ...
  resources "/reviews", ReviewController
end

scope "/admin", HelloWeb.Admin do
  pipe_through :browser

  resources "/reviews", ReviewController
end
```

그리고 `mix phx.routes`를 실행하면 범위가 지정된 각 경로에 대한 출력을 얻습니다:

```shell
...
GET     /reviews                 HelloWeb.ReviewController :index
GET     /reviews/:id/edit        HelloWeb.ReviewController :edit
GET     /reviews/new             HelloWeb.ReviewController :new
GET     /reviews/:id             HelloWeb.ReviewController :show
POST    /reviews                 HelloWeb.ReviewController :create
PATCH   /reviews/:id             HelloWeb.ReviewController :update
PUT     /reviews/:id             HelloWeb.ReviewController :update
DELETE  /reviews/:id             HelloWeb.ReviewController :delete
...
GET     /admin/reviews           HelloWeb.Admin.ReviewController :index
GET     /admin/reviews/:id/edit  HelloWeb.Admin.ReviewController :edit
GET     /admin/reviews/new       HelloWeb.Admin.ReviewController :new
GET     /admin/reviews/:id       HelloWeb.Admin.ReviewController :show
POST    /admin/reviews           HelloWeb.Admin.ReviewController :create
PATCH   /admin/reviews/:id       HelloWeb.Admin.ReviewController :update
PUT     /admin/reviews/:id       HelloWeb.Admin.ReviewController :update
DELETE  /admin/reviews/:id       HelloWeb.Admin.ReviewController :delete
```

관리자가 모두 처리하는 리소스가 여러 개 있다면 어떨까요? 이렇게 모든 리소스를 동일한 범위 안에 넣을 수 있습니다:

```perl Elixir
scope "/admin", HelloWeb.Admin do
  pipe_through :browser

  resources "/images",  ImageController
  resources "/reviews", ReviewController
  resources "/users",   UserController
end
```

다음은 `mix phx.routes`가 알려주는 내용입니다:

```shell
...
GET     /admin/images            HelloWeb.Admin.ImageController :index
GET     /admin/images/:id/edit   HelloWeb.Admin.ImageController :edit
GET     /admin/images/new        HelloWeb.Admin.ImageController :new
GET     /admin/images/:id        HelloWeb.Admin.ImageController :show
POST    /admin/images            HelloWeb.Admin.ImageController :create
PATCH   /admin/images/:id        HelloWeb.Admin.ImageController :update
PUT     /admin/images/:id        HelloWeb.Admin.ImageController :update
DELETE  /admin/images/:id        HelloWeb.Admin.ImageController :delete
GET     /admin/reviews           HelloWeb.Admin.ReviewController :index
GET     /admin/reviews/:id/edit  HelloWeb.Admin.ReviewController :edit
GET     /admin/reviews/new       HelloWeb.Admin.ReviewController :new
GET     /admin/reviews/:id       HelloWeb.Admin.ReviewController :show
POST    /admin/reviews           HelloWeb.Admin.ReviewController :create
PATCH   /admin/reviews/:id       HelloWeb.Admin.ReviewController :update
PUT     /admin/reviews/:id       HelloWeb.Admin.ReviewController :update
DELETE  /admin/reviews/:id       HelloWeb.Admin.ReviewController :delete
GET     /admin/users             HelloWeb.Admin.UserController :index
GET     /admin/users/:id/edit    HelloWeb.Admin.UserController :edit
GET     /admin/users/new         HelloWeb.Admin.UserController :new
GET     /admin/users/:id         HelloWeb.Admin.UserController :show
POST    /admin/users             HelloWeb.Admin.UserController :create
PATCH   /admin/users/:id         HelloWeb.Admin.UserController :update
PUT     /admin/users/:id         HelloWeb.Admin.UserController :update
DELETE  /admin/users/:id         HelloWeb.Admin.UserController :delete
```

우리가 원하던 바로 그 결과입니다.
모든 경로와 컨트롤러의 네임스페이스가 올바르게 지정되어 있다는 점에 유의하세요.

범위를 임의로 중첩할 수도 있지만, 중첩하면 코드가 혼란스럽고 명확하지 않을 수 있으므로 신중하게 수행해야 합니다.
이미지, 리뷰 및 사용자에 대해 정의된 리소스가 있는 버전이 있는 API가 있다고 가정해 보겠습니다.
그렇다면 기술적으로 다음과 같이 버전이 지정된 API에 대한 경로를 설정할 수 있습니다:

```perl Elixir
scope "/api", HelloWeb.Api, as: :api do
  pipe_through :api

  scope "/v1", V1, as: :v1 do
    resources "/images",  ImageController
    resources "/reviews", ReviewController
    resources "/users",   UserController
  end
end
```

`mix phx.routes`를 실행하여 이러한 정의가 어떻게 표시되는지 확인할 수 있습니다.

흥미롭게도 경로가 중복되지 않도록 주의하는 한 동일한 경로로 여러 범위를 사용할 수 있습니다.
다음 라우터는 동일한 경로에 대해 정의된 두 개의 범위로 완벽하게 작동합니다:

```perl Elixir
defmodule HelloWeb.Router do
  use Phoenix.Router
  ...
  scope "/", HelloWeb do
    pipe_through :browser

    resources "/users", UserController
  end

  scope "/", AnotherAppWeb do
    pipe_through :browser

    resources "/posts", PostController
  end
  ...
end
```

경로가 중복되는 경우, 즉 경로가 같은 두 개의 경로가 있는 경우 익숙한 경고가 표시됩니다:

```shell
warning: this clause cannot match because a previous clause at line 16 always matches
```

## Pipelines

라우터에서 본 첫 번째 줄 중 하나에 대해 이야기하지 않고 이 가이드에서 꽤 먼 길을 왔습니다: pipe_through :browser`.
이제 바로잡을 시간입니다.

파이프라인은 특정 범위에 연결할 수 있는 일련의 플러그입니다.
플러그에 익숙하지 않다면 [플러그에 대한 심층 가이드](plug.html)를 참조하세요.

경로는 스코프 내부에 정의되며 스코프는 여러 파이프라인을 통과할 수 있습니다.
경로가 일치하면 Phoenix는 해당 경로와 연결된 모든 파이프라인에 정의된 모든 플러그를 호출합니다.
예를 들어 `/`에 액세스하면 `:browser` 파이프라인을 통과하여 결과적으로 모든 플러그가 호출됩니다.

Phoenix는 기본적으로 `:browser`와 `:api`라는 두 개의 파이프라인을 정의하며, 이 파이프라인은 여러 가지 일반적인 작업에 사용할 수 있습니다.
이를 사용자 정의하고 필요에 따라 새로운 파이프라인을 생성할 수도 있습니다.

### The `:browser` and `:api` pipelines

이름에서 알 수 있듯이 `:browser` 파이프라인은 브라우저에 대한 요청을 렌더링하는 경로를 준비하고 `:api` 파이프라인은 API에 대한 데이터를 생성하는 경로를 준비합니다.

`:browser` 파이프라인에는 6개의 플러그가 있습니다: plug :accepts, ["html"]`는 허용되는 요청 형식을 정의합니다.
`:fetch_session`은 세션 데이터를 가져와서 연결에서 사용할 수 있도록 합니다.
`:fetch_live_flash`는 라이브뷰에서 모든 플래시 메시지를 가져와 컨트롤러 플래시 메시지와 병합합니다.
그런 다음 `:put_root_layout` 플러그는 렌더링 목적으로 루트 레이아웃을 저장합니다.
이후 `:protect_from_forgery`와 `:put_secure_browser_headers`는 사이트 간 위조로부터 양식 게시물을 보호합니다.

현재 `:api` 파이프라인은 `plug :accepts, ["json"]`만 정의합니다.

라우터는 범위 내에 정의된 경로에서 파이프라인을 호출합니다.
범위 외부의 경로에는 파이프라인이 없습니다.
중첩된 범위의 사용은 권장되지 않지만(위의 버전이 지정된 API 예제 참조), 중첩된 범위 내에서 `pipe_through`를 호출하면 라우터는 부모 범위의 모든 `pipe_through`를 호출한 다음 중첩된 범위에서 호출합니다.

많은 단어가 한데 묶여 있습니다.
몇 가지 예를 통해 그 의미를 풀어보겠습니다.

다음은 새로 생성된 Phoenix 애플리케이션의 라우터로, 이번에는 `/api` 스코프가 주석 처리되지 않고 경로가 추가된 상태입니다.

```perl Elixir
defmodule HelloWeb.Router do
  use HelloWeb, :router

  pipeline :browser do
    plug :accepts, ["html"]
    plug :fetch_session
    plug :fetch_live_flash
    plug :put_root_layout, html: {HelloWeb.Layouts, :root}
    plug :protect_from_forgery
    plug :put_secure_browser_headers
  end

  pipeline :api do
    plug :accepts, ["json"]
  end

  scope "/", HelloWeb do
    pipe_through :browser

    get "/", PageController, :home
  end

  # Other scopes may use custom stacks.
  scope "/api", HelloWeb do
    pipe_through :api

    resources "/reviews", ReviewController
  end
  # ...
end
```

서버가 요청을 수락하면 요청은 항상 엔드포인트의 플러그를 먼저 통과한 후 경로와 HTTP 동사를 일치시키려고 시도합니다.

요청이 첫 번째 경로인 `/`에 대한 GET과 일치한다고 가정해 보겠습니다.
라우터는 먼저 세션 데이터를 가져오고, 플래시를 가져오고, 위변조 방지를 실행하는 `:browser` 파이프라인을 통해 해당 요청을 파이프한 후 `PageController`의 `home` 액션으로 요청을 전송합니다.

반대로, 요청이 [`resources/2`](`Phoenix.Router.resources/2`) 매크로에 정의된 경로와 일치한다고 가정해 보겠습니다.
이 경우 라우터는 현재 콘텐츠 협상만 수행하는 `:api` 파이프라인을 통해 요청을 파이프한 후 `HelloWeb.ReviewController`의 올바른 작업으로 추가 발송합니다.

일치하는 경로가 없으면 파이프라인이 호출되지 않고 404 오류가 발생합니다.

### Creating new pipelines

Phoenix를 사용하면 라우터 내 어디에서나 사용자 정의 파이프라인을 생성할 수 있습니다.
이를 위해 새 파이프라인의 이름을 나타내는 원자, 원하는 모든 플러그가 포함된 블록을 인수로 사용하여 [`pipeline/2`](`Phoenix.Router.pipeline/2`) 매크로를 호출합니다.

```perl Elixir
defmodule HelloWeb.Router do
  use HelloWeb, :router

  pipeline :browser do
    plug :accepts, ["html"]
    plug :fetch_session
    plug :fetch_live_flash
    plug :put_root_layout, html: {HelloWeb.Layouts, :root}
    plug :protect_from_forgery
    plug :put_secure_browser_headers
  end

  pipeline :auth do
    plug HelloWeb.Authentication
  end

  scope "/reviews", HelloWeb do
    pipe_through [:browser, :auth]

    resources "/", ReviewController
  end
end
```

위에서는 인증을 수행하는 `HelloWeb.Authentication`이라는 플러그가 있으며 이제 `:auth` 파이프라인의 일부라고 가정합니다.

파이프라인 자체는 플러그이므로 다른 파이프라인 안에 파이프라인을 플러그할 수 있습니다.
예를 들어, 위의 `auth` 파이프라인을 재작성하여 `browser`를 자동으로 호출하여 다운스트림 파이프라인 호출을 간소화할 수 있습니다:

```perl Elixir
  pipeline :auth do
    plug :browser
    plug :ensure_authenticated_user
    plug :ensure_user_owns_review
  end

  scope "/reviews", HelloWeb do
    pipe_through :auth

    resources "/", ReviewController
  end
```

## How to organize my routes?

Phoenix에서는 특정 기능을 제공하는 여러 파이프라인을 정의하는 경향이 있습니다.
예를 들어, `:browser` 및 `:api` 파이프라인은 각각 특정 클라이언트, 브라우저 및 http 클라이언트에서 액세스하기 위한 것입니다.

더 중요한 것은 인증 및 권한 부여와 관련된 파이프라인을 정의하는 것도 매우 일반적이라는 점입니다.
예를 들어 모든 사용자가 인증을 받아야 하는 파이프라인이 있을 수 있습니다.
또 다른 파이프라인에서는 관리자 사용자만 특정 경로에 액세스할 수 있도록 할 수도 있습니다.

파이프라인을 정의한 후에는 원하는 범위에서 파이프라인을 재사용하여 해당 파이프라인을 중심으로 경로를 그룹화합니다.
예를 들어 리뷰의 예로 돌아가 보겠습니다.
누구나 리뷰를 읽을 수 있지만 인증된 사용자만 리뷰를 작성할 수 있다고 가정해 보겠습니다.
경로는 다음과 같이 보일 수 있습니다:

```perl Elixir
pipeline :browser do
  ...
end

pipeline :auth do
  plug HelloWeb.Authentication
end

scope "/" do
  pipe_through [:browser]

  get "/reviews", PostController, :index
  get "/reviews/:id", PostController, :show
end

scope "/" do
  pipe_through [:browser, :auth]

  get "/reviews/new", PostController, :new
  post "/reviews", PostController, :create
end
```

위 그림에서 경로가 여러 범위에 걸쳐 분할되어 있는 것을 확인할 수 있습니다.
이렇게 분리하면 처음에는 혼란스러울 수 있지만, 한 가지 큰 장점이 있습니다. 예를 들어 경로를 검사하여 인증이 필요한 경로와 그렇지 않은 경로를 모두 쉽게 확인할 수 있다는 점입니다.
이는 경로를 감사하고 경로의 범위가 적절한지 확인하는 데 도움이 됩니다.

원하는 만큼의 범위를 만들 수 있습니다.
파이프라인은 여러 범위에서 재사용할 수 있으므로 공통 기능을 캡슐화하는 데 도움이 되며, 정의한 각 범위에서 필요에 따라 구성할 수 있습니다.

## Forward

특정 경로로 시작하는 모든 요청을 특정 플러그에 보내는 데 `Phoenix.Router.forward/4` 매크로를 사용할 수 있습니다.
백그라운드에서 작업을 실행하는 시스템 일부(별도의 애플리케이션이나 라이브러리일 수도 있음)에 작업 상태를 확인하기 위한 자체 웹 인터페이스가 있다고 가정해 봅시다.
이 관리자 인터페이스를 사용하여 전달할 수 있습니다:

```perl Elixir
defmodule HelloWeb.Router do
  use HelloWeb, :router

  ...

  scope "/", HelloWeb do
    ...
  end

  forward "/jobs", BackgroundJob.Plug
end
```

즉, `/jobs`로 시작하는 모든 경로가 `HelloWeb.BackgroundJob.Plug` 모듈로 전송됩니다.
플러그 내에서 특정 작업의 상태를 표시하는 `/pending` 및 `/active`와 같은 하위 경로를 일치시킬 수 있습니다.

[`forward/4`](`Phoenix.Router.forward/4`) 매크로를 파이프라인과 혼합할 수도 있습니다.
작업 페이지를 보기 위해 사용자가 인증되고 관리자인지 확인하려면 라우터에서 다음을 사용할 수 있습니다.

```perl Elixir
defmodule HelloWeb.Router do
  use HelloWeb, :router

  ...

  scope "/" do
    pipe_through [:authenticate_user, :ensure_admin]
    forward "/jobs", BackgroundJob.Plug
  end
end
```

즉, `authenticate_user` 및 `ensure_admin` 파이프라인의 플러그가 `BackgroundJob.Plug`보다 먼저 호출되어 적절한 응답을 보내고 그에 따라 요청을 중단할 수 있습니다.

모듈 플러그의 `init/1` 콜백에서 수신되는 `옵션`은 세 번째 인수로 전달할 수 있습니다.
예를 들어 백그라운드 작업을 통해 페이지에 표시할 애플리케이션의 이름을 설정할 수 있습니다.
이를 함께 전달할 수 있습니다:

```perl Elixir
forward "/jobs", BackgroundJob.Plug, name: "Hello Phoenix"
```

전달할 수 있는 네 번째 `router_opts` 인수가 있습니다.
이러한 옵션은 `Phoenix.Router.scope/2` 문서에 설명되어 있습니다.

`BackgroundJob.Plug` can be implemented as any Module Plug discussed in the [Plug guide](plug.html).
다른 Phoenix 엔드포인트로 포워딩하는 것은 권장하지 않습니다.
앱과 전달된 엔드포인트에 의해 정의된 플러그가 두 번 호출되어 오류가 발생할 수 있기 때문입니다.

## Summary

라우팅은 매우 중요한 주제이며 여기서는 많은 부분을 다루었습니다.
이 가이드에서 알아두어야 할 중요한 사항은 다음과 같습니다:

- HTTP 동사 이름으로 시작하는 경로는 일치 함수의 단일 절로 확장됩니다.
- `resources`로 선언된 경로는 일치 함수의 8개 절로 확장됩니다.
- 리소스는 `only:` 또는 `except:` 옵션을 사용하여 일치 함수 절의 수를 제한할 수 있습니다.
- 이러한 경로는 중첩될 수 있습니다.
- 이러한 경로는 지정된 경로로 범위가 지정될 수 있습니다.
- 컴파일 타임 경로 검사에 `~p`와 함께 확인된 경로 사용
