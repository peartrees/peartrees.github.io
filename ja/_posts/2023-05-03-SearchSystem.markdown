---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: content_3
title: Vue.js + Flask + MySQL + 各種APIを使った検索アプリ開発

# post specific
# if not specified, .name will be used from _data/owner/[language].yml
#author: "peartrees"
# multiple category is not supported
category: programming
# multiple tag entries are possible
tags: [Web, programming,database,SQL,MySQL]
# thumbnail image for post
img: ":post_pic3.jpg"
# disable comments on this page
#comments_disable: true

# publish date
date: 2023-05-03 17:45:00 +0900

# seo
# if not specified, date will be used.
#meta_modify_date: date: 2023-04-13 17:45:00 +0900
# check the meta_common_description in _data/owner/[language].yml
#meta_description: ""

# optional
# please use the "image_viewer_on" below to enable image viewer for individual pages or posts (_posts/ or [language]/_posts folders).
# image viewer can be enabled or disabled for all posts using the "image_viewer_posts: true" setting in _data/conf/main.yml.
#image_viewer_on: true
# please use the "image_lazy_loader_on" below to enable image lazy loader for individual pages or posts (_posts/ or [language]/_posts folders).
# image lazy loader can be enabled or disabled for all posts using the "image_lazy_loader_posts: true" setting in _data/conf/main.yml.
#image_lazy_loader_on: true
# exclude from on site search
#on_site_search_exclude: true
# exclude from search engines
#search_engine_exclude: true
# to disable this page, simply set published: false or delete this file
#published: false
---

