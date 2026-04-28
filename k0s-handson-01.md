# k0s Hands-On 01: Kubernetes Basics in Action

This hands-on guide demonstrates a few core Kubernetes behaviors using a simple NGINX deployment on k0s:

- Self-healing when a pod fails
- Scaling a deployment up and down
- Performing a rolling update with no downtime
- Moving from imperative commands to declarative YAML

## Prerequisites

- A running k0s cluster
- `kubectl` configured to access the cluster
- A sample application already deployed as `nginx-test`

If you have not created the sample deployment yet, do that first and then return to this guide.

## Exercise 1: See Self-Healing in Action

Kubernetes automatically replaces failed pods to keep your application available.

1. List the running pods:

   ```bash
   kubectl get pods
   ```

2. Copy the full pod name from the output and delete it:

   ```bash
   kubectl delete pod <your-pod-name>
   ```

3. Check the pods again:

   ```bash
   kubectl get pods
   ```

What to observe: Kubernetes detects the missing pod and creates a replacement automatically.

## Exercise 2: Scale the Application

Kubernetes can increase the number of running pods when traffic grows.

1. Scale the deployment to three replicas:

   ```bash
   kubectl scale deployment nginx-test --replicas=3
   ```

2. Watch the new pods appear:

   ```bash
   kubectl get pods
   ```

What to observe: You now have three identical pods running behind the same service.

## Exercise 3: Perform a Zero-Downtime Update

Kubernetes updates pods gradually so the application stays available during a version change.

1. Update the deployment image to a different NGINX version:

   ```bash
   kubectl set image deployment/nginx-test nginx=nginx:alpine
   ```

2. Watch the rollout:

   ```bash
   kubectl rollout status deployment/nginx-test
   ```

3. Verify the updated pods:

   ```bash
   kubectl get pods
   ```

What to observe: Kubernetes replaces the old pods one at a time, which keeps the service online during the update.

## Exercise 4: Move to YAML

Imperative commands are useful for learning, but real Kubernetes workflows are usually managed with YAML manifests.

1. Export the deployment to a YAML file:

   ```bash
   kubectl get deployment nginx-test -o yaml > my-app.yaml
   ```

2. Open `my-app.yaml` in your editor.

3. Find the `spec:` section and review the values for `replicas` and `image`.

4. Change `replicas: 3` to `replicas: 5`, then save the file.

5. Apply the updated manifest:

   ```bash
   kubectl apply -f my-app.yaml
   ```

6. Confirm the new replica count:

   ```bash
   kubectl get pods
   ```

What to observe: Kubernetes updates the live deployment to match the desired state defined in the YAML file.

## Cleanup

When you are done, remove the sample deployment if you no longer need it:

```bash
kubectl delete deployment nginx-test
kubectl delete service nginx-test
```

## Summary

In this exercise, you saw four core Kubernetes concepts in action: self-healing, horizontal scaling, rolling updates, and declarative configuration with YAML. These are the building blocks for running applications reliably on k0s and other Kubernetes platforms.
