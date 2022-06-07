# Pre-Boot Configuration of NCN Images

**NOTE:** Some of the documentation linked from this page mentions use of the BOS service. The use of BOS
is only relevant for booting compute nodes and can be ignored when working with NCN images.

This document describes the configuration of a Kubernetes NCN image. The same steps are relevant for modifying
a Ceph image.

1. Locate the NCN image to be modified.

    This example assumes you want to modify the image that is currently in use by NCNs. However, the steps
    are the same for any NCN SquashFS image.

    ```console
    ncn# cray artifacts get ncn-images k8s/<version>/filesystem.squashfs ./<version>-filesystem.squashfs

    ncn# cray artifacts get ncn-images k8s/<version>/kernel ./<version>-kernel

    ncn# cray artifacts get ncn-images k8s/<version>/initrd ./<version>-initrd

    ncn# export IMS_ROOTFS_FILENAME=<version>-filesystem.squashfs

    ncn# export IMS_KERNEL_FILENAME=<version>-kernel

    ncn# export IMS_INITRD_FILENAME=<version>-initrd
    ```

1. Import the external image to IMS.

    See [Import External Image to IMS](../image_management/Import_External_Image_to_IMS.md).

    This document instructs the administrator to set several environment variables, including the three set in
    the previous step.

1. Create and populate a VCS configuration repository.

    See [Create and Populate a VCS Configuration Repository](Create_and_Populate_a_VCS_Configuration_Repository.md).

   **NOTE:** If the image modification is a kernel-level change, a new `initrd` can be created by invoking
   the following script: `/srv/cray/scripts/common/create-ims-initrd.sh`. This script is embedded in the
   NCN SquashFS. After the script completes, a new `initrd` file will be available in the `/boot/` directory. CFS will
   automatically make this `initrd` available at the end of the CFS session.

   **WARNING:** If the above script is not run, **DO NOT** download the `initrd` or kernel after the
   CFS session completes (see below). CFS will collect `/boot/initrd` and `/boot/vmlinuz` from the modified
   SquashFS. If the `create-ims-initrd.sh` script is not run, then these artifacts will be incorrect and
   will not be able to boot an NCN.

   When not modifying anything at the kernel-level, there is no need to create a new `initrd`. In
   that case, simply use the existing `initrd`.

1. Create a CFS configuration.

    See [Create a CFS Configuration](Create_a_CFS_Configuration.md).

1. Create an image customization CFS session.

    See [Create an Image Customization CFS Session](Create_an_Image_Customization_CFS_Session.md).

1. Download the resultant NCN artifacts.

    > **NOTE:** `$IMS_RESULTANT_IMAGE_ID` is the IMS image ID returned in the output of the last command
    > in the previous step:
    >
    > ```console
    > ncn# cray cfs sessions describe example --format json | jq .status.artifacts
    > ```

    ```console
    ncn# cray artifacts get boot-images $IMS_RESULTANT_IMAGE_ID/rootfs kubernetes-<version>-1.squashfs

    ncn# cray artifacts get boot-images $IMS_RESULTANT_IMAGE_ID/initrd initrd.img-<version>-1.xz

    ncn# cray artifacts get boot-images $IMS_RESULTANT_IMAGE_ID/kernel 5.3.18-150300.59.43-default-<version>-1.kernel
    ```

1. Upload NCN boot artifacts into S3.

    This steps requires that the `docs-csm` RPM is installed. See [Check for Latest Documentation](../../update_product_stream/index.md#documentation).

    ```console
    ncn# /usr/share/doc/csm/scripts/ceph-upload-file-public-read.py --bucket-name ncn-images --key-name 'k8s/<new-version>/filesystem.squashfs' --file-name kubernetes-<version>-1.squashfs

    ncn# /usr/share/doc/csm/scripts/ceph-upload-file-public-read.py --bucket-name ncn-images --key-name 'k8s/<new-version>/initrd' --file-name initrd.img-<version>-1.xz

    ncn# /usr/share/doc/csm/scripts/ceph-upload-file-public-read.py --bucket-name ncn-images --key-name 'k8s/<new-version>/kernel' --file-name <version>-1.kernel
    ```

1. Update NCN boot parameters.

    1. Set a variable with the component name (xname) of the node of interest.

        ```console
        ncn# XNAME=<your-xname>
        ```

    1. Get the existing `metal.server` setting for the node.

        ```console
        ncn# METAL_SERVER=$(cray bss bootparameters list --hosts $XNAME --format json | jq '.[] |."params"' \
             | awk -F 'metal.server=' '{print $2}' \
             | awk -F ' ' '{print $1}')
        ```

    1. Verify that the variable was set correctly.

        ```console
        ncn# echo $METAL_SERVER
        ````

    1. Update the kernel, `initrd`, and `metal.server` to point to the new artifacts.

        Note: Multiple xnames can be updated at the same time, if desired.

        1. Set variables.

            ```console
            ncn# S3_ARTIFACT_PATH=ncn-images/k8s/<new-version>
            ncn# NEW_METAL_SERVER=http://rgw-vip.nmn/$S3_ARTIFACT_PATH

            ncn# PARAMS=$(cray bss bootparameters list --hosts $XNAME --format json | jq '.[] |."params"' | \
                 sed "/metal.server/ s|$METAL_SERVER|$NEW_METAL_SERVER|")
            ```

        1. Verify that the value of `$NEW_METAL_SERVER` was set correctly.

            ```console
            ncn# echo $PARAMS
            ```

        1. Update BSS.

            In the following invocation, `$XNAME` may be one or more xnames.

            ```console
            ncn# cray bss bootparameters update --hosts $XNAME   \
                 --kernel "s3://$S3_ARTIFACT_PATH/kernel" \
                 --initrd "s3://$S3_ARTIFACT_PATH/initrd" \
                 --params "$PARAMS"
            ```

1. Reboot the NCN.

    See [Reboot NCNs](../node_management/Reboot_NCNs.md).
