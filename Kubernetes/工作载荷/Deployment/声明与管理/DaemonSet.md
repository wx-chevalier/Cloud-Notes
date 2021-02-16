# DaemonSet

** DaemonSet ** is basically the same as ReplicaSet, with the difference that when you use DaemonSet you do not specify the number of replicas, it will go up one pod per node in your cluster.

It is always interesting when creating, using and abusing labels, so you will be able to have better flexibility in the most appropriate distribution of your application.

It is very interesting for services that need to run on all nodes in the cluster, such as log collectors and monitoring agents.

Let's create our first DaemonSet:

`I came first daemonset.yaml`

The content should be as follows:

`yaml apiVersion: apps / v1 kind: DaemonSet metadata: name: daemon-set-first spec: selector: matchLabels: system: Strigus template: metadata: labels: system: Strigus spec: containers: - name: nginx image: nginx: 1.7.9 ports: - containerPort: 80`

But first let’s allow all of our nodes to run pods:

```
kubectl taint nodes --all node-role.kubernetes.io/master-

node / elliot-01 untainted
taint "node-role.kubernetes.io/master:" not found
taint "node-role.kubernetes.io/master:" not found
```

Now we can create our DaemonSet:

```
kubectl create -f first-daemonset.yaml

daemonset.extensions / daemon-set-first created
```

Let's list our DaemonSet:

```
kubectl get daemonset

NAME DESIRED CURRENT READY UP-TO-DATE ... AGE
daemon-set-first 3 3 3 3 30s
```

Viewing the DaemonSet details:

```
kubectl describe ds daemon-set-first

Name: daemon-set-first
Selector: system = Strigus
Node-Selector: <none>
Labels: system = Strigus
Annotations: <none>
Desired Number of Nodes Scheduled: 3
Current Number of Nodes Scheduled: 3
Number of Nodes Scheduled with Up-to-date Pods: 3
Number of Nodes Scheduled with Available Pods: 3
Number of Nodes Misscheduled: 0
Pods Status: 3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
Labels: system = Strigus
Containers:
nginx:
Image: nginx: 1.7.9
Port: 80 / TCP
Host Port: 0 / TCP
Environment: <none>
Mounts: <none>
Volumes: <none>
Events:
Type Reason Age From Message

---

Normal SuccessfulCreate 41s daemonset-controller Created pod: daemon-set-first-jl6f5
Normal SuccessfulCreate 412 daemonset-controller Created pod: daemon-set-first-jh2sp
Normal SuccessfulCreate 412 daemonset-controller Created pod: daemon-set-first-t9rv9
```

Viewing pod details:

```
kubectl get pods -o wide

NAME READY STATUS RESTARTS AGE .. NODE
daemon-set-first .. 1/1 Running 0 1m elliot-01
daemon-set-first .. 1/1 Running 0 1m elliot-02
daemon-set-first .. 1/1 Running 0 1m elliot-03
```

As we can see we have one pod per node running our ` daemon-set-first`.

Let's change the image of this pod directly in DaemonSet, using the command ` kubectl set`:

```
kubectl set image ds daemon-set-first nginx = nginx: 1.15.0

daemonset.extensions / daemon-set-first image updated
```

Let's confirm that the image has really been changed:

```
kubectl describe ds daemon-set-first

Name: daemon-set-first
Selector: system = Strigus
Node-Selector: <none>
Labels: system = Strigus
Annotations: <none>
Desired Number of Nodes Scheduled: 3
Current Number of Nodes Scheduled: 3
Number of Nodes Scheduled with Up-to-date Pods: 0
Number of Nodes Scheduled with Available Pods: 3
Number of Nodes Misscheduled: 0
Pods Status: 3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
Labels: system = Strigus
Containers:
nginx:
Image: nginx: 1.15.0
Port: 80 / TCP
Host Port: 0 / TCP
Environment: <none>
Mounts: <none>
Volumes: <none>
Events:
Type Reason Age From Message

---

Normal SuccessfulCreate 2m daemonset-controller Created pod: daemon-set-first-jl6f5
Normal SuccessfulCreate 2m daemonset-controller Created pod: daemon-set-first-jh2sp
Normal SuccessfulCreate 2m daemonset-controller Created pod: daemon-set-first-t9rv9
```

