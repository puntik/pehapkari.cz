---
id: 51
layout: post
title: "Testujeme api v codeception"
perex: ""
author: 31
related_posts: [23, 19]
---

## Motivace a proč to všechno
Budeme předpokládat, je v projektu je funční codeception, tzn. existuje adresář tests v rootu projektu. Bližší informace
jak to rozchodit jsou uvedeny v článku pod čarou.

Nějaký motivační úvod - máme dva endpointy, /v1/login a /v1/homepage.
Testy provedeme na dvou našich základních endpointech - login a homepage.
Login akceptuje přihlašovací údaje a vrací autentifikační token, se kterým je potom možné používat další endpointy.
Homepage akceptuje autentifikační token a vrací základní informace pro zobrazení na mobilním zařízení.
TODO - ukázkové apiary
TODO - hele, funguje to v postmanu :)

## Nastavení codeception
Codeception obsahuje šikovný modul [REST](http://codeception.com/docs/modules/REST), který použijeme. Dále ještě 
použijeme moduly [Db](http://codeception.com/docs/modules/Db) a [Asserts](http://codeception.com/docs/modules/Asserts).

Napřed vytvoříme testovací suite Api.
```sh
php vendor/bin/codecept generate:suite api
```
 
Nakonfigurujeme tests/**api.suite.yml**. Jednotlivé parametry konfigurace jsou samopopisné. Důvod použití modulu Db 
popíšu později.

```yaml
actor: ApiTester
modules:
	enabled:
		- \Helper\Api
		- Asserts
		- REST:
			url: 'https://localhost:8080/api'
			depends: PhpBrowser
		- Db:
			dsn: ''
			user: ''
			password: ''
			dump: tests/_data/fixtures.dump.sql
			populate: true
```

Vyrobíme api testera a tím máme připraveno vše k zahájení testování.
 
```sh
php vendor/bin/codecept build
```  

## Testujeme první endpoint `/v1/login`
Vyrobíme LoginCest
```sh
php vendor/bin/codecept generate:cest api LoginCest
```

```php
/**
 * @dataprovider loginDataProvider
 * @group        basic
 */
public function justLogMeIn(
	ApiTester $I, 
	\Codeception\Scenario $scenario, 
	$fixtures
) {
	// 1 .. load a page
	$I->am(sprintf('not logged user into %s', $fixtures['countryCode']));
	$I->lookForwardTo('log into application and see my homepage.');

	// 2 .. send a login request
	$I->amGoingTo(sprintf('send login request and sign in as %s', $fixtures['email']));
	$I->haveHttpHeader('Country-Code', $fixtures['countryCode']);
	$I->haveHttpHeader('Content-Type', 'application/json');

	$I->sendPOST('/login', [
		'email'          => $fixtures['email'],
		'password'       => $fixtures['password'],
	]);

	// 3 .. check login response, access token should be in response and in db as well
	$I->amGoingTo('check response and I should see access token in database.');
	$I->seeResponseCodeIs(HttpCode::OK);
	$I->seeResponseIsJson();

	$response = json_decode($I->grabResponse(), true);
	$I->assertArrayHasKey('accessToken', $response);
	$I->seeInDatabase('access_tokens', ['token' => $response['accessToken']]);
}

protected function loginDataProvider(): array
{
	return [
		[
			'countryCode'    => 'cs',
			'email'          => 'user2@email.cc',
			'password'       => 'abcd1234',
		],
	];
}
```


Spustíme LoginCest

Vyrobíme HomepageCest
Do ApiTestera si dáme funkci pro logování.
S touto funkcí vytvoříme příslušný test.

Uklidíme po sobě databázi.

Pro příště .. test uploadu souboru, gr





