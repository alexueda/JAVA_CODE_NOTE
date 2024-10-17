
# Web API

HTTP が理論的にどのように機能するかを理解したので、Java コードを使用して HTTP クライアントからのリクエストを作成し、HTTP サーバーからの応答を受け取ることができます。

## Web サーバー
サーバーコードには、JavaSpark というライブラリを使用します。JavaSpark は、複数のエンドポイントリクエストを処理する HTTP サーバーを書くのを非常に簡単にしてくれます。エンドポイントとは、特定の HTTP リソースリクエストを処理するコードです。サービスエンドポイントは、サービスインターフェースのパブリックメソッドのようなものと考えることができます。

例として、名前リストという HTTP サービスを書き、名前のリストを管理します。このサービスを有効にするために、以下のエンドポイントを提供します。

| エンドポイント | HTTP メソッド | HTTP パス    | 目的                                        |
| -------------- | ------------- | ------------ | ------------------------------------------- |
| addName        | POST          | /name/:name  | name パス変数で表される名前を追加           |
| listNames      | GET           | /name        | 名前のリストを取得                          |
| deleteName     | DELETE        | /name/:name  | name パス変数で表される名前を削除           |

## エンドポイントの実装
JavaSpark でエンドポイントを定義する際には、HTTP メソッド、パス、そしてマッチング HTTP リクエストが発生したときに呼び出される Functional Interface メソッドの実装を指定します。パス定義には、呼び出し側から提供される値に割り当てられる変数を、`:プレフィックス`を使用して含めることができます。例えば、次のようにエンドポイントを実装して名前を追加できます。

```java
private void run() {
    Spark.post("/name/:name", new Route() {
        public Object handle(Request req, Response res) {
            names.add(req.params(":name"));
            return listNames(req, res);
        }
    });
}

private Object listNames(Request req, Response res) {
    res.type("application/json");
    return new Gson().toJson(Map.of("name", names));
}
```

上記の例では、`Spark.post` メソッドが `HTTP POST` リクエストに対して `/name/:name` パスを処理するために呼び出されています。`Spark.post` メソッドは、HTTP パスと Functional Interface `spark.Route` の匿名クラスの実装の 2 つのパラメーターを受け取ります。インターフェースには `route` というメソッドが 1 つあり、HTTP メソッドとパスが受信 HTTP リクエストと一致するときに呼び出されます。

エンドポイントの戻り値は `listNames` メソッドを呼び出すことによって生成されます。これは、`Content-Type` HTTP ヘッダーを `application/json` に設定し、現在の名前リストを JSON 文字列としてシリアライズして、HTTP レスポンスのレスポンスボディとして返します。

### Lambda を使用したエンドポイントの簡略化
エンドポイントを呼び出す方法を実装するメソッドを呼び出すために Lambda 関数を使用して、ルートハンドラーを簡略化できます。

```java
private void run() {
    Spark.post("/name/:name", (req, res) -> addName(req, res));
}

private Object addName(Request req, Response res) {
    names.add(req.params(":name"));
    return listNames(req, res);
}
```

最終的に、Lambda 関数が別の関数への単なるパススルーであるため、Java のメソッド参照構文で置き換えることができます。

```java
Spark.post("/name/:name", this::addName);
```

### 静的ファイルの配信
HTTP リソースは何でも表現できます。上記の例では、名前リストのインメモリ表現を使用していますが、ファイルのディレクトリ構造を永続的なストレージとして表現することも可能です。JavaSpark は `staticFiles.location` メソッドを使用してファイルをホストするためのディレクトリを簡単に設定できます。

```java
Spark.staticFiles.location("web");
```

これで、サーバーにリクエストを送信すると、URL パスと一致するファイルが見つかった場合、そのファイルの内容が返されます。

## 完全なサーバーの例

静的ファイルと名前リストサービスエンドポイントをホスティングするための完全なサーバーコードのリストです。

