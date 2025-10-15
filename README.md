# K8s GitOps 設定 (k8s-gitops-config)

這個 Repository 是我們 Kubernetes 叢集狀態的「唯一真相來源」(Single Source of Truth)。它體現了 GitOps 的核心精神，所有期望部署到叢集上的應用程式與設定，都以宣告式 (declarative) 的 YAML 檔案形式存放在這裡。

本專案由 [Argo CD](https://argo-cd.readthedocs.io/en/stable/) 持續監控與同步。

## 技術棧

*   **Kubernetes**: 容器編排平台。
*   **Argo CD**: GitOps 模式的持續交付工具。
*   **Kustomize**: 用於客製化 Kubernetes YAML 設定的工具。

## Repository 結構

本專案採用 Kustomize 的 `base` 和 `overlays` 標準結構來管理不同環境的設定。

*   `argocd/`
    *   存放 Argo CD `Application` 的定義檔。`application.yaml` 告訴 Argo CD 要去監控哪個 repo 的哪個路徑，並部署到哪個叢集。

*   `base/`
    *   存放所有環境通用的基礎 Kubernetes 設定檔，例如：
        *   `deployment.yaml`: 定義應用程式如何部署，使用哪個 image。
        *   `service.yaml`: 定義應用程式的內部網路服務。
        *   `kustomization.yaml`: Kustomize 的資源入口點。

*   `overlays/`
    *   存放特定環境的客製化設定。每個子目錄代表一個環境。
    *   `production/`: 生產環境的設定。
        *   `deployment.yaml`: 用於覆寫 (patch) `base` 中的設定，例如增加 `replicas` 數量。
        *   `ingress.yaml`: 定義應用程式如何對外暴露服務。
        *   `kustomization.yaml`: 宣告此 overlay 會使用 `base` 的資源，並加上自己的客製化設定。

*   `system/`
    *   存放 CI/CD 系統工具或其他叢集級服務的設定檔。
    *   `jenkins/`: 存放 Jenkins 的 Kubernetes 部署設定。

## GitOps 工作流程

1.  任何對此 repo `main` 分支的 `push` 操作，都會被 Argo CD 偵測到。
2.  Argo CD 會比較 Git 中的「期望狀態」與 Kubernetes 叢集中的「實際狀態」。
3.  如果兩者不一致，Argo CD 會自動執行「同步 (Sync)」操作，將叢集調整成和 Git 中定義的一樣。
4.  應用程式的 image 版本更新，通常由 CI 系統 (例如 Jenkins) 自動修改 `overlays/production/deployment.yaml` 並推送到此 repo 來觸發。
