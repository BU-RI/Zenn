---
title: "高校生がRagを使って学校用AIを作る"
emoji: "🏫"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [nextjs]
published: false
---
# はじめに

高校の活動でRagを使った学校用のAIを作成したので、主にRagのことと軽いハンズオンを高校生なりの解釈で記事に書いていきます。

# 前提知識

## RAGって何?

RAGは「検索で補強された生成」という意味です。簡単に言うと:

1. 質問に関連する情報を検索して集める
2. その情報を使って、より正確な回答を生成する

という2つのステップで動く仕組みです。


例えば:
- 「明日の天気は?」と聞かれたら、天気予報データベースから最新情報を検索
- 「この学校の校則は?」と聞かれたら、学校の規則集から関連部分を探す
- それらの情報を基に、人間が理解しやすい形で回答を作成

このように独自のデータベースを使用する事によってオンリーワンのAIを制作できます。

## これから作るもの

この記事ではNext.jsとAzure Open AI Serviceとcosmosdbを使用し学校の時間割をデータベースにした時間割AIを作っていきます。

![RAGの仕組み](/images//schoolai/screanshot.png)
※今回ハンズオンするものとは少々異なります
それではハンズオンしていきましょう✋

# 環境構築

### 必要なツール
- Node.js (バージョン18.17.0以上)
- npm (Node.jsに付属)
- コードエディタ (VSCode推奨)

### プロジェクトの作成

1. 新しいNext.jsプロジェクトを作成します：

```bash:bash
npx create-next-app@latest school-rag-app
```

以下の質問が表示されたら、次のように回答してください：

```bash:bash
✔ Would you like to use TypeScript? · Yes
✔ Would you like to use ESLint? · Yes
✔ Would you like to use Tailwind CSS? · Yes
✔ Would you like to use `src/` directory? · Yes
✔ Would you like to use App Router? · Yes
✔ Would you like to customize the default import alias? · No
```

2. プロジェクトディレクトリに移動：

```bash:bash
cd school-rag-app
```

3. 開発サーバーを起動：

```bash:bash
npm run dev
```

これで、`http://localhost:3000` にアクセスすると、Next.jsのデフォルトページが表示されます。

### 必要なパッケージのインストール

RAGシステムを構築するために必要なパッケージをインストールします：
以下はpackage.jsonです。

```json:package.json
{
  "name": "school-rag-app",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  },
  "dependencies": {
    "@azure/openai": "^1.0.0-beta.12",
    "@azure/cosmos": "^4.0.1-beda.3",
    "@azure/storage-blob": "^12.18.0",
    "@reduxjs/toolkit": "^2.2.3",
    "@types/node": "20.12.10",
    "@types/react": "18.3.1",
    "@types/react-dom": "18.3.0",
    "autoprefixer": "10.4.19",
    "axios": "^1.7.2",
    "framer-motion": "^11.1.9",
    "next": "14.2.3",
    "postcss": "8.4.38",
    "react": "18.3.1",
    "react-dom": "18.3.1",
    "react-icons": "^5.2.1",
    "react-redux": "^9.1.2",
    "tailwindcss": "3.4.3",
    "typescript": "5.4.5"
  },
  "devDependencies": {
    "@types/node": "^20",
    "@types/react": "^18",
    "@types/react-dom": "^18",
    "eslint": "^8",
    "eslint-config-next": "14.2.2",
    "postcss": "^8",
    "tailwindcss": "^3.4.1",
    "typescript": "^5"
  }
}

```
package.json内のdependenciesをコピーしてください
出来たらインストール

```bash:bash
npm install
```

これで基本的な環境構築は完了です。次のセクションでは、実際のアプリケーション開発に進んでいきます。

# 実装

最終的なディレクトリ配下はこのようになります

![RAGの仕組み](/images/schoolai/screanshot_directory.png)

まず環境変数を設定するため、ルート階層に.env.localを作成します

```.env:.env.local  
AZURE_OPENAI_ENDPOINT="xxxxxxxxxxxxxxxxxxxxxxx"
AZURE_OPENAI_API_KEY="xxxxxxxxxxxxxxxxxxxxxxx"
AZURE_OPENAI_DEPLOYMENT_ID="xxxxxxxxxxxxxxxxxxxxxxx"
AZURE_OPENAI_VEC_DEPLOYMENT_ID="xxxxxxxxxxxxxxxxxxxxxxx"
COSMOS_CONNECTION_STRING="xxxxxxxxxxxxxxxxxxxxxxx"
COSMOS_DATABASE_NAME="xxxxxxxxxxxxxxxxxxxxxxx"
COSMOS_CONTAINER_NAME="xxxxxxxxxxxxxxxxxxxxxxx"
AZURE_STORAGE_ACCOUNT_NAME="xxxxxxxxxxxxxxxxxxxxxxx"
AZURE_STORAGE_ACCOUNT_ACCESS_KEY="xxxxxxxxxxxxxxxxxxxxxxx"
AZURE_STORAGE_CONTAINER_NAME="xxxxxxxxxxxxxxxxxxxxxxx"
```

appの配下に、utilディレクトリを作成し、その下にさらに、
extra-1ディレクトリを作成してください

![](/images/schoolai/screanshot_extra-1.png)

extra-1/blob.tsを作成してください
```ts:extra-1/brob.ts

import {
    BlobServiceClient,
    StorageSharedKeyCredential,
  } from '@azure/storage-blob';
  
  export const getBase64File = async (file_path: string): Promise<string> => {
    return new Promise(async (resolve, reject) => {
      const sharedKeyCredential = new StorageSharedKeyCredential(
        process.env.AZURE_STORAGE_ACCOUNT_NAME!,
        process.env.AZURE_STORAGE_ACCOUNT_ACCESS_KEY!
      );
      const blobServiceClient = new BlobServiceClient(
        `https://${process.env.AZURE_STORAGE_ACCOUNT_NAME}.blob.core.windows.net`,
        sharedKeyCredential
      );
      const containerClient = blobServiceClient.getContainerClient(
        process.env.AZURE_STORAGE_CONTAINER_NAME!
      );
      const blobClient = containerClient.getBlobClient(file_path);
  
      const downloadResponse = await blobClient.download(0);
  
      let encodedData = '';
      if (downloadResponse.readableStreamBody) {
        const downloaded = await streamToBuffer(
          downloadResponse.readableStreamBody
        );
        const encodedData = downloaded.toString('base64');
        resolve(encodedData);
      } else {
        reject('readableStreamBody is undefined');
      }
  
      resolve(encodedData);
    });
  };
  
  async function streamToBuffer(
    readableStream: NodeJS.ReadableStream
  ): Promise<any> {
    return new Promise((resolve, reject) => {
      const chunks: any[] = [];
      readableStream.on('data', (data) => {
        chunks.push(data instanceof Buffer ? data : Buffer.from(data));
      });
      readableStream.on('end', () => {
        resolve(Buffer.concat(chunks));
      });
      readableStream.on('error', reject);
    });
  }
  ```
  次にextra-1/cosmos.tsを作成します
  ```ts:extra-1/cosmos.ts
  import { CosmosClient } from '@azure/cosmos';


