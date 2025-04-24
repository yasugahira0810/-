# バイブコーディングで自分が欲しいアプリを作ろう!
2024年4月
安ケ平 雄太

---
## はじめに
- バイブコーディングという新たなプログラミングのパラダイムが生まれようとしている
- 最近コーディングしていない人こそ、乗るしかない、このビッグウェーブに

---
## 2025年2月 GitHub Copilot Agent Mode Release!
![[Agent-Sunrise-1.png]]
- やっと、普段プログラミングをしない自分でも、GitHub Copilotを使いこなせるかもしれないと思った

---
## それまでのGitHub Copilot
![[Screenshot 2025-04-23 at 0.02.21.png]]
- 認定資格の勉強のため、GitHub Copilotを触り始めて驚いた
	- GitHub Copilotは、新規ファイル作成すらできない！
	- 主人公には見えるけど、現実世界には影響しないキャラクターみたい
![[20190626121320.jpg]]![[char08.jpg]]

---
## GitHub Copilot Agent Modeを使おうと決めた理由
- 仕様をmarkdownで書けば、コードを書かなくても、自分が作りたいアプリを作れるのでは？と思った（≒バイブコーディング）
- GitHub Copilotの無料版で試して、ダメならやめればいいと思った

---
## バイブコーディングとは
- 生成AIの進歩により、開発者と生成AIが対話を通じて"ノリ"(≒Vibe)でコーディングをする新たなプログラミング手法。2025年2月にXに投稿されたポストがきっかけで広がった
	- コードは一切自分で書かず、AIに書かせる
	- 基本的に、AIの提案するコードをそのまま受け入れる
	- エラーメッセージが出たら、チャットにコピペして直してもらう

---
## 自分が作りたいアプリ
- VPNサービスの残りデータ容量を可視化するアプリ
	- 月毎の利用上限をオーバーして困ることがあるので、可視化したかった
	- VPNサービス側がこの機能を実装するとは思えなかったので、自分で作るしかないと思った

---
## アーキテクチャ
![[UCSS_proto_architecture.drawio (1).png]]

-------
## リポジトリの中で唯一書いたファイル”design.md”

### 概要：
- VPNサービス「UCSS」の残りのデータ通信量をグラフ化するアプリ「UCSSMonitor」
- UCSSMonitorはユーザーに代わってUCSSにメールアドレスとパスワードを入力して、「サービスの詳細」ページからスクレイピングによってデータ通信量を確認してGistに記録し、グラフ化する（以降、これをジョブと呼ぶ）
- ジョブは1時間毎に実行され、実行結果として記録「日時」と「残りのデータ通信量」を記録する
- ユーザはアプリへのログインなしでグラフを確認できる（GitHubへのログインはしても良い）
- UCSSの費用以外は全て無料で実現する
- 開発者のみがユーザの想定である

### 技術スタック：
- フロントエンド: Vue.js + Chart.js
- バックエンド: node.js
- 実行環境: GitHub Actions
- データ保存: Gist（JSONフォーマット）
- 認証情報管理: GitHub ActionのSecrets
- グラフの表示先: GitHub Pages
- スクレイピング: Puppeteer

### 必要な機能：
- メールアドレスとパスワードをGitHub ActionsのSecretsで安全に管理
- GitHub Actionsで1時間ごとにジョブを実行し、PuppeteerでUCSSにログイン
- ログインしてログイン後のページに遷移したら、URLとログイン成功のメッセージをログに出力する
- ログイン失敗時にはエラーメッセージが表示されるので、その文字列を検出して、ログイン失敗を伝える
- ログイン後のページに「サービス詳細」ページへのリンクのCSSセレクタが表示されるまで待つ
- UCSSの「サービスの詳細」ページへ遷移して残りデータ通信量を取得
- Gist(https://gist.github.com/(process.env.GIST_USER)/(process.env.GIST_ID))を、データ保存（Gist）のフォーマットに従って、更新
- Gistの更新を契機に次のGitHub Actionsを動かして、横軸がdateで、縦軸がremainingDataの積み上げ折れ線グラフをChart.jsを使ってindex.htmlを生成
- 再生成したグラフをpeaceiris/actions-gh-pages@v4を使って、GitHub Pagesへデプロイ
- 残りデータ容量が増えたときには、（増えた時刻、残りデータ量）の地点から、（1ヶ月後の時刻、0GB）の地点まで、補助線を引く。これにより、ユーザは平均ペースに比べて早くデータ量を消費しているから使用を控えるか、データ量余る見込みだから、積極的に使うかを判断できる。

### セキュリティ：

- GitHub SecretsでUCSSのメールアドレスとパスワードを安全に管理
- リポジトリはパブリックリポジトリ

### データ保存（Gist）のフォーマット：
- Gistに保存するデータは以下のJSON形式：
```json
[
  {"date": "2025-03-22T12:00:00Z", "remainingData": 50.9},
  {"date": "2025-03-22T13:00:00Z", "remainingData": 50.4}
]
```

### UCSSのURLやCSSセレクタやXPath
- [ログインページ](https://my.undercurrentss.biz/index.php?rp=/login)
- [ログイン後のページ](https://my.undercurrentss.biz/clientarea.php)
- ログインページのログインボタンのCSSセレクタ #login
- ログインページのメールアドレス入力欄のCSSセレクタ #inputEmail
- ログインページのパスワード入力欄のCSSセレクタ #inputPassword
- ログインページでログイン失敗した時のエラーメッセージのCSSセレクタ body > div.app-main > div.main-body > div > div > div > div > div > div
- ログインページから「サービス詳細」ページへのリンクのCSSセレクタ　#ClientAreaHomePagePanels-Active_Products_Services-0 > div > div.list-group-item-actions > button
- 「サービス詳細」ページの残りデータ通信量を示すテキストのCSSセレクタ #traffic-header > p.free-traffic > span.traffic-number
- [GitHubのユーザ名](yasugahira0810)
- [Gistのパス](https://gist.github.com/yasugahira0810/ec00ab4d6ed6cdb4f1b21f65377fc6af)

### GitHub ActionsのSecrets
- UCSS_EMAIL: UCSSのログイン用メールアドレス
- UCSS_PASSWORD: UCSSのログイン用パスワード
- GH_PAT: Gist更新用のPersonal Access Token。PAT生成時に付与する権限はGistのみでOK。
- GIST_USER: 更新対象のGistファイルを保有するユーザ名
- GIST_ID: 更新対象のGist

## GitHub Pagesの設定
- Build and deploymentのSourceはDeploy from a branch。Branchはgh-pagesの/(root)。
---
## 感想
- 設計書を書くだけで、自分が欲しいアプリを作ることができた！
- 面倒くさくてやらないところも全部やってくれたので、保守性が高い
	- テストコード、readmeの作成
	- コードからの詳細設計書、シーケンス図、クラス図のリバース作成

---
## バイブコーディングは誰のもの？
- プログラミング未経験者のためのものではない
	- プログラミングの流れがわかっていないと使えない
- 仕事でコードを書く開発者のためのものである
	- [The End of Programming as We Know It](https://www.oreilly.com/radar/the-end-of-programming-as-we-know-it/)
	- `熟練のプログラマであり、確かな技術ウォッチャーであるSteve Yegge氏は、AIに取って代わられるのはジュニアおよび中級レベルのプログラマではなく、新しいプログラミング ツールやパラダイムを受け入れず過去に固執するプログラマであると指摘しています。新しいスキルを習得もしくは開発できる人は高い需要があるでしょう。AIツールを習得したジュニア開発者であれば、習得していないシニアプログラマーよりも優れたパフォーマンスを発揮できるのです。`
- 開発者以外のプロダクト開発の関係者のものでもある
	- プロジェクトマネージャー
	- プロダクトオーナー
	- デザイナー

---
## おわりに
- 開発者、PM、PO、デザイナーなどどの職種であっても、生成AIのおかげでできる仕事の幅が広くなるので、役割分担も現在とは変わりそう
- 開発者、最近開発していない人も、ぜひ一度触って、「私たちが知っているプログラミングの終」と「新たなプログラミングのパラダイム」を実感してみてほしい