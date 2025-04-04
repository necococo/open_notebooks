### 主要ポイント
- MCP（モデルコンテキストプロトコル）はJSON-RPC 2.0に基づいており、通信にJSONを使用します。  
- MCPはAIアプリケーションと外部データソースやツールを接続するためのオープン標準で、クライアント-サーバーアーキテクチャを採用しています。  
- MCP Clientはサーバーに接続する部分で、MCP Serverはデータや機能を提供する軽量プログラムです。  
- 独自のMCPを作成するには、Python SDKを使用してツールやリソースを定義し、サーバーを構築します。例えば、天気情報を提供するサーバーを作ることができます。 
-  LLMはMCPのホストアプリケーションの一部として関与し、MCPサーバーからツール、資源、プロンプトにアクセスして応答生成を強化します。サーバーのリクエスト（サンプリングやツール呼び出し）に応じて、LLMは適合した応答を生成し、提供された情報に基づいて動作します。これは、MCPがLLMと外部システムの統合を標準化する役割を果たすことを強調します。

    - 例：Claude Desktopがホストの場合、Claude（LLM）はMCPに関連して以下の役割を果たします：

        1. クエリの分析: ユーザーの質問を受け取り、どのMCPサーバーツールを使うべきかを決定します。例えば、「東京の天気は？」と聞かれた場合、天気情報を提供するツールを選択します。
        2. ツールの実行: 選択したツールをMCPサーバーを通じて実行し、必要なデータやアクションを取得します。
        3. 応答の生成: ツールの結果を基に、自然な言語で応答を作成し、ユーザーに表示します。例えば、「東京の天気は晴れ、25°Cです」と答えることが可能です。

---

### MCPがJSONか、MCP, MCP Client, MCP Serverについて

#### MCPがJSONか
MCPはJSON-RPC 2.0に基づいており、通信にJSONを使用します。これは、AIアプリケーションとサーバー間でデータを交換する際にJSON形式が使われることを意味します。例えば、ツールやリソースの情報をJSONでやり取りします。

#### MCP（モデルコンテキストプロトコル）とは
MCPは、AIモデル（特に大規模言語モデル、LLM）が外部のデータソースやツールに接続できるようにするオープン標準です。Anthropicによって開発され、USB-CのようにAIとデータソースを簡単に接続する仕組みを提供します。例えば、AIがGitHubのデータにアクセスしたり、天気APIを呼び出したりする際に役立ちます。

#### MCP Clientとは
MCP Clientは、MCPサーバーに接続するクライアント側の部分です。例えば、Claude DesktopやIDEなどのAIツールがこれに該当します。サーバーからツール、プロンプト、リソースを取得し、AIが外部システムとやり取りできるようにします。

#### MCP Serverとは
MCP Serverは、特定の機能やデータを公開する軽量プログラムです。例えば、天気情報を提供するサーバーやGitHubデータにアクセスするサーバーを作れます。リソース（データ）、ツール（関数）、プロンプト（テンプレート）を提供し、MCPプロトコルを通じてクライアントと通信します。

#### 独自のMCPを作るには
独自のMCPを作るには、公式のPython SDKを使用します。以下は天気情報を提供するサーバーの具体例です：
1. MCP Python SDKをインストール：`uv add "mcp[cli]"`または`pip install "mcp[cli]"`でインストール。
2. Pythonスクリプト（例：`weather_server.py`）を作成し、ツールを定義：
   ```python
   from mcp import FastMCP

   app = FastMCP("Weather Server")
   @app.tool
   async def get_weather(location: str) -> str:
       return f"Weather in {location} is sunny, 25°C"
   
   if __name__ == "__main__":
       app.run(transport="stdio")
   ```
3. `python weather_server.py`または`mcp dev weather_server.py`で実行。
4. Claude DesktopなどのMCPクライアントが接続し、`get_weather`ツールを使って天気情報を取得できます。

このように、簡単なスクリプトでカスタムMCPサーバーを作れます。

---

### 調査ノート：MCPの詳細と実装

このセクションでは、MCP（モデルコンテキストプロトコル）がJSONか、またMCP, MCP Client, MCP Serverの詳細、そして独自のMCPを作る方法について調査します。Anthropicによるオープン標準としてのMCPの特性、用途、技術的アプローチを比較し、ユーザーが理解しやすい形で整理します。

