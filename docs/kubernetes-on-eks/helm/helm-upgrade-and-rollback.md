---
description: Effortlessly upgrade and rollback Helm charts. Our guide provides easy steps for a smooth Kubernetes experience.
---

# Helm Upgrade and Rollback

Let's see how you can upgrade or rollback a helm chart release.


## Helm Upgrade

The `helm upgrade` command is used to upgrade an existing release to a new version.

It takes the name of the release, the name of the chart to upgrade to, and any values or options to apply during the upgrade process.

For example, assume you have a release named `myapp` and you want to upgrade it to a new version of the `myapp` chart. You can use the following command to upgrade the release:

```
helm upgrade myapp myapp-chart --set some.key=value
```

You can use `--set` multiple times to set multiple values.

You can also use a YAML file instead of `--set` argument as discussed earlier.



## Helm Upgrade Flags

During the helm upgrade, you have the option to reset values using the `--reset-values` flag or retain them with the `--reuse-values` flag.

### Reset Values

The `--reset-values` flag resets the values to default, i.e, the values provided in the `values.yaml` file of the chart.

For example, the following command will upgrade the chart by setting all values to default:

```
helm upgrade <name> <chart> --reset-values
```

### Reuse Values

The `--reuse-values` allows you to reuse the previously set values.

For example, let's say you want to upgrade the chart by setting `foo=bar`.

```
helm upgrade <name> <chart> --set foo=bar
```

The above command will upgrade the chart by setting all values to default and `foo=bar`. The previously set values will be lost.

Instead, you can use `--reuse-values` flag to reuse the previously set values and set `foo=bar`.

```
helm upgrade <name> <chart> --set foo=bar --resue-values
```


## Helm Rollback

The `helm rollback` command allows you to roll back to a previous version of a release.

For example, assume that you have a helm release named `myapp` and you want to roll back to the previous version.

First, list the release history to see the available revisions:

```
helm history myapp
```

The above command will show a list of all revisions for the `myapp` release, along with the corresponding revision number.

Next, choose the revision you want to roll back to and use the `helm rollback` command to perform the rollback:

```
helm rollback myapp <revision-number>
```

For example, if you want to roll back to revision 2 of the "my-release" release, you would use the following command:

```
helm rollback myapp 3
```

The above command will roll back the release to the state it was in at revision 3. Any changes made in subsequent revisions will be undone, and the release will be returned to the state it was in at revision 3.

You can also use the `--wait` flag with the `helm rollback` command to wait for the rollout to be complete before exiting, and the `--force` flag to force resource updates through a `delete/recreate` process rather than the default rolling upgrade process.

Note that you also need to provide `-n` or `--namespace` argument if `myapp` is not in the `default` namespace.