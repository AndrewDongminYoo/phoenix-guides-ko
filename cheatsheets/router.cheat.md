# Routing cheatsheet

> 올바른 라우터 모듈과 범위에서 선언해야 합니다.

일반적인 라우팅 기능의 구문에 대한 빠른 참조입니다.
전체 개요는 [라우팅 가이드](routing.md)를 참조하세요.

## Routing declaration
{: .col-2}

### Single route

```elixir
get "/users", UserController, :index
patch "/users/:id", UserController, :update
```

```elixir
# generated routes
~p"/users"
~p"/users/9" # user_id is 9
```

또한 `put`, `patch`, `options`, `delete` 및 `head`도 사용할 수 있습니다.

### Resources

#### Simple

```elixir
resources "/users", UserController
```

`index`, `:edit`, `:new`, `:show`, `:create`, `:update` 및 `:delete`를 생성합니다.

#### Options

```elixir
resources "/users", UserController, only: [:show]
resources "/users", UserController, except: [:create, :delete]
resources "/users", UserController, as: :person # ~p"/person"
```

#### Nested

```elixir
resources "/users", UserController do
  resources "/posts", PostController
end
```

```elixir
# generated routes
~p"/users/3/posts" # user_id is 3
~p"/users/3/posts/17" # user_id is 3 and post_id = 17
```

자세한 내용은 [리소스 문서](routing-1.html#resources)에서 확인하세요.

### Scopes

#### Simple

```elixir
scope "/admin", HelloWeb.Admin do
  pipe_through :browser

  resources "/users",   UserController
end
```

```elixir
# generated path helpers
~p"/admin/users"
```

#### Nested
```elixir
scope "/api", HelloWeb.Api, as: :api do
  pipe_through :api

  scope "/v1", V1, as: :v1 do
    resources "/users", UserController
  end
end
```

```elixir
# generated path helpers
~p"/api/v1/users"
```

자세한 내용은 [범위 지정 경로](routing.md#scoped-routes) 문서를 참조하세요.
