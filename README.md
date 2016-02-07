ワンパンマン更新チェッカー
====

ワンパンマンが更新されたかどうかをチェックし、
更新があれば、Trello に新しいカードを追加し、通知します。

## Usage

### Configuration

まずは Trello の Consumer Key と Consumer Secret を取得し、
続いて Trello の OAuth Token を取得します。

これらの取得方法は、Qiita の次の記事が参考になります。

* [Trello API を叩いてカードを作成する方法（curl利用）](http://qiita.com/isseium/items/8eebac5b79ff6ed1a180)
* [Trello APIを利用してスクリプトからタスクを管理する](http://qiita.com/AKB428/items/a4a9ff2893affb20f99c)

Trello の Consumer Key、Consumer Secret、OAuth Token の 3 つが取得できたら
`dotenv.templ` を `.env` にコピーし、`.env` をテキストエディタで開き、
`TRELLO_CONSUMER_KEY`, `TRELLO_CONSUMER_SECRET`, `TRELLO_OAUTH_TOKEN` にぞれぞれ設定します。

最後に `.env` の `TRELLO_BOARD_NAME` と `TRELLO_LIST_NAME` に
通知するボード名とカード名を設定します。

### How to Execute

次のコマンドを実行します。

```sh
$ bundle install --path vendor/bundle
$ bundle exec rake onepunchman:check
```

## Licence

[MIT](https://github.com/tcnksm/tool/blob/master/LICENCE)

## Author

[sunny4381](https://github.com/sunny4381)
