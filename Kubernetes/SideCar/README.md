# SideCar

我们还是以 Kubernetes 的 Pod 为例介绍 SideCar 模式在 K8s 中的应用：

```yml
# Example YAML configuration for the sidecar pattern.

# It defines a main application container which writes
# the current date to a log file every five seconds.

# The sidecar container is nginx serving that log file.
# (In practice, your sidecar is likely to be a log collection
# container that uploads to external storage.)

# To run:
#   kubectl apply -f pod.yaml

# Once the pod is running:
#
#   (Connect to the sidecar pod)
#   kubectl exec pod-with-sidecar -c sidecar-container -it bash
#
#   (Install curl on the sidecar)
#   apt-get update && apt-get install curl
#
#   (Access the log file via the sidecar)
#   curl 'http://localhost:80/app.txt'

apiVersion: v1
kind: Pod
metadata:
  name: pod-with-sidecar
spec:
  # Create a volume called 'shared-logs' that the
  # app and sidecar share.
  volumes:
    - name: shared-logs
      emptyDir: {}

  # In the sidecar pattern, there is a main application
  # container and a sidecar container.
  containers:
    # Main application container
    - name: app-container
      # Simple application: write the current date
      # to the log file every five seconds
      image: alpine # alpine is a simple Linux OS image
      command: ["/bin/sh"]
      args: ["-c", "while true; do date >> /var/log/app.txt; sleep 5;done"]

      # Mount the pod's shared log file into the app
      # container. The app writes logs here.
      volumeMounts:
        - name: shared-logs
          mountPath: /var/log

    # Sidecar container
    - name: sidecar-container
      # Simple sidecar: display log files using nginx.
      # In reality, this sidecar would be a custom image
      # that uploads logs to a third-party or storage service.
      image: nginx:1.7.9
      ports:
        - containerPort: 80

      # Mount the pod's shared log file into the sidecar
      # container. In this case, nginx will serve the files
      # in this directory.
      volumeMounts:
        - name: shared-logs
          mountPath: /usr/share/nginx/html # nginx-specific mount path
```
