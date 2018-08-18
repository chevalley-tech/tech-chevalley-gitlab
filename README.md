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
1. `kubectl create -f gitlab-secret.yaml`
1. `kubectl create -f gitlab-cm.yaml`
1. `kubectl create -f gitlab-postgresql-pvc.yaml`
1. `kubectl create -f gitlab-redis-pvc.yaml`
1. `kubectl create -f gitlab-ce-pvc.yaml`
1. `kubectl create -f gitlab-postgresql.yaml`
1. `kubectl create -f gitlab-redis.yaml`
1. `kubectl create -f gitlab-ce.yaml`


### ingress
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
