# HTTP 테스트

- [소개](#introduction)
- [세션 / 인증](#session-and-authentication)
- [JSON API 테스팅](#testing-json-apis)
- [사용가능한 Assertions](#available-assertions)

<a name="introduction"></a>
## 소개

라라벨은 어플리케이션에 HTTP request-요청을 하고, 결과를 검사하는데 사용할 수 있는, 유연한 API를 제공합니다. 다음에 정의된 테스트 예제를 살펴보겠습니다: 

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class ExampleTest extends TestCase
    {
        /**
         * A basic test example.
         *
         * @return void
         */
        public function testBasicTest()
        {
            $response = $this->get('/');

            $response->assertStatus(200);
        }
    }

`get` 메소드는 어플리케이션에 `GET` request-요청을 만들고, `assertStatus` 메소드는 반환된 response-응답이 주어진 HTTP 상태 코드와 일치하는지 확인합니다. 간단한 테스트에 더해, 라라벨은 response의 헤더값, 컨텐츠, JSON 구조 및 기타 확인을 할 수 있는 기능을 제공합니다.  

<a name="session-and-authentication"></a>
### 세션 / 인증

라라벨은 HTTP 테스팅 중 세션 작업을 하는 데 필요한 여러 헬퍼 메소드를 제공합니다. 먼저, `withSession` 메소드를 이용하여 주어진 배열을 세션 데이터로 설정할 수 있습니다. 이것은 어플리케이션에 response-응답을 전달하기 전에 데이터를 세션에 로드하는 경우에 유용합니다:

    <?php

    class ExampleTest extends TestCase
    {
        public function testApplication()
        {
            $response = $this->withSession(['foo' => 'bar'])
                             ->get('/');
        }
    }

물론 일반적인 세션의 이용법 중 하나는 인증된 사용자를 위해서 상태를 유지하는 것입니다. `actingAs` 헬퍼 메소드는 특정 사용자를 현재 사용자로 인증하는 단순한 방법을 제공합니다. 예를 들어, 사용자를 생성하고 인증하기 위해 [model factory](/docs/{{version}}/database-testing#writing-factories)를 사용할 수 있습니다: 

    <?php

    use App\User;

    class ExampleTest extends TestCase
    {
        public function testApplication()
        {
            $user = factory(User::class)->create();

            $response = $this->actingAs($user)
                             ->withSession(['foo' => 'bar'])
                             ->get('/')
        }
    }

또한 `actingAs` 메소드의 두 번째 인자로 주어진 사용자에 대한 인증에 어떤 guard를 사용해야 하는지 지정하도록 guard 이름을 전달 할 수도 있습니다:

    $this->actingAs($user, 'api')

<a name="testing-json-apis"></a>
### JSON API 테스팅하기

라라벨은 또한 JSON API와 그 결과를 테스트하기 위해 여러 헬퍼들을 제공합니다. 예를 들어, `json`, `get`, `post`, `put`, `patch`, 그리고 `delete` 메소드들을 이용하여 다양한 HTTP verb를 가진 request-요청을 할 수 있습니다. 이 메소드들에 손쉽게 데이터와 헤더를 전달할 수도 있습니다. 이를 위해 `/user`에 `POST` request-요청을 하고 원하는 데이터가 반환되는지 확인하는 테스트를 작성해보겠습니다: 

    <?php

    class ExampleTest extends TestCase
    {
        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $response = $this->json('POST', '/user', ['name' => 'Sally']);

            $response
                ->assertStatus(200)
                ->assertJson([
                    'created' => true,
                ]);
        }
    }

> {tip} `assertJson` 메소드는 response-응답을 배열로 변환하고 `PHPUnit::assertArraySubset`을 사용하여 어플리케이션에서 반환된 JSON 배열 안에 주어진 내용이 존재하는지 확인합니다. 따라서 JSON response-응답에 다른 속성이 있더라도, 주어진 내용이 존재하면 테스트는 통과합니다.

<a name="verifying-exact-match"></a>
### 정확하게 일치하는지 확인하기

주어진 배열이 반환된 JSON과 **정확히** 일치하는지 확인하고자 한다면, `assertExactJson` 메소드를 사용하면 됩니다: 

    <?php

    class ExampleTest extends TestCase
    {
        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $response = $this->json('POST', '/user', ['name' => 'Sally']);

            $response
                ->assertStatus(200)
                ->assertExactJson([
                    'created' => true,
                ]);
        }
    }

<a name="available-assertions"></a>
### 사용 가능한 Assertions

라라벨은 [PHPUnit](https://phpunit.de/) 테스트를 위해 다양한 커스텀 assertion 메소드를 제공합니다. 이러한 assertions 은 `json`, `get`, `post`, `put`, 그리고 `delete` 테스트 메소드에서 반환된 response-응답에 엑세스 할 수 있습니다: 

메소드  | 설명
------------- | -------------
`$response->assertStatus($code);`  |  응답이 주어진 코드를 가지고 있는지 확인.
`$response->assertRedirect($uri);`  |  응답이 주어진 URI로 리다이렉트되었는지 여부를 확인.
`$response->assertHeader($headerName, $value = null);`  |  응답에서 주어진 헤더가 존재하는지 확인.
`$response->assertCookie($cookieName, $value = null);`  |  응답에서 주어진 쿠키가 포함되어 있는지 확인.
`$response->assertPlainCookie($cookieName, $value = null);`  |  응답에서 주어진 (암호화 되지 않은)쿠키가 포함되어 있는지 확인.
`$response->assertSessionHas($key, $value = null);`  |  세션에 주어진 데이터가 포함되어 있는지 확인.
`$response->assertSessionHasErrors(array $keys);`  |  세션에 주어진 필드에 대한 에러가 포함되어 있는지 확인.
`$response->assertSessionMissing($key);`  |  세션에 주어진 키가 포함되어 있지 않은 것을 확인.
`$response->assertJson(array $data);`  |  응답에 주어진 JSON 데이터가 포함되어 있는지 확인.
`$response->assertJsonFragment(array $data);`  |  응답에 주어진 JSON 내용이 포함되어 있는지 확인
`$response->assertExactJson(array $data);`  |  응답에 주어진 JSON 데이터와 정확하게 일치하게 포함되어 있는지 확인.
`$response->assertJsonStructure(array $structure);`  |  응답이 주어진 JOSN 구조를 가지고 있는지 확인.
`$response->assertViewHas($key, $value = null);`  |  응답 뷰가 주어진 데이터인지 확인.