# mix phx.gen.auth

`mix phx.gen.auth` 명령은 유연한 사전 빌드된 인증 시스템을 Phoenix 앱에 생성합니다.
이 제너레이터를 사용하면 코드베이스에 인증을 추가하는 작업을 빠르게 처리하고 애플리케이션이 해결하려는 실제 문제에 집중할 수 있습니다.

## Getting started

> 여러 파일을 생성하므로 이 명령을 실행하기 전에 작업을 커밋하는 것이 좋습니다.

앱의 루트(또는 엄브렐러 앱의 경우 `apps/my_app_web`)에서 다음 명령을 실행하여 시작해 보겠습니다:

```shell
$ mix phx.gen.auth Accounts User users

An authentication system can be created in two different ways:
- Using Phoenix.LiveView (default)
- Using Phoenix.Controller only

Do you want to create a LiveView based authentication system? [Y/n] Y
```

인증 생성기는 향상된 UX를 위해 피닉스 라이브뷰를 지원하므로 여기서는 '예'로 답하겠습니다.
컨트롤러 기반 인증 시스템의 경우 `n`으로 대답할 수도 있습니다.

어느 쪽이든 `Accounts.User` 스키마 모듈을 사용하여 `Accounts` 컨텍스트를 생성합니다.
마지막 인수는 데이터베이스 테이블 이름과 경로 경로를 생성하는 데 사용되는 스키마 모듈의 복수 버전입니다.
`mix phx.gen.auth` 생성기는 스키마에 추가할 추가 필드 목록을 허용하지 않고 더 많은 컨텍스트 함수를 생성한다는 점을 제외하면 `mix phx.gen.html`과 유사합니다.

이 생성기는 `mix.exs`에 추가 종속성을 설치하므로 이를 가져와 보겠습니다:

```shell
mix deps.get
```

이제 마이그레이터와 테스트가 제대로 실행될 수 있도록 `config/`에서 개발 및 테스트 환경에 대한 데이터베이스 연결 세부 정보를 확인해야 합니다.
그런 다음 다음을 실행하여 데이터베이스를 생성합니다:

```shell
mix ecto.setup
```

테스트를 실행하여 새 인증 시스템이 예상대로 작동하는지 확인해 보겠습니다.

```shell
mix test
```

마지막으로 피닉스 서버를 시작하여 사용해 보겠습니다.

```shell
mix phx.server
```

## 개발자 책임

피닉스는 이러한 모듈을 피닉스 자체에 빌드하는 대신 애플리케이션에 이 코드를 생성하므로 이제 인증 시스템을 자유롭게 수정할 수 있으므로 사용 사례에 가장 잘 맞습니다.
생성된 인증 시스템을 사용할 때 한 가지 주의할 점은 인증 시스템이 생성된 후에는 업데이트되지 않는다는 것입니다.
따라서 `mix phx.gen.auth`의 출력이 개선되면 이러한 변경 사항을 애플리케이션에 포팅해야 하는지 여부를 결정하는 것은 사용자의 책임입니다.
보안 관련 및 기타 중요한 개선 사항은 `CHANGELOG.md` 파일 및 업그레이드 노트에 명시적이고 명확하게 표시됩니다.

## 생성된 코드

다음은 생성된 인증 시스템에 대한 참고 사항입니다.

### 비밀번호 해싱

