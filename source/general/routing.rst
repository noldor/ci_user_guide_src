###############
URIルーティング
###############

一般的に、 URL 文字列とそれに対応する
コントローラクラス/メソッドは 1 対 1 の関係にあります。URI
のセグメントは通常、つぎのパターンに従います::

	example.com/class/function/id/

しかしながらいくつかのケースでは、この関係を再マッピングしてたいことでしょう、
そのため
URL に対応するものではなく別のクラス/メソッドが呼び出されるようにすることができます。

たとえば、 URL がつぎのプロトタイプを持つようにしたいとしましょう::

	example.com/product/1/
	example.com/product/2/
	example.com/product/3/
	example.com/product/4/

通常、 URL の第 2 セグメントはメソッド名となっていますが、
上記の例では代わりにプロダクト ID を持っています。
これをクリアするために、 CodeIgniter は URI ハンドラを再マップすることができます。

独自のルーティングルールを設定する
==================================

ルーティングルールは *application/config/routes.php* ファイルで定義されています。
その中で、独自のルーティング基準を指定することを可能にする ``$route`` という配列があります。
ルートはワイルドカードや正規表現を使用して
指定することができます。

ワイルドカード
==============

典型的なワイルドカードのルートは次のようになります::

	$route['product/:num'] = 'catalog/product_lookup';

ルートには、配列のキーには URI が一致するもの、
配列の値は再ルーティングすべき先が含まれています。
上記の例では、リテラル文字「 product 」が URL の最初のセグメントにあり、
第 2 セグメントに数値がある場合、
「 catalog 」クラスと「 product_lookup 」メソッドが代わりに使用されます。

リテラル値の一致または 2 つのワイルドカードの種類を使用することができます:

**(:num)** は数字のみを含むセグメントと一致します。
**(:any)** は任意の文字を含むセグメントと一致します (セグメント区切り文字である「 / 」を除く) 。

.. note:: ワイルドカードは実際には正規表現の別名です、
	それぞれ **:any** は **[^/]+** に、 **:num** は **[0-9]+**
	に変換されます。

.. note:: ルートは、それらの定義順で実行されます。
	上のルートは常に下のものよりも優先されます。

.. note:: ルートルールはフィルターではありません！　例えばルール
	「 foo/bar/(:num) 」はそれが正しいルートである場合、コントローラ *Foo* とメソッド
	*bar* が数値以外の値とともに呼ばれるのを防ぐことは
	できません。

例
==

ここではいくつかのルーティング例を示します::

	$route['journals'] = 'blogs';

最初のセグメントで単語「 journals 」を含む URL は
「 blogs 」クラスに再マッピングされます。

::

	$route['blog/joe'] = 'blogs/users/34';

blog/joe セグメントを含むURLは、「 blogs 」
クラスと「 users 」メソッドに再マッピングされます。 ID は「 34 」に設定されます。

::

	$route['product/(:any)'] = 'catalog/product_lookup';

最初のセグメントに「 product 」と第 2 セグメントに何かがある URL は、
「 catalog 」クラスと「 product_lookup 」メソッドに
再マッピングされます。

::

	$route['product/(:num)'] = 'catalog/product_lookup_by_id/$1';

最初のセグメントに「 product 」と第 2 セグメントに数字がある URL は
「 catalog 」クラスと
「 product_lookup_by_id 」メソッドに再マッピングされ
変数が渡されます。

.. important:: 最初と最後のスラッシュは付けないでください。

正規表現
========

ご希望の場合、自分のルーティングルールを定義するために正規表現を使用することができます。
任意の正しい正規表現と後方参照が許可されています。

.. note:: 後方参照を使用する場合は、二重バックスラッシュの構文ではなく
	ドルの構文を使用する必要があります。

典型的なルートの正規表現は次のようになります::

	$route['products/([a-z]+)/(\d+)'] = '$1/id_$2';

