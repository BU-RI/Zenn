---
title: "高校生でもできる! DifyをNext.jsに実装"
emoji: "🚀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [ dify, nextjs, 初心者, ai, チャットボット ]
published: true
---

‎
‎

![](/images/difynextjs/samune.drawio.png)

‎
‎

# 🎯 はじめに
こんにちは！今回はDifyを使用したwebアプリの制作過程を、ハンズオン形式でご紹介します！
この記事を読むことで、APIキーの取得から実装まで、一連の流れを理解することができます。

![](/images/difynextjs/finalresult.png)

**完成イメージはこんな感じ！** シンプルで使いやすいチャットボットが作れます✨

## 📊 アプリケーションの流れ

![](/images/difynextjs/finaldifynext.drawio.png)

このアプリはこのような流れで動作します。
1. ユーザーがメッセージを入力
2. フロントエンドからNext.jsのAPI Routeにリクエストを送信
3. API RouteがDify APIを呼び出し
4. Difyからのレスポンスをフロントエンドに返す
5. ユーザーにレスポンスを表示

## 📚 前提知識

https://zenn.dev/bu_ri/articles/dify_chat_highs

Difyのアカウント作成からチャットボット作成までの手順はこちらの記事で解説しています👆
まずはこちらを参考に、Difyのアカウントを作成してください！

# 🛠️ 製作開始！！

それでは、実際にAIを作っていきましょう！

## 🤖 AI作成かAPIキー取得まで

### 1. プロジェクト作成
![](/images/difynextjs/newproject.png)

### 2. AIタイプの選択
チャットボットを選択し、アプリ名を決めましょう！
![](/images/difynextjs/selectaitype.png)

### 3. AIモデルの設定
使用するAIモデルを選択し、基本設定を完了させます
![](/images/difynextjs/completeAI.png)

### 4. API設定
APIにアクセスをクリックして、次のステップへ進みましょう
![](/images/difynextjs/apipage.png)


### 5. APIキーの取得
APIキーを取得するために、以下の手順で進めていきましょう。
![](/images/difynextjs/apikey.png)

:::message alert
**重要！**
取得したAPIキーは必ず安全に保管してください。他人と共有しないようにしましょう。
:::

### 6. APIの動作確認
実際にAPIが動作するか確認してみましょう。
以下のcurlコマンドを実行して、レスポンスを確認します。

```bash
curl -X POST 'https://api.dify.ai/v1/chat-messages' \
--header 'Authorization: Bearer YOUR-SECRET-KEY' \
--header 'Content-Type: application/json' \
--data-raw '{
    "inputs": {},
    "query": "What are the specs of the iPhone 13 Pro Max?",
    "response_mode": "streaming",   
    "conversation_id": "",
    "user": "abc-123",
    "files": [
      {
        "type": "image",
        "transfer_method": "remote_url",
        "url": "https://cloud.dify.ai/logo/logo-site.png"
      }
    ]
}''
```

実行結果はこのようになります：
![](/images/difynextjs/curlresult.png)

`answer`の部分にレスポンスが返ってきているのが確認できますね！

## 💻 Next.jsで開発

### 1. セットアップ
まずはNext.jsのプロジェクトを作成しましょう。

```bash
npx create-next-app@latest dify_next_app
```

### 2. 必要なパッケージの確認
`package.json`の内容を確認しましょう。

```json:package.json
{
  "name": "dify",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  },
  "dependencies": {
    "next": "15.2.4",
    "react": "^19.0.0",
    "react-dom": "^19.0.0"
  },
  "devDependencies": {
    "@eslint/eslintrc": "^3",
    "@types/node": "^20",
    "@types/react": "^19",
    "@types/react-dom": "^19",
    "autoprefixer": "^10.4.21",
    "eslint": "^9",
    "eslint-config-next": "15.2.4",
    "postcss": "^8.5.3",
    "tailwindcss": "^3.4.1",
    "typescript": "^5"
  }
}
```

:::message
**インストール**
コピペができたらインストールするのを忘れないでください：
```bash
npm　install
```
:::

### 3. APIの実装
`api/chat/route.ts`を作成し、以下のコードを実装します。

:::message alert
**注意点**
`YOUR-SECRET-KEY`の部分は、先ほど取得したAPIキーに置き換えてください！
:::

```ts:chat/route.ts
import { NextResponse } from 'next/server';

export async function POST(request: Request) {
  try {
    const { message } = await request.json();

    if (!process.env.NEXT_PUBLIC_DIFY_API_KEY) {
      throw new Error('DIFY_API_KEY is not set in environment variables');
    }

    const response = await fetch('https://api.dify.ai/v1/chat-messages', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer YOUR-SECRET-KEY`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        inputs: {},
        query: message,
        response_mode: 'blocking',
        conversation_id: '',
        user: 'user-123',
        files: []
      }),
    });

    if (!response.ok) {
      const errorData = await response.json().catch(() => null);
      console.error('Dify API Error:', {
        status: response.status,
        statusText: response.statusText,
        errorData,
      });
      throw new Error(`Dify API request failed: ${response.status} ${response.statusText}`);
    }

    const data = await response.json();
    return NextResponse.json({ 
      answer: data.answer,
      conversation_id: data.conversation_id 
    });
  } catch (error) {
    console.error('Error details:', {
      message: error instanceof Error ? error.message : 'Unknown error',
      stack: error instanceof Error ? error.stack : undefined,
    });

    return NextResponse.json(
      { 
        error: 'Internal Server Error',
        details: error instanceof Error ? error.message : 'Unknown error occurred',
      },
      { status: 500 }
    );
  }
} 
```

### 4. フロントエンドの実装
シンプルで使いやすいチャットインターフェースを作成します。

```tsx:page.tsx
'use client';

import { useState } from 'react';

export default function Home() {
  const [input, setInput] = useState('');
  const [answer, setAnswer] = useState('');
  const [isLoading, setIsLoading] = useState(false);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    if (!input.trim() || isLoading) return;

    setIsLoading(true);
    try {
      const res = await fetch('/api/chat', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({ message: input }),
      });

      if (!res.ok) {
        throw new Error('API request failed');
      }

      const data = await res.json();
      setAnswer(data.answer);
    } catch (error) {
      console.error('Error:', error);
      setAnswer('エラーが発生しました');
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <div>
      <h1>Difyチャット</h1>
      
      <form onSubmit={handleSubmit}>
        <input
          type="text"
          value={input}
          onChange={(e) => setInput(e.target.value)}
          placeholder="メッセージを入力..."
          disabled={isLoading}
        />
        <button
          type="submit"
          disabled={isLoading}
        >
          {isLoading ? '送信中...' : '送信'}
        </button>
      </form>

      <div>
        <p>{answer}</p>
      </div>
    </div>
  );
} 
```

# 🎉 動作確認

開発サーバーを起動して、実際に動作確認をしてみましょう！

```bash
npm run dev
```

起動直後の画面：
![](/images/difynextjs/final.png)

実際に質問してみると...
![](/images/difynextjs/finalresult.png)

ちゃんとAIが応答してくれました！😊

# 🌟 最後に
これで、Difyで作成したAIをNext.jsアプリケーションに組み込むことができました！
これをテンプレートにうまく活用すれば、このようなチャットアプリもできます

![](/images/difynextjs/example.png)

この記事が、みなさんのWebアプリ制作の参考になれば幸いです。

これからも、より良いアプリ制作を目指して頑張っていきましょう！✨