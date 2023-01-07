# zup-k8s

### K8s configurations for zup application.

---
![GitHub top language](https://img.shields.io/github/languages/top/cccaaannn/zup-k8s?style=flat-square) ![](https://img.shields.io/github/repo-size/cccaaannn/zup-k8s?style=flat-square) [![GitHub license](https://img.shields.io/github/license/cccaaannn/zup-k8s?style=flat-square)](https://github.com/cccaaannn/zup-k8s/blob/master/LICENSE)

### zup is a messaging application, built by microservice architecture.
### Related repos
- [Frontend](https://github.com/cccaaannn/zup-frontend)
- [User service](https://github.com/cccaaannn/zup-user-service)
- [Message service](https://github.com/cccaaannn/zup-message-service)
- [K8s configurations](https://github.com/cccaaannn/zup-k8s) (This project)

<hr>

## Configurations
1. **Set service url's for frontend. `zup-frontend-configmap.yaml`**
    1. Use HTTP if you are not going to set up `cert-manager` for HTTPS
2. **Set message service secrets, encode values as base64. `zup-message-service/zup-message-service-secret.yaml`**
    1. `POSTGRESQL_CONNECTION_STRING`: base64(`http://localhost user=user password=password dbname=zup-message-db port=5432 TimeZone=Turkey`)
    2. `RABBITMQ_CONNECTION_STRING`: base64(`amqp://user:password@localhost:5672/`)
3. **Set user service secrets, encode values as base64. `zup-user-service/zup-user-service-secret.yaml`**
    1. `spring_datasource_url`: base64(`jdbc:mysql://localhost/zup-user-db`)
    2. `spring_datasource_username`: base64(`<DB_USERNAME>`)
    3. `spring_datasource_password`: base64(`<DB_PASSWORD>`)
    4. `security_jwt_secret_key`: base64(`<RANDOM_JWT_SECRET_KEY>`)
    5. `email_gmail_username`: base64(`<EMAIL>`)
    6. `email_gmail_password`: base64(`<EMAIL_PASSWORD>`)
4. **Ingress setup**
    1. Install [helm](https://helm.sh/)
    2. Add nginx repo and install `ingress-nginx`
    ```shell
    helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
    helm repo update
    helm install ingress-nginx ingress-nginx/ingress-nginx
    ```
    3. **HTTPS**
        1. Add jetstack repo and install `cert-manager`
        ```shell
        helm repo add jetstack https://charts.jetstack.io
        helm repo update
        helm install cert-manager jetstack/cert-manager --set installCRDs=true
        ```
        2. Add your email to `zup-ingress/zup-ssl-issuer.yaml`
            - Debug certificates with `kubectl describe cert app-web-cert`, this should show `The certificate has been successfully issued`
            - `ClusterIssuer` is used so ingress has `cert-manager.io/cluster-issuer` annotation, for regular `Issuer` use `cert-manager.io/issuer` annotation.
        3. Set project url's. `zup-ingress/zup-ssl-ingress.yaml`

    - **HTTP (only)**
        1. Set project url's. `zup-ingress/zup-ingress.yaml`
        2. Comment https configs. `zup-ingress/zup-ssl-issuer.yaml`, `zup-ingress/zup-ssl-ingress.yaml`

<hr>

## Running application
1. **Run `kubectl apply` command on project's root directory to apply all configurations.**
    - Disable HTTPS ingress files if you are using HTTP  
```shell
kubectl apply -R -f .
```