export const getItemsByVector = async (embedding: number[]): Promise<any[]> => {
  return new Promise(async (resolve, reject) => {
    const cosmosClient = new CosmosClient(
      process.env.COSMOS_CONNECTION_STRING!
    );
    const database = cosmosClient.database(process.env.COSMOS_DATABASE_NAME!);
    const container = database.container(process.env.COSMOS_CONTAINER_NAME!);

    const { resources } = await container.items
      .query({
        query:
          'SELECT TOP 10 c.file_name, c.content, c.is_contain_image, c.image_blob_path, VectorDistance(c.content_vector, @embedding) AS SimilarityScore FROM c WHERE VectorDistance(c.content_vector, @embedding) > 0.50 ORDER BY VectorDistance(c.content_vector, @embedding)',
        parameters: [{ name: '@embedding', value: embedding }],
      })
      .fetchAll();
    for (const item of resources) {
      console.log(
        `🚀${item.file_name}, ${item.content}, ${item.image_blob_path}, ${item.SimilarityScore} is a capitol \n`
      );
    }
    resolve(resources);
  });
};

```
次にextra-1/openai-extra-shrkm.tsを作成します
```ts:openai-extra-shrkm.ts
import { AzureKeyCredential, OpenAIClient } from '@azure/openai';

export const getChatCompletions = async (
  systemMessage: string,
  message: string,
  images: string[]
): Promise<any[]> => {
  console.log('start', process.env.AZURE_OPENAI_ENDPOINT!);
  return new Promise(async (resolve, reject) => {
    const endpoint = process.env.AZURE_OPENAI_ENDPOINT!;
    const azureApiKey = process.env.AZURE_OPENAI_API_KEY!;
    const deploymentId = process.env.AZURE_OPENAI_DEPLOYMENT_ID!;

    const client = new OpenAIClient(
      endpoint,
      new AzureKeyCredential(azureApiKey)
    );

    let messages;
    if (images.length > 0) {
      try {
        const response = await client.getChatCompletions(
          deploymentId,
          [
            { role: 'system', content: systemMessage },
            { role: 'user', content: message },
            {
              role: 'user',
              content: [
                {
                  type: 'image_url',
                  imageUrl: {
                    url: `data:image/jpeg;base64,${images[0]}`,
                  },
                },
              ],
            },
          ],
          { maxTokens: 4096 }
        );
        resolve(response.choices);
      } catch (error: any) {
        reject(error);
      }
    }
    else {
      try {
        const response = await client.getChatCompletions(
          deploymentId,
          [
            { role: 'system', content: systemMessage },
            { role: 'user', content: message },
          ],
          { maxTokens: 4096 }
        );
        resolve(response.choices);
      } catch (error: any) {
        reject(error);
      }
    }
  });
};