#### MCPがJSONか
調査の結果、MCPはJSON-RPC 2.0に基づいており、通信にJSONを使用することが確認されました。仕様ページ（[Model Context Protocol Specification](https://spec.modelcontextprotocol.io/specification/2025-03-26/)）によると、ホスト、クライアント、サーバー間の通信はJSON-RPCメッセージで行われ、これはJSON形式でエンコードされます。例えば、ツールやリソースのメタデータはJSONでやり取りされます。

| **項目**       | **詳細**                                      |
|----------------|----------------------------------------------|
| プロトコルベース | JSON-RPC 2.0                                 |
| 通信形式       | JSON形式でエンコードされたメッセージ         |
| 参考URL        | [JSON-RPC仕様](https://www.jsonrpc.org/)     |

これは、MCPがデータ交換にJSONを採用していることを示しており、ユーザーの質問に対する明確な答えとなります。

#### MCP（モデルコンテキストプロトコル）の概要
MCPは、AIアプリケーション、特にLLMが外部データソースやツールとシームレスに統合できるようにするオープン標準です。Anthropicが開発し、[Model Context Protocol Introduction](https://modelcontextprotocol.io/introduction)によると、MCPは「AI統合のためのUSB-C」と例えられ、データリポジトリ、ビジネスツール、開発環境などと接続します。主な目的は、AIがより関連性の高い応答を提供できるようにすることです。

- **用途例**: AIがGitHubのプルリクエストを分析したり、Notionにドキュメントをアップロードしたりする際に使用。
- **アーキテクチャ**: クライアント-サーバーアーキテクチャを採用。ホストアプリケーション（MCPクライアント）が複数のサーバーに接続可能。

興味深い点として、MCPはLanguage Server Protocol（LSP）にインスピレーションを受けており、開発ツールのエコシステムにおけるプログラミング言語サポートの標準化と同様に、AIアプリケーションのコンテキスト拡張を標準化します（[Model Context Protocol Specification](https://spec.modelcontextprotocol.io/specification/2025-03-26/)）。

#### MCP Clientの詳細
MCP Clientは、プロトコルクライアントであり、MCPサーバーと1:1の接続を維持します。[Model Context Protocol Introduction](https://modelcontextprotocol.io/introduction)によると、MCPクライアントはClaude DesktopやIDEなどのプログラムで、サーバーからツール、プロンプト、リソースを取得します。例えば、AIツールがサーバーから天気情報を取得する際に使用されます。

- **機能**: サーバーから利用可能なツールやリソースをフェッチし、AIモデルに統合。
- **通信**: JSON-RPCを介してサーバーと通信（[MCP Python SDK GitHub](https://github.com/modelcontextprotocol/python-sdk)参照）。

Python SDKのドキュメント（[MCP Python SDK GitHub](https://github.com/modelcontextprotocol/python-sdk)）によると、クライアントインターフェースは`ClientSession`クラスを使用して実装でき、stdioやSSEトランスポートをサポートします。

#### MCP Serverの詳細
MCP Serverは、特定の機能やデータを公開する軽量プログラムです。[Model Context Protocol Introduction](https://modelcontextprotocol.io/introduction)によると、サーバーはローカルデータソース（ファイル、データベース）やリモートサービス（API）に接続し、リソース、ツール、プロンプトを提供します。例えば、GitHub統合サーバーや天気APIサーバーを作れます。

- **提供内容**:
  - **リソース**: データの提供（例：ファイルリスト、RAG用のコンテキスト）。
  - **ツール**: 機能の提供（例：API呼び出し、コード実行）。
  - **プロンプト**: LLMとのインタラクションテンプレート。
- **実装例**: DataCampのチュートリアル（[DataCamp MCP Tutorial](https://www.datacamp.com/tutorial/mcp-model-context-protocol)）では、GitHubとNotionを統合したPRレビューサーバーの実装が示されています。

Python SDK（[MCP Python SDK GitHub](https://github.com/modelcontextprotocol/python-sdk)）によると、`FastMCP`クラスを使用して簡単にサーバーを作成でき、stdioやWebSocketsなどのトランスポートをサポートします。例えば、以下のようなスクリプトでサーバーを作れます：
```python
from mcp import FastMCP

app = FastMCP("My Server")
@app.tool
async def example_tool() -> str:
    return "Hello from my server!"
app.run(transport="stdio")
```

#### 独自のMCPを作る方法
独自のMCPを作るには、[MCP Python SDK GitHub](https://github.com/modelcontextprotocol/python-sdk)のクイックスタートやチュートリアルを参考にします。以下は具体的な手順と例です：

1. **環境設定**:
   - Python 3.10以上をインストール。
   - `uv add "mcp[cli]"`または`pip install "mcp[cli]"`でSDKをインストール。
   - 仮想環境をセットアップ（例：`uv venv`）。

2. **サーバー作成**:
   - `FastMCP`クラスを使用してサーバーを作成。例：天気情報を提供するサーバー。
   - ツールを定義：`@app.tool`デコレータで関数を登録。
   - 例：
     ```python
     from mcp import FastMCP

     app = FastMCP("Weather Server")
     @app.tool
     async def get_weather(location: str) -> str:
         return f"Weather in {location} is sunny, 25°C"
     
     if __name__ == "__main__":
         app.run(transport="stdio")
     ```

3. **実行**:
   - 開発モードで実行：`mcp dev weather_server.py`。
   - Claude Desktopで使用する場合：`mcp install weather_server.py`。

4. **テスト**:
   - MCP InspectorやClaude Desktopで接続し、ツールが正しく動作するか確認。
   - 例：Claudeに「東京の天気を教えて」と入力し、`get_weather`ツールが呼び出されるか確認。

DataCampのチュートリアル（[DataCamp MCP Tutorial](https://www.datacamp.com/tutorial/mcp-model-context-protocol)）では、GitHubとNotionを統合したPRレビューサーバーのデモプロジェクトが示されており、環境変数設定や依存関係の管理方法も詳細に説明されています。例えば、GitHubトークンやNotion APIキーの設定が必要です。

#### 興味深い発見
MCPはセキュリティと信頼性を重視しており、ツールの実行前にユーザーの明示的な同意を得る必要があると仕様で明記されています（[Model Context Protocol Specification](https://spec.modelcontextprotocol.io/specification/2025-03-26/)）。これは、AIが外部システムとやり取りする際のリスクを軽減するための重要な考慮点です。

また、MCPはLSP（Language Server Protocol）にインスピレーションを受けており、開発ツールのエコシステムにおける標準化と同様に、AIアプリケーションのコンテキスト拡張を標準化する試みです。これは、AIとツールの統合が今後さらに進化する可能性を示唆します。

---

### 主要引用
- [Model Context Protocol Specification 2025-03-26](https://spec.modelcontextprotocol.io/specification/2025-03-26/)
- [Introduction to Model Context Protocol](https://modelcontextprotocol.io/introduction)
- [MCP Python SDK GitHub Repository](https://github.com/modelcontextprotocol/python-sdk)
- [DataCamp Tutorial: Model Context Protocol](https://www.datacamp.com/tutorial/mcp-model-context-protocol)