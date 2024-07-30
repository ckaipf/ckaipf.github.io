# Sniff Kubernetes pods

The next example shows how network debugging in Kubernetes can be simplified. Analyzing traffic in a pod can quickly become complex. TLS and mTLS, for example, can be managed by a backend or by a service mesh. The following function can be used to get a quick actual state. It uses ephemeral containers and `tcpdump` to get a network dump. The dump is then copied to the host to be analyzed with e.g. Wireshark.

```bash
${KUBE_HELPERS_PREFIX}_sniff_pod() {
    local pod=$(kubectl get pods --no-headers -o custom-columns=":metadata.name" | fzf)
    local timestamp=$(date +%s)
    local target_local_path=${1:-~}
    local max_time=${2:-30}
    local dump_name=$pod-$timestamp.pcap
    
    kubectl debug $pod -it --attach=false -c tcpdump-$timestamp --image=nicolaka/netshoot -- timeout $max_time tcpdump -i any -vv -s 65535 -w $dump_name
    kubectl wait --for='jsonpath={.status.ephemeralContainerStatuses[?(@.name=="tcpdump-'$timestamp'")].state.running}' pod/$pod  

    while true; do
        is_running=$(kubectl get pod $pod -o jsonpath='{.status.ephemeralContainerStatuses[?(@.name=="tcpdump-'$timestamp'")].state.running}')
        if [ -z "${is_running}" ]; then break; fi
        kubectl cp -c tcpdump-$timestamp $pod:$dump_name $target_local_path/$dump_name
    done

    kubectl wait --for='jsonpath={.status.ephemeralContainerStatuses[?(@.name=="tcpdump-'$timestamp'")].state.terminated}' pod/$pod  
}
```

The function can be invoked by either running `kh ~ 20` or `kh_sniff_pod ~ 20`.
