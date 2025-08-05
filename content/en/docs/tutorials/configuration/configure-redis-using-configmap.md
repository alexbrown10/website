---
reviewers:
- eparis
- pmorie
title: Configuring a ConfigMap using Redis
content_type: tutorial
weight: 30
---

<!-- overview -->

This tutorial provides a real-world example of how to configure a ConfigMap using a Redis cache. 

{{% alert title="What is Redis?" color="info" %}}

[Redis](https://github.com/redis/redis?tab=readme-ov-file#what-is-redis) (**RE**mote **DI**ctionary **S**erver) is an open source storage and data retrieval structure where the key-value pairs are cached directly in memory. This increases the data retrieval speed in comparison to other systems like SQL databases or external file systems.

{{% /alert %}}

This tutorial builds upon the [Configure a Pod to Use a ConfigMap](/docs/tasks/configure-pod-container/configure-pod-configmap/) task, **which should be completed first**. 

## {{% heading "objectives" %}}

* Create a ConfigMap with Redis configuration values.
* Create a Redis Pod that mounts and uses the created ConfigMap.
* Verify that the configuration was correctly applied.

## {{% heading "prerequisites" %}}

{{< include "task-tutorial-prereqs.md" >}} 

* All prerequisites from the [Configure a Pod to Use a ConfigMap](/docs/tasks/configure-pod-container/configure-pod-configmap/) task.

{{< version-check >}}

<!-- lessoncontent -->

## Tutorial: Configuring a ConfigMap using Redis

### Step 1: Create a ConfigMap with an empty configuration

1. Create an `example-redis-config` ConfigMap YAML file:

  In your terminal, enter:

  ```shell
  cat <<EOF >./example-redis-config.yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: example-redis-config
  data:
    redis-config: ""
  EOF
  ```

2. Apply the ConfigMap created above with the following command:

  Enter `kubectl apply -f example-redis-config.yaml`

  This should return an output similar to:

  ```shell
  > $ kubectl apply -f example-redis-config.yaml

  configmap/example-redis-config created
  ```

3. Verify the ConfigMap configuration key `redis-config` is empty.

  Enter `kubectl describe configmap example-redis-config`.

  This should return an output similar to:

  ```shell
  > $ kubectl describe configmap example-redis-config

  Name:         example-redis-config
  Namespace:    default
  Labels:       <none>
  Annotations:  <none>

  Data
  ====
  redis-config:
  ----



  BinaryData
  ====

  Events:  <none>
  ```

4. Verify the creation of the ConfigMap.

  Enter `kubectl get configmap/example-redis-config`.

  This should return an output similar to:

  ```shell
  > $ kubectl get configmap example-redis-config

  NAME                            DATA   AGE
  configmap/example-redis-config   1     17s
  ```

### Step 2: Create and mount a Redis Pod

1. Create a `redis` Pod based on the following `redis-pod.yaml` file:

  {{% code_sample file="pods/config/redis-pod.yaml" %}}

  Enter `kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/pods/config/redis-pod.yaml`.

  This should return an output similar to:

  ```shell
  > $ kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/pods/config/redis-pod.yaml

  pod/redis created
  ```

  {{% alert title="Note the following from the `redis-pod.yaml` file above:" color="info" %}}
    * `spec.volumes[1]` created a volume named `config`
    * The `key` and `path` items under `spec.volumes[1].configMap.items[0]` exposes the `redis-config` key from the 
      `example-redis-config` ConfigMap as a file named `redis.conf`.
    * The `config` volume is mounted at `/redis-master` by `spec.containers[0].volumeMounts[1]`.

    This configuration maps the (currently empty) data in `data.redis-config` from the `example-redis-config`
    ConfigMap above as `"/redis-master/redis.conf"`.
  {{% /alert %}}

2. Verify the Pod was created successfully.

  Enter `kubectl get pod/redis`.

  This should return an output similar to:

  ```shell
  > $ kubectl get pod/redis

  NAME          READY   STATUS    RESTARTS   AGE
  pod/redis     1/1     Running   0          30s
  ```

3. Examine the Pod configuration to confirm that the configuration is correct.

  Enter `kubectl describe pod/redis`.

  This should return an output similar to:

  ```shell
  > $ kubectl describe pod/redis

  Name:             redis
  Namespace:        default
  Priority:         0
  Service Account:  default
  Node:             {worker-node-name}/192.168.58.4
  Start Time:       {Day, Date Month Year HH:MM:SS} -0700
  Labels:           <none>
  Annotations:      <none>
  Status:           Running
  IP:               10.244.2.10
  IPs:
    IP:  10.244.2.10
  Containers:
    redis:
      Container ID:  docker://{container-id}
      Image:         redis:8.0.2
      Image ID:      docker-pullable://redis@sha256:{image-id}
      Port:          6379/TCP
      Host Port:     0/TCP
      Command:
        redis-server
        /redis-master/redis.conf
      State:          Running
        Started:      {Day, Date Month Year HH:MM:SS} -0700
      Ready:          True
      Restart Count:  0
      Limits:
        cpu:  100m
      Requests:
        cpu:  100m
      Environment:
        MASTER:  true
      Mounts:
        /redis-master from config (rw)
        /redis-master-data from data (rw)
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-rj2bj (ro)
  Conditions:
    Type                        Status
    PodReadyToStartContainers   True 
    Initialized                 True 
    Ready                       True 
    ContainersReady             True 
    PodScheduled                True 
  Volumes:
    data:
      Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
      Medium:     
      SizeLimit:  <unset>
    config:
      Type:      ConfigMap (a volume populated by a ConfigMap)
      Name:      example-redis-config
      Optional:  false
    kube-api-access-rj2bj:
      Type:                    Projected (a volume that contains injected data from multiple sources)
      TokenExpirationSeconds:  3607
      ConfigMapName:           kube-root-ca.crt
      Optional:                false
      DownwardAPI:             true
  QoS Class:                   Burstable
  Node-Selectors:              <none>
  Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                               node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
  Events:
    Type    Reason     Age    From               Message
    ----    ------     ----   ----               -------
    Normal  Scheduled  3m12s  default-scheduler  Successfully assigned default/redis to {worker-node-name}
    Normal  Pulling    3m12s  kubelet            Pulling image "redis:8.0.2"
    Normal  Pulled     3m7s   kubelet            Successfully pulled image "redis:8.0.2" in 5.488s (5.488s including waiting). Image size: 150474675 bytes.
    Normal  Created    3m7s   kubelet            Created container: redis
    Normal  Started    3m6s   kubelet            Started container redis
  ``` 

### Step 3: Confirm the overall configuration

1. Activate the `redis-cli` tool within the `redis` Pod.

  Enter `kubectl exec -it redis -- redis-cli`.

  This should return an output similar to:

  ```shell
  > $ kubectl exec -it redis -- redis-cli

  127.0.0.1:6379>
  ```

2. Check `maxmemory` for the `redis` Pod.

  Enter `CONFIG GET maxmemory`.

  This should return similar to:

  ```shell
  127.0.0.1:6379> CONFIG GET maxmemory

  1) "maxmemory"
  2) "0"
  ```

  which represents the default `maxmemory` value of `0`.

2. Check `maxmemory-policy`.

  Enter `CONFIG GET maxmemory-policy`.

  This should return similar to:

  ```shell
  127.0.0.1:6379> CONFIG GET maxmemory-policy
  
  1) "maxmemory-policy"
  2) "noeviction"
  ```

  which represents the default `maxmemory-policy` value of `noeviction`.

3. Exit the `redis-cli` tool with `exit`.

### Step 4: Update the ConfigMap

1. Update the `example-redis-config` ConfigMap with the desired configuration values.

  Enter `kubectl edit configmap example-redis-config` to access the `vim` editor.

2. Click `i` to enter "insert mode".

3. Add the following values to `data.redis-config`:

  ```shell
  [...]
  data:
    redis-config: | 
      maxmemory 2mb
      maxmemory-policy allkeys-lru
  ```

  {{% alert title="Important Indenting Note" color="warning" %}}

  `YAML` files are very sensitive to indentation and spacing, and incorrect formatting will result in an error and the ConfigMap will not be updated. 
  
  Ensure that the two new values in `redis-config` are indented exactly four spaces from the left margin, and that the `|` pipe character is followed by a whitespace character.

  {{% /alert %}}

4. Click `Esc` to exit "insert mode", and `:wq` to save and quit the `vim` editor.

  Successfully editing the file should return:

  ```shell
  > $ kubectl edit configmap example-redis-config

  configmap/example-redis-config edited
  ```

5. Apply the ConfigMap changes.

  Enter `kubectl apply -f example-redis-config.yaml`.

  This should return similar to:

  ```shell
  > $ kubectl apply -f example-redis-config.yaml

  configmap/example-redis-config configured
  ```

6. Verify that the ConfigMap was updated properly.

  Enter `kubectl describe configmap example-redis-config`.

  This should return similar to:

  ```shell
  > $ kubectl describe configmap/example-redis-config

  Name:         example-redis-config
  Namespace:    default
  Labels:       <none>
  Annotations:  <none>

  Data
  ====
  redis-config:
  ----
  maxmemory 2mb
  maxmemory-policy allkeys-lru



  BinaryData
  ====

  Events:  <none>
  ```

### Step 5: Recreate the `redis` Pod to apply the updated configuration

1. Delete the existing `redis` Pod.

  Enter `kubectl delete pod redis`.

  This should return similar to:

  ```shell
  > $ kubectl delete pod redis

  pod "redis" deleted
  ```

2. Recreate the `redis` Pod from the `redis-pod.yaml` file.

  Enter `kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/pods/config/redis-pod.yaml`.

  This should return an output similar to:

  ```shell
  > $ kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/pods/config/redis-pod.yaml

  pod/redis created
  ```

3. Verify the configuration update via `redis-cli`.

  Enter `kubectl exec -it redis -- redis-cli`.  

  This should return an output similar to:

  ```shell
  > $ kubectl exec -it redis -- redis-cli

  127.0.0.1:6379>
  ```

4. Check the updated`maxmemory` value for the `redis` Pod.

  Enter `CONFIG GET maxmemory`.

  This should return an output similar to:

  ```shell
  127.0.0.1:6379> CONFIG GET maxmemory

  1) "maxmemory"
  2) "2097152"
  ```

5. Check the updated `maxmemory-policy` value.

  Enter `CONFIG GET maxmemory-policy`.

  This should return an output similar to:

  ```shell
  127.0.0.1:6379> CONFIG GET maxmemory-policy

  1) "maxmemory-policy"
  2) "allkeys-lru"
  ```

### Step 6: Clean up your work by deleting the created resources.

Enter `kubectl delete pod/redis configmap/example-redis-config`.

This should return similar to:

```shell
> $ kubectl delete pod/redis configmap/example-redis-config

pod "redis" deleted
configmap "example-redis-config" deleted
```

## Conclusion

This concludes your tutorial on how to configure a ConfigMap using Redis values. If you'd like to learn more, follow our suggested **Next Steps** below:

## {{% heading "whatsnext" %}}

* Learn more about [ConfigMaps](/docs/tasks/configure-pod-container/configure-pod-configmap/).
* Learn more about [kubectl commands](/docs/reference/kubectl/).
* Follow another tutorial on [Updating configuration via a ConfigMap](/docs/tutorials/configuration/updating-configuration-via-a-configmap/).