Now let's check if the pod images are up to date:

```
kubectl get pods

NAME READY STATUS RESTARTS AGE
daemon-set-first-jh2sp 1/1 Running 0 2m
daemon-set-first-jl6f5 1/1 Running 0 2m
daemon-set-first-t9rv9 1/1 running 0 2m
```

As we can see, we had no restart on the pods.

Let's check the image running on one of the pods:

```
kubectl describe pod daemon-set-first-jh2sp | grep -i image:

Image: nginx: 1.7.9
```

Exactly, we were unable to change information from the running DaemonSet.

What if the pod is deleted?

```
kubectl delete pod daemon-set-first-jh2sp

pod "daemon-set-first-jh2sp" deleted
```

Viewing the pods:

```
kubectl get pods

NAME READY STATUS RESTARTS AGE
daemon-set-first-hp4qc 1/1 Running 0 3s
daemon-set-first-jl6f5 1/1 running 0 10m
daemon-set-first-t9rv9 1/1 running 0 10m
```

Let's list the new Pod that was created, after deleting the old Pod:

```
kubectl describe pod daemon-set-first-hp4qc | grep -i image:

    Image: nginx: 1.15.0

```

Now a Pod that was already running:

```
kubectl describe pod daemon-set-first-jl6f5 | grep -i image:

    Image: nginx: 1.7.9

```

As we can see, to update all the pods in DaemonSet we need to recreate it or destroy all the pods related to it, but isn't that too bad? Yes, it's really bad. To improve our lives we have the option ` RollingUpdate` that we will see in the next chapter.

# Rollouts and Rollbacks

Now let's imagine that our last edition using the command ` kubectl set` in DaemonSet was not correct and we need to go back to the previous configuration, where the image version was different, how do we do it?

It's very simple, for that there is _Rollout_. With it you can check what were the changes that happened in your Deployment or DaemonSet, as if it were a version. Look! (With the voice of Nelson Rubens)

```
kubectl rollout history ds daemon-set-first

daemonsets "daemon-set-first"
REVISION CHANGE-CAUSE
1 <none>
2 <none>
```

It will show two lines, the first which is the original, with the image of ` nginx: 1.7.9` and the second already with the image ` nginx: 1.15.0`. The information is not very detailed, do you agree?

Here's how to check the details of each of these entries, which are called ** revision **.

Viewing revision 1:

```
kubectl rollout history ds daemon-set-first --revision = 1

daemonsets "daemon-set-first" with revision # 1
Pod Template:
Labels: system = DaemonOne
Containers:
nginx:
Image: nginx: 1.7.9
Port: 80 / TCP
Host Port: 0 / TCP
Environment: <none>
Mounts: <none>
Volumes: <none>
```

Viewing revision 2:

```
kubectl rollout history ds daemon-set-first --revision = 2

daemonsets "daemon-set-first" with revision # 2
Pod Template:
Labels: system = DaemonOne
Containers:
nginx:
Image: nginx: 1.15.0
Port: 80 / TCP
Host Port: 0 / TCP
Environment: <none>
Mounts: <none>
Volumes: <none>
```

To return to the desired revision, simply do the following:

```
kubectl rollout undo ds daemon-set-first --to-revision = 1

daemonset.extensions / daemon-set-first rolled back
```

Notice that we changed the ` history` for ` undo` and the ` revision` for ` to-revision`, so we will do the ** rollback ** in our DaemonSet, and return the version of the image that we wish. 😃

---

> NOTE: By default, DaemonSet stores only the last 10 revisions. To change the maximum number of revisions in our Daemonset, run the following command.
> Source: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#clean-up-policy

---

`kubectl edit daemonsets.apps daemon-set-first`

Change the quantity in the ` revisionHistoryLimit` parameter:

`yaml revisionHistoryLimit: 10`

---

Returning to our line of reasoning, to follow the rollout, execute the following command:

`kubectl rollout status ds daemon-set-first`

Let's confirm that we are already running the new image and one of our pods:

