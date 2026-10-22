---
layout:        post
title:         "Google Cloud Workload Identity 設定備忘録"
date:          2025-11-04 00:00:00 +0900
modified_date: 2025-11-04 00:00:00 +0900
categories:    [tech, conf]
tags:          [coding]
---

![thumbnail](/assets/2025-11-04-google-cloud-workload-identity/thumbnail.png)

{% include coding-notice.md %}

## 概要

### Workload Identityとは何か
Google CloudのWorkload Identity連携（Workload Identity Federation: WIF）は、外部IDプロバイダ（OIDC/SAML）が発行する短期トークンをGoogle CloudのSecurity Token Service（STS）で交換し、サービスアカウントの一時的な資格情報として利用する仕組み。  
サービスアカウント鍵を配布・保管せずに、GitHub ActionsなどのCI/CDや他クラウド・オンプレ環境から安全にGCPへアクセスできる。

### Workload Identityのメリット・デメリット
#### メリット
- サービスアカウント鍵の廃止（キーレス）で漏えいリスクを低減
- 短命なトークンを都度発行・自動ローテーションし、運用負荷と影響範囲を最小化
- 最小権限の原則を徹底（SA権限は最小限、紐付けは属性条件でリポジトリ/ブランチ単位に制限）
- 監査性の向上（IAM監査ログとOIDCクレームで誰が何を実行したか追跡しやすい）
- マルチ環境対応（他クラウドやオンプレ、複数CIから安全に利用可能）

#### デメリット
- 初期設定がやや複雑（プール/プロバイダ、属性マッピングと条件式、バインディング設計）
- トラブルシュートに知識が必要（クレーム不一致、`aud`/`sub`、時刻ずれ、ロール不足）
- 一部の古いツール/SDKでは対応が不十分な場合がある
- トークン有効期限の制約により長時間ジョブは再認証の考慮が必要

### 参考リンク

- [デプロイメント パイプラインとの Workload Identity 連携を構成する](https://cloud.google.com/iam/docs/workload-identity-federation-with-deployment-pipelines?hl=ja#github-actions_2)
- [Configuring OpenID Connect in Google Cloud Platform](https://docs.github.com/en/actions/how-tos/secure-your-work/security-harden-deployments/oidc-in-google-cloud-platform)
- [Authenticate to Google Cloud from GitHub Actions](https://github.com/google-github-actions/auth#direct-wif)


## Workload Identity
`Google Cloud Console > IAMと管理 > Workload Identity 連携` で以下の設定を行う。

### プール作成

##### 名前
```
GitHub
```

##### プールID
```
github
```

### プロバイダ作成

##### 名前
```
GitHub
```

##### プロバイダID
```
github
```

##### 発行元（URL）
```
https://token.actions.githubusercontent.com
```

##### JWKファイル（JSON）
ブランク

##### オーディエンス
デフォルト

##### 属性のマッピング

| Google                       | OIDC                         |
|:-----------------------------|:-----------------------------|
| `google.subject`             | `assertion.sub`              |
| `attribute.actor`            | `assertion.actor`            |
| `attribute.environment`      | `assertion.environment`      |
| `attribute.sha`              | `assertion.sha`              |
| `attribute.ref`              | `assertion.ref`              |
| `attribute.ref_type`         | `assertion.ref_type`         |
| `attribute.repository`       | `assertion.repository`       |
| `attribute.repository_owner` | `assertion.repository_owner` |
| `attribute.workflow`         | `assertion.workflow`         |

##### 属性条件
```
attribute.repository == "{ owner }/{ repo }"
```


## Service Account
`Google Cloud Console > IAMと管理 > サービス アカウント` で以下の設定を行う。

### サービスアカウント作成

##### 名前
```
GitHub Actions
```

##### サービスアカウントID
```
github-actions
```

##### 権限
デプロイ対象に応じて権限を変更する。以下は `functions` をデプロイする場合。

```
Cloud Functions 開発者
Service Usage ユーザー
閲覧者
```

##### アクセス権を持つプリンシパル
プリンシパル
```
principalSet://iam.googleapis.com/projects/{ project }/locations/global/workloadIdentityPools/github/attribute.repository/{ owner }/{ repo }
```

ロール
```
Workload Identity ユーザー
```

### プリンシパル追加

#### App Engine default service account

##### アクセス権を持つプリンシパル
プリンシパル
```
github-actions@{ your-project-id }.iam.gserviceaccount.com
```

ロール
```
サービス アカウント ユーザー
```

#### Default compute service account

##### アクセス権を持つプリンシパル
プリンシパル
```
github-actions@{ your-project-id }.iam.gserviceaccount.com
```

ロール
```
サービス アカウント ユーザー
```


## GitHub Actions
以下のステップをデプロイ直前に追加する。

```yaml
- id: auth
  name: Authenticate to GCP (OIDC)
  uses: google-github-actions/auth@v3
  with:
    create_credentials_file: true
    workload_identity_provider: projects/{ project }/locations/global/workloadIdentityPools/{ pool }/providers/{ provider }
    service_account: github-actions@{ your-project-id }.iam.gserviceaccount.com

- name: gcloud auth login
  run: gcloud auth login --brief --cred-file="${ steps.auth.outputs.credentials_file_path }"
```
