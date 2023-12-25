# Contexts

> **Requirement**: This guide expects that you have gone through the [introductory guides](installation.html) and got a Phoenix application [up and running](up_and_running.html).
>
> **Requirement**: This guide expects that you have gone through the [Request life-cycle guide](request_lifecycle.html).
>
> **Requirement**: This guide expects that you have gone through the [Ecto guide](ecto.html).

지금까지 페이지를 만들고, 라우터를 통해 컨트롤러 동작을 연결하고, Ecto를 통해 데이터를 검증하고 지속시키는 방법을 배웠습니다.
이제 더 큰 Elixir 애플리케이션과 상호 작용하는 웹 기능을 작성하여 이 모든 것을 하나로 묶을 차례입니다.

Phoenix 프로젝트를 구축할 때, 우리는 무엇보다도 먼저 Elixir 애플리케이션을 구축합니다.
Phoenix의 역할은 Elixir 애플리케이션에 웹 인터페이스를 제공하는 것입니다.
당연히 모듈과 함수로 애플리케이션을 구성하지만, 애플리케이션을 설계할 때 단순히 몇 가지 기능으로 모듈을 정의하는 것만으로는 충분하지 않습니다.
모듈 간의 경계와 기능을 그룹화하는 방법을 고려해야 합니다.
즉, 코드를 작성할 때 애플리케이션 디자인에 대해 생각하는 것이 중요합니다.

## Thinking about design

컨텍스트는 관련 기능을 노출하고 그룹화하는 전용 모듈입니다.
예를 들어, `Logger.info/1` 또는 `Stream.map/2` 등 Elixir의 표준 라이브러리를 호출할 때마다 서로 다른 컨텍스트에 액세스하게 됩니다.
내부적으로 Elixir의 로거는 여러 모듈로 구성되어 있지만, 이러한 모듈과 직접 상호작용하지는 않습니다.
우리는 '로거' 모듈을 컨텍스트라고 부르는데, 바로 이 모듈이 모든 로깅 기능을 노출하고 그룹화하기 때문입니다.

관련 기능을 노출하고 그룹화하는 모듈에 **컨텍스트**라는 이름을 부여함으로써 개발자가 이러한 패턴을 식별하고 이에 대해 이야기할 수 있도록 돕습니다.
결국 컨텍스트는 컨트롤러, 뷰 등과 마찬가지로 모듈에 불과합니다.

Phoenix에서 컨텍스트는 종종 데이터 액세스와 데이터 유효성 검사를 캡슐화합니다.
컨텍스트는 종종 데이터베이스 또는 API와 통신합니다.
전반적으로 애플리케이션의 일부를 분리하고 격리하는 경계라고 생각하면 됩니다.
이러한 아이디어를 사용하여 웹 애플리케이션을 구축해 봅시다.
우리의 목표는 제품을 전시하고, 사용자가 장바구니에 제품을 추가하고, 주문을 완료할 수 있는 전자상거래 시스템을 구축하는 것입니다.

> How to read this guide: Using the context generators is a great way for beginners and intermediate Elixir programmers alike to get up and running quickly while thoughtfully designing their applications.
> This guide focuses on those readers.

### Adding a Catalog Context

전자상거래 플랫폼은 코드베이스 전반에 걸쳐 광범위하게 결합되어 있으므로 잘 정의된 인터페이스를 작성하는 것에 대해 미리 생각하는 것이 중요합니다.
이를 염두에 두고 시스템에서 사용 가능한 제품의 생성, 업데이트, 삭제를 처리하는 제품 카탈로그 API를 구축하는 것이 목표입니다.
먼저 제품을 보여주는 기본 기능부터 시작하고 나중에 쇼핑 카트 기능을 추가하겠습니다.
경계를 분리한 견고한 기반에서 시작하여 기능을 추가하면서 자연스럽게 애플리케이션을 성장시키는 방법을 살펴보겠습니다.

Phoenix에는 애플리케이션의 기능을 컨텍스트에 분리하는 아이디어를 적용하는 `mix phx.gen.html`, `mix phx.gen.json`, `mix phx.gen.live`, `mix phx.gen.context` 생성기가 포함되어 있습니다.
이러한 제너레이터는 Phoenix가 애플리케이션을 성장시키는 올바른 방향으로 안내하는 동안 시작을 위한 좋은 방법입니다.
새로운 제품 카탈로그 컨텍스트에 이러한 도구를 사용해 보겠습니다.

컨텍스트 생성기를 실행하려면 빌드 중인 관련 기능을 그룹화하는 모듈 이름을 만들어야 합니다.
[Ecto 가이드](ecto.html)에서 변경 집합과 리포지토리를 사용하여 사용자 스키마의 유효성을 검사하고 유지하는 방법을 살펴봤지만, 이를 애플리케이션 전체에 통합하지는 않았습니다.
사실, 애플리케이션에서 'user'가 어디에 있어야 하는지 전혀 생각하지 않았습니다.
한 걸음 물러나서 우리 시스템의 여러 부분에 대해 생각해 봅시다.
우리는 판매 페이지에 설명, 가격 등과 함께 소개할 제품이 있다는 것을 알고 있습니다.
제품 판매와 함께 카트, 주문 결제 등을 지원해야 한다는 것도 알고 있습니다.
구매되는 제품은 카트 및 결제 프로세스와 관련이 있지만, 제품을 소개하고 제품의 '전시'를 관리하는 것은 사용자가 카트에 무엇을 담았는지 또는 주문이 어떻게 이루어졌는지 추적하는 것과는 분명히 다릅니다.
`Catalog` 컨텍스트는 제품 세부 정보를 관리하고 판매 중인 제품을 전시하기 위한 자연스러운 장소입니다.

> Naming things is hard.
> If you're stuck when trying to come up with a context name when the grouped functionality in your system isn't yet clear, you can simply use the plural form of the resource you're creating.
> For example, a `Products` context for managing products.
> As you grow your application and the parts of your system become clear, you can simply rename the context to a more refined one.

카탈로그 컨텍스트를 시작하기 위해 `mix phx.gen.html`을 사용하여 제품 생성, 업데이트 및 삭제를 위한 엑토 액세스와 웹 인터페이스용 컨트롤러 및 템플릿과 같은 웹 파일을 컨텍스트에 래핑하는 컨텍스트 모듈을 만들겠습니다.
프로젝트 루트에서 다음 명령을 실행합니다:

```shell
$ mix phx.gen.html Catalog Product products title:string \
description:string price:decimal views:integer

* creating lib/hello_web/controllers/product_controller.ex
* creating lib/hello_web/controllers/product_html/edit.html.heex
* creating lib/hello_web/controllers/product_html/index.html.heex
* creating lib/hello_web/controllers/product_html/new.html.heex
* creating lib/hello_web/controllers/product_html/show.html.heex
* creating lib/hello_web/controllers/product_html/product_form.html.heex
* creating lib/hello_web/controllers/product_html.ex
* creating test/hello_web/controllers/product_controller_test.exs
* creating lib/hello/catalog/product.ex
* creating priv/repo/migrations/20210201185747_create_products.exs
* creating lib/hello/catalog.ex
* injecting lib/hello/catalog.ex
* creating test/hello/catalog_test.exs
* injecting test/hello/catalog_test.exs
* creating test/support/fixtures/catalog_fixtures.ex
* injecting test/support/fixtures/catalog_fixtures.ex

lib/hello_web/router.ex의 브라우저 범위에 리소스를 추가합니다:

    resources "/products", ProductController


마이그레이션을 실행하여 리포지토리를 업데이트하는 것을 잊지 마세요:

    $ mix ecto.migrate
```

> Note: we are starting with the basics for modeling an ecommerce system.
> In practice, modeling such systems yields more complex relationships such as product variants, optional pricing, multiple currencies, etc.
> We'll keep things simple in this guide, but the foundations will give you a solid starting point to building such a complete system.

Phoenix가 `lib/hello_web/`에서 예상대로 웹 파일을 생성했습니다.
또한 컨텍스트 파일이 `lib/hello/catalog.ex` 파일과 같은 이름의 디렉터리에 있는 제품 스키마에 생성된 것을 확인할 수 있습니다.
`lib/hello`와 `lib/hello_web`의 차이점에 주목하세요.
제품 카탈로그 기능을 위한 공용 API 역할을 하는 `Catalog` 모듈과 제품 데이터의 캐스팅 및 유효성 검사를 위한 Ecto 스키마인 `Catalog.Product` 구조체가 있습니다.
또한 Phoenix는 웹 및 컨텍스트 테스트를 제공했으며, 나중에 살펴볼 `Hello.Catalog` 컨텍스트를 통해 엔티티를 생성하기 위한 테스트 헬퍼도 포함했습니다.
지금은 콘솔의 지침에 따라 `lib/hello_web/router.ex`에 경로를 추가해 보겠습니다:

```diff
  scope "/", HelloWeb do
    pipe_through :browser

    get "/", PageController, :index
+   resources "/products", ProductController
  end
```

새 경로가 설정되면 Phoenix는 `mix ecto.migrate`를 실행하여 리포지토리를 업데이트하라고 알려주지만, 먼저 `priv/repo/migrations/*_create_products.exs`에서 생성된 마이그레이션을 몇 가지 조정해야 합니다:

```diff Elixir
  def change do
    create table(:products) do
      add :title, :string
      add :description, :string
-     add :price, :decimal
+     add :price, :decimal, precision: 15, scale: 6, null: false
-     add :views, :integer
+     add :views, :integer, default: 0, null: false

      timestamps()
    end
```

가격 열을 특정 정밀도 15, 스케일 6으로 수정하고 null이 아닌 제약 조건을 적용했습니다.
이렇게 하면 수행할 수 있는 모든 수학적 연산에 적합한 정밀도로 통화를 저장할 수 있습니다.
다음으로 조회 수에 기본값과 null이 아닌 제약 조건을 추가했습니다.
변경 사항을 적용했으므로 데이터베이스를 마이그레이션할 준비가 되었습니다.
이제 마이그레이션을 해보겠습니다:

```shell
mix ecto.migrate
14:09:02.260 [info] == Running 20210201185747 Hello.Repo.Migrations.CreateProducts.change/0 forward

14:09:02.262 [info] create table products

14:09:02.273 [info] == Migrated 20210201185747 in 0.0s
```

