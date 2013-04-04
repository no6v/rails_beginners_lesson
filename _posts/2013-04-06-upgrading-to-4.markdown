---
layout: default
title: upgrading form 3.2 to 4.0
permalink: upgrading-from3.2-to4.0
---

## Rails 3.2 から Rails 4.0 へのアップグレード

このページは、Rails Guides の "A Guide for Upgrading Ruby on Rails" の一部を翻訳したものです。

翻訳ミス等もあると思いますので、英語の元サイトも参考にしながらご利用ください。

[A Guide for Upgrading Ruby on Rails](http://edgeguides.rubyonrails.org/upgrading_ruby_on_rails.html)

---

`この内容はまだ改訂の可能性があります。`

もしアプリケーションが Rails 3.2.x より古いバージョンであれば、Rails 4.0 にする前に Rails 3.2 にアップグレードすることをオススメします。

次の変更点は、アプリケーションを Rails 4.0 にアップグレードするためのものです。

### 2.1 Gemfile

Rails 4.0 では、Gemfile の `assets` グループはなくなりました。アップグレードする時には、その行を削除する必要があります。

### 2.2 vendor/plugins

Rails 4.0 は、もう `vendor/plugins` からロードするプラグインのサポートをしません。プラグインを gem に置き換え、Gemfile に追加しなければならないのです。もし、gem にしないのなら、例えば、`lib/my_plugin/*` に移し、適当な初期値を `config/initializers/my_plugin.rb` に追加することもできます。

### 2.3 Active Record

* Rails 4.0 では、[いくつかの関連づけの矛盾](https://github.com/rails/rails/commit/302c912bf6bcd0fa200d964ec2dc4a44abe328a6)のために、Active Record から "identity map" を削除しました。もし、手入力で "identity map" を有効にしたかったら、次の config を削除すればアプリケーションへの影響がなくなります。`config.active_record.identity_map`

* 関連付けされた集合内における `delete` メソッドは、レコードの id として `Fixnum` か `String` で受け取ることができ、それらレコードに加え、`destroy` メソッドがすることは大抵することができます。以前は、`ActiveRecord::AssociationTypeMismatch` がその引数(?arguments)を担っていましたが、Rails4.0 からは、自動的な `delete` が削除を実行する前に、マッチしたレコードの id を探そうとします。

* Rails4.0 は、`ActiveRecord::Relation` に命令を溜め込むような変更をしました。以前のバージョンでは、新しい命令は、その前の命令が明確化された後に適応されていました。しかし、これでは通用しません。詳しくは、[Active Record Query guide](http://edgeguides.rubyonrails.org/active_record_querying.html#ordering) を参照ください。

* Rails4.0 は、`serialized_attributes` と `attr_readonly` をクラスメソッドに限定する変更をしました。インスタンスメソッドは廃止する予定ですので使わずにクラスメソッドに変更してください。例：`self.serialized_attributes` -> `self.class.serialized_attributes`

* Rails4.0 は、Strong Parameters を利用する特徴として、attr_accessible と attr_protected を削除しました。スムーズなアップグレードの道筋として、[Protected Attributes gem](https://github.com/rails/protected_attributes) を使うといいでしょう。

* Rails4.0 は、scope を Proc や lambda のような呼び出し可能なオプジェクトにすることをお勧めします。

```ruby
scope :active, where(active: true)

# becomes
scope :active, -> { where active: true }
```

* Rails4.0 は、`ActiveRecord::FixtureSet` を利用して、`ActiveRecord::Fixtures` を廃止予定です。

* Rails4.0 は、`ActiveSupport::TestCase` を利用して、`ActiveRecord::TestCase` を廃止予定です。

### 2.4 Active Resource

Rails 4.0 は、Active Resource を gem に置き換えました。その機能が必要であれば、Gemfile に [Active Resource gem](https://github.com/rails/activeresource) を置けば大丈夫です。

### 2.5 Active Model

* Rails 4.0 は、エラーと `ActiveModel::Validations::ConfirmationValidator` との連携方法を変更しました。confirmation validations がコケたとき、エラーは `attribute` の代わりに `:#{attribute}_confirmation` と連携されます。

* Rails 4.0 は、`ActiveModel::Serializers::JSON.include_root_in_json` の初期値を `false` に変更しました。Active Model Serializers と Active Record objects は、同じデフォルトの挙動になります。これで、`config/initializers/wrap_parameters.rb` ファイルの次のオプションを、コメントアウトしたり削除したりすることができます。

```ruby
# Disable root element in JSON by default.
# ActiveSupport.on_load(:active_record) do
#   self.include_root_in_json = false
# end
```

### 2.6 Action Pack

* Rails 4.0 では `ActiveSupport::KeyGenerator` が導入され、(特に) 署名付きクッキーの生成や検証の基盤として利用します。所定の位置にある `secret_token` をそのままにして、新たに `secret_key_base` を追加すると、Rails 3.x で生成された既存の署名付きクッキーは透過的にアップグレードされます。

```ruby
  # config/initializers/secret_token.rb
  Myapp::Application.config.secret_token = 'existing secret token'
  Myapp::Application.config.secret_key_base = 'new secret key base'
```

ユーザベースが完全に Rails 4.x に移行できて、Rails 3.x に巻き戻さなくてもよいと確信できるまでは、`secret_key_base` を設定しないように注意してください。これは、Rails 4.x の `secret_key_base` で署名されたクッキーが、Rails 3.x のものと後方互換性がないためです。他の部分のアップグレードが確実に完了するまでは、所定の位置にある既存の `secret_token` はそのままにして、新たな `secret_key_base` は設定せず、廃止警告を無視しておいて構いません。

もし、外部のアプリケーションや Javascript が Rails アプリの署名付きセッションクッキー (もしくは一般的な署名付きクッキー) の読み込めることに依存しているなら、これらの関係を断ち切るまでは、 `secret_key_base` をセットするべきではありません。

* Rails 4.0 では、 `secret_key_base` がセットされていると、クッキーベースのセッションの内容を暗号化します。Rails 3.x では、クッキーベースのセッションの内容は署名されていますが、暗号化はされていません。署名付きのクッキーは、あなたのアプリによって生成されたことが検証でき、改竄を防止できるので「安全」です。しかし、その内容はエンドユーザにも見えてしまい、内容を暗号化することは、大きなパフォーマンスのロスなしに、この危険や懸念を排除することが出来ます。

上で述べたように、所定の位置にある `secret_token` をそのままにして、新たに `secret_key_base` を追加すると、Rails 3.x で生成された既存の署名付きクッキーは透過的にアップグレードされます。

```ruby
  # config/initializers/secret_token.rb
  Myapp::Application.config.secret_token = 'existing secret token'
  Myapp::Application.config.secret_key_base = 'new secret key base'
```

ここでも注意が必要です。ユーザベースが完全に Rails 4.x に移行できて、Rails 3.x に巻き戻さなくてもよいと確信できるまでは、`secret_key_base` を設定しないように注意してください。また、アップグレードする前に、外部のアプリや Javascript があなたのアプリによって生成された署名付きのクッキーをデコード出来ることに依存していないかどうかにも気をつけてください。

暗号化されたセッションクッキーへの移行の詳細は、[Pull Request #9978](https://github.com/rails/rails/pull/9978) を読んでください。

* Rails 4.0 では、`ActionController::Base.asset_path` オプションは削除されました。Asset Pipeline 機構を使ってください。

* Rails 4.0 では、`ActionController::Base.page_cache_extension` オプションは非推奨になりました。代わりに `ActionController::Base.default_static_extension` を使ってください。

* Rails 4.0 では、ActionPack から Action と Page のキャッシュが削除されました。コントローラで、`caches_action` を使うには `actionpack-action_caching` gem を、`caches_pages` を使うには `actionpack-page_caching` gem 追加する必要があります。

* Rails 4.0 では、XML のパラメータパーサーが削除されました。もしこの機能が必要なら、`actionpack-xml_parser` gem を追加する必要があります。

* Rails 4.0 では、memcached クライアントのデフォルトが、`memcache-client` から `dalli` に変更されました。アップグレードするには、`Gemfile` に `gem 'dalli'` の行を追加するだけです。

* Rails 4.0 では、`dom_id` および `dom_class` メソッドが非推奨になりました。この機能が必要なら、コントローラで `ActionView::RecordIdentifier` モジュールをインクルードする必要があります。

* Rails 4.0 では、`assert_generates`、`assert_recognizes`、`assert_routing` の挙動が変更されました。これらのアサーションは、`ActionController::RoutingError` 例外ではなく、`Assertion` 例外をあげるようになっています。

* Rails 4.0 では、相反する名前付きルートが定義されたときに、`ArgumentError` があがるようになりました。これは、名前付きルートが明示的に定義されるか、`resources` メソッドで起こります。ここでは、`example_path` という名前のルートが相反するサンプルを二つあげています。

```ruby
get 'one' => 'test#example', as: :example
get 'two' => 'test#example', as: :example
```

```ruby
resources :examples
get 'clashing/:id' => 'test#example', as: :example
```

最初の例では、複数のルートに同じ名前を使わないようにすることで簡単に回避できます。二つ目では、[Routing Guide](http://edgeguides.rubyonrails.org/routing.html#restricting-the-routes-created) に詳細があるように、`resources` メソッドの `only` もしくは `except` オプションを使って、ルートを制限できます。

* Rails 4.0 では、unicode キャラクターのルートの書き方も変わりました。unicode キャラクターのルートも直接書けるようになっています。そのようなルートを書いている場合は、それらを変更しなければなりません。例えば、

```ruby
get Rack::Utils.escape('こんにちは'), controller: 'welcome', action: 'index'
```

は、

```ruby
get 'こんにちは', controller: 'welcome', action: 'index'
```

のようになります。

* Rails 4.0 では、`match` を使ったルートでは、リクエストメソッドを指定する必要があります。例えば、

```ruby
# Rails 3.x
match "/" => "root#index"
 
# becomes
match "/" => "root#index", via: :get
 
# or
get "/" => "root#index"
```

* Rails 4.0 has removed ActionDispatch::BestStandardsSupport middleware, <!DOCTYPE html> already triggers standards mode per http://msdn.microsoft.com/en-us/library/jj676915(v=vs.85).aspx and ChromeFrame header has been moved to config.action_dispatch.default_headers.

* ↑これ、よく分からないです

アプリケーションのコードから、ミドルウェアへの参照をすべて削除するのも忘れてはいけません。例:

```ruby
# 例外があがる
config.middleware.insert_before(Rack::Lock, ActionDispatch::BestStandardsSupport)
```
また、環境設定を確認して、`config.action_dispatch.best_standards_support` があれば削除してください。

* Rails 4.0 では、asset プリコンパイルで `vendor/assets` や `lib/assets` から JS/CSS 以外の asset を自動ではコピーしません。Rails アプリケーションやエンジンの開発者は、これらを `app/assets` に置くか、`config.assets.precompile` を設定する必要があります。

* Rails 4.0 では、アクションが要求されたフォーマットを処理できない場合に `ActionController::UnknownFormat` 例外をあげます。デフォルトでは、この例外は 406 Not Acceptable を返すよう処理されますが、これは変更可能になりました。Rails 3 では常に 406 Not Acceptable が返され、変更出来ませんでした。

* Rails 4.0 では、`ParamsParser` がリクエストパラメータのパースに失敗した場合に、`ActionDispatch::ParamsParser::ParseError` を継承した例外があがります。より下位の、例えば `MultiJson::DecodeError` の代わりに、この例外を rescue してもよいです。

* Rails 4.0 では、エンジンが URL プレフィックス付きのアプリケーションにマウントされている場合でも、`SCRIPT_NAME` は正しくネストされます。URL プレフィックスの上書きに対処するために、`default_url_options[:script_name]` をセットする必要はなくなりました。

* Rails 4.0 では、`ActionController::Integration` は非推奨になりました。`ActionDispatch::Integration` を使ってください。

* Rails 4.0 では、`ActionController::IntegrationTest` は非推奨になりました。`ActionDispatch::IntegrationTest` を使ってください。

* Rails 4.0 では、`ActionController::PerformanceTest` は非推奨になりました。`ActionDispatch::PerformanceTest` を使ってください。

* Rails 4.0 では、`ActionController::AbstractRequest` は非推奨になりました。`ActionDispatch::Request` を使ってください。

* Rails 4.0 では、`ActionController::Request` は非推奨になりました。`ActionDispatch::Request` を使ってください。

* Rails 4.0 では、`ActionController::AbstractResponse` は非推奨になりました。`ActionDispatch::Response` を使ってください。

* Rails 4.0 では、`ActionController::Response` は非推奨になりました。`ActionDispatch::Response` を使ってください。

* Rails 4.0 では、`ActionController::Routing` は非推奨になりました。`ActionDispatch::Routing` を使ってください。

### 2.7 Active Support

Rails 4.0 では、`ERB::Util#json_escape` に対するエイリアス `j` が削除されました。`j` は `ActionView::Helpers::JavaScriptHelper#escape_javascript` のために使われているからです。

### 2.8 ヘルパーのロード順

Rails 4.0 では、二つ以上のディレクトリからロードされるヘルパーのロード順が変更されました。以前は、これらは一つにまとめられて、アルファベット順に並べ替えられていました。Rails 4.0 にアップデートすると、ロードされたディレクトリの順序が保持され、各ディレクトリの中でのみアルファベット順に並べ替えられます。`helpers_path` パラメータを明示的に使用しない限り、この変更はエンジンからヘルパーをロードする方法にのみ影響します。もし、順序に依存しているなら、アップグレード後に正しいメソッドが利用できるかどうかチェックしてください。エンジンがロードされる順序を変更したい場合は、`config.railties_order=` メソッドが使えます。

### 2.9 Active Record Observer と Action Controller Sweeper

Active Record Observer と Action Controller Sweeper は、`rails-observers` gem に置き換えられました。これらの機能が必要なら、`rails-observers` gem を追加する必要があります。

### 2.10 sprockets-rails

* `assets:precompile:primary` は削除されました。代わりに、`assets:precompile` を使ってください。

### 2.11 sass-rails

* 引数 2 個の `asset_url` は非推奨になりました。例えば、`asset-url("rails.png", image)` は `asset-url("rails.png")` になります。

---
