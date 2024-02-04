---
title: "GitLabのCI/CD componentsでRenovateを導入する"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [ "GitLab", "Renovate" ]
published: false
---

GitLabにCI/CDコンポーネントという機能が導入された。
GitHubで言えばGitHub Actionsのカスタムアクションのようなものである。

https://docs.gitlab.com/ee/ci/components/index.html

今までもCI/CD templatesというyamlの設定ファイルをインクルードする機能を応用する形で
似たような機能があったがより洗練された独立した機能として導入された。

Renovateはソースコードが依存しているパッケージにバージョンアップがあったときに
アップデートするためのMerge Requestを自動作成してくれるソフトである
(同様なソフトでDependabotなどもある)。

https://docs.renovatebot.com/

今までもCI/CD templateを使ってRenovateは使っていたがあらためて
CI/CD componentsを使って導入したので導入方法を説明する。

CI/CD catalogというページが用意されている。
GitHubで言うところのMarketplaceのようなものである。
まだ立ち上がったばかりなので2024-02-04ではまだ98個しかない。

https://gitlab.com/explore/catalog

検索でrenovateを探すと複数みつかるのだが次を使うことにした。

https://gitlab.com/explore/catalog/to-be-continuous/renovate

まず事前にトークンを用意しておく。
ドキュメントにもあるようにGitLabとGitHubのトークンが必要になる。
GitLabはプロジェクトのトークンを作成する。
説明にあるように api・write_repository・read_read_regsitryで
Developerのトークンを作成する。
なお個人トークンでも動くのだがautodiscoverで動かすと権限のあるレポジトリ全部に
対して動作するのでわかってやるのならいいが範囲が広くなるので注意。

GitHubは
fine-grained personal access tokenで
read-onlyのトークンで良い。

以下の説明にあるようにして `.gitlab-ci.yml` に設定を追加する。

https://gitlab.com/explore/catalog/to-be-continuous/renovate#use-as-a-cicd-component

```.gitlab-ci.yaml
include:
  - component: gitlab.com/to-be-continuous/renovate/gitlab-ci-renovate@1.2.0

variables:
  RENOVATE_PLATFORM: "gitlab"
  RENOVATE_EXTRA_FLAGS: "--autodiscover"
```

現時点ではRenovateに渡すオプションを明示的に指定しないと動かない。

`.gitlab/renovate.json`を用意する。中身は以下のようで実質的に空でも良い。

```.gitlab/renovate.json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json"
}
```

最後にパイプラインスケジュールを作成してRENOVATE_TOKElNとGITHUB_COM_TOKENを設定する。
パイプラインスケジュールを手動で動かしてrenovate-depcheckというジョブが動くのを確認する。

以下のような出力がされるはずである。

```shell
$ renovate $RENOVATE_EXTRA_FLAGS
 WARN: env config dryRun property has been changed to null
 INFO: Autodiscovered repositories
       "length": 1,
       "repositories": ["takatan/warikanjs"]
 INFO: Repository started (repository=takatan/warikanjs)
       "renovateVersion": "37.170.0"
 INFO: Dependency extraction complete (repository=takatan/warikanjs, baseBranch=master)
```

実際導入した感想だが，設定ファイルの書き方は簡易になっていてわかりやすくなった。
Renovateのコンポーネントが変数の設定の説明が少なくてちょっと苦労した。
