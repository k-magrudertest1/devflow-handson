*ハンズオンの手順の参照とハンズオンの実施は、ブラウザで別タブか別ウィンドウを開いて行うことをおすすめします。  
*二人一組を想定(開発者とレビューア)していますが、一人でも実施可能です。

# devflow-handson

## 0.事前準備

1. 以下のレポジトリをimportします。Owner は ご自身のユーザスペース or ユーザアカウントを選択してください。repository name は `devflow-training` とします。  
   また、Repository は `Public` で作成します。  
URL：https://github.com/k-magrudertest1/chat-app-demo.git

1. 「devflow-training」という Repository が表示されていればokです。

1. Settings > General > Collaboraors にて、「Add people」をクリックし、共同作業者を招待します。

2. 「Add <共同作業者>」をクリックします。

![0-1.jpg](./img/0/0-1.jpg)

3. Repository に招待された旨の通知が共同作業者に届きます。「Accept invitation」をクリックします。

![0-2.jpg](./img/0/0-2.jpg)

4. 「Accept invitation」をクリックします。

![0-3.jpg](./img/0/0-3.jpg)

## 1.テストの追加と機能追加

1. 「devflow-training」という Repository で作業します。

1. 画面の左上タブの「<> Code」が選択されていることを確認してください。

1. 「main」と表示されている部分をクリックします。

1. 「Find or create a branch」に `feature/show-username` と入力します。

1. 「Create branch *feature/show-username* from *main*」をクリックします。

1. 画面右側、緑色で表示されている「<> Code」をクリックし、「Create codespace on feature/show-user...」をクリックします。

---

1. codespace が起動されたら、ターミナルで現在のアプリの状態を確認します。

1. codespace 内のターミナルで、以下のコマンドを実行し、現状テストにクリアすることを確認します。(テストファイルが2件、テスト項目6件がクリアしていることがわかります。)
   テストを確認後、「q」を押下することで、再度プロンプトを表示することができます。

```
npm test
```

1. codespace 内のターミナルで、以下のコマンドを実行し、コンテナイメージをビルドします。

```
docker build -t test:1 .
```

1. 以下のコマンドで、コンテナイメージが作成されたことを確認します。

```
docker image ls
```

1. 以下のコマンドで、先ほど作成したコンテナイメージを使用し、コンテナを起動します。

```
docker run -dp 3000:3000 test:1
```

1. 画面右下にポップアップが表示されます。「ブラウザーで開く」をクリックします。

1. 「メッセージを入力」に文字を入力し、「送信」をクリックすることで、入力したメッセージが表示されるシンプルなチャットアプリです。

1. 以下のコマンドで、先ほど起動したコンテナを削除します。

```
docker rm -f $(docker ps -qa)
```

---

