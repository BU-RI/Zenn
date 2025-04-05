---
title: "高校生がDifyさわってみる"
emoji: "🤖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [ dify, azure, 初心者 ]
published: false
---

# はじめに
今度作ろうと思っているアプリのために Dify についてまとめようと思い、
この記事を作成しました。

## 前提知識
といっても自分自身も習いたてなのでAIに聞いてみました。

Dify とは、LLMアプリケーションを簡単に作成・デプロイ・運用できるオープンソースのプラットフォームです。

主な特徴は以下の通りです:

- LLMベースのアプリケーションを、コーディング不要で作成可能
  - 例: ドラッグ&ドロップでチャットボットを作成できる
- OpenAI、Azure OpenAI Service、Anthropic など、複数のLLMプロバイダーに対応　
  - 例: GPT-4 や Claude 2 などの最新モデルを利用可能
- プロンプトエンジニアリング、RAG(Retrieval Augmented Generation) などの機能を搭載
  - 例: 独自のデータを学習させて回答の精度を向上
- APIを通じて作成したアプリケーションを外部サービスと連携可能
  - 例: Slack や Discord などのチャットツールと連携できる

だそうです。
この記事では、Azure OpenAI Service と連携してチャットボットを作成する方法を説明します。


# ログイン
まず Dify に SignIn しましょう
https://dify.ai/jp

![](/images/difystart/firstpage.png)

なんか青くてうにょうにょしているページが開けたら、始める をクリック

![](/images/difystart/signin.png)

アカウント作成には GitHub Google メールアドレスのどれかを使用し、
ログインすることができます。

# アプリの作成

ログインができたら下のような画面になります。

![](/images/difystart/home.png)

そうしたらアプリを作成するの中にある、最初から 作成 を選択し
チャットボット を選択
その後、アプリのアイコンと名前と説明(任意)を書いてください。
作成する を押したら作ったアプリの構成のページになります

![](/images/difystart/configuration.png)

# LMMプロバイダーキーの設定
見ての通りLMMプロバイダーキーの設定ができていません...

![](/images/difystart/settinglmmstart.png)

ここにある設定に移動からも設定できますが。
追加したいときのために一般的な方法でやりましょう。
自分のアカウントのアイコンをクリックし、設定 を選択。

<![](/images/difystart/acount.png)

設定のページでワークスペース内にある モデルプロバイダー を選択
下のような画面になったら、

![](/images/difystart/install.png)

Azure OpenAI をインストールし、
モデルを追加+ をクリックする

![](/images/difystart/modelclick.png)

クリックしたらLMMの各種項目の値を入力
<!-- 上の文章の使い方が不安 -->
* Deployment Name
* API Endpoint URL
* API Key
* API Version(選択)　デプロイした時のurlに書いてある`?api-version=2024-08-01-preview`
* Base Model(選択)

![](/images/difystart/settinglmm.png)

出来たら構成ページに戻り、赤丸でくくったところをクリック

![](/images/difystart/selectyellow.png)

そして モデルを設定する でさっき追加したモデルを選択

![](/images/difystart/selectAI.png)

これで準備完了です！


# 動作確認

チャットボックスに質問したいことを入力すると

![](/images/difystart/result.png)

画像のようにAIが返答してくれました！！

# まとめ
チャットをするところまで行けましたか？
今回の事だけだと「 Azure だけでよくね？」と思うかもしれません。
ですが、Difyのすごいところはコーディングではなくワークフローで
比較的簡単にAIを作成できる点です。

![](/images/difystart/workflow.png)
お見せした画像はテンプレートです。

この機能以外にもRAGを簡単に作成できる、複数のAIモデルを組み合わせるなどの
たくさんの素晴らしい機能があります。
自分はほんの少し触った程度なのでまだまだ勉強していきます。
