Absolutely, it’s important to make the documentation clear and understandable by using the names of the resources we actually created. Here’s the revised document with specific resource names included for better clarity.

---

# Kubernetes RBAC Setup Documentation

## Overview

This document outlines the steps to set up RBAC (Role-Based Access Control) in Kubernetes, including the creation of service accounts, roles, role bindings, and managing contexts. It includes commands, YAML configurations, and key points related to the management of permissions.

## 1. Creating a Namespace

First, create a namespace for your project:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: devops-proj
```

```bash
kubectl apply -f namespace.yaml
```

## 2. Creating a Service Account

Create a service account in the `devops-proj` namespace:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: vishnu-sa
  namespace: devops-proj
```

```bash
kubectl apply -f service-account.yaml
```

## 3. Creating a Secret for the Service Account Token

Create a secret that will hold the token for the service account:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: vishnu-secret
  namespace: devops-proj
  annotations:
    kubernetes.io/service-account.name: vishnu-sa
type: kubernetes.io/service-account-token
```

```bash
kubectl apply -f secret.yaml
```

## 4. Creating a Role

Create a role that defines permissions for resources within the `devops-proj` namespace:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: devops-proj
  name: vishnu
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch", "create"]  # Add more verbs as necessary
```

```bash
kubectl apply -f role.yaml
```

## 5. Creating a RoleBinding

Bind the role to the service account:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: vishnu-rolebinding
  namespace: devops-proj
subjects:
  - kind: ServiceAccount
    name: vishnu-sa
    namespace: devops-proj
roleRef:
  kind: Role
  name: vishnu
  apiGroup: rbac.authorization.k8s.io
```

```bash
kubectl apply -f rolebinding.yaml
```

## 6. Retrieving the Token

To get the token associated with the `vishnu-sa` service account, you can retrieve it using the following command:

```bash
kubectl get secret -n devops-proj
```

Identify the secret associated with the `vishnu-sa` service account (likely `vishnu-secret`), then retrieve the token:

```bash
kubectl get secret vishnu-secret -o jsonpath='{.data.token}' | base64 --decode
```

## 7. Configuring kubectl Context

### Setting Credentials

Set the user credentials using the token obtained earlier:

```bash
kubectl config set-credentials vishnu-user --token=<TOKEN_OBTAINED_FROM_STEP_6>
```

### Creating a Context

Create a context that uses the service account and specifies the namespace:

```bash
kubectl config set-context vishnu-context --cluster=kubernetes --namespace=devops-proj --user=vishnu-user
```

### Switching Contexts

To switch to the new context, use the following command:

```bash
kubectl config use-context vishnu-context
```

## 8. Managing Roles and Permissions

### Editing an Existing Role

To add new permissions to an existing role, edit the YAML file of the role, for example:

```yaml
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch", "create", "update", "delete"]  # Add new verbs here
```

After editing, apply the changes:

```bash
kubectl apply -f role.yaml
```

### Key Points

- **Editing Roles**: To add permissions to existing roles, edit the role YAML file and reapply it. No need to rebind the role.
- **Service Account and Context**: The context (`vishnu-context`) is linked to the service account (`vishnu-sa`) through the user credentials (`vishnu-user`), allowing operations to be executed with the permissions defined by the role.
- **Token Persistence**: Once created, the service account token remains valid as long as the service account and role bindings are unchanged.

## 9. Verifying Permissions

To verify the permissions of the service account, you can describe the role and role binding:

```bash
kubectl describe role vishnu --namespace=devops-proj
kubectl describe rolebinding vishnu-rolebinding --namespace=devops-proj
```

## Conclusion

This document summarizes the key steps and commands used to set up RBAC in a Kubernetes cluster, create service accounts, roles, and role bindings, and manage contexts. By following these instructions, you can effectively manage permissions and access control within your Kubernetes environment.

---

This should provide clarity on the names of the resources we created and how they relate to the actions you need to perform. Feel free to make any further adjustments! If there’s anything else you need, just let me know!