생성된 코드를 살펴보기 전에 `mix phx.server`로 서버를 시작하고 [http://localhost:4000/products](http://localhost:4000/products)를 방문해 보겠습니다.
"새 제품" 링크를 따라 아무 입력 없이 "저장" 버튼을 클릭합니다.
다음과 같은 출력이 표시되어야 합니다:

```shell
문제가 발생했습니다! 아래에서 오류를 확인해 보세요.
```

양식을 제출하면 입력과 함께 모든 유효성 검사 오류가 인라인으로 표시되는 것을 볼 수 있습니다.
멋지네요! 컨텍스트 생성기는 기본적으로 양식 템플릿에 스키마 필드를 포함했으며 필수 입력에 대한 기본 유효성 검사가 적용되고 있음을 알 수 있습니다.
몇 가지 예제 제품 데이터를 입력하고 양식을 다시 제출해 보겠습니다:

```shell
제품이 성공적으로 만들어졌습니다.

Title: 메타프로그래밍 엘릭서
설명: 코드 작성은 줄이고, 더 많은 작업을 수행하세요(그리고 재미있게!).
가격: 15.000000
조회수 0
```

"뒤로" 링크를 따라가면 모든 제품 목록이 표시되며, 여기에는 방금 만든 제품이 포함되어야 합니다.
마찬가지로 이 기록을 업데이트하거나 삭제할 수 있습니다.
이제 브라우저에서 어떻게 작동하는지 살펴봤으니 생성된 코드를 살펴볼 차례입니다.

## Starting With Generators

이 작은 `mix phx.gen.html` 명령에는 놀라운 기능이 담겨 있습니다.
카탈로그에서 제품을 생성, 업데이트, 삭제할 수 있는 많은 기능을 바로 사용할 수 있습니다.
모든 기능을 갖춘 앱과는 거리가 멀지만, 생성기는 무엇보다도 학습 도구이자 실제 기능을 구축하기 위한 출발점이라는 점을 기억하세요.
코드 생성이 모든 문제를 해결할 수는 없지만, Phoenix의 모든 것을 알려주고 애플리케이션을 설계할 때 올바른 사고방식을 갖도록 도와줄 것입니다.

먼저 `lib/hello_web/controllers/product_controller.ex`에서 생성된 `ProductController`를 확인해 보겠습니다:

```perl Elixir
defmodule HelloWeb.ProductController do
  use HelloWeb, :controller

  alias Hello.Catalog
  alias Hello.Catalog.Product

  def index(conn, _params) do
    products = Catalog.list_products()
    render(conn, :index, products: products)
  end

  def new(conn, _params) do
    changeset = Catalog.change_product(%Product{})
    render(conn, :new, changeset: changeset)
  end

  def create(conn, %{"product" => product_params}) do
    case Catalog.create_product(product_params) do
      {:ok, product} ->
        conn
        |> put_flash(:info, "Product created successfully.")
        |> redirect(to: ~p"/products/#{product}")

      {:error, %Ecto.Changeset{} = changeset} ->
        render(conn, :new, changeset: changeset)
    end
  end

  def show(conn, %{"id" => id}) do
    product = Catalog.get_product!(id)
    render(conn, :show, product: product)
  end
  ...
end
```

컨트롤러가 어떻게 작동하는지 [컨트롤러 가이드](controllers.html)에서 살펴봤으므로 이 코드가 그리 놀랍지 않을 것입니다.
주목할 만한 점은 컨트롤러가 `Catalog` 컨텍스트에서 호출하는 방식입니다.
`index` 액션이 `Catalog.list_products/0`으로 제품 목록을 가져오고, `create` 액션에서 `Catalog.create_product/1`로 제품이 어떻게 유지되는지 알 수 있습니다.
아직 카탈로그 컨텍스트를 살펴보지 않았기 때문에 내부에서 제품 가져오기 및 생성이 어떻게 이루어지는지 아직 알 수 없지만, 중요한 것은 그 점입니다.
Phoenix 컨트롤러는 더 큰 애플리케이션에 대한 웹 인터페이스입니다.
데이터베이스에서 제품을 가져오는 방법이나 스토리지에 저장하는 방법에 대한 세부 사항은 신경 쓰지 않아야 합니다.
우리는 애플리케이션에 몇 가지 작업을 수행하도록 지시하는 것만 신경 쓰면 됩니다.
비즈니스 로직과 스토리지 세부 정보가 애플리케이션의 웹 계층에서 분리되어 있기 때문에 이 점이 좋습니다.
나중에 SQL 쿼리 대신 제품을 가져오기 위해 전체 텍스트 스토리지 엔진으로 이동하는 경우 컨트롤러를 변경할 필요가 없습니다.
마찬가지로 채널, 믹스 작업 또는 CSV 데이터를 가져오는 장기 실행 프로세스 등 애플리케이션의 다른 인터페이스에서 컨텍스트 코드를 재사용할 수 있습니다.

`create` 액션의 경우, 제품을 성공적으로 만들면 `Phoenix.Controller.put_flash/3`을 사용하여 성공 메시지를 표시한 다음 라우터의 제품 표시 페이지로 리디렉션합니다.
반대로 `Catalog.create_product/1`이 실패하면 `"new.html"` 템플릿을 렌더링하고 템플릿에서 오류 메시지를 가져올 Ecto 변경 집합을 전달합니다.

다음으로, `lib/hello/catalog.ex`에서 `Catalog` 컨텍스트를 더 자세히 살펴보겠습니다:

```perl Elixir
defmodule Hello.Catalog do
  @moduledoc """
  The Catalog context.
  """

  import Ecto.Query, warn: false
  alias Hello.Repo

  alias Hello.Catalog.Product

  @doc """
  Returns the list of products.

  ## Examples

      iex> list_products()
      [%Product{}, ...]

  """
  def list_products do
    Repo.all(Product)
  end
  ...
end
```

이 모듈은 우리 시스템의 모든 제품 카탈로그 기능을 위한 공용 API가 됩니다.
예를 들어, 제품 세부 정보 관리 외에도 제품 카테고리 분류 및 옵션 사이즈, 트림 등의 제품 이형 상품도 처리할 수 있습니다.
`list_products/0` 함수를 보면 제품 가져오기의 비공개 세부 정보를 볼 수 있습니다.
매우 간단합니다.
`Repo.all(Product)`에 대한 호출이 있습니다.
엑토 리포지토리 쿼리가 어떻게 작동하는지 [엑토 가이드](ecto.html)에서 보았으므로 이 호출은 익숙하게 보일 것입니다.
`list_products` 함수는 코드의 _intent_, 즉 제품을 나열하는 것을 지정하는 일반화된 함수 이름입니다.
리포지토리를 사용하여 PostgreSQL 데이터베이스에서 제품을 가져오는 의도의 세부 사항은 호출자에게 숨겨져 있습니다.
이는 Phoenix 생성기를 사용하면서 반복적으로 보게 될 일반적인 주제입니다.
Phoenix를 사용하면 애플리케이션에서 서로 다른 책임이 있는 부분을 생각한 다음, 코드의 의도를 명확하게 하면서도 세부 사항을 캡슐화하는 잘 명명된 모듈과 함수로 이러한 다양한 영역을 마무리할 수 있습니다.

이제 데이터를 가져오는 방법은 알았지만 제품을 유지하는 방법은 어떻게 될까요? `Catalog.create_product/1` 함수를 살펴봅시다:

```perl Elixir
  @doc """
  Creates a product.

  ## Examples

      iex> create_product(%{field: value})
      {:ok, %Product{}}

      iex> create_product(%{field: bad_value})
      {:error, %Ecto.Changeset{}}

  """
  def create_product(attrs \\ %{}) do
    %Product{}
    |> Product.changeset(attrs)
    |> Repo.insert()
  end
```

여기에는 코드보다 문서가 더 많지만 몇 가지를 강조할 필요가 있습니다.
먼저, 데이터베이스 액세스를 위해 내부적으로 Ecto Repo가 사용된다는 것을 다시 한 번 확인할 수 있습니다.
또한 `Product.changeset/2`에 대한 호출을 눈치채셨을 것입니다.
앞서 변경 집합에 대해 이야기했는데, 이제 컨텍스트에서 변경 집합이 실제로 작동하는 것을 볼 수 있습니다.

`lib/hello/catalog/product.ex`에서 `Product` 스키마를 열면 즉시 익숙하게 보일 것입니다:

```perl Elixir
defmodule Hello.Catalog.Product do
  use Ecto.Schema
  import Ecto.Changeset

  schema "products" do
    field :description, :string
    field :price, :decimal
    field :title, :string
    field :views, :integer

    timestamps()
  end

  @doc false
  def changeset(product, attrs) do
    product
    |> cast(attrs, [:title, :description, :price, :views])
    |> validate_required([:title, :description, :price, :views])
  end
end
```

이것은 앞서 `mix phx.gen.schema`를 실행했을 때 보았던 것과 같지만, 여기서는 `changeset/2` 함수 위에 `@doc false`가 있습니다.
이는 이 함수가 공개적으로 호출 가능하지만 공용 컨텍스트 API의 일부가 아님을 알려줍니다.
변경 집합을 빌드하는 호출자는 컨텍스트 API를 통해 변경 집합을 빌드합니다.
예를 들어, `Catalog.create_product/1`은 `Product.changeset/2`를 호출하여 사용자 입력으로부터 변경 집합을 빌드합니다.
컨트롤러 액션과 같은 호출자는 `Product.changeset/2`에 직접 액세스하지 않습니다.
제품 변경 집합과의 모든 상호 작용은 공용 `Catalog` 컨텍스트를 통해 이루어집니다.

## Adding Catalog functions

지금까지 살펴본 것처럼 컨텍스트 모듈은 관련 기능을 노출하고 그룹화하는 전용 모듈입니다.
Phoenix는 `list_products` 및 `update_product`와 같은 일반 함수를 생성하지만, 이는 비즈니스 로직과 애플리케이션을 확장할 수 있는 기반 역할을 할 뿐입니다.
제품 페이지 조회 수를 추적하여 카탈로그의 기본 기능 중 하나를 추가해 보겠습니다.

모든 전자상거래 시스템에서 제품 페이지 조회 횟수를 추적하는 기능은 마케팅, 제안, 랭킹 등을 위해 필수적입니다.
기존의 `Catalog.update_product` 함수를 사용할 수도 있지만, `Catalog.update_product(product, %{views: product.views + 1})`와 같이 호출자가 카탈로그 시스템에 대해 너무 많은 것을 알고 있어야 한다는 단점이 있습니다.
경쟁 조건이 존재하는 이유를 알아보기 위해 가능한 이벤트 실행을 살펴봅시다:

직관적으로 다음과 같은 이벤트를 가정할 수 있습니다:

1.  User 1 loads the product page with count of 13
2.  User 1 saves the product page with count of 14
3.  User 2 loads the product page with count of 14
4.  User 2 saves the product page with count of 15

실제로는 이런 일이 발생할 수 있습니다:

1.  User 1 loads the product page with count of 13
2.  User 2 loads the product page with count of 13
3.  User 1 saves the product page with count of 14
4.  User 2 saves the product page with count of 14

여러 호출자가 오래된 뷰 값을 업데이트할 수 있으므로 경쟁 조건은 기존 테이블을 업데이트하는 신뢰할 수 없는 방법이 될 수 있습니다.
더 좋은 방법이 있습니다.

우리가 수행하고자 하는 작업을 설명하는 함수를 생각해 봅시다.
이 함수를 사용하는 방법은 다음과 같습니다:

```perl Elixir
product = Catalog.inc_page_views(product)
```

멋져 보입니다.
호출자는 이 함수가 수행하는 작업에 대해 혼동할 필요가 없으며, 경쟁 조건을 방지하기 위해 원자 연산으로 증분을 마무리할 수 있습니다.

카탈로그 컨텍스트(`lib/hello/catalog.ex`)를 열고 이 새 함수를 추가합니다:

```perl Elixir
  def inc_page_views(%Product{} = product) do
    {1, [%Product{views: views}]} =
      from(p in Product, where: p.id == ^product.id, select: [:views])
      |> Repo.update_all(inc: [views: 1])

    put_in(product.views, views)
  end
```

ID가 주어진 현재 제품을 가져오는 쿼리를 작성하여 `Repo.update_all`에 전달합니다.
Ecto의 `Repo.update_all`을 사용하면 데이터베이스에 대한 일괄 업데이트를 수행할 수 있으며, 조회 수 증가와 같이 값을 원자적으로 업데이트하는 데 적합합니다.
리포지토리 작업의 결과는 `select` 옵션으로 지정된 선택된 스키마 값과 함께 업데이트된 레코드 수를 반환합니다.
새 제품 보기를 받으면 `put_in(product.views, views)`을 사용하여 제품 구조체 내에 새 보기 수를 배치합니다.

컨텍스트 함수가 준비되었으므로 이제 제품 컨트롤러에서 이를 활용해 보겠습니다.
`lib/hello_web/controllers/product_controller.ex`에서 `show` 액션을 업데이트하여 새 함수를 호출합니다:

```perl Elixir
  def show(conn, %{"id" => id}) do
    product =
      id
      |> Catalog.get_product!()
      |> Catalog.inc_page_views()

    render(conn, :show, product: product)
  end
```

`show` 액션을 수정하여 가져온 제품을 업데이트된 제품을 반환하는 `Catalog.inc_page_views/1`로 파이프했습니다.
그런 다음 이전과 마찬가지로 템플릿을 렌더링했습니다.
한 번 해보겠습니다.
제품 페이지 중 하나를 몇 번 새로고침하고 조회 수가 증가하는 것을 확인합니다.

또한 ecto 디버그 로그에서 원자 업데이트가 작동하는 것을 확인할 수 있습니다:

```shell
[debug] QUERY OK source="products" db=0.5ms idle=834.5ms
UPDATE "products" AS p0 SET "views" = p0."views" + $1 WHERE (p0."id" = $2) RETURNING p0."views" [1, 1]
```

잘했어요 짝짝짝!

지금까지 살펴본 바와 같이 컨텍스트를 사용한 디자인은 애플리케이션을 성장시킬 수 있는 견고한 토대를 제공합니다.
시스템의 의도를 노출하는 개별적이고 잘 정의된 API를 사용하면 재사용 가능한 코드로 더 유지 관리하기 쉬운 애플리케이션을 작성할 수 있습니다.
이제 컨텍스트 API 확장을 시작하는 방법을 알았으니 컨텍스트 내에서 관계를 처리하는 방법을 살펴보겠습니다.

## In-context Relationships

기본 카탈로그 기능도 훌륭하지만 제품을 분류하여 한 단계 더 발전시켜 보겠습니다.
많은 전자상거래 솔루션에서는 제품을 패션, 전동 공구 등으로 표시하는 등 다양한 방식으로 제품을 분류할 수 있습니다.
제품과 카테고리 간의 일대일 관계로 시작하면 나중에 여러 카테고리를 지원해야 하는 경우 코드를 크게 변경해야 합니다.
제품당 하나의 카테고리를 추적하는 것으로 시작하지만 나중에 기능을 확장하면서 더 많은 카테고리를 쉽게 지원할 수 있는 카테고리 연결을 설정해 보겠습니다.

지금은 카테고리에는 텍스트 정보만 포함됩니다.
가장 먼저 해야 할 일은 애플리케이션에서 카테고리의 위치를 결정하는 것입니다.
저희는 제품 전시를 관리하는 `Catalog` 컨텍스트가 있습니다.
제품 분류는 여기에 자연스럽게 적합합니다.
또한 Phoenix는 기존 컨텍스트 내에서 코드를 생성할 수 있을 정도로 똑똑하기 때문에 컨텍스트에 새로운 리소스를 쉽게 추가할 수 있습니다.
프로젝트 루트에서 다음 명령을 실행하세요:

> Sometimes it may be tricky to determine if two resources belong to the same context or not.
> In those cases, prefer distinct contexts per resource and refactor later if necessary.
> Otherwise you can easily end up with large contexts of loosely related entities.
> Also keep in mind that the fact two resources are related does not necessarily mean they belong to the same context, otherwise you would quickly end up with one large context, as the majority of resources in an application are connected to each other.
> To sum it up: if you are unsure, you should prefer separate modules (contexts).

```shell
$ mix phx.gen.context Catalog Category categories \
title:string:unique

기존 컨텍스트에서 생성 중입니다.
...
계속 진행하시겠습니까? [Yn] y
* creating lib/hello/catalog/category.ex
* creating priv/repo/migrations/20210203192325_create_categories.exs
* injecting lib/hello/catalog.ex
* injecting test/hello/catalog_test.exs
* injecting test/support/fixtures/catalog_fixtures.ex

마이그레이션을 실행하여 리포지토리를 업데이트하는 것을 잊지 마세요:

    $ mix ecto.migrate
```

이번에는 웹 파일을 생성하지 않는다는 점을 제외하면 `mix phx.gen.context`를 사용했는데, 이는 `mix phx.gen.html`과 비슷합니다.
이미 제품 관리를 위한 컨트롤러와 템플릿이 있으므로 새 카테고리 기능을 기존 웹 양식과 제품 쇼 페이지에 통합할 수 있습니다.
이제 제품 스키마와 함께 새로운 `Category` 스키마가 `lib/hello/catalog/category.ex`에 있는 것을 볼 수 있으며, Phoenix는 카테고리 기능을 위해 기존 카탈로그 컨텍스트에 새로운 함수를 _injecting_ 한다고 알려주었습니다.
삽입된 함수는 제품 함수와 매우 유사하며, `create_category`, `list_categories` 등과 같은 새로운 함수가 포함되어 있습니다.
마이그레이션하기 전에 두 번째 코드 생성을 수행해야 합니다.
카테고리 스키마는 시스템에서 개별 카테고리를 표현하는 데는 훌륭하지만, 제품과 카테고리 간의 다대다 관계를 지원해야 합니다.
다행히도 ecto를 사용하면 조인 테이블을 사용하여 간단히 이 작업을 수행할 수 있으므로 이제 `ecto.gen.migration` 명령을 사용하여 생성해 보겠습니다:

```shell
mix ecto.gen.migration create_product_categories

* creating priv/repo/migrations/20210203192958_create_product_categories.exs
```

다음으로, 새 마이그레이션 파일을 열고 `change` 함수에 다음 코드를 추가합니다:

```perl Elixir

defmodule Hello.Repo.Migrations.CreateProductCategories do
  use Ecto.Migration

  def change do
    create table(:product_categories, primary_key: false) do
      add :product_id, references(:products, on_delete: :delete_all)
      add :category_id, references(:categories, on_delete: :delete_all)
    end

    create index(:product_categories, [:product_id])
    create unique_index(:product_categories, [:category_id, :product_id])
  end
end
```

조인 테이블에 기본 키가 필요하지 않으므로 `product_categories` 테이블을 만들고 `primary_key: false` 옵션을 사용했습니다.
다음으로 `:product_id` 및 `:category_id` 외래 키 필드를 정의하고, 연결된 제품 또는 카테고리가 삭제될 경우 데이터베이스가 조인 테이블 레코드를 정리하도록 `on_delete: :delete_all`을 전달했습니다.
데이터베이스 제약 조건을 사용하면 오류가 발생하기 쉬운 임시 애플리케이션 로직에 의존하지 않고 데이터베이스 수준에서 데이터 무결성을 강화할 수 있습니다.

다음으로, 외래 키에 대한 인덱스를 만들었는데, 그 중 하나는 고유 인덱스로서 제품이 중복된 카테고리를 가질 수 없도록 보장합니다.
`category_id`는 다중 열 인덱스의 가장 왼쪽 접두사에 있기 때문에 데이터베이스 최적화 프로그램에서 단일 열 인덱스가 반드시 필요한 것은 아닙니다.
반면에 중복 인덱스를 추가하면 쓰기 오버헤드만 추가됩니다.

마이그레이션이 준비되었으므로 이제 마이그레이션할 수 있습니다.

```shell
mix ecto.migrate

18:20:36.489 [info] == Running 20210222231834 Hello.Repo.Migrations.CreateCategories.change/0 forward

18:20:36.493 [info] create table categories

18:20:36.508 [info] create index categories_title_index

18:20:36.512 [info] == Migrated 20210222231834 in 0.0s

18:20:36.547 [info] == Running 20210222231930 Hello.Repo.Migrations.CreateProductCategories.change/0 forward

18:20:36.547 [info] create table product_categories

18:20:36.557 [info] create index product_categories_product_id_index

18:20:36.560 [info]  create index product_categories_category_id_product_id_index

18:20:36.562 [info] == Migrated 20210222231930 in 0.0s
```

이제 제품과 카테고리를 연결하기 위한 `Catalog.Product` 스키마와 조인 테이블이 생겼으므로 새 기능을 연결할 준비가 거의 완료되었습니다.
본격적으로 시작하기 전에 먼저 웹 UI에서 선택할 실제 카테고리가 필요합니다.
애플리케이션에 몇 가지 새로운 카테고리를 빠르게 시딩해 보겠습니다.
`priv/repo/seeds.exs`의 시드 파일에 다음 코드를 추가합니다:

```perl Elixir
for title <- ["Home Improvement", "Power Tools", "Gardening", "Books", "Education"] do
  {:ok, _} = Hello.Catalog.create_category(%{title: title})
end
```

카테고리 제목 목록을 열거하고 카탈로그 컨텍스트의 생성된 `create_category/1` 함수를 사용하여 새 레코드를 유지합니다.
`mix run`으로 시드를 실행할 수 있습니다:

```shell
mix run priv/repo/seeds.exs

[debug] QUERY OK db=3.1ms decode=1.1ms queue=0.7ms idle=2.2ms
INSERT INTO "categories" ("title","inserted_at","updated_at") VALUES ($1,$2,$3) RETURNING "id" ["Home Improvement", ~N[2021-02-03 19:39:53], ~N[2021-02-03 19:39:53]]
[debug] QUERY OK db=1.2ms queue=1.3ms idle=12.3ms
INSERT INTO "categories" ("title","inserted_at","updated_at") VALUES ($1,$2,$3) RETURNING "id" ["Power Tools", ~N[2021-02-03 19:39:53], ~N[2021-02-03 19:39:53]]
[debug] QUERY OK db=1.1ms queue=1.1ms idle=15.1ms
INSERT INTO "categories" ("title","inserted_at","updated_at") VALUES ($1,$2,$3) RETURNING "id" ["Gardening", ~N[2021-02-03 19:39:53], ~N[2021-02-03 19:39:53]]
[debug] QUERY OK db=2.4ms queue=1.0ms idle=17.6ms
INSERT INTO "categories" ("title","inserted_at","updated_at") VALUES ($1,$2,$3) RETURNING "id" ["Books", ~N[2021-02-03 19:39:53], ~N[2021-02-03 19:39:53]]
```

완벽합니다.
웹 레이어에서 카테고리를 통합하기 전에 컨텍스트에 제품과 카테고리를 연결하는 방법을 알려야 합니다.
먼저 `lib/hello/catalog/product.ex`를 열고 다음 연관을 추가합니다:

```diff
+ alias Hello.Catalog.Category

  schema "products" do
    field :description, :string
    field :price, :decimal
    field :title, :string
    field :views, :integer

+   many_to_many :categories, Category, join_through: "product_categories", on_replace: :delete

    timestamps()
  end

```

`Ecto.Schema`의 `many_to_many` 매크로를 사용하여 `"product_categories"` 조인 테이블을 통해 제품을 여러 범주에 연결하는 방법을 Ecto에 알렸습니다.
또한 `on_replace: :delete` 옵션을 사용하여 카테고리를 변경할 때 기존 조인 레코드가 삭제되어야 함을 선언했습니다.

스키마 연결이 설정되었으므로 이제 제품 양식에서 카테고리 선택을 구현할 수 있습니다.
그러기 위해서는 프론트엔드에서 카탈로그 ID의 사용자 입력을 다대다 연결로 변환해야 합니다.
다행히 Ecto를 사용하면 스키마가 설정되었으므로 이 작업이 매우 간편합니다.
카탈로그 컨텍스트를 열고 다음과 같이 변경합니다:

```diff
+ alias Hello.Catalog.Category

- def get_product!(id), do: Repo.get!(Product, id)
+ def get_product!(id) do
+   Product |> Repo.get!(id) |> Repo.preload(:categories)
+ end

  def create_product(attrs \\ %{}) do
    %Product{}
-   |> Product.changeset(attrs)
+   |> change_product(attrs)
    |> Repo.insert()
  end

  def update_product(%Product{} = product, attrs) do
    product
-   |> Product.changeset(attrs)
+   |> change_product(attrs)
    |> Repo.update()
  end

  def change_product(%Product{} = product, attrs \\ %{}) do
-   Product.changeset(product, attrs)
+   categories = list_categories_by_id(attrs["category_ids"])

+   product
+   |> Repo.preload(:categories)
+   |> Product.changeset(attrs)
+   |> Ecto.Changeset.put_assoc(:categories, categories)
  end

+ def list_categories_by_id(nil), do: []
+ def list_categories_by_id(category_ids) do
+   Repo.all(from c in Category, where: c.id in ^category_ids)
+ end
```

먼저, 제품을 가져올 때 카테고리를 미리 로드하기 위해 `Repo.preload`를 추가했습니다.
이렇게 하면 컨트롤러, 템플릿 및 카테고리 정보를 사용하려는 다른 모든 곳에서 `product.categories`를 참조할 수 있습니다.
다음으로, 기존 `change_product` 함수를 호출하여 변경 집합을 생성하도록 `create_product` 및 `update_product` 함수를 수정했습니다.
`change_product` 내에 `"category_ids"` 속성이 있는 경우 모든 카테고리를 찾을 수 있는 조회를 추가했습니다.
그런 다음 카테고리를 미리 로드하고 `Ecto.Changeset.put_assoc`을 호출하여 가져온 카테고리를 변경 집합에 배치했습니다.
마지막으로 `list_categories_by_id/1` 함수를 구현하여 카테고리 ID와 일치하는 카테고리를 쿼리하거나 `"category_ids"` 속성이 없는 경우 빈 목록을 반환합니다.
이제 `create_product` 및 `update_product` 함수는 리포지토리에 대한 삽입 또는 업데이트를 시도하면 카테고리 연결이 모두 준비된 변경 집합을 받습니다.

다음으로, 제품 양식에 카테고리 입력을 추가하여 새 기능을 웹에 노출해 보겠습니다.
양식 템플릿을 깔끔하게 유지하기 위해 제품에 대한 카테고리 선택 입력을 렌더링하는 세부 사항을 마무리하는 새 함수를 작성해 보겠습니다.
`lib/hello_web/controllers/product_html.ex`에서 `ProductHTML` 뷰를 열고 이 값을 입력합니다:

```perl Elixir
  def category_opts(changeset) do
    existing_ids =
      changeset
      |> Ecto.Changeset.get_change(:categories, [])
      |> Enum.map(& &1.data.id)

    for cat <- Hello.Catalog.list_categories(),
        do: [key: cat.title, value: cat.id, selected: cat.id in existing_ids]
  end
```

곧 추가할 다중 선택 태그에 대한 선택 옵션을 생성하는 새로운 `category_opts/1` 함수를 추가했습니다.
변경 집합에서 기존 카테고리 ID를 계산한 다음 입력 태그의 선택 옵션을 생성할 때 해당 값을 사용했습니다.
이를 위해 모든 카테고리를 열거하고 적절한 'key', 'value' 및 'selected' 값을 반환하는 방식으로 수행했습니다.
변경 집합의 카테고리 ID에서 해당 카테고리 ID가 발견되면 옵션을 선택된 것으로 표시했습니다.

`category_opts` 함수가 준비되었으므로 `lib/hello_web/controllers/product_html/product_form.html.heex`를 열고 추가할 수 있습니다:

```diff
  ...
  <.input field={f[:views]} type="number" label="Views" />

+ <.input field={f[:category_ids]} type="select" multiple={true} options={category_opts(@changeset)} />

  <:actions>
    <.button>Save Product</.button>
  </:actions>
```

저장 버튼 위에 `category_select`를 추가했습니다.
이제 사용해 봅시다.
다음으로 제품 표시 템플릿에서 제품의 카테고리를 표시해 보겠습니다.
`lib/hello_web/controllers/product_html/show.html.heex`의 목록에 다음 코드를 추가합니다:

```perl HEEx
<.list>
  ...
+ <:item title="Categories">
+   <%= for cat <- @product.categories do %>
+     <%= cat.title %>
+     <br/>
+   <% end %>
+ </:item>
</.list>
```

이제 `mix phx.server`로 서버를 시작하고 [http://localhost:4000/products/new](http://localhost:4000/products/new)를 방문하면 새 카테고리 다중 선택 입력란이 표시됩니다.
유효한 제품 세부 정보를 입력하고 카테고리를 한두 개 선택한 다음 저장을 클릭합니다.

```shell
제목: 엘릭서 플래시카드
설명: 엘릭서 프로그래밍 언어용 플래시 카드 세트입니다.
가격: 5.000000
조회 수 0
카테고리:
교육
도서
```

아직 보기에는 별거 아니지만 작동합니다! 데이터베이스에 의해 강화된 데이터 무결성을 갖춘 컨텍스트 내 관계를 추가했습니다.
나쁘지 않네요.
계속 구축해 봅시다!

## Cross-context dependencies

이제 제품 카탈로그 기능의 시작을 알았으니 애플리케이션의 다른 주요 기능인 카탈로그에서 제품을 카트화하는 작업을 시작해 보겠습니다.
사용자의 카트에 추가된 제품을 제대로 추적하려면 카트 시점의 가격과 같은 특정 시점의 제품 정보와 함께 이 정보를 유지할 새로운 공간이 필요합니다.
이는 향후 제품 가격 변화를 감지하기 위해 필요합니다.
무엇을 구축해야 하는지 알았으니 이제 애플리케이션에서 카트 기능을 어디에 배치할지 결정해야 합니다.

한 걸음 물러서서 애플리케이션의 고립성에 대해 생각해 보면 카탈로그의 제품 전시와 사용자의 카트 관리는 분명히 다릅니다.
제품 카탈로그는 장바구니 시스템의 규칙을 신경 쓰지 않아야 하며, 그 반대의 경우도 마찬가지입니다.
따라서 새로운 카트 책임을 처리하기 위한 별도의 컨텍스트가 분명히 필요합니다.
이를 `ShoppingCart`라고 부르겠습니다.

기본적인 카트 업무를 처리할 `ShoppingCart` 컨텍스트를 만들어 보겠습니다.
코드를 작성하기 전에 다음과 같은 기능 요구 사항이 있다고 가정해 보겠습니다:

1.  Add products to a user's cart from the product show page
2.  Store point-in-time product price information at time of carting
3.  Store and update quantities in cart
4.  Calculate and display sum of cart prices

설명을 보면 사용자의 장바구니를 저장하기 위한 `Cart` 리소스와 장바구니의 제품을 추적하기 위한 `CartItem`이 필요하다는 것을 알 수 있습니다.
계획이 정해졌으니 이제 작업을 시작하겠습니다.
다음 명령을 실행하여 새 컨텍스트를 생성합니다:

```shell
$ mix phx.gen.context ShoppingCart Cart carts user_uuid:uuid:unique

* creating lib/hello/shopping_cart/cart.ex
* creating priv/repo/migrations/20210205203128_create_carts.exs
* creating lib/hello/shopping_cart.ex
* injecting lib/hello/shopping_cart.ex
* creating test/hello/shopping_cart_test.exs
* injecting test/hello/shopping_cart_test.exs
* creating test/support/fixtures/shopping_cart_fixtures.ex
* injecting test/support/fixtures/shopping_cart_fixtures.ex

생성된 데이터베이스 열 중 일부는 고유합니다.
다음 고정 함수(들)에 대해 고유한 구현을 제공하세요.
test/support/fixtures/shopping_cart_fixtures.ex:

    def unique_cart_user_uuid do
      raise "implement the logic to generate a unique cart user_uuid"
    end

마이그레이션을 실행하여 리포지토리를 업데이트하는 것을 잊지 마세요:

    $ mix ecto.migrate
```

사용자를 카트 항목이 있는 카트에 연결하기 위해 새로운 `ShoppingCart.Cart` 스키마가 포함된 새 컨텍스트 `ShoppingCart`를 생성했습니다.
아직 실제 사용자가 없으므로 지금은 잠시 후 플러그 세션에 추가할 익명 사용자 UUID로 카트를 추적할 것입니다.
카트가 준비되었으니 이제 카트 항목을 생성해 보겠습니다:

```shell
$ mix phx.gen.context ShoppingCart CartItem cart_items \
cart_id:references:carts product_id:references:products \
price_when_carted:decimal quantity:integer

기존 컨텍스트에서 생성하고 있습니다.
...
계속 진행하시겠습니까? [Yn] y
* creating lib/hello/shopping_cart/cart_item.ex
* creating priv/repo/migrations/20210205213410_create_cart_items.exs
* injecting lib/hello/shopping_cart.ex
* injecting test/hello/shopping_cart_test.exs
* injecting test/support/fixtures/shopping_cart_fixtures.ex

마이그레이션을 실행하여 리포지토리를 업데이트하는 것을 잊지 마세요:

    $ mix ecto.migrate

```

`ShoppingCart` 내부에 '카트 항목'이라는 새 리소스를 생성했습니다.
이 스키마와 테이블에는 카트 및 제품에 대한 참조와 함께 카트에 항목을 추가한 시점의 가격 및 사용자가 구매하고자 하는 수량이 포함됩니다.
생성된 마이그레이션 파일을 `priv/repo/migrations/*_create_cart_items.ex`에서 수정해 보겠습니다:

```perl Elixir
    create table(:cart_items) do
-     add :price_when_carted, :decimal
+     add :price_when_carted, :decimal, precision: 15, scale: 6, null: false
      add :quantity, :integer
-     add :cart_id, references(:carts, on_delete: :nothing)
+     add :cart_id, references(:carts, on_delete: :delete_all)
-     add :product_id, references(:products, on_delete: :nothing)
+     add :product_id, references(:products, on_delete: :delete_all)

      timestamps()
    end

-   create index(:cart_items, [:cart_id])
    create index(:cart_items, [:product_id])
+   create unique_index(:cart_items, [:cart_id, :product_id])
```

데이터 무결성을 강화하기 위해 `:delete_all` 전략을 다시 사용했습니다.
이렇게 하면 애플리케이션에서 카트나 제품이 삭제될 때 `ShoppingCart` 또는 `Catalog` 컨텍스트의 애플리케이션 코드에 의존하여 레코드 정리를 걱정할 필요가 없습니다.
이렇게 하면 애플리케이션 코드가 분리되어 있고 데이터 무결성 적용이 데이터베이스의 원래 위치에 유지됩니다.
또한 고유한 제약 조건을 추가하여 중복된 제품을 카트에 추가할 수 없도록 했습니다.
`product_categories` 테이블과 마찬가지로 다중 열 인덱스를 사용하면 가장 왼쪽 필드(`cart-id`)에 대한 별도의 인덱스를 제거할 수 있습니다.
데이터베이스 테이블이 준비되었으므로 이제 마이그레이션할 수 있습니다:

```shell
mix ecto.migrate

16:59:51.941 [info] == Running 20210205203342 Hello.Repo.Migrations.CreateCarts.change/0 forward

16:59:51.945 [info] create table carts

16:59:51.949 [info] create index carts_user_uuid_index

16:59:51.952 [info] == Migrated 20210205203342 in 0.0s

16:59:51.988 [info] == Running 20210205213410 Hello.Repo.Migrations.CreateCartItems.change/0 forward

16:59:51.988 [info] create table cart_items

16:59:51.998 [info] create index cart_items_cart_id_index

16:59:52.000 [info] create index cart_items_product_id_index

16:59:52.001 [info] create index cart_items_cart_id_product_id_index

16:59:52.002 [info] == Migrated 20210205213410 in 0.0s
```

데이터베이스에 새로운 `carts` 및 `cart_items` 테이블을 사용할 준비가 되었지만 이제 이를 애플리케이션 코드에 다시 매핑해야 합니다.
서로 다른 테이블에서 데이터베이스 외래 키를 혼합하는 방법과 이것이 격리되고 그룹화된 기능의 컨텍스트 패턴과 어떤 관련이 있는지 궁금할 것입니다.
지금부터 접근 방식과 그 장단점에 대해 논의해 보겠습니다.

### Cross-context data

지금까지 애플리케이션의 두 가지 주요 컨텍스트를 서로 분리하는 작업을 훌륭하게 수행했지만 이제 처리해야 할 종속성이 생겼습니다.

우리의 `Catalog.Product` 리소스는 카탈로그 내에서 제품을 나타내는 역할을 하지만, 궁극적으로 카트에 항목이 존재하려면 카탈로그의 제품이 있어야 합니다.
이를 감안할 때 `ShoppingCart` 컨텍스트는 `Catalog` 컨텍스트에 대한 데이터 종속성을 갖게 됩니다.
이를 염두에 두고 두 가지 옵션이 있습니다.
하나는 `Catalog` 컨텍스트에 API를 노출하여 `ShoppingCart` 시스템에서 사용할 제품 데이터를 효율적으로 가져와서 수동으로 연결할 수 있도록 하는 것입니다.
또는 데이터베이스 조인을 사용하여 종속 데이터를 가져올 수 있습니다.
두 가지 방법 모두 장단점과 애플리케이션 규모를 고려할 때 유효한 옵션이지만, 데이터 종속성이 심한 경우 데이터베이스에서 데이터를 조인하는 것이 대규모 애플리케이션에 적합하며 여기서는 이 방법을 사용하겠습니다.

이제 데이터 종속성이 어디에 있는지 알았으니 스키마 연결을 추가하여 장바구니에 있는 항목을 제품에 연결해 보겠습니다.
먼저 `lib/hello/shopping_cart/cart.ex`에서 카트 스키마를 간단히 변경하여 카트를 해당 항목에 연결해 보겠습니다:

```perl Elixir
  schema "carts" do
    field :user_uuid, Ecto.UUID

+   has_many :items, Hello.ShoppingCart.CartItem

    timestamps()
  end
```

이제 카트가 장바구니에 넣은 품목과 연결되었으므로 `lib/hello/shopping_cart/cart_item.ex`에서 카트 품목 연결을 설정해 보겠습니다:

```perl Elixir
  schema "cart_items" do
    field :price_when_carted, :decimal
    field :quantity, :integer
-   field :cart_id, :id
-   field :product_id, :id

+   belongs_to :cart, Hello.ShoppingCart.Cart
+   belongs_to :product, Hello.Catalog.Product

    timestamps()
  end

  @doc false
  def changeset(cart_item, attrs) do
    cart_item
    |> cast(attrs, [:price_when_carted, :quantity])
    |> validate_required([:price_when_carted, :quantity])
+   |> validate_number(:quantity, greater_than_or_equal_to: 0, less_than: 100)
  end
```

먼저, `cart_id` 필드를 `ShoppingCart.Cart` 스키마를 가리키는 표준 `belongs_to`로 대체했습니다.
다음으로, 첫 번째 교차 컨텍스트 데이터 종속성을 `Catalog.Product` 스키마에 대한 `belongs_to`로 추가하여 `product_id` 필드를 대체했습니다.
여기서 의도적으로 데이터 경계를 연결한 이유는 시스템에서 제품을 참조하는 데 필요한 최소한의 지식만 있으면 격리된 컨텍스트 API를 정확히 제공할 수 있기 때문입니다.
다음으로, 변경 집합에 새로운 유효성 검사를 추가했습니다.
`validate_number/3`을 사용하여 사용자 입력에 의해 제공된 수량이 0에서 100 사이인지 확인합니다.

스키마가 준비되었으므로 새로운 데이터 구조와 `ShoppingCart` 컨텍스트 API를 웹 대면 기능에 통합할 수 있습니다.

### Adding Shopping Cart functions

앞서 언급했듯이 컨텍스트 생성기는 애플리케이션의 시작점일 뿐입니다.
우리는 컨텍스트의 목표를 달성하기 위해 잘 명명된 목적에 맞는 함수를 작성할 수 있고 작성해야 합니다.
구현해야 할 몇 가지 새로운 기능이 있습니다.
먼저, 애플리케이션의 모든 사용자에게 아직 카트가 없는 경우 카트가 부여되도록 해야 합니다.
그런 다음 사용자가 카트에 항목을 추가하고, 항목 수량을 업데이트하고, 카트 총액을 계산할 수 있도록 허용할 수 있습니다.
이제 시작해보겠습니다!

지금은 실제 사용자 인증 시스템에 초점을 맞추지 않겠지만, 이 과정이 끝나면 여기에서 작성한 내용을 자연스럽게 통합할 수 있을 것입니다.
현재 사용자 세션을 시뮬레이션하려면 `lib/hello_web/router.ex`를 열고 이 값을 입력합니다:

```diff Elixir
  pipeline :browser do
    plug :accepts, ["html"]
    plug :fetch_session
    plug :fetch_live_flash
    plug :put_root_layout, html: {HelloWeb.LayoutView, :root}
    plug :protect_from_forgery
    plug :put_secure_browser_headers
+   plug :fetch_current_user
+   plug :fetch_current_cart
  end

+ defp fetch_current_user(conn, _) do
+   if user_uuid = get_session(conn, :current_uuid) do
+     assign(conn, :current_uuid, user_uuid)
+   else
+     new_uuid = Ecto.UUID.generate()
+
+     conn
+     |> assign(:current_uuid, new_uuid)
+     |> put_session(:current_uuid, new_uuid)
+   end
+ end

+ alias Hello.ShoppingCart
+
+ defp fetch_current_cart(conn, _opts) do
+   if cart = ShoppingCart.get_cart_by_user_uuid(conn.assigns.current_uuid) do
+     assign(conn, :cart, cart)
+   else
+     {:ok, new_cart} = ShoppingCart.create_cart(conn.assigns.current_uuid)
+     assign(conn, :cart, new_cart)
+   end
+ end
```

모든 브라우저 기반 요청에서 실행되도록 브라우저 파이프라인에 새로운 `:fetch_current_user` 및 `:fetch_current_cart` 플러그를 추가했습니다.
다음으로, 세션에서 이전에 추가한 사용자 UUID가 있는지 간단히 확인하는 `fetch_current_user` 플러그를 구현했습니다.
이를 찾으면 연결에 `current_uuid` 할당을 추가하면 완료됩니다.
아직 이 방문자를 식별하지 못한 경우 `Ecto.UUID.generate()`로 고유 UUID를 생성한 다음, 이 값을 새 세션 값과 함께 `current_uuid` 할당에 배치하여 향후 요청에서 이 방문자를 식별합니다.
임의의 고유 ID는 사용자를 대표하기에는 부족하지만, 여러 요청에서 방문자를 추적하고 식별하는 데는 충분하므로 현재로서는 이 정도면 충분합니다.
나중에 애플리케이션이 더 완성되면 완전한 사용자 인증 솔루션으로 마이그레이션할 수 있습니다.
현재 사용자가 보장된 상태에서 사용자 UUID에 대한 장바구니를 찾거나 현재 사용자에 대한 장바구니를 생성하고 그 결과를 연결 할당에 할당하는 `fetch_current_cart` 플러그를 구현했습니다.
`ShoppingCart.get_cart_by_user_uuid/1`을 구현하고 카트 생성 함수를 수정하여 UUID를 받도록 해야 하지만 먼저 경로를 추가해 보겠습니다.

카트 보기, 수량 업데이트, 결제 프로세스 시작과 같은 카트 작업을 처리하기 위한 카트 컨트롤러와 카트에 개별 품목을 추가 및 제거하기 위한 카트 품목 컨트롤러를 구현해야 합니다.
`lib/hello_web/router.ex`에서 라우터에 다음 경로를 추가합니다:

```diff
  scope "/", HelloWeb do
    pipe_through :browser

    get "/", PageController, :index
    resources "/products", ProductController

+   resources "/cart_items", CartItemController, only: [:create, :delete]

+   get "/cart", CartController, :show
+   put "/cart", CartController, :update
  end
```

카트 아이템 컨트롤러`에 대한 `resources` 선언을 추가하여 개별 카트 아이템의 추가 및 제거를 위한 생성 및 삭제 작업의 경로를 연결합니다.
다음으로 `CartController`를 가리키는 두 개의 새 경로를 추가했습니다.
첫 번째 경로인 GET 요청은 카트 콘텐츠를 표시하기 위해 show 액션에 매핑됩니다.
두 번째 경로인 PUT 요청은 카트 수량을 업데이트하기 위한 양식 제출을 처리합니다.

경로가 준비되었으므로 제품 표시 페이지에서 카트에 상품을 추가하는 기능을 추가해 보겠습니다.
`lib/hello_web/controllers/cart_item_controller.ex`에 새 파일을 생성하고 이 파일을 입력합니다:

```perl Elixir
defmodule HelloWeb.CartItemController do
  use HelloWeb, :controller

  alias Hello.ShoppingCart

  def create(conn, %{"product_id" => product_id}) do
    case ShoppingCart.add_item_to_cart(conn.assigns.cart, product_id) do
      {:ok, _item} ->
        conn
        |> put_flash(:info, "Item added to your cart")
        |> redirect(to: ~p"/cart")

      {:error, _changeset} ->
        conn
        |> put_flash(:error, "There was an error adding the item to your cart")
        |> redirect(to: ~p"/cart")
    end
  end

  def delete(conn, %{"id" => product_id}) do
    {:ok, _cart} = ShoppingCart.remove_item_from_cart(conn.assigns.cart, product_id)
    redirect(conn, to: ~p"/cart")
  end
end
```

라우터에서 선언한 생성 및 삭제 액션을 사용하여 새로운 `CartItemController`를 정의했습니다.
`create`의 경우, 잠시 후에 구현할 `ShoppingCart.add_item_to_cart/2` 함수를 호출합니다.
성공하면 플래시 성공 메시지를 표시하고 카트 표시 페이지로 리디렉션하고, 그렇지 않으면 플래시 오류 메시지를 표시하고 카트 표시 페이지로 리디렉션합니다.
`delete`의 경우 `ShoppingCart` 컨텍스트에서 구현할 `remove_item_from_cart` 함수를 호출한 다음 카트 표시 페이지로 다시 리디렉션합니다.
이 두 가지 쇼핑 카트 함수는 아직 구현하지 않았지만 이름에서 그 의도를 알 수 있습니다: 장바구니에 항목 추가`와 `카트에서 항목 제거`는 여기서 수행하려는 작업이 무엇인지 명확하게 보여줍니다.
또한 모든 구현 세부 사항을 한꺼번에 생각하지 않고도 웹 계층과 컨텍스트 API를 지정할 수 있습니다.

이제 `lib/hello/shopping_cart.ex`에서 `ShoppingCart` 컨텍스트 API를 위한 새로운 인터페이스를 구현해 보겠습니다:

```perl Elixir
+  alias Hello.Catalog
-  alias Hello.ShoppingCart.Cart
+  alias Hello.ShoppingCart.{Cart, CartItem}

+  def get_cart_by_user_uuid(user_uuid) do
+    Repo.one(
+      from(c in Cart,
+        where: c.user_uuid == ^user_uuid,
+        left_join: i in assoc(c, :items),
+        left_join: p in assoc(i, :product),
+        order_by: [asc: i.inserted_at],
+        preload: [items: {i, product: p}]
+      )
+    )
+  end

- def create_cart(attrs \\ %{}) do
-   %Cart{}
-   |> Cart.changeset(attrs)
+ def create_cart(user_uuid) do
+   %Cart{user_uuid: user_uuid}
+   |> Cart.changeset(%{})
    |> Repo.insert()
+   |> case do
+     {:ok, cart} -> {:ok, reload_cart(cart)}
+     {:error, changeset} -> {:error, changeset}
+   end
  end

+  defp reload_cart(%Cart{} = cart), do: get_cart_by_user_uuid(cart.user_uuid)
+
+  def add_item_to_cart(%Cart{} = cart, product_id) do
+    product = Catalog.get_product!(product_id)
+
+    %CartItem{quantity: 1, price_when_carted: product.price}
+    |> CartItem.changeset(%{})
+    |> Ecto.Changeset.put_assoc(:cart, cart)
+    |> Ecto.Changeset.put_assoc(:product, product)
+    |> Repo.insert(
+      on_conflict: [inc: [quantity: 1]],
+      conflict_target: [:cart_id, :product_id]
+    )
+  end
+
+  def remove_item_from_cart(%Cart{} = cart, product_id) do
+    {1, _} =
+      Repo.delete_all(
+        from(i in CartItem,
+          where: i.cart_id == ^cart.id,
+          where: i.product_id == ^product_id
+        )
+      )
+
+    {:ok, reload_cart(cart)}
+  end
```

먼저 카트를 가져오고 카트 항목과 해당 제품을 결합하여 전체 카트가 미리 로드된 모든 데이터로 채워지도록 하는 `get_cart_by_user_uuid/1`을 구현했습니다.
다음으로, `create_cart` 함수를 수정하여 속성 대신 사용자 UUID를 받아들이고 이를 `user_uuid` 필드를 채우는 데 사용했습니다.
삽입에 성공하면 비공개 `reload_cart/1` 함수를 호출하여 카트 콘텐츠를 다시 로드하고, 이 함수는 `get_cart_by_user_uuid/1`을 호출하여 데이터를 다시 가져옵니다.

다음으로, 카트 구조체와 제품 ID를 받는 새로운 `add_item_to_cart/2` 함수를 작성했습니다.
카탈로그.get_product!/1`로 제품을 가져와서 필요한 경우 컨텍스트가 다른 컨텍스트를 자연스럽게 호출할 수 있는 방법을 보여줍니다.
제품을 인수로 받도록 선택해도 비슷한 결과를 얻을 수 있습니다.
그런 다음 저장소에 대한 업서트 작업을 사용하여 데이터베이스에 새 카트 항목을 삽입하거나 카트에 이미 있는 경우 수량을 하나씩 늘렸습니다.
이 작업은 `on_conflict` 및 `conflict_target` 옵션을 통해 수행되며, 이는 삽입 충돌을 처리하는 방법을 레포에 알려줍니다.

마지막으로, 제품 ID와 일치하는 카트 내 카트 항목을 삭제하는 쿼리와 함께 `Repo.delete_all` 호출을 실행하는 `remove_item_from_cart/2`를 구현했습니다.
마지막으로 `reload_cart/1`을 호출하여 카트 콘텐츠를 다시 로드합니다.

새로운 카트 함수가 적용되었으므로 이제 제품 카탈로그 쇼 페이지에 '카트에 추가' 버튼을 노출할 수 있습니다.
`lib/hello_web/controllers/product_html/show.html.heex`에서 템플릿을 열고 다음과 같이 변경합니다:

```diff
...
     <.link href={~p"/products/#{@product}/edit"}>
       <.button>Edit product</.button>
     </.link>
+    <.link href={~p"/cart_items?product_id=#{@product.id}"} method="post">
+      <.button>Add to cart</.button>
+    </.link>
...
```

`Phoenix.Component`의 `link` 함수 컴포넌트는 기본 GET 요청 대신 클릭 시 HTTP 동사를 발행하는 `:method` 속성을 허용합니다.
이 링크를 사용하면 "장바구니에 추가" 링크가 POST 요청을 발행하고, 이 요청은 라우터에서 정의한 경로와 일치하여 `CartItemController.create/2` 함수로 디스패치됩니다.

한 번 해보겠습니다.
`mix phx.server`로 서버를 시작하고 제품 페이지를 방문합니다.
장바구니에 추가 링크를 클릭하면 콘솔에 다음과 같은 로그가 포함된 오류 페이지가 표시됩니다:

```shell
[info] POST /cart_items
[debug] Processing with HelloWeb.CartItemController.create/2
  Parameters: %{"_method" => "post", "product_id" => "1", ...}
  Pipelines: [:browser]
INSERT INTO "cart_items" ...
[info] Sent 302 in 24ms
[info] GET /cart
[debug] Processing with HelloWeb.CartController.show/2
  Parameters: %{}
  Pipelines: [:browser]
[debug] QUERY OK source="carts" db=1.9ms idle=1798.5ms

[error] #PID<0.856.0> running HelloWeb.Endpoint (connection #PID<0.841.0>, stream id 5) terminated
Server: localhost:4000 (http)
Request: GET /cart
** (exit) an exception was raised:
    ** (UndefinedFunctionError) function HelloWeb.CartController.init/1 is undefined
       (module HelloWeb.CartController is not available)
       ...
```

작동 중입니다! 일종의.
로그를 따라가 보면 `/cart_items` 경로에 대한 POST를 확인할 수 있습니다.
다음으로, `ShoppingCart.add_item_to_cart` 함수가 `cart_items` 테이블에 행을 성공적으로 삽입한 다음 `/cart`로 리디렉션한 것을 볼 수 있습니다.
오류가 발생하기 전에 `carts` 테이블에 대한 쿼리도 표시되며, 이는 현재 사용자의 카트를 가져오고 있음을 의미합니다.
지금까지는 괜찮습니다.
`CartItem` 컨트롤러와 새로운 `ShoppingCart` 컨텍스트 함수가 제대로 작동하고 있지만, 라우터가 존재하지 않는 장바구니 컨트롤러로 디스패치하려고 시도할 때 구현되지 않은 다음 기능에 부딪혔습니다.
사용자 카트를 표시하고 관리하기 위한 카트 컨트롤러, 뷰, 템플릿을 만들어 보겠습니다.

`lib/hello_web/controllers/cart_controller.ex`에 새 파일을 생성하고 이를 입력합니다:

```perl Elixir
defmodule HelloWeb.CartController do
  use HelloWeb, :controller

  alias Hello.ShoppingCart

  def show(conn, _params) do
    render(conn, :show, changeset: ShoppingCart.change_cart(conn.assigns.cart))
  end
end
```

`get "/cart"` 경로를 처리할 새 카트 컨트롤러를 정의했습니다.
카트를 표시하기 위해 잠시 후에 생성할 `"show.html"` 템플릿을 렌더링합니다.
수량 업데이트를 통해 카트 항목을 변경할 수 있도록 허용해야 하므로 바로 카트 변경 집합이 필요하다는 것을 알고 있습니다.
다행히 컨텍스트 생성기에는 `ShoppingCart.change_cart/1` 함수가 포함되어 있으므로 이 함수를 사용하겠습니다.
라우터에서 정의한 `fetch_current_cart` 플러그 덕분에 이미 연결 할당에 있는 카트 구조체를 전달합니다.

다음으로 뷰와 템플릿을 구현할 수 있습니다.
`lib/hello_web/controllers/cart_html.ex`에 다음 내용으로 새 뷰 파일을 생성합니다:

```perl Elixir
defmodule HelloWeb.CartHTML do
  use HelloWeb, :html

  alias Hello.ShoppingCart

  embed_templates "cart_html/*"

  def currency_to_str(%Decimal{} = val), do: "$#{Decimal.round(val, 2)}"
end
```

`show.html` 템플릿을 렌더링하는 보기를 만들고 `ShoppingCart` 컨텍스트에 별칭을 지정하여 템플릿의 범위에 포함되도록 했습니다.
제품 품목 가격, 카트 총액 등과 같은 카트 가격을 표시해야 하므로 10진수 구조체를 가져와서 표시를 위해 적절하게 반올림하고 USD 달러 기호를 추가하는 `currency_to_str/1`을 정의했습니다.

다음으로 `lib/hello_web/controllers/cart_html/show.html.heex`에서 템플릿을 만들 수 있습니다:

```perl HEEx
<%= if @cart.items == [] do %>
  <.header>
    My Cart
    <:subtitle>Your cart is empty</:subtitle>
  </.header>
<% else %>
  <.header>
    My Cart
  </.header>

  <.simple_form :let={f} for={@changeset} action={~p"/cart"}>
    <.inputs_for :let={item_form} field={f[:items]}>
  <% item = item_form.data %>
      <.input field={item_form[:quantity]} type="number" label={item.product.title} />
      <%= currency_to_str(ShoppingCart.total_item_price(item)) %>
    </.inputs_for>
    <:actions>
      <.button>Update cart</.button>
    </:actions>
  </.simple_form>

  <b>Total</b>: <%= currency_to_str(ShoppingCart.total_cart_price(@cart)) %>
<% end %>

<.back navigate={~p"/products"}>Back to products</.back>
```

먼저 미리 로드된 `cart.items`이 비어 있는 경우 빈 카트 메시지를 표시합니다.
항목이 있는 경우 `HelloWeb.CoreComponents`에서 제공하는 `simple_form` 컴포넌트를 사용하여 `CartController.show/2` 액션에서 할당된 카트 변경 집합을 가져와 카트 컨트롤러 `update/2` 액션에 매핑되는 폼을 생성합니다.
폼 내에서 [`inputs_for`](`Phoenix.Component.inputs_for/1`) 컴포넌트를 사용하여 중첩된 카트 항목에 대한 입력을 렌더링합니다.
이렇게 하면 양식이 제출될 때 항목 입력을 다시 매핑할 수 있습니다.
다음으로 품목 수량에 대한 숫자 입력을 표시하고 제품 제목으로 레이블을 지정합니다.
품목 가격을 문자열로 변환하여 품목 양식을 완성합니다.
아직 `ShoppingCart.total_item_price/1` 함수를 작성하지는 않았지만, 컨텍스트에 맞는 명확하고 설명적인 공용 인터페이스라는 아이디어를 다시 한 번 사용했습니다.
모든 카트 항목에 대한 입력을 렌더링한 후 전체 카트의 총 가격과 함께 '카트 업데이트' 제출 버튼을 표시합니다.
이는 곧 구현할 또 다른 새로운 `ShoppingCart.total_cart_price/1` 함수를 통해 이루어집니다.
마지막으로 제품 페이지로 돌아가는 `back` 컴포넌트를 추가했습니다.

장바구니 페이지를 사용해 볼 준비가 거의 다 되었지만 먼저 새로운 통화 계산 함수를 구현해야 합니다.
`lib/hello/shopping_cart.ex`에서 장바구니 컨텍스트를 열고 이 새 함수를 추가합니다:

```perl Elixir
  def total_item_price(%CartItem{} = item) do
    Decimal.mult(item.product.price, item.quantity)
  end

  def total_cart_price(%Cart{} = cart) do
    Enum.reduce(cart.items, 0, fn item, acc ->
      item
      |> total_item_price()
      |> Decimal.add(acc)
    end)
  end
```

총 가격을 계산하는 함수는 `%CartItem{}` 구조체를 받아들이는 `total_item_price/1`을 구현했습니다.
총 가격을 계산하려면 미리 로드된 제품의 가격에 품목의 수량을 곱하기만 하면 됩니다.
십진수 통화 구조체에서 적절한 정밀도로 곱하기 위해 `Decimal.mult/2`를 사용했습니다.
마찬가지로 총 카트 가격을 계산하기 위해 카트를 받아 카트에 미리 로드된 품목의 제품 가격을 합산하는 `total_cart_price/1` 함수를 구현했습니다.
다시 `Decimal` 함수를 사용하여 십진수 구조를 더합니다.

이제 가격 합계를 계산할 수 있게 되었으니 직접 사용해 보겠습니다! [http://localhost:4000/cart](http://localhost:4000/cart)를 방문하면 장바구니에 첫 번째 품목이 이미 표시되어 있을 것입니다.
동일한 제품으로 돌아가서 "장바구니에 추가"를 클릭하면 업서트가 작동하는 것을 볼 수 있습니다.
이제 수량이 2개로 표시됩니다.
수고하셨습니다!

장바구니 페이지가 거의 완성되었지만 양식을 제출하면 또 다른 오류가 발생합니다.

```shell
Request: POST /cart
** (exit) an exception was raised:
    ** (UndefinedFunctionError) function HelloWeb.CartController.update/2 is undefined or private
```

이제 `lib/hello_web/controllers/cart_controller.ex`의 `CartController`로 돌아가서 업데이트 작업을 구현해 봅시다:

```perl Elixir
  def update(conn, %{"cart" => cart_params}) do
    case ShoppingCart.update_cart(conn.assigns.cart, cart_params) do
      {:ok, _cart} ->
        redirect(conn, to: ~p"/cart")

      {:error, _changeset} ->
        conn
        |> put_flash(:error, "There was an error updating your cart")
        |> redirect(to: ~p"/cart")
    end
  end
```

먼저 양식 제출에서 카트 매개변수를 추출하는 것으로 시작했습니다.
다음으로 컨텍스트 생성기가 추가한 기존 `ShoppingCart.update_cart/2` 함수를 호출합니다.
이 함수에 약간의 변경이 필요하지만 인터페이스는 그대로 사용해도 좋습니다.
업데이트가 성공하면 카트 페이지로 리디렉션하고, 그렇지 않으면 플래시 오류 메시지를 표시하고 사용자를 카트 페이지로 다시 보내 실수를 수정합니다.
기본적으로 `ShoppingCart.update_cart/2` 함수는 카트 매개변수를 변경 집합으로 캐스팅하고 리포지토리에 대해 업데이트하는 데만 관여합니다.
이제 중첩된 카트 항목 연결을 처리하고, 가장 중요한 것은 수량 0개 품목이 카트에서 제거되는 것과 같은 수량 업데이트를 처리하는 비즈니스 로직이 필요합니다.

다시 장바구니 컨텍스트의 `lib/hello/shopping_cart.ex`로 돌아가서 `update_cart/2` 함수를 다음 구현으로 바꾸세요:

```perl Elixir
  def update_cart(%Cart{} = cart, attrs) do
    changeset =
      cart
      |> Cart.changeset(attrs)
      |> Ecto.Changeset.cast_assoc(:items, with: &CartItem.changeset/2)

    Ecto.Multi.new()
    |> Ecto.Multi.update(:cart, changeset)
    |> Ecto.Multi.delete_all(:discarded_items, fn %{cart: cart} ->
      from(i in CartItem, where: i.cart_id == ^cart.id and i.quantity == 0)
    end)
    |> Repo.transaction()
    |> case do
      {:ok, %{cart: cart}} -> {:ok, cart}
      {:error, :cart, changeset, _changes_so_far} -> {:error, changeset}
    end
  end
```

장바구니 구조체를 가져와서 사용자 입력을 장바구니 변경 집합으로 캐스팅하지만, 이번에는 `Ecto.Changeset.cast_assoc/3`을 사용하여 중첩된 항목 데이터를 `CartItem` 변경 집합으로 캐스팅한다는 점을 제외하면 기본 코드에서 시작한 것과 매우 유사합니다.
카트 양식 템플릿에서 [`<.inputs_for />`](`Phoenix.Component.inputs_for/1`) 호출을 기억하시나요? 이 숨겨진 ID 데이터는 Ecto의 `cast_assoc`이 아이템 데이터를 카트의 기존 아이템 연결에 다시 매핑할 수 있게 해줍니다.
다음으로는 이전에 보지 못했을 수도 있는 `Ecto.Multi.new/0`을 사용합니다.
Ecto의 `Multi`는 데이터베이스 트랜잭션 내에서 실행되는 일련의 명명된 작업 체인을 느리게 정의할 수 있는 기능입니다.
멀티 체인의 각 연산은 이전 단계의 값을 받아 실패한 단계가 발생할 때까지 실행됩니다.
연산이 실패하면 트랜잭션이 롤백되고 오류가 반환되며, 그렇지 않으면 트랜잭션이 커밋됩니다.

다중 작업의 경우 `:cart`라는 이름의 카트 업데이트를 발행하는 것으로 시작합니다.
카트 업데이트가 발행된 후, 업데이트된 카트를 가져와서 제로 수량 로직을 적용하는 다중 `delete_all` 작업을 수행합니다.
수량이 0인 이 카트의 모든 카트 품목을 찾는 엑토 쿼리를 반환하여 수량이 0인 카트의 모든 품목을 정리합니다.
멀티를 사용하여 `Repo.transaction/1`을 호출하면 새 트랜잭션에서 작업이 실행되고 원래 함수와 마찬가지로 호출자에게 성공 또는 실패 결과를 반환합니다.

이제 브라우저로 돌아가서 사용해 보겠습니다.
장바구니에 몇 가지 제품을 추가하고 수량을 업데이트한 다음 가격 계산과 함께 값이 변경되는 것을 지켜보세요.
수량을 0으로 설정하면 해당 품목도 제거됩니다.
아주 깔끔하죠!

## Adding an Orders context

`Catalog` 및 `ShoppingCart` 컨텍스트를 통해 잘 고려한 모듈과 함수 이름이 어떻게 명확하고 유지 관리하기 쉬운 코드를 생성하는지 직접 확인하고 있습니다.
마지막 단계는 사용자가 결제 프로세스를 시작할 수 있도록 하는 것입니다.
결제 처리나 주문 처리를 통합하는 데까지 나아가지는 않겠지만, 그 방향으로 나아갈 수 있도록 도와드리겠습니다.
이전과 마찬가지로 주문 완료를 위한 코드를 어디에 배치할지 결정해야 합니다.
카탈로그의 일부일까요? 분명 카탈로그는 아니지만 장바구니는 어떨까요? 장바구니는 주문과 관련이 있습니다. 결국 사용자가 제품을 구매하기 위해서는 항목을 추가해야 하는데, 주문 결제 프로세스를 여기에 그룹화해야 할까요?

주문 프로세스를 잠시 멈추고 생각해 보면, 주문에는 관련성이 있지만 카트 내용과는 확연히 다른 데이터가 포함된다는 것을 알 수 있습니다.
또한 결제 프로세스와 관련된 비즈니스 규칙은 카트 작성과는 많이 다릅니다.
예를 들어, 사용자가 이월 주문한 품목을 카트에 추가하는 것은 허용할 수 있지만 재고가 없는 주문을 완료하는 것은 허용할 수 없습니다.
또한 주문이 완료된 시점의 제품 정보(예: 결제 거래 시점의 제품 가격)를 캡처해야 합니다.
이는 향후 제품 가격이 변경될 수 있지만 주문의 품목은 항상 구매 시점에 청구된 금액을 기록하고 표시해야 하기 때문에 필수적입니다.
이러한 이유로 우리는 주문이 자체적인 데이터 문제와 비즈니스 규칙을 통해 독립적으로 존재할 수 있음을 알 수 있습니다.

이름 지정이 현명한 `Orders`은 컨텍스트의 범위를 명확하게 정의하므로 컨텍스트 생성기를 다시 활용하여 시작해 보겠습니다.
콘솔에서 다음 명령을 실행합니다:

```shell
$ mix phx.gen.context Orders Order orders user_uuid:uuid total_price:decimal

* creating lib/hello/orders/order.ex
* creating priv/repo/migrations/20210209214612_create_orders.exs
* creating lib/hello/orders.ex
* injecting lib/hello/orders.ex
* creating test/hello/orders_test.exs
* injecting test/hello/orders_test.exs
* creating test/support/fixtures/orders_fixtures.ex
* injecting test/support/fixtures/orders_fixtures.ex

마이그레이션을 실행하여 리포지토리를 업데이트하는 것을 잊지 마세요:

    $ mix ecto.migrate
```

`Orders` 컨텍스트를 생성했습니다.
플레이스홀더인 현재 사용자를 주문에 연결하기 위해 `user_uuid` 필드를 추가하고 `total_price` 열을 추가했습니다.
시작점이 마련되었으므로 `priv/repo/migrations/*_create_orders.exs`에서 새로 생성된 마이그레이션을 열고 다음과 같이 변경해 보겠습니다:

```perl Elixir
  def change do
    create table(:orders) do
      add :user_uuid, :uuid
-     add :total_price, :decimal
+     add :total_price, :decimal, precision: 15, scale: 6, null: false

      timestamps()
    end
  end
```

이전에 했던 것처럼 소수점 열에 적절한 정밀도 및 스케일 옵션을 지정하여 정밀도 손실 없이 통화를 저장할 수 있도록 했습니다.
또한 모든 주문에 가격을 적용하기 위해 null이 아닌 제약 조건을 추가했습니다.

주문 테이블만으로는 많은 정보를 저장할 수 없지만, 주문에 포함된 모든 품목의 특정 시점 제품 가격 정보를 저장해야 한다는 것을 알고 있습니다.
이를 위해 이 컨텍스트에 `LineItem`이라는 이름의 구조체를 추가하겠습니다.
라인 아이템은 _결제 거래 시점의_ 제품 가격을 캡처합니다.
다음 명령을 실행하세요:

```shell
$ mix phx.gen.context Orders LineItem order_line_items \
price:decimal quantity:integer \
order_id:references:orders product_id:references:products

기존 컨텍스트로 생성하고 있습니다.
...
계속 진행하시겠습니까? [Yn] y
* creating lib/hello/orders/line_item.ex
* creating priv/repo/migrations/20210209215050_create_order_line_items.exs
* injecting lib/hello/orders.ex
* injecting test/hello/orders_test.exs
* injecting test/support/fixtures/orders_fixtures.ex

마이그레이션을 실행하여 리포지토리를 업데이트하는 것을 잊지 마세요:

    $ mix ecto.migrate
```

`phx.gen.context` 명령을 사용하여 `LineItem` 엑토 스키마를 생성하고 지원 기능을 주문 컨텍스트에 삽입했습니다.
이전과 마찬가지로 `priv/repo/migrations/*_create_order_line_items.exs`에서 마이그레이션을 수정하고 다음과 같이 소수점 필드를 변경해 보겠습니다:

```perl Elixir
  def change do
    create table(:order_line_items) do
-     add :price, :decimal
+     add :price, :decimal, precision: 15, scale: 6, null: false
      add :quantity, :integer
      add :order_id, references(:orders, on_delete: :nothing)
      add :product_id, references(:products, on_delete: :nothing)

      timestamps()
    end

    create index(:order_line_items, [:order_id])
    create index(:order_line_items, [:product_id])
  end
```

마이그레이션이 완료되었으므로 이제 `lib/hello/orders/order.ex`에서 주문과 품목 연결을 연결해 보겠습니다:

```perl Elixir
  schema "orders" do
    field :total_price, :decimal
    field :user_uuid, Ecto.UUID

+   has_many :line_items, Hello.Orders.LineItem
+   has_many :products, through: [:line_items, :product]

    timestamps()
  end
```

앞서 살펴본 것처럼 `has_many :line_items`를 사용하여 주문과 품목을 연결했습니다.
다음으로, 다른 관계에서 리소스를 연결하는 방법을 엑토에 지시할 수 있는 `has_many`의 `:through` 기능을 사용했습니다.
이 경우 연관된 라인 항목을 통해 모든 제품을 찾아 주문의 제품을 연결할 수 있습니다.
다음으로, `lib/hello/orders/line_item.ex`에서 다른 방향으로 연결을 설정해 보겠습니다:

```perl Elixir
  schema "order_line_items" do
    field :price, :decimal
    field :quantity, :integer
-   field :order_id, :id
-   field :product_id, :id

+   belongs_to :order, Hello.Orders.Order
+   belongs_to :product, Hello.Catalog.Product

    timestamps()
  end
```

여기서는 `belongs_to`를 사용하여 품목을 주문 및 제품에 연결했습니다.
연결이 완료되었으므로 이제 웹 인터페이스를 주문 프로세스에 통합할 수 있습니다.
라우터 `lib/hello_web/router.ex`를 열고 다음 줄을 추가합니다:

```perl Elixir
  scope "/", HelloWeb do
    pipe_through :browser

    ...
+   resources "/orders", OrderController, only: [:create, :show]
  end
```

현재 필요한 작업은 이 두 가지뿐이므로 생성된 `OrderController`에 대한 `create` 및 `show` 경로를 연결했습니다.
경로가 준비되었으므로 이제 마이그레이션할 수 있습니다:

```shell
mix ecto.migrate

17:14:37.715 [info] == Running 20210209214612 Hello.Repo.Migrations.CreateOrders.change/0 forward

17:14:37.720 [info] create table orders

17:14:37.755 [info] == Migrated 20210209214612 in 0.0s

17:14:37.784 [info] == Running 20210209215050 Hello.Repo.Migrations.CreateOrderLineItems.change/0 forward

17:14:37.785 [info] create table order_line_items

17:14:37.795 [info] create index order_line_items_order_id_index

17:14:37.796 [info] create index order_line_items_product_id_index

17:14:37.798 [info] == Migrated 20210209215050 in 0.0s
```

주문에 대한 정보를 렌더링하기 전에 주문 데이터가 완전히 채워져 있고 현재 사용자가 조회할 수 있는지 확인해야 합니다.
`lib/hello/orders.ex`에서 주문 컨텍스트를 열고 `get_order!/1` 함수를 새로운 `get_order!/2` 정의로 바꿉니다:

```perl Elixir
  def get_order!(user_uuid, id) do
    Order
    |> Repo.get_by!(id: id, user_uuid: user_uuid)
    |> Repo.preload([line_items: [:product]])
  end
```

사용자 UUID를 수락하고 주어진 주문 ID에 대해 사용자 ID와 일치하는 주문이 있는지 저장소에 쿼리하도록 함수를 재작성했습니다.
그런 다음 품목과 제품 연결을 미리 로드하여 주문을 채웠습니다.

주문을 완료하려면 카트 페이지에서 `OrderController.create` 액션에 POST를 발행할 수 있지만 실제로 주문을 완료하려면 작업과 로직을 구현해야 합니다.
이전과 마찬가지로 웹 인터페이스에서 시작하겠습니다.
`lib/hello_web/controllers/order_controller.ex`에 새 파일을 생성하고 이 파일을 입력합니다:

```perl Elixir
defmodule HelloWeb.OrderController do
  use HelloWeb, :controller

  alias Hello.Orders

  def create(conn, _) do
    case Orders.complete_order(conn.assigns.cart) do
      {:ok, order} ->
        conn
        |> put_flash(:info, "Order created successfully.")
        |> redirect(to: ~p"/orders/#{order}")

      {:error, _reason} ->
        conn
        |> put_flash(:error, "There was an error processing your order")
        |> redirect(to: ~p"/cart")
    end
  end
end
```

아직 구현되지 않은 `Orders.complete_order/1` 함수를 호출하기 위해 `create` 액션을 작성했습니다.
이 코드는 기술적으로 주문을 '생성'하는 것이지만, 한 걸음 물러나서 인터페이스의 이름 지정을 고려하는 것이 중요합니다.
주문을 '완료'하는 행위는 우리 시스템에서 매우 중요합니다.
거래에서 돈이 오고가고, 실제 상품이 자동으로 배송되는 등의 작업이 이루어질 수 있습니다.
이러한 작업에는 '주문 완료'와 같이 더 명확하고 더 나은 함수 이름이 필요합니다.
주문이 성공적으로 완료되면 쇼 페이지로 리디렉션되고, 그렇지 않으면 카트 페이지로 다시 리디렉션되면서 플래시 오류가 표시됩니다.

컨텍스트가 다른 컨텍스트에서 정의한 데이터에서도 자연스럽게 작동할 수 있다는 점을 강조할 수 있는 좋은 기회이기도 합니다.
이는 특히 여기서의 장바구니와 같이 애플리케이션 전체에서 사용되는 데이터(프로젝트에 따라 현재 사용자 또는 현재 프로젝트 등이 될 수도 있음)에서 흔히 볼 수 있습니다.

이제 `Orders.complete_order/1` 함수를 구현할 수 있습니다.
주문을 완료하려면 몇 가지 작업이 필요합니다:

1.  A new order record must be persisted with the total price of the order
2.  All items in the cart must be transformed into new order line items records
    with quantity and point-in-time product price information
3.  After successful order insert (and eventual payment), items must be pruned
    from the cart

요구 사항만 보더라도 일반적인 `create_order` 함수가 적합하지 않은 이유를 알 수 있습니다.
이 새로운 함수를 `lib/hello/orders.ex`에 구현해 봅시다:

```perl Elixir
  alias Hello.Orders.LineItem
  alias Hello.ShoppingCart

  def complete_order(%ShoppingCart.Cart{} = cart) do
    line_items =
      Enum.map(cart.items, fn item ->
        %{product_id: item.product_id, price: item.product.price, quantity: item.quantity}
      end)

    order =
      Ecto.Changeset.change(%Order{},
        user_uuid: cart.user_uuid,
        total_price: ShoppingCart.total_cart_price(cart),
        line_items: line_items
      )

    Ecto.Multi.new()
    |> Ecto.Multi.insert(:order, order)
    |> Ecto.Multi.run(:prune_cart, fn _repo, _changes ->
      ShoppingCart.prune_cart_items(cart)
    end)
    |> Repo.transaction()
    |> case do
      {:ok, %{order: order}} -> {:ok, order}
      {:error, name, value, _changes_so_far} -> {:error, {name, value}}
    end
  end
```

먼저 쇼핑 카트의 `%ShoppingCart.CartItem{}`을 주문 품목 구조의 맵에 매핑하는 것으로 시작하겠습니다.
주문 품목 레코드의 역할은 _결제 거래 시점의_ 제품 가격을 캡처하는 것이므로 여기서 제품 가격을 참조합니다.
다음으로 `Ecto.Changeset.change/2`를 사용하여 베어 주문 변경 집합을 생성하고 사용자 UUID를 연결하고 총 가격 계산을 설정한 다음 주문 품목을 변경 집합에 배치합니다.
새로운 주문 변경 집합을 삽입할 준비가 되었으면 다시 `Ecto.Multi`를 사용하여 데이터베이스 트랜잭션에서 작업을 실행할 수 있습니다.
먼저 주문을 삽입한 다음 `run` 연산을 실행합니다.
`Ecto.Multi.run/3` 함수를 사용하면 `{:ok, result}`로 성공하거나 트랜잭션을 중단하고 롤백하는 오류가 발생해야 하는 함수에서 모든 코드를 실행할 수 있습니다.
여기서는 장바구니 컨텍스트를 호출하여 장바구니에 있는 모든 항목을 잘라내도록 요청하기만 하면 됩니다.
트랜잭션을 실행하면 이전과 같이 멀티가 실행되고 호출자에게 결과를 반환합니다.

주문 완료를 마무리하려면 `lib/hello/shopping_cart.ex`에서 `ShoppingCart.prune_cart_items/1` 함수를 구현해야 합니다:

```perl Elixir
  def prune_cart_items(%Cart{} = cart) do
    {_, _} = Repo.delete_all(from(i in CartItem, where: i.cart_id == ^cart.id))
    {:ok, reload_cart(cart)}
  end
```

새 함수는 카트 구조체를 받아들이고 제공된 카트의 모든 항목에 대한 쿼리를 수락하는 `Repo.delete_all`을 실행합니다.
호출자에게 정리된 카트를 다시 로드하여 성공 결과를 반환합니다.
컨텍스트가 완료되었으므로 이제 사용자에게 완료된 주문을 표시해야 합니다.
주문 컨트롤러로 돌아가서 `show/2` 액션을 추가합니다:

```perl Elixir
  def show(conn, %{"id" => id}) do
    order = Orders.get_order!(conn.assigns.current_uuid, id)
    render(conn, :show, order: order)
  end
```

주문 소유자만 주문을 볼 수 있도록 권한을 부여하는 `conn.assigns.current_uuid`를 `get_order!`에 전달하기 위해 show 액션을 추가했습니다.
다음으로 보기와 템플릿을 구현할 수 있습니다.
`lib/hello_web/controllers/order_html.ex`에 다음 내용으로 새 보기 파일을 만듭니다:

```perl Elixir
defmodule HelloWeb.OrderHTML do
  use HelloWeb, :html

  embed_templates "order_html/*"
end
```

다음으로 `lib/hello_web/controllers/order_html/show.html.heex`에서 템플릿을 만들 수 있습니다:

```perl HEEx
<.header>
  Thank you for your order!
  <:subtitle>
     <strong>User uuid: </strong><%= @order.user_uuid %>
  </:subtitle>
</.header>

<.table id="items" rows={@order.line_items}>
  <:col :let={item} label="Title"><%= item.product.title %></:col>
  <:col :let={item} label="Quantity"><%= item.quantity %></:col>
  <:col :let={item} label="Price">
    <%= HelloWeb.CartHTML.currency_to_str(item.price) %>
  </:col>
</.table>

<strong>Total price:</strong>
<%= HelloWeb.CartHTML.currency_to_str(@order.total_price) %>

<.back navigate={~p"/products"}>Back to products</.back>
```

완료된 주문을 표시하기 위해 주문 사용자를 표시한 다음 제품 제목, 수량, 주문을 완료할 때 "거래한" 가격 및 총 가격을 포함한 품목 목록을 표시했습니다.

마지막으로 카트 페이지에 '주문 완료' 버튼을 추가하여 주문을 완료할 수 있도록 했습니다.
`lib/hello_web/controllers/cart_html/show.html.heex`에 있는 카트 쇼 템플릿의 <.header>에 다음 버튼을 추가합니다:

```diff
  <.header>
    My Cart
+    <:actions>
+      <.link href={~p"/orders"} method="post">
+        <.button>Complete order</.button>
+      </.link>
+    </:actions>
  </.header>
```

method="post"`가 포함된 링크를 추가하여 `OrderController.create` 액션에 POST 요청을 보냅니다.
[http://localhost:4000/cart](`http://localhost:4000/cart`)의 카트 페이지로 돌아가 주문을 완료하면 렌더링된 템플릿이 표시됩니다:

```shell
주문해 주셔서 감사합니다!

사용자 UUID: 08964C7C-908C-4A55-BCD3-9811AD8B0B9D
제목 수량 가격
메타프로그래밍 엘릭서 2 $15.00

총 가격: $30.00
```

수고하셨습니다! 아직 결제 기능을 추가하지는 않았지만 `ShoppingCart`와 `Orders` 컨텍스트 분할이 어떻게 유지 관리 가능한 솔루션으로 우리를 이끌고 있는지 이미 확인할 수 있습니다.
카트 항목과 주문 항목이 분리되었으므로 향후 결제 거래, 카트 가격 감지 등을 추가할 수 있는 준비가 완료되었습니다.

수고하셨습니다!

## FAQ

### How do I structure code inside contexts?

컨텍스트 내에서 코드를 구성하는 방법이 궁금할 수 있습니다.
예를 들어 변경 집합(예: ProductChangesets)을 위한 모듈과 쿼리(예: ProductQueries)를 위한 다른 모듈을 정의해야 하나요?

컨텍스트의 한 가지 중요한 이점은 이러한 결정이 크게 중요하지 않다는 것입니다.
컨텍스트는 공개 API이고 다른 모듈은 비공개입니다.
컨텍스트는 이러한 모듈을 작은 그룹으로 분리하므로 애플리케이션의 표면 영역은 모든 코드가 아닌 컨텍스트입니다.

따라서 여러분과 여러분의 팀이 이러한 비공개 모듈을 구성하는 패턴을 설정할 수 있지만, 저희는 이 패턴이 다르더라도 괜찮다고 생각합니다.
컨텍스트가 정의되는 방식과 컨텍스트가 서로(그리고 웹 애플리케이션과) 상호 작용하는 방식에 중점을 두어야 합니다.

잘 관리된 동네라고 생각하세요.
컨텍스트는 집들이고, 잘 보존되고, 잘 연결되어 있는 등의 상태를 유지하고 싶을 것입니다.
집 안에서는 모두 조금씩 다를 수 있으며, 이는 괜찮습니다.

### Returning Ecto structures from context APIs

컨텍스트 API를 살펴보면서 궁금한 점이 있었을 것입니다:

> If one of the goals of our context is to encapsulate Ecto Repo access, why does `create_user/1` return an `Ecto.Changeset` struct when we fail to create a user?

변경 집합은 Ecto의 일부이지만 데이터베이스에 묶여 있지 않고 모든 소스의 데이터를 매핑하는 데 사용할 수 있으므로 필드 변경 사항을 추적하고 유효성 검사를 수행하며 오류 메시지를 생성하는 데 일반적이고 유용한 데이터 구조가 됩니다.

이러한 이유로 `%Ecto.Changeset{}`은 컨텍스트와 웹 레이어 간의 데이터 변경을 모델링하는 데 적합하며, API 또는 데이터베이스와 통신하는 경우 모두 사용할 수 있습니다.

마지막으로, 컨트롤러와 뷰가 Ecto에서만 작동하도록 하드코딩되어 있지 않다는 점에 유의하세요.
대신, Phoenix는 `Phoenix.Param` 및 `Phoenix.HTML.FormData`와 같은 프로토콜을 정의하여 모든 라이브러리가 Phoenix가 URL 매개변수를 생성하거나 양식을 렌더링하는 방식을 확장할 수 있도록 합니다.
편리하게도 `phoenix_ecto` 프로젝트에서 이러한 프로토콜을 구현하지만, 자체 데이터 구조를 가져와서 직접 구현할 수도 있습니다.
