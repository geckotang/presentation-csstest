# CSSのテスト

[@GeckoTang](http://twitter.com/GeckoTang)

---

## CSSのテストって何？

- CSSが正しく書かれているかをテスト
- ある部分を変更して、それ以外の部分に影響が出てないかのテスト
- Sassとかで作った@functionが正しい値を返しているかのテスト

---

## やる意味は？

- 目視で気づけない微妙なミスを探せる
- やりようによっては多くのブラウザでチェックを自動化出来る
- Sassが登場したことでCSSがプログラムっぽい感じになってるので、きっと必要
----
<small>※目視チェックももちろん必要だけど…</small>

---

## テスト方法は？

- CSSの値チェックするツールを使う
- 変更する前と後の画像比較するツールを使う
- Sassの@functionのテストツールを使う

---

## CSSの値チェック

間違ったCSSが書かれていないかをLintしたり、

指定した要素に正しい値か適用されてるかをテストする。

- [Cactus](https://github.com/winston/cactus)
- [CSSunit](https://github.com/gagarine/CSSunit)
- [CSS Lint](http://csslint.net/)
- [cssert](https://github.com/thingsinjars/cssert)
- [Hardy](http://hardy.io/)

----

それぞれのチェック方法↓

-v-

### Cactus

ブラウザ上でテストする(要Ruby on Rails)

テストケースはJSで書く


```javascript
Cactus.expect(".header", "font-size").toEqual("24px");
```
<small>.headerのfont-sizeが24pxかどうかをチェック</small>

-v-

### CSSunit

Qunitを使用したCSSのテストツール

Qunitと一緒で、ブラウザでテストをする

```
var actual = $('h2').css('color');
module('Simple CSS');
test("title color blue", 1, function() {
  same( actual, 'rgb(0, 128, 0)', 'message' );
});
```

<small>h2のcolorがrgb(0, 128, 0)と同じかをチェック</small>

-v-

### CSS Lint

propertyやvalueに不正がないかをチェック

コマンドライン上で動かす

```css
nav { paddling: 250px; } nav ul { background: #red; }
```

```sh
$ csslint test.css
csslint: There are 3 problems in test.css.

test.css
1: warning at line 5, col 3
Unknown property 'paddling'.
  paddling: 250px;

test.css
2: warning at line 7, col 1
Rule is empty.
nav ul {

test.css
3: error at line 8, col 15
Expected a hex color but found '#red' at line 8, col 15.
  background: #red;
```

-v-

### cssert

HTMLでテストケースを書いてチェックする

ブラウザ上でも、コマンドライン上でも動かせる。

![ss1.png](img/ss1.png)

<small>pxgrid.comのh3が[base.css](http://www.pxgrid.com/common/css/base.css)の内容が反映されているかをチェック</small>

```sh
$ ./cssert testcases.html
--
h3 : Passed
```

-v-

### Hardy

NodeJSとSeleniumを使用したテストツール

```
Feature: Website layout test
As a user I want visual consistency on the http://csste.st/ website

Scenario: Content layout
Given I visit "http://csste.st/"
Then "section > p" should have "color" of "rgb(68, 68, 68)"
```

```
Hardy v0.0.11
CSS Utils Steps Loaded
CSS Steps Loaded
Generic Steps Loaded
Loading browser firefox
...Shutting down browser


1 scenario (1 passed)
3 steps (3 passed)
firefox success
```

---

## 画像比較をする

何かアクション(CSSの変更、要素をクリックなど)を

起こす前と起こした後の画像を比較します。

目視確認の手間を減らすことが出来る（と思う）

- [PhantomCSS](https://github.com/Huddle/PhantomCSS) ※PhantomJSが必要
- [CSSCritic](http://csste.st/tools/csscritic.html)

----

PhantomCSSを使ってみた↓

-v-

### PhantomCSS (1)

<small>www.pxgrid.netの業務案内をクリックして正しくカレント表示しているかをチェックしたい</small>

```sh
$ git clone git@github.com:Huddle/PhantomCSS.git
$ cd PhantomCSS
$ cp demo/testsuite.js demo/testsuite2.js
$ vi demo/testsuite2.js
```


```javascript
/*
	Initialise CasperJs
*/

phantom.casperPath = 'CasperJs';
phantom.injectJs('jquery.js');

var casper = require('casper').create({
	viewportSize: {
		width: 1027,
		height: 800
	}
});

//Require and initialise PhantomCSS module

var phantomcss = require('./phantomcss.js');
phantomcss.init({
	screenshotRoot: './screenshots',
	failedComparisonsRoot: './failures'
});

//The test scenario

casper.
	start( "http://www.pxgrid.com/" ).
	then(function(){
		casper.click('#page-header a[href="#service"]');
		casper.waitForSelector('#service:not([style*="display: none"])',
			function success(){
				phantomcss.screenshot('#page-header a[href="#service"]', 'show service page');
			},
			function timeout(){
				casper.test.fail('not show service page');
			}
		);
	});

// End tests and compare screenshots

casper.
	then( function now_check_the_screenshots(){
		phantomcss.compareAll();
	}).
	run( function end_it(){
		console.log('\nTHE END.');
		phantom.exit(phantomcss.getExitStatus());
	});
```

-v-

### PhantomCSS (2)

- 1回目の実行時に正しい状態のSSを取ります。

```
$ phantomjs demo/testsuite2.js

Must be your first time?
Some screenshots have been generated in the directory ./screenshots
This is your 'baseline', check the images manually. If they're wrong, delete the images.
The next time you run these tests, new screenshots will be taken.  These screenshots will be compared to the original.
If they are different, PhantomCSS will report a failure.

THE END.
```

-v-

### PhantomCSS (3)

- 再度実行するとSSを取って差分をチェックします。問題がなければPASSされます
- 特に変更してないのでエラーは出ません

```
$ phantomjs demo/testsuite2.js
PASS No changes found for screenshot ./screenshots/show service page_0.png

PhantomCSS found: 1 tests.
None of them failed. Which is good right?
If you want to make them fail, go change some CSS - weirdo.

THE END.
```

-v-

### PhantomCSS (4)

- common/css/base.cssに変更を加えてみます

```css
#page-header nav ul li a.current{
	/*...some code...*/
	background-color: red;
	//background-color: #eee;
	/*...some code...*/
}
```

-v-

### PhantomCSS (5)

- CSSに変更を加えて差分が出た場合エラーが発生します。

```sh
$ phantomjs demo/testsuite2.js
FAIL Visual change found for screenshot ./screenshots/show service page_0.png (92.36% mismatch)
#    type: fail
#    subject: false

PhantomCSS found: 1 tests.
1 of them failed.
PhantomCSS has created some images that try to show the difference (in the directory ./failures). Fuchsia colored pixels indicate a difference betwen the new and old screenshots.

THE END.
```
![img/show service page_0.png](img/show%20service%20page_0.png)
![img/show service page_0.fail.png](img/show%20service%20page_0.fail.png)
![img/show service page_0.diff.png](img/show%20service%20page_0.diff.png)

<small>左から　正しい画像、どこに差が出ているかの画像、CSS変更後の画像</small>

---

## @functionのテストツール

Sassの中にJSみたいなテストを書く。

@functionが正しい値を返せてるかをテストする

- [true](https://github.com/ericam/true)
- [bootcamp](https://github.com/tctcl/bootcamp) 

----

trueとbootcampのテストの書き方↓

-v-

### true (1)

Sassの中にテストを書く。

正しい値かどうかをassert-equalを使ってチェック

```
@function calculateRem($size) {
	$root-fontSize: 10px;
	$remSize: ($size / $root-fontSize);
	@return #{$remSize}rem;
}

@include test-module('Font Functions') {

	@include test('[function] calculateRem()') {
		@include assert-equal(calculateRem(16px), "1.6rem", '正しいremになっている');
	}

}
.hoge {
	font-size: calculateRem(16px);
}
```

-v-

### true (2)

SassをコンパイルするとコマンドラインとCSSに結果が出力される

```
$be compass compile
/Users/geckotang/Work/sandbox/true/sass/true/_messages.scss:11 DEBUG: SUMMARY: 1 Tests, 1 Passed, 0 Failed
overwrite css/test.css
```

```
/*  

### Font Functions ------ */
/*  - [function] calculateRem() (1 Assertions, 1 Passed, 0 Failed) */
/*  
    1 Tests:
    - 1 Passed
    - 0 Failed */
.hoge {
  font-size: 1.6rem;
}
```


-v-

### bootcamp

Jasmineっぽいテストを書くことが出来る。

```
@include describe("Math Power") {
  @include it("should expect positive values to be calculated correctly") {
    @include should( expect( power( 10, 2) ), to( equal(  100 )));
    @include should( expect( power(  2, 2) ), to( equal(    4 )));
    @include should( expect( power(0.5, 2) ), to( equal( 0.25 )));
  }

  @include it("should expect negative values to be calculated correctly") {
    @include should( expect( power( 10, -2) ), to( equal( 0.01 )));
    @include should( expect( power(  2, -2) ), to( equal( 0.25 )));
    @include should( expect( power(0.5, -2) ), to( equal(    4 )));
  }
}
```

```
✔ ✔
2 Tests, 6 assertions, 0 failures, 0 skipped
```

<small>[https://github.com/tctcl/bootcamp/wiki/Example-Test-Suite](https://github.com/tctcl/bootcamp/wiki/Example-Test-Suite)</small>

---

## まとめ

- まだまだ発展途上なCSS(手動||自動)テスト
- 複雑なWebアプリケーションならやってもいいかも
- 簡単なウェブページは必要なさそう
- @functionのテストは実用的かも（そこまで複雑な関数を書くかは置いておいて）
- @mixinのテストは出来ないのかな…うーむ

----
<small>全部試せてなくてすみません…</small>

---

## 参考

- [csste.st](http://csste.st/tools/)
- [CSS Regression Testing](http://tldr.huddle.com/blog/css-testing/)
- [4 tools for automatic CSS](http://www.creativebloq.com/css3/4-tools-automatic-css-testing-7133777)
- [Automatic CSS Testing](http://css-tricks.com/automatic-css-testing/)

