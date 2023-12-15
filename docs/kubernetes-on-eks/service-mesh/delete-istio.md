---
description: Delete Istio with our straightforward guide. Cleanly delete Istio from your system and streamline your setup.
---

# Delete Istio

Delete all the resources created by Istio since we'll be re-installing it in a way that will allow Istio to work with AWS application load balancer.

Remember, by default Istio creates an AWS classic load balancer. We can configure Istio to create application load balancer instead of classic load balancer. We'll see how in the next section.


## Uninstall Istio

```
# Uninstall istio
istioctl uninstall --purge
```

Also, make sure to delete all the ingress resources we created for prometheus, grafana, kiali and jaegar. We'll use Istio gateway this time instead of ingress.

We'll also delete book management microservices that we created. We'll recreate them using istio gateway.