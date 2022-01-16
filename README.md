# alpine-ssh-client

Example: You can send your machine port to remote server.

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
    name: ssh-portfowarder
spec:
    replicas: 1
    selector:
        matchLabels:
            app: ssh-portfowarder
    template:
        metadata:
            labels:
                app: ssh-portfowarder
        spec:
            containers:
                -   name: main
                    image: ghcr.io/stdll00/alpine-ssh-client:master
                    command: [ "/usr/bin/ssh", "-N" , "{USER}@{YOUR_SERVER}","-i","/mnt/ssh/ssh-privatekey","-o","ExitOnForwardFailure=yes","-o","StrictHostKeyChecking=no", "-R 8888:host.minikube.internal:80" , "-L 443:127.0.0.1:443"]
                    volumeMounts:
                        -   mountPath: /mnt/ssh
                            name: ssh-secret
                    livenessProbe:
                        httpGet:
                            path: /
                            port: 443
                            scheme: HTTPS
                            host: {YOUR_SERVER}
                            httpHeaders:
                                -   name: "Authorization"
                                    value: "Basic {BASE64 USERNAME:PASSEORD}"

            volumes:
                -   name: ssh-secret
                    secret:
                        secretName: secret-ssh-auth-portfowarder
                        defaultMode: 0600
    serviceName: "dummy"

---
apiVersion: v1
kind: Secret
metadata:
    name: secret-ssh-auth-portfowarder
type: kubernetes.io/ssh-auth
data:
    ssh-privatekey: |
        {YOUR-BASE64-SSH-KEY}

```
