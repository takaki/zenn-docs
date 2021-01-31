NIO2のFileSystemを使ってメモリ上にFileSystemを作るJimfsの紹介。
通常のファイルシステムとは別にファイルシステムを持てる。ただNIO2で全部処理しなくてはならないようで`Path#toFile`なんかを使うと例外を飛ばしてくる。一時ファイルを使ってごちゃごちゃやるような処理に使えば使用後にまとめてゴミ捨てできるので便利かなあとも思ったが使い勝手はどうなんだろう。

```java
package example.misc;

import com.google.common.jimfs.Configuration;
import com.google.common.jimfs.Jimfs;

import java.nio.charset.StandardCharsets;
import java.nio.file.FileSystem;
import java.nio.file.Files;
import java.nio.file.Path;

public class JimFsDemo {

    public static void main(final String[] args) throws Exception {
        try (final FileSystem fs = Jimfs.newFileSystem(Configuration.unix())) {
            final Path tmp = fs.getPath("/tmp");
            Files.createDirectories(tmp);
            final Path tmpFile = Files.createTempFile(tmp, "jimfs", "tmp");
            Files.write(tmpFile, "abcdef".getBytes(StandardCharsets.UTF_8));

            // System.out.println(tmpFile.toFile().getAbsolutePath()); // toFileで例外が出るのでダメ
            System.out.println(tmpFile.getParent().getFileName());
            System.out.println(tmpFile.getFileName());

            final byte[] read = Files.readAllBytes(tmpFile);
            System.out.println(new String(read, StandardCharsets.UTF_8));
        }
    }

}
```

* https://github.com/google/jimfs
