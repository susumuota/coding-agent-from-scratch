# Coding Agent from Scratch

Follow [this tutorial](https://azukiazusa.dev/blog/build-your-own-coding-ai-agent/) to build a coding agent from scratch using the Google Gemini API and the `ai` package.

- Create a repository `coding-agent-from-scratch`.

```shell
mkdir -p coding-agent-from-scratch
cd coding-agent-from-scratch

corepack enable pnpm
pnpm init
git init
```

- Modify `package.json`.

Add this line.

```json
{
  "type": "module"
}
```

- Install dependencies.

```shell
pnpm install ai @ai-sdk/google zod dotenv
pnpm install --save-dev @types/node tsx typescript
```

- Create `.env`.

```shell
GOOGLE_GENERATIVE_AI_API_KEY="your api key"
```

- Create `.gitignore`.

```gitignore
.env
.vscode
node_modules
```

- Create `src/index.ts`.

```shell
mkdir -p src
touch src/index.ts
```

```typescript
import { google } from "@ai-sdk/google";
import { CoreMessage, streamText } from "ai";
import dotenv from "dotenv";
import * as readline from "node:readline/promises";

// const modelId = "gemini-2.5-pro-exp-03-25";
const modelId = "gemini-2.0-flash";

// 環境変数を .env ファイルから読み込む
dotenv.config();

// 標準出力と標準入力を使用して、ユーザーとの対話を行うためのインターフェースを作成
const terminal = readline.createInterface({
  input: process.stdin,
  output: process.stdout,
});

const messages: CoreMessage[] = [];

async function main() {
  while (true) {
    // ユーザーからの入力を待機
    const userInput = await terminal.question("You: ");

    // ユーザー入力をチャットの履歴として追加
    messages.push({ role: "user", content: userInput });

    // streamText 関数はストリーミングで応答を生成する
    const result = streamText({
      // AI モデルを指定
      // 引数で最新のモデルを指定している
      model: google(modelId),
      messages,
    });

    let fullResponse = "";

    terminal.write("\nAssistant: ");
    // ストリーミングをチャンクごとに処理
    for await (const delta of result.textStream) {
      fullResponse += delta;
      // 受信したチャンクを標準出力に書き込む
      terminal.write(delta);
    }
    terminal.write("\n\n");

    // チャットの履歴に AI の応答を追加
    messages.push({ role: "assistant", content: fullResponse });
  }
}

main().catch(console.error);
```

- Run

```shell
pnpm start
```