```java
import com.google.gson.Gson;
import spark.*;
import java.util.*;

public class ServerExample {
    private ArrayList<String> names = new ArrayList<>();

    public static void main(String[] args) {
        new ServerExample().run();
    }

    private void run() {
        // サーバーがリッスンするポートを指定
        Spark.port(8080);

        // 静的ファイルをホスティングするためのディレクトリを登録
        Spark.externalStaticFileLocation("public");

        // メソッド参照構文を使用して各エンドポイントのハンドラーを登録
        Spark.post("/name/:name", this::addName);
        Spark.get("/name", this::listNames);
        Spark.delete("/name/:name", this::deleteName);
    }

    private Object addName(Request req, Response res) {
        names.add(req.params(":name"));
        return listNames(req, res);
    }

    private Object listNames(Request req, Response res) {
        res.type("application/json");
        return new Gson().toJson(Map.of("name", names));
    }

    private Object deleteName(Request req, Response res) {
        names.remove(req.params(":name"));
        return listNames(req, res);
    }
}
```

### このコードで実験する方法
1. `public` というディレクトリを作成し、その中に `<h1>Hello World</h1>` のテキストを含む `index.html` ファイルを入れます。
2. コードを `public` ディレクトリが含まれているディレクトリで実行します。
3. ブラウザを開き、`localhost:8080` にアクセスします。`index.html` ファイルの内容が表示されるはずです。

### Curl でのコマンド
```bash
curl localhost:8080/name       # => {"name":[]}
curl -X POST localhost:8080/name/cow   # => {"name":["cow"]}
curl -X POST localhost:8080/name/dog   # => {"name":["cow","dog"]}
curl localhost:8080/name       # => {"name":["cow","dog"]}
curl -X DELETE localhost:8080/name/dog # => {"name":["cow"]}
curl localhost:8080/name       # => {"name":["cow"]}
```

### リクエストとレスポンスのシリアライズ
HTTP リクエストでオブジェクトを送信するために JSON が一般的に使用されます。したがって、Gson を使用して HTTP リクエストのボディをオブジェクトに解析し、レスポンスを表す JSON を作成します。以下はエコーエンドポイントを持つサーバーの例です。

```java
public class ServerEchoExample {
    public static void main(String[] args) {
        new ServerEchoExample().run();
    }

    private void run() {
        Spark.port(8080);
        Spark.post("/echo", this::echoBody);
    }

    private Object echoBody(Request req, Response res) {
        var bodyObj = getBody(req, Map.class);

        res.type("application/json");
        return new Gson().toJson(bodyObj);
    }

    private static <T> T getBody(Request request, Class<T> clazz) {
        var body = new Gson().fromJson(request.body(), clazz);
        if (body == null) {
            throw new RuntimeException("missing required body");
        }
        return body;
    }
}
```

`getBody` メソッドは、リクエストボディを指定されたクラスのオブジェクトに解析する汎用メソッドです。このパターンにより、ジェネリクス、Gson、および HTTP ボディを組み合わせてサービスのデータの入出力が容易になります。

### エラーハンドリング
エンドポイントの表現に加えて、Spark はエラーケースを処理するためのメソッドを提供します。例えば、未処理の例外がスローされた場合の `Spark.exception` メソッドや、不明なリクエストが発生した場合の `Spark.notFound` メソッドがあります。以下のコードはその実装例を示しています。

```java
public class ServerErrorsExample {
    public static void main(String[] args) {
        new ServerErrorsExample().run();
    }

    private void run() {
        // サーバーがリッスンするポートを指定
        Spark.port(8080);

        // エンドポイントごとにハンドラーを登録
        Spark.get("/error", this::throwError);

        Spark.exception(Exception.class, this::errorHandler);
        Spark.notFound((req, res) -> {
            var msg = String.format("[%s] %s not found", req.requestMethod(), req.pathInfo());
            return errorHandler(new Exception(msg), req, res);
        });
    }

    private Object throwError(Request req, Response res) {
        throw new RuntimeException("Server on fire");
    }

    public Object errorHandler(Exception e, Request req, Response res) {
        var body = new Gson().toJson(Map.of("message", String.format("Error: %s", e.getMessage()), "success", false));
        res.type("application/json");
        res.status(500);
        res.body(body);
        return body;
    }
}
```

### クライアントの例
標準の JDK `java.net` ライブラリを使用して HTTP リクエストを行います。以下の例では URL をハードコードしてリクエストの基本的な部分を簡略化しています。

