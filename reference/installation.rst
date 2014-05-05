インストール
==============

SonataAdminBundle は、Symfony2 をクリーンインストールした状態か、既にあるプロジェクトなのかを問わず、プロジェクトのライフサイクルのどの段階からでも導入できます。

ソースコードのダウンロード
---------------------------

依存関係の管理と SonataAdminBundle のダウンロードには composer を使ってください。

.. code-block:: bash

    php composer.phar require sonata-project/admin-bundle


すると、バージョンの指定をタイプするように求められます。 'dev-master' とすると、 Symfony2 の最新版に対応する最新版を使うことができます。
以前のバージョンを使いたい場合は `packagist <https://packagist.org/packages/sonata-project/admin-bundle>`_ を参照してください。

.. code-block:: bash

    Please provide a version constraint for the sonata-project/admin-bundle requirement: dev-master

保存形式に合わせたバンドルの選択とダウンロード
------------------------------------------------

SonataAdminBundle は保存形式を問いません。つまり、複数の保存形式に対応しています。プロジェクトで使っているデータ保存形式によって、下記のいずれかのバンドルをインストールする必要があります。
リンク先には各バンドルのインストール方法の説明があります。

    - `SonataDoctrineORMAdminBundle <http://sonata-project.org/bundles/doctrine-orm-admin/master/doc/reference/installation.html>`_
    - `SonataDoctrineMongoDBAdminBundle <https://github.com/sonata-project/SonataDoctrineMongoDBAdminBundle/blob/master/Resources/doc/reference/installation.rst>`_
    - `SonataPropelAdminBundle <http://sonata-project.org/bundles/propel-admin/master/doc/reference/installation.html>`_
    - `SonataDoctrinePhpcrAdminBundle <https://github.com/sonata-project/SonataDoctrinePhpcrAdminBundle/blob/master/Resources/doc/reference/installation.rst>`_

.. note::
    どれを選べば良いかわからないですか？大抵の場合、このバンドルを新しく使う人にとっては伝統的なリレーショナルデータベース（MySQL, PostgreSQL 等）を使う SonataDoctrineORMAdminBundle が良いでしょう。


SonataAdminBundle と依存バンドルの有効化
-----------------------------------------------

SonataAdminBundle は機能を実装するにあたって他のバンドルに依存しています。
SonataAdminBundle を動かすためには、ステップ2で説明したデータ保存用のバンドルの他に、下記のバンドルが必要です。 

    - `SonataBlockBundle <http://sonata-project.org/bundles/block/master/doc/reference/installation.html>`_
    - `KnpMenuBundle <https://github.com/KnpLabs/KnpMenuBundle/blob/master/Resources/doc/index.md#installation>`_ (Version 1.1.*)

上記の依存バンドルは SonataAdminBundle の依存ライブラリとして、 composer が自動的にダウンロードしてくれます。
しかし、開発者は手動で、プロジェクトの AppKernel.php でこの依存バンドルを有効化し、バンドルの設定をしなければなりません。
SonataAdminBundle も忘れずに有効化してください。

.. code-block:: php

    <?php
    // app/AppKernel.php
    public function registerBundles()
    {
        return array(
            // ...

            // 依存バンドルを有効化
            new Sonata\CoreBundle\SonataCoreBundle(),
            new Sonata\BlockBundle\SonataBlockBundle(),
            new Knp\Bundle\MenuBundle\KnpMenuBundle(),
            //...

            // まだ有効化していなければ、データ保存形式ごとの
            // バンドルも有効化してください。
            // この例では SonataDoctrineORMAdminBundleを使っていますが
            // 他のバンドルでも同じようにしてください。
            new Sonata\DoctrineORMAdminBundle\SonataDoctrineORMAdminBundle(),

            // SonataAdminBundleを有効化
            new Sonata\AdminBundle\SonataAdminBundle(),
            // ...
        );
    }

.. note::
    依存バンドルがプロジェクトの AppKernel.php の中で既に有効化されている場合は再度有効化する必要はありません。

.. note::
    バージョン2.3以上ではアセットはバンドル内に含まれるようになり、 SonatajQueryBundle は必要なくなりました。
    バンドルは bower.io にも登録されており、 bower を使ってアセットを管理することもできるようになりました。

SonataAdminBundle の依存バンドルの設定
------------------------------------------

SonataAdminBundle の依存バンドルを設定しましょう。 Symfony2 の設定をどのように変更する必要があるのかについては、各バンドルのインストールや設定に関するドキュメントを参照してください。

SonataAdminBundle は、管理画面のダッシュボードで利用できる SonataBlockBundle のブロックを提供しています。
このブロックを利用するためには、 SonataBlockBundle の設定でブロックを確実に有効化してください。

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        sonata_block:
            default_contexts: [cms]
            blocks:
                # SonataAdminBundle のブロックを有効化
                sonata.admin.block.admin_list:
                    contexts:   [admin]
                # 他のブロック

.. note::
    心配しないでください。少なくともこの段階では、ブロックとは何なのか完全に理解している必要はありません。
    SonataBlockBundle は便利なツールですが、すぐに理解できなくても問題はありません。

キャッシュクリア
-----------------

バンドルのアセットをインストールしましょう。

.. code-block:: bash

    php app/console assets:install web

新しいバンドルを追加したときにはキャッシュを削除するほうが良いです。

.. code-block:: bash

    php app/console cache:clear

この時点では、 Symfony2 のプロジェクトは、SonataAdminBundle とその依存バンドルに由来するエラーが表示されること無く、機能しているはずです。
SonataAdminBundle はインストールされていますが、まだ設定されていません（設定については次のセクションで説明します）。つまり、 SonataAdminBundle を使うことはまだできないのです。

この時点あるいはインストールの途中で何かエラーが発生しても落ち着いてください。

    - エラーメッセージを注意深く読んでください。正確にどのバンドルがエラーを引き起こしているのか見極めてみましょう。 SonataAdminBundle なのか、依存バンドルのどれかなのか？
    - SonataAdminBundle と依存バンドル両方について、全ての手順を正確に実行したか確認してください。 
    - もしかしたら、既に誰かが同じエラーに行き当たって、解決方法が書かれているかもしれません。 `Google <http://www.google.com>`_, `Sonata Users Group <https://groups.google.com/group/sonata-users>`_, `Symfony2 Users Group <https://groups.google.com/group/symfony2>`_, `Symfony Forum <forum.symfony-project.org>`_ をチェックしてみてください。
    - それでもまだ解決できなければ、 Github で未解決の issue をチェックしてみてください。

上に出てきたバンドルのインストールに成功した後、プロジェクトのモデルを管理するためには、 SonataAdminBundle を設定する必要があります。
SonataAdminBundle を素早くセットアップするために必要な設定については :doc:`getting_started` で説明します。

