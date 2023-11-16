---
description: Deploy Vertical Pod Autoscaler to your EKS cluster in a snap! Our guide makes it easy. Optimize your apps with no fuss. Get started now!
---

# Deploy Vertical Pod Autoscaler in EKS

Let's deploy the Vertical Pod Autoscaler (VPA) in our EKS cluster.

1. Clone the [kubernetes/autoscaler]{:target="_blank"} GitHub repository:

    ```
    git clone https://github.com/kubernetes/autoscaler.git
    ```


2. Change to the `vertical-pod-autoscaler` directory:

    ```
    cd autoscaler/vertical-pod-autoscaler/
    ```


3. (Optional) If you have already deployed another version of the Vertical Pod Autoscaler, remove it with the following command:

    ```
    ./hack/vpa-down.sh
    ```


4. Deploy the Vertical Pod Autoscaler (VPA) to your cluster with the following command: 

    ```
    ./hack/vpa-up.sh
    ```


5. Verify that the Vertical Pod Autoscaler pods have been created successfully:

    ```
    kubectl get pods -n kube-system | grep vpa
    ```

    The output will look somthing like this:

    ```yaml
    vpa-admission-controller-bb59bb7c7-g7vqr        1/1     Running   0              8s
    vpa-recommender-5555d76bfd-9rzz7                1/1     Running   0              11s
    vpa-updater-8b7f687dc-pnqkh                     1/1     Running   0              13s
    ```


!!! quote "References:"
    !!! quote ""
        * [Deploy Vertical Pod Autoscaler]{:target="_blank"}
        * [kubernetes/autoscaler GitHub Repo]{:target="_blank"}

<!-- Hyperlinks -->
[Deploy Vertical Pod Autoscaler]: https://docs.aws.amazon.com/eks/latest/userguide/vertical-pod-autoscaler.html
[kubernetes/autoscaler GitHub Repo]: https://github.com/kubernetes/autoscaler/tree/master
[kubernetes/autoscaler]: https://github.com/kubernetes/autoscaler/tree/master