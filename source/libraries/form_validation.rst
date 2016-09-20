############################
フォームバリデーション(検証)
############################

CodeIgniter では包括的なフォーム検証とデータ準備のクラスを提供します。
あなたが書くつもりのコード量を最小限に抑えることができるでしょう。

.. contents:: Page Contents

****
概要
****

CodeIgniter のデータ検証に対するアプローチを説明する前に、
理想的なシナリオを説明してみましょう:

#. フォームが表示されます。
#. それを記入し、送信します。
#. 無効なものを送信したなら、つまりもしかしたら必須項目を逃した場合は、
   問題を説明するエラーメッセージとともに
   フォームはあなたのデータを再表示します。
#. 有効なフォームを送信するまで、このプロセスが継続します。

受信側では、スクリプトは次のことを満たす必要があります:

#. 必須データを確認します。
#. データが正しい型であり、正しく基準を満たしていることを確認します。
   例えばユーザ名が送信された場合、
   許可された文字だけを含むように検証する必要があります。
   これは最小長より大きく、最大長を超えてはなりません。
   ユーザ名は他の誰かの既存ユーザ名は使えず、またはおそらく予約語も使えません。
   など。
#. セキュリティのためにデータをサニタイズします。
#. 必要に応じてデータをプリフォーマットします
   （データはトリミングを必要とする？　HTML のエンコードは？　など）。
#. データベースに挿入するためのデータを準備します。

上記のプロセスはひどく複雑なものではありませんが、
通常はかなりの量コードを必要とします。エラーメッセージを表示するためには
通常、様々な制御構造がフォームの HTML 内に配置されます。
フォームバリデーションは作成は簡単ながら、
一般的には非常に面倒で退屈な実装作業になります。

************************************
フォームバリデーションチュートリアル
************************************

以下は CodeIgniter のフォームバリデーションを実装するための
「ハンズオン」チュートリアルです。

フォームバリデーションを実装するためには次の 3 つが必要になります:

#. フォームを持つ :doc:`ビュー <../general/views>` ファイル。
#. 送信成功の時に表示される「成功」メッセージを含む
   ビューファイル。
#. 送信されたデータを受信および処理する :doc:`コントローラ <../general/controllers>`
   メソッド。

それではサンプルとしてメンバー登録フォームを使い、
これらの 3 つを作成してみましょう。

入力フォーム
============

テキストエディタを使用して、 myform.php というフォームを作成します。
それには application/views/ フォルダにこのコードを配置して保存します::

	<html>
	<head>
	<title>私のフォーム</title>
	</head>
	<body>

	<?php echo validation_errors(); ?>

	<?php echo form_open('form'); ?>

	<h5>ユーザ名</h5>
	<input type="text" name="username" value="" size="50" />

	<h5>パスワード</h5>
	<input type="text" name="password" value="" size="50" />

	<h5>パスワード確認</h5>
	<input type="text" name="passconf" value="" size="50" />

	<h5>メールアドレス</h5>
	<input type="text" name="email" value="" size="50" />

	<div><input type="submit" value="Submit" /></div>

	</form>

	</body>
	</html>

成功ページ
==========

テキストエディタを使用して、 formsuccess.php というフォームを作成します。
それには application/views/ フォルダにこのコードを配置して保存します::

	<html>
	<head>
	<title>私のフォーム</title>
	</head>
	<body>

	<h3>あなたのフォームは送信成功しました！</h3>

	<p><?php echo anchor('form', 'もういっかい！'); ?></p>

	</body>
	</html>

コントローラ
============

テキストエディタを使用して、 Form.php というコントローラを作成します。
それには application/controllers フォルダにこのコードを配置して保存します::

	<?php

	class Form extends CI_Controller {

		public function index()
		{
			$this->load->helper(array('form', 'url'));

			$this->load->library('form_validation');

			if ($this->form_validation->run() == FALSE)
			{
				$this->load->view('myform');
			}
			else
			{
				$this->load->view('formsuccess');
			}
		}
	}

動かしてみよう！
================

フォームを試すには次のようなURLを使ってサイトを開いてください::

	example.com/index.php/form/

フォームを送信すると、単にフォームがリロードされるはずです。
それはまだ検証ルールを設定していないためです。

**フォームバリデーションクラスに何の検証も指示していないので、
デフォルトの FALSE（ブール偽）を返します。 ``run()`` メソッドはあなたのルールを適用でき、
1 つも失敗しなかった場合にのみ TRUE
を返します。**