```java
public class ClientExample {
    public static void main(String[] args) throws Exception {
        URI uri = new URI("http://localhost:8080/name");
        HttpURLConnection http = (HttpURLConnection) uri.toURL().openConnection();
        http.setRequestMethod("GET");

        http.connect();

        try (InputStream respBody = http.getInputStream()) {
            InputStreamReader inputStreamReader = new InputStreamReader(respBody);
            System.out.println(new Gson().fromJson(inputStreamReader, Map.class));
        }
    }
}
```

他にも、リクエストボディを追加する方法、エラーハンドリングなどの例があり、Gson の型アダプタを使用してオブジェクトのシリアライズ方法を制御することもできます。


## リクエストボディとヘッダーの作成

HTTP 本文やヘッダーを HttpURLConnection クラスで送信するには、まず `http.setDoOutput` を true に設定する必要があります。次に、`addRequestProperty` を使用してヘッダーを設定したり、`getOutputStream` から返されたストリームを使用してボディを送信することができます。

```java
// データの書き出しを指定
http.setDoOutput(true);

// ヘッダーを書き出し
http.addRequestProperty("Content-Type", "application/json");

// ボディを書き出し
var body = Map.of("name", "joe", "type", "cat");
try (var outputStream = http.getOutputStream()) {
    var jsonBody = new Gson().toJson(body);
    outputStream.write(jsonBody.getBytes());
}
```

## 簡単な Curl の実装
Web クライアントの例を拡張し、簡単なバージョンの Curl を実装できます。この例では、コマンドラインパラメータから HTTP メソッド、URL、およびボディを読み取ります。これらの情報を使用して HTTP リクエストを行い、レスポンスを受け取ります。

```java
public class ClientCurlExample {
    public static void main(String[] args) throws Exception {
        if (args.length >= 2) {
            var method = args[0];
            var url = args[1];
            var body = args.length == 3 ? args[2] : "";

            HttpURLConnection http = sendRequest(url, method, body);
            receiveResponse(http);
        } else {
            System.out.println("ClientCurlExample <method> <url> [<body>]");
        }
    }

    private static HttpURLConnection sendRequest(String url, String method, String body) throws URISyntaxException, IOException {
        URI uri = new URI(url);
        HttpURLConnection http = (HttpURLConnection) uri.toURL().openConnection();
        http.setRequestMethod(method);
        writeRequestBody(body, http);
        http.connect();
        System.out.printf("= Request =========
[%s] %s

%s

", method, url, body);
        return http;
    }

    private static void writeRequestBody(String body, HttpURLConnection http) throws IOException {
        if (!body.isEmpty()) {
            http.setDoOutput(true);
            try (var outputStream = http.getOutputStream()) {
                outputStream.write(body.getBytes());
            }
        }
    }

    private static void receiveResponse(HttpURLConnection http) throws IOException {
        var statusCode = http.getResponseCode();
        var statusMessage = http.getResponseMessage();

        Object responseBody = readResponseBody(http);
        System.out.printf("= Response =========
[%d] %s

%s

", statusCode, statusMessage, responseBody);
    }

    private static Object readResponseBody(HttpURLConnection http) throws IOException {
        Object responseBody = "";
        try (InputStream respBody = http.getInputStream()) {
            InputStreamReader inputStreamReader = new InputStreamReader(respBody);
            responseBody = new Gson().fromJson(inputStreamReader, Map.class);
        }
        return responseBody;
    }
}
```

上記の例では、エコーサーバーを起動すると、簡単な Curl クライアントを使用してさまざまな HTTP サービスリクエストを作成できます。

### 実行例
```bash
➜  java -cp ../../lib/gson-2.10.1.jar ClientCurlExample.java POST 'http://localhost:8080/echo' '{"name":"joe", "count":3}'
= Request =========
[POST] http://localhost:8080/echo

{"name":"joe", "count":3}

= Response =========
[200] OK

{name=joe, count=3.0}
```

## 理解するべきこと
- サーバーコードの例（Ticket to Ride）
- メインサーバークラスの作成
- GET および POST リクエスト用の HTTP ハンドラーの作成
- FileHandler を使用してテスト Web ページの実装
- Web クライアントの作成



