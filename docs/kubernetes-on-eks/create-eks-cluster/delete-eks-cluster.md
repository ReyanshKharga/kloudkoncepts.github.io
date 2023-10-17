---
description: Effortlessly delete an Amazon EKS cluster with our step-by-step guide using eksctl. Seamlessly manage cluster removal for a hassle-free experience.
---


# Delete EKS Cluster

We won't delete the cluster we created since we will be using it for further demos and tutorials. But if you need to delete the cluster, you can follow the instructions below.


## Delete Cluster

You can delete the EKS cluster using the following commands:

```
# Command template
eksctl delete cluster -f <cluster-config-file>
{OR}
eksctl delete cluster --name <cluster-name> --region <region_name>

# Actual command
eksctl delete cluster -f cluster.yml
{OR}
eksctl delete cluster --name my-cluster --region ap-south-1
```

!!! note
    To delete an EKS cluster, only two parameters are required `name` and `region`. If you don't provide the `region` parameter, it will default to the region you set while configuring AWS CLI.

When you use the configuration file to delete the cluster, `eksctl` basically extracts the `name` and `region` parameters of the cluster from configuration file and then uses them to delete the cluster.


## Troubleshooting

If for reason you get an error that says `Error: failed to delete all resources`, go to AWS Cloudformation and check the Stack info. You will find the reason as to why the deletion failed.

You can then retry deleting the resources once you have identified and fixed the issue.