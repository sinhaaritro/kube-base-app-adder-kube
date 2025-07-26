# kube-base-app-adder-kube
A basic app that will be containerized later for GitOps Kubernetes

Kebernetics for https://github.com/sinhaaritro/kube-base-app-adder


## **Secretes Used** 
1. **GH_PAT**: This is in Repo. But will need the code from `GHA_IMAGE_PUSHER` secret at the user level
2. **k8s-image-pull-token**: This is at the user level


## **Steps**

#### **Phase 1: First-Time Setup (Only do this once or if your environment is new)**

**Step 1.1: Create the GitHub Personal Access Token (PAT)**

This token is a password your Kubernetes cluster will use to download your private container image from the GitHub Container Registry (GHCR).

1.  **On the GitHub Website:**
    *   Go to your profile **Settings** > **Developer settings** > **Personal access tokens** > **Tokens (classic)**.
    *   Click **Generate new token (classic)**.
    *   Give it a descriptive **Note** (e.g., `K8S_PULL_SECRET`).
    *   Set an **Expiration** date (e.g., 30 or 90 days).
    *   Select the required scopes: **`repo`** and **`write:packages`**.
    *   Click **Generate token**.
2.  **CRITICAL:** Copy the new token (it starts with `ghp_...`) and save it somewhere temporarily, like a notepad. **You will not be shown this token again.**

**Step 1.2 (Optional): Create the `kind` Cluster**

This step creates the Kubernetes environment itself. Skip this if your cluster is already running.

1.  **Check for an existing cluster:**
    ```bash
    kind get clusters
    ```
    *   If you see your cluster name listed, you can skip to Phase 2.
    *   If it says `No kind clusters found`, proceed to the next step.

2.  **Create a simple cluster:** This command creates a default cluster and automatically configures `kubectl` to connect to it.
    ```bash
    KIND_EXPERIMENTAL_PROVIDER=podman kind create cluster
    ```

---

#### **Phase 2: Deploying the Adder Application**

**Step 2.1: Clone the Kubernetes Deployment Repository**

Get the Kubernetes configuration files for the adder app onto your VM.

1.  Navigate to your home directory:
    ```bash
    cd ~
    ```
2.  Clone the adder app's specific deployment repository:
    ```bash
    git clone https://github.com/sinhaaritro/kube-base-app-adder-kube.git
    ```3.  Navigate into the newly cloned directory:
    ```bash
    cd kube-base-app-adder-kube
    ```

**Step 2.2: Create the Kubernetes Image Pull Secret**

Give your Kubernetes cluster the PAT you created so it can log in to GHCR. **You only need to do this once per cluster.** If you already did this for the multiplier app, you can skip this step.

1.  Run the `kubectl create secret` command.
    *   Replace `YOUR_PERSONAL_ACCESS_TOKEN` with the actual token you copied from GitHub.

    ```bash
    kubectl create secret docker-registry ghcr-creds \
      --docker-server=ghcr.io \
      --docker-username=sinhaaritro \
      --docker-password=YOUR_PERSONAL_ACCESS_TOKEN
    ```
    You should see the output `secret/ghcr-creds created` (or `...already exists`).

**Step 2.3: Verify the `deployment.yaml` Uses the Secret**

Ensure your deployment manifest is configured to use the secret. The file you cloned should already contain this, but it's good practice to check.

1.  View the contents of `deployment.yaml`:
    ```bash
    cat deployment.yaml
    ```
2.  Confirm that the `imagePullSecrets` block is present under the `spec:` section, like so:
    ```yaml
    spec:
      imagePullSecrets:
      - name: ghcr-creds
      replicas: 1
    ```

**Step 2.4: Apply the Kubernetes Configuration**

Deploy the adder application to your cluster.

1.  From inside the `kube-base-app-adder-kube` directory, run the apply command:
    ```bash
    kubectl apply -f .
    ```
    You will see output like `deployment.apps/adder-app-deployment created` and `service/adder-app-service created`.

**Step 2.5: Verify the Deployment**

Check the status of your newly created pod.

1.  Watch the pod status. It should eventually transition to `Running`.
    ```bash
    kubectl get pods
    ```
2.  **If it gets stuck on `ErrImagePull` or `ImagePullBackOff`:**
    *   Make sure you have created the `v1.0.0` release/tag for your `kube-base-app-adder-container` repository and that the Action has finished successfully.
    *   If the image exists and the pod is just stuck, you can force a retry by deleting the pod: `kubectl delete pod <pod-name>`.

3.  **Desired Output:**
    ```
    NAME                                         READY   STATUS    RESTARTS   AGE
    adder-app-deployment-xxxxxxxxxx-yyyyy        1/1     Running   0          30s
    multiplier-app-deployment-xxxxxxxxxx-zzzzz   1/1     Running   0          ...
    ```

---

#### **Phase 3: Accessing Your Running Adder App**

Use `port-forward` to create a direct tunnel to your running application pod.

1.  **Start the port forward:** This command will take over your current terminal. Choose a free local port, like `8888`.
    ```bash
    # This forwards port 8888 on your VM to port 8000 inside the adder pod.
    kubectl port-forward deployment/adder-app-deployment 8888:8000
    ```
2.  **Access the application:** Open a web browser on your local machine and go to:
    **`http://localhost:8888`**

You have now successfully deployed your second application, the adder app, to Kubernetes.