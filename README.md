Kustomize is a tool for customizing Kubernetes YAML configurations without using templates. It allows you to manage multiple variants of your application configuration by applying patches that modify the base resources. Kustomize is built into kubectl as a subcommand `apply -k` and can also be used as a standalone binary.

Kustomize works by reading a file called `kustomization.yaml` that specifies the base resources, patches, variables, and other options. A base resource is a Kubernetes object definition in YAML format that can be a local file, a remote URL, or a reference to another kustomization. A patch is a partial or complete resource definition that modifies or replaces the fields of a base resource. Patches can be either strategic (using merge keys to identify the fields to change) or JSON (using JSON patch syntax). Variables are key-value pairs that can be used to replace placeholders in the resource definitions.

Kustomize supports several features to help you customize your application configuration, such as:

- **Common labels and annotations**: You can add labels and annotations to all resources in a kustomization.
- **Name prefix and suffix**: You can add a prefix or suffix to the names of all resources in a kustomization.
- **Namespace**: You can set or override the namespace of all resources in a kustomization.
- **Secrets and ConfigMaps**: You can generate secrets and configmaps from literal values, files, or directories, and reference them in your resources.
- **Images**: You can change the image name, tag, or digest of containers in your resources.
- **Replicas**: You can scale up or down the number of replicas of your resources.
- **Patches**: You can apply patches to modify your resources using strategic merge patch, JSON patch, or JSON 6902 patch formats.
- **Generators**: You can create custom resource definitions (CRDs) and use them as generators to create resources dynamically.
- **Transformers**: You can apply custom transformations to your resources using Go plugins or starlark scripts.
- **Composition**: You can compose multiple kustomizations into a single one by using bases or components.

To use kustomize, you need to create a `kustomization.yaml` file in your directory and specify the resources and options you want to apply. For example, here is a simple `kustomization.yaml` file that creates a deployment and a service for an nginx application:

```yaml
resources:
- deployment.yaml
- service.yaml

commonLabels:
  app: nginx

namePrefix: dev-

replicas:
- name: nginx
  count: 3

images:
- name: nginx
  newTag: 1.19.6
```

The `resources` field lists the files that contain the base resource definitions. The `commonLabels` field adds a label `app: nginx` to all resources. The `namePrefix` field adds a prefix `dev-` to the names of all resources. The `replicas` field sets the number of replicas for the deployment named `nginx` to 3. The `images` field changes the image tag for the container named `nginx` to 1.19.6.

To apply this kustomization, you can run:

```bash
kubectl apply -k .
```

This will create the following resources in your cluster:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: dev-nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.19.6
        name: nginx

---

apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx
  name: dev-nginx
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
```

You can see that kustomize has applied the patches and options specified in the `kustomization.yaml` file to the base resources.


OK, I will continue the summary of kustomize. Here is the rest of the summary:

Kustomize is a powerful and flexible tool for customizing Kubernetes YAML configurations. It has several advantages over using templates, such as:

- **Maintainability**: You can keep your base resources unchanged and apply patches to create different variants without duplicating or modifying the original files. This makes it easier to update and manage your configurations.
- **Reusability**: You can reuse your base resources and patches across different environments and applications by using bases or components. You can also share your kustomizations with others or use existing ones from online sources.
- **Consistency**: You can ensure that your resources have consistent labels, annotations, names, namespaces, and other options by using common fields and variables. You can also validate and process your resources as plain YAML files.
- **Compatibility**: You can use kustomize with any Kubernetes object type, including custom resource definitions (CRDs). You can also use kustomize with other tools, such as Helm, ArgoCD, Flux, Kubestack, and Unfork.

To install kustomize, you can download the latest binary from the GitHub releases page or use a package manager such as Homebrew or Chocolatey. You can also use the built-in kustomize in kubectl by running `kubectl apply -k` or `kubectl kustomize`.


## Example:
 kustomization.yaml file for a Python Flask application with a Redis database:

```yaml
# kustomization.yaml
resources:
- flask-deployment.yaml
- flask-service.yaml
- redis-deployment.yaml
- redis-service.yaml

commonLabels:
  app: flask-redis

namePrefix: dev-

replicas:
- name: flask
  count: 2
- name: redis
  count: 1

images:
- name: flask
  newTag: latest
- name: redis
  newTag: 6.2.6

configMapGenerator:
- name: flask-config
  literals:
  - FLASK_APP=app.py
  - FLASK_ENV=development

secretGenerator:
- name: flask-secret
  literals:
  - FLASK_SECRET_KEY=some-random-string

patchesStrategicMerge:
- flask-patch.yaml
```

This file assumes that you have the following base resource files in your directory:

```yaml
# flask-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask
spec:
  selector:
    matchLabels:
      app: flask
  template:
    metadata:
      labels:
        app: flask
    spec:
      containers:
      - name: flask
        image: flask
        ports:
        - containerPort: 5000
        envFrom:
        - configMapRef:
            name: flask-config
        - secretRef:
            name: flask-secret

# flask-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: flask
spec:
  selector:
    app: flask
  ports:
  - protocol: TCP
    port: 80
    targetPort: 5000

# redis-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
spec:
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis

# redis-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: redis
spec:
  selector:
    app: redis
  ports:
  - protocol: TCP
    port: 6379

```

This file also assumes that you have the following patch file in your directory:

```yaml
# flask-patch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask
spec:
  template:
    spec:
      containers:
      - name: flask
        env:
        - name: REDIS_HOST 
          valueFrom:
            configMapKeyRef:
              name: redis-service # this is the name of the service created by kustomize 
              key: clusterIP # this is the key that holds the IP address of the service 
