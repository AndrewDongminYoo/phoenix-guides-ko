# 패키지 용어집

기본적으로 Phoenix 애플리케이션은 용도가 다른 여러 패키지에 의존합니다.
이 페이지는 Phoenix 개발자로서 작업할 수 있는 다양한 패키지에 대한 빠른 참조입니다.

주요 패키지는 다음과 같습니다:

- [Ecto](https://hexdocs.pm/ecto) - 언어 통합 쿼리 및 데이터베이스 래퍼

- [Phoenix](https://hexdocs.pm/phoenix) - Phoenix 웹 프레임워크(이 문서)

- [피닉스 라이브뷰](https://hexdocs.pm/phoenix_live_view) - 서버 렌더링 HTML로 풍부한 실시간 사용자 경험을 구축합니다.
  라이브뷰 프로젝트는 일반 애플리케이션과 실시간 애플리케이션 모두에서 HTML 콘텐츠를 렌더링하는 데 사용되는 [`Phoenix.Component`](https://hexdocs.pm/phoenix_live_view/Phoenix.Component.html) 및 [HEEx 템플릿 엔진](https://hexdocs.pm/phoenix_live_view/Phoenix.Component.html#sigil_H/2)도 정의합니다.

- [플러그](https://hexdocs.pm/plug) - 컴포저블 모듈 웹 애플리케이션을 구축하기 위한 사양 및 편의 기능.
  연결 추상화와 정기적인 요청-응답 라이프사이클을 담당하는 패키지입니다.

또한 다음과 함께 작동합니다:

- [ExUnit](https://hexdocs.pm/ex_unit) - Elixir의 빌트인 테스트 프레임워크

- [Gettext](https://hexdocs.pm/gettext) - [`gettext`](https://www.gnu.org/software/gettext/)를 통한 국제화 및 현지화

- [Swoosh](https://hexdocs.pm/swoosh) - 이메일 작성, 전달 및 테스트를 위한 라이브러리로, `mix phx.gen.auth`에서도 사용됩니다.

이 라이브러리들을 자세히 들여다보면 Phoenix 애플리케이션에서 중요한 역할을 하는 것을 알 수 있습니다:

- [Phoenix HTML](https://hexdocs.pm/phoenix_html) - HTML 및 양식 작업을 안전하게 하기 위한 빌딩 블록입니다.
- [피닉스 엑토](https://hex.pm/packages/phoenix_ecto) - 피닉스를 엑토와 함께 사용하기 위한 플러그 및 프로토콜 구현

- [피닉스 펍서브](https://hexdocs.pm/phoenix_pubsub) - 실시간 프레즌스 지원 기능을 갖춘 분산형 펍/서브 시스템

계측 및 모니터링에 관해서는 다음을 확인하세요:

- [피닉스 라이브 대시보드](https://hexdocs.pm/phoenix_live_dashboard) - 피닉스 개발자를 위한 실시간 성능 모니터링 및 디버깅 도구

- [텔레메트리 메트릭](https://hexdocs.pm/telemetry_metrics) - 텔레메트리 이벤트에 기반한 메트릭을 정의하기 위한 공통 인터페이스입니다.
