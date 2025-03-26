# skupper-sizing-example
Example of configuring site sizing with v2 controller.

Apply the sizing-config.yaml to the namespace where the controller is
running. Then apply the two site resources - small-site.yaml and
default-site.yaml - to different namespaces covered by that
controller.

The resources specified should match the small configuration in the
first instance and the medium conifguration in the second (as that is
set to be the default).

E.g.

```
$ helm install -n skupper skupper oci://quay.io/skupper/helm/skupper --version 2.0.0
Pulled: quay.io/skupper/helm/skupper:2.0.0
Digest: sha256:3e21caca7f16e5981b4940807f5d545408c4b71078c9ffaaf25d5d7a8aeb7792
NAME: skupper
LAST DEPLOYED: Wed Mar 26 12:36:28 2025
NAMESPACE: skupper
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
===========================================================
  Skupper chart is now installed in the cluster.
  Skupper controller was deployed in the namespace "skupper".

===========================================================
$ kubectl -n skupper apply -f size-config.yaml
configmap/skupper-site-small created
configmap/skupper-site-medium created
configmap/skupper-site-large created
$ kubectl create namespace one
namespace/one created
$ kubectl apply -n one -f small-site.yaml
site.skupper.io/one created
$ kubectl create namespace two
namespace/two created
$
$ kubectl apply -n two -f default-site.yaml
site.skupper.io/two created
$
$ kubectl get deployment -n one skkupper-router
Error from server (NotFound): deployments.apps "skkupper-router" not found
$ kubectl get deployment -n one skupper-router
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
skupper-router   0/1     1            0           30s
$ kubectl get deployment -n one skupper-router -o yaml | grep -C 5 resources
            scheme: HTTP
          initialDelaySeconds: 1
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
        resources:
          limits:
            cpu: 400m
            memory: 500M
          requests:
            cpu: 300m
--
            scheme: HTTP
          initialDelaySeconds: 1
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
        resources:
          limits:
            cpu: 300m
            memory: 200M
          requests:
            cpu: 200m
--
        - name: SKUPPER_ROUTER_CONFIG
          value: skupper-router
        image: quay.io/skupper/kube-adaptor:2.0.0
        imagePullPolicy: Always
        name: config-init
        resources:
          limits:
            cpu: 300m
            memory: 200M
          requests:
            cpu: 200m
$ kubectl get deployment -n two skupper-router -o yaml | grep -C 5 resources
            scheme: HTTP
          initialDelaySeconds: 1
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
        resources:
          limits:
            cpu: 500m
            memory: 700M
          requests:
            cpu: 400m
--
            scheme: HTTP
          initialDelaySeconds: 1
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
        resources:
          requests:
            cpu: 200m
            memory: 100M
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
--
        - name: SKUPPER_ROUTER_CONFIG
          value: skupper-router
        image: quay.io/skupper/kube-adaptor:2.0.0
        imagePullPolicy: Always
        name: config-init
        resources:
          requests:
            cpu: 200m
            memory: 100M
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File

```
