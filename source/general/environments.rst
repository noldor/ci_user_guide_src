################
複数環境への対応
################

開発者は多くの場合、開発または本番環境で実行されているかどうかに応じて、
アプリケーションに異なるシステムの振る舞いを望みます。
たとえば、詳細なエラー出力はアプリケーションの開発中は有用であろうものですが、
それは「稼働中」にはセキュリティ上の問題を引き起こす可能性が
あります。

ENVIRONMENT 定数
================

デフォルトでは CodeIgniter には環境定数セットがついてきます、
``$_SERVER['CI_ENV']`` で提供された値か、なければデフォルトでは
「 development 」です。index.php の上のほうにつぎのものがあるでしょう::

	define('ENVIRONMENT', isset($_SERVER['CI_ENV']) ? $_SERVER['CI_ENV'] : 'development');

このサーバ変数は、 .htaccess ファイル、つまり Apache の
`SetEnv <https://httpd.apache.org/docs/2.2/mod/mod_env.html#setenv>`_. で設定することができます。
別の方法で nginx や他のサーバでも利用可能ですし、もしくはこのロジックを完全に取り除いてサーバの
IP アドレスに基づいて定数を設定することもできます。

これはいくつかの基本的なフレームワークの動作に影響を与えることに加え (次のセクションを参照) 、
実行している環境を区別するために
あなた独自の制作物のなかでこの定数を使用することができます。

デフォルトのフレームワーク動作への影響
======================================

CodeIgniter のシステム内にはいくつか ENVIRONMENT
定数が使用されている場所があります。このセクションではデフォルトのフレームワークの動作が
どのように影響を受けるかについて説明します。

エラーレポーティング
--------------------

ENVIRONMENT 定数を「 development 」に設定すると、
発生するすべての PHP エラーがブラウザにレンダリングされます。
逆に、定数を「 production 」に設定すると、すべてのエラー出力が無効になります。
本番環境でにエラー報告を無効にすることは :doc:`良いセキュリティプラクティス
<security>` です。

設定ファイル
------------

オプションで、 CodeIgniter
に環境固有の設定ファイルをロードさせることができます。
これはAPIキーのような、複数の環境間で異なるものを管理するために便利です。
:doc:`設定クラス
<../libraries/config>` ドキュメントの環境セクションで詳細に説明されています。