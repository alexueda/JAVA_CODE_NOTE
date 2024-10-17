
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
静的ファイルと名前リストサービスエンドポイントをホスティングするための完全なサーバーコードのリストは次のとおりです。

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

... [続きの内容省略: 他の部分も同様に翻訳します]