export const getEmbedding = async (message: string): Promise<number[]> => {
  return new Promise(async (resolve, reject) => {
    const endpoint = process.env.AZURE_OPENAI_ENDPOINT!;
    const azureApiKey = process.env.AZURE_OPENAI_API_KEY!;
    const deploymentId = process.env.AZURE_OPENAI_VEC_DEPLOYMENT_ID!;
    
    const client = new OpenAIClient(
      endpoint,
      new AzureKeyCredential(azureApiKey)
    );
    const embedding = await client.getEmbeddings(deploymentId, [message]);
    resolve(embedding.data[0].embedding);
    
  });
};
```
そして、app配下にapiディレクトリを作成し、rag-extra-1/route.tsを
作成してください。
```ts:rag-extra-1/route.ts

import { getBase64File } from '../../util/extra-1/blob';
import { getItemsByVector } from '../../util/extra-1/cosmos';
import {
  getChatCompletions,
  getEmbedding,
} from '../../util/extra-1/openai-extra-shrkm';
import { NextRequest } from 'next/dist/server/web/spec-extension/request';
import { NextResponse } from 'next/dist/server/web/spec-extension/response';

export const POST = async (req: NextRequest) => {
  try {
    const { message } = await req.json();

    console.log('🚀RAG-extra用のAPIルート');

    console.log('🚀Get embedding from Azure OpenAI.');
    const embeddedMessage = await getEmbedding(message);

    console.log('🚀Search vector from Azure CosmosDB.');
    const cosmosItems = await getItemsByVector(embeddedMessage);

    console.log('🚀Create system message and image_content.');
    let systemMessage =
      'あなたは優秀なアシスタントです。"# 検索結果" を言及されたときにのみ"# 検索結果"について答えてください。\n';
    systemMessage += '# 検索結果\n';
    let images = [];
    for (const result of cosmosItems) {
      systemMessage +=
        '## ' +
        (cosmosItems.indexOf(result) + 1) +
        '\n' +
        result.content +
        '\n\n';
      if (result.is_contain_image === true) {
        const image = await getBase64File(result.image_blob_path);
        images.push(image);
      }
    }
    console.log('🚀systemMessage:', systemMessage);

    const result = await getChatCompletions(systemMessage, message, images);
    let aiMessage = result[0].message.content;

    return NextResponse.json({ aiMessage }, { status: 200 });
  } catch (error: any) {
    return NextResponse.json({ aiMessage: error.message }, { status: 500 });
  }
};