解説
====

上記のページについて、いくつかのことに気付いたことでしょう。:

このフォーム（myform.php）は標準的な Web フォームですが、 2 つの例外があります:

#. フォームの開きタグを作成するために、フォームヘルパーを使用しています。
   技術的には、これは必須ではありません。標準の HTML を使用してフォームを作成することもできます。
   しかしながらヘルパーを使用する利点として、
   config ファイル内の URL に基づいてアクション URL を生成することができます。
   これはあなたのURLを変更する際に、アプリケーションをよりポータブルにしてくれます。
#. フォームの一番上のところで、次の関数呼び出しに気付くでしょう:
   ::

	<?php echo validation_errors(); ?>

   この関数は、バリデータによって戻されたすべてのエラーメッセージを返します。
   メッセージがない場合、空の文字列を返します。

コントローラ（Form.php）には 1 つのメソッドがあります: ``index()`` です。
このメソッドはバリデーションクラスを初期化し、ビューファイルで使用されるフォームヘルパーと URL ヘルパーをロードします。
また、バリデーションルーチンを実行します。
検証が成功したかどうかに基づいて、
フォームと成功ページのどちらかを表示します。

.. _setting-validation-rules:

検証ルールを設定する
====================

CodeIgniter は、与えられたフィールドに必要なだけの多くの検証ルールを設定することができ、
その順序でカスケード処理し、
さらには同時にフィールドデータの準備と前処理をすることができます。
検証ルールを設定するには ``set_rules()`` メソッドを使用します::

	$this->form_validation->set_rules();

上のメソッドは、入力として **3 つ** のパラメータを取ります:

#. フィールド名 - フォームフィールドを与えた正確な名前。
#. このフィールドの「人間向け」の名前。エラーメッセージに挿入されます。
   たとえば、フィールドに「user」と名付けた場合、
   人間向けの名前として「ユーザ名」と名付けるかもしれません。
#. このフォームフィールドの検証ルール。
#. （オプション）このフィールドに指定された任意のルールにカスタムエラーメッセージを設定します。指定されない場合、デフォルトのエラーメッセージを使用します。

.. note:: 言語ファイルに格納されているフィールド名をご希望の場合、
	:ref:`translating-field-names` を参照してください。

ここで一例を示しましょう。コントローラ（Form.php）で、
バリデーション初期化メソッドの直下にこのコードを追加します::

	$this->form_validation->set_rules('username', 'ユーザ名', 'required');
	$this->form_validation->set_rules('password', 'パスワード', 'required');
	$this->form_validation->set_rules('passconf', 'パスワード確認', 'required');
	$this->form_validation->set_rules('email', 'メールアドレス', 'required');

コントローラは次のようになります::

	<?php

	class Form extends CI_Controller {

		public function index()
		{
			$this->load->helper(array('form', 'url'));

			$this->load->library('form_validation');

			$this->form_validation->set_rules('username', 'ユーザ名', 'required');
			$this->form_validation->set_rules('password', 'パスワード', 'required',
				array('required' => '%s は必須です。')
			);
			$this->form_validation->set_rules('passconf', 'パスワード確認', 'required');
			$this->form_validation->set_rules('email', 'メールアドレス', 'required');

			if ($this->form_validation->run() == FALSE)
			{
				$this->load->view('myform');
			}
			else
			{
				$this->load->view('formsuccess');
			}
		}
	}

いま、フィールドが空欄のままフォームを送信すると、エラーメッセージが表示されるはずです。
すべてのフィールドを埋めてフォームを送信すると、
成功ページが表示されるでしょう。

.. note:: エラーが存在するとき、フォームフィールドはまだ入力データで埋めなおされず空欄のままです。
	すぐあとで説明します。

配列を使った検証ルールの指定
============================

先に進む前に、次のことに注意すべきです。
単一の操作ですべてのルールを設定したい場合は、配列を渡すことができます。
この方法を使う場合、つぎに示されているように、配列のキーに名前を付ける必要があります::

	$config = array(
		array(
			'field' => 'username',
			'label' => 'ユーザ名',
			'rules' => 'required'
		),
		array(
			'field' => 'password',
			'label' => 'パスワード',
			'rules' => 'required',
			'errors' => array(
				'required' => '%s は必須です。',
			),
		),
		array(
			'field' => 'passconf',
			'label' => 'パスワード確認',
			'rules' => 'required'
		),
		array(
			'field' => 'email',
			'label' => 'メールアドレス',
			'rules' => 'required'
		)
	);

	$this->form_validation->set_rules($config);

