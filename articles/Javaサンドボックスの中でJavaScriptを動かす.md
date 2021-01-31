Javaのサンドボックスの中でJavaScript(Nashorn)を動かす。やってることはJSR223に対応しているスクリプト言語ならなんでも行けるはずである。


```java
package example.misc;

import javax.script.ScriptEngine;
import javax.script.ScriptEngineManager;
import java.io.FilePermission;
import java.security.AccessControlContext;
import java.security.AccessControlException;
import java.security.AccessController;
import java.security.AllPermission;
import java.security.CodeSource;
import java.security.PermissionCollection;
import java.security.Permissions;
import java.security.Policy;
import java.security.PrivilegedActionException;
import java.security.PrivilegedExceptionAction;
import java.security.ProtectionDomain;
import java.security.cert.Certificate;
import java.util.Arrays;

public class SandboxJSDemo {
    public static void main(final String[] args) {
        // 全許可にしておかないと本体のほうで何もできなくなる
        Policy.setPolicy(new Policy() {
            @Override
            public PermissionCollection getPermissions(
                    final ProtectionDomain domain) {
                final Permissions perms = new Permissions();
                perms.add(new AllPermission());
                return perms;
            }
        });
        // セキュリティマネージャーを有効にする
        System.setSecurityManager(new SecurityManager());

        // Nashornを呼出。JavaScript以外も行ける。
        final ScriptEngineManager engineManager = new ScriptEngineManager();
        final ScriptEngine engine = engineManager.getEngineByName("javascript");
        if (engine == null) {
            throw new RuntimeException("No engine");
        }

        // 空のパーミッションを作成
        final Permissions permissions = new Permissions();
        // /etc/groupの読み込みだけ許可
        permissions.add(new FilePermission("/etc/group", "read"));
        // sandboxの設定
        final ProtectionDomain domain = new ProtectionDomain(
                new CodeSource(null, (Certificate[]) null), permissions);
        final AccessControlContext ctx = new AccessControlContext(
                new ProtectionDomain[]{domain});

        // 色々動かす
        for (final String s : Arrays
                .asList("print('Hello JavaScript')", "load(\"hoge.js\")",
                        "load(\"http://example.org/\")",
                        "java.nio.file.Files.readAllLines(java.nio.file.Paths.get(\"/etc/passwd\"))",
                        "print(java.nio.file.Files.readAllLines(java.nio.file.Paths.get(\"/etc/group\"))[0])")) {
            try {
                AccessController.doPrivileged(
                        (PrivilegedExceptionAction<Object>) () -> engine
                                .eval(s), ctx);
            } catch (AccessControlException | PrivilegedActionException e) {
                System.out.println(e.getMessage());
            }

        }

    }
}
```

実行すると次のようになる。

```
Hello JavaScript
access denied ("java.io.FilePermission" "hoge.js" "read")
access denied ("java.net.SocketPermission" "example.org:80" "connect,resolve")
access denied ("java.io.FilePermission" "/etc/passwd" "read")
root:x:0:
```
`/etc/group`だけはアクセスが許可されているので読み取りができる。他のファイルの読み書きやネットワークアクセスは許可がないので拒否されて例外が発生する。

# 参考
* http://d.hatena.ne.jp/seraphy/20131220
