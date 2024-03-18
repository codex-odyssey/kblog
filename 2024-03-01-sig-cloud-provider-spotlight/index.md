---
layout: blog
title: "SIG Cloud Providerスポットライト"
slug: sig-cloud-provider-spotlight-2024
date: 2024-03-01
---

**筆者**: Arujjwal Negi

> One of the most popular ways developers use Kubernetes-related services is via cloud providers, but have you ever wondered how cloud providers can do that? How does this whole process of integration of Kubernetes to various cloud providers happen? To answer that, let's put the spotlight on [SIG Cloud Provider](https://github.com/kubernetes/community/blob/master/sig-cloud-provider/README.md).

開発者がKubernetes関連サービスを使用する最も一般的な方法の1つは、クラウド・プロバイダーを経由することですが、クラウド・プロバイダーがどのようにそれを行うのか疑問に思ったことはないだろうか？Kubernetesと様々なクラウド・プロバイダーとの統合は、どのように行われるのだろうか？それに答えるために、[SIG Cloud Provider](https://github.com/kubernetes/community/blob/master/sig-cloud-provider/README.md)に焦点を当ててみましょう。

> SIG Cloud Provider works to create seamless integrations between Kubernetes and various cloud providers. Their mission? Keeping the Kubernetes ecosystem fair and open for all. By setting clear standards and requirements, they ensure every cloud provider plays nicely with Kubernetes. It is their responsibility to configure cluster components to enable cloud provider integrations.

SIGクラウド・プロバイダーは、Kubernetesと様々なクラウド・プロバイダー間の円滑な統合のために活動しています。彼らの使命とは？Kubernetesのエコシステムを誰にとっても公平かつオープンなものに保つことです。明確な標準と要件を設定することで、すべてのクラウド・プロバイダーがKubernetesとうまく連携できるようにします。クラウド・プロバイダーとの統合を可能にするためのクラスター・コンポーネントを構成することは、彼らの責任です。

> In this blog of the SIG Spotlight series, [Arujjwal Negi](https://twitter.com/arujjval) interviews [Michael McCune](https://github.com/elmiko) (Red Hat), also known as _elmiko_, co-chair of SIG Cloud Provider, to give us an insight into the workings of this group.

SIGスポットライトシリーズのこのブログでは、[Arujjwal Negi](https://twitter.com/arujjval)が_elmiko_としても知られるSIGクラウド・プロバイダーの共同議長である[Michael McCune](https://github.com/elmiko) (Red Hat)にインタビューをし、我々にこのグループの活動についての洞察を与えてくれます。

## はじめに

> **Arujjwal**: Let's start by getting to know you. Can you give us a small intro about yourself and how you got into Kubernetes?

**Arujjwal**: まずは、あなたのことを知ることから始めましょう。あなた自身についてと、Kubernetesを始めたきっかけについて簡単に紹介してもらえますか？

> **Michael**: Hi, I’m Michael McCune, most people around the community call me by my handle, _elmiko_. I’ve been a software developer for a long time now (Windows 3.1 was popular when I started!), and I’ve been involved with open-source software for most of my career. I first got involved with Kubernetes as a developer of machine learning and data science applications; the team I was on at the time was creating tutorials and examples to demonstrate the use of technologies like Apache Spark on Kubernetes. That said, I’ve been interested in distributed systems for many years and when an opportunity arose to join a team working directly on Kubernetes, I jumped at it!

**Michael**: こんにちは、私はMichael McCuneです。コミュニティ周りのほとんど人は、私のことをハンドルネームである_elmiko_と呼びます。ソフトウェア開発者としてのキャリアは長く（私が始めた頃は、Windows 3.1が有名でした！）、キャリアのほとんどはオープンソースソフトウェアに関わってきました。私が最初にKubernetesに関わったのは、機械学習とデータサイエンスアプリケーションの開発者としてでした。当時所属していたチームでは、Kubernetes上でApache Sparkのような技術を使用する方法を示すためのチュートリアルや例を作成していました。とはいえ、分散システムには長年興味があったため、Kubernetesに直接取り組むチームに参加する機会が訪れた時は、飛びつきました！

## 機能と活動

> **Arujjwal**: Can you give us an insight into what SIG Cloud Provider does and how it functions?

**Arujjwal:** SIGクラウド・プロバイダーが何をするのか、どのように機能するのか教えていただけますか？

> **Michael**: SIG Cloud Provider was formed to help ensure that Kubernetes provides a neutral integration point for all infrastructure providers. Our largest task to date has been the extraction and migration of in-tree cloud controllers to out-of-tree components. The SIG meets regularly to discuss progress and upcoming tasks and also to answer questions and bugs that arise. Additionally, we act as a coordination point for cloud provider subprojects such as the cloud provider framework, specific cloud controller implementations, and the [Konnectivity proxy project](https://kubernetes.io/docs/tasks/extend-kubernetes/setup-konnectivity/).

**Michael:** SIGクラウド・プロバイダーは、Kubernetesがすべてのインフラストラクチャー・プロバイダーに中立的な統合ポイントを提供できるようにするために設立されました。私たちのこれまでの最大の仕事は、ツリー内のクラウド・コントローラーを抽出し、ツリー外のクラウド・コントローラー・コンポーネントへの移行です。SIGは定期的に会合を開き、進捗状況や今後のタスクについて議論するとともに、発生した質問やバグに回答しています。加えて、クラウド・プロバイダー・フレームワークやクラウド・コントローラーの実装や[Konnectivity proxy project](https://kubernetes.io/docs/tasks/extend-kubernetes/setup-konnectivity/)のようなクラウド・プロバイダーのサブプロジェクト調整ポイントとして活動しています。

> **Arujjwal:** After going through the project [README](https://github.com/kubernetes/community/blob/master/sig-cloud-provider/README.md), I learned that SIG Cloud Provider works with the integration of Kubernetes with cloud providers. How does this whole process go?

**Arujjwal:** プロジェクトの[README](https://github.com/kubernetes/community/blob/master/sig-cloud-provider/README.md)を見て、SIGクラウド・プロバイダーがKubernetesとクラウド・プロバイダーの連携に取り組んでいることを知りました。このプロセス全体はどのように進むのですか？

> **Michael:** One of the most common ways to run Kubernetes is by deploying it to a cloud environment (AWS, Azure, GCP, etc). Frequently, the cloud infrastructures have features that enhance the performance of Kubernetes, for example, by providing elastic load balancing for Service objects. To ensure that cloud-specific services can be consistently consumed by Kubernetes, the Kubernetes community has created cloud controllers to address these integration points. Cloud providers can create their own controllers either by using the framework maintained by the SIG or by following the API guides defined in the Kubernetes code and documentation. One thing I would like to point out is that SIG Cloud Provider does not deal with the lifecycle of nodes in a Kubernetes cluster; for those types of topics, SIG Cluster Lifecycle and the Cluster API project are more appropriate venues.

**Michael:** Kubernetesを実行する最も一般的な方法の1つは、クラウド環境(AWS, Azure, GCPなど)にデプロイすることです。クラウド・インフラストラクチャには、Kubernetesのパフォーマンスを向上させる機能が備わっていることが多く、例えばServiceオブジェクトに対して弾力的な負荷分散を行うことができます。クラウド固有のサービスをKubernetesで一貫して消費できるようにするため、Kubernetesコミュニティはこれらの連携ポイントに対応するクラウド・コントローラーを作成しました。クラウド・プロバイダーは、SIGによって維持されているフレームワークを使用するか、Kubernetesのコードとドキュメントで定義されているAPIガイドに従って、独自のコントローラーを作成することができます。1つ指摘しておきたいのは、SIGクラウド・プロバイダーはKubernetesクラスター内のノードのライフサイクルを扱っていないということです。この種のトピックについては、SIGクラスター・ライフサイクルやCluster APIプロジェクトの方が適切な場です。

## 重要なサブプロジェクト

> **Arujjwal:** There are a lot of subprojects within this SIG. Can you highlight some of the most important ones and what job they do?

**Arujjwal:** このSIGには多くのサブプロジェクトがあります。最も重要なものをいくつか挙げてもらえますか？

> **Michael:** I think the two most important subprojects today are the [cloud provider framework](https://github.com/kubernetes/community/blob/master/sig-cloud-provider/README.md#kubernetes-cloud-provider) and the [extraction/migration project](https://github.com/kubernetes/community/blob/master/sig-cloud-provider/README.md#cloud-provider-extraction-migration). The cloud provider framework is a common library to help infrastructure integrators build a cloud controller for their infrastructure. This project is most frequently the starting point for new people coming to the SIG. The extraction and migration project is the other big subproject and a large part of why the framework exists. A little history might help explain further: for a long time, Kubernetes needed some integration with the underlying infrastructure, not necessarily to add features but to be aware of cloud events like instance termination. The cloud provider integrations were built into the Kubernetes code tree, and thus the term "in-tree" was created (check out this [article on the topic](https://kaslin.rocks/out-of-tree/) for more info). The activity of maintaining provider-specific code in the main Kubernetes source tree was considered undesirable by the ommunity. The community’s decision inspired the creation of the extraction and migration project to remove the "in-tree" cloud controllers in favor of "out-of-tree" components.

私が考える今日、最も重要な2つのサブプロジェクトは、[クラウド・プロバイダー・フレームワーク](https://github.com/kubernetes/community/blob/master/sig-cloud-provider/README.md#kubernetes-cloud-provider)と[移行／抽出プロジェクト](https://github.com/kubernetes/community/blob/master/sig-cloud-provider/README.md#cloud-provider-extraction-migration)です。クラウド・プロバイダー・フレームワークは、インフラストラクチャ・インテグレーターがインフラストラクチャ用のクラウド・コントローラーを構築するのに役立つ共通ライブラリです。このプロジェクトは、SIGに新しく参加する人たちの出発点となることが多いです。抽出と移行プロジェクトは、もう一つの大きなサブプロジェクトであり、フレームワークが存在する理由の大部分を占めています。長い間、Kubernetesは基礎となるインフラストラクチャとの統合を必要としていました。クラウド・プロバイダーの統合はKubernetesのコードツリーに組み込まれていたため、"ツリー内 "という言葉が生まれました（詳しくはこちらの[トピックに関する記事](https://kaslin.rocks/out-of-tree/)を参照）。プロバイダー固有のコードをメインのKubernetesソースツリーで管理する活動は、コミュニティにとって望ましくないと考えられていました。このコミュニティの決断に触発されて、抽出と移行プロジェクトを立ち上げ、"ツリー内"のクラウド・コントローラーを削除して "ツリー外"のコンポーネントを使用するようにしました。

> **Arujjwal:** What makes [the cloud provider framework] a good place to start? Does it have consistent good beginner work? What kind?

**Arujjwal:** [クラウド・プロバイダー・フレームワーク](https://github.com/kubernetes/community/blob/master/sig-cloud-provider/README.md#kubernetes-cloud-provider)が良いスタート地点である理由は何ですか？初心者のための一貫した良い仕事がありますか？どのようなものですか？

> **Michael:** I feel that the cloud provider framework is a good place to start as it encodes the community’s preferred practices for cloud controller managers and, as such, will give a newcomer a strong understanding of how and what the managers do. Unfortunately, there is not a consistent stream of beginner work on this component; this is due in part to the mature nature of the framework and that of the individual providers as well. For folks who are interested in getting more involved, having some [Go language](https://go.dev/) knowledge is good and also having an understanding of how at least one cloud API (e.g., AWS, Azure, GCP) works is also beneficial. In my personal opinion, being a newcomer to SIG Cloud Provider can be challenging as most of the code around this project deals directly with specific cloud provider interactions. My best advice to people wanting to do more work on cloud providers is to grow your familiarity with one or two cloud APIs, then look for open issues on the controller managers for those clouds, and always communicate with the other contributors as much as possible.

**Michael:** クラウド・プロバイダー・フレームワークは、コミュニティがクラウド・コントローラー・マネージャーに推奨するプラクティスをコード化しているため、初心者がマネージャーがどのように、何をするのかを強く理解するのに良いスタート地点だと感じています。残念ながら、このコンポーネントに関する一貫した初心者の仕事は行われていない。これは、フレームワークの成熟度や個々のプロバイダーの成熟度のせいでもあります。より深く関わることに興味がある人にとっては、[Go言語](https://go.dev/)の知識があることは良いことですし、少なくとも1つのクラウドAPI（AWS、Azure、GCPなど）の仕組みを理解していることも有益です。個人的な意見ですが、SIGクラウド・プロバイダーの初心者は、このプロジェクトのコードのほとんどが、特定のクラウド・プロバイダーとのやりとりを直接扱っているため、難しいかもしれません。クラウド・プロバイダーに関する仕事をもっとしたい人への私の最良のアドバイスは、1つか2つのクラウドAPIに精通し、それらのクラウドのコントローラーマネージャーでオープンな課題を探し、他の貢献者と常にコミュニケーションをとることです。

## 成果

> **Arujjwal:** Can you share about an accomplishment(s) of the SIG that you are proud of?

SIGの成果であなたが誇りに思っているものについて教えてください。

> **Michael:** Since I joined the SIG, more than a year ago, we have made great progress in advancing the extraction and migration subproject. We have moved from an alpha status on the defining [KEP](https://github.com/kubernetes/enhancements/blob/master/keps/README.md) to a beta status and are inching ever closer to removing the old provider code from the Kubernetes source tree. I've been really proud to see the active engagement from our community members and to see the progress we have made towards extraction. I have a feeling that, within the next few releases, we will see the final removal of the in-tree cloud controllers and the completion of the subproject.

私がSIGに参加してから1年以上経ちますが、抽出と移行のサブプロジェクトを進める上で大きな進歩を遂げました。私たちは[KEP](https://github.com/kubernetes/enhancements/blob/master/keps/README.md)を定義するアルファ版からベータ版へと移行し、Kubernetesのソースツリーから古いプロバイダーコードを削除することに少しずつ近づいています。私たちのコミュニティ・メンバーからの積極的な関与と、抽出に向けた進捗を見ることができて、本当に誇りに思っています。今後数回のリリースのうちに、ツリー内クラウド・コントローラーの最終的な削除とサブプロジェクトの完了を見ることができるだろう。

## 新しい貢献者へのアドバイス

> **Arujjwal:** Is there any suggestion or advice for new contributors on how they can start at SIG Cloud Provider?

**Arujjwal:** 新しい貢献者に対して、SIGクラウド・プロバイダーでどのように始めれば良いか、何か提案や助言はありますか？

> **Michael:** This is a tricky question in my opinion. SIG Cloud Provider is focused on the code pieces that integrate between Kubernetes and an underlying infrastructure. It is very common, but not necessary, for members of the SIG to be representing a cloud provider in an official capacity. I recommend that anyone interested in this part of Kubernetes should come to an SIG meeting to see how we operate and also to study the cloud provider framework project. We have some interesting ideas for future work, such as a common testing framework, that will cut across all cloud providers and will be a great opportunity for anyone looking to expand their Kubernetes involvement.

**Michael:** これは難しい質問だと思います。SIGクラウド・プロバイダーは、Kubernetesと基盤となるインフラストラクチャの間を統合するコードに焦点を当てています。SIGのメンバーが公式な立場でクラウド・プロバイダーの代表を務めることは非常に一般的ですが、必須ではありません。Kubernetesのこの部分に興味がある人は、SIGの会議に来て、私たちがどのように活動しているのか、またクラウド・プロバイダー・フレームワーク・プロジェクトについて勉強することをお勧めします。私たちは、すべてのクラウド・プロバイダーを横断するような共通のテストフレームワークなど、今後の作業について興味深いアイデアを持っていますし、Kubernetesへの関わりを広げたいと考えている人にとっては素晴らしい機会になるでしょう。

> **Arujjwal:** Are there any specific skills you're looking for that we should highlight? To give you an example from our own [SIG ContribEx] (https://github.com/kubernetes/community/blob/master/sig-contributor-experience/README.md): if you're an expert in [Hugo](https://gohugo.io/), we can always use some help with k8s.dev!

**Arujjwal:** 求めるスキルで、特筆すべきものはありますか？私たち自身の[SIG ContribEx](https://github.com/kubernetes/community/blob/master/sig-contributor-experience/README.md)の例を挙げると、もしあなたが[Hugo](https://gohugo.io/)のエキスパートなら、私たちはいつでもk8s.devの助けを借りることができます！

> **Michael:** The SIG is currently working through the final phases of our extraction and migration process, but we are looking toward the future and starting to plan what will come next. One of the big topics that the SIG has discussed is testing. Currently, we do not have a generic common set of tests that can be exercised by each cloud provider to confirm the behaviour of their controller manager. If you are an expert in Ginkgo and the Kubetest framework, we could probably use your help in designing and implementing the new tests.

**Michael:** SIGは現在、抽出と移行プロセスの最終段階を進めています。SIGが議論してきた大きなトピックの1つはテストです。現在のところ、各クラウド・プロバイダがコントローラー・マネージャーの動作を確認するために実行できる、一般的な共通テストセットがありません。もしあなたがGinkgoとKubetestフレームワークの専門家であれば、新しいテストの設計と実装にあなたの助けを借りることができるでしょう。

---

> This is where the conversation ends. I hope this gave you some insights about SIG Cloud Provider's aim and working. This is just the tip of the iceberg. To know more and get involved with SIG Cloud Provider, try attending their meetings [here](https://github.com/kubernetes/community/blob/master/sig-cloud-provider/README.md#meetings).

ここで会話は終わります。SIGクラウド・プロバイダーの狙いと仕事について、少しはご理解いただけたでしょうか。これは氷山の一角に過ぎません。SIGクラウド・プロバイダーについてもっと知りたい、関わりたいという方は、会議に参加してみてください[こちら](https://github.com/kubernetes/community/blob/master/sig-cloud-provider/README.md#meetings)。