ルールの連結（カスケード）
==========================

CodeIgniter では複数のルールをパイプで一緒につなげることができます。試してみましょう。
ルール設定メソッドの第 3 パラメータに指定するルールを変更します。このように::

	$this->form_validation->set_rules(
		'username', 'ユーザ名',
		'required|min_length[5]|max_length[12]|is_unique[users.username]',
		array(
			'required'	=> '%s を入力していません。',
			'is_unique'	=> '%s はすでに存在します。'
		)
	);
	$this->form_validation->set_rules('password', 'パスワード', 'required');
	$this->form_validation->set_rules('passconf', 'パスワード確認', 'required|matches[password]');
	$this->form_validation->set_rules('email', 'メールアドレス', 'required|valid_email|is_unique[users.email]');

上のコードは次のルールを設定します:

#. ユーザ名フィールドは 5 文字未満または
   12 文字を超えることはありません。
#. パスワードフィールドは、パスワード確認フィールドと一致する必要があります。
#. メールアドレスフィールドは有効なメールアドレスを含める必要があります。

試してみましょう！　まちがったデータでフォームを送信すると、
新しいルールに対応する新しいエラーメッセージが表示されます。
利用可能なルールは多数あり、バリデーションリファレンスでそれらについて読むことができます。

.. note:: 文字列のかわりに配列で ``set_rules()`` にルールを渡すことができます。
	例::

	$this->form_validation->set_rules('username', 'ユーザ名', array('required', 'min_length[5]'));

データの整形
============

上記で使用しているようなバリデーションメソッドに加え、
様々な方法でデータを整形することもできます。
たとえば、次のようなルールを設定することができます::

	$this->form_validation->set_rules('username', 'ユーザ名', 'trim|required|min_length[5]|max_length[12]');
	$this->form_validation->set_rules('password', 'パスワード', 'trim|required|min_length[8]');
	$this->form_validation->set_rules('passconf', 'パスワード確認', 'trim|required|matches[password]');
	$this->form_validation->set_rules('email', 'メールアドレス', 'trim|required|valid_email');

上の例では、フィールドを「トリミング」し、必要なところでは文字列長をチェックし、
パスワードフィールドの両方が一致することを確認しています。

**あらゆる PHP ネイティブ関数のうちパラメータを 1 つ受けとるものは、ルールとして使用することができます。
``htmlspecialchars()`` 、 ``trim()`` などです。**

.. note:: 一般的には、バリデーションルールの
	**後で** データ整形機能を使用したいことでしょう。
	エラーがある場合にオリジナルのデータをフォームに表示させるためです。

フォームの再表示（データの引き継ぎ）
====================================

ここまではエラーのみを取り扱ってきました。
ここからは送信されたデータでフォームフィールドを埋めなおしていきましょう。CodeIgniter
ではそうするためのヘルパー関数をいくつか提供しています。
最も一般的に使用されるのは、次のものです::

	set_value('field name')

myform.php ビューファイルを開き、
:php:func:`set_value()` 関数を使用して各フィールドの **value** を変えていきましょう:

**:PHP:FUNC:`set_value()` 関数呼び出しに各フィールド名を含めることを
忘れないでください！**

::

	<html>
	<head>
	<title>私のフォーム</title>
	</head>
	<body>

	<?php echo validation_errors(); ?>

	<?php echo form_open('form'); ?>

	<h5>ユーザ名</h5>
	<input type="text" name="username" value="<?php echo set_value('username'); ?>" size="50" />

	<h5>パスワード</h5>
	<input type="text" name="password" value="<?php echo set_value('password'); ?>" size="50" />

	<h5>パスワード確認</h5>
	<input type="text" name="passconf" value="<?php echo set_value('passconf'); ?>" size="50" />

	<h5>メールアドレス</h5>
	<input type="text" name="email" value="<?php echo set_value('email'); ?>" size="50" />

	<div><input type="submit" value="送信" /></div>

	</form>

	</body>
	</html>

さて、ページをリロードしてエラーを起こすようにフォームを送信します。
フォームフィールドはいま、埋めなおされたことでしょう。

.. note:: 下記の :ref:`class-reference` セクションには
	<select>メニュー、ラジオボタン、およびチェックボックスを埋めなおす
	メソッドがあります。

.. important:: フォームフィールドの name に配列を使用する場合は、
	関数に配列としてそれを指定する必要があります。例::

	<input type="text" name="colors[]" value="<?php echo set_value('colors[]'); ?>" size="50" />

