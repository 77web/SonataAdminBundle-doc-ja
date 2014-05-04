SonataAdminBundle を始めよう
======================================

インストール手順を完了したなら、 SonataAdminBundle はインストールされていますがアクセスできない状態になっています。
SonataAdminBundle を使い始める前に、最初にプロジェクトのモデルに合わせて設定する必要があります。
下記は SonataAdminBundle を素早く設定し、アプリケーションのモデルの最初の管理画面を作るための、簡単なチェックリストです。

* ステップ1: SonataAdminBundle のルーティング設定を定義する
* ステップ2: Admin クラスを作成する
* ステップ3: Admin サービスを作成する
* ステップ4: 設定する

ステップ1: SonataAdminBundle のルーティング設定を定義する
-----------------------------------------------------------

SonataAdminBundle のページにアクセスできるようにするためには、アプリケーションのルーティング設定ファイルに SonataAdminBundle のページのルーティング設定を追加する必要があります。

.. configuration-block::

    .. code-block:: yaml

        # app/config/routing.yml
        admin:
            resource: '@SonataAdminBundle/Resources/config/routing/sonata_admin.xml'
            prefix: /admin

        _sonata_admin:
            resource: .
            type: sonata_admin
            prefix: /admin

.. note::

    アプリケーションの設定ファイルとして XML や PHP 形式の設定ファイルを
    使っている場合、上記のルーティング設定は、使っているファイル形式によって
    routing.xml あるいは routing.php ファイルの中に追加しなければなりません。

.. note::

    ``resource: .`` 設定がどういう意味か気になった開発者のために。
    この書き方は通常の書き方ではありませんが、 Symfony がこの設定を要求し、
    しかも実際のファイルを指していなければならないために定義されています。
    Symfony によるバリデーションさえパスしてしまえば、 Sonata の ``AdminPoolLoader``
    がルーティングを処理し、 resource の設定は無視されます。

この時点で既に、下記のURLにアクセスすると、管理画面のダッシュボード（ただし、空っぽの）にアクセスすることができるはずです。
``http://yoursite.local/admin/dashboard``


ステップ2: Admin クラスを作成する
-----------------------------------

SonataAdminBundle を使うことで、データの作成・更新・検索をGUIによるデータ管理が簡単にできるようになります。
データ管理のためのアクションを使うためには設定が必要ですが、その設定は Admin クラスを使って行うことができます。

Admin クラスは、アプリケーションのモデルを各管理アクションへの関連づけを表すものです。
どのフィールドを一覧に表示し、どのフィールドを検索に使用し、どのフィールドを登録・編集フォームに含めるか、開発者はこの Admin クラスの中で指定することができます。

最も簡単にアプリケーションのモデルについての Admin クラスを作成するには ``Sonata\AdminBundle\Admin\Admin`` クラスを継承します。

アプリケーションの AcmeDemoBundle に Post エンティティがあるとします。
このモデルのための基本的な Admin クラスは下記のようになります。

.. code-block:: php

   <?php
   // src/Acme/DemoBundle/Admin/PostAdmin.php

   namespace Acme\DemoBundle\Admin;

   use Sonata\AdminBundle\Admin\Admin;
   use Sonata\AdminBundle\Datagrid\ListMapper;
   use Sonata\AdminBundle\Datagrid\DatagridMapper;
   use Sonata\AdminBundle\Form\FormMapper;

   class PostAdmin extends Admin
   {
       // 登録・編集フォームに表示されるフィールド
       protected function configureFormFields(FormMapper $formMapper)
       {
           $formMapper
               ->add('title', 'text', array('label' => 'Post Title'))
               ->add('author', 'entity', array('class' => 'Acme\DemoBundle\Entity\User'))
               ->add('body') //タイプを指定しなかった場合は SonataAdminBundle が推測しようとします
           ;
       }

       // 検索フォームに表示されるフィールド
       protected function configureDatagridFilters(DatagridMapper $datagridMapper)
       {
           $datagridMapper
               ->add('title')
               ->add('author')
           ;
       }

       // 一覧に表示されるフィールド
       protected function configureListFields(ListMapper $listMapper)
       {
           $listMapper
               ->addIdentifier('title')
               ->add('slug')
               ->add('author')
           ;
       }
   }

上記の3つのメソッドを実装することが Admin クラス作成の最初のステップです。
アプリケーションのモデルの操作や表示をより自由にカスタマイズできる他のオプションもあります。他のオプションについては、このマニュアルのもっと後の章で説明します。

ステップ3: Admin サービスを定義する
--------------------------------------

アプリケーションの Admin クラスを作成し終わったら、サービスとして定義する必要があります。
サービスには、 ``sonata.admin`` タグが必要で、このタグによって SonataAdminBundle ではサービスが Admin クラスであることを認識します。

新しく ``admin.xml`` 又は ``admin.yml`` を ``Acme/DemoBundle/Resources/config/`` フォルダ内に作成してください。

