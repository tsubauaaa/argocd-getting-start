## 構築方法

### microk8s install
1. `brew install ubuntu/microk8s/microk8s` でmicrok8sのインストーラーをDL
2. `microk8s install` でmicrok8sをインストール
3. `microk8s enable dns && microk8s stop && microk8s start` でmicrok8sを開始する

### argocd install
4. `microk8s kubectl create namespace argocd`でnamespaceを作成する
5. `microk8s kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml` でargocdをインストール
6. [brew install argocd](https://kostis-argo-cd.readthedocs.io/en/refresh-docs/getting_started/install_cli/#install-on-macos-darwin)からargocd cliをインストール
7. `microk8s kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'`でargocd-serverをLoadBalancer typeに変更
8. `microk8s kubectl port-forward svc/argocd-server -n argocd 8080:443` で8080にport forwardする
9. https://localhost:8080 にアクセスしてargocd管理画面を表示する
10. `microk8s kubectl get secret --namespace argocd argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 --decode ; echo` でadminの初期パスワードを取得
11. argocd管理画面にログインしてパスワードを変更
12. `microk8s kubectl config set-context --current --namespace=argocd` でnamespaceをset
13. $HOME/.kube/configを $HOME/.microk8s/configに置き換える

### application create
14. `argocd login localhost:8080` でログインする
15. `argocd app create guestbook --repo https://github.com/tsubauaaa/argocd-getting-start.git --path guestbook --dest-server https://kubernetes.default.svc --dest-namespace default --port-forward-namespace argocd` でアプリを作成する
16. `argocd app get guestbook`で作成したアプリを確認する
17. `argocd app sync guestbook`でアプリを同期する

### convert Deployment to Rollout
18. [Convert Deployment to Rollout](https://argoproj.github.io/argo-rollouts/migrating/#convert-deployment-to-rollout)からアプリのマニフェストを編集してgit pushする
19. `kubectl create namespace argo-rollouts`にてnamespaceを作成する
20. `kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml` にてrolloutsをインストールする
21. [Kubectl Plugin Installation](https://argoproj.github.io/argo-rollouts/installation/#kubectl-plugin-installation)からkubectl pluginをインストールする。コマンドは`brew install argoproj/tap/kubectl-argo-rollouts`
22. `kubectl argo rollouts dashboard --namespace default` でargo rollouts dashboardを起動する
23. argocd管理画面からpruneありのsyncを行う
24. http://localhost:3100/rollouts にアクセスしてargo rollouts管理画面を表示する

### canary release
25. アプリのマニフェストのimageを編集してgit pushする
26. `kubectl argo rollouts get rollout guestbook-ui --namespace default --watch` でカナリアリリースの進行状況を監視する
27. 