詳細については下記の :ref:`using-arrays-as-field-names` セクションを参照してください。

Callbacks: Your own Validation Methods
======================================

The validation system supports callbacks to your own validation
methods. This permits you to extend the validation class to meet your
needs. For example, if you need to run a database query to see if the
user is choosing a unique username, you can create a callback method
that does that. Let's create an example of this.

In your controller, change the "username" rule to this::

	$this->form_validation->set_rules('username', 'Username', 'callback_username_check');

Then add a new method called ``username_check()`` to your controller.
Here's how your controller should now look::

	<?php

	class Form extends CI_Controller {

		public function index()
		{
			$this->load->helper(array('form', 'url'));

			$this->load->library('form_validation');

			$this->form_validation->set_rules('username', 'Username', 'callback_username_check');
			$this->form_validation->set_rules('password', 'Password', 'required');
			$this->form_validation->set_rules('passconf', 'Password Confirmation', 'required');
			$this->form_validation->set_rules('email', 'Email', 'required|is_unique[users.email]');

			if ($this->form_validation->run() == FALSE)
			{
				$this->load->view('myform');
			}
			else
			{
				$this->load->view('formsuccess');
			}
		}

		public function username_check($str)
		{
			if ($str == 'test')
			{
				$this->form_validation->set_message('username_check', 'The {field} field can not be the word "test"');
				return FALSE;
			}
			else
			{
				return TRUE;
			}
		}

	}

Reload your form and submit it with the word "test" as the username. You
can see that the form field data was passed to your callback method
for you to process.

To invoke a callback just put the method name in a rule, with
"callback\_" as the rule **prefix**. If you need to receive an extra
parameter in your callback method, just add it normally after the
method name between square brackets, as in: ``callback_foo[bar]``,
then it will be passed as the second argument of your callback method.

.. note:: You can also process the form data that is passed to your
	callback and return it. If your callback returns anything other than a
	boolean TRUE/FALSE it is assumed that the data is your newly processed
	form data.

Callable: Use anything as a rule
================================

If callback rules aren't good enough for you (for example, because they are
limited to your controller), don't get disappointed, there's one more way
to create custom rules: anything that ``is_callable()`` would return TRUE for.

Consider the following example::

	$this->form_validation->set_rules(
		'username', 'Username',
		array(
			'required',
			array($this->users_model, 'valid_username')
		)
	);

The above code would use the ``valid_username()`` method from your
``Users_model`` object.

This is just an example of course, and callbacks aren't limited to models.
You can use any object/method that accepts the field value as its' first
parameter. You can also use an anonymous function::

	$this->form_validation->set_rules(
		'username', 'Username',
		array(
			'required',
			function($value)
			{
				// Check $value
			}
		)
	);

Of course, since a Callable rule by itself is not a string, it isn't
a rule name either. That is a problem when you want to set error messages
for them. In order to get around that problem, you can put such rules as
the second element of an array, with the first one being the rule name::

	$this->form_validation->set_rules(
		'username', 'Username',
		array(
			'required',
			array('username_callable', array($this->users_model, 'valid_username'))
		)
	);

Anonymous function version::

	$this->form_validation->set_rules(
		'username', 'Username',
		array(
			'required',
			array(
				'username_callable',
				function($str)
				{
					// Check validity of $str and return TRUE or FALSE
				}
			)
		)
	);

.. _setting-error-messages:

エラーメッセージの設定
======================

標準のエラーメッセージはすべて、次の言語ファイルの中にあります:
**system/language/english/form_validation_lang.php**

あなた独自のグローバルなカスタムメッセージを
ルールに設定するには、
**application/language/english/form_validation_lang.php** で言語ファイル上書き/拡張するか（これについては
:doc:`言語クラス <language>` のドキュメントを読んでください）、
または次のメソッドを使用します::

	$this->form_validation->set_message('rule', 'エラーメッセージ');

もし特定のルールかつ特定のフィールド用のカスタムエラーメッセージを設定する必要がある場合は、
set_rules() メソッドを使用します::

	$this->form_validation->set_rules('field_name', 'フィールド名', 'rule1|rule2|rule3',
		array('rule2' => 'この field_name の rule2 に使用するエラーメッセージ')
	);

ルールのところはエラーを表示したい特定のルール名に対応し、
エラーメッセージのところは表示したいテキストです。