```

This patch file adds an environment variable `REDIS_HOST` to the flask container that points to the IP address of the redis service.

To apply this kustomization, you can run:

```bash 
kubectl apply -k .
```

This will create the following resources in your cluster:

```yaml 
apiVersion: v1 
kind: ConfigMap 
metadata: 
  labels: 
    app : flask-redis 
  name : dev-flask-config-gf9h9c8g8d 
data : 
  FLASK_APP : app.py 
  FLASK_ENV : development 

---

apiVersion : v1 
kind : Secret 
metadata : 
  labels : 
    app : flask-redis 
  name : dev-flask-secret-tt4m7c5g7m 
type : Opaque 
data : 
  FLASK_SECRET_KEY : c29tZS1yYW5kb20tc3RyaW5n 

---

apiVersion : v1 
kind : Service 
metadata : 
  labels : 
    app : flask-redis 
  name : dev-flask 
spec : 
  ports : 
    - port : 80 
      protocol : TCP 
      targetPort : 5000 
  selector : 
    app : flask-redis 

---

apiVersion : v1 
kind : Service 
metadata : 
  labels : 
    app : flask-redis 
  name : dev-redis 
spec : 
  ports :
    - port : 6379  
      protocol : TCP  
      targetPort :6379  
   selector :
     app :flask-redis 

---

OK, I will continue the output of the kustomization.yaml file. Here is the rest of the output:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: flask-redis
  name: dev-flask
spec:
  replicas: 2
  selector:
    matchLabels:
      app: flask-redis
  template:
    metadata:
      labels:
        app: flask-redis
    spec:
      containers:
      - name: flask
        image: flask:latest
        ports:
        - containerPort: 5000
        envFrom:
        - configMapRef:
            name: dev-flask-config-gf9h9c8g8d
        - secretRef:
            name: dev-flask-secret-tt4m7c5g7m
        env:
        - name: REDIS_HOST
          valueFrom:
            configMapKeyRef:
              name: dev-redis # this is the name of the service created by kustomize 
              key: clusterIP # this is the key that holds the IP address of the service 

---

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: flask-redis
  name: dev-redis
spec:
  replicas: 1
  selector:
    matchLabels:
      app: flask-redis
  template:
    metadata:
      labels:
        app: flask-redis
    spec:
      containers:
      - name: redis
        image: redis:6.2.6

```

You can see that kustomize has applied the patches and options specified in the kustomization.yaml file to the base resources.

I hope this output has helped you understand how kustomize works.

# Kustomize vs Helm

Kustomize and Helm are two popular tools for managing Kubernetes YAML configurations. They have different approaches and features that can suit different use cases and preferences. Here is a brief comparison of Kustomize and Helm based on the search results:

- **Philosophy**: Kustomize is a declarative tool that uses patches to customize base resources without templates. Helm is an imperative tool that uses templates and values to generate resources as packages called charts.
- **Features**: Kustomize supports common labels, annotations, name prefix, suffix, namespace, secrets, configmaps, images, replicas, patches, generators, transformers, composition, and inheritance. Helm supports packaging, sharing, installing, hooks, rollbacks, dependencies, subcharts, libraries, and schema validation.
- **Complexity**: Kustomize is simple to use and aligns with the Kubernetes philosophy. It supports an inherited-base model that scales well. It can be used natively with kubectl or as a standalone binary. Helm is more complex to use and requires learning its templating language and chart structure. It supports a dependency model that can be challenging to manage. It can be used as a standalone binary or with other tools.
- **Compatibility**: Kustomize can work with any Kubernetes object type, including custom resource definitions (CRDs). It can also work with Helm charts as bases or components. Helm can work with most Kubernetes object types, but may have some limitations with CRDs. It can also work with Kustomize as a dependency or a subchart.

Given that both Kustomize and Helm offer tool-specific benefits and complement each other well, the best course of action would be to use the two tools side by side. Helm is particularly useful for packaging, sharing, and installing apps that are well-defined, while Kustomize works best for last-minute modifications of existing Kubernetes apps.

There are two main ways to use them together:

One way is to use helm template to generate the manifest from a Helm chart and dump it into a file, and then use kubectl kustomize to apply patches to the manifest. The downside of this method is that Helm does not manage any release or lifecycle of the resources.
Another way is to use helm install or helm upgrade --install and specify a custom post-renderer that runs kubectl kustomize. The benefit of this method is that Helm manages the release and its full lifecycle, and Kustomize can modify the resources before they are applied.

### Why combine the two?

It's important to comprehend why and in what contexts you might contemplate using them together.

**You do not control the Helm chart** - One of the benefits of Helm is that it is considered the "package manager of Kubernetes". It's quite common to retrieve and employ a Helm chart that someone else published and stored in a Helm repository. What happens if you wish to modify a manifest? Kustomize makes this straightforward.

**Various environments** - One of the most intuitive applications of Kustomize is to have various configurations for different environments. Although this could be accomplished with Helm alone, it would be preferable to keep various environment customizations abstracted from Helm charts.

**Creation of Secrets** - When working with Secret and ConfigMap resources, you may not want them incorporated into your Helm charts. It is common practise to inject this sensitive data into the cluster by having Kustomize create the resources after Helm has inflated the charts.

**Cross-cutting fields** - In some cases, you may want to assign a label to all (or a subset of) the resources in a namespace, or you may want to assign a namespace to all of the resources. Normally, you wouldn't want to include this in your Helm charts, but Kustomize makes it simple to overlay this configuration on your resources.

Furthermore... - Each environment is unique, and there are likely innumerable other instances in which you may wish to combine Helm and Kustomize. Prior to making your first technical decisions, you must have a thorough understanding of your requirements.
