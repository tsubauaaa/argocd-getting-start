## 構築方法

### microk8s install
1. brew install ubuntu/microk8s/microk8sでmicrok8sのインストーラーをDL
2. microk8s installでmicrok8sをインストール
3. microk8s enable dns && microk8s stop && microk8s start microk8sを開始する

### argocd install
4. microk8s kubectl create namespace argocdでnamespaceを作成する
5. microk8s kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml でargocdをインストール
6. [brew install argocd](https://kostis-argo-cd.readthedocs.io/en/refresh-docs/getting_started/install_cli/#install-on-macos-darwin)からargocd cliをインストール
7. microk8s kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'でargocd-serverをLoadBalancer typeに変更
8. microk8s kubectl port-forward svc/argocd-server -n argocd 8080:443で8080にport forwardする
7. https://localhost:8080にアクセスしてargocd管理画面を表示する
8. microk8s kubectl get secret --namespace argocd argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 --decode ; echoでadminの初期パスワードを取得
9. argocd管理画面にログインしてパスワードを変更
10. microk8s kubectl config set-context --current --namespace=argocdでnamespaceをset
11. ~/.kube/configを~/.microk8s/configに置き換える

### application create
12. argocd login localhost:8080でログインする
13. argocd app create guestbook --repo https://github.com/tsubauaaa/argocd-getting-start.git --path guestbook --dest-server https://kubernetes.default.svc --dest-namespace default --port-forward-namespace argocdでアプリを作成する
14. argocd app get guestbookで作成したアプリを確認する
15. argocd app sync guestbookでアプリを同期する

### convert Deployment to Rollout
16. [Convert Deployment to Rollout](https://argoproj.github.io/argo-rollouts/migrating/#convert-deployment-to-rollout)からアプリのマニフェストを編集してgit pushする
17. kubectl create namespace argo-rolloutsにてnamespaceを作成する
18. kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yamlにてrolloutsをインストールする
19. [Kubectl Plugin Installation](https://argoproj.github.io/argo-rollouts/installation/#kubectl-plugin-installation)からkubectl argo rolloutsをインストールする
20. kubectl argo rollouts dashboard --namespace defaultでargo rollouts dashboardを起動する
21. argocd管理画面からpruneありのsyncを行う
22. http://localhost:3100/rolloutsにアクセスしてargo rollouts管理画面を表示する