この記事はQiitaに以前投稿した記事：[Vue.js + Flask + MySQL + 各種APIを使った検索アプリ開発](https://qiita.com/peartrees/items/74a8fc43fa7286973141){:target="_blank"}を移植したものになります．

---

# 概要
GoogleのようなイメージのWeb検索アプリを作成しました．

大まかなシステム構成は，フロントエンドとして**Vue.js**，バックエンドとして**Flask**，検索結果を取得するAPIとして**Google Suggest API・Google Search API**，検索ログの収集として**MySQL**を用いています．

![System_image](:/2023_05_03/System_image.png)

![App_image.gif](:/2023_05_03/App_image.gif)


本記事はアプリ開発の備忘録としてまとめたものです．
流れを重視しているので，細かいところまでは記述していません🙇‍♂️
[gitにプログラムを載せています](https://github.com/peartrees/Web_Search_App)

# 大まかな流れ
### 1. 環境構築編
- front-end部分
    - Vue projectを作成
- back-end部分
    - pipenvでpython仮想環境を構築
    - Flaskを利用

### 2. Autocomplete機能と検索結果取得編
- 類似クエリの候補を表示します
    - Google Sugget API
- APIで結果を取得するところをやります
    - Google Search API

### 3. DBに繋げる編
- 検索ログを取得してDBに格納していきます

### 4. 詰まった点とまとめ
- 時間が溶けた点を記述します
- & まとめです

以下本編です
<hr>
# 1. 環境構築編
### 1. front-end部分
- Vue Projectを作成する
    - 一部のmoduleがVue2系のみ対応のためvue2.xでinstall

- フロントエンド開発のために，```npm install Vuetify```

### 2. back-end部分
バックエンド開発のために，pipenvで仮想環境構築する

- root directoryで，```pipenv --python ```を実行
    - command not foundであれば```pip install pipenv```を実行

- 仮想環境に必要なmoduleを入れる
    - ```pipenv install flask```, ```pipenv install flask-restful```

- root directoryで```pipenv --python {path/to/python}```を実行

# 2. Autocomplete機能と検索結果取得編
### 1. google suggest apiを使って，検索キーワードを入手します

検索クエリを入力した際に，類似するクエリを提示する**autocomplete**を実装します．
autocompleteという聞き慣れない語を紹介しましたが，以下のような非常に慣れ親しんだやつです．

![Google_Image](:/2023_05_03/Google_image.png)

最終的には，[https://whatdoyousuggest.net/](https://whatdoyousuggest.net/)のようなUIに出来れば良いなあと考えています．

今回は天下のGoogle様が公開している**Google Suggest API**を使うことで，入力されたクエリに続く語をXML形式で取得することができます．

PythonでこのAPIを利用する場合は，次のようなプログラムになります．

```python:Google-Suggest-APIの利用
qstr = "{your_query}"
google_r = requests.get("http://www.google.com/complete/search",
                 params={'q':qstr,
                         'hl':'ja',
                         'ie':'utf_8',
                         'oe':'utf_8',
                         'output': 'toolbar'})

google_root = etree.XML(google_r.text)
google_sugs = google_root.xpath("//suggestion")
google_sugstrs = [s.get("data") for s in google_sugs]
print(google_sugstrs)
```

このプログラムに対して**元旦**を入力した場合，次のようなリストが得られます．

```terminal:「クエリ：元旦」を入力した際の類似クエリ
['元旦とは', '元旦', '元旦競歩', '元旦英語', '元旦営業 東京', '元旦ビューティ工業', '元旦年賀状', '元旦に出した年賀状いつ届く', '元旦 ランチ 東京 2022', '元旦 電車']
```

本来は，フロント側（Vue.js側）でAPIを呼び出して処理する方が高速で，その実装をしたかったのですが，CORSのエラーをクリアすることができず，~~（無限に時間が溶けたため）~~，バックエンド側(Flask側)にクエリを渡してAPIを呼び出すことにしました．

フロントエンドとバックエンドの通信には**axios**ライブラリを使います．
HTTP通信の流れとしては以下のような感じになります．

1. axiosのPOSTメソッドでFlaskにクエリを渡す
2. Flask側でGoogle Suggest APIを呼び出して結果を取得する
3. axiosの.then(response)で2の結果をリストとして取得

```javascript:FlaskサーバーとのHTTP通信(ajax)
querySelections: function (v) {
  const UserQuery = { text: this.search }
  this.Sug_Loading = true
  axios
    .post('/api/get_suggest/get', UserQuery)
    .then(response => {
      this.items = response.data
      console.log(response.data)
      this.Sug_Loading = false
    })
    .catch(err => {
      alert('APIサーバと接続できません')
      err = null
    })
}
```

こうして取得できたリストを良い感じに表示させるのが，Vuetifyの**v-autocomplete**です．[公式ドキュメントはこちら](https://vuetifyjs.com/ja/components/autocompletes/)があります．
色々いじった結果，以下のような感じになりました．

![AutoComplete_image](:/2023_05_03/AutoComplete_image.png)


良い感じですね！

余談ですが，ユーザーの入力値を制御する際にVueの監視プロパティを挟んでいます．
この監視プロパティのおかげで，ユーザーがクエリを入力する度にAPIを呼び出すことができます．

```javascript:監視プロパティ
  watch: {
    search (val) {
      val && val !== this.select && this.querySelections(val)
    }
  }
```

Autocomplete機能を実装する際にはCORSのエラーなど様々な課題に直面しましたが，何とか実装することができました．やがてはフロント側で処理を行なって，より処理性能の良いものにできたらなあと考えています．

---

### 2. 各種検索apiを使って，検索結果を取得します

本節では，クエリを入力してそれに適合したドキュメントを返す機能を実装します（いつものGoogle検索のようなもの）．
こちらも相変わらず天下のGoogle様が公開しているAPIを使いたいと思います．

プログラムは以下のようになりました．
APIを呼び出して結果をリストとして格納しています．

```python:Google-Search-APIの利用
class Call_Search(Resource):
    def post(self):
        time.sleep(1)
        total_result = []
        # postされたデータを読み込み
        input_data = request.json
        CUSTOM_SEARCH_ENGINE_ID = "XXXXXX"
        GOOGLE_API_KEY          = "XXXXXX"
        search = build("customsearch","v1",developerKey = GOOGLE_API_KEY)
        ggl_response = search.cse().list(q = input_data["text"], cx = CUSTOM_SEARCH_ENGINE_ID,
        lr = 'lang_ja', gl = 'jp', num = 10, start = 1).execute()
        ggl_result = ggl_response["items"]
        total_result.append(ggl_result)
        return total_result
```

上記のプログラムは，下記のJavascriptで呼び出されて実行されます．

```javascript:FlaskサーバーとのHTTP通信（ajax）
SendData: function (input) {
  const data = { text: input }
  axios
    .post('/api/call_search/post', data)
    .then(response => {
      this.toChildSearchResult = response.data
      this.loading = false
      this.currentComponent = 'Search'
    })
    .catch(err => {
      alert('APIサーバと接続できません')
      err = null
    })
}
```

こうして得られた検索結果をVueに渡して表示させたのが以下になります．

![Search_Results_Image](:/2023_05_03/Search_Results_image.png)


Googleライクになってきましたね！
画像の下の方（コンソール）をよく見ると，**データベースへの送信完了**と出ています．
検索実行時にデータベースに検索ログを送信しているので，このようなメッセージが表示されています．
次の章ではデータベースへの接続と送信について，詳しく扱ってみたいと思います．

本章は以上になります．

# 3. DBと繋げる編
バックエンドとDBを繋げることで，Userのログを取得できるようにします．
今回はDBとして**MySQL**を導入します．
↓↓↓MySQLの環境構築と導入の流れはこちらから↓↓↓

https://qiita.com/peartrees/items/e3ae29fed26f0365b130


本章では，FlaskとMySQLの接続方法について紹介します．

FlaskとMySQLを接続する方法は幾つかあります．
有名なものだと**mysql-connector**や**PyMySQL**がありますが，今回はMySQLが公式で出している[**mysql-connector**](https://dev.mysql.com/doc/connector-python/en/)を利用したいと思います．

事前にMySQL側でデータベースとテーブルを作成しておく必要がありますが，FlaskとMySQLを繋ぐプログラムは以下のようになります．


```python:FlaskとMySQLの接続
class Send_SQL(Resource):
    def post(self):
        user_query = request.json["text"]
        dt_now = datetime.datetime.now()
        print("Executing SQL")
        def conn_db():
              conn_to_db = mysql.connector.connect(
                      host = 'XXXX',      # localhostでもOK
                      user = 'XXXX',    # データベースにログインするuser
                      passwd = 'XXXX',     # データベースへのパスワード
                      db = 'XXXX'        # 利用するデータベース
              )
              return conn_to_db

        # SQL文
        sql = ('''
            INSERT INTO logs
                (time, query)
            VALUES
                (%s, %s)
            ''')
        data = [(dt_now, user_query)]
        # sql = 'SELECT * FROM logs'

        try:
              # DBとのconnection確立
              con_to_db = conn_db()
              # カーソルを取得
              cursor = con_to_db.cursor()
              # sql文を投げる
              cursor.executemany(sql, data)
              # cursor.execute(sql)
              con_to_db.commit()
              print(f"{cursor.rowcount} records inserted.")
              cursor.close()
              con_to_db.close()
              # selectの結果を全件タプルに格納
              # rows = cursor.fetchall()
              # print(rows)

        except(mysql.connector.errors.ProgrammingError) as e:
              print('エラー')
              print(e)
```
検索実行時間と検索クエリをInsert文を実行することで，データベースに送信するようにしています．
SQLインジェクションなど，データベースと接続したWebアプリ特有のセキュリティ対策には要注意ですね...


# 4. 詰まった点とまとめ！
- #### Vueファイルを変更したら，その都度buildする必要がある(読むだけならserveでも良い)
当たり前っちゃ当たり前なんですが，コンパイルしないとブラウザで読み込めないので，Vueファイルを変更してアプリが動くかテストする時にはbuildしないといけないです．

- #### 修復不可能なバグに直面して，git cloneで復元した時
プログラムをいじっていたら，バグが次から次へと湧いてくる状況に陥り，git cloneで元に戻しました．その際にnode_moduleなどのモジュール設定までは修復できない事を知り，修復に時間がかかりました．
git cloneで直前の状態に戻しても，各種設定ファイルはreproduceできない: [参考リンクはこちら](https://ysko909.github.io/posts/fix-vue-cli-service-command-not-found-error)
次のコマンドを実行してその辺りを修復しました ```rm -rf node_modules && npm install```  
~~git cloneからやり直すのは鬼のように大変で，gitでコード管理することの重要さに気付かされました~~
（gitで修復できなかったらproject作成の１からやり直しでした）

- #### 取得したjsonのHTMLがVueによってescapeされた時
Vue2.X系では，クロスサイトスクリプティングを防ぐために，出力データにHTMLが含まれている時，サニタイジングしてくれます．この所為で，APIで取得したsnippetに```<b>神戸<b>```のようなHTMLが含まれている時にそのまま文字列として表示されてしまいました．
Vue1.X系では，\{\{\{XXX\}\}\}の形で簡単にescapeできましたが，Vue2.X系では厳しくなり，```<p v-html="XXXX"></p>```のように,```v-html```を使わなければescapeできなくなりました．このことを知らずに３重の中括弧で記述していたのでいつまでも解決せず，時間が溶けました．

--- 

front-endとback-end，データベースの繋がりを意識でき，とても楽しいアプリ開発になりました．次回以降は，検索結果のランキングを考える方に重きを置いてみたいと思います（卒業研究のテーマになるかも）

ここまでご覧いただきありがとうございました！

---
# 参考文献など
[1] [Vue.jsでサジェストを実現するライブラリ3選](https://rightcode.co.jp/blog/become-engineer/vue-js-3-libraries-for-suggestions){:target="_blank"}

[2] [MySQLサーバの起動と停止](https://prog-8.com/docs/mysql-env){:target="_blank"}

[3] [Vue.js + FlaskでWebアプリケーション制作 - herokuにデプロイするまで -](https://qiita.com/Nonta0605/items/5d8fa9a8eda9b3b7bc33#vuejs%E3%81%A8%E3%81%AF){:target="_blank"}
