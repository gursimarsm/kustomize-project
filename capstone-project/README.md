# Project Tasks: 

## Task 1:

Assign a `Label Transformer` to add the label defined below to `every` resource across all **3 environments**:

project: lambda

> *Only modify one Kustomization.yaml file*

-   `Label transformer` applied to all resources ?

### Solution:

The requirement is to add a `label` to **all resources in every environment**.

Since this will apply to every environment, the `commonLabel` transformer should be applied to the `kustomization.yaml` file in the `base` directory.

> `*base/kustomization.yaml*`

```yaml
commonLabels:\
  project: lambda 
  ```

## Task 2:

Please make the necessary modifications so that `all resources` in the `prod` environment should be **suffixed** with the word `-prod`

-   Resouces suffixed with `-prod` in prod environment?

### Solution:

To add a `nameSuffix` to only `production` resources, add a **nameSuffix** to the `**kustomization.yaml**` **file in the prod overlay**.

> `*overlay/prod/kustomization.yaml*`

```yaml
nameSuffix: -prod
```

## Task 3:

For `development` environment.

Perform an `inline json6902 patch` to change the image of the `db` container in the `db-deployment` to be a `mongo` image instead of `postgres`.

-   Correct `operation` used for **inline json6902 patch**?
-   Correct **container** `path` specified for the patch?
-   `mongo` image used for correct container ?

### Solution:

> `*overlay/dev/kustomization.yaml*`

```yaml
patches:\
  - target:\
      kind: Deployment\
      name: db-deployment\
    patch: |-\
      - op: replace\
        path: /spec/template/spec/containers/0\
        value:\
          name: db\
          image: mongo
```

## Task 4:

For the `production` environment, change the `replicaCount` of the `api-deployment` to `5`.

> *Use a strategic merge patch in a seperate file called* `*api-patch.yaml*`

-   `replicaCount`=**5** specified for **api-deployment** in `api-patch.yaml`?
-   Strategic merge patch included in the **kustomization.yaml**?

### Solution:

> `*overlay/prod/kustomization.yaml*`

```yaml
patches:\
  - api-patch.yaml
```

> `*overlay/prod/api-patch.yaml*`

```yaml
apiVersion: apps/v1\
kind: Deployment\
metadata:\
  name: api-deployment\
spec:\
  replicas: 5
```

## Task 5:

Create `secretGenerator` called `db-secret` with the following `literals`:

``` port=5432\
password=mypass 
```

Apply an **env variable** to the `db-deployment` container called `POSTGRES_PASSWORD` and set the value to be the **value of the password key** in the newly created `db-secret` secretgenerator.

> *Create the secret in the* `*kustomization.yaml*` *in the* `*base/db/*` *directory.*

-   secretGenerator name equals to `db-secret` ?
-   Correct `literals` used for **db-secret**?
-   `POSTGRES_PASSWORD` env var created for **db-deployment** with correct key from `db-secret`?

### Solution:

> *Create a* ***secret generator*** *in* `*base/db/kustomization.yaml*`

> `*base/db/kustomization.yaml*`

```yaml
secretGenerator:\
  - name: db-secret\
    literals:\
      - port=5432\
      - password=mypass
 ```

In the `db-deployment` add an **environment variable** referencing the secret generator:

> `*base/db/db-deployment*`

```yaml
- name: db\
          image: postgres\
          env:\
            - name: POSTGRES_PASSWORD\
              valueFrom:\
                secretKeyRef:\
                  name: db-secret\
                  key: password
```

## Task 6:

In the previous section a `secretGenerator` was created with the `two` following literals:

``` port=5432\
password=mypass
```

In the `dev` overlay **update** the `secretGenerator` so the `password=mypass` is changed to `password=devpassword`.

> *The* `*port*`***literal*** *should remain untouched.*

-   secretGenerator used to add `password=devpassword` for dev overlay?
-   Correct **behavior** used to update only one literal ?

### Solution:

To complete this task, a `secretGenerator` with the same name `db-secret` needs to be defined in the `kustomization.yaml` file in the `dev` overlay.

In there the behavior property needs to be set to **merge**, so we can update just the **password literal** and keep the port literal untouched.

> `*overlay/dev/kustomization.yaml*`

```yaml
secretGenerator:\
  - name: db-secret\
    literals:\
      - password=devpassword\
    behavior: merge
``` 

## Task 7:

Now let's add the `auth` component to the `prod` overlay and the `logging` component to the `dev` overlay.

-   `auth` component added to **prod** overlay?
-   `logging` component added to **dev** overlay?

### Solution:

> `*overlay/dev/kustomization.yaml*`

```yaml
components:\
  - ../../components/logging
```

> `*overlay/prod/kustomization.yaml*`

```yaml
components:\
  - ../../components/auth
```

## Task 8:

We want to modify the postgres container **default configuration** only in our `staging` environment.

The database team, has created a `postgresql.conf` file in the **staging overlay directory** that contains all the custom configuration.

Load the configuration in a `configMapGenerator` called `db-config` and mount it in the `db` container.

To add the mount, create a `Strategic Merge Patch` in a separate file called `db-patch.yaml` **(this should be done in the staging overlay only)** and create a volume called `config-volume` and a volume mount called `config-volume`.

> *The* `*mountPath*` *should be* `*/var/lib/postgresql/config/*`

-   `postgresql.conf` loaded in **configMapGenerator** `db-config`?
-   volume with name `config-volume` created for **db-deployment** with configMap `db-config`?
-   `volumeMount` created for `db` container with correct `name` and `mountPath`?

### Solution:

Setup the `configMapGenerator` to import the `postgresql.conf` file and create a patch as follows:

> `*overlay/staging/kustomization.yaml*`

```yaml
configMapGenerator:\
  - name: db-config\
    files:\
      - postgresql.conf\
patches:\
  - db-patch.yaml
```

Add the `volumeMount` to the container in the **patch file**

> `*overlay/staging/db-patch.yaml*`

```yaml 
apiVersion: apps/v1\
kind: Deployment\
metadata:\
  name: db-deployment\
spec:\
  template:\
    spec:\
      containers:\
        - name: db\
          volumeMounts:\
            - name: config-volume\
              mountPath: /var/lib/postgresql/config/\
      volumes:\
        - name: config-volume\
          configMap:\
            name: db-config

