# API Authentication

> **요구사항**: 이 가이드에서는 [`mix phx.gen.auth`](mix_phx_gen_auth.html) 가이드를 완료하셨을 것으로 가정합니다.

이 가이드는 `mix phx.gen.auth` 위에 API 인증을 추가하는 방법을 보여줍니다.
인증 생성기에는 이미 토큰 테이블이 포함되어 있으므로 모범 보안 사례에 따라 API 토큰을 저장하는 데도 사용합니다.

이 가이드는 컨텍스트 보강과 플러그 구현의 두 부분으로 나누어 설명하겠습니다.
다음 `mix phx.gen.auth` 명령이 실행되었다고 가정하겠습니다:

```shell
mix phx.gen.auth Accounts User users
```

다른 것을 실행한 경우 이름을 변경하는 것은 간단합니다.

## 컨텍스트에 API 함수 추가하기

인증 시스템에는 두 가지 함수가 필요합니다.
하나는 API 토큰을 생성하는 함수이고 다른 하나는 이를 확인하는 함수입니다.
`lib/my_app/accounts.ex`를 열고 이 두 가지 새 함수를 추가합니다:

```perl Elixir
  ## API

  @doc """
  사용자에 대한 새 API 토큰을 생성합니다.

  반환된 토큰은 안전한 곳에 저장해야 합니다.
  이 토큰은 데이터베이스에서 복구할 수 없습니다.
  """
  def create_user_api_token(user) do
    {encoded_token, user_token} = UserToken.build_email_token(user, "api-token")
    Repo.insert!(user_token)
    encoded_token
  end

  @doc """
  API 토큰으로 사용자를 가져옵니다.
  """
  def fetch_user_by_api_token(token) do
    with {:ok, query} <- UserToken.verify_email_token_query(token, "api-token"),
         %User{} = user <- Repo.one(query) do
      {:ok, user}
    else
      _ -> :error
    end
  end
```

새로운 함수는 기존의 `UserToken` 함수를 사용하여 "api-token"이라는 새로운 유형의 토큰을 저장합니다.
이 토큰은 이메일 토큰이므로 사용자가 이메일을 변경하면 토큰이 만료됩니다.

또한 두 번째 함수를 `get_user_by_api_token` 대신 `fetch_user_by_api_token`으로 호출한 것도 주목하세요.
사용자를 찾았는지 여부에 따라 API에서 다른 상태 코드를 렌더링하고 싶기 때문에 `{:ok, user}` 또는 `:error`를 반환합니다.
일반적으로 튜플 대신 `nil`을 반환하는 `get_*` 대신 이러한 함수를 `fetch_*`으로 호출하는 것이 Elixir의 관례입니다.

새 함수가 제대로 작동하는지 확인하기 위해 테스트를 작성해 봅시다.
`test/my_app/accounts_test.exs`를 열고 이 새로운 설명 블록을 추가합니다:

```perl Elixir
  describe "create_user_api_token/1 and fetch_user_by_api_token/1" do
    test "creates and fetches by token" do
      user = user_fixture()
      token = Accounts.create_user_api_token(user)
      assert Accounts.fetch_user_by_api_token(token) == {:ok, user}
      assert Accounts.fetch_user_by_api_token("invalid") == :error
    end
  end
```

테스트를 실행하면 실제로 실패합니다.
이와 비슷합니다:

```perl Elixir
1) test create_user_api_token/1 and fetch_user_by_api_token/1 creates and verify token (Demo.AccountsTest)
   test/demo/accounts_test.exs:21
   ** (FunctionClauseError) no function clause matching in Demo.Accounts.UserToken.days_for_context/1

   The following arguments were given to Demo.Accounts.UserToken.days_for_context/1:

       # 1
       "api-token"

   Attempted function clauses (showing 2 out of 2):

       defp days_for_context("confirm")
       defp days_for_context("reset_password")

   code: assert Accounts.verify_api_token(token) == {:ok, user}
   stacktrace:
     (demo 0.1.0) lib/demo/accounts/user_token.ex:129: Demo.Accounts.UserToken.days_for_context/1
     (demo 0.1.0) lib/demo/accounts/user_token.ex:114: Demo.Accounts.UserToken.verify_email_token_query/2
     (demo 0.1.0) lib/demo/accounts.ex:301: Demo.Accounts.verify_api_token/1
     test/demo/accounts_test.exs:24: (test)
```

원하는 경우 오류를 살펴보고 직접 수정해 보세요.
설명은 다음에 이어집니다.

`UserToken` 모듈은 각 토큰의 유효성을 선언할 것을 기대하지만 "api-token"에 대한 유효성을 정의하지 않았습니다.
길이는 애플리케이션과 보안 측면에서 얼마나 민감한지에 따라 달라집니다.
이 예제에서는 토큰의 유효 기간이 365일이라고 가정해 보겠습니다.

`lib/my_app/accounts/user_token.ex`를 열고 `defp days_for_context`가 정의된 위치를 찾아 다음과 같이 새 절을 추가합니다:

```perl Elixir
  defp days_for_context("api-token"), do: 365
  defp days_for_context("confirm"), do: @confirm_validity_in_days
  defp days_for_context("reset_password"), do: @reset_password_validity_in_days
```

이제 테스트가 통과되어 앞으로 나아갈 준비가 되었습니다!

## API 인증 플러그

마지막 단계는 API에 인증을 추가하는 것입니다.

`mix phx.gen.auth`를 실행하면 `conn`을 수신하고 요청/응답 라이프사이클을 사용자 정의하는 작은 함수인 여러 개의 플러그가 있는 `MyAppWeb.UserAuth` 모듈이 생성됩니다.
`lib/my_app_web/user_auth.ex`를 열고 이 새 함수를 추가합니다:

```perl Elixir
def fetch_api_user(conn, _opts) do
  with ["Bearer " <> token] <- get_req_header(conn, "authorization"),
       {:ok, user} <- Accounts.fetch_user_by_api_token(token) do
    assign(conn, :current_user, user)
  else
    _ ->
      conn
      |> send_resp(:unauthorized, "No access for you")
      |> halt()
  end
end
```

이 함수는 연결을 수신하고 '인증' 헤더가 '무기명 토큰'으로 설정되었는지 확인합니다. 여기서 '토큰'은 `Accounts.create_user_api_token/1`이 반환하는 값입니다.
토큰이 유효하지 않거나 해당 사용자가 없는 경우 요청을 중단합니다.

마지막으로 이 `플러그`를 파이프라인에 추가해야 합니다.
`lib/my_app_web/router.ex`를 열면 API용 파이프라인을 찾을 수 있습니다.
그 아래에 다음과 같이 새 플러그를 추가해 보겠습니다:

```perl Elixir
  pipeline :api do
    plug :accepts, ["json"]
    plug :fetch_api_user
  end
```

이제 API 요청을 수신하고 유효성을 검사할 준비가 되었습니다.
`test/my_app_web/user_auth_test.exs`를 열고 자신만의 테스트를 작성하세요.
다른 플러그에 대한 테스트를 템플릿으로 사용할 수 있습니다!

## 여러분의 차례

전반적인 API 인증 흐름은 애플리케이션에 따라 달라집니다.

자바스크립트 클라이언트에서 이 토큰을 사용하려면 `UserSessionController`를 약간 변경하여 `Accounts.create_user_api_token/1`을 호출하고 JSON 응답을 반환한 후 반환된 토큰을 포함시켜야 합니다.

타사 사용자에게 API를 제공하려면 해당 사용자가 토큰을 만들 수 있도록 허용하고 `Accounts.create_user_api_token/1`의 결과를 표시해야 합니다.
이 토큰을 안전한 곳에 저장하고 "authorization" 헤더를 사용하여 요청의 일부로 포함시켜야 합니다.
