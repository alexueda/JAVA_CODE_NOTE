# GSONとは

GSONは、Googleが提供するJSONデータとJavaオブジェクトを相互に変換するためのライブラリです。もっと簡単に表現すると、Javaオブジェクト（Javaで作った情報）をJSON形式（特定の文字表現）に変換してくれます。

Gsonを使う場合に重要なのは、`Gson`クラスの`toJson()`メソッドと`fromJson()`メソッドの2つだけです。もちろん、GSONを使用する際には`import com.google.gson.Gson;`を忘れずに。

## `toJson()`

`toJson()`はJavaオブジェクトをJSONオブジェクトに変換するためのメソッドです。

### Example Code
'''Java
import com.google.gson.Gson;

public class GsonExample {
    public static void main(String[] args) {
        Gson gson = new Gson();
        // サンプルオブジェクトを作成
        Person person = new Person("John", 25);
        // オブジェクトをJSONに変換
        String json = gson.toJson(person);
        System.out.println(json);  // 出力: {"name":"John","age":25}
    }
}

class Person {
    private String name;
    private int age;
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
'''

##`fromJson():`
`fromJson()`はこの逆にJSONからJavaへ変換するためのメソッド

### Example Code
'''Java
import com.google.gson.Gson;

public class GsonExample {
    public static void main(String[] args) {
        Gson gson = new Gson();
        // JSON文字列
        String json = "{\"name\":\"John\",\"age\":25}";
        // JSONをオブジェクトに変換
        Person person = gson.fromJson(json, Person.class);
        System.out.println(person.name + ", " + person.age);  // 出力: John, 25
    }
}

class Person {
    String name;
    int age;
}
'''