.. configuration-block::

    .. code-block:: xml

       <!-- Acme/DemoBundle/Resources/config/admin.xml -->
       <container xmlns="http://symfony.com/schema/dic/services"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://symfony.com/schema/dic/services/services-1.0.xsd">
           <services>
              <service id="sonata.admin.post" class="Acme\DemoBundle\Admin\PostAdmin">
                 <tag name="sonata.admin" manager_type="orm" group="Content" label="Post"/>
                 <argument />
                 <argument>Acme\DemoBundle\Entity\Post</argument>
                 <argument />
                 <call method="setTranslationDomain">
                     <argument>AcmeDemoBundle</argument>
                 </call>
             </service>
          </services>
       </container>


    .. code-block:: yaml

       # Acme/DemoBundle/Resources/config/admin.yml
       services:
           sonata.admin.post:
               class: Acme\DemoBundle\Admin\PostAdmin
               tags:
                   - { name: sonata.admin, manager_type: orm, group: "Content", label: "Post" }
               arguments:
                   - ~
                   - Acme\DemoBundle\Entity\Post
                   - ~
               calls:
                   - [ setTranslationDomain, [AcmeDemoBundle]]

Admin サービスの基本的な設定はとてもシンプルです。
ステップ2で作成したクラスのインスタンスをサービスとして作成し、3つの引数を受け取ります。

    1. Admin サービスのコード（デフォルト値はサービスの名前）
    2. Admin クラスがマッピングされるモデル名（必須）
    3. 管理画面のアクションを担当するコントローラー名（デフォルト値は SonataAdminBundle:CRUDController）

大抵のアプリケーションでは1つ目と3つ目の引数についてはデフォルト値のままで動作するため、通常は開発者は2つ目の引数だけを指定すれば事足ります。

``setTranslationDomain`` の呼び出しにより、管理ページの翻訳ドメインとして何を使うか指定することができます。
詳しくは `symfony translations page`_ を参照してください。


Admin サービスの設定ファイルができたら、あとは Symfony2 に読み込ませるだけです。それには2つの方法があります。

1 - メインの config.yml からインポートする
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

新しい設定ファイルをメインの config.yml から読み込ませます。（正しいファイル形式を使っていることを確認してください）

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        imports:
            - { resource: @AcmeDemoBundle/Resources/config/admin.xml }

2 - バンドルに読み込ませる
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

バンドルに admin 設定ファイルを読み込ませることもできます。
バンドルの extension ファイルの中、 `symfony cookbook`_ で説明されているように ``load()`` メソッドを使って読み込ませます。

.. configuration-block::

    .. code-block:: xml

        # Acme/DemoBundle/DependencyInjection/AcmeDemoBundleExtension.php for XML configurations
        
        namespace Acme\DemoBundle\DependencyInjection;

        use Symfony\Component\DependencyInjection\Loader;
        use Symfony\Component\Config\FileLocator;
        
        class AcmeDemoBundleExtension extends Extension
        {
            public function load(array $configs, ContainerBuilder $container) {
                // ...
                $loader = new Loader\XmlFileLoader($container, new FileLocator(__DIR__.'/../Resources/config'));
                $loader->load('admin.xml');
            }
        }

    .. code-block:: yaml

        # Acme/DemoBundle/DependencyInjection/AcmeDemoBundleExtension.php for YAML configurations
        
        namespace Acme\DemoBundle\DependencyInjection;

        use Symfony\Component\DependencyInjection\Loader;
        use Symfony\Component\Config\FileLocator;

        class AcmeDemoBundleExtension extends Extension
        {
            public function load(array $configs, ContainerBuilder $container) {
                // ...
                $loader = new Loader\YamlFileLoader($container, new FileLocator(__DIR__.'/../Resources/config'));
                $loader->load('admin.yml');
            }
        }

ステップ 4: 設定する
---------------------

この時点で、アプリケーションのモデルのための基本的な管理アクションが使えるようになっています。

``http://yoursite.local/admin/dashboard`` に再度アクセスすると、マッピングしたモデルのパネルが表示されます。データを作成したり、一覧で見たり、編集したり、削除したりできます。

プロジェクトの名前とロゴをページトップのバーに表示したくなるでしょう。

プロジェクトのロゴファイルを ``src/Acme/DemoBundle/Resources/public/img/fancy_acme_logo.png`` に配置してください。
    
アセットをインストールします。

.. code-block:: sh

    $ php app/console assets:install

プロジェクトのメインの config.yml を下記のように変更します。

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        sonata_admin:
            title:      Acme Demo Bundle
            title_logo: bundles/acmedemo/img/fancy_acme_logo.png



次のステップ　認証
---------------------

もう気づいているかもしれませんが、URLをタイプするだけで管理画面ダッシュボードとデータにアクセスできています。
デフォルトでは、柔軟性を最大に高めるために、 SonataAdminBundle はユーザー認証の仕組みを備えていません。しかし、アプリケーションにはおそらくその機能が必要でしょう。 Sonata Project はとても有名な ``FOSUserBundle`` と連携するための ``SonataUserBundle`` を開発しています。詳しくはこのドキュメントの :doc:`security` セクションを参照してください。

おめでとうございます！これで SonataAdminBundle を使い始めることができます。他のモデルの管理画面を作成することができ、また、既に作成したモデルの管理画面をカスタマイズすることもできます。
続きの各章では、バンドルの部品や機能ごとに、どんな設定ができるか、 SonataAdminBundle を使ってどんなことができるのかより詳しく解説しています。

.. _`symfony cookbook`: http://symfony.com/doc/master/cookbook/bundles/extension.html#using-the-load-method
.. _`symfony translations page`: http://symfony.com/doc/current/book/translation.html#using-message-domains

