# Kubernetes helpers with `fzf`

Running Kubernetes clusters often involves memorizing quite a few `kubctl` subcommands, options and pipes. To make this easier, the fuzzy finder tool [`fzf`](https://junegunn.github.io/fzf/) can be used to interactively switch between self-defined shortcuts. Additionally, it facilitates searching.

Firstly, it is necessary to define a prefix that will be used for all functions.

```bash
export KUBE_HELPERS_PREFIX='kh'
```

Then we define a parent function to toggle between the prefixed function definitions. The bash expansion `$@` is used to pass the arguments to the child call.

```bash
${KUBE_HELPERS_PREFIX} () {
    local selected=$(compgen -A function | grep "^${KUBE_HELPERS_PREFIX}_" | fzf)
    $selected $@
}
```
Now we can define some shortcuts, e.g. one to return all pod IPs in the cluster.

```bash
${KUBE_HELPERS_PREFIX}_pod_ips() {
    kubectl get po -A -o json | jq -r '.items[] | [ .status.podIP, .metadata.namespace, .metadata.name ] | @tsv ' | column -t
}
```

Or a interactive way to open a shell session or to run a self-defined command in a pod.

```bash
${KUBE_HELPERS_PREFIX}_pod_shell() {
    local pod=$(kubectl get pods --no-headers -o custom-columns=":metadata.name" | fzf)
    kubectl exec -it ${pod} -- "$@" || kubectl exec -it ${pod} -- /bin/bash || kubectl exec -it ${pod} -- /bin/sh
}
```

Running `kh` now will prompt us for our defined functions.