フィールドの「人間向け」の名前、
またはいくつかのルールが許可しているオプションパラメータ（max_length など）を含めたい場合は、
**{field}** タグと **{param}** タグをメッセージに追加することができます::

	$this->form_validation->set_message('min_length', '{field} は少なくとも {param} 文字必要です。');

人間向けの名前「ユーザ名」と min_length[5] のルールのフィールドでは、
次のエラーが表示されます: "ユーザ名 は少なくとも 5 文字必要です。"

.. note:: **%s** をエラーメッセージ内に使用する古い `sprintf()` の方法はまだ動作しますが、
	しかしそれは上記のタグを上書きします。
	どちらか一方だけを使用すべきです。

上記のコールバックルール例では、エラーメッセージはメソッドの名前を渡すことによって設定されます
（「callback\_」プレフィックスは不要です）::

	$this->form_validation->set_message('username_check')

.. _translating-field-names:

フィールド名の翻訳
==================

``set_rules()`` メソッドに渡される「人間向け」の名前を言語ファイルに保持したい場合、
つまり名前を翻訳できるようにしたい場合、
方法は次のとおりです:

まず、「人間向け」の名前をにプレフィックス **lang:** をつけます。この例のように:

	 $this->form_validation->set_rules('first_name', 'lang:first_name', 'required');

次に、言語ファイルの配列に名前を格納します
（プレフィックスなし）::

	$lang['first_name'] = '名前';

.. note:: CI によって自動的にロードされない言語ファイルで
	配列の項目を追加する場合は、
	コントローラでロードすることを忘れないでください::

	$this->lang->load('file_name');

言語ファイルに関する詳細情報については :doc:`言語クラス <language>`
のページを見てください。

.. _changing-delimiters:

Changing the Error Delimiters
=============================

By default, the Form Validation class adds a paragraph tag (<p>) around
each error message shown. You can either change these delimiters
globally, individually, or change the defaults in a config file.

#. **Changing delimiters Globally**
   To globally change the error delimiters, in your controller method,
   just after loading the Form Validation class, add this::

      $this->form_validation->set_error_delimiters('<div class="error">', '</div>');

   In this example, we've switched to using div tags.

#. **Changing delimiters Individually**
   Each of the two error generating functions shown in this tutorial can
   be supplied their own delimiters as follows::

      <?php echo form_error('field name', '<div class="error">', '</div>'); ?>

   Or::

      <?php echo validation_errors('<div class="error">', '</div>'); ?>

#. **Set delimiters in a config file**
   You can add your error delimiters in application/config/form_validation.php as follows::

      $config['error_prefix'] = '<div class="error_prefix">';
      $config['error_suffix'] = '</div>';

Showing Errors Individually
===========================

If you prefer to show an error message next to each form field, rather
than as a list, you can use the :php:func:`form_error()` function.

Try it! Change your form so that it looks like this::

	<h5>Username</h5>
	<?php echo form_error('username'); ?>
	<input type="text" name="username" value="<?php echo set_value('username'); ?>" size="50" />

	<h5>Password</h5>
	<?php echo form_error('password'); ?>
	<input type="text" name="password" value="<?php echo set_value('password'); ?>" size="50" />

	<h5>Password Confirm</h5>
	<?php echo form_error('passconf'); ?>
	<input type="text" name="passconf" value="<?php echo set_value('passconf'); ?>" size="50" />

	<h5>Email Address</h5>
	<?php echo form_error('email'); ?>
	<input type="text" name="email" value="<?php echo set_value('email'); ?>" size="50" />

If there are no errors, nothing will be shown. If there is an error, the
message will appear.

.. important:: If you use an array as the name of a form field, you
	must supply it as an array to the function. Example::

	<?php echo form_error('options[size]'); ?>
	<input type="text" name="options[size]" value="<?php echo set_value("options[size]"); ?>" size="50" />

For more info please see the :ref:`using-arrays-as-field-names` section below.

Validating an Array (other than $_POST)
=======================================

Sometimes you may want to validate an array that does not originate from ``$_POST`` data.

In this case, you can specify the array to be validated::

	$data = array(
		'username' => 'johndoe',
		'password' => 'mypassword',
		'passconf' => 'mypassword'
	);

	$this->form_validation->set_data($data);

Creating validation rules, running the validation, and retrieving error
messages works the same whether you are validating ``$_POST`` data or
another array of your choice.

.. important:: You have to call the ``set_data()`` method *before* defining
	any validation rules.

