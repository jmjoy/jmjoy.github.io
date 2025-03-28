+++
date = '2025-03-28T13:57:10+08:00'
title = 'K8S搭建Rabbitmq集群'
categories = ["运维"]
tags = ["rabbitmq"]
+++

## ConfigMap

根据实际的cluster domain和namespace修改。

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: rabbitmq-cluster
data:
  enabled_plugins: |
    [rabbitmq_peer_discovery_k8s, rabbitmq_management, rabbitmq_prometheus].
  rabbitmq.conf: |
    ## Cluster formation. See http://www.rabbitmq.com/cluster-formation.html to learn more.
    cluster_formation.peer_discovery_backend  = rabbit_peer_discovery_k8s
    cluster_formation.k8s.host = kubernetes.default.svc.cluster.local

    ## Should RabbitMQ node name be computed from the pod's hostname or IP address?
    ## IP addresses are not stable, so using [stable] hostnames is recommended when possible.
    ## Set to "hostname" to use pod hostnames.
    ## When this value is changed, so should the variable used to set the RABBITMQ_NODENAME
    ## environment variable.
    cluster_formation.k8s.address_type = hostname

    ## Important - this is the suffix of the hostname, as each node gets "rabbitmq-#", we need to tell what's the suffix
    ## it will give each new node that enters the way to contact the other peer node and join the cluster (if using hostname)
    cluster_formation.k8s.hostname_suffix = .rabbitmq-cluster.your-namespace.svc.cluster.local

    ## How often should node cleanup checks run?
    cluster_formation.node_cleanup.interval = 30

    ## Set to false if automatic removal of unknown/absent nodes
    ## is desired. This can be dangerous, see
    ##  * http://www.rabbitmq.com/cluster-formation.html#node-health-checks-and-cleanup
    ##  * https://groups.google.com/forum/#!msg/rabbitmq-users/wuOfzEywHXo/k8z_HWIkBgAJ
    cluster_formation.node_cleanup.only_log_warning = true
    cluster_partition_handling = pause_minority

    ## See http://www.rabbitmq.com/ha.html#master-migration-data-locality
    queue_master_locator = min-masters

    ## See http://www.rabbitmq.com/access-control.html#loopback-users
    loopback_users.guest = false
```

## ServiceAccount

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: rabbitmq
```

## Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: viewer
rules:
- apiGroups:
  - '*'
  resources:
  - '*'
  verbs:
  - get
  - list
  - watch
```

## RoleBinding

根据实际的namespace修改。

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: rabbitmq
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: viewer
subjects:
- kind: ServiceAccount
  name: rabbitmq
  namespace: your-namespace
```

## StatefulSet

根据实际的cluster domain修改。

```yaml
kind: StatefulSet
apiVersion: apps/v1
metadata:
  name: rabbitmq-cluster
spec:
  replicas: 3
  selector:
    matchLabels:
      app: rabbitmq-cluster
  template:
    metadata:
      labels:
        app: rabbitmq-cluster
    spec:
      volumes:
        - name: rabbitmq-config
          configMap:
            name: rabbitmq-cluster
            defaultMode: 420
      containers:
        - name: rabbitmq
          image: 'rabbitmq:3.10.25-management'
          ports:
            - name: amqp
              containerPort: 5672
              protocol: TCP
            - name: management
              containerPort: 15672
              protocol: TCP
          env:
            - name: RABBITMQ_DEFAULT_PASS
              value: guest
            - name: RABBITMQ_DEFAULT_USER
              value: guest
            - name: RABBITMQ_ERLANG_COOKIE
              value: rabbitmq-cluster
            - name: MY_POD_IP
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: status.podIP
            - name: NAMESPACE
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: metadata.namespace
            - name: HOSTNAME
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: metadata.name
            - name: RABBITMQ_USE_LONGNAME
              value: 'true'
            - name: RABBITMQ_NODENAME
              value: >-
                rabbit@$(HOSTNAME).rabbitmq-cluster.$(NAMESPACE).svc.cluster.local
            - name: K8S_SERVICE_NAME
              value: rabbitmq-cluster
          resources: {}
          volumeMounts:
            - name: rabbitmq-config
              mountPath: /etc/rabbitmq
            - name: rabbitmq-cluster-data
              mountPath: /var/lib/rabbitmq/mnesia
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          imagePullPolicy: IfNotPresent
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      dnsPolicy: ClusterFirst
      serviceAccountName: rabbitmq
      serviceAccount: rabbitmq
      securityContext:
        runAsUser: 999
        runAsGroup: 999
        fsGroup: 999
      schedulerName: default-scheduler
  volumeClaimTemplates:
    - metadata:
        name: rabbitmq-cluster-data
      spec:
        accessModes:
          - ReadWriteOnce
        resources:
          requests:
            storage: 10Gi
        volumeMode: Filesystem
  serviceName: rabbitmq-cluster
  podManagementPolicy: OrderedReady
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      partition: 0
  revisionHistoryLimit: 10
```

## Servcie

```yaml
kind: Service
apiVersion: v1
metadata:
  name: rabbitmq-cluster
  labels:
    app: rabbitmq-cluster
spec:
  ports:
    - name: amqp
      protocol: TCP
      port: 5672
      targetPort: 5672
    - name: management
      protocol: TCP
      port: 15672
      targetPort: 15672
  selector:
    app: rabbitmq-cluster
  clusterIP: None
  type: ClusterIP
  sessionAffinity: None
```
