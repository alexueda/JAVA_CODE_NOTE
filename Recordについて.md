
# Java レコード (Records)

オブジェクト指向のコードを書く際には、データフィールドのコレクションを表すだけのクラスを持つことが非常に一般的です。これらはデータオブジェクトと呼ばれます。データオブジェクトは、他のオブジェクトがそれらに対して操作を行うための入力または出力としてのみ存在します。例えば、ID、名前、種類のフィールドを持つ `Pet` オブジェクトがあるとします。ペットオブジェクトは `Walker` オブジェクトに渡され、運動を与えます。別のオブジェクトである `PetHealth` は、運動の効果を追跡するためにペットに関連付けられます。このタイプのカプセル化は、単一の責任のみを持つ凝集性のあるオブジェクトを作成するのに役立ちます。データオブジェクトのもう1つの望ましい属性は、通常不変（immutable）であることです。これは、作成された後に変更されないことを意味します。不変オブジェクトは、複数のコンテキストで同時に使用しても安全であるため、望ましいものです。戻ってきたときに変化していることを心配する必要がありません。

以下のペットを表すデータオブジェクトクラスを考えてみましょう。この例では、ペットのプロパティをコンストラクタで設定し、すべてのフィールドに `final` キーワードを使用して不変にします。また、すべてのフィールドに対するゲッターも提供し、`equals`、`hashCode`、および `toString` をオーバーライドしています。

```java
class PetClass {
    private final int id;
    private final String name;
    private final String type;

    PetClass(int id, String name, String type) {

        this.id = id;
        this.name = name;
        this.type = type;
    }

    public int getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public String getType() {
        return type;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        PetClass petClass = (PetClass) o;

        if (id != petClass.id) return false;
        if (!Objects.equals(name, petClass.name)) return false;
        return Objects.equals(type, petClass.type);
    }

    @Override
    public int hashCode() {
        int result = id;
        result = 31 * result + (name != null ? name.hashCode() : 0);
        result = 31 * result + (type != null ? type.hashCode() : 0);
        return result;
    }

    @Override
    public String toString() {
        return "PetClass[" +
                "id=" + id +
                ", name=" + name +
                ", type=" + type +
                ']';
    }
}
```

これですべての機能を持つ不変データオブジェクトが完成します。しかし、これはたった3つのフィールドを表すために52行のボイラープレートコードが必要です。データオブジェクトが非常に一般的であるため、Java では構文を簡素化するために `record` キーワードが導入されました。

必要なのは、レコードを宣言し、含めたいフィールドを指定することだけです。以下は、ペットクラスのレコードの代替で、機能的には同等です。

```java
record PetRecord(int id, String name, String type) {}
```

Javaレコードを使用すると、次のすべての利点が得られます。

- 不変性：すべてのフィールドは `final` です。
- コンストラクタ構文の簡素化。
- 自動的に生成されるゲッター。
- すべてのフィールドを比較する自動的に生成される `equals` メソッド。
- すべてのフィールドに基づいたハッシュコードが自動生成されます。
- すべてのフィールドを表す `toString` が自動生成されます。

レコードにもメソッドを追加することができます。これは、ペットの名前を変更する方法を提供する場合などに便利です。レコードが不変であるため、このメソッドは `name` フィールドを変更することができず、新しい名前で新しいペットを作成します。

```java
public record PetRecord(int id, String name, String type) {
    PetRecord rename(String newName) {
        return new PetRecord(id, newName, type);
    }
}
```