[//]: # (TODO: テストメソッド名は日本語にします。僕の方でやります)

1. はじめに、「tests/messageHandler.test.js」ファイルを編集します。(以下のファイルをコピー&ペーストし、差分を確認します)

```
import { describe, it, expect } from 'vitest';
import messageHandler from '../src/messageHandler';

describe('MessageHandler', () => {
  describe('validateMessage', () => {
    it('should return false for empty message', () => {
      const data = { username: 'test', message: '' };
      expect(messageHandler.validateMessage(data)).toBe(false);
    });

    it('should return false for empty username', () => {
      const data = { username: '', message: 'hello' };
      expect(messageHandler.validateMessage(data)).toBe(false);
    });

    it('should return false for missing fields', () => {
      const data = { username: 'test' };
      expect(messageHandler.validateMessage(data)).toBe(false);
    });

    it('should return true for valid message', () => {
      const data = { username: 'test', message: 'hello' };
      expect(messageHandler.validateMessage(data)).toBe(true);
    });
  });

  describe('formatMessage', () => {
    it('should format message correctly', () => {
      const data = { message: 'hello' };
      const result = messageHandler.formatMessage(data);
      
      expect(result).toEqual({
        message: 'hello',
        timestamp: expect.any(String)
      });
    });

    it('should trim whitespace', () => {
      const data = { message: ' hello ' };
      const result = messageHandler.formatMessage(data);
      
      expect(result).toEqual({
        message: 'hello',
        timestamp: expect.any(String)
      });
    });
  });
});
```

1. codespace 内のターミナルで、以下のコマンドを実行し、テストを行います。(テストファイル1件、うちテスト項目1件が failed となっていることがわかります。)
   テストを確認後、「q」を押下することで、再度プロンプトを表示することができます。

```
npm test
```

1. 「src/messageHandler.js」ファイルを編集します。(以下のファイルをコピー&ペーストし、差分を確認します)

```
class MessageHandler {
  validateMessage(data) {
    if (!data.username || !data.message) {
      return false;
    }
    return data.username.trim().length > 0 && data.message.trim().length > 0;
  }

  formatMessage(data) {
    return {
      message: data.message.trim(),
      timestamp: new Date().toISOString()
    };
  }
}

module.exports = new MessageHandler();
```

1. codespace 内のターミナルで、以下のコマンドを実行し、テストを行います。(テスト項目がクリアしたことがわかります。)
   テストを確認後、「q」を押下することで、再度プロンプトを表示することができます。

```
npm test
```

1. 再び、「tests/messageHandler.test.js」ファイルを編集します。(以下のファイルをコピー&ペーストし、差分を確認します)

```
import { describe, it, expect } from 'vitest';
import messageHandler from '../src/messageHandler';

describe('MessageHandler', () => {
  describe('validateMessage', () => {
    it('should return false for empty message', () => {
      const data = { username: 'test', message: '' };
      expect(messageHandler.validateMessage(data)).toBe(false);
    });

    it('should return false for empty username', () => {
      const data = { username: '', message: 'hello' };
      expect(messageHandler.validateMessage(data)).toBe(false);
    });

    it('should return false for missing fields', () => {
      const data = { username: 'test' };
      expect(messageHandler.validateMessage(data)).toBe(false);
    });

    it('should return true for valid message', () => {
      const data = { username: 'test', message: 'hello' };
      expect(messageHandler.validateMessage(data)).toBe(true);
    });
  });

  describe('formatMessage', () => {
    it('should format message correctly', () => {
      const data = { username: 'test', message: 'hello' };
      const result = messageHandler.formatMessage(data);
      
      expect(result).toEqual({
        username: 'test',
        message: 'hello',
        timestamp: expect.any(String)
      });
    });

    it('should trim whitespace', () => {
      const data = { username: ' test ', message: ' hello ' };
      const result = messageHandler.formatMessage(data);
      
      expect(result).toEqual({
        username: 'test',
        message: 'hello',
        timestamp: expect.any(String)
      });
    });
  });
});
```

1. codespace 内のターミナルで、以下のコマンドを実行し、テストを行います。(テストファイル1件、うちテスト項目2件が failed となっていることがわかります。)
   テストを確認後、「q」を押下することで、再度プロンプトを表示することができます。

```
npm test
```

1. 「src/messageHandler.js」ファイルを編集します。(以下のファイルをコピー&ペーストし、差分を確認します)

```
class MessageHandler {
  validateMessage(data) {
    if (!data.username || !data.message) {
      return false;
    }
    return data.username.trim().length > 0 && data.message.trim().length > 0;
  }

  formatMessage(data) {
    return {
      username: data.username.trim(),
      message: data.message.trim(),
      timestamp: new Date().toISOString()
    };
  }
}

module.exports = new MessageHandler();
```

1. 「public/index.html」ファイルを編集します。(以下のファイルをコピー&ペーストし、差分を確認します)

```
<!DOCTYPE html>
<html>
<head>
  <title>シンプルチャット</title>
  <meta charset="UTF-8">
  <style>
    body { margin: 0; padding: 20px; font-family: sans-serif; }
    #messages { list-style-type: none; margin: 0; padding: 0; }
    #messages li { padding: 5px 10px; }
    #messages li:nth-child(odd) { background: #eee; }
    #form { background: #fff; padding: 3px; position: fixed; bottom: 0; width: 100%; }
    #input { border: 1px solid #ddd; padding: 10px; width: 80%; margin-right: .5%; }
    #form button { width: 18%; background: #4CAF50; color: white; padding: 10px; border: none; }
    .username { font-weight: bold; color: #2196F3; margin-right: 8px; }
  </style>
</head>
<body>
  <ul id="messages"></ul>
  <form id="form" action="">
    <input id="input" autocomplete="off" placeholder="メッセージを入力..."/><button>送信</button>
  </form>

  <script src="/socket.io/socket.io.js"></script>
  <script>
    let username = '';
    while (!username.trim()) {
      username = prompt('ユーザー名を入力してください:');
    }

    const socket = io({
      transports: ['websocket', 'polling'],
      reconnectionAttempts: 5,
      reconnectionDelay: 1000
    });
    const form = document.getElementById('form');
    const input = document.getElementById('input');
    const messages = document.getElementById('messages');

    form.addEventListener('submit', (e) => {
      e.preventDefault();
      if (input.value) {
        socket.emit('chat message', {
          username: username,
          message: input.value
        });
        input.value = '';
      }
    });

    socket.on('chat message', (data) => {
      const li = document.createElement('li');
      const usernameSpan = document.createElement('span');
      usernameSpan.className = 'username';
      usernameSpan.textContent = data.username;
      
      li.appendChild(usernameSpan);
      li.appendChild(document.createTextNode(data.message));
      messages.appendChild(li);
      window.scrollTo(0, document.body.scrollHeight);
    });
  </script>
</body>
</html>
```

1. codespace 内のターミナルで、以下のコマンドを実行し、テストを行います。(テスト項目がクリアしたことがわかります。)
   テストを確認後、「q」を押下することで、再度プロンプトを表示することができます。

```
npm test
```

---

1. codespace 内のターミナルで、以下のコマンドを実行し、変更したコードで改めてコンテナイメージをビルドします。

```
docker build -t test:2 .
```

1. 以下のコマンドで、コンテナイメージが作成されたことを確認します。

```
docker image ls
```

1. 以下のコマンドで、先ほど作成したコンテナイメージを使用し、コンテナを起動します。

```
docker run -dp 3000:3000 test:2
```

1. 画面右下にポップアップが表示されます。「ブラウザーで開く」をクリックします。

1. ポップアップで、「ユーザー名を入力してください」と表示されます。任意のユーザー名を入力してください。

1. 次に、「メッセージを入力」に文字を入力し、「送信」をクリックすることで、先ほど入力したユーザー名と、入力したメッセージが表示されます。

1. 以下のコマンドで、先ほど起動したコンテナを削除します。

```
docker rm -f $(docker ps -qa)
```

1. ここで、変更作業を commit して、push します。コミットメッセージを入力して、「コミット」をクリックします。

1. 「変更の同期」をクリックします。

---

1. codespace から抜けて、「devflow-training」という Repository に戻ります。

1. 「feature/show-username」ブランチ上で作業します。

1. 「Actions」タブをクリックします。

1. 先ほど コミット したことで実行されたパイプラインのステータスを確認します。(パイプラインが失敗していないことを確認します。)

1. 「feature/show-username」ブランチ上で、pull request を作成します。

1. 作成した pull request の Reviewers に共同作業者を追加します。(追加された共同作業者がレビューを行います)

---

[//]: # (TODO: 複数メンバー前提にしたいので、ここで開発者②がコメントを入れて承認するという流れを追加したいです) → ok だと思うのですが、今思うとレビューアってほとんどやることない気が,,,

**<↓レビューア作業ここから↓>**

1. レビューアは、作成された pull request を開きます。

1. 「Add your review」をクリックします。

1. 変更内容を確認したら、「Review changes」をクリックし、「問題なさそうですb」とコメントを記し、「Approve」を選択して、「Submit review」をクリックします。

**<↑レビューア作業ここまで↑>**

---

1. 開発者は、レビューアから Approved されたことを確認します。

1. 「Merge pull request」をクリックします。

1. 「Confirm merge」をクリックします。

1. 「Delete branch」をクリックします。

1. 「Delete codespace」をクリックします。

1. 「Actions」タブをクリックします。

1. 先ほど pull request をマージしたことで実行されたパイプラインのステータスを確認します。(パイプラインが失敗していないことを確認します。)

---

## 2.セキュリティスキャンの追加とCVEの対応

1. 「devflow-training」という Repository で作業します。

1. 画面の左上タブの「<> Code」が選択されていることを確認してください。

1. 「main」と表示されている部分をクリックします。

1. 「Find or create a branch」に `feature/introduce-trivy-pipeline` と入力します。

1. 「Create branch *feature/introduce-trivy-pipeline* from *main*」をクリックします。

1. 画面右側、緑色で表示されている「<> Code」をクリックし、「Create codespace on feature/introduce-t...」をクリックします。

---

1. 「.github/workflows/ci-cd.yml」ファイルを編集します。(以下のファイルをコピー&ペーストし、差分を確認します)

```
name: CI/CD Pipeline

on:
  push:
    branches: [ 'feature/**' ]
  pull_request:
    types: [ closed ]
    branches: [ 'main' ]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}
  IMAGE_TAG: run-${{ github.run_number }}

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Run tests
        run: npm test

  docker-build-push:
    needs: build-and-test
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    outputs:
      image-digest: ${{ steps.build.outputs.digest }}
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Log in to GitHub Container Registry
        uses: docker/login-action@v2
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
          
      - name: Extract metadata for Docker
        id: meta
        uses: docker/metadata-action@v4
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=raw,value=${{ env.IMAGE_TAG }}
          
      - name: Build and push Docker image
        id: build
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}

  security-scan:
    needs: docker-build-push
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ env.IMAGE_TAG }}
          format: 'table'
          exit-code: '1'
          ignore-unfixed: true
          vuln-type: 'os,library'
          severity: 'CRITICAL,HIGH'
          ignorefile: .trivyignore
```

1. ここで、変更作業を commit して、push します。コミットメッセージを入力して、「コミット」をクリックします。

1. 「変更の同期」をクリックします。

---

1. codespace から抜けて、「devflow-training」という Repository に戻ります。

1. 「feature/introduce-trivy-pipeline」ブランチ上で作業します。

1. 「Actions」タブをクリックします。

1. 先ほど コミット したことで実行されたパイプラインのステータスを確認します。(追加したセキュリティスキャンが失敗していることを確認します。)

---

1. codespaceに戻ります。

1. まず、「.github/workflows/ci-cd.yml」ファイルを編集します。( `node-version: '18'` を `node-version: '23'` に変更してください。)

1. 次に、「Dockerfile」ファイルを編集します。( `FROM node:18-slim` を `FROM node:23-slim` に変更してください。)

1. 変更作業を commit して、push します。コミットメッセージを入力して、「コミット」をクリックします。

1. 「変更の同期」をクリックします。

---

1. codespace から抜けて、「devflow-training」という Repository に戻ります。

1. 「feature/introduce-trivy-pipeline」ブランチ上で作業します。

1. 「Actions」タブをクリックします。

1. 先ほど コミット したことで実行されたパイプラインのステータスを確認します。(パイプラインが失敗していないことを確認します。)

1. 「feature/introduce-trivy-pipeline」ブランチ上で、pull request を作成します。

1. 作成した pull request の Reviewers に共同作業者を追加します。(追加された共同作業者がレビューを行います)
---

**<↓レビューア作業ここから↓>**

1. レビューアは、作成された pull request を開きます。

1. 「Add your review」をクリックします。

1. 変更内容を確認したら、「Review changes」をクリックし、「問題なさそうですb」とコメントを記し、「Approve」を選択して、「Submit review」をクリックします。

**<↑レビューア作業ここまで↑>**

---

1. 開発者は、レビューアから Approved されたことを確認します。

1. 「Merge pull request」をクリックします。

1. 「Confirm merge」をクリックします。

1. 「Delete branch」をクリックします。

1. 「Delete codespace」をクリックします。

1. 「Actions」タブをクリックします。

1. 先ほど pull request をマージしたことで実行されたパイプラインのステータスを確認します。(パイプラインが失敗していないことを確認します。)

