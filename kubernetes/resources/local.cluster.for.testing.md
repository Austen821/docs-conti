# local cluster for testing

## main usage

* create a local k8s cluster for testing

## conceptions

* none

## purpose

* create a kubernetes cluster by kind
* setup docker registry

## pre-requirement

* [a k8s cluster created by kind](../create.local.cluster.with.kind.md) have been read and practised
* [download kubernetes binary tools](download.kubernetes.binary.tools.md)
   + kind
   + kubectl
   + helm
* we recommend to use [qemu machine](../../linux/qemu/README.md)

## Do it

1. optional, [create centos 8 with qemu](../../linux/qemu/create.centos.8.with.qemu.md)
    * create centos8-qemu
      * ```shell
        qemu-system-x86_64 \
            -accel kvm \
            -smp cpus=3 \
            -m 5G \
            -drive file=$(pwd)/centos.8.qcow2,if=virtio,index=0,media=disk,format=qcow2 \
            -rtc base=localtime \
            -pidfile $(pwd)/centos.8.qcow2.pid \
            -display none \
            -nic user,hostfwd=tcp::1022-:22 \
            -daemonize
        ```
    * login with ssh
        + ```shell
          ssh -o "UserKnownHostsFile /dev/null" -p 1022 root@localhost
          ```
        + default password is `123456`
    * optional, replace yum repositories with [aliyun-centos-8](aliyun-centos-8.repo.md)
    * install docker
        + ```shell
          dnf -y install tar yum-utils device-mapper-persistent-data lvm2 docker-ce \
              && systemctl enable docker \
              && systemctl start docker
          ```
2. [download kubernetes binary tools](download.kubernetes.binary.tools.md)
    * link tools to `./bin`
    * Set `kind` `kubectl` `helm` permissions to 744
3. create cluster with a docker registry
    * prepare [kind-with-registry.sh](../basic/resources/kind-with-registry.sh.md)
        + `kind.cluster.yaml` will be created by script.
    * prepare images
        + ```shell
          DOCKER_IMAGE_PATH=/root/docker-images && mkdir -p ${DOCKER_IMAGE_PATH}
          BASE_URL="https://aconti.oss-cn-hangzhou.aliyuncs.com/docker-images"
          LOCAL_IMAGE="localhost:5000"
          for IMAGE in "docker.io/kindest/node:v1.22.1" \
              "docker.io/registry:2"
          do
              IMAGE_FILE=$(echo ${IMAGE} | sed "s/\//_/g" | sed "s/\:/_/g").dim
              LOCAL_IMAGE_FIEL=${DOCKER_IMAGE_PATH}/${IMAGE_FILE}
              if [ ! -f ${LOCAL_IMAGE_FIEL} ]; then
                  curl -o ${IMAGE_FILE} -L ${BASE_URL}/${IMAGE_FILE} \
                      && mv ${IMAGE_FILE} ${LOCAL_IMAGE_FIEL} \
                      || rm -rf ${IMAGE_FILE}
              fi
              docker image load -i ${LOCAL_IMAGE_FIEL}
              docker image inspect ${IMAGE} || docker pull ${IMAGE}
              docker image tag ${IMAGE} ${LOCAL_IMAGE}/${IMAGE}
              docker push ${LOCAL_IMAGE}/${IMAGE}
          done
          ```
    * create cluster
        + ```shell
          bash kind-with-registry.sh kind kubectl
          ```
    * checking
        + ```shell
          kubectl get pod
          ```