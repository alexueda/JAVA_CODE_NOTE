##Java, clone()について
クローンを使用する際にはまずクラス定義にclonableインターフェイスを実装することをimplementsを使用して明らかにしなければなりません。
implementsで定義のちに@overrideを使用してclone()メソッドを実装します。
```java:example.jav
public class ChessGame implements Cloneable {

    private ChessBoard board;
    private TeamColor currentTurn;
    @overrideまでスキップ

    @Override
    public ChessGame clone() {
        try {
            ChessGame cloned = (ChessGame) super.clone();
            cloned.board = board.clone();  // Ensure the board is also cloned
            return cloned;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError("Cloning not supported");
        }
    }
```
そしてクローンを作成する際は
```java:example2.jav
ChessGame clonedGame = this.clone();
```
このようにクローンを作成する。

なおclone()には２通りのメソッドが存在する。
- シャローコピィ (Shallow Copy)
- ディープコピィ (Deep Copy)
