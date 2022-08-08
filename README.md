# play with kustomize and helm

The example shows the interaction between kustomize and helm.

It shows how resource without modifining the HelmChart, Kubernetes resources can be changed, added and deleted with the help of kustomize.

Here, using the Bitnami Wordpress HelmChart, delete the pre-templated Kubernetes Secrets and add it back with the help of the External-Secrets in Kubernetes.

## Steps

**A running k3s Kubernetes Cluster, the kubernetes-cli and an activated kubeconfig should be present**

1. Create a HashiCorp Vault instance inside the cluster

    ```..sh
    kubectl apply -f lab/manifests/helmchart-vault.yaml
    ```

    if not useing the included k3s helmController, or another non-k3s Kubernetes Cluster a alternative is to use the helm-cli:

    ```..sh
    helm upgrade --install vault \
      vault --repo https://helm.releases.hashicorp.com
      --namespace vault \
      --create-namespace \
      --set "server.dev.enabled=true" \
      --set "injector.enabled=false" \
      --set "csi.enabled=true"
    ```

2. Add values for wordpress at k/v vault

    ```..sh
    cat lab/vault-values-for-wordpress.json | \
      kubectl exec -t -i -n vault vault-0 -- \
        vault kv put secret/wordpress -
    ```

3. Add secret values for mariadb at k/v vault

    ```..sh
    cat lab/vault-values-for-mariadb.json | \
      kubectl exec -t -i -n vault vault-0 -- \
        vault kv put secret/mariadb -
    ```

4. Create the external-secretes Operator

    ```..sh
    kubectl apply -f lab/manifests/helmchart-vault.yaml
    ```

    if not useing the included k3s helmController, or another non-k3s Kubernetes Cluster a alternative is to use the helm-cli:

    ```..sh
    helm upgrade --install external-secrets \
      external-secrets --repo https://charts.external-secrets.io \
      --namespace external-secrets \
      --create-namespace \
      --set installCRDs=true
    ```

5. Kustomize, helming and apply to the cluster

    ```..sh
    kustomize build --enable-helm | \
      kubectl apply -f -
    ```

## Resources

- Kustomize
    <https://kustomize.io/>
- Helm
    <https://helm.sh/>
- Kustomize and InlinePatching
    <https://github.com/kubernetes-sigs/kustomize/blob/master/examples/inlinePatch.md>
- Kustomize and Helm usage
    <https://github.com/kubernetes-sigs/kustomize/blob/master/examples/chart.md>
- Bitnami HelmChart for Wordpress
    <https://github.com/bitnami/charts/tree/master/bitnami/wordpress>
- External-Secrets-Operator
    <https://external-secrets.io/>

## Authors

[Thorsten Schifferdecker](https://github.com/curx)