export const dynamic = 'force-dynamic';
```
最後に、appの配下にpage.tsxを作成します
```tsx:page.tsx
'use client'
import Link from "next/link";
import { useState } from "react";

export default function Home() {
  const [text, setText] = useState("");
  const [content, setContent] = useState<string>('おこたえします');
  const message = text 

  const sendMessage = async () => {
    
    const url = '/api/rag-extra-1';
    console.log('🚀 ~ sendMessage ~ url:', url);
    const response = await fetch(`${url}`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({ message }),
    });
    
    const  aiMessage  = await response.json();
    const data = typeof aiMessage.aiMessage === 'string' 
        ? aiMessage.aiMessage 
        : JSON.stringify(aiMessage.aiMessage);
    setContent(data);
    
  }; 

  return (
    <div>
      <div className="div">
      <Link href={"/Login"} className="login"> <img src="signin.png" alt="image" className="img2"/></Link>
      <form>
        <input
          type="text"
          onChange={(e) => ( setText(e.target.value) )}
          className="input"
          placeholder="聞きたいことを入力してください" required>
        </input>

        <button className="submit" type="button" onClick={sendMessage}>⇓</button>


      </form>  

      </div>
      <div className="ai">
      <p className="answer">{content}</p>
      </div>
    </div>
  );
}
```
cssも追加します
```css:global.css


body,html {
  margin: 0px;
  padding: 0px;
}

.center{
    text-align: center;
    background: #6d368e;
    padding: 0.15em;
    color: white;
}

.img{
  width: 360px;
  height: 75px;
}

.img2{
  width: 200px;
  height: 75px;
}


.ai {
    margin-top: 10px;
    height: 100px;
    width: 95%;
    border-width: 5px;
    border-color: rgb(83, 151, 239);
    border-style: ridge;
}

.answer{
    color: rgb(0, 0, 0);
    margin-top: 40px;
    margin-bottom: 40px;
    margin-left: 5px;
}

.div{
    text-align: center;
}

.input{
    width: 50%;
    height: 40px;
    font-size: 20px;
}

.submit{
    border: 0;
    line-height: 2.5;
    padding: 0 20px;
    font-size: 1rem;
    text-align: center;
    color: #fff;
    text-shadow: 1px 1px 1px #ffffff;
    border-radius: 10px;
    background-color: rgb(0, 0, 0);
    background-image: linear-gradient(
      to top left,
      rgba(0, 0, 0, 0.2),
      rgba(0, 0, 0, 0.2) 30%,
      rgba(0, 0, 0, 0)
    );
    box-shadow:
      inset 2px 2px 3px rgba(255, 255, 255, 0.6),
      inset -2px -2px 3px rgba(0, 0, 0, 0.6);
}
  
  .submit:hover{
    background-color: rgb(102, 102, 102);
  }
  
  .submit:active{
    box-shadow:
      inset -2px -2px 3px rgba(102, 102, 102, 0.6),
      inset 2px 2px 3px rgba(0, 0, 0, 0.6);
  }

```
## 動作確認
今回は私の学校の時間割が入っています
![](/images/schoolai/screanshot_result.png)
このように箱の中に結果が出たら成功です

# まとめ
いかがだったでしょうか？
今回はragを使用し独自のデータを持ったAIをハンズオンしてもらいました。
うまくいきましたでしょうか？
僕の場合は、作るとき、ragのことについてすら知らなかったので、大変苦労しました。
だれか困っている人の助けになったらうれしいです。