.. important:: If you want to validate more than one array during a single
	execution, then you should call the ``reset_validation()`` method
	before setting up rules and validating the new array.

For more info please see the :ref:`class-reference` section below.

.. _saving-groups:

************************************************
Saving Sets of Validation Rules to a Config File
************************************************

A nice feature of the Form Validation class is that it permits you to
store all your validation rules for your entire application in a config
file. You can organize these rules into "groups". These groups can
either be loaded automatically when a matching controller/method is
called, or you can manually call each set as needed.

How to save your rules
======================

To store your validation rules, simply create a file named
form_validation.php in your application/config/ folder. In that file
you will place an array named $config with your rules. As shown earlier,
the validation array will have this prototype::

	$config = array(
		array(
			'field' => 'username',
			'label' => 'Username',
			'rules' => 'required'
		),
		array(
			'field' => 'password',
			'label' => 'Password',
			'rules' => 'required'
		),
		array(
			'field' => 'passconf',
			'label' => 'Password Confirmation',
			'rules' => 'required'
		),
		array(
			'field' => 'email',
			'label' => 'Email',
			'rules' => 'required'
		)
	);

Your validation rule file will be loaded automatically and used when you
call the ``run()`` method.

Please note that you MUST name your ``$config`` array.

Creating Sets of Rules
======================

In order to organize your rules into "sets" requires that you place them
into "sub arrays". Consider the following example, showing two sets of
rules. We've arbitrarily called these two rules "signup" and "email".
You can name your rules anything you want::

	$config = array(
		'signup' => array(
			array(
				'field' => 'username',
				'label' => 'Username',
				'rules' => 'required'
			),
			array(
				'field' => 'password',
				'label' => 'Password',
				'rules' => 'required'
			),
			array(
				'field' => 'passconf',
				'label' => 'Password Confirmation',
				'rules' => 'required'
			),
			array(
				'field' => 'email',
				'label' => 'Email',
				'rules' => 'required'
			)
		),
		'email' => array(
			array(
				'field' => 'emailaddress',
				'label' => 'EmailAddress',
				'rules' => 'required|valid_email'
			),
			array(
				'field' => 'name',
				'label' => 'Name',
				'rules' => 'required|alpha'
			),
			array(
				'field' => 'title',
				'label' => 'Title',
				'rules' => 'required'
			),
			array(
				'field' => 'message',
				'label' => 'MessageBody',
				'rules' => 'required'
			)
		)
	);

Calling a Specific Rule Group
=============================

In order to call a specific group, you will pass its name to the ``run()``
method. For example, to call the signup rule you will do this::

	if ($this->form_validation->run('signup') == FALSE)
	{
		$this->load->view('myform');
	}
	else
	{
		$this->load->view('formsuccess');
	}

Associating a Controller Method with a Rule Group
=================================================

An alternate (and more automatic) method of calling a rule group is to
name it according to the controller class/method you intend to use it
with. For example, let's say you have a controller named Member and a
method named signup. Here's what your class might look like::

	<?php

	class Member extends CI_Controller {

		public function signup()
		{
			$this->load->library('form_validation');

			if ($this->form_validation->run() == FALSE)
			{
				$this->load->view('myform');
			}
			else
			{
				$this->load->view('formsuccess');
			}
		}
	}

In your validation config file, you will name your rule group
member/signup::

	$config = array(
		'member/signup' => array(
			array(
				'field' => 'username',
				'label' => 'Username',
				'rules' => 'required'
			),
			array(
				'field' => 'password',
				'label' => 'Password',
				'rules' => 'required'
			),
			array(
				'field' => 'passconf',
				'label' => 'PasswordConfirmation',
				'rules' => 'required'
			),
			array(
				'field' => 'email',
				'label' => 'Email',
				'rules' => 'required'
			)
		)
	);

When a rule group is named identically to a controller class/method it
will be used automatically when the ``run()`` method is invoked from that
class/method.

.. _using-arrays-as-field-names:

***************************
Using Arrays as Field Names
***************************

The Form Validation class supports the use of arrays as field names.
Consider this example::

	<input type="text" name="options[]" value="" size="50" />

If you do use an array as a field name, you must use the EXACT array
name in the :ref:`Helper Functions <helper-functions>` that require the
field name, and as your Validation Rule field name.

For example, to set a rule for the above field you would use::

	$this->form_validation->set_rules('options[]', 'Options', 'required');

Or, to show an error for the above field you would use::

	<?php echo form_error('options[]'); ?>