```
kubectl describe pod daemon-set-first-hp4qc | grep -i image:

Image: nginx: 1.15.0
```

It didn't work, why? Because we will have to kill the Pod to be recreated with the new settings.

Let's fine-tune our DaemonSet, add the RollingUpdate and this guy will automatically update the Pods when there are changes.

Come on, first let's remove the ` DaemonSet`, add two new information to our yaml manifest and then create another DaemonSet instead:

```
kubectl delete -f first-daemonset.yaml

daemonset.extensions "daemon-set-first" deleted
```

Edit the ` first-daemonset.yaml` file.

`I came first daemonset.yaml`

The content should be as follows:

`yaml apiVersion: apps / v1 kind: DaemonSet metadata: name: daemon-set-first spec: selector: matchLabels: system: Strigus template: metadata: labels: system: Strigus spec: containers: - name: nginx image: nginx: 1.7.9 ports: - containerPort: 80 updateStrategy: type: RollingUpdate`

Create the DaemonSet:

```
kubectl create -f first-daemonset.yaml

daemonset.extensions / daemon-set-first created
```

Success, let's check if our DaemonSet has started up correctly.

```
kubectl get daemonset

NAME DESIRED CURRENT READY ... AGE
daemon-set-first 3 3 3 ... 5m
```

Viewing the DaemonSet details:

```
kubectl describe ds daemon-set-first

Name: daemon-set-first
Selector: system = DaemonOne
Node-Selector: <none>
Labels: system = DaemonOne
Annotations: <none>
Desired Number of Nodes Scheduled: 3
Current Number of Nodes Scheduled: 3
Number of Nodes Scheduled with Up-to-date Pods: 3
Number of Nodes Scheduled with Available Pods: 3
Number of Nodes Misscheduled: 0
Pods Status: 3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
Labels: system = DaemonOne
Containers:
nginx:
Image: nginx: 1.7.9
Port: 80 / TCP
Host Port: 0 / TCP
Environment: <none>
Mounts: <none>
Volumes: <none>
Events:
Type Reason Age From Message

---

Normal SuccessfulCreate 5m daemonset-controller Created pod: daemon-set-first-52k8k
Normal SuccessfulCreate 5m daemonset-controller Created pod: daemon-set-first-6sln2
Normal SuccessfulCreate 5m daemonset-controller Created pod: daemon-set-first-9v2w9
daemonset-controller Created pod: daemon-set-first-9dktj
```

Let's check out our newly added ` RollingUpdate` configuration:

```
kubectl get ds daemon-set-first -o yaml | grep -A 2 Strategy

updateStrategy:
rollingUpdate:
maxUnavailable: 1
```

Now with our DaemonSet already configured, let's change that same ` nginx` image and see what actually happens:

```
kubectl set image ds daemon-set-first nginx = nginx: 1.15.0

daemonset.extensions / daemon-set-first image updated
```

Let's list the DaemonSet and the Pods to make sure nothing is broken:

```
kubectl get daemonset

NAME DESIRED CURRENT READY ... AGE
daemon-set-first 3 3 3 ... 6m
```

Viewing the pods:

```
kubectl get pods -o wide

NAME READY STATUS RESTARTS AGE NODE
daemon-set-first-7m ... 1/1 Running 0 10s elliot-02
daemon-set-first-j7 ... 1/1 Running 0 10s elliot-03
daemon-set-first-v5 ... 1/1 Running 0 10s elliot-01
```

As we can see, our DaemonSet has remained the same, but the Pods have been recreated, we will detail the DaemonSet to see the changes made.

