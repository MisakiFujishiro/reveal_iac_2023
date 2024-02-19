# Introduction
----
## 参加の理由
<div style="font-size: 0.6em;">
    <p>
    導入前中後で課題が発生した。
    </p>
</div>



----

## テーマと目標設定
計画シート参照

---




# IaCの価値とPJ適用時の課題
----
## DevOpsとIaC
<div style="font-size: 0.8em;">
    <p>
    Userは価値の変化が早いだけでよい。ペインポイントも「変化の速さ」、価値も「素早いリリース」に変更
    </p>
</div>

----

## IaCのメリット
<div style="font-size: 0.8em;">
    <p>
        ポータビリティ：異なる環境に同じリソースをリリースすることができる<br>
        品質担保：同じリソースを環境差分なく同一品質で作成できる<br>
        可視化：ソースコードを確認することで環境の状態を確認できる<br>
        自動化：テストやデプロイなどを自動化することができる
    </p>
</div>

----

## IaCを適用する際の課題
<div style="font-size: 0.8em;">
    <p>
    以下の表をきれいにしておく。<br>
    https://iac-fujishiroms.atlassian.net/browse/SCRUM-21<br>
    </p>
</div>





----
## 取り扱うテーマ
<div style="font-size: 0.8em;">
    <p>
    可視化・自動化から始めるスタータセットと業務とIaCを連携させる部分を対象とする<br>
    IaCと業務背景を紐づける課題管理を利用した開発フロー
    </p>
</div>

************

<div style="font-size: 0.6em;">
    <p>
    ざっくりしたIACコードとgitとPipelineの連携図<br>
    ざっくりしたJIRAとgitlabとkanbanとCFNの連携図
    </p>
</div>


---







# IaCのスターターセット
----
## IaCにおけるCICDとGitOps

<div style="font-size: 0.8em;">
    <p>
        IaCのメリットである、ポータビリティや可視化のメリットを十分に享受するためには、IaCを導入する際にGitOpsと合わせて導入することが望ましい。
    </p>
</div>

*********

<div style="font-size: 0.8em;">
    <p>
        2枚スライドを利用して、課題と改善点が明確になるようにする。（環境の設定なども絵でわかるように）<br>
        IaC単独：リソースを可視化できるが、環境への反映状況などが不明瞭で管理が手間<br>
        CICD/GitOps導入：環境反映の可視化やバージョン管理が簡易になる
    </p>
</div>


----
## GitOpsスターターセット
<div style="font-size: 0.8em;">
    <p>
        使い慣れたgitとAWSのを連携させ、ソースコードからAWSのIaCのサービスであるCloudFormationの実行までを連携させることが望ましい。<br>
        実行の方法は案件によって望ましい形があるが、今回はシンプルなCICD Pipelineを利用したスタータセットとする。
    </p>
</div>

*********

<div style="font-size: 0.8em;">
    <p>
        Roleを含めない、一連のPipelineの図を用意
    </p>
</div>

----
## GitOpsのスターターセット注意点
<div style="font-size: 0.8em;">
    <p>
        課題でもあったが、導入時はステークホルダーに対して心理的障壁を下げることが望ましい。<br>
        特に認可周りなどを整理して合意しないと導入を遠ざけることになるので、Roleの役割などを整理しておく必要がある。
    </p>
</div>

*********

<div style="font-size: 0.8em;">
    <p>
        Roleまで含めた一連のPipelineの図を用意
    </p>
</div>


----
## ソースステージ
<div style="font-size: 0.8em;">
    <p>
        AWSにはCodeCommitというgit機能を提供するサービスがあるが、機能的にはgitlabやgithubの方が優れている<br>
        gitlabやgithubで開発を行い、codecommitへミラーリングすることでAWS側にコードを反映させるフローが推奨<br>
        ポイントは、gitlab/hub側がAWSのCodeCommitへアクセスするための認証認可のフロー
    </p>
</div>

*********

左：codecommitへのアクセス権限だけを持ったUser

右：gitlab/hubからこのユーザーを利用したい。2つの方法があり、gitlabを利用する場合はミラーリング機能、それ以外の場合はSSHによる認証機能を利用


