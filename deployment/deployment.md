# 배포 소개

작동하는 애플리케이션이 완성되면 배포할 준비가 된 것입니다.
아직 애플리케이션을 완성하지 못했더라도 걱정하지 마세요.
[가동 및 실행 가이드](up_and_running.html)에 따라 작업할 기본 애플리케이션을 만들면 됩니다.

배포를 위해 애플리케이션을 준비할 때는 크게 세 가지 단계가 있습니다:

- 애플리케이션 비밀 처리
- 애플리케이션 자산 컴파일
- 프로덕션 환경에서 서버 시작하기

이 가이드에서는 프로덕션 환경을 로컬에서 실행하는 방법을 알아봅니다.
이 가이드의 동일한 기술을 사용하여 프로덕션 환경에서 애플리케이션을 실행할 수 있지만 배포 인프라에 따라 추가 단계가 필요할 수 있습니다.

이 가이드에서는 다른 인프라에 배포하는 예로 'mix release'와 함께 [Elixir의 릴리스](releases.html) 사용, [Gigalixir 사용](gigalixir.html), [Fly 사용](fly.html), [Heroku 사용](heroku.html) 등 네 가지 접근 방식에 대해서도 설명합니다.
또한 [커뮤니티 배포 가이드](/deployment.md#community-deployment-guides)에 다른 플랫폼에 Phoenix를 배포할 수 있는 링크도 포함되어 있습니다.
마지막으로, 릴리스 가이드에는 컨테이너 기술로 배포하는 것을 선호하는 경우 사용할 수 있는 샘플 Docker파일이 있습니다.

위의 단계를 하나씩 살펴보겠습니다.

## 애플리케이션 비밀 처리

모든 Phoenix 애플리케이션에는 프로덕션 데이터베이스의 사용자 이름과 비밀번호, 중요한 정보를 서명하고 암호화하는 데 사용하는 비밀과 같이 보안을 유지해야 하는 데이터가 있습니다.
일반적으로 이러한 데이터는 환경 변수에 보관하고 애플리케이션에 로드하는 것이 좋습니다.
이 작업은 환경 변수에서 시크릿과 구성을 로드하는 역할을 하는 `config/runtime.exs`(이전의 `config/prod.secret.exs` 또는 `config/releases.exs`)에서 수행됩니다.

따라서 프로덕션 환경에서 적절한 관련 변수가 설정되어 있는지 확인해야 합니다:

```shell
$ mix phx.gen.secret
REALLY_LONG_SECRET
export SECRET_KEY_BASE=REALLY_LONG_SECRET
export DATABASE_URL=ecto://USER:PASS@HOST/database
```

해당 값을 직접 복사하지 말고 `mix phx.gen.secret`의 결과에 따라 `SECRET_KEY_BASE`를 설정하고 데이터베이스 주소에 따라 `DATABASE_URL`을 설정하세요.

어떤 이유로 환경 변수에 의존하고 싶지 않다면 `config/runtime.exs`에 시크릿을 하드 코딩할 수 있지만, 버전 관리 시스템에서 파일을 확인하지 않도록 주의하세요.

비밀 정보가 제대로 보호되었으면 이제 에셋을 구성할 차례입니다!

이 단계를 수행하기 전에 한 가지 준비가 필요합니다.
프로덕션을 위해 모든 것을 준비할 것이므로 해당 환경에서 종속성을 가져오고 컴파일하여 몇 가지 설정을 수행해야 합니다.

```shell
mix deps.get --only prod
MIX_ENV=prod mix compile
```

## 애플리케이션 에셋 컴파일

이 단계는 JavaScript 및 스타일시트와 같은 컴파일 가능한 에셋이 있는 경우에만 필요합니다.
기본적으로 Phoenix는 `esbuild`를 사용하지만 모든 것이 `mix.exs`에 정의된 단일 `mix assets.deploy` 태스크에 캡슐화됩니다:

```shell
MIX_ENV=prod mix assets.deploy
Check your digested files at "priv/static".
```

이게 다입니다! Mix 태스크는 기본적으로 에셋을 빌드한 다음 캐시 매니페스트 파일과 함께 다이제스트를 생성하여 Phoenix가 프로덕션 환경에서 에셋을 신속하게 제공할 수 있도록 합니다.

> 참고: 로컬 머신에서 위의 작업을 실행하면 `priv/static`에 많은 디제스트된 에셋이 생성됩니다.
> 이 에셋은 `mix phx.digest.clean --all`을 실행하여 정리할 수 있습니다.

혹시라도 위 단계를 실행하는 것을 잊어버리면 Phoenix에서 오류 메시지를 표시한다는 점에 유의하세요:

```shell
PORT=4001 MIX_ENV=prod mix phx.server
10:50:18.732 [info] Running MyAppWeb.Endpoint with Cowboy on http://example.com
10:50:18.735 [error] Could not find static manifest at "my_app/_build/prod/lib/foo/priv/static/cache_manifest.json".
Run "mix phx.digest" after building your static files or remove the configuration from "config/prod.exs".
```

오류 메시지는 매우 명확합니다. Phoenix가 정적 매니페스트를 찾을 수 없다는 내용입니다.
위의 명령을 실행하여 문제를 해결하거나, 서비스를 제공하지 않거나 에셋을 전혀 신경 쓰지 않는 경우 구성에서 `cache_static_manifest` 구성을 제거하면 됩니다.

## 프로덕션 환경에서 서버 시작하기

프로덕션 환경에서 Phoenix를 실행하려면 `mix phx.server`를 호출할 때 `PORT` 및 `MIX_ENV` 환경 변수를 설정해야 합니다:

```shell
PORT=4001 MIX_ENV=prod mix phx.server
10:59:19.136 [info] Running MyAppWeb.Endpoint with Cowboy on http://example.com
```

터미널을 닫아도 Phoenix 서버가 멈추지 않고 계속 실행되도록 분리 모드로 실행합니다:

```shell
PORT=4001 MIX_ENV=prod elixir --erl "-detached" -S mix phx.server
```

오류 메시지가 표시되면 주의 깊게 읽어보고 해결 방법이 명확하지 않은 경우 버그 리포트를 열어주세요.

대화형 셸에서 애플리케이션을 실행할 수도 있습니다:

```shell
PORT=4001 MIX_ENV=prod iex -S mix phx.server
10:59:19.136 [info] Running MyAppWeb.Endpoint with Cowboy on http://example.com
```

## 모든 것을 종합하기

이전 섹션에서는 Phoenix 애플리케이션을 배포하는 데 필요한 주요 단계에 대한 개요를 설명했습니다.
실제로는 자신만의 단계를 추가하게 될 것입니다.
예를 들어 데이터베이스를 사용하는 경우 서버를 시작하기 전에 `mix ecto.migrate`를 실행하여 데이터베이스가 최신 상태인지 확인해야 합니다.

전반적으로 다음은 시작점으로 사용할 수 있는 스크립트입니다:

```shell
# Initial setup
mix deps.get --only prod
MIX_ENV=prod mix compile

# Compile assets
MIX_ENV=prod mix assets.deploy

# Custom tasks (like DB migrations)
MIX_ENV=prod mix ecto.migrate

# Finally run the server
PORT=4001 MIX_ENV=prod mix phx.server
```

여기까지입니다.
다음으로 공식 가이드 중 하나를 사용하여 배포할 수 있습니다:

- [엘릭서 릴리즈와 함께](releases.html)
- 엘릭서 중심의 서비스형 플랫폼(PaaS)인 [Gigalixir](gigalixir.html)로 배포하기
- 기본 제공 배포 지원으로 사용자와 가까운 곳에 서버를 배포하는 PaaS인 [Fly.io](fly.html)로 이동합니다.
- 그리고 가장 인기 있는 PaaS 중 하나인 [Heroku](heroku.html)로 이동합니다.

## 커뮤니티 배포 가이드

- [Render](https://render.com)는 Phoenix 애플리케이션에 대한 최고 수준의 지원을 제공합니다.
  [Mix 릴리즈](https://render.com/docs/deploy-phoenix), [Distillery](https://render.com/docs/deploy-phoenix-distillery), [Distributed Elixir 클러스터](https://render.com/docs/deploy-elixir-cluster)로 Phoenix를 호스팅하는 방법에 대한 가이드가 있습니다.
