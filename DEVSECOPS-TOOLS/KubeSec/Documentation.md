# KUBESEC

Reference: [https://kubesec.io/](https://kubesec.io/)

_Kubesec is an open-source Kubernetes security scanner and analysis tool. The way it works, it accepts a single Kubernetes manifests file and provides a severity score for each found vulnerability._

_There is a common phrase in the DevSecOps world-shifting security to the left, which means catching any security at the earlier stage of the development cycle. So if we want to catch security-related issues or enforce standards before it���s deployed to the cluster or even before the user runs the kubectl command right after you developed your Kubernetes yaml files. This can be done with the help of Kubesec, which is a static analysis tool. Using Kubesec, you can review the resource yaml file and enforce the policies by checking against the certain rule during the earlier stage of the development cycle._

_Kubesec analyzes your resource yaml file and returns the score(higher score is better)and details about the critical issue found in it._

**_Installation_**

*   _Kubesec can be installed as a binary package, container image, Admission controller or even as a kubectl plugin._

```
wget https://github.com/controlplaneio/kubesec/releases/download/v2.11.4/kubesec_linux_amd64.tar.gztar -xvf kubesec_linux_amd64.tar.gz
```


_To download kubesec for other platforms, follow these links_ [_https://github.com/controlplaneio/kubesec#download-kubesec_](https://github.com/controlplaneio/kubesec#download-kubesec)

**_Using Kubesec_**

*   _If you pass the below pod.yaml to the kubesec_

```
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  containers:
  - name: sec-ctx-demo
    image: busybox
    command: [ "sh", "-c", "sleep 1h" ]
    securityContext:
      privileged: true
```


*   _As you can see in the output, Kubesec assigned the score of minus 30 with the reason “Privileged containers can allow almost completely unrestricted host access”._

```
./kubesec scan pod.yaml 
[
  {
    "object": "Pod/security-context-demo.default",
    "valid": true,
    "fileName": "pod.yaml",
    "message": "Failed with a score of -30 points",
    "score": -30,
    "scoring": {
      "critical": [
        {
          "id": "Privileged",
          "selector": "containers[] .securityContext .privileged == true",
          "reason": "Privileged containers can allow almost completely unrestricted host access",
          "points": -30
        }
      ],
      "advise": [
        {
          "id": "ApparmorAny",
          "selector": ".metadata .annotations .\"container.apparmor.security.beta.kubernetes.io/nginx\"",
          "reason": "Well defined AppArmor policies may provide greater protection from unknown threats. WARNING: NOT PRODUCTION READY",
          "points": 3
        },
        {
          "id": "ServiceAccountName",
          "selector": ".spec .serviceAccountName",
          "reason": "Service accounts restrict Kubernetes API access and should be configured with least privilege",
          "points": 3
        },
        {
          "id": "SeccompAny",
          "selector": ".metadata .annotations .\"container.seccomp.security.alpha.kubernetes.io/pod\"",
          "reason": "Seccomp profiles set minimum privilege and secure against unknown threats",
          "points": 1
        },
        {
          "id": "LimitsCPU",
          "selector": "containers[] .resources .limits .cpu",
          "reason": "Enforcing CPU limits prevents DOS via resource exhaustion",
          "points": 1
        },
        {
          "id": "LimitsMemory",
          "selector": "containers[] .resources .limits .memory",
          "reason": "Enforcing memory limits prevents DOS via resource exhaustion",
          "points": 1
        },
        {
          "id": "RequestsCPU",
          "selector": "containers[] .resources .requests .cpu",
          "reason": "Enforcing CPU requests aids a fair balancing of resources across the cluster",
          "points": 1
        },
        {
          "id": "RequestsMemory",
          "selector": "containers[] .resources .requests .memory",
          "reason": "Enforcing memory requests aids a fair balancing of resources across the cluster",
          "points": 1
        },
        {
          "id": "CapDropAny",
          "selector": "containers[] .securityContext .capabilities .drop",
          "reason": "Reducing kernel capabilities available to a container limits its attack surface",
          "points": 1
        },
        {
          "id": "CapDropAll",
          "selector": "containers[] .securityContext .capabilities .drop | index(\"ALL\")",
          "reason": "Drop all capabilities and add only those required to reduce syscall attack surface",
          "points": 1
        },
        {
          "id": "ReadOnlyRootFilesystem",
          "selector": "containers[] .securityContext .readOnlyRootFilesystem == true",
          "reason": "An immutable root filesystem can prevent malicious binaries being added to PATH and increase attack cost",
          "points": 1
        },
        {
          "id": "RunAsNonRoot",
          "selector": "containers[] .securityContext .runAsNonRoot == true",
          "reason": "Force the running image to run as a non-root user to ensure least privilege",
          "points": 1
        },
        {
          "id": "RunAsUser",
          "selector": "containers[] .securityContext .runAsUser -gt 10000",
          "reason": "Run as a high-UID user to avoid conflicts with the host's user table",
          "points": 1
        }
      ]
    }
  }
]
```


*   _Now fix the yaml file by removing the security context block or set privileged: false_

```
apiVersion: v1
kind: Pod
metadata:
    name:security-context-demo
    spec:
        containers:
        - name: sec-ctx-demo
          image: busybox
          command: [ "sh", "-c", "sleep 1h" ]
```


*   _Run the kubesec scan again. As you can see this time score is 0 and the message of passed._

```
./kubesec scan pod.yaml
[
    {
        "object": "Pod/security-context-demo.default",
        "valid": true,
        "fileName": "pod.yaml",
        "message": "Passed with a score of 0 points",
        "score": 0,
        "scoring": {
        "advise": [
            {
                "id": "ApparmorAny",
                "selector": ".metadata .annotations .\"container.apparmor.security.beta.kubernetes.io/nginx\"",
                "reason": "Well defined AppArmor policies may provide greater protection from unknown threats. WARNING: NOT PRODUCTION READY",
                "points": 3
            },
            {
                "id": "ServiceAccountName",
                "selector": ".spec .serviceAccountName",
                "reason": "Service accounts restrict Kubernetes API access and should be      configured with least privilege",
                "points": 3
            },
            {
                "id": "SeccompAny",
                "selector": ".metadata .annotations .\"container.seccomp.security.alpha.kubernetes.io/pod\"",
                "reason": "Seccomp profiles set minimum privilege and secure against unknown threats",
                "points": 1
            },
            {
                "id": "LimitsCPU",
                "selector": "containers[] .resources .limits .cpu",
                "reason": "Enforcing CPU limits prevents DOS via resource exhaustion",
                "points": 1
            },
            {
                "id": "LimitsMemory",
                "selector": 
                "containers[] .resources .limits .memory",
                "reason": "Enforcing memory limits prevents DOS via resource exhaustion",
                "points": 1
            },
            {
                "id": "RequestsCPU",
                "selector": "containers[] .resources .requests .cpu",
                "reason": "Enforcing CPU requests aids a fair balancing of resources across the cluster",
                "points": 1
             },
             {
                "id": "RequestsMemory",
                "selector": "containers[] .resources .requests .memory",
                "reason": "Enforcing memory requests aids a fair balancing of resources across the cluster",
                "points": 1
             },
             {
                "id": "CapDropAny",
                "selector": "containers[] .securityContext .capabilities .drop",
                "reason": "Reducing kernel capabilities available to a container limits its attack surface",
                "points": 1
              },
              {
                "id": "CapDropAll",
                "selector": "containers[] .securityContext .capabilities .drop | index(\"ALL\")",
                "reason": "Drop all capabilities and add only those required to reduce syscall attack surface",
                "points": 1
              },
              {
                "id": "ReadOnlyRootFilesystem",
                "selector": "containers[] .securityContext .readOnlyRootFilesystem == true",
                "reason": "An immutable root filesystem can prevent malicious binaries being added to PATH and increase attack cost",
                "points": 1
              },
              {
                "id": "RunAsNonRoot",
                "selector": "containers[] .securityContext .runAsNonRoot == true",
                "reason": "Force the running image to run as a non-root user to ensure least privilege",
                "points": 1
              },
              {
                "id": "RunAsUser",
                "selector": "containers[] .securityContext .runAsUser -gt 10000",
                "reason": "Run as a high-UID user to avoid conflicts with the host's user table",
                "points": 1
              }
          ]
        }
      }
    ]
```


*   _Similarly, you can use kubesec as a service via HTTPS at_ [_https://v2.kubesec.io/scan_](https://v2.kubesec.io/scan)

```
curl -sSX POST --data-binary @"pod.yaml" https://v2.kubesec.io/scan[{"object": "Pod/security-context-demo.default","valid": true,"fileName": "API","message": "Passed with a score of 0 points","score": 0,"scoring": {"advise": [{"id": "ApparmorAny","selector": ".metadata .annotations .\"container.apparmor.security.beta.kubernetes.io/nginx\"","reason": "Well defined AppArmor policies may provide greater protection from unknown threats. WARNING: NOT PRODUCTION READY","points": 3},{"id": "ServiceAccountName","selector": ".spec .serviceAccountName","reason": "Service accounts restrict Kubernetes API access and should be configured with least privilege","points": 3},{"id": "SeccompAny","selector": ".metadata .annotations .\"container.seccomp.security.alpha.kubernetes.io/pod\"","reason": "Seccomp profiles set minimum privilege and secure against unknown threats","points": 1},{"id": "LimitsCPU","selector": "containers[] .resources .limits .cpu","reason": "Enforcing CPU limits prevents DOS via resource exhaustion","points": 1},{"id": "LimitsMemory","selector": "containers[] .resources .limits .memory","reason": "Enforcing memory limits prevents DOS via resource exhaustion","points": 1},{"id": "RequestsCPU","selector": "containers[] .resources .requests .cpu","reason": "Enforcing CPU requests aids a fair balancing of resources across the cluster","points": 1},{"id": "RequestsMemory","selector": "containers[] .resources .requests .memory","reason": "Enforcing memory requests aids a fair balancing of resources across the cluster","points": 1},{"id": "CapDropAny","selector": "containers[] .securityContext .capabilities .drop","reason": "Reducing kernel capabilities available to a container limits its attack surface","points": 1},{"id": "CapDropAll","selector": "containers[] .securityContext .capabilities .drop | index(\"ALL\")","reason": "Drop all capabilities and add only those required to reduce syscall attack surface","points": 1},{"id": "ReadOnlyRootFilesystem","selector": "containers[] .securityContext .readOnlyRootFilesystem == true","reason": "An immutable root filesystem can prevent malicious binaries being added to PATH and increase attack cost","points": 1},{"id": "RunAsNonRoot","selector": "containers[] .securityContext .runAsNonRoot == true","reason": "Force the running image to run as a non-root user to ensure least privilege","points": 1},{"id": "RunAsUser","selector": "containers[] .securityContext .runAsUser -gt 10000","reason": "Run as a high-UID user to avoid conflicts with the host's user table","points": 1}]}}]
```


The other ways to run kubesec is :

**Docker Container**

```
docker run -i kubesec/kubesec:v2 scan /dev/stdin < pod.yaml
```


**HTTP Server**

```
kubesec http 8080 &
```


*   Then you can use curl to post the request

```
$ curl -sSX POST --data-binary @pod.yaml http://localhost:8080/scan

```


**_Conclusion_**

_As you can see, kubesec is an easy to use powerful tool to catch security-related issues or enforce standards before it’s deployed to the cluster._