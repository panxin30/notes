```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: bw-account
  name: bw-account #metadata.name的 值 bw-account ，将会成为创建成功后Pod的名称,集群中唯一
  namespace: crm-sprod
spec:
  progressDeadlineSeconds: 600
  replicas: 3
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: bw-account
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: bw-account
    spec:
      containers:
      - env: #给容器增加环境变量
        - name: POD_NAME
          valueFrom:
            fieldRef:
              apiVersion: v1
              fieldPath: metadata.name
        - name: POD_IP
          valueFrom:
            fieldRef:
              apiVersion: v1
              fieldPath: status.podIP
        - name: NODE_IP
          valueFrom:
            fieldRef:
              apiVersion: v1
              fieldPath: status.hostIP
        - name: TZ
          value: Asia/Shanghai
        - name: spring.cloud.consul.host
          value: 10.76.72.24
        - name: spring.cloud.consul.discovery.ip-address
          value: ${NODE_IP}
        - name: spring.cloud.consul.discovery.port
          value: "30258"
        - name: aliyun_logs_crm-sprod
          value: stdout
        image: registry-hk-tools.lwork.com/bw-account:master-29
        imagePullPolicy: IfNotPresent
        livenessProbe:
          failureThreshold: 3
          httpGet:
            path: /actuator/health
            port: 10399
            scheme: HTTP
          initialDelaySeconds: 50
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 10
        name: bw-account #container.name只是容器在Pod中的昵称
        ports:
        - containerPort: 10399
          protocol: TCP
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /actuator/health
            port: 10399
            scheme: HTTP
          initialDelaySeconds: 10
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 5
        resources:
          limits:
            cpu: "2"
            memory: 2Gi
          requests:
            cpu: 200m
            memory: 2Gi
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:
        - mountPath: /proc/cpuinfo
          name: lxcfs-proc-cpuinfo
        - mountPath: /proc/meminfo
          name: lxcfs-proc-meminfo
        - mountPath: /proc/diskstats
          name: lxcfs-proc-diskstats
        - mountPath: /proc/stat
          name: lxcfs-proc-stat
        - mountPath: /etc/localtime
          name: localtime
        - mountPath: /etc/timezone
          name: timezone
      dnsConfig:
        options:
        - name: single-request-reopen
      dnsPolicy: ClusterFirst
      imagePullSecrets:
      - name: pull-secret
      restartPolicy: Always #容器的重启策略
      schedulerName: default-scheduler
      securityContext:
        sysctls:
        - name: kernel.shm_rmid_forced
          value: "1"
        - name: net.ipv4.ip_local_port_range
          value: 1025 60000
        - name: net.core.somaxconn
          value: "40000"
        - name: net.ipv4.tcp_keepalive_time
          value: "600"
        - name: net.ipv4.tcp_syncookies
          value: "1"
        - name: net.ipv4.tcp_tw_reuse
          value: "1"
        - name: net.ipv4.tcp_timestamps
          value: "0"
        - name: net.ipv4.tcp_fin_timeout
          value: "30"
      terminationGracePeriodSeconds: 30
      volumes:
      - hostPath:
          path: /var/lib/lxcfs/proc/cpuinfo
          type: ""
        name: lxcfs-proc-cpuinfo
      - hostPath:
          path: /var/lib/lxcfs/proc/diskstats
          type: ""
        name: lxcfs-proc-diskstats
      - hostPath:
          path: /var/lib/lxcfs/proc/meminfo
          type: ""
        name: lxcfs-proc-meminfo
      - hostPath:
          path: /var/lib/lxcfs/proc/stat
          type: ""
        name: lxcfs-proc-stat
      - hostPath:
          path: /etc/localtime
          type: ""
        name: localtime
      - hostPath:
          path: /etc/timezone
          type: ""
        name: timezone
```