---
title: "高校生でもできる! DifyをNext.jsに実装"
emoji: "💻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [ dify, nextjs, 初心者 ]
published: false
---

# はじめに
Dify を使用した web アプリを制作する過程で必要になったので記事に軽いハンズオンと一緒にまとめました。
この記事で学べるのは、apiキーの取得する方法から実装までです。

## Dify初めての方へ

https://zenn.dev/bu_ri/articles/dify_chat_highs

Difyのアカウント作成からのアカウント作成からチャットボット作成までの手順があります👆

# 製作開始！！

まずは新しくAIを作っていきましょう。

## AI作成

![](/images/difynextjs/newproject.png)

そしたらAIの種類はチャットボットのままでアプリ名を決めたら作成

![](/images/difynextjs/selectaitype.png)

そうしたらアプリに使うAIのモデルを選択し、元となるチャットボット

![](/images/difynextjs/completeAI.png)

APIにアクセスをクリック

![](/images/difynextjs/apipage.png)

すると Dify の API について色々書かれたタブに行きます。
そして右上にあるAPIキーをクリックして、新しいシークレットキーを取得 を
クリックしてキーを取得

![](/images/difynextjs/apikey.png)

ここには API について一つ一つ書かれていますがどれを使えばいいのか
こんがらがる(実際にこんがらがった💦)ので、簡単に今回使うものについてまとめます。
下にスクロールしたらでてくるcurlコマンドだけです。


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

これを実行するとこのようになります

![](/images/difynextjs/curlresult.png)

よく見たらanswerのところにレスポンスが帰ってきているのがわかります。

## next.jsで開発

### セットアップ

まず、next.jsのセットアップをしましょう。
アプリを作りたいディレクトリの中で以下のコマンドを実行します。

```
npx create-next-app@latest dify_next_app
```
今回、package.jsonはこのようになっています。

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

### apiを叩く
ではapiを叩いていきましょう。
apiというディレクトリを作成し、さらにその中にchatというディレクトリを作成。
そして、route.tsファイルをその中に作成。
出来たら以下のコードを入力
Authorizationのなかにある`YOUR-SECRET-KEY`は取得したAPIキーを入れてください

```ts:chat/route.ts
import { NextResponse } from 'next/server';

export async function POST(request: Request) {
  try {
    const { message } = await request.json();

    if (!process.env.NEXT_PUBLIC_DIFY_API_KEY) {
      throw new Error('DIFY_API_KEY is not set in environment variables');
    }

//ここがapiの主な部分です
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

### フロント側の作成
では、画面側を作っていきましょう
構成としては、タイトル、入力フォーム、送信ボタン、その下にレスポンスが表示されるというシンプルなものにしました。
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
    <div className="min-h-screen p-4 bg-gray-50">

        <h1>Difyチャット</h1>
        
        <form onSubmit={handleSubmit} className="mb-4">
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

          <div >
            <p >{answer}</p>
          </div>

    </div>
  );
} 
```
これで完了です！
実際に動かしてみましょう

# 動作確認

実行して動作確認をしてみましょう

```bash
npm run dev
```
実行直後の画面はこのような感じになります。

![](/images/difynextjs/final.png)

実際に質問してみたらちゃんと返ってきました😊

![](/images/difynextjs/finalresult.png)

# 最後に
このようにすることにより、Difyで作ったAIをnext.jsに落とし込むことができます。
みなさんの今後webアプリ制作に参考になったのなら幸いです。
これからもアプリ制作頑張っていきます！