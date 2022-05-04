# Troubleshoot Loss of Console Connections and Logs on Gigabyte Nodes

Gigabyte console log information will no longer be collected, and if attempting to initiate a console session through the `cray-conman` pod, there will be an error reported. This error will occur every time the node is rebooted unless this workaround is applied.

### Prerequisites

Console log information is no longer being collected for Gigabyte nodes or ConMan is reporting an error.

### Procedure

1.  Use `ipmitool` to deactivate the current console connection.

    (`ncn-m001#`)
    ```bash
    export USERNAME=root
    export IPMI_PASSWORD=changeme
    ipmitool -H xname -U $USERNAME -E sol deactivate
    ```

2.  Retrieve the `cray-conman` pod ID.

    (`ncn-m001#`)
    ```bash
    CONPOD=$(kubectl get pods -n services \
    -o wide|grep cray-conman|awk '{print $1}')
    echo $CONPOD
    ```

3.  Log on to the pod.

    (`ncn-m001#`)
    ```bash
    kubectl exec -it -n services $CONPOD /bin/bash
    ```

4.  Initiate a console session to reconnect.

    ```bash
    [root@cray-conman-POD_ID app]# conman -j XNAME
    ```

