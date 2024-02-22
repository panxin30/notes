 # server
```
        terminationMessagePolicy: File
        volumeMounts:
        - mountPath: /preconfigure_server.sh
          name: config-vol
          subPath: preconfigure_server.sh
        - mountPath: /godata
          name: goserver-vol
          subPath: godata
        - mountPath: /home/go
          name: goserver-vol
          subPath: homego
        - mountPath: /docker-entrypoint.d
          name: goserver-vol
          subPath: scripts
      dnsPolicy: ClusterFirst
      hostAliases:
      - hostnames:
        - gitlab.51sw.cc
        ip: 192.168.60.236
      - hostnames:
        - vpn.51sw.cc
        ip: 192.168.60.179
      nodeSelector:
        gocd: client
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext:
        fsGroup: 0
        fsGroupChangePolicy: Always
        runAsGroup: 0
        runAsUser: 1000
      serviceAccount: gocd
      serviceAccountName: gocd
      terminationGracePeriodSeconds: 30
      volumes:
      - configMap:
          defaultMode: 420
          name: gocd
        name: config-vol
      - hostPath:
          path: /data/ops
          type: Directory
        name: goserver-vol
```
# agent
```
        terminationMessagePolicy: File
        volumeMounts:
        - mountPath: /home/go
          name: goserver-vol
          subPath: homego
        - mountPath: /var/run/docker.sock
          name: docker-sock
      dnsPolicy: ClusterFirst
      hostAliases:
      - hostnames:
        - gitlab.51sw.cc
        ip: 192.168.60.236
      - hostnames:
        - vpn.51sw.cc
        ip: 192.168.60.179
      nodeSelector:
        gocd: client
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext:
        fsGroup: 0
        fsGroupChangePolicy: Always
        runAsGroup: 0
        runAsUser: 1000
      serviceAccount: default
      serviceAccountName: default
      terminationGracePeriodSeconds: 30
      volumes:
      - hostPath:
          path: /data/ops
          type: Directory
        name: goserver-vol
      - hostPath:
          path: /var/run/docker.sock
          type: ""
        name: docker-sock
```