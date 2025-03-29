---
title: "高校生がazureのアカウントを作ってみる"
emoji: "🔍"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [azure]
published: true
---
# はじめに
Azure OpenAI Serviceを触ってみようかなと思い、ついでにこの記事を制作しました。
Azureアカウントを作成するときに思ったことを実況形式で書き連ねていきます。

# アカウントを作成してみる
とりあえず、azureの公式サイトに飛んでみました。
https://azure.microsoft.com/ja-jp/pricing/purchase-options/azure-account

Azureを無料でお試しくださいをクリック

![](/images/azuresignup/azuremainpage.png)

クリックしたらmicrosoftアカウントのサインインの画面が出たのでサインイン

![](/images/azuresignup/microsoftsignin.png)


以下の画像のようになったら必須項目に入力
* 国/地域　(選択)
* 名　
* 姓
* 電子メールアドレス
* 電話番号　(認証コードが届くので入力)
* 名の読み方
* 姓の読み方
* 郵便番号
* 都道府県 (選択)
* 市区町村
* 町名番地
![](/images/azuresignup/signup.png)

入力して次へを押したら問題発生
クレジットカードによる本人認証が必要なようです。

![](/images/azuresignup/creditproblem.png)

高校生なので当然持ってるわけもなく...
なので、親に許可を取って親の名義でアカウント作ることにしました
入力して、次に進むと、

![](/images/azuresignup/complete.png)

いけました！
そして、Go to Azure potalをクリック

![](/images/azuresignup/final.png)

クイックスタートセンターに到着。
ここで初めての方用に説明を見れるみたいです

# まとめ
なんとかアカウントを作成することに成功しました！
無料トークンを使ってAIで何か作ってみたいと思います。
クレジットカードが必要だなんて学生に優しくないなと思っていたのですが、
後々、調べていたらAzure for Studentsというものがあるそうです。
利用すればクレジットカード不要、しかもUSD100＄分のクレジットももらえるそうです
早めに知っていればと後悔。早速試してみます。

(追記)学校のメールアドレスを入力して試してみたが(2025/03/29)時点で有効期限が過ぎていたため使用できず来年度に期待