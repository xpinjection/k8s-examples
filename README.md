# k8s-examples

Basic examples for K8S training.

### Sample Kuard application deployment

Sample Kuard application could be deployed from plain manifests or the Helm chart. Access to UI could be automatically granted in Nginx Ingress Controller is installed to process Ingress rules or with simple port forwarding. 

1. Create TLS secret:

> _kubectl create secret generic kuard-tls --from-file=k8s/demo/kuard.crt --from-file=k8s/demo/kuard.key_

2. Deploy application with plain manifests or Helm chart:

> _kubectl apply -f k8s/demo_

OR

> _helm install kuard k8s/kuard_

### Local installation of Nginx Ingress Controller

Installation could be performed with official Helm chart: https://artifacthub.io/packages/helm/ingress-nginx/ingress-nginx. 

1. Add Helm repository: 

> _helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx_

> _helm repo update_

2. Install chart with local configuration:

> _helm install nginx ingress-nginx/ingress-nginx -f k8s/nginx/values-local.yaml_

### Basic ArgoCD installation

ArgoCD could be installed from the git repository by official instruction: https://argo-cd.readthedocs.io/en/stable/getting_started/.

1. Create dedicated namespace:

> _kubectl create namespace argocd_

2. Install from the plain manifests in git repository:

> _kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml_

3. To access ArgoCD UI either patch service to type LoadBalancer or just forward port:

> _kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'_

OR

> _kubectl port-forward svc/argocd-server -n argocd 8080:443_

4. Grab default admin password from the secret to access UI:

> _kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo_

5. Deploy sample application:

> _kubectl apply -f k8s/argocd/kuard-app.yaml_

### Sample Spring Boot application with Spring Boot Admin deployment

To demonstrate how Spring Boot Admin could use K8S discovery to automatically detect Sprint Boot applications in the K8S cluster following instructions could be used.

1. Build local image _xpinjection.com/library:0.1.0_ for Spring Boot sample project: https://github.com/xpinjection/test-driven-spring-boot.

2. Build local image _xpinjection.com/boot-admin:0.1.0_ for Spring Boot sample project: https://github.com/xpinjection/spring-boot-admin.

3. Create required secrets:

> _kubectl create secret generic actuator-credentials --from-literal=password=xpinjection --from-literal=username=admin_

> _kubectl create secret generic library-db-credentials --from-literal=spring.datasource.username=test --from-literal=spring.datasource.password=test_

> _kubectl create secret generic boot-admin-auth-credentials --from-literal=spring.security.user.password=xpinjection --from-literal=spring.security.user.name=boot-admin_

4. Install PostgreSQL with the official Helm chart: https://bitnami.com/stack/postgresql/helm.

> _helm repo add bitnami https://charts.bitnami.com/bitnami_

> _helm repo update_

> _helm install postgres bitnami/postgresql --set auth.username=test,auth.password=test,auth.database=library_

5. Deploy Spring Boot library application:

> _kubectl apply -f k8s/library_

6. Deploy Spring Boot Admin application:

> _kubectl apply -f k8s/boot-admin_