Or to re-populate the field you would use::

	<input type="text" name="options[]" value="<?php echo set_value('options[]'); ?>" size="50" />

You can use multidimensional arrays as field names as well. For example::

	<input type="text" name="options[size]" value="" size="50" />

Or even::

	<input type="text" name="sports[nba][basketball]" value="" size="50" />

As with our first example, you must use the exact array name in the
helper functions::

	<?php echo form_error('sports[nba][basketball]'); ?>

If you are using checkboxes (or other fields) that have multiple
options, don't forget to leave an empty bracket after each option, so
that all selections will be added to the POST array::

	<input type="checkbox" name="options[]" value="red" />
	<input type="checkbox" name="options[]" value="blue" />
	<input type="checkbox" name="options[]" value="green" />

Or if you use a multidimensional array::

	<input type="checkbox" name="options[color][]" value="red" />
	<input type="checkbox" name="options[color][]" value="blue" />
	<input type="checkbox" name="options[color][]" value="green" />

When you use a helper function you'll include the bracket as well::

	<?php echo form_error('options[color][]'); ?>


**************
Rule Reference
**************

The following is a list of all the native rules that are available to
use:

========================= ========== ============================================================================================= =======================
Rule                      Parameter  Description                                                                                   Example
========================= ========== ============================================================================================= =======================
**required**              No         Returns FALSE if the form element is empty.
**matches**               Yes        Returns FALSE if the form element does not match the one in the parameter.                    matches[form_item]
**regex_match**           Yes        Returns FALSE if the form element does not match the regular expression.                      regex_match[/regex/]
**differs**               Yes        Returns FALSE if the form element does not differ from the one in the parameter.              differs[form_item]
**is_unique**             Yes        Returns FALSE if the form element is not unique to the table and field name in the            is_unique[table.field]
                                     parameter. Note: This rule requires :doc:`Query Builder <../database/query_builder>` to be
                                     enabled in order to work.
**min_length**            Yes        Returns FALSE if the form element is shorter than the parameter value.                        min_length[3]
**max_length**            Yes        Returns FALSE if the form element is longer than the parameter value.                         max_length[12]
**exact_length**          Yes        Returns FALSE if the form element is not exactly the parameter value.                         exact_length[8]
**greater_than**          Yes        Returns FALSE if the form element is less than or equal to the parameter value or not         greater_than[8]
                                     numeric.
**greater_than_equal_to** Yes        Returns FALSE if the form element is less than the parameter value,                           greater_than_equal_to[8]
                                     or not numeric.
**less_than**             Yes        Returns FALSE if the form element is greater than or equal to the parameter value or          less_than[8]
                                     not numeric.
**less_than_equal_to**    Yes        Returns FALSE if the form element is greater than the parameter value,                        less_than_equal_to[8]
                                     or not numeric.
**in_list**               Yes        Returns FALSE if the form element is not within a predetermined list.                         in_list[red,blue,green]
**alpha**                 No         Returns FALSE if the form element contains anything other than alphabetical characters.
**alpha_numeric**         No         Returns FALSE if the form element contains anything other than alpha-numeric characters.
**alpha_numeric_spaces**  No         Returns FALSE if the form element contains anything other than alpha-numeric characters
                                     or spaces.  Should be used after trim to avoid spaces at the beginning or end.
**alpha_dash**            No         Returns FALSE if the form element contains anything other than alpha-numeric characters,
                                     underscores or dashes.
**numeric**               No         Returns FALSE if the form element contains anything other than numeric characters.
**integer**               No         Returns FALSE if the form element contains anything other than an integer.
**decimal**               No         Returns FALSE if the form element contains anything other than a decimal number.
**is_natural**            No         Returns FALSE if the form element contains anything other than a natural number:
                                     0, 1, 2, 3, etc.
**is_natural_no_zero**    No         Returns FALSE if the form element contains anything other than a natural
                                     number, but not zero: 1, 2, 3, etc.
**valid_url**             No         Returns FALSE if the form element does not contain a valid URL.
**valid_email**           No         Returns FALSE if the form element does not contain a valid email address.
**valid_emails**          No         Returns FALSE if any value provided in a comma separated list is not a valid email.
**valid_ip**              No         Returns FALSE if the supplied IP is not valid.
                                     Accepts an optional parameter of 'ipv4' or 'ipv6' to specify an IP format.
**valid_base64**          No         Returns FALSE if the supplied string contains anything other than valid Base64 characters.
========================= ========== ============================================================================================= =======================

