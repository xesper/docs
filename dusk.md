# 브라우저 테스트 (라라벨 Dusk)

- [소개하기](#introduction)
- [설치하기](#installation)
    - [다른 브라우저 사용하기](#using-other-browsers)
- [시작하기](#getting-started)
    - [테스트 클래스 생성하기](#generating-tests)
    - [테스트 실행하기](#running-tests)
    - [구동환경 처리](#environment-handling)
    - [브라우저 생성하기](#creating-browsers)
    - [인증](#authentication)
- [Interacting With Elements](#interacting-with-elements)
    - [링크 클릭](#clicking-links)
    - [Text, Values, & Attributes](#text-values-and-attributes)
    - [Using Forms](#using-forms)
    - [파일 첨부](#attaching-files)
    - [키보드 사용하기](#using-the-keyboard)
    - [마우스 사용하기](#using-the-mouse)
    - [Scoping Selectors](#scoping-selectors)
    - [Waiting For Elements](#waiting-for-elements)
- [Available Assertions](#available-assertions)
- [Pages](#pages)
    - [Generating Pages](#generating-pages)
    - [Configuring Pages](#configuring-pages)
    - [Navigating To Pages](#navigating-to-pages)
    - [Shorthand Selectors](#shorthand-selectors)
    - [Page Methods](#page-methods)

<a name="introduction"></a>
## 소개하기

라라벨 Dusk는 구성과 사용이 쉬운 브라우저 자동화 및 테스팅 API를 제공합니다. 기본적으로 Dusk는 사용자 머신에 JDK 나 Selenium을 설치하도록 요구하지 않습니다. 대신에 Dusk는 독립적인 [ChromeDriver](https://sites.google.com/a/chromium.org/chromedriver/home)를 사용합니다. 그렇긴 하지만, 원하는 경우 다른 Selenium 호환 드라이버를 사용할 수도 있습니다.

<a name="installation"></a>
## 설치하기

시작하기 위해서, 컴포저 의존성에 `laravel/dusk`을 추가해야 합니다:

    composer require laravel/dusk

Dusk가 설치되고 나면, `Laravel\Dusk\DuskServiceProvider` 서비스 프로바이더를 등록해야 합니다. 다른 사용자로 로그인 할 수 있는 기능을 가지고 있어, Dusk를 사용할 수있는 환경을 제한하기 위해서 `AppServiceProvider`의 `register` 메소드에서 프로바이더를 등록해야합니다:

    use Laravel\Dusk\DuskServiceProvider;

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        if ($this->app->environment('local', 'testing')) {
            $this->app->register(DuskServiceProvider::class);
        }
    }

다음으로 `dusk:install` 아티즌 명령어를 실행합니다:

    php artisan dusk:install

`tests` 디렉토리 안에 `Browser` 디렉토리가 생성되고 예제 테스트가 포함됩니다. 그런 다음에 `.env` 파일에서 `APP_URL` 환경 변수를 설정하십시오. 이 변수는 브라우저에서 어플리케이션에 엑세스 하는데 사용하는 URL과 일치해야 합니다.

테스트를 실행하기 위해서는 `dusk` 아티즌 명령어를 사용합니다. `dusk` 명령어는 `phpunit` 명령어에 전달할 수 있는 모든 인자를 받을 수 있습니다:

    php artisan dusk

<a name="using-other-browsers"></a>
### 다른 브라우저 사용하기

기본적으로, Dusk는 브라우저 테스트에서 구글 크롬 및 [ChromeDriver](https://sites.google.com/a/chromium.org/chromedriver/home)을 사용합니다. 그렇지만, 여러분이 다른 Selenium 서버를 구성하고 원하는 브라우저를 테스트할 수도 있습니다.

다른 브라우저를 사용하기 위해서는, 어플리케이션의 베이스 Dusk 테스트 케이스가 되는 `tests/DuskTestCase.php` 파일을 엽니다. 이 파일안에 있는 `startChromeDriver`을 제거하면 됩니다. 이렇게 하면 Dusk가 자동으로 ChromeDriver를 시작하지 않게 됩니다:

    /**
     * Prepare for Dusk test execution.
     *
     * @beforeClass
     * @return void
     */
    public static function prepare()
    {
        // static::startChromeDriver();
    }

다음으로 `driver` 메소드에 접속하고자 하는 URL을 수정하면 됩니다. 또한 WebDriver에 전달되어야하는 "desired capabilities"을 수정할 수도 있습니다.

(역자주 : "desired capabilities"는 Facebook\WebDriver\Remote\RemoteWebDriver 의 create 메소드에서 필요한 DesiredCapabilities 클래스 호출을 의미합니다)

    /**
     * Create the RemoteWebDriver instance.
     *
     * @return \Facebook\WebDriver\Remote\RemoteWebDriver
     */
    protected function driver()
    {
        return RemoteWebDriver::create(
            'http://localhost:4444', DesiredCapabilities::phantomjs()
        );
    }

<a name="getting-started"></a>
## 시작하기

<a name="generating-tests"></a>
### 테스트 클래스 생성하기

Dusk 테스트를 생성하기 위해서는 `dusk:make` 아티즌 명령어를 사용합니다. 생성된 테스트 파일은 `tests/Browser` 디렉토리에 저장됩니다:

    php artisan dusk:make LoginTest

<a name="running-tests"></a>
### 테스트 실행하기

브라우저 테스트를 실행하려면, `dusk` 아티즌 명령어를 사용하십시오:

    php artisan dusk

`dusk` 명령어는 일반적으로 PHPUnit 테스트에 전달할 수 있는 인자들을 받을 수 있기 때문에, 지정된 [그룹](https://phpunit.de/manual/current/en/appendixes.annotations.html#appendixes.annotations.group)에 대해서 테스트를 실행할 수도 있습니다.

    php artisan dusk --group=foo

#### 수동으로 ChromeDriver 시작하기

기본적으로는 Dusk가 자동으로 ChromeDriver를 시작합니다. 만약 특정 시스템에서 ChromeDriver가 동작하지 않으면 `dusk` 명령어를 실행하기 전에 수동으로 ChromeDriver를 시작해야 합니다. 수동으로 ChromeDriver를 시작하려면, `tests/DuskTestCase.php` 파일의 다음 라인을 주석으로 표시해야합니다:

    /**
     * Prepare for Dusk test execution.
     *
     * @beforeClass
     * @return void
     */
    public static function prepare()
    {
        // static::startChromeDriver();
    }

또한, ChromeDriver가 9515포트가 아닌 다른 포트를 사용한다면, 이 클래스의 `driver` 메소드를 수정해야합니다:

    /**
     * Create the RemoteWebDriver instance.
     *
     * @return \Facebook\WebDriver\Remote\RemoteWebDriver
     */
    protected function driver()
    {
        return RemoteWebDriver::create(
            'http://localhost:9515', DesiredCapabilities::chrome()
        );
    }

<a name="environment-handling"></a>
### Environment Handling

To force Dusk to use its own environment file when running tests, create a `.env.dusk.{environment}` file in the root of your project. For example, if you will be initiating the `dusk` command from your `local` environment, you should create a `.env.dusk.local` file.

When running tests, Dusk will back-up your `.env` file and rename your Dusk environment to `.env`. Once the tests have completed, your `.env` file will be restored.

<a name="creating-browsers"></a>
### Creating Browsers

To get started, let's write a test that verifies we can log into our application. After generating a test, we can modify it to navigate to the login page, enter some credentials, and click the "Login" button. To create a browser instance, call the `browse` method:

    <?php

    namespace Tests\Browser;

    use App\User;
    use Tests\DuskTestCase;
    use Laravel\Dusk\Chrome;
    use Illuminate\Foundation\Testing\DatabaseMigrations;

    class ExampleTest extends DuskTestCase
    {
        use DatabaseMigrations;

        /**
         * A basic browser test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $user = factory(User::class)->create([
                'email' => 'taylor@laravel.com',
            ]);

            $this->browse(function ($browser) use ($user) {
                $browser->visit('/login')
                        ->type('email', $user->email)
                        ->type('password', 'secret')
                        ->press('Login')
                        ->assertPathIs('/home');
            });
        }
    }

As you can see in the example above, the `browse` method accepts a callback. A browser instance will automatically be passed to this callback by Dusk and is the main object used to interact with and make assertions against your application.

> {tip} This test can be used to test the login screen generated by the `make:auth` Artisan command.

#### Creating Multiple Browsers

Sometimes you may need multiple browsers in order to properly carry out a test. For example, multiple browsers may be needed to test a chat screen that interacts with websockets. To create multiple browsers, simply "ask" for more than one browser in the signature of the callback given to the `browse` method:

    $this->browse(function ($first, $second) {
        $first->loginAs(User::find(1))
              ->visit('/home')
              ->waitForText('Message');

        $second->loginAs(User::find(2))
               ->visit('/home')
               ->waitForText('Message')
               ->type('message', 'Hey Taylor')
               ->press('Send');

        $first->waitForText('Hey Taylor')
              ->assertSee('Jeffrey Way');
    });

<a name="authentication"></a>
### Authentication

Often, you will be testing pages that require authentication. You can use Dusk's `loginAs` method in order to avoid interacting with the login screen during every test. The `loginAs` method accepts a user ID or user model instance:

    $this->browse(function ($first, $second) {
        $first->loginAs(User::find(1))
              ->visit('/home');
    });

<a name="interacting-with-elements"></a>
## Interacting With Elements

<a name="clicking-links"></a>
### Clicking Links

To click a link, you may use the `clickLink` method on the browser instance. The `clickLink` method will click the link that has the given display text:

    $browser->clickLink($linkText);

> {note} This method interacts with jQuery. If jQuery is not available on the page, Dusk will automatically inject it into the page so it is available for the test's duration.

<a name="text-values-and-attributes"></a>
### Text, Values, & Attributes

#### Retrieving & Setting Values

Dusk provides several methods for interacting with the current display text, value, and attributes of elements on the page. For example, to get the "value" of an element that matches a given selector, use the `value` method:

    // Retrieve the value...
    $value = $browser->value('selector');

    // Set the value...
    $browser->value('selector', 'value');

#### Retrieving Text

The `text` method may be used to retrieve the display text of an element that matches the given selector:

    $text = $browser->text('selector');

#### Retrieving Attributes

Finally, the `attribute` method may be used to retrieve an attribute of an element matching the given selector:

    $attribute = $browser->attribute('selector', 'value');

<a name="using-forms"></a>
### Using Forms

#### Typing Values

Dusk provides a variety of methods for interacting with forms and input elements. First, let's take a look at an example of typing text into an input field:

    $browser->type('email', 'taylor@laravel.com');

Note that, although the method accepts one if necessary, we are not required to pass a CSS selector into the `type` method. If a CSS selector is not provided, Dusk will search for an input field with the given `name` attribute. Finally, Dusk will attempt to find a `textarea` with the given `name` attribute.

You may "clear" the value of an input using the `clear` method:

    $browser->clear('email');

#### Dropdowns

To select a value in a dropdown selection box, you may use the `select` method. Like the `type` method, the `select` method does not require a full CSS selector. When passing a value to the `select` method, you should pass the underlying option value instead of the display text:

    $browser->select('size', 'Large');

#### Checkboxes

To "check" a checkbox field, you may use the `check` method. Like many other input related methods, a full CSS selector is not required. If an exact selector match can't be found, Dusk will search for a checkbox with a matching `name` attribute:

    $browser->check('terms');

    $browser->uncheck('terms');

#### Radio Buttons

To "select" a radio button option, you may use the `radio` method. Like many other input related methods, a full CSS selector is not required. If an exact selector match can't be found, Dusk will search for a radio with matching `name` and `value` attributes:

    $browser->radio('version', 'php7');

<a name="attaching-files"></a>
### Attaching Files

The `attach` method may be used to attach a file to a `file` input element. Like many other input related methods, a full CSS selector is not required. If an exact selector match can't be found, Dusk will search for a file input with matching `name` attribute:

    $browser->attach('photo', __DIR__.'/photos/me.png');

<a name="using-the-keyboard"></a>
### Using The Keyboard

The `keys` method allows you to provide more complex input sequences to a given element than normally allowed by the `type` method. For example, you may hold modifier keys entering values. In this example, the `shift` key will be held while `taylor` is entered into the element matching the given selector. After `taylor` is typed, `otwell` will be typed without any modifier keys:

    $browser->keys('selector', ['{shift}', 'taylor'], 'otwell');

You may even send a "hot key" to the primary CSS selector that contains your application:

    $browser->keys('.app', ['{command}', 'j']);

> {tip} All modifier keys are wrapped in `{}` characters, and match the constants defined in the `Facebook\WebDriver\WebDriverKeys` class, which can be [found on GitHub](https://github.com/facebook/php-webdriver/blob/community/lib/WebDriverKeys.php).

<a name="using-the-mouse"></a>
### Using The Mouse

#### Clicking On Elements

The `click` method may be used to "click" on an element matching the given selector:

    $browser->click('.selector');

#### Mouseover

The `mouseover` method may be used when you need to move the mouse over an element matching the given selector:

    $browser->mouseover('.selector');

#### Drag & Drop

The `drag` method may be used to drag an element matching the given selector to another element:

    $browser->drag('.from-selector', '.to-selector');

<a name="scoping-selectors"></a>
### Scoping Selectors

Sometimes you may wish to perform several operations while scoping all of the operations within a given selector. For example, you may wish to assert that some text exists only within a table and then click a button within that table. You may use the `with` method to accomplish this. All operations performed within the callback given to the `with` method will be scoped to the original selector:

    $browser->with('.table', function ($table) {
        $table->assertSee('Hello World')
              ->clickLink('Delete');
    });

<a name="waiting-for-elements"></a>
### Waiting For Elements

When testing applications that use JavaScript extensively, it often becomes necessary to "wait" for certain elements or data to be available before proceeding with a test. Dusk makes this a cinch. Using a variety of methods, you may wait for elements to be visible on the page or even wait until a given JavaScript expression evaluates to `true`.

#### Waiting

If you need to pause the test for a given number of milliseconds, use the `pause` method:

    $browser->pause(1000);

#### Waiting For Selectors

The `waitFor` method may be used to pause the execution of the test until the element matching the given CSS selector is displayed on the page. By default, this will pause the test for a maximum of five seconds before throwing an exception. If necessary, you may pass a custom timeout threshold as the second argument to the method:

    // Wait a maximum of five seconds for the selector...
    $browser->waitFor('.selector');

    // Wait a maximum of one second for the selector...
    $browser->waitFor('.selector', 1);

You may also wait until the given selector is missing from the page:

    $browser->waitUntilMissing('.selector');

    $browser->waitUntilMissing('.selector', 1);

#### Scoping Selectors When Available

Occasionally, you may wish to wait for a given selector and then interact with the element matching the selector. For example, you may wish to wait until a modal window is available and then press the "OK" button within the modal. The `whenAvailable` method may be used in this case. All element operations performed within the given callback will be scoped to the original selector:

    $browser->whenAvailable('.modal', function ($modal) {
        $modal->assertSee('Hello World')
              ->press('OK');
    });

#### Waiting For Text

The `waitForText` method may be used to wait until the given text is displayed on the page:

    // Wait a maximum of five seconds for the text...
    $browser->waitForText('Hello World');

    // Wait a maximum of one second for the text...
    $browser->waitForText('Hello World', 1);

#### Waiting For Links

The `waitForLink` method may be used to wait until the given link text is displayed on the page:

    // Wait a maximum of five seconds for the link...
    $browser->waitForLink('Create');

    // Wait a maximum of one second for the link...
    $browser->waitForLink('Create', 1);

#### Waiting On JavaScript Expressions

Sometimes you may wish to pause the execution of a test until a given JavaScript expression evaluates to `true`. You may easily accomplish this using the `waitUntil` method. When passing an expression to this method, you do not need to include the `return` keyword or an ending semi-colon:

    // Wait a maximum of five seconds for the expression to be true...
    $browser->waitUntil('App.dataLoaded');

    $browser->waitUntil('App.data.servers.length > 0');

    // Wait a maximum of one second for the expression to be true...
    $browser->waitUntil('App.data.servers.length > 0', 1);

<a name="available-assertions"></a>
## Available Assertions

Dusk provides a variety of assertions that you may make against your application. All of the available assertions are documented in the table below:

Assertion  | Description
------------- | -------------
`$browser->assertTitle($title)`  |  Assert the page title matches the given text.
`$browser->assertTitleContains($title)`  |  Assert the page title contains the given text.
`$browser->assertPathIs('/home')`  |  Assert the current path matches the given path.
`$browser->assertHasCookie($name)`  |  Assert the given cookie is present.
`$browser->assertCookieValue($name, $value)`  |  Assert a cookie has a given value.
`$browser->assertPlainCookieValue($name, $value)`  |  Assert an unencrypted cookie has a given value.
`$browser->assertSee($text)`  |  Assert the given text is present on the page.
`$browser->assertDontSee($text)`  |  Assert the given text is not present on the page.
`$browser->assertSeeIn($selector, $text)`  |  Assert the given text is present within the selector.
`$browser->assertDontSeeIn($selector, $text)`  |  Assert the given text is not present within the selector.
`$browser->assertSeeLink($linkText)`  |  Assert the given link is present on the page.
`$browser->assertDontSeeLink($linkText)`  |  Assert the given link is not present on the page.
`$browser->assertInputValue($field, $value)`  |  Assert the given input field has the given value.
`$browser->assertInputValueIsNot($field, $value)`  |  Assert the given input field does not have the given value.
`$browser->assertChecked($field)`  |  Assert the given checkbox is checked.
`$browser->assertNotChecked($field)`  |  Assert the given checkbox is not checked.
`$browser->assertSelected($field, $value)`  |  Assert the given dropdown has the given value selected.
`$browser->assertNotSelected($field, $value)`  |  Assert the given dropdown does not have the given value selected.
`$browser->assertValue($selector, $value)`  |  Assert the element matching the given selector has the given value.
`$browser->assertVisible($selector)`  |  Assert the element matching the given selector is visible.
`$browser->assertMissing($selector)`  |  Assert the element matching the given selector is not visible.

<a name="pages"></a>
## Pages

Sometimes, tests require several complicated actions to be performed in sequence. This can make your tests harder to read and understand. Pages allow you to define expressive actions that may then be performed on a given page using a single method. Pages also allow you to define short-cuts to common selectors for your application or a single page.

<a name="generating-pages"></a>
### Generating Pages

To generate a page object, use the `dusk:page` Artisan command. All page objects will be placed in the `tests/Browser/Pages` directory:

    php artisan dusk:page Login

<a name="configuring-pages"></a>
### Configuring Pages

By default, pages have three methods: `url`, `assert`, and `selectors`. We will discuss the `url` and `assert` methods now. The `elements` method will be [discussed in more detail below](#shorthand-selectors).

#### The `url` Method

The `url` method should return the path of the URL that represents the page. Dusk will use this URL when navigating to the page in the browser:

    /**
     * Get the URL for the page.
     *
     * @return string
     */
    public function url()
    {
        return '/login';
    }

#### The `assert` Method

The `assert` method may make any assertions necessary to verify that the browser is actually on the given page. Completing this method is not necessary; however, you are free to make these assertions if you wish. These assertions will be run automatically when navigating to the page:

    /**
     * Assert that the browser is on the page.
     *
     * @return void
     */
    public function assert(Browser $browser)
    {
        $browser->assertPathIs($this->url());
    }

<a name="navigating-to-pages"></a>
### Navigating To Pages

Once a page has been configured, you may navigate to it using the `visit` method:

    use Tests\Browser\Pages\Login;

    $browser->visit(new Login);

Sometimes you may already be on a given page and need to "load" the page's selectors and methods into the current test context. This is common when pressing a button and being redirected to a given page without explicitly navigating to it. In this situation, you may use the `on` method to load the page:

    use Tests\Browser\Pages\CreatePlaylist;

    $browser->visit('/dashboard')
            ->clickLink('Create Playlist')
            ->on(new CreatePlaylist)
            ->assertSee('@create');

<a name="shorthand-selectors"></a>
### Shorthand Selectors

The `elements` method of pages allows you to define quick, easy-to-remember shortcuts for any CSS selector on your page. For example, let's define a shortcut for the "email" input field of the application's login page:

    /**
     * Get the element shortcuts for the page.
     *
     * @return array
     */
    public function elements()
    {
        return [
            '@email' => 'input[name=email]',
        ];
    }

Now, you may use this shorthand selector anywhere you would use a full CSS selector:

    $browser->type('@email', 'taylor@laravel.com');

#### Global Shorthand Selectors

After installing Dusk, a base `Page` class will be placed in your `tests/Browser/Pages` directory. This class contains a `siteElements` method which may be used to define global shorthand selectors that should be available on every page throughout your application:

    /**
     * Get the global element shortcuts for the site.
     *
     * @return array
     */
    public static function siteElements()
    {
        return [
            '@element' => '#selector',
        ];
    }

<a name="page-methods"></a>
### Page Methods

In addition to the default methods defined on pages, you may define additional methods which may be used throughout your tests. For example, let's imagine we are building a music management application. A common action for one page of the application might be to create a playlist. Instead of re-writing the logic to create a playlist in each test, you may define a `createPlaylist` method on a page class:

    <?php

    namespace Tests\Browser\Pages;

    use Laravel\Dusk\Browser;

    class Dashboard extends Page
    {
        // Other page methods...

        /**
         * Create a new playlist.
         *
         * @param  \Laravel\Dusk\Browser  $browser
         * @param  string  $name
         * @return void
         */
        public function createPlaylist(Browser $browser, $name)
        {
            $browser->type('name', $name)
                    ->check('share')
                    ->press('Create Playlist');
        }
    }

Once the method has been defined, you may use it within any test that utilizes the page. The browser instance will automatically be passed to the page method:

    use Tests\Browser\Pages\Dashboard;

    $browser->visit(new Dashboard)
            ->createPlaylist('My Playlist')
            ->assertSee('My Playlist');