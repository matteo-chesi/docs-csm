# Stage 1 - Ceph image upgrade

## Procedure

1. Run `ncn-upgrade-ceph-nodes.sh` for `ncn-s001`. Follow output of the script carefully. The script will pause for manual interaction.

    ```bash
    ncn-m001# /usr/share/doc/csm/upgrade/1.2/scripts/upgrade/ncn-upgrade-ceph-nodes.sh ncn-s001
    ```

    > **NOTE:** The root password for the node may need to be reset after it is rebooted.

    **Known Issues:**
    * If the below error is observed, then re-run the same command for the node upgrade. It will pick up at that point and continue.

        ```text
        ====> REDEPLOY_CEPH ...
        /usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/ceph.pub"Number of key(s) added: 1Now try logging into the machine, with:   "ssh 'root@ncn-s003'"
        and check to make sure that only the key(s) you wanted were added.Error EINVAL: Traceback (most recent call last):
        ```

    * During the storage node rebuild, Ceph health may report `HEALTH_WARN 1 daemons have recently crashed`. This occurs occasionally as part of the shutdown process of the node
      being rebuilt. See [Dump Ceph Crash Data](../../operations/utility_storage/Dump_Ceph_Crash_Data.md).

1. **IMPORTANT:** Ensure that the Ceph cluster is healthy prior to continuing.

    If there are processes not running, then see [Utility Storage Operations](../../operations/utility_storage/Utility_Storage.md) for operational and troubleshooting procedures.

1. Repeat the previous steps for each other storage node, one at a time.

1. After `ncn-upgrade-ceph-nodes.sh` has successfully run for all storage nodes, then rescan the SSH keys on all storage nodes.

    ```bash
    ncn-m001# grep -oP "(ncn-s\w+)" /etc/hosts | sort -u | xargs -t -i ssh {} 'truncate --size=0 ~/.ssh/known_hosts'
    ncn-m001# grep -oP "(ncn-s\w+)" /etc/hosts | sort -u | xargs -t -i ssh {} 'grep -oP "(ncn-s\w+|ncn-m\w+|ncn-w\w+)" /etc/hosts | sort -u | xargs -t -i ssh-keyscan -H \{\} >> /root/.ssh/known_hosts'
    ```

1. Deploy `node-exporter` and `alertmanager`.

    **NOTE:** This procedure must run on a node running `ceph-mon`, which in most cases will be `ncn-s001`, `ncn-s002`, and `ncn-s003`. It only needs to be run once, not on every one of these nodes.

    1. Deploy `node-exporter` and `alertmanager`.

        ```bash
        ncn-s# ceph orch apply node-exporter && ceph orch apply alertmanager
        ```

        Expected output looks similar to the following:

        ```text
        Scheduled node-exporter update...
        Scheduled alertmanager update...
        ```

    1. Verify that `node-exporter` is running.

        > **IMPORTANT:** There should be one `node-exporter` container per Ceph node.

        ```bash
        ncn-s# ceph orch ps --daemon_type node-exporter
        ```

        Expected output on a system with three Ceph nodes should look similar to the following:

        ```text
        NAME                    HOST      STATUS         REFRESHED  AGE  VERSION  IMAGE NAME                                       IMAGE ID           CONTAINER ID
        node-exporter.ncn-s001  ncn-s001  running (57m)  3m ago     67m  0.18.1   docker.io/prom/node-exporter:v0.18.1             e5a616e4b9cf       3465eade21da
        node-exporter.ncn-s002  ncn-s002  running (57m)  3m ago     67m  0.18.1   registry.local/prometheus/node-exporter:v0.18.1  e5a616e4b9cf       7ed9b6cc9991
        node-exporter.ncn-s003  ncn-s003  running (57m)  3m ago     67m  0.18.1   registry.local/prometheus/node-exporter:v0.18.1  e5a616e4b9cf       1078d9e555e4
        ```

    1. Verify that `alertmanager` is running.

        > **IMPORTANT:** There should be a single `alertmanager` container for the cluster.

        ```bash
        ncn-s# ceph orch ps --daemon_type alertmanager
        ```

        Expected output looks similar to the following:

        ```text
        NAME                   HOST      STATUS         REFRESHED  AGE  VERSION  IMAGE NAME                                      IMAGE ID           CONTAINER ID
        alertmanager.ncn-s001  ncn-s001  running (66m)  3m ago     68m  0.20.0   registry.local/prometheus/alertmanager:v0.20.0  0881eb8f169f       775aa53f938f
        ```

1. Update BSS to ensure that the Ceph images are loaded if a node is rebuilt.

    ```bash
    ncn-m001# . /usr/share/doc/csm/upgrade/1.2/scripts/ceph/lib/update_bss_metadata.sh
    ncn-m001# update_bss_storage
    ```

## Stage completed

All the Ceph nodes have been rebooted into the new image.

This stage is completed. Continue to [Stage 2](Stage_2.md).