.. note:: These rules can also be called as discrete methods. For
	example::

		$this->form_validation->required($string);

.. note:: You can also use any native PHP functions that permit up
	to two parameters, where at least one is required (to pass
	the field data).

******************
Prepping Reference
******************

The following is a list of all the prepping methods that are available
to use:

==================== ========= ==============================================================================================================
Name                 Parameter Description
==================== ========= ==============================================================================================================
**prep_for_form**    No        DEPRECATED: Converts special characters so that HTML data can be shown in a form field without breaking it.
**prep_url**         No        Adds "\http://" to URLs if missing.
**strip_image_tags** No        Strips the HTML from image tags leaving the raw URL.
**encode_php_tags**  No        Converts PHP tags to entities.
==================== ========= ==============================================================================================================

.. note:: You can also use any native PHP functions that permits one
	parameter, like ``trim()``, ``htmlspecialchars()``, ``urldecode()``,
	etc.

.. _class-reference:

***************
Class Reference
***************

.. php:class:: CI_Form_validation

	.. php:method:: set_rules($field[, $label = ''[, $rules = '']])

		:param	string	$field: Field name
		:param	string	$label: Field label
		:param	mixed	$rules: Validation rules, as a string list separated by a pipe "|", or as an array or rules
		:returns:	CI_Form_validation instance (method chaining)
		:rtype:	CI_Form_validation

		Permits you to set validation rules, as described in the tutorial
		sections above:

		-  :ref:`setting-validation-rules`
		-  :ref:`saving-groups`

	.. php:method:: run([$group = ''])

		:param	string	$group: The name of the validation group to run
		:returns:	TRUE on success, FALSE if validation failed
		:rtype:	bool

		Runs the validation routines. Returns boolean TRUE on success and FALSE
		on failure. You can optionally pass the name of the validation group via
		the method, as described in: :ref:`saving-groups`

	.. php:method:: set_message($lang[, $val = ''])

		:param	string	$lang: The rule the message is for
		:param	string	$val: The message
		:returns:	CI_Form_validation instance (method chaining)
		:rtype:	CI_Form_validation

		Permits you to set custom error messages. See :ref:`setting-error-messages`

	.. php:method:: set_error_delimiters([$prefix = '<p>'[, $suffix = '</p>']])

		:param	string	$prefix: Error message prefix
		:param	string	$suffix: Error message suffix
		:returns:	CI_Form_validation instance (method chaining)
		:rtype:	CI_Form_validation

		Sets the default prefix and suffix for error messages.

	.. php:method:: set_data($data)

		:param	array	$data: Array of data validate
		:returns:	CI_Form_validation instance (method chaining)
		:rtype:	CI_Form_validation

		Permits you to set an array for validation, instead of using the default
		``$_POST`` array.

	.. php:method:: reset_validation()

		:returns:	CI_Form_validation instance (method chaining)
		:rtype:	CI_Form_validation

		Permits you to reset the validation when you validate more than one array.
		This method should be called before validating each new array.

	.. php:method:: error_array()

		:returns:	Array of error messages
		:rtype:	array

		Returns the error messages as an array.

	.. php:method:: error_string([$prefix = ''[, $suffix = '']])

		:param	string	$prefix: Error message prefix
		:param	string	$suffix: Error message suffix
		:returns:	Error messages as a string
		:rtype:	string

		Returns all error messages (as returned from error_array()) formatted as a
		string and separated by a newline character.

	.. php:method:: error($field[, $prefix = ''[, $suffix = '']])

		:param	string $field: Field name
		:param	string $prefix: Optional prefix
		:param	string $suffix: Optional suffix
		:returns:	Error message string
		:rtype:	string

		Returns the error message for a specific field, optionally adding a
		prefix and/or suffix to it (usually HTML tags).

	.. php:method:: has_rule($field)

		:param	string	$field: Field name
		:returns:	TRUE if the field has rules set, FALSE if not
		:rtype:	bool

		Checks to see if there is a rule set for the specified field.

.. _helper-functions:

****************
Helper Reference
****************

Please refer to the :doc:`Form Helper <../helpers/form_helper>` manual for
the following functions:

-  :php:func:`form_error()`
-  :php:func:`validation_errors()`
-  :php:func:`set_value()`
-  :php:func:`set_select()`
-  :php:func:`set_checkbox()`
-  :php:func:`set_radio()`

Note that these are procedural functions, so they **do not** require you
to prepend them with ``$this->form_validation``.