----
## ビルドステージ
<div style="font-size: 0.8em;">
    <p>
        CodeBuildを利用し、デプロイステージで利用するために、ソースステージで取得したコードをS3にPutする。
        ソースコードに対して、環境ごとのパラメータを渡すような設定をする場合はbuildspec.yamlやCodebuildの環境変数を利用。
        また、ビルドのログを残すためにCloudWatchへの認可も設定しておく
    </p>
</div>

*********

左：IAM Roleの概説

右：CodeBuildの役割や環境変数のイメージ

----
## デプロイステージ
<div style="font-size: 0.8em;">
    <p>
        デプロイステージとしてはCloudFormationの実行を行う。<br>
        CFNにはRoleを付与するが、このRoleは実際にリソースを作成・削除するので権限が強くなる点に注意
    </p>
</div>

*********

実際に作成したいリソースに対する権限を持っていなくてはいけないことに注意。
そして、ロールバックに備えて削除権限なども必要になってくる。こちらの運用に対しては後述

----
## CodeDeployが持つ権限に対する対策
- gitのマージで止める
- pipelineの承認プロセスで止める


----
## Codepipelineについて
<div style="font-size: 0.8em;">
    <p>
        上記のソースステージ・ビルドステージ・デプロイステージを一気に実行する<br>
        CodePipelineはソースステージにアクセス、CFNを実行する。
        必要な認可としては、CodeCommitに対する権限とCDNを実行するための権限が必要。<br>
        併せて、CodeCommitの変化をきっかけにPipelineをキックする必要があるのでそのためのEventも作成する
    </p>
</div>

*******

左：IAM Roleの概説

右：CodeBuildの役割や環境変数のイメージ



---







# デモ
----
## ストーリー
<div style="font-size: 0.8em;">
    <p>
        IaCを利用して、dev/prdの2環境にリリースしたい<br>
        JIRAでチケット管理・gitlabでソースコード管理をしている<br>
        リリースしたいのはLambdaとEvent Bridge
    </p>
</div>



----
## CodePipelineのArtifact用のS3作成（初回のみ）
<div style="font-size: 0.8em;">
    <p>
    各環境でCodePipelineの成果物を作成するS3を作成
    </p>
</div>


----
## CodePipelineのRole作成（初回のみ+認可のコントロールで修正）
<div style="font-size: 0.8em;">
    <p>
    各環境でCodePipelineで利用するRoleを作成<br>
    本来CodeDeployに付与するRoleは細かく制御する必要性がある点に注意<br>
    </p>
</div>

----
## CodePipelineのワンセット作成（紐づけるgitlabPJごとに実施）
<div style="font-size: 0.8em;">
    <p>
    Pipelineのワンセットを作成<br>
    gitlab側でミラーリング設定<br>
    </p>
</div>


----
## gitlabでCFNのソースコードを開発
<div style="font-size: 0.8em;">
    <p>
    gitlabで開発していたfeatureブランチをdevelopmentにマージ<br>
    開発していた、Lambda/EventBridgeがリリース<br>
    Lambdaの内容やEventBridgeの修正をするとその内容がリリース
    </p>
</div>
---








# 課題管理を利用した開発フロー
----
## IaCと業務背景の連携不足による課題感
<div style="font-size: 0.8em;">
    <p>
    業務的な背景と結びつきなく、IaCに対する追加や修正依頼がされ、
    結果として、管理がしにくい単位やカテゴリでIaCが作成される
    </p>
</div>



----
## 業務背景と課題管理
<div style="font-size: 0.8em;">
    <p>
    一般的にJIRAやgitlabの課題管理を利用することが多い<br>
    JIRAとgitlabの対応表。
    </p>
</div>





----
## JIRAとgitlabの連携
<div style="font-size: 0.8em;">
    <p>
    gitは課題管理とブランチの紐付けが充実している。一方でJIRAは課題の紐付けやAgile開発時の機能が充実<br>
    gitとJIRAを連携させることでJIRAのメリットを享受しつつ、課題とブランチを紐づけることが可能。
    </p>
</div>

----
## 連携時に可能な機能
<div style="font-size: 0.8em;">
    <p>
    jiraからのブランチの作成とgitからjiraへの参照<br>
    gitlabのステータス変更に伴った、jiraのステータス変更
    </p>
</div>


# まとめ
----
## 目標達成状況