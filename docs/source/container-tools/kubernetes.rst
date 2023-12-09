Kubernetes: release the kraken!
===============================

.. _Kubernetes: https://kubernetes.io/docs/tutorials/kubernetes-basics/
.. _Docker Swarm: https://docs.docker.com/engine/swarm/

`Kubernetes`_ is the default choice for container management and orchestration.

What that means is that it is somewhat similar to docker-compose, but distributed
across different cluster nodes, being able to scale up and down services on a much
larger scale than docker-compose. It is somewhat similar to `Docker Swarm`_.

As such, we have a configuration file like ``docker-compose.yml``, called a
deployment configuration file. But let's first start with installation before
proceeding.


Installation
------------

.. _kubectl: https://kubernetes.io/docs/tasks/tools/#kubectl
.. _microk8s: https://microk8s.io/

Different from docker-compose, that is readily available via ``apt`` and other
package managers, kubernetes is a pain in the ass and requires either manual
installation or a snap.

Since we don't want to deal with it, just use snap to install both `kubectl`_
and `microk8s`_ (a Kubernetes cluster). For Ubuntu, use the following commands:

.. sourcecode:: console

    $ mkdir ~/.kube
    $ sudo usermod -a -G microk8s `whoami`
    $ sudo chown -R `whoami` ~/.kube
    $ newgrp microk8s
    $ sudo snap install microk8s --classic

Since we are using ``microk8s``, and kubectl is shipped with it,
we can use ``microk8s kubectl`` or define an alias to skip ``microk8s`` part.

To set up the alias permanently, write ``sudo snap alias microk8s.kubectl kubectl``.

The entire procedure can be done via the terminal as follows:

.. sourcecode:: console

    $ kubectl
    bash: /snap/bin/kubectl: No such file or directory
    $ sudo snap alias microk8s.kubectl kubectl
    $ kubectl
    kubectl controls the Kubernetes cluster manager.
    ...

Now we have our cluster running, that we can check using:

.. sourcecode:: console

    $ microk8s status --wait-ready
    microk8s is running
    high-availability: no
      datastore master nodes: 127.0.0.1:19001
      datastore standby nodes: none
    addons:
      enabled:
        dns                  # (core) CoreDNS
        ha-cluster           # (core) Configure high availability on the current node
        helm                 # (core) Helm - the package manager for Kubernetes
        helm3                # (core) Helm 3 - the package manager for Kubernetes
      disabled:
        cert-manager         # (core) Cloud native certificate management
        cis-hardening        # (core) Apply CIS K8s hardening
        community            # (core) The community addons repository
        dashboard            # (core) The Kubernetes dashboard
        gpu                  # (core) Automatic enablement of Nvidia CUDA
        host-access          # (core) Allow Pods connecting to Host services smoothly
        hostpath-storage     # (core) Storage class; allocates storage from host directory
        ingress              # (core) Ingress controller for external access
        kube-ovn             # (core) An advanced network fabric for Kubernetes
        mayastor             # (core) OpenEBS MayaStor
        metallb              # (core) Loadbalancer for your Kubernetes cluster
        metrics-server       # (core) K8s Metrics Server for API access to service metrics
        minio                # (core) MinIO object storage
        observability        # (core) A lightweight observability stack for logs, traces and metrics
        prometheus           # (core) Prometheus operator for monitoring and logging
        rbac                 # (core) Role-Based Access Control for authorisation
        registry             # (core) Private image registry exposed on localhost:32000
        rook-ceph            # (core) Distributed Ceph storage using Rook
        storage              # (core) Alias to hostpath-storage add-on, deprecated

We can then enable some features we want, like the dashboard for easy management and registry,
for docker container images.

