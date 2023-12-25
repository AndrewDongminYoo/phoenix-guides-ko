# 가동 및 실행

최대한 빨리 Phoenix 애플리케이션을 실행해 보겠습니다.

시작하기 전에 잠시 시간을 내어 [설치 가이드](installation.html)를 읽어주세요.
필요한 종속 요소를 미리 설치하면 애플리케이션을 원활하게 실행할 수 있습니다.

Phoenix 애플리케이션을 부트스트랩하기 위해 어느 디렉토리에서나 `mix phx.new`를 실행할 수 있습니다.
Phoenix는 새 프로젝트의 디렉터리에 대해 절대 경로 또는 상대 경로를 허용합니다.
애플리케이션의 이름이 `hello`라고 가정하고 다음 명령을 실행해 보겠습니다:

```shell
mix phx.new hello
```

> [Ecto](ecto.html)에 대한 참고 사항: Ecto를 사용하면 Phoenix 애플리케이션이 PostgreSQL, MySQL 등과 같은 데이터 저장소와 통신할 수 있습니다.
> 애플리케이션에 이 구성 요소가 필요하지 않은 경우 `--no-ecto` 플래그를 `mix phx.new`에 전달하여 이 종속성을 건너뛸 수 있습니다.
>
> mix phx.new`에 대한 자세한 내용은 [믹스 태스크 가이드](mix_tasks.html#phoenix-specific-mix-tasks)를 참조하세요.

```shell
$ mix phx.new hello
* creating hello/config/config.exs
* creating hello/config/dev.exs
* creating hello/config/prod.exs
...

Fetch and install dependencies? [Yn]
```

Phoenix는 애플리케이션에 필요한 디렉토리 구조와 모든 파일을 생성합니다.

> 피닉스는 버전 관리 소프트웨어로 git을 사용하도록 권장합니다. 생성된 파일 중 '.gitignore'를 찾습니다.
> 저장소를 `git init`하고 무시로 표시되지 않은 모든 것을 즉시 추가하고 커밋할 수 있습니다.

커밋이 완료되면 의존성을 설치할지 묻는 메시지가 표시됩니다.
'예'라고 대답해 봅시다.

```shell
Fetch and install dependencies? [Yn] Y
* running mix deps.get
* running mix assets.setup
* running mix deps.compile

We are almost there! The following steps are missing:

    $ cd hello

Then configure your database in config/dev.exs and run:

    $ mix ecto.create

Start your Phoenix app with:

    $ mix phx.server

You can also run your app inside IEx (Interactive Elixir) as:

    $ iex -S mix phx.server
```

종속성이 설치되면 프로젝트 디렉토리로 변경하고 애플리케이션을 시작하라는 메시지가 표시됩니다.

Phoenix는 PostgreSQL 데이터베이스에 올바른 권한과 "postgres"의 비밀번호를 가진 `postgres` 사용자 계정이 있다고 가정합니다.
그렇지 않은 경우, [믹스 작업 가이드](mix_tasks.html#ecto-specific-mix-tasks)를 참조하여 `mix ecto.create` 작업에 대해 자세히 알아보시기 바랍니다.

자, 한 번 해보겠습니다.
먼저 방금 만든 `hello/` 디렉터리에 `cd`를 생성합니다:

```shell
cd hello
```

이제 데이터베이스를 생성합니다:

```shell
$ mix ecto.create
Compiling 13 files (.ex)
Generated hello app
The database for Hello.Repo has been created
```

데이터베이스를 만들 수 없는 경우, 일반적인 문제 해결을 위해 [`mix ecto.create`](mix_tasks.html#mix-ecto-create) 가이드를 참조하세요.

> 참고: 이 명령을 처음 실행하는 경우 Phoenix에서 Rebar 설치를 요청할 수도 있습니다.
> Rebar는 Erlang 패키지를 빌드하는 데 사용되므로 설치를 계속 진행합니다.

마지막으로 Phoenix 서버를 시작합니다:

```shell
$ mix phx.server
[info] Running HelloWeb.Endpoint with cowboy 2.9.0 at 127.0.0.1:4000 (http)
[info] Access HelloWeb.Endpoint at http://localhost:4000
[watch] build finished, watching for changes...
...
```

새 애플리케이션을 생성할 때 Phoenix가 종속성을 설치하지 않도록 선택하면, 설치하려는 경우 `mix phx.new` 작업에서 필요한 단계를 수행하라는 메시지를 표시합니다.

```shell
Fetch and install dependencies? [Yn] n

We are almost there! The following steps are missing:

    $ cd hello
    $ mix deps.get

Then configure your database in config/dev.exs and run:

    $ mix ecto.create

Start your Phoenix app with:

    $ mix phx.server

You can also run your app inside IEx (Interactive Elixir) as:

    $ iex -S mix phx.server
```

기본적으로 Phoenix는 포트 4000에서 요청을 수락합니다.
즐겨찾는 웹 브라우저에서 [http://localhost:4000](http://localhost:4000)를 가리키면 Phoenix 프레임워크 시작 페이지가 표시됩니다.

![피닉스 시작 페이지](assets/images/welcome-to-phoenix.png)

화면이 위 이미지와 같다면 축하합니다! 이제 정상적으로 작동하는 Phoenix 애플리케이션이 생겼습니다.
위 페이지가 보이지 않는 경우, [http://127.0.0.1:4000](http://127.0.0.1:4000)를 통해 접속한 후 OS에서 "localhost"를 "127.0.0.1"로 정의했는지 확인하세요.

이를 중지하기 위해 `ctrl-c`를 두 번 누릅니다.

이제 피닉스가 제공하는 세계를 탐험할 준비가 되었습니다! 책, 스크린캐스트, 강좌 등을 보려면 [커뮤니티 페이지](community.html)를 참조하세요.

또는 이 가이드를 계속 읽으면서 Phoenix 애플리케이션을 구성하는 모든 부분을 간략하게 소개할 수 있습니다.
이 경우, 가이드를 순서와 상관없이 읽거나 [Phoenix 디렉토리 구조](directory_structure.html)를 설명하는 가이드부터 시작할 수 있습니다.
