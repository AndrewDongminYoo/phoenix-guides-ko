# 설치

Phoenix 애플리케이션을 빌드하려면 운영 체제에 몇 가지 종속성을 설치해야 합니다:

- Erlang 가상 머신과 Elixir 프로그래밍 언어
- 데이터베이스 - Phoenix는 PostgreSQL을 권장하지만 다른 데이터베이스를 선택하거나 데이터베이스를 전혀 사용하지 않을 수 있습니다.
- 기타 옵션 패키지.

이 목록을 살펴보고 시스템에 필요한 모든 것을 설치하세요.
종속성을 미리 설치하면 나중에 문제가 발생하는 것을 방지할 수 있습니다.

## 엘릭서 1.14 이상 버전

Phoenix는 Elixir로 작성되었으며, 애플리케이션 코드도 Elixir로 작성됩니다.
이 기능 없이는 Phoenix 앱에서 멀리 갈 수 없습니다! 엘릭서 사이트에는 도움이 되는 훌륭한 [설치 페이지](https://elixir-lang.org/install.html)가 있습니다.

Elixir를 처음 설치했다면 Hex 패키지 관리자도 설치해야 합니다.
Hex는 Phoenix 앱을 실행하고(종속성을 설치하여) 그 과정에서 필요한 추가 종속성을 설치하는 데 필요합니다.

다음은 Hex를 설치하는 명령어입니다(Hex가 이미 설치되어 있는 경우 Hex를 최신 버전으로 업그레이드합니다):

```shell
mix local.hex
```

## Erlang 24 이상

Elixir 코드는 Erlang 바이트 코드로 컴파일되어 Erlang 가상 머신에서 실행됩니다.
Erlang이 없으면 Elixir 코드를 실행할 가상 머신이 없으므로 Erlang도 설치해야 합니다.

Elixir [설치 페이지](https://elixir-lang.org/install.html)의 안내에 따라 Elixir를 설치하면 일반적으로 Erlang도 함께 설치됩니다.
Elixir와 함께 설치되지 않은 경우, Elixir 설치 페이지의 [Erlang 지침](https://elixir-lang.org/install.html#installing-erlang) 섹션에서 지침을 참조하세요.

## Phoenix

Elixir 1.14 및 Erlang 24 이상을 사용 중인지 확인하려면 다음과 같이 실행하세요:

```shell
$ elixir -v
Erlang/OTP 24 [erts-12.0] [source] [64-bit] [smp:8:8] [async-threads:10] [hipe] [kernel-poll:false] [dtrace]

Elixir 1.14.0
```

Elixir와 Erlang이 준비되었으면 이제 Phoenix 애플리케이션 생성기를 설치할 준비가 되었습니다:

```shell
mix archive.install hex phx_new
```

이제 `phx.new` 생성기를 사용하여 다음 가이드인 [가동 및 실행](up_and_running.html)에서 새 애플리케이션을 생성할 수 있습니다.
아래에 언급된 플래그는 생성기에 대한 명령줄 옵션이며, `mix help phx.new`를 호출하여 사용 가능한 모든 옵션을 확인할 수 있습니다.

## PostgreSQL

PostgreSQL은 관계형 데이터베이스 서버입니다.
Phoenix는 기본적으로 이 서버를 사용하도록 애플리케이션을 구성하지만, 새 애플리케이션을 만들 때 `--database` 플래그를 전달하여 MySQL, MSSQL 또는 SQLite3로 전환할 수 있습니다.

데이터베이스와 통신하기 위해 Phoenix 애플리케이션은 [Ecto](https://github.com/elixir-ecto/ecto)라는 다른 Elixir 패키지를 사용합니다.
애플리케이션에서 데이터베이스를 사용하지 않으려는 경우 `--no-ecto` 플래그를 전달하면 됩니다.

그러나 Phoenix를 처음 시작하는 경우 PostgreSQL을 설치하고 실행 중인지 확인하는 것이 좋습니다.
다양한 시스템에 대한 [설치 가이드](https://wiki.postgresql.org/wiki/Detailed_installation_guides)는 PostgreSQL 위키에 있습니다.

## inotify-tools(Linux 사용자)

Phoenix는 라이브 리로딩이라는 매우 편리한 기능을 제공합니다.
보기나 에셋을 변경하면 브라우저에서 페이지를 자동으로 새로고침합니다.
이 기능이 작동하려면 파일 시스템 감시기가 필요합니다.

macOS 및 Windows 사용자에게는 이미 파일 시스템 감시기가 있지만 Linux 사용자는 inotify-tools를 설치해야 합니다.
배포판별 설치 지침은 [inotify-tools 위키](https://github.com/rvoicilas/inotify-tools/wiki)를 참조하세요.

## 요약

이 섹션이 끝나면 Elixir, Hex, Phoenix 및 PostgreSQL을 설치했어야 합니다.
이제 모든 것이 설치되었으므로 첫 번째 Phoenix 애플리케이션을 생성하고 [가동 및 실행](up_and_running.html)해 보겠습니다.
