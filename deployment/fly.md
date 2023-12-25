# Fly.io에 배포하기

Fly.io는 여기에서 Elixir/Phoenix에 대한 자체 가이드를 유지 관리합니다: [Fly.io/docs/elixir/getting-started/](https://fly.io/docs/elixir/getting-started/) 이 가이드를 계속 업데이트할 예정이지만, 가장 최신의 내용은 이 가이드를 통해 확인하시기 바랍니다!

준비물 ## 필요한 것

이 가이드를 위해 필요한 것은 작동하는 Phoenix 애플리케이션뿐입니다.
간단한 애플리케이션을 배포해야 하는 경우 [시작 및 실행 가이드](https://hexdocs.pm/phoenix/up_and_running.html)를 참조하세요.

You can just:

```shell
mix phx.new my_app
```

## Goals

이 가이드의 주요 목표는 [Fly.io](https://fly.io)에서 Phoenix 애플리케이션을 실행하는 것입니다.

## Sections

이 프로세스를 몇 가지 단계로 나누어 현재 위치를 추적할 수 있도록 하겠습니다.

- Fly.io CLI 설치
- Fly.io에 가입하기
- Fly.io에 앱 배포
- 추가 Fly.io 팁
- 유용한 Fly.io 리소스

## Fly.io CLI 설치하기

[여기](https://fly.io/docs/getting-started/installing-flyctl/)의 지침에 따라 Fly.io 플랫폼용 명령줄 인터페이스인 Flyctl을 설치하세요.

## Fly.io에 가입하기

CLI를 사용하여 [계정 가입](https://fly.io/docs/getting-started/log-in-to-fly/)을 할 수 있습니다.

```shell
fly auth signup
```

또는 로그인합니다.

```shell
flyctl auth login
```

Fly에는 대부분의 애플리케이션을 위한 [무료 티어](https://fly.io/docs/about/pricing/)가 있습니다.
계정 설정 시 신용카드는 도용 방지를 위해 필요합니다.
자세한 내용은 [가격](https://fly.io/docs/about/pricing/) 페이지를 참조하세요.

## Fly.io에 앱 배포하기

Fly에 애플리케이션에 대한 정보를 알려주려면 소스 코드가 있는 디렉토리에서 `fly launch`를 실행합니다.
그러면 Fly.io 앱이 생성되고 구성됩니다.

```shell
fly launch
```

그러면 소스를 스캔하고 Phoenix 프로젝트를 감지한 후 `mix phx.gen.release --docker`를 실행합니다! 그러면 도커 파일이 생성됩니다.

`fly launch` 명령은 몇 가지 질문을 안내합니다.

- 앱의 이름을 직접 지정하거나 임의의 이름을 생성하도록 할 수 있습니다.
- 조직을 선택합니다(기본값은 `개인`).
  조직은 Fly.io 사용자 간에 애플리케이션과 리소스를 공유하는 방법입니다.
- 배포할 지역을 선택합니다.
  기본값은 가장 가까운 Fly.io 지역입니다.
  전체 리전 목록은 여기에서 확인할 수 있습니다(https://fly.io/docs/reference/regions/).
- Postgres DB를 설정합니다.
- 도커파일을 빌드합니다.
- 애플리케이션을 배포합니다!

`fly launch` 명령은 `fly.toml` 파일도 생성합니다.
이 파일에서 환경 값과 기타 구성을 설정할 수 있습니다.

### Fly.io에 시크릿 저장하기

앱에 설정하고 싶은 시크릿이 있을 수도 있습니다.

이를 설정하려면 [`fly secrets`](https://fly.io/docs/reference/secrets/#setting-secrets)를 사용하세요.

```shell
fly secrets set MY_SECRET_KEY=my_secret_value
```

### 다시 배포하기

애플리케이션에 변경 사항을 배포하려면 `fly deploy`를 사용합니다.

```shell
fly deploy
```

참고: Apple Silicon(M1) 컴퓨터에서 docker는 qemu를 사용하여 크로스 플랫폼 빌드를 실행하는데, 이는 항상 작동하지 않을 수 있습니다.
다음과 같은 세분화 오류 오류가 발생하면:

```shell
 => [build  7/17] RUN mix deps.get --only
 => => # qemu: uncaught target signal 11 (Segmentation fault) - core dumped
```

`--remote-only` 플래그를 추가하여 플라이의 원격 빌더를 사용할 수 있습니다:

```shell
fly deploy --remote-only
```

언제든지 배포 상태를 확인할 수 있습니다.

```shell
fly status
```

앱 로그 확인

```shell
fly logs
```

모든 것이 정상이라면 Fly에서 앱을 엽니다.

```shell
fly open
```

## 추가 Fly.io 팁

### 실행 중인 노드에 IEx 셸 가져오기

엘릭서는 실행 중인 프로덕션 노드에 IEx 셸을 가져오는 기능을 지원합니다.

몇 가지 전제 조건이 있는데, 먼저 Fly.io에서 우리 머신에 [SSH 셸](https://fly.io/docs/flyctl/ssh/)을 설정해야 합니다.

이 단계에서는 계정에 대한 루트 인증서를 설정한 다음 인증서를 발급합니다.

```shell
fly ssh issue --agent
```

SSH를 구성한 후 콘솔을 열어 보겠습니다.

```shell
fly ssh console
Connecting to my-app-1234.internal... complete
/ #
```

모든 것이 순조롭게 진행되었다면 머신에 셸을 설치한 것입니다! 이제 원격 IEx 셸을 실행하기만 하면 됩니다.
배포 도커파일은 애플리케이션을 `/app`으로 가져오도록 구성되었습니다.
따라서 `my_app`이라는 이름의 앱에 대한 명령은 다음과 같습니다:

```shell
app/bin/my_app remote
Erlang/OTP 23 [erts-11.2.1] [source] [64-bit] [smp:1:1] [ds:1:1:10] [async-threads:1]

Interactive Elixir (1.11.2) - press Ctrl+C to exit (type h() ENTER for help)
iex(my_app@fdaa:0:1da8:a7b:ac4:b204:7e29:2)1>
```

이제 노드에 IEx 셸이 실행 중입니다! CTRL+C, CTRL+C를 사용하여 안전하게 연결을 끊을 수 있습니다.

### 애플리케이션 클러스터링

Elixir와 BEAM은 서로 클러스터링하여 노드 간에 메시지를 원활하게 전달할 수 있는 놀라운 기능을 가지고 있습니다.
이 가이드에서는 Elixir 애플리케이션을 클러스터링하는 방법을 안내합니다.

Fly.io에서 클러스터링을 빠르게 설정하는 방법은 크게 두 부분으로 나뉩니다.

- `libcluster` 설치 및 사용하기
- 애플리케이션을 여러 인스턴스로 확장하기

#### `libcluster` 추가하기

널리 사용되는 라이브러리인 [libcluster](https://github.com/bitwalker/libcluster)가 도움이 됩니다.

`libcluster`가 다른 노드를 찾고 연결하는 데 사용할 수 있는 여러 가지 전략이 있습니다.
Fly.io에서 사용할 전략은 `DNSPoll`입니다.

`libcluster`를 설치한 후 다음과 같이 애플리케이션에 추가합니다:

```perl Elixir
defmodule MyApp.Application do
  use Application

  def start(_type, _args) do
    topologies = Application.get_env(:libcluster, :topologies) || []

    children = [
      # ...
      # setup for clustering
      {Cluster.Supervisor, [topologies, [name: MyApp.ClusterSupervisor]]}
    ]

    # ...
  end

  # ...
end
```

다음 단계는 `config/runtime.exs`에 `topologies` 구성을 추가하는 것입니다.

```perl Elixir
  app_name =
    System.get_env("FLY_APP_NAME") ||
      raise "FLY_APP_NAME not available"

  config :libcluster,
    topologies: [
      fly6pn: [
        strategy: Cluster.Strategy.DNSPoll,
        config: [
          polling_interval: 5_000,
          query: "#{app_name}.internal",
          node_basename: app_name
        ]
      ]
    ]
```

이렇게 하면 `libcluster`가 `DNSPoll` 전략을 사용하고 `.internal` 사설 네트워크에서 `$FLY_APP_NAME`을 사용하여 배포된 다른 앱을 찾도록 구성됩니다.

#### 노드 이름 제어하기

Elixir 노드의 이름을 제어해야 합니다.
연결을 돕기 위해 `your-fly-app-name@the.ipv6.address.on.fly` 패턴을 사용하여 이름을 지정합니다.
이를 위해 릴리스 구성을 생성합니다.

```shell
mix release.init
```

그런 다음 생성된 `rel/env.sh.eex` 파일을 편집하고 다음 줄을 추가합니다:

```shell
ip=$(grep fly-local-6pn /etc/hosts | cut -f 1)
export RELEASE_DISTRIBUTION=name
export RELEASE_NODE=$FLY_APP_NAME@$ip
```

변경한 후 앱을 배포하세요!

```shell
fly deploy
```

앱을 클러스터링하려면 여러 개의 인스턴스가 있어야 합니다.
다음으로 노드 인스턴스를 추가하겠습니다.

#### 여러 인스턴스 실행하기

여러 인스턴스를 실행하는 방법에는 두 가지가 있습니다.

1.. 애플리케이션을 확장하여 한 지역에 여러 인스턴스가 있도록 합니다.
2.. 다른 리전(여러 리전)에 인스턴스를 추가합니다.

먼저 단일 배포의 기준선부터 시작하겠습니다.

```shell
fly status
...
Instances
ID       VERSION REGION DESIRED STATUS  HEALTH CHECKS      RESTARTS CREATED
f9014bf7 26      sea    run     running 1 total, 1 passing 0        1h8m ago
```

#### 단일 리전에서 확장

현재 리전에서 인스턴스를 최대 2개까지 확장해 보겠습니다.

```shell
fly scale count 2
Count changed to 2
```

상태를 확인하면 어떤 일이 발생했는지 알 수 있습니다.

```shell
fly status
...
Instances
ID       VERSION REGION DESIRED STATUS  HEALTH CHECKS      RESTARTS CREATED
eb4119d3 27      sea    run     running 1 total, 1 passing 0        39s ago
f9014bf7 27      sea    run     running 1 total, 1 passing 0        1h13m ago
```

이제 같은 지역에 두 개의 인스턴스가 있습니다.

이 인스턴스들이 서로 클러스터링되어 있는지 확인해 보겠습니다.
로그를 확인하면 됩니다:

```shell
fly logs
...
app[eb4119d3] sea [info] 21:50:21.924 [info] [libcluster:fly6pn] connected to :"my-app-1234@fdaa:0:1da8:a7b:ac2:f901:4bf7:2"
...
```

하지만 노드 내부에서 보는 것만큼 보람 있는 일은 아닙니다.
IEx 셸에서 연결된 노드에 어떤 다른 노드를 볼 수 있는지 물어볼 수 있습니다.

```shell
fly ssh console -C "/app/bin/my_app remote"
```

```perl Elixir
iex(my-app-1234@fdaa:0:1da8:a7b:ac2:f901:4bf7:2)1> Node.list
[:"my-app-1234@fdaa:0:1da8:a7b:ac4:eb41:19d3:2"]
```

IEx 프롬프트는 연결된 노드의 IP 주소를 표시하는 데 도움이 되도록 포함되어 있습니다.
그런 다음 `Node.list`를 가져오면 다른 노드가 반환됩니다.
두 인스턴스가 연결되고 클러스터링되었습니다!

#### 여러 지역으로 확장

Fly를 사용하면 사용자와 더 가까운 곳에 인스턴스를 쉽게 배포할 수 있습니다.
DNS의 마법을 통해 사용자는 애플리케이션이 위치한 가장 가까운 지역으로 연결됩니다.
Fly.io 리전에 대한 자세한 내용은 여기(https://fly.io/docs/reference/regions/)에서 확인할 수 있습니다.

미국 워싱턴주 시애틀의 `sea`에서 실행되는 단일 인스턴스의 기준선에서 다시 시작하여 미국 뉴저지주 파시파니의 `ewr` 지역을 추가해 보겠습니다.
이렇게 하면 미국 양쪽 해안에 인스턴스가 배치됩니다.

```shell
fly regions add ewr
Region Pool:
ewr
sea
Backup Region:
iad
lax
sjc
vin
```

상태를 보면 카운트가 1로 설정되어 있기 때문에 지역이 1개뿐이라는 것을 알 수 있습니다.

```shell
fly status
...
Instances
ID       VERSION REGION DESIRED STATUS  HEALTH CHECKS      RESTARTS CREATED
cdf6c422 29      sea    run     running 1 total, 1 passing 0        58s ago
```

두 번째 인스턴스를 추가하고 `ewr`에 배포해 보겠습니다.

```shell
fly scale count 2
Count changed to 2
```

이제 상태는 2개의 지역에 2개의 인스턴스가 분산되어 있음을 보여줍니다!

```shell
fly status
...
Instances
ID       VERSION REGION DESIRED STATUS  HEALTH CHECKS      RESTARTS CREATED
0a8e6666 30      ewr    run     running 1 total, 1 passing 0        16s ago
cdf6c422 30      sea    run     running 1 total, 1 passing 0        6m47s ago
```

서로 묶여 있는지 확인합니다.

```shell
fly ssh console -C "/app/bin/my_app remote"
```

```perl Elixir
iex(my-app-1234@fdaa:0:1da8:a7b:ac2:cdf6:c422:2)1> Node.list
[:"my-app-1234@fdaa:0:1da8:a7b:ab2:a8e:6666:2"]
```

북미 대륙의 서해안과 동해안에 두 개의 애플리케이션 인스턴스가 배포되어 있으며, 이 인스턴스들은 서로 클러스터링되어 있습니다! 사용자는 자동으로 가장 가까운 서버로 연결됩니다.

Fly.io 플랫폼에는 배포 지원이 내장되어 있어 여러 지역에 분산된 Elixir 노드를 쉽게 클러스터링할 수 있습니다.

## 유용한 Fly.io 리소스

계정 대시보드 열기

```shell
fly dashboard
```

애플리케이션 배포

```shell
fly deploy
```

배포된 애플리케이션의 상태 표시

```shell
fly status
```

로그 액세스 및 추적

```shell
fly logs
```

애플리케이션 확장 또는 축소

```shell
fly scale count 2
```

자세한 내용은 [Fly.io 엘릭서 문서](https://fly.io/docs/getting-started/elixir)를 참조하세요.

[Fly.io 애플리케이션으로 작업하기](https://fly.io/docs/getting-started/working-with-fly-apps/)에서는 다음과 같은 내용을 다룹니다:

- 상태 및 로그
- 사용자 정의 도메인
- 인증서

## 문제 해결

[문제 해결](https://fly.io/docs/getting-started/troubleshooting/) 및 [엘릭서 문제 해결](https://fly.io/docs/elixir/the-basics/troubleshooting/)을 참조하세요.

[Fly.io 커뮤니티](https://community.fly.io/)를 방문하여 해결책을 찾고 질문하세요.
