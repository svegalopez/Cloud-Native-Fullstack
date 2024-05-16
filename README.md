# Local Dev Environment Setup

## Using the NGINX Ingress Controller in the Local Dev Environment

You need to install it because it is not included in the local Kubernetes cluster by default.

You also need to make sure `mkcert` is installed locally in your machine. Then make sure
the local CA is installed in your machine's trust store:

```bash
mkcert -install
```

Then generate the certificate for the domain `codehead.training`:

```bash
mkcert codehead.training "*.codehead.training" codehead.training localhost 127.0.0.1 ::1
```

Then create a Kubernetes secret with the certificate:

```bash
kubectl create secret tls mydomain-tls --cert=path/to/yourdomain.local.pem --key=path/to/yourdomain.local-key.pem
```

Then install the NGINX Ingress Controller:

```bash
# Add the Nginx Helm repository
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

# Update your local Helm chart repository cache
helm repo update

# Install the ingress-nginx chart
# mydomain-tls is the name of the secret created above
helm install ingress-nginx ingress-nginx/ingress-nginx \
--set controller.extraArgs.default-ssl-certificate=mydomain-tls \
--set controller.service.type=NodePort \
--set controller.service.nodePorts.http=30905 \
--set controller.service.nodePorts.https=30906
```

Don't forget to update your machine's `/etc/hosts` file with the following entry:

```bash
127.0.0.1 acmestore.codehead.training storeapi.codehead.training
```

Finally, you can deploy the application with the following command:

```bash
helm install acmestore . -f values-dev.yaml
```

## Installing Telepresence Chart

Make sure [telepresence](https://telepresence.io) is installed in your local machine.
You can install the Telepresence chart with the following command:

```bash
telepresence helm install
```

With telepresence we can do two things:

- We can access the pg server from outside the cluster by using the in-cluster hostname, eg: acmestore-pg:5432. It essentially moves the laptop to the cluster. This is possible after running intercept for the acmestore-pg service, which triggers the installation of the telepresence traffic agent in the cluster. After disconnecting from the intercept, the agent is not removed and we can access the service from outside.

- We can intercept network requests to any service in the cluster and redirect them to a local process. This is useful for developing and debugging services in the cluster.
