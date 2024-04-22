---
layout: blog
title: "Windows Operational Readinesの仕様を紹介する"
date: 2024-04-03
---

**著者:** Jay Vyas (Tesla), Amim Knabben (Broadcom), and Tatenda Zifudzi (AWS)

>Since Windows support [graduated to stable](/blog/2019/03/25/kubernetes-1-14-release-announcement/)
with Kubernetes 1.14 in 2019, the capability to run Windows workloads has been much
appreciated by the end user community. The level of and availability of Windows workload
support has consistently been a major differentiator for Kubernetes distributions used by
large enterprises. However, with more Windows workloads being migrated to Kubernetes
and new Windows features being continuously released, it became challenging to test
Windows worker nodes in an effective and standardized way.

2019 年にKubernetes 1.14でWindowsサポートが[安定版に移行](/blog/2019/03/25/kubernetes-1-14-release-announcement/)して以来 、Windowsワークロードを実行する機能がエンドユーザーコミュニティから高く評価されてきました。
Windowsワークロードサポートのレベルと可用性は、大企業が使用する Kubernetesディストリビューションにとって常に大きな差別化要因となっています。
しかし、より多くのWindowsワークロードがKubernetesに移行され、新しいWindows機能が継続的にリリースされるにつれ、効果的かつ標準化された方法でWindowsワーカーノードをテストすることが困難になりました。

> The Kubernetes project values the ability to certify conformance without requiring a 
closed-source license for a certified distribution or service that has no intention 
of offering Windows.

Kubernetesプロジェクトは、Windowsを提供する意図のない認定ディストリビューションやサービスのクローズドソースライセンスを必要とすることなく、適合性を認定できる機能を重視しています。

> Some notable examples brought to the attention of SIG Windows were:

SIG Windows の注目を集めたいくつかの注目すべき例は次のとおりです。

- An issue with load balancer source address ranges functionality not operating correctly on
  Windows nodes, detailed in a GitHub issue:
  [kubernetes/kubernetes#120033](https://github.com/kubernetes/kubernetes/issues/120033).
- GitHub Issueの[kubernetes/kubernetes#120033](https://github.com/kubernetes/kubernetes/issues/120033)で説明されたWindows Node上でロードバランサーのソースアドレス範囲機能が正しく動作しない問題。
- Reports of functionality issues with Windows features, such as
  “[GMSA](https://learn.microsoft.com/en-us/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview) not working with containerd,
  discussed in [microsoft/Windows-Containers#44](https://github.com/microsoft/Windows-Containers/issues/44).
- [microsoft/Windows-Containers#44](https://github.com/microsoft/Windows-Containers/issues/44)で議論された、[GMSA](https://learn.microsoft.com/en-us/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)がcontainerd で動作しないといった、Windows機能に関する機能上のIssuの報告。
- Challenges developing networking policy tests that could objectively evaluate
  Container Network Interface (CNI) plugins across different operating system configurations,
  as discussed in [kubernetes/kubernetes#97751](https://github.com/kubernetes/kubernetes/issues/97751).
- [kubernetes/kubernetes#97751](https://github.com/kubernetes/kubernetes/issues/97751)で説明されているような、さまざまなオペレーティングシステム構成にわたるコンテナ・ネットワーク・インターフェイス(CNI) プラグインを客観的に評価できるネットワーク ポリシー テストを開発するという課題。

> SIG Windows therefore recognized the need for a tailored solution to ensure Windows
nodes' operational readiness *before* their deployment into production environments.
Thus, the idea to develop a [Windows Operational Readiness Specification](https://kep.k8s.io/2578)
was born.

したがって、SIG Windows は、Windows Nodeを本番環境に導入する前に運用準備(Operational Readiness)を整えるためのカスタマイズされたソリューションの必要性を認識しました。
結果として、 [Windows Operational Readinessの仕様](https://kep.k8s.io/2578)を開発するというアイデアが生まれました。

## 公式の適合性テストを実行することはできないのでしょうか?

> The Kubernetes project contains a set of [conformance tests](https://www.cncf.io/training/certification/software-conformance/#how), 
which are standardized tests designed to ensure that a Kubernetes cluster meets 
the required Kubernetes specifications.

Kubernetesプロジェクトには、Kubernetesクラスターが必要なKubernetes仕様を満たしていることを確認するために設計された標準化されたテストである[適合テスト](https://www.cncf.io/training/certification/software-conformance/#how)が含まれています。

> However, these tests were originally defined at a time when Linux was the *only* 
operating system compatible with Kubernetes, and thus, they were not easily 
extendable for use with Windows. 

> Given that Windows workloads, despite their 
importance, account for a smaller portion of the Kubernetes community, it was 
important to ensure that the primary conformance suite relied upon by many 
Kubernetes distributions to certify Linux conformance, didn't become encumbered 
with Windows specific features or enhancements such as GMSA or multi-operating 
system kube-proxy behavior.

ただし、これらのテストはもともとLinuxがKubernetesと互換性のある*唯一*のオペレーティング システムだった時代に定義されたものであるため、Windowsで使用できるように簡単に拡張できませんでした。
Windowsワークロードは、その重要性にもかかわらず、Kubernetes コミュニティで占める割合が小さく、Linuxの適合性を証明するために多くのKubernetesディストリビューションが依存している主要な適合性スイートが、
GMSAやマルチオペレーティング システムの kube-proxy 動作などのWindows固有の機能や機能強化で障害にならないことが重要でした。

> Therefore, since there was a specialized need for Windows conformance testing, 
SIG Windows went down the path of offering Windows specific conformance tests 
through the Windows Operational Readiness Specification.

したがって、Windows適合性テストには特殊なニーズがあったため、SIG Windowsは、Windows Operational Readinessの仕様を通じて Windows固有の適合性テストを提供する道を歩みました。

## Kubernetes のエンドツーエンド テスト スイートを実行することはできないでしょうか?

>In the Linux world, tools such as [Sonobuoy](https://sonobuoy.io/) simplify execution of the 
conformance suite, relieving users from needing to be aware of Kubernetes' 
compilation paths or the semantics of [Ginkgo](https://onsi.github.io/ginkgo) tags.

Linuxの世界では、[Sonobuoy](https://sonobuoy.io/)のようなツールが適合性スイートの実行を簡素化し、Kubernetesのコンパイルパスや[Ginkgo](https://onsi.github.io/ginkgo)タグの意味論を意識する必要性からユーザーを解放します。

> Regarding needing to compile the Kubernetes tests, we realized that Windows 
users might similarly find the process of compiling and running the Kubernetes 
e2e suite from scratch similarly undesirable, hence, there was a clear need to 
provide a user-friendly, "push-button" solution that is ready to go. Moreover, 
regarding Ginkgo tags, applying conformance tests to Windows nodes through a set 
of [Ginkgo](https://onsi.github.io/ginkgo/) tags would also be burdensome for 
any user, including Linux enthusiasts or experienced Windows system admins alike.

Kubernetesテストをコンパイルする必要性については、Windowsユーザーも同様に、Kubernetes e2eスイートをゼロからコンパイルして実行するプロセスを好ましくないと感じる可能性があることに気づきました。
そのため、使いやすくてすぐに利用できる"プッシュボタン"ソリューションを提供する必要が明らかにありました。
さらに、Ginkgoのタグに関しても、[Ginkgo](https://onsi.github.io/ginkgo/)タグの集合を通じてWindows Nodeに適合性テストを適用することは、Linux愛好家や経験豊富なWindowsシステム管理者を含め、どのユーザーにとっても負担が大きいでしょう。

> To bridge the gap and give users a straightforward way to confirm their clusters 
support a variety of features, the Kubernetes SIG for Windows found it necessary to 
therefore create the Windows Operational Readiness application. This application 
written in Go, simplifies the process to run the necessary Windows specific tests 
while delivering results in a clear, accessible format.

ギャップを埋め、ユーザーがクラスターがさまざまな機能をサポートしていることを簡単な方法で確認できるようにするために、Windowsに対するKubernetes SIGは、Windows Operational Readinessのアプリケーションを作成する必要性を感じました。
このGoで書かれたアプリケーションは、必要なWindows固有のテストを実行するプロセスを簡略化し、結果を明確でアクセスしやすい形式で提供します。

> This initiative has been a collaborative effort, with contributions from different 
cloud providers and platforms, including Amazon, Microsoft, SUSE, and Broadcom.

この取り組みは、Amazon、Microsoft、SUSE、Broadcomが含まれており、異なるクラウドプロバイダーやプラットフォームからの貢献を含む、協力的な取り組みです。

## Windows Operational Readinessの仕様をより詳しく見る {#specification}

> The Windows Operational Readiness specification specifically targets and executes 
tests found within the Kubernetes repository in a more user-friendly way than 
simply targeting [Ginkgo](https://onsi.github.io/ginkgo/) tags. It introduces a 
structured test suite that is split into sets of core and extended tests, with 
each set of tests containing categories directed at testing a specific area of 
testing, such as networking. Core tests target fundamental and critical 
functionalities that Windows nodes should support as defined by the Kubernetes 
specification. On the other hand, extended tests cover more complex features, 
more aligned with diving deeper into Windows-specific capabilities such as 
integrations with Active Directory. These goal of these tests is to be extensive, 
covering a wide array of Windows-specific capabilities to ensure compatibility 
with a diverse set of workloads and configurations, extending beyond basic 
requirements. Below is the current list of categories.

Windows Operational Readinessの仕様は、[Ginkgo](https://onsi.github.io/ginkgo/)タグを単に対象とするよりもユーザーフレンドリーな方法で、Kubernetesリポジトリ内で見つかるテストを対象して実行します。
これは、主要テストと拡張テストの集合に分割された構造化されたテストスイートを導入し、各テスト集合には、ネットワーキングなどの特定のテスト領域をテストするために向けられたカテゴリが含まれています。
主要テストは、Kubernetes仕様で定義されたWindowsノードがサポートすべき基本的で重要な機能を対象とします。
一方、拡張テストは、Active Directoryとの統合など、Windows固有の機能にさらに深く関連したより複雑な機能をカバーします。
これらのテストの目標は、さまざまなワークロードと構成との互換性を確保するためにWindows固有の機能の広範な範囲を網羅し、基本的な要件を超えて拡張されます。
以下は、現在のカテゴリのリストです。


| Category Name            | Category Description                                                                                                                |
|--------------------------|-------------------------------------------------------------------------------------------------------------------------------------|
| `Core.Network`           | Tests minimal networking functionality (ability to access pod-by-pod IP.)                                                           |
| `Core.Storage`           | Tests minimal storage functionality, (ability to mount a hostPath storage volume.)                                                  |
| `Core.Scheduling`        | Tests minimal scheduling functionality, (ability to schedule a pod with CPU limits.)                                                |
| `Core.Concurrent`        | Tests minimal concurrent functionality, (the ability of a node to handle traffic to multiple pods concurrently.)                    |
| `Extend.HostProcess`     | Tests features related to Windows HostProcess pod functionality.                                                                    |
| `Extend.ActiveDirectory` | Tests features related to Active Directory functionality.                                                                           |
| `Extend.NetworkPolicy`   | Tests features related to Network Policy functionality.                                                                             |
| `Extend.Network`         | Tests advanced networking functionality, (ability to support IPv6)                                                                  |
| `Extend.Worker`          | Tests features related to Windows worker node functionality, (ability for nodes to access TCP and UDP services in the same cluster) |


| カテゴリ名                | カテゴリの説明                                                                                                                |
|--------------------------|-------------------------------------------------------------------------------------------------------------------------------------|
| `Core.Network`           | 最小限のネットワーク機能(PodがPodのIPにアクセスする機能)をテストします。                                                          |
| `Core.Storage`           | 最小限のストレージ機能(hostPathのストレージボリュームをマウントする機能)をテストします。                                                  |
| `Core.Scheduling`        | 最小限のスケジューリング機能(CPU制限のあるPodをスケジュールする機能)をテストします。                                                |
| `Core.Concurrent`        | 最小限の同時機能(同時に複数のPodへのトラフィックを処理するためのノードの機能)をテストします。                  |
| `Extend.HostProcess`     | Windowns HostProcess Pod機能に関連した機能をテストします。                                                                     |
| `Extend.ActiveDirectory` | Active Directory機能に関連した機能をテストします。                                                                           |
| `Extend.NetworkPolicy`   | Network Policy機能に関連した機能をテストします。                                                                              |
| `Extend.Network`         | 高度なネットワーク機能(IPv6をサポートする機能)をテストします。                                                                   |
| `Extend.Worker`          | Windowsワーカーノード機能に関連した機能(同じクラスター内でTCPやUDPサービスにアクセスするためのノードの機能)をテストします。  |

## WindowsノードのOperational Readinessテストを実施するための方法

> To run the Windows Operational Readiness test suite, refer to the test suite's [`README`](https://github.com/kubernetes-sigs/windows-operational-readiness/blob/main/README.md), 
which explains how to set it up and run it. 
The test suite offers flexibility in how you can execute tests, either using a compiled binary or a Sonobuoy plugin. 
You also have the choice to run the tests against the entire test suite or by specifying a list of categories.
Cloud providers have the choice of uploading their conformance results, enhancing transparency and reliability.

Windows Operational Readinesテストスイートを実行するために、セットアップ方法と実行方法が記載されているテストスイートの[`README`](https://github.com/kubernetes-sigs/windows-operational-readiness/blob/main/README.md)を参照してください。
テストスイートは、コンパイルされたバイナリかSonobuoyプラグインのどちらかを使用して、テストの実行方法に柔軟性を提供します。
テストスイート全体に対してテストを実行するか、カテゴリのリストを指定してテストを実行するかを選択することもできます。
クラウドプロバイダーは適合性の結果をアップロードすることを選択でき、透明性と信頼性が向上します。

> Once you have checked out that code, you can run a test. For example, this sample 
command runs the tests from the `Core.Concurrent` category:

コードをチェックアウトしたら、テストを実行できます。
たとえば、このサンプルコマンドは`Core.Concurrent`カテゴリーのテストを実行します。

```shell
./op-readiness --kubeconfig $KUBE_CONFIG --category Core.Concurrent
```

> As a contributor to Kubernetes, if you want to test your changes against a specific pull 
request using the Windows Operational Readiness Specification, use the following bot 
command in the new pull request.

Kubernetes のコントリビューターとして、Windows Operational Readiness仕様を使用して特定のプルリクエストに対する変更をテストしたい場合は、新しいプルリクエストで次のボットコマンドを使用してください。

```shell
/test operational-tests-capz-windows-2019
```

## 将来を見据えて

> We’re looking to improve our curated list of Windows-specific tests by adding 
new tests to the Kubernetes repository and also identifying existing test cases 
that can be targetted. The long term goal for the specification is to continually 
enhance test coverage for Windows worker nodes and improve the robustness of 
Windows support, facilitating a seamless experience across diverse cloud 
environments. We also have plans to integrate the Windows Operational Readiness 
tests into the official Kubernetes conformance suite.

Kubernetesリポジトリに新しいテストを追加することや、対象となる既存のテストケースを特定することで、厳選されたWindows固有のテストリストを改善することを目指しています。
仕様の長期的な目標は、Windowsワーカーノードのテストカバレッジを継続的に向上させ、Windowsサポートの堅牢性を向上させ、多様なクラウド環境でシームレスな経験を促進することです。
また、Windows Operational Readinessテストを公式のKubernetes適合性スイートに統合する計画もあります。

> If you are interested in helping us out, please reach out to us! We welcome help 
in any form, from giving once-off feedback to making a code contribution, 
to having long-term owners to help us drive changes. The Windows Operational 
Readiness specification is owned by the SIG Windows team. You can reach out 
to the team on the [Kubernetes Slack workspace](https://slack.k8s.io/) **#sig-windows** 
channel. You can also explore the [Windows Operational Readiness test suite](https://github.com/kubernetes-sigs/windows-operational-readiness/#readme) 
and make contributions directly to the GitHub repository.

私たちの支援に興味がございましたら、ぜひご連絡ください！
1回限りのフィードバックからコードへの貢献、変更の促進を支援してくれる長期的なオーナーの存在まで、あらゆる形式の支援を歓迎します。
Windows Operational Readiness仕様は、SIG Windowsチームが所有しています。
[Kubernetes Slack ワークスペース](https://slack.k8s.io/) **#sig-windows**チャネルでチームに連絡できます。
[Windows Operational Readinessテストスイート](https://github.com/kubernetes-sigs/windows-operational-readiness/#readme)を探索し 、GitHubリポジトリに直接貢献することもできます。

> Special thanks to Kulwant Singh (AWS), Pramita Gautam Rana (VMWare), Xinqi Li 
(Google) for their help in making notable contributions to the specification. Additionally, 
appreciation goes to James Sturtevant (Microsoft), Mark Rossetti (Microsoft), 
Claudiu Belu (Cloudbase Solutions) and Aravindh Puthiyaparambil 
(Softdrive Technologies Group Inc.) from the SIG Windows team for their guidance and support.

仕様へ多大な貢献をいただいた、Kulwant Singh (AWS)、Pramita Gautam Rana (VMWare)、Xinqi Li (Google)、および Marcio Morales (AWS) に深く感謝します。
さらに、SIG WindowsチームのJames Sturtevant (Microsoft)、Mark Rossetti (Microsoft)、Claudiu Belu (Cloudbase Solutions)、および Aravindh Puthiyaparambil (Softdrive Technologies Group Inc.) の指導とサポートに感謝します。
