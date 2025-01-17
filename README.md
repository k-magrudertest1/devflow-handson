# devflow-handson

簡易な手順

・以下のレポジトリをimportする  
URL：https://github.com/k-magrudertest1/chat-app-demo.git

・issueの作成する
　・誰がチャットにコメントしたのか分かりづらいのでユーザ名が表示されるように変更する
　　・テストも変更する

・GitHub Packageにpushされたdockerコンテナイメージを以下の環境でpullする
https://shell.cloud.google.com/

ex. docker pull ghcr.io/k-magrudertest1/dev-flow-handson:run-1

・以下のコマンドでアプリを実行する

ex. docker run -d -p 3001:3000 ghcr.io/k-magrudertest1/dev-flow-handson:run-1

・「ウェブでプレビュー」で3001ポートでアクセスする

・アプリを止める

ex. docker rm -f $(docker ps -qa)

---

・「useradd」というブランチを作成

・Codespacesでファイルを編集する

・まず、「tests/messageHandler.test.js」ファイルを編集する(以下のファイルを丸ごとコピペ⇒差分はCodespaces上か、GitHubで確認)

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

・次に、「tests/server.test.js」ファイルを編集する(以下のファイルを丸ごとコピペ⇒差分はCodespaces上か、GitHubで確認)

```
import { describe, it, expect, beforeAll, afterAll } from 'vitest';
import { createServer } from 'http';
import { Server } from 'socket.io';
import Client from 'socket.io-client';
import { app } from '../server';

describe('Chat Server', () => {
  let io, serverSocket, clientSocket;

  beforeAll((done) => {
    const httpServer = createServer(app);
    io = new Server(httpServer);
    httpServer.listen(() => {
      const port = httpServer.address().port;
      clientSocket = new Client(`http://localhost:${port}`);
      io.on('connection', (socket) => {
        serverSocket = socket;
      });
      clientSocket.on('connect', done);
    });
  });

  afterAll(() => {
    io.close();
    clientSocket.close();
  });

  it('should receive and broadcast chat messages', (done) => {
    const testMessage = {
      username: 'testUser',
      message: 'Hello, World!'
    };

    clientSocket.on('chat message', (data) => {
      expect(data.username).toBe(testMessage.username);
      expect(data.message).toBe(testMessage.message);
      expect(data.timestamp).toBeDefined();
      done();
    });

    clientSocket.emit('chat message', testMessage);
  });

  it('should not broadcast invalid messages', (done) => {
    const invalidMessage = {
      username: '',
      message: ''
    };

    let messageReceived = false;
    clientSocket.on('chat message', () => {
      messageReceived = true;
    });

    clientSocket.emit('chat message', invalidMessage);

    setTimeout(() => {
      expect(messageReceived).toBe(false);
      done();
    }, 100);
  });
});
```

・一度commitする

・pull request作成

・テストは、usernameがあることを前提に書かれているが、まだアプリに反映していないので、パイプラインはテストで失敗する。

・まず、「public/index.html」ファイルを編集する(以下のファイルを丸ごとコピペ⇒差分はCodespaces上か、GitHubで確認)

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

・次に、「src/messageHandler.js」ファイルを編集する(以下のファイルを丸ごとコピペ⇒差分はCodespaces上か、GitHubで確認)

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

・commitする

・テストがクリアしたことを確認し、mergeする(ブランチも消す)

---

・GitHub Packageにpushされた新たなdockerコンテナイメージを以下の環境でpullする
https://shell.cloud.google.com/

ex. docker pull ghcr.io/k-magrudertest1/dev-flow-handson:run-4

・以下のコマンドでアプリを実行する

ex. docker run -d -p 3001:3000 ghcr.io/k-magrudertest1/dev-flow-handson:run-4

・「ウェブでプレビュー」で3001ポートでアクセスする（ユーザ名を入力し、チャットにコメントをするとユーザ名が表示されるようになったことを確認）

・アプリを止める

ex. docker rm -f $(docker ps -qa)

---

・「.github/workflows/ci-cd.yml」を編集する(セキュリティスキャンを追加)

・以下のファイルを丸ごとコピペ⇒差分はCodespaces上か、GitHubで確認

```
name: CI/CD Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

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

・commitする

・github actionsのステータスを確認すると、セキュリティテストで失敗していることがわかる(CVE-2024-21538)

---

・「patch」というブランチを作成

・まず、「.github/workflows/ci-cd.yml」ファイルを編集する(差分はCodespaces上か、GitHubで確認)

node-version: '18' を node-version: '23' に変更する

・次に、「Dockerfile」ファイルを編集する(差分はCodespaces上か、GitHubで確認)

FROM node:18-slim を FROM node:23-slim に変更する

・一度commitする

・pull request作成

・テストがクリアしたことを確認する。