.. sourcecode:: console

    $ microk8s enable dashboard registry
    microk8s enable dashboard registry
    Infer repository core for addon dashboard
    Infer repository core for addon registry
    WARNING: Do not enable or disable multiple addons in one command.
             This form of chained operations on addons will be DEPRECATED in the future.
             Please, enable one addon at a time: 'microk8s enable <addon>'
    Enabling Kubernetes Dashboard
    Infer repository core for addon metrics-server
    Enabling Metrics-Server
    serviceaccount/metrics-server created
    clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
    clusterrole.rbac.authorization.k8s.io/system:metrics-server created
    rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
    clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
    clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
    service/metrics-server created
    deployment.apps/metrics-server created
    apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
    clusterrolebinding.rbac.authorization.k8s.io/microk8s-admin created
    Metrics-Server is enabled
    Applying manifest
    serviceaccount/kubernetes-dashboard created
    service/kubernetes-dashboard created
    secret/kubernetes-dashboard-certs created
    secret/kubernetes-dashboard-csrf created
    secret/kubernetes-dashboard-key-holder created
    configmap/kubernetes-dashboard-settings created
    role.rbac.authorization.k8s.io/kubernetes-dashboard created
    clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard created
    rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
    clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
    deployment.apps/kubernetes-dashboard created
    service/dashboard-metrics-scraper created
    deployment.apps/dashboard-metrics-scraper created
    secret/microk8s-dashboard-token created

    If RBAC is not enabled access the dashboard using the token retrieved with:

    microk8s kubectl describe secret -n kube-system microk8s-dashboard-token

    Use this token in the https login UI of the kubernetes-dashboard service.

    In an RBAC enabled setup (microk8s enable RBAC) you need to create a user with restricted
    permissions as shown in:
    https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md

    Infer repository core for addon hostpath-storage
    Enabling default storage class.
    WARNING: Hostpath storage is not suitable for production environments.
             A hostpath volume can grow beyond the size limit set in the volume claim manifest.

    deployment.apps/hostpath-provisioner created
    storageclass.storage.k8s.io/microk8s-hostpath created
    serviceaccount/microk8s-hostpath created
    clusterrole.rbac.authorization.k8s.io/microk8s-hostpath created
    clusterrolebinding.rbac.authorization.k8s.io/microk8s-hostpath created
    Storage will be available soon.
    The registry will be created with the size of 20Gi.
    Default storage class will be used.
    namespace/container-registry created
    persistentvolumeclaim/registry-claim created
    deployment.apps/registry created
    service/registry created
    configmap/local-registry-hosting configured

To get to the dashboard, run ``microk8s dashboard-proxy``:

.. sourcecode:: console

    $ microk8s dashboard-proxy
    Checking if Dashboard is running.
    Infer repository core for addon dashboard
    Waiting for Dashboard to come up.
    Trying to get token from microk8s-dashboard-token
    Waiting for secret token (attempt 0)
    Dashboard will be available at https://127.0.0.1:10443
    Use the following token to login:
    GIGANTIC_TOKEN_STRING_TO_COPY_AND_LOG_IN_VIA_THE_URL_ABOVE

Starting and stopping the kubernetes cluster
--------------------------------------------

If running on a battery powered device, it is recommended to shutdown the cluster
when not in use. This can be done via ``microk8s stop``.

.. sourcecode:: console

    $ microk8s stop
    Stopped.

The cluster can be re-enabled via ``microk8s start``.

.. sourcecode:: console

    $ microk8s start
    $ microk8s status
    microk8s is running
    high-availability: no
      datastore master nodes: 127.0.0.1:19001
      datastore standby nodes: none
    addons:
      enabled:
        dashboard            # (core) The Kubernetes dashboard
        dns                  # (core) CoreDNS
        ha-cluster           # (core) Configure high availability on the current node
        helm                 # (core) Helm - the package manager for Kubernetes
        helm3                # (core) Helm 3 - the package manager for Kubernetes
        hostpath-storage     # (core) Storage class; allocates storage from host directory
        metrics-server       # (core) K8s Metrics Server for API access to service metrics
        registry             # (core) Private image registry exposed on localhost:32000
        storage              # (core) Alias to hostpath-storage add-on, deprecated
    ...

