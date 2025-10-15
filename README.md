# K8s 範例應用程式 (k8s-app-example)

這是一個基於 Node.js 和 Express 的簡易 Web 應用程式，主要作為 CI/CD 流程的示範專案。

## 專案概覽

本專案的核心是一個返回 "Hello World" 的簡單 HTTP 伺服器。它的主要價值不在於應用程式本身的功能，而在於它如何被整合到一個完整的自動化建置、測試與部署流程中。

## CI/CD 流程

本專案的 CI (持續整合) 流程由根目錄下的 `Jenkinsfile` 所定義，並由 Jenkins 伺服器執行。主要包含以下階段：

1.  **Checkout**: 從 GitHub 拉取最新的程式碼。
2.  **Install Dependencies**: 執行 `npm ci` 來安裝所有必要的 Node.js 套件。
3.  **Test**: 執行 `npm test` 來運行自動化測試 (如果有的話)。
4.  **Build Docker Image**:
    *   使用專案中的 `Dockerfile` 來建置應用程式的 Docker Image。
    *   將建置好的 Image 推送到 GitHub Container Registry (`ghcr.io`)。
5.  **Update GitOps**:
    *   自動 `clone` `k8s-gitops-config` repository。
    *   修改其中的 `deployment.yaml`，將 image 版本更新為剛剛建置的最新版本。
    *   將此變更 `push` 回 `k8s-gitops-config` repository，從而觸發 Argo CD 的自動部署。

## 本地開發

### 前置需求

*   [Node.js](https://nodejs.org/) (建議 v18 或以上)
*   [npm](https://www.npmjs.com/)

### 安裝與啟動

1.  安裝依賴套件：
    ```bash
    npm install
    ```

2.  啟動應用程式：
    ```bash
    node index.js
    ```

伺服器將會啟動在 `http://localhost:3000`。
