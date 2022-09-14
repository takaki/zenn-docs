---
title: "GitLab CIでRustのcoverageを取得する"
emoji: "🙆"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [gitlab, rust]
published: true
---
https://docs.gitlab.com/ee/ci/testing/test_coverage_visualization.html

GitLab CIでのカバレッジの取り方は上記リンク。まだ(2022/09)Rustの記述はないが
要はCobertura XML形式のカバレッジレポートをartifactとして出力すれば良い。

カバレッジレポートを取得するにはgrcovを使う。

https://github.com/mozilla/grcov

以上をまとめると次にようになる。

```.gitlab-ci.yml
test:
  stage: test
  variables:
    RUSTFLAGS: "-Cinstrument-coverage"
    LLVM_PROFILE_FILE: "coverage-%p-%m.profraw"
  script:
    - rustup component add llvm-tools-preview
    - cargo test
    - cargo install grcov
    - grcov . --binary-path ./target/debug/ -s . -t cobertura --branch --ignore-not-existing --ignore "*cargo*" -o coverage.xml
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage.xml
```


## その他参考
https://www.collabora.com/news-and-blog/blog/2021/03/24/rust-integrating-llvm-source-base-code-coverage-with-gitlab/
この記事の頃にはgrcovがcobertura出力がなかったようなのでそのあたりの変換をやっている。
