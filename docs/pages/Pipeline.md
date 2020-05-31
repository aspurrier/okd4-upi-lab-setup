# WIP

Create a maven group in Nexus: homelab-central

Expose Registry:

    oc patch configs.imageregistry.operator.openshift.io/cluster --patch '{"spec":{"defaultRoute":true}}' --type=merge
    podman login -u $(oc whoami) -p $(oc whoami -t) --tls-verify=false $(oc get route default-route -n openshift-image-registry --template='{{ .spec.host }}')

    docker login -u $(oc whoami) -p $(oc whoami -t) $(oc get route default-route -n openshift-image-registry --template='{{ .spec.host }}')

Tekton:

    tkn clustertask ls

    IMAGE_REGISTRY=$(oc get route default-route -n openshift-image-registry --template='{{ .spec.host }}')
    podman login -u $(oc whoami) -p $(oc whoami -t) --tls-verify=false ${IMAGE_REGISTRY}
    podman pull quay.io/openshift/origin-cli:4.4.0
    podman tag quay.io/openshift/origin-cli:4.4.0 ${IMAGE_REGISTRY}/openshift/origin-cli:4.4.0
    podman push ${IMAGE_REGISTRY}/openshift/origin-cli:4.4.0 --tls-verify=false

    docker pull quay.io/buildah/stable
    docker tag quay.io/buildah/stable:latest ${IMAGE_REGISTRY}/openshift/buildah:stable
    docker push ${IMAGE_REGISTRY}/openshift/buildah:stable

    docker pull docker.io/maven:3.6.3-jdk-8-slim
    docker tag docker.io/library/maven:3.6.3-jdk-8-slim ${IMAGE_REGISTRY}/openshift/maven:3.6.3-jdk-8-slim
    docker push ${IMAGE_REGISTRY}/openshift/maven:3.6.3-jdk-8-slim

    docker pull quay.io/openshift/origin-cli:4.4.0
    docker tag quay.io/openshift/origin-cli:4.4.0 ${IMAGE_REGISTRY}/openshift/origin-cli:4.4.0
    docker push ${IMAGE_REGISTRY}/openshift/origin-cli:4.4.0

    docker build -t ${IMAGE_REGISTRY}/openshift/jdk-ubi-minimal:8.1 .
    docker push ${IMAGE_REGISTRY}/openshift/jdk-ubi-minimal:8.1

Create a Secret for your git repo:

    apiVersion: v1
    kind: Secret
    metadata:
      name: git-secret
      annotations:
        tekton.dev/git-0: bitbucket.org
    type: kubernetes.io/ssh-auth
    data:
      ssh-privatekey: Private Key here - Generated by: "cat id_rsa | base64 -w 0" where id_rsa is the key used to grant authority to the bitbucket.org project
      known_hosts: Known Hosts here - Generated by: ssh-keyscan bitbucket.org | base64 -w 0


    oc apply -f git-secret.yml
    oc patch sa pipeline --type merge --patch '{"secrets":[{"name":"git-secret"}]}'