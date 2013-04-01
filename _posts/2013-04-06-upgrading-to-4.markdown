---
layout: default
title: upgrading form 3.2 to 4.0
permalink: upgrading-from3.2-to4.0
---

## Rails3.2からRails4.0へのアップグレード

このページは、Rails Guidesの"A Guide for Upgrading Ruby on Rails"の一部を翻訳したものです。

翻訳ミス等もあると思いますので、英語の元サイトも参考にしながらご利用ください。

[A Guide for Upgrading Ruby on Rails](http://edgeguides.rubyonrails.org/upgrading_ruby_on_rails.html)

---

`この内容はまだ改訂の可能性があります。`

もしアプリケーションがRails3.2.xより古いバージョンであれば、Rails4.0にする前にRails3.2にアップグレードすることをオススメします。

次の変更点は、アプリケーションをRails4.0にアップグレードするためのものです。

>

### 2.1 vendor/plugins

Rails4.0は、もうvendor/pluginsからロードするプラグインのサポートをしません。プラグインをgemに置き換え、Gemfileに追加しなければならないのです。もし、gemにしないのなら、例えば、lib/my_plugin/*に移し、適当な初期値をconfig/initializers/my_plugin.rbに追加することもできます。

>

### 2.2 Active Record

* Rails4.0では、[いくつかの関連づけの矛盾](https://github.com/rails/rails/commit/302c912bf6bcd0fa200d964ec2dc4a44abe328a6)のために、Active Recordから"identity map"を削除しました。もし、手入力で"identity map"を有効にしたかったら、次のconfigを削除すればアプリケーションへの影響がなくなります。
{% highlight ruby %}
config.active_record.identity_map
{% endhighlight %}

* 関連付けされた集合内におけるdeleteメソッドは、レコードのidとしてFixnumかStringで受け取ることができ、それらレコードに加え、destroyメソッドがすることは大抵することができます。以前は、ActiveRecord::AssociationTypeMismatchがその引数(?arguments)を担っていましたが、Rails4.0からは、自動的なdeleteが削除を実行する前に、マッチしたレコードのidを探そうとします。

* Rails4.0は、ActiveRecord::Relationに命令を溜め込むような変更をしました。以前のバージョンでは、新しい命令は、その前の命令が明確化された後に適応されていました。しかし、これでは通用しません。詳しくは、[Active Record Query guide](http://edgeguides.rubyonrails.org/active_record_querying.html#ordering)を参照ください。

* Rails4.0は、serialized_attributesとattr_readonlyをクラスメソッドに限定する変更をしました。インスタンスメソッドは廃止する予定ですので使わずにクラスメソッドに変更してください。例：self.serialized_attributes -> self.class.serialized_attributes

* Rails4.0は、Strong Parametersを利用する特徴として、attr_accessibleとattr_protectedを削除しました。スムーズなアップグレードの道筋として、[Protected Attributes gem](https://github.com/rails/protected_attributes)を使うといいでしょう。

* Rails4.0は、scopeをProcやlambdaのような呼び出し可能なオプジェクトにすることをお勧めします。
{% highlight ruby %}
scope :active, where(active: true)

# becomes
scope :active, -> { where active: true }
{% endhighlight %}

* Rails4.0は、ActiveRecord::FixtureSetを利用して、ActiveRecord::Fixturesを廃止予定です。

* Rails4.0は、ActiveSupport::TestCaseを利用して、ActiveRecord::TestCaseを廃止予定です。

>

### 2.3 Active Resource

Rails4.0は、Active Resourceをgemに置き換えました。その機能が必要であれば、Gemfileに[Active Resource gem](https://github.com/rails/activeresource)を置けば大丈夫です。

>

### 2.4 Active Model

* Rails4.0は、エラーとActiveModel::Validations::ConfirmationValidatorとの連携方法を変更しました。confirmation validationsがコケたとき、エラーはattributeの代わりに:#{attribute}_confirmationと連携されます。

* Rails4.0は、ActiveModel::Serializers::JSON.include_root_in_jsonの初期値をfalseに変更しました。Active Model SerializersとActive Record objectsは、同じデフォルトの挙動になります。これで、 config/initializers/wrap_parameters.rbファイルの次のオプションを、コメントアウトしたり削除したりすることができます。

{% highlight ruby %}
# Disable root element in JSON by default.
# ActiveSupport.on_load(:active_record) do
#   self.include_root_in_json = false
# end
{% endhighlight %}

>

### 2.5 Action Pack



---
