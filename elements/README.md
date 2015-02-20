    git clone https://github.com/yuanying/heat-kubernetes.git
    cd heat-kubernetes
    git checkout -b ironic origin/ironic
    cd ..
    git clone https://git.openstack.org/openstack/diskimage-builder.git
    git clone https://git.openstack.org/openstack/tripleo-image-elements.git
    git clone https://git.openstack.org/openstack/heat-templates.git
    git clone https://git.openstack.org/openstack/dib-utils.git
    export PATH="${PWD}/dib-utils/bin:$PATH"
    export ELEMENTS_PATH=tripleo-image-elements/elements:heat-templates/hot/software-config/elements:heat-kubernetes/elements
    export DIB_RELEASE=21
    diskimage-builder/bin/disk-image-create baremetal \
      fedora selinux-permissive \
      os-collect-config \
      os-refresh-config \
      os-apply-config \
      heat-config-ansible \
      heat-config-cfn-init \
      heat-config-docker \
      heat-config-puppet \
      heat-config-salt \
      heat-config-script \
      kubernetes \
      -o fedora-21-kubernetes.qcow2

    KERNEL_ID=`glance image-create --name fedora-k8s-kernel --is-public True --disk-format=aki --container-format=aki --file=fedora-21-kubernetes.vmlinuz | grep id | tr -d '| ' | cut --bytes=3-57`
    echo Registered fedora-kernel\($KERNEL_ID\).
    RAMDISK_ID=`glance image-create --name fedora-k8s-ramdisk --is-public True --disk-format=ari --container-format=ari --file=fedora-21-kubernetes.initrd | grep id |  tr -d '| ' | cut --bytes=3-57`
    echo Registered fedora-ramdisk\($RAMDISK_ID\).
    BASE_ID=`glance image-create --name fedora-k8s --is-public True --disk-format=qcow2 --container-format=bare --property kernel_id=$KERNEL_ID --property ramdisk_id=$RAMDISK_ID --file=fedora-21-kubernetes.qcow2 | grep -v kernel | grep -v ramdisk | grep id | tr -d '| ' | cut --bytes=3-57`
    echo Registered fedora-base\($BASE_ID\).
