# https://gitlab.chevalley.tech

based on the work of https://edenmal.moe/post/2017/GitLab-Kubernetes-GitLab-on-top-of-Kubernetes/ and https://github.com/sameersbn/docker-gitlab

## kubernetes deployment

### prepare your secret file
`echo -n "YOUR_VALUE" | base64 -w0`

```
apiVersion: v1
kind: Secret
metadata:
  labels:
    app: gitlab
  name: gitlab-secret
type: Opaque
data:
  GITLAB_SECRETS_DB_KEY_BASE: __YOUR_GITLAB_SECRETS_DB_KEY_BASE__
  GITLAB_SECRETS_SECRET_KEY_BASE: __YOUR_GITLAB_SECRETS_SECRET_KEY_BASE__
  GITLAB_SECRETS_OTP_KEY_BASE: __YOUR_GITLAB_SECRETS_OTP_KEY_BASE__
  DB_PASS: __YOUR_DB_PASS__
  SMTP_PASS: __YOUR_SMTP_PASS__
  IMAP_PASS: __YOUR_IMAP_PASS__
  GITLAB_ROOT_PASSWORD: __YOUR_GITLAB_ROOT_PASSWORD__
```

### adapt the config `gitlab-cm.yaml`

```
GITLAB_HOST
GITLAB_SSH_HOST
GITLAB_ROOT_EMAIL
TZ
GITLAB_TIMEZONE
```

### deploy
1. `kubectl create -f gitlab-namespace.yaml`
1. `kubectl create -f gitlab-secret.yaml`
1. `kubectl create -f gitlab-cm-commmon.yaml`
1. `kubectl create -f gitlab-cm-http.yaml`
1. `kubectl create -f gitlab-cm-https.yaml`
1. `kubectl create -f gitlab-postgresql-pvc.yaml`
1. `kubectl create -f gitlab-redis-pvc.yaml`
1. `kubectl create -f gitlab-ce-pvc.yaml`
1. `kubectl create -f gitlab-postgresql.yaml`
1. `kubectl create -f gitlab-redis.yaml`
1. `kubectl create -f gitlab-ce-httpX.yaml`


#### ingress SSL
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-gitlab
  annotations:
    ingress.kubernetes.io/ssl-redirect: "true"
    kubernetes.io/tls-acme: "true"
    certmanager.k8s.io/issuer: letsencrypt-prod
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/proxy-body-size: "256m"
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "300s"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "120s"
spec:
  tls:
  - hosts:
    - gitlab.gchevalley.tech
    secretName: gitlab-gchevalley-service-tls
  backend:
    serviceName: default-http-backend
    servicePort: 80
  rules:
  - host: gitlab.gchevalley.tech
    http:
      paths:
      - path: /
        backend:
          serviceName: gitlab-ce
          servicePort: 80
```

### gitlab ci runner

retrieve registration token from the page `/admin/runners`


prepare `gitlab-ci-runner-secret`

```
---
apiVersion: v1
kind: Secret
metadata:
  name: gitlab-ci-token
  namespace: gitlab-build
  labels:
    app: gitlab-ci-runner
data:
  GITLAB_CI_TOKEN: <base64 gitlab-ci-runner-registration-token>
```

1. `kubectl create -f gitlab-ci-runner-namespace.yaml`
1. `kubectl create -f gitlab-ci-runner-secret.yaml`
1. `kubectl create -f gitlab-ci-runner-rbac.yaml`
1. `kubectl create -f gitlab-ci-runner-configmap.yaml`
1. `kubectl create -f gitlab-ci-runner.yaml`

