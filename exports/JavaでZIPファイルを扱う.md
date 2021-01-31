---
title: JavaでZIPファイルを扱う
tags: Java
author: takaki@github
slide: false
---
JavaでZIPファイルを読み込む。サンプルではファイル名一覧とbyte[]でデータを読み込んだ上でサイズを表示している。

以前の方法はZipInputStreamを使う。

```java
public class ZipDemo {
    public static final String PATH = "/path/to/zipfile.zip";

    public static void main(final String[] args) throws IOException {
        final Path zipfile = Paths.get(PATH);
        try (ZipInputStream zis = new ZipInputStream(
                new ByteArrayInputStream(Files.readAllBytes(zipfile)));) {
            ZipEntry entry;
            while ((entry = zis.getNextEntry()) != null) {
                if (entry.isDirectory()) {
                    continue;
                }
                System.out.printf("%s - %d\n", entry.getName(),
                        IOUtils.toByteArray(zis).length);

            }
        }
    }
}
```

NIO2のFilesystemsを使うとコールバックを使ってわかりやすく書ける。

```java
public class ZipDemo {
    public static final String PATH = "/path/to/zipfile.zip";

    public static void main(final String[] args) throws IOException {
        final Path zipfile = Paths.get(PATH);

        try (FileSystem fs = FileSystems.newFileSystem(zipfile, null)) {
            final Path path = fs.getPath("/");
            Files.walkFileTree(path, new SimpleFileVisitor<Path>() {
                @Override
                public FileVisitResult visitFile(final Path file,
                                                 final BasicFileAttributes attrs) throws IOException {
                    System.out.printf("%s - %d\n", file,
                            Files.readAllBytes(file).length);
                    return FileVisitResult.CONTINUE;
                }
            });
        }
    }
}
```