上記の例では、 product/shirts/123 の URI は
「 shirts 」コントローラクラスと「 id_123 」メソッドを代わりに呼び出します。

正規表現を使用すると、複数のセグメントをひとつとしてキャッチすることができます。
たとえば、ユーザがウェブアプリケーションのパスワード保護された領域にアクセスし、
彼らがログインした後に同じページに戻るようリダイレクトしたい場合、
この例が役に立つでしょう::

	$route['login/(.+)'] = 'auth/login/$1';

.. note:: 上記の例では、 ``$1`` プレースホルダに
	スラッシュが含まれている場合、
	``Auth::login()`` に渡される際にまた複数のパラメータに分割されます。

正規表現を知らず詳細を学びたい人には、
`regular-expressions.info <http://www.regular-expressions.info/>`_
は良い手掛かりになるでしょう。

.. note:: 正規表現とワイルドカードは混ぜて使うことができます。

コールバック
============

PHP 5.3 以上を使用している場合、後方参照を処理するために、
通常のルーティングルールの代わりにコールバックを使用することができます。例::

	$route['products/([a-zA-Z]+)/edit/(\d+)'] = function ($product_type, $id)
	{
		return 'catalog/product_edit/' . strtolower($product_type) . '/' . $id;
	};

ルートでの HTTP 動詞の利用
==========================

HTTP 動詞 (リクエストメソッド) をルーティングルール定義に使用することができます。
RESTful なアプリケーションを構築する際に特に便利です。標準の HTTP
動詞 ( GET 、 PUT 、 POST 、 DELETE 、 PATCH ) またはカスタムしたもの (例えば PURGE )を使用することができます。 HTTP 動詞のルールは
大文字小文字を区別しません。必要なことは、ルート配列のキーとして動詞を追加することです。
例::

	$route['products']['put'] = 'product/insert';

上記の例では、 URI 「 products 」への PUT リクエストは ``Product::insert()``
コントローラメソッドを呼び出します。

::

	$route['products/(:num)']['DELETE'] = 'product/delete/$1';

最初のセグメントとして「 product 」と第 2 セグメントに数値を持つ URL への DELETE リクエストは
``Product::delete()`` にマッピングされ、数値は第 1 引数として渡されます。

HTTP動詞の利用はもちろん、任意です。

予約済みルート
==============

予約済みのルートが 3 つあります::

	$route['default_controller'] = 'welcome';

このルートポイントは URI になにもデータがない場合に呼び出されるべきアクションです、
これはルート URL を読み込んだ時のケースになります。
この設定は **controller/method** の値を指定できますが、指定しない場合は ``index()``
がデフォルトになります。上記の例では
``Welcome::index()`` が呼び出されることになります。

.. note:: この設定の一部としてディレクトリを使用することはできません！

常にデフォルトルートを持つことが推奨されます、さもなくば 404
ページがデフォルトになります。

::

	$route['404_override'] = '';

このルートには、要求されたコントローラが見つからない場合にロードされるべき
コントローラクラスを指定します。これはデフォルトの 404 エラーページを
オーバーライドします。「 default_controller 」と同じディレクトリごとのルールは
ここにも適用されます。

これは ``show_404()`` 関数には影響しません。
それは引き続き *application/views/errors/error_404.php* にあるデフォルトの *error_404.php*
ファイルをロードします。

::

	$route['translate_uri_dashes'] = FALSE;

真偽値なので明らかですが、これは正確にルートではありません。
このオプションはコントローラとメソッドの URI セグメントにおいて自動的にダッシュ「 - 」を
アンダースコアに置換するもので、
そうすることが必要な時にルートに設定追加する必要をなくすものです。
これは必要なものです。なぜならダッシュはクラスやメソッド名の文字としては有効ではなく、
それを使用しようとした場合に fatal error の原因となるからです。

.. important:: 予約済みのルートは、あらゆるワイルドカードや正規表現のルートの
	前に来なければなりません。
