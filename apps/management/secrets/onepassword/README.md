# 1Password Secrets Provider for External Secrets

This directory contains manifests for supporting [external secrets via 1Password Connect](https://external-secrets.io/main/provider/1password-automation/).

Two "secrets zero" are required in the `onepassword` namespace to bootstrap this provider as described by the External Secrets documentation.

## 1Password API Token

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: onepassword-connect-token
type: Opaque
stringData:
  token: <token>
```

## 1Password JWT

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: onepassword-connect-server-credentials
type: Opaque
stringData:
  1password-credentials.json: |-
    <base64-encoded jwt>
```

> The External Secrets documentation suggests to double-encode the JWT, but this configuration seems to work with a single decode a la `echo '<jwt>' | base64 -w 0`.

> Remember to use the "actual" JSON document with quoted fields. The JWT shown at the end of the 1Password setup form is not well-formed JSON.