Deploying the first pre-built container
---------------------------------------

Just like Docker, we start first with a pre-built image. In kubernetes-land, we use
``kubectl create deployment deployment_name --image=container_image_name``.

.. _test image in the Kubernetes manual: https://kubernetes.io/docs/tutorials/hello-minikube/

For the `test image in the Kubernetes manual`_, that contains a web server, we use the
following:

.. sourcecode:: console

    $ kubectl create deployment hello-node --image=registry.k8s.io/e2e-test-images/agnhost:2.39 -- /agnhost netexec --http-port=8080
    deployment.apps/hello-node created
    $ kubectl get deployments
    NAME         READY   UP-TO-DATE   AVAILABLE   AGE
    hello-node   1/1     1            1           2m31s
    $ kubectl get pods -A
    NAMESPACE            NAME                                         READY   STATUS    RESTARTS      AGE
    kube-system          dashboard-metrics-scraper-5657497c4c-7lxr2   1/1     Running   3 (38m ago)   15h
    kube-system          kubernetes-dashboard-54b48fbf9-qq66r         1/1     Running   3 (38m ago)   15h
    container-registry   registry-6c9fcc695f-22n2k                    1/1     Running   3 (38m ago)   15h
    kube-system          hostpath-provisioner-7df77bc496-fvxqh        1/1     Running   3 (38m ago)   15h
    kube-system          calico-kube-controllers-77bd7c5b-49qm9       1/1     Running   3 (38m ago)   15h
    kube-system          coredns-864597b5fd-gpxtv                     1/1     Running   3 (38m ago)   15h
    kube-system          calico-node-z4n4l                            1/1     Running   2 (38m ago)   15h
    kube-system          metrics-server-848968bdcd-w594k              1/1     Running   3 (38m ago)   15h
    default              hello-node-ccf4b9788-9f9rq                   1/1     Running   0             19s

As we can see, our hello-node deployment is working. We can also see the container pod that
is running the container image as part of the default namespace (since we didn't specify one).

Note that the other container pods were created by microk8s.

Sometimes our container can fail and we need to discover why.
We see how to debug next.

Debugging a deployment
----------------------

There are a few commands that can be used to help identify what went wrong during
a deployment. The primary command is ``kubectl logs name_of_pod``.

.. sourcecode:: console

    $ kubectl logs hello-node-ccf4b9788-9f9rq
    I1129 21:32:24.251151       1 log.go:195] Started HTTP server on port 8080
    I1129 21:32:24.251314       1 log.go:195] Started UDP server on port  8081

The secondary command is ``kubectl get events``. This command is related to the cluster
and not specific pods.

.. sourcecode:: console

    $ kubectl get events
    LAST SEEN   TYPE      REASON                OBJECT       MESSAGE
    90s         Warning   FreeDiskSpaceFailed   node/ryzen   Failed to garbage collect required amount of images. Attempted to free 7632552755 bytes, but only found 0 bytes eligible to free.
    90s         Warning   ImageGCFailed         node/ryzen   Failed to garbage collect required amount of images. Attempted to free 7632552755 bytes, but only found 0 bytes eligible to free.

Based on the log, our server is up and running, while listening on the ports 8080 and 8081.
We can check if this is actually the case by connecting to the server.

Exposing a service provided by a deployment
-------------------------------------------

If you tried to connect to the local IP on the port 8080, you would fail miserably.
In the case of microk8s, the cluster is hosted in a VM, that you can get the IP using
``kubectl get services``.

.. sourcecode:: console

    $ kubectl get services
    NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
    kubernetes   ClusterIP   10.152.183.1   <none>        443/TCP   20h

To expose our server, like we did in docker using port mappings, we need to use
``kubectl expose deployment deployment_name --type=LoadBalancer --port=internal_port_to_expose``.

.. sourcecode:: console

    $ kubectl expose deployment hello-node --type=LoadBalancer --port=8080
    service/hello-node exposed
    $ kubectl get services
    NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
    kubernetes   ClusterIP      10.152.183.1    <none>        443/TCP          20h
    hello-node   LoadBalancer   10.152.183.26   <pending>     8080:30582/TCP   12s

We can now reach the hosted service hosted.

.. sourcecode:: console

    $ wget http://10.152.183.26:8080/index.html
    --2023-11-29 19:31:57--  http://10.152.183.26:8080/index.html
    Connecting to 10.152.183.26:8080... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 62 [text/plain]
    Saving to: ‘index.html’
    index.html         100%[==========>]      62  --.-KB/s    in 0s
    2023-11-29 19:31:57 (4,50 MB/s) - ‘index.html’ saved [62/62]
    $ cat index.html
    NOW: 2023-11-29 22:31:57.081552945 +0000 UTC m=+3572.905677346

We can also get to the pod terminal, like we used to do with ``docker exec -it container_name``,
now using ``kubectl exec -it container_pod_name -- command``.

.. sourcecode:: console

    $ kubectl exec -it hello-node-ccf4b9788-9f9rq -- bash
    I have no name!@hello-node-ccf4b9788-9f9rq:~/$

Removed an exposed service provided by a deployment
---------------------------------------------------

To remove a service that is going to be replaced, we need to
delete that service deployment with ``kubectl delete service deployment_name``.

.. sourcecode:: console

    $ kubectl delete service hello-node
    service "hello-node" deleted
    $ kubectl get services
    NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
    kubernetes   ClusterIP   10.152.183.1   <none>        443/TCP   20h

As we can see, the service is gone.

Removing a deployment
---------------------

The deployed pods will continue to run, just not be exposed. If you want
to remove them too, delete the deployment with ``kubectl delete deployment deployment_name``.

.. sourcecode:: console

    $ kubectl delete deployment hello-node
    deployment.apps "hello-node" deleted
    $ kubectl get deployments -A
    NAMESPACE            NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
    kube-system          coredns                     1/1     1            1           20h
    kube-system          calico-kube-controllers     1/1     1            1           20h
    kube-system          dashboard-metrics-scraper   1/1     1            1           20h
    kube-system          hostpath-provisioner        1/1     1            1           20h
    kube-system          kubernetes-dashboard        1/1     1            1           20h
    container-registry   registry                    1/1     1            1           20h
    kube-system          metrics-server              1/1     1            1           20h
    $ kubectl get pods -A
    NAMESPACE            NAME                                         READY   STATUS    RESTARTS      AGE
    kube-system          dashboard-metrics-scraper-5657497c4c-7lxr2   1/1     Running   5 (66m ago)   20h
    kube-system          hostpath-provisioner-7df77bc496-fvxqh        1/1     Running   5 (66m ago)   20h
    kube-system          calico-kube-controllers-77bd7c5b-49qm9       1/1     Running   5 (66m ago)   20h
    kube-system          kubernetes-dashboard-54b48fbf9-qq66r         1/1     Running   5 (66m ago)   20h
    container-registry   registry-6c9fcc695f-22n2k                    1/1     Running   5 (66m ago)   20h
    kube-system          coredns-864597b5fd-gpxtv                     1/1     Running   5 (66m ago)   20h
    kube-system          calico-node-z4n4l                            1/1     Running   3 (67m ago)   20h
    kube-system          metrics-server-848968bdcd-w594k              1/1     Running   5 (66m ago)   20h

As we can see by the list of active deployments and container pods,
our ``hello-node`` deployment is no more.

Redeploying the first pre-built container
-----------------------------------------

After all this, you should know how to create a deployment and expose it to consumers.
However, we never told you how to update the deployed service. Now we look into that process.

Let's start by deploying nginx, like we did in the Docker-compose examples.

.. sourcecode:: console

    $ kubectl create deployment nginx --image=lscr.io/linuxserver/nginx:latest
    deployment.apps/nginx created
    $ kubectl get deployments
    NAME    READY   UP-TO-DATE   AVAILABLE   AGE
    nginx   1/1     1            1           24s
    $ kubectl expose deployment nginx --type=LoadBalancer --port=80
    service/nginx exposed
    $ kubectl get services
    NAME         TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
    kubernetes   ClusterIP      10.152.183.1     <none>        443/TCP        21h
    nginx        LoadBalancer   10.152.183.106   <pending>     80:30954/TCP   11s
    $ curl 10.152.183.106:80
        <html>
            <head>
                <title>Welcome to our server</title>
                ...
            </head>
            <body>
                <div class="message">
                    <h1>Welcome to our server</h1>
                    <p>The website is currently being setup under this address.</p>
                    <p>For help and support, please contact: <a href="me@example.com">me@example.com</a></p>
                </div>
            </body>
        </html>

Now that we have nginx deployment and service running, we can probe its details
using ``kubectl describe pods``.

.. sourcecode:: console

    $ kubectl describe pods
    Name:             nginx-5f69765c9c-qhmgk
    Namespace:        default
    Priority:         0
    Service Account:  default
    Node:             ryzen/192.168.0.114
    Start Time:       Wed, 29 Nov 2023 21:07:29 -0300
    Labels:           app=nginx
                      pod-template-hash=5f69765c9c
    Annotations:      cni.projectcalico.org/containerID: b27d314ddd6d404a83405a6e3537307cd7ed30ffc719b77a295c47885ebfaaaf
                      cni.projectcalico.org/podIP: 10.1.215.112/32
                      cni.projectcalico.org/podIPs: 10.1.215.112/32
    Status:           Running
    IP:               10.1.215.112
    IPs:
      IP:           10.1.215.112
    Controlled By:  ReplicaSet/nginx-5f69765c9c
    Containers:
      nginx:
        Container ID:   containerd://f6ce0a96698e8346b7eb8c9d650424be57c9092c8aa86df72f3f938ed8b968d2
        Image:          lscr.io/linuxserver/nginx:latest
        Image ID:       lscr.io/linuxserver/nginx@sha256:b022f503603da72a66a3d07f142c791257dcc682c7a4749881aecf0dc615b266
        Port:           <none>
        Host Port:      <none>
        State:          Running
          Started:      Wed, 29 Nov 2023 21:07:46 -0300
        Ready:          True
        Restart Count:  0
        Environment:    <none>
        Mounts:
          /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-9mxzn (ro)
    Conditions:
      Type              Status
      Initialized       True
      Ready             True
      ContainersReady   True
      PodScheduled      True
    Volumes:
      kube-api-access-9mxzn:
        Type:                    Projected (a volume that contains injected data from multiple sources)
        TokenExpirationSeconds:  3607
        ConfigMapName:           kube-root-ca.crt
        ConfigMapOptional:       <nil>
        DownwardAPI:             true
    QoS Class:                   BestEffort
    Node-Selectors:              <none>
    Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
    Events:                      <none>


Notice that the container image ID says we are using the latest version of the image.
Which means redeploying will fetch the latest image, which may be the same used in the
previous deployment.

.. _their Docker Hub: https://hub.docker.com/r/linuxserver/nginx/tags

We can specify a different image using the command
``kubectl set image deployments/deployment_name deployment_name=docker_image_name:docker_image_version``.
For nginx specifically, we have multiple possible versions to target, according to
`their Docker Hub`_ tag history. I'm picking randomly the version ``linuxserver/nginx:1.22.1-r0-ls214``.

.. sourcecode:: console

    $ kubectl set image deployments/nginx nginx=linuxserver/nginx:1.22.1-r0-ls214
    deployment.apps/nginx image updated
    $ kubectl describe pods
    Name:             nginx-5c76575475-8qkkv
    Namespace:        default
    Priority:         0
    Service Account:  default
    Node:             ryzen/192.168.0.114
    Start Time:       Wed, 29 Nov 2023 21:22:05 -0300
    Labels:           app=nginx
                      pod-template-hash=5c76575475
    Annotations:      cni.projectcalico.org/containerID: 3fa7a8421d8a18c4db96744403bc4fa54f252a438ecf1fd3fe4460ef9b8241fd
                      cni.projectcalico.org/podIP: 10.1.215.113/32
                      cni.projectcalico.org/podIPs: 10.1.215.113/32
    Status:           Running
    IP:               10.1.215.113
    IPs:
      IP:           10.1.215.113
    Controlled By:  ReplicaSet/nginx-5c76575475
    Containers:
      nginx:
        Container ID:   containerd://b2572df4b190fe7da8313ee4facf25cdf140a660079d1f5a0eb3a70201653f39
        Image:          linuxserver/nginx:1.22.1-r0-ls214
        Image ID:       docker.io/linuxserver/nginx@sha256:81ad878e810fbb84e505a72fa0a18992243ff600a89fc3d587b55eeded00af64
        Port:           <none>
        Host Port:      <none>
        State:          Running
          Started:      Wed, 29 Nov 2023 21:22:27 -0300
        Ready:          True
        Restart Count:  0
        Environment:    <none>
        Mounts:
          /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-qrq2d (ro)
    Conditions:
      Type              Status
      Initialized       True
      Ready             True
      ContainersReady   True
      PodScheduled      True
    Volumes:
      kube-api-access-qrq2d:
        Type:                    Projected (a volume that contains injected data from multiple sources)
        TokenExpirationSeconds:  3607
        ConfigMapName:           kube-root-ca.crt
        ConfigMapOptional:       <nil>
        DownwardAPI:             true
    QoS Class:                   BestEffort
    Node-Selectors:              <none>
    Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
    Events:
      Type    Reason     Age   From               Message
      ----    ------     ----  ----               -------
      Normal  Scheduled  37s   default-scheduler  Successfully assigned default/nginx-5c76575475-8qkkv to ryzen
      Normal  Pulling    34s   kubelet            Pulling image "linuxserver/nginx:1.22.1-r0-ls214"
      Normal  Pulled     16s   kubelet            Successfully pulled image "linuxserver/nginx:1.22.1-r0-ls214" in 18.219s (18.219s including waiting)
      Normal  Created    16s   kubelet            Created container nginx
      Normal  Started    16s   kubelet            Started container nginx

As we can see, right after switching the deployment image, kubernetes stopped the
running pod and switched to the new (or in this case old) version of the container image.

The deployment and service continue working as usual.

.. sourcecode:: console

    $ kubectl get deployments
    NAME    READY   UP-TO-DATE   AVAILABLE   AGE
    nginx   1/1     1            1           17m
    $ kubectl get services
    NAME         TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
    kubernetes   ClusterIP      10.152.183.1     <none>        443/TCP        22h
    nginx        LoadBalancer   10.152.183.106   <pending>     80:30954/TCP   14m
    $ curl 10.152.183.106:80
        <html>
            <head>
                <title>Welcome to our server</title>
                ...
            </head>
            <body>
                <div class="message">
                    <h1>Welcome to our server</h1>
                    <p>The website is currently being setup under this address.</p>
                    <p>For help and support, please contact: <a href="me@example.com">me@example.com</a></p>
                </div>
            </body>
        </html>

We can also check the logs:

.. sourcecode:: console

    $ kubectl logs -f nginx-5c76575475-8qkkv
    [migrations] started
    [migrations] 01-nginx-site-confs-default: executing...
    [migrations] 01-nginx-site-confs-default: succeeded
    [migrations] done
    usermod: no changes
    ───────────────────────────────────────

          ██╗     ███████╗██╗ ██████╗
          ██║     ██╔════╝██║██╔═══██╗
          ██║     ███████╗██║██║   ██║
          ██║     ╚════██║██║██║   ██║
          ███████╗███████║██║╚██████╔╝
          ╚══════╝╚══════╝╚═╝ ╚═════╝

       Brought to you by linuxserver.io
    ───────────────────────────────────────

    To support LSIO projects visit:
    https://www.linuxserver.io/donate/

    ───────────────────────────────────────
    GID/UID
    ───────────────────────────────────────

    User UID:    911
    User GID:    911
    ───────────────────────────────────────

    Setting resolver to  10.152.183.10
    Setting worker_processes to 16
    generating self-signed keys in /config/keys, you can replace these with your own keys if required
    .+......+...+...+....+...........+...............+..........+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*........+......+.........+......+...+................+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*....+.............+...+...+..+.............+..+.......+.....+...+...+.+...+......+..+.........................+.........+...+.....+..........+..+....+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
    ................+....+.....+.+..+...+....+...+..................+...+..+.........+...+.............+...+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*..+...+.+.....+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.+..........+.....+.+.....+......+....+......+.................+...+....+.........+.....+.+......+............+..+.........+..........+..+....+...+..+.+.....+.......+......+..+...+....+..+..................+......+.+......+..+.............+..+.+............+..+......+.....................+.+...............+........................+...+.....+...+...+....+..+.+..................+.....+......+.+.........+...+..+....+.....................+...+.....+.+.........+......+......+.........+......+.....+.+.................+...+....+.........+..+..........+...+......+...+.....+.+..+...+.......+..+......+.+...+......+...........+.+..+.+..+.......+.....+......+......+...............+.+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
    -----
    [custom-init] No custom files found, skipping...
    [ls.io-init] done.

The next thing we need to learn is how to configure these beasts.

Since we are reusing the same image, let's clean things up first.

.. sourcecode:: console

    $ kubectl delete deployment nginx
    deployment.apps "nginx" deleted
    $ kubectl delete services nginx
    service "nginx" deleted


Deployment manifest
-------------------

Relying on command line to tie down complex services is way too crazy for normal people.
So Kubernetes made the same decision that Docker-compose and chose to use an YAML file
to describe different deployments.

.. _this Stack Overflow response: https://stackoverflow.com/a/56259811/12280200

We can extract the deployment/pods/services manifest file using commands such as the
ones listed in `this Stack Overflow response`_, copied below for the posterity.

Export deployment, services and pod information related to a specific deployment:
*********************************************************************************

- ``kubectl get deployment,service,pod deployment_name -o yaml``

Export all deployments in all namespaces:
*****************************************

- ``kubectl get deploy --all-namespaces -o yaml``

Export all deployments, stateful sets, services, configuration maps and secrets of a namespace:
***********************************************************************************************

- ``kubectl get deploy,sts,svc,configmap,secret -n default -o yaml > default.yaml``

.. sourcecode:: console

    $ kubectl get deployment,service,pod deployment_name -o yaml --export


Let's deploy nginx yet again and use the first option.

.. sourcecode:: console

    $ kubectl create deployment nginx --image=lscr.io/linuxserver/nginx:latest
    deployment.apps/nginx created
    $ kubectl expose deployment nginx --type=LoadBalancer --port=80
    service/nginx exposed
    $ kubectl get deployment,service nginx -o yaml
    apiVersion: v1
    items:
    - apiVersion: apps/v1
      kind: Deployment
      metadata:
        annotations:
          deployment.kubernetes.io/revision: "1"
        creationTimestamp: "2023-11-30T16:44:37Z"
        generation: 1
        labels:
          app: nginx
        name: nginx
        namespace: default
        resourceVersion: "47540"
        uid: 3d3eff11-5148-4574-b7e4-54f96ad15c24
      spec:
        progressDeadlineSeconds: 600
        replicas: 1
        revisionHistoryLimit: 10
        selector:
          matchLabels:
            app: nginx
        strategy:
          rollingUpdate:
            maxSurge: 25%
            maxUnavailable: 25%
          type: RollingUpdate
        template:
          metadata:
            creationTimestamp: null
            labels:
              app: nginx
          spec:
            containers:
            - image: lscr.io/linuxserver/nginx:latest
              imagePullPolicy: Always
              name: nginx
              resources: {}
              terminationMessagePath: /dev/termination-log
              terminationMessagePolicy: File
            dnsPolicy: ClusterFirst
            restartPolicy: Always
            schedulerName: default-scheduler
            securityContext: {}
            terminationGracePeriodSeconds: 30
      status:
        availableReplicas: 1
        conditions:
        - lastTransitionTime: "2023-11-30T16:44:54Z"
          lastUpdateTime: "2023-11-30T16:44:54Z"
          message: Deployment has minimum availability.
          reason: MinimumReplicasAvailable
          status: "True"
          type: Available
        - lastTransitionTime: "2023-11-30T16:44:37Z"
          lastUpdateTime: "2023-11-30T16:44:54Z"
          message: ReplicaSet "nginx-5f69765c9c" has successfully progressed.
          reason: NewReplicaSetAvailable
          status: "True"
          type: Progressing
        observedGeneration: 1
        readyReplicas: 1
        replicas: 1
        updatedReplicas: 1
    - apiVersion: v1
      kind: Service
      metadata:
        creationTimestamp: "2023-11-30T16:44:41Z"
        labels:
          app: nginx
        name: nginx
        namespace: default
        resourceVersion: "47507"
        uid: 9e981939-2abc-4f0a-b53e-c9e2ae8646de
      spec:
        allocateLoadBalancerNodePorts: true
        clusterIP: 10.152.183.35
        clusterIPs:
        - 10.152.183.35
        externalTrafficPolicy: Cluster
        internalTrafficPolicy: Cluster
        ipFamilies:
        - IPv4
        ipFamilyPolicy: SingleStack
        ports:
        - nodePort: 32415
          port: 80
          protocol: TCP
          targetPort: 80
        selector:
          app: nginx
        sessionAffinity: None
        type: LoadBalancer
      status:
        loadBalancer: {}
    kind: List
    metadata:
      resourceVersion: ""

We can see the YAML that defines the deployment and associated service.
Redirecting the output to a file, we get the manifest file to reproduce our setup.

.. sourcecode:: console

    $ kubectl get deployment,service nginx -o yaml > nginx_manifest.yml

Now we can tear it down yet again.

.. sourcecode:: console

    $ kubectl delete deployment nginx
    deployment.apps "nginx" deleted
    $ kubectl delete services nginx
    service "nginx" deleted
    $ kubectl get deployments
    No resources found in default namespace.

Now, we can use the ``nginx_manifest.yml`` file to setup everything in a single command.

.. sourcecode:: console

    $ kubectl apply -f nginx_manifest.yml
    deployment.apps/nginx created
    service/nginx created
    $ kubectl get deployments
    NAME    READY   UP-TO-DATE   AVAILABLE   AGE
    nginx   1/1     1            1           11s
    $ kubectl get services
    NAME         TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
    kubernetes   ClusterIP      10.152.183.1     <none>        443/TCP        38h
    nginx        LoadBalancer   10.152.183.231   <pending>     80:32236/TCP   44s
    $ curl 10.152.183.231:80
    <html>
        <head>
            <title>Welcome to our server</title>
            ...
        </head>
        <body>
            <div class="message">
                <h1>Welcome to our server</h1>
                <p>The website is currently being setup under this address.</p>
                <p>For help and support, please contact: <a href="me@example.com">me@example.com</a></p>
            </div>
        </body>
    </html>

To tear it down, we can use ``kubectl delete -f manifest.yml`` instead.

.. sourcecode:: console

    $ kubectl delete -f nginx_manifest.yml
    deployment.apps "nginx" deleted
    service "nginx" deleted