```
kubectl describe ds daemon-set-first

Name: daemon-set-first
Selector: system = DaemonOne
Node-Selector: <none>
Labels: system = DaemonOne
Annotations: <none>
Desired Number of Nodes Scheduled: 3
Current Number of Nodes Scheduled: 3
Number of Nodes Scheduled with Up-to-date Pods: 3
Number of Nodes Scheduled with Available Pods: 3
Number of Nodes Misscheduled: 0
Pods Status: 3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
Labels: system = DaemonOne
Containers:
nginx:
Image: nginx: 1.15.0
Port: 80 / TCP
Host Port: 0 / TCP
Environment: <none>
Mounts: <none>
Volumes: <none>
Events:
Type Reason Age From Message

---

Normal SuccessfulCreate 8m daemonset-controller Created pod: daemon-set-first-52k8k
Normal SuccessfulCreate 8m daemonset-controller Created pod: daemon-set-first-6sln2
Normal SuccessfulCreate 8m daemonset-controller Created pod: daemon-set-first-9v2w9
Normal SuccessfulDelete 10m daemonset-controller Deleted pod: daemon-set-first-6sln2
Normal SuccessfulCreate 1m daemonset-controller Created pod: daemon-set-first-j788v
Normal SuccessfulDelete 10m daemonset-controller Deleted pod: daemon-set-first-52k8k
Normal SuccessfulCreate 1m daemonset-controller Created pod: daemon-set-first-7mpwr
Normal SuccessfulDelete 10m daemonset-controller Deleted pod: daemon-set-first-9v2w9
Normal SuccessfulCreate 1m daemonset-controller Created pod: daemon-set-first-v5m47
```

Look how cool! If we look at the ** Events ** field we can see that the ` RollingUpdate` killed the old pods and recreated with the new image that we changed using the ` kubectl set`.

We can also check in one of the Pods if this change really happened.

```
kubectl describe pod daemon-set-first-j788v | grep -i image:

Image: nginx: 1.15.0
```

See? Very sensational this business of ` RollingUpdate`.

Let's check out our change history:

```
kubectl rollout history ds daemon-set-first

daemonsets "daemon-set-first"
REVISION CHANGE-CAUSE
1 <none>
2 <none>
```

Yes, we have two changes. Let's drill down to see which is which.

Viewing revision 1:

```
kubectl rollout history ds daemon-set-first --revision = 1

daemonsets "daemon-set-first" with revision # 1
Pod Template:
Labels: system = DaemonOne
Containers:
nginx:
Image: nginx: 1.7.9
Port: 80 / TCP
Host Port: 0 / TCP
Environment: <none>
Mounts: <none>
Volumes: <none>
```

Viewing revision 2:

```
kubectl rollout history ds daemon-set-first --revision = 2

daemonsets "daemon-set-first" with revision # 2
Pod Template:
Labels: system = DaemonOne
Containers:
nginx:
Image: nginx: 1.15.0
Port: 80 / TCP
Host Port: 0 / TCP
Environment: <none>
Mounts: <none>
Volumes: <none>
```

Now let's rollback our DaemonSet to revision 1:

```
kubectl rollout undo ds daemon-set-first --to-revision = 1

daemonset.extensions / daemon-set-first rolled back
kubectl rollout undo ds daem kubectl rollout undo ds daem
```

Viewing the pods:

```
kubectl get pods

NAME READY STATUS RESTARTS AGE
daemon-set-first-c2jjk 1/1 Running 0 19s
daemon-set-first-hrn48 1/1 Running 0 19s
daemon-set-first-t6mr9 1/1 Running 0 19s
```

Viewing pod details:

```
kubectl describe pod daemon-set-first-c2jjk | grep -i image:

Image: nginx: 1.7.9
```

Sensational isn't it?

Did it go bad?

Just return to the other setting:

```
kubectl rollout undo ds daemon-set-first --to-revision = 2

daemonset.extensions / daemon-set-first rolled back
```

Viewing the rollout status:

```
kubectl rollout status ds daemon-set-first

daemon set "daemon-set-first" successfully rolled out
```

Viewing the pods:

```
kubectl get pods

NAME READY STATUS RESTARTS AGE
daemon-set-first-jzck9 1/1 Running 0 32s
daemon-set-first-td7h5 1/1 Running 0 29s
daemon-set-first-v5c86 1/1 Running 0 40s
```

Viewing pod details:

```
kubectl describe pod daemon-set-first-jzck9 | grep -i image:

Image: nginx: 1.15.0
```

Now let's delete our DaemonSet:

```
kubectl delete ds daemon-set-first

daemonset.extensions "daemon-set-first" deleted
```
