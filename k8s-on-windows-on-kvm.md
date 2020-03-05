# k8s on windows on kvm

KVM上にインストールしたWindows 10 pro の上で、k8s (docker の kubernates機能 or minikube )を起動できるのか検証してみました。

結論から言うと、dockerは無理だった。minikubeはできた。  
という結果になりました。


## 検証環境

- Hyper-V を有効にできるプロセッサ

- linux
  - CPU: Intel(R) Xeon(R) CPU E3-1245 V2 @ 3.40GHz
    - 結構古い
    - `Intel VT-x` と `Intel EPT` がギリギリ使えるプロセッサ
  - Memory: 16GB
- windows 10 pro を KVM上にインストール
  - memory: 8GB割り当てた

## WindowsでHyper-Vを有効化

まずは、Hyper-Vを有効にする。

`コントロールパネル` -> `プログラムと機能` -> `Windowsの機能の有効化または無効化`

で、Hyper-V を有効化して、windowsを再起動する。


## docker for windows でk8sを試す

[Install Docker Desktop on Windows \| Docker Documentation](https://docs.docker.com/docker-for-windows/install/#install-docker-desktop-for-windows-desktop-app)
上記より、dockerをダウンロードしてインストールする

インストールが完了したら再起動の儀式をする。

### docker起動

自動起動してくるので起動完了まで待つ。  
Hyper-V上に、docker用のVMが作られる。(非常に時間がかかった)

起動してこない場合は、Docker Desktopを実行する。
起動完了すると、タスクバーに、Dockerが確認できる。

![スクリーンショット 2020-03-05 10.01.24.png](:storage/45c02d73-d2f1-41d6-a344-96a9a795208e/823f0700.png)

### kubernatesを有効化する

タスクバーのDockerアイコンよりsettingを実行。  
kubernatesを有効化する。

いつまでたっても処理が完了せず、ずっと放置していたらタイムアウトしてエラーになった。  
どうやらダメらしい。  

Hyper-Vの上のdocker VMに接続するが、接続しても真っ黒画面のままで何か動いている気配なし。  
Dockerをfactoryリセットして何度か試すが、Dockerすらまともに起動してこない状態になった。  
無理そう（無茶そう）なので諦める。


## minikube を試す

- 参考
  - [Minikubeを使用してローカル環境でKubernetesを動かす - Kubernetes](https://kubernetes.io/ja/docs/setup/learning-environment/minikube/#%e3%82%af%e3%82%a4%e3%83%83%e3%82%af%e3%82%b9%e3%82%bf%e3%83%bc%e3%83%88)

### インストール

以下よりminikube `minikube-windows-amd64.exe` をダウンロードする。
[Releases · kubernetes/minikube · GitHub](https://github.com/kubernetes/minikube/releases)

以下よりkubectlをダウンロードする。  
[kubectlのインストールおよびセットアップ - Kubernetes](https://kubernetes.io/ja/docs/tasks/tools/install-kubectl/#windows%e3%81%b8kubectl%e3%82%92%e3%82%a4%e3%83%b3%e3%82%b9%e3%83%88%e3%83%bc%e3%83%ab%e3%81%99%e3%82%8b)

それぞれのコマンドを任意の場所に配置して、環境変数でPATHを通す。

### minikubeの起動

```
minikube start --vm-driver=hyperv
```

Hyper-Vを有効にしていても、有効にしろとエラーが出たので、forceをつけて起動した。

```
minikube start --vm-driver=hyperv --force
```

Hyper-V 上に、minikubeという名前のVMが自動で作られる。  
ちなみに、このVMには、user=`docker`, password=`tcuser` でログインできる。

![スクリーンショット 2020-03-05 23.28.31.png](:storage/45c02d73-d2f1-41d6-a344-96a9a795208e/979b5136.png)

VM作成されたが、しばらくするとコマンドはエラーになった・・・  
VMが起動してくるのが遅すぎてエラーになった模様？？

```
C:\opt>minikube start --vm-driver=hyperv --hyperv-virtual-switch=kube --force
* minikube v1.7.3 on Microsoft Windows 10 Pro 10.0.18363 Build 18363
* Using the hyperv driver based on user configuration

! 'hyperv' driver reported an issue: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe Get-WindowsOptionalFeature -FeatureName Microsoft-Hyper-V-All -Online failed:

* Suggestion: Start PowerShell as Administrator, and run: 'Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All'
* Documentation: https://minikube.sigs.k8s.io/docs/reference/drivers/hyperv/

* Creating hyperv VM (CPUs=2, Memory=2000MB, Disk=20000MB) ...
*
X Unable to start VM. Please investigate and run 'minikube delete' if possible: creating host: create host timed out in 120.000000 seconds
*
* minikube is exiting due to an error. If the above message is not useful, open an issue:
  - https://github.com/kubernetes/minikube/issues/new/choose

C:\opt>
C:\opt>
```

一度stopしてからもう一度startしたらいけた。希望が見えた。

```
minikube stop
```

```
C:\opt>minikube start --force
* minikube v1.7.3 on Microsoft Windows 10 Pro 10.0.18363 Build 18363
* Using the hyperv driver based on existing profile

! 'hyperv' driver reported an issue: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe Get-WindowsOptionalFeature -FeatureName Microsoft-Hyper-V-All -Online failed:

* Suggestion: Start PowerShell as Administrator, and run: 'Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All'
* Documentation: https://minikube.sigs.k8s.io/docs/reference/drivers/hyperv/

* Reconfiguring existing host ...
* Starting existing hyperv VM for "minikube" ...
* Preparing Kubernetes v1.17.3 on Docker 19.03.6 ...
* Launching Kubernetes ...
* Enabling addons: dashboard, default-storageclass, storage-provisioner
* Done! kubectl is now configured to use "minikube"

C:\opt>
```

```
C:\opt>minikube status
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

dashboardが見れるか確認。

```
minikube dashboard
```
勝手にブラウザが立ち上がり、ダッシュボードが見れた。しかし、描画がすごく遅い。

![スクリーンショット 2020-03-05 23.03.42.png](:storage/45c02d73-d2f1-41d6-a344-96a9a795208e/d275f1a3.png)



### podの起動

```
C:\opt>kubectl create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.10
deployment.apps/hello-minikube created

C:\opt>kubectl expose deployment hello-minikube --type=NodePort --port=8080
service/hello-minikube exposed

C:\opt>kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
hello-minikube-797f975945-2cz52   1/1     Running   0          10h

```

立ち上がったので、接続してみる。  

URLを調べて、ブラウザで接続する。

```
C:\opt>minikube service hello-minikube --url
http://192.168.187.8:31775
```

![スクリーンショット 2020-03-05 23.13.37.png](:storage/45c02d73-d2f1-41d6-a344-96a9a795208e/8842ff65.png)

無事接続できた。


## まとめ

- minikubeなら、k8s on windows on KVM ができた。
- しかし、非常に遅い、重いので実利用には向かない（勉強にも向かない）
  - windows VM の上に、VM でさらにコンテナってそもそも無茶ですが・・・
  - もしかしたらスペシャルマシン上でリッチなwindows VM を立てれば快適なのかもしれない。
