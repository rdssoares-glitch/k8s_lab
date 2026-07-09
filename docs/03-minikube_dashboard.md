# 🚀 Enable the Minikube Dashboard

This guide explains how to enable the **Kubernetes Dashboard** in Minikube, install the **Metrics Server**, verify that the Dashboard is running, and access it from a web browser.

---

# 1. Start the Kubernetes Dashboard

Run the following command:

```bash
minikube dashboard
```

You may see output similar to the following:

```text
🤔  Verifying dashboard health ...
🚀  Launching proxy ...
🤔  Verifying proxy health ...
🎉  Opening http://127.0.0.1:33903/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/ in your default browser...

xdg-open: no method available for opening ...
```

> **Note**
>
> When running Minikube inside **WSL2**, this message is expected because Linux cannot automatically launch a Windows web browser. The Dashboard is still running successfully.

---

# 2. Enable the Metrics Server

The Metrics Server provides CPU and memory metrics for nodes and Pods, allowing the Dashboard to display resource utilization.

Enable the addon:

```bash
minikube addons enable metrics-server
```

Expected output:

```text
💡  metrics-server is an addon maintained by Kubernetes.
▪ Using image registry.k8s.io/metrics-server/metrics-server:v0.8.0
🌟  The 'metrics-server' addon is enabled
```

---

# 3. Verify that the Dashboard Pods are running

Check the Dashboard namespace:

```bash
kubectl get pods -n kubernetes-dashboard
```

Example output:

```text
NAME                                         READY   STATUS    RESTARTS   AGE
dashboard-metrics-scraper-77bf4d6c4c-kztx5   1/1     Running   0          3m8s
kubernetes-dashboard-855c9754f9-sp5bw        1/1     Running   0          3m8s
```

Both Pods should be in the **Running** state.

---

# 4. Retrieve the Dashboard URL

Instead of automatically opening a browser, print the Dashboard URL:

```bash
minikube dashboard --url
```

Example output:

```text
🤔  Verifying dashboard health ...
🚀  Launching proxy ...
🤔  Verifying proxy health ...

http://127.0.0.1:45185/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/
```

---

# 5. Open the Dashboard

Copy the URL displayed by the previous command.

Open your preferred web browser (Google Chrome, Microsoft Edge, Firefox, etc.) and navigate to the Dashboard URL.

Example:

```text
http://127.0.0.1:45185/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/
```

If everything is configured correctly, the Kubernetes Dashboard will load successfully.

---

# ✅ Verification

The Dashboard is ready to use when:

- The **metrics-server** addon is enabled.
- Both Dashboard Pods are in the **Running** state.
- The Dashboard URL is accessible from your web browser.
- You can browse Kubernetes resources such as Nodes, Pods, Deployments, Services, ConfigMaps, and Namespaces.