비밀번호 해싱 메커니즘의 기본값은 유닉스 시스템의 경우 `bcrypt`, 윈도우 시스템의 경우 `pbkdf2`입니다.
두 시스템 모두 [컴오닌 인터페이스](https://hexdocs.pm/comeonin/)를 사용합니다.

비밀번호 해싱 메커니즘은 `--hashing-lib` 옵션으로 재정의할 수 있습니다.
지원되는 값은 다음과 같습니다:

- `bcrypt` - [bcrypt_elixir](https://hex.pm/packages/bcrypt_elixir)
- `pbkdf2` - [pbkdf2_elixir](https://hex.pm/packages/pbkdf2_elixir)
- `argon2` - [argon2_elixir](https://hex.pm/packages/argon2_elixir)

개발자는 3가지 중 가장 강력한 `argon2`를 사용하는 것을 고려하는 것이 좋습니다.
단점은 `argon2`가 CPU와 메모리를 상당히 많이 사용하므로 애플리케이션을 실행하려면 더 강력한 인스턴스가 필요하다는 것입니다.

이러한 라이브러리 선택에 대한 자세한 내용은 [Comeonin 프로젝트](https://github.com/riverrun/comeonin)를 참조하세요.

### 접근 금지

생성된 코드는 현재 사용자를 불러오고, 인증을 요구하는 등의 몇 가지 플러그가 있는 인증 모듈과 함께 제공됩니다.
예를 들어, `mix phx.gen.auth Accounts User users`가 실행된 Demo라는 앱에는 다음과 같은 플러그가 있는 `DemoWeb.UserAuth`라는 이름의 모듈이 있습니다:

- `fetch_current_user` - 사용 가능한 경우 현재 사용자 정보를 가져옵니다.
- `require_authenticated_user` - `fetch_current_user` 뒤에 호출되어야 하며 현재 사용자가 존재하고 인증되어야 합니다.
- `redirect_if_user_is_authenticated` - 인증된 사용자에게 제공되지 않아야 하는 몇 가지 페이지에 사용됩니다.

### 확인

생성된 기능은 계정 확인 메커니즘과 함께 제공되며, 사용자는 일반적으로 이메일을 통해 계정을 확인해야 합니다.
그러나 생성된 코드는 사용자가 아직 계정을 확인하지 않은 경우 애플리케이션 사용을 금지하지 않습니다.
이 기능은 `Auth` 모듈의 `require_authenticated_user`를 사용자 정의하여 `confirmed_at` 필드(및 원하는 다른 속성)를 확인하여 추가할 수 있습니다.

### 알림

생성된 코드는 계정 확인, 비밀번호 재설정 등을 위한 SMS나 이메일을 전송하기 위해 시스템과 통합되지 않습니다.
대신, 단순히 터미널에 메시지를 기록할 뿐입니다.
생성 후 적절한 시스템과 통합하는 것은 사용자의 책임입니다.

Phoenix 프로젝트를 `mix phx.new`로 생성한 경우, 프로젝트는 기본적으로 [Swoosh](https://hexdocs.pm/swoosh/Swoosh.html) 메일러를 사용하도록 구성됩니다.
Swoosh로 개발 중 알림 이메일을 보려면 `/dev/mailbox`로 이동합니다.

### 추적 세션

모든 세션과 토큰은 별도의 테이블에서 추적됩니다.
이를 통해 각 계정에 대해 활성화된 세션 수를 추적할 수 있습니다.
원하는 경우 이 정보를 사용자에게 노출할 수도 있습니다.

비밀번호가 변경될 때마다(비밀번호 재설정을 통해 또는 직접 변경) 모든 토큰이 삭제되며, 사용자는 모든 디바이스에서 다시 로그인해야 한다는 점에 유의하세요.

### 사용자 열거 공격

사용자 열거 공격은 누군가가 애플리케이션에 등록된 이메일이 있는지 확인할 수 있게 해줍니다.
생성된 인증 코드는 이러한 확인으로부터 보호하려고 시도하지 않습니다.
예를 들어, 계정을 등록할 때 이메일이 이미 등록되어 있으면 코드가 사용자에게 이메일이 이미 등록되어 있음을 알려줍니다.

애플리케이션이 열거형 공격에 민감한 경우, 보안과 사용자 경험의 균형을 신중하게 고려해야 하므로 대부분의 애플리케이션과는 매우 다른 자체 워크플로를 구현해야 합니다.

또한 열거형 공격이 걱정된다면 타이밍 공격도 주의해야 합니다.
예를 들어, 새 계정을 등록하려면 일반적으로 이미 계정이 있을 때보다 데이터베이스에 글을 쓰거나 이메일을 보내는 등의 추가 작업이 필요합니다.
누군가 이메일을 열거하기 위해 이러한 추가 작업을 실행하는 데 걸리는 시간을 측정할 수 있습니다.
이는 이메일, 인앱 알림 등을 전송할 수 있는 모든 엔드포인트(등록, 확인, 비밀번호 복구 등)에 적용됩니다.

### 대소문자 구분

이메일 조회는 대소문자를 구분하지 않도록 설정되어 있습니다.
대소문자를 구분하지 않는 조회는 MySQL과 MSSQL에서 기본값입니다.
SQLite3에서는 이를 지원하기 위해 열 정의에 [`COLLATE NOCASE`](https://www.sqlite.org/datatype3.html#collating_sequences)를 사용합니다.
PostgreSQL에서는 [`citext` 확장자](https://www.postgresql.org/docs/current/citext.html)를 사용합니다.

참고 `citext`는 PostgreSQL 자체의 일부이며 대부분의 운영 체제 및 패키지 관리자에 번들로 제공됩니다. `mix phx.gen.auth`가 확장자 생성을 처리하므로 대부분의 경우 추가 작업이 필요하지 않습니다.
혹시라도 패키지 관리자가 `citext`를 별도의 패키지로 분할하는 경우 마이그레이션 중에 오류가 발생할 수 있으며, 이 경우 `postgres-contrib` 패키지를 설치하면 해결할 수 있습니다.

### 동시성 테스트

생성된 테스트는 동시 테스트를 지원하는 데이터베이스를 사용하는 경우, 즉 PostgreSQL의 경우 동시 테스트를 지원하는 데이터베이스를 사용하는 경우 동시에 실행됩니다.

## `mix phx.gen.auth`에 대한 추가 정보

다른 비밀번호 해싱 라이브러리 사용, 웹 모듈 네임스페이스 사용자 지정, 바이너리 ID 유형 생성, 기본 옵션 구성, 사용자 정의 테이블 이름 사용 등 자세한 내용은 `mix phx.gen.auth`를 참조하세요.

## 추가 리소스

다음 링크에서 생성하는 코드의 동기와 설계에 관한 자세한 정보를 확인할 수 있습니다.

- 인증용 라이브뷰(기존 컨트롤러 및 뷰 대신)를 생성하는 방법에 대한 Berenice Medel의 블로그 게시물 - [Phoenix 인증에 생명을 불어넣다](https://fly.io/phoenix-files/phx-gen-auth/)
- 호세 발림의 블로그 게시물 - [곧 출시될 Phoenix용 인증 솔루션](https://dashbit.co/blog/a-new-authentication-solution-for-phoenix)
- [[원본 `phx_gen_auth`](Phoenix 1.5 애플리케이션용)](https://github.com/aaronrenner/phx_gen_auth) - 프로젝트의 이전 버전에서 이루어진 결정에 대한 토론을 볼 수 있는 훌륭한 리소스입니다.
- [베어 피닉스 앱의 원본 풀 리퀘스트](https://github.com/dashbitco/mix_phx_gen_auth_demo/pull/1)
- [원본 디자인 사양](https://github.com/dashbitco/mix_phx_gen_auth_demo/blob/auth/README.md)
