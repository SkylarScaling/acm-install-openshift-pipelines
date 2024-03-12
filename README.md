# acm-install-openshift-pipelines
ACM Policy for installing Openshift Pipelines operator

Notes:
- User needs to be added to open-cluster-management:subscription-admin ClusterRoleBinding


    kind: ClusterRoleBinding  
    apiVersion: rbac.authorization.k8s.io/v1  
    metadata:  
      name: 'open-cluster-management:subscription-admin'
    subjects:
      - kind: User
        apiGroup: rbac.authorization.k8s.io
        name: admin
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: 'open-cluster-management:subscription-admin'

- Add user to system:image-puller role for project


    kind: ClusterRoleBinding  
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: sky-image-pull-binding
    subjects:
      - kind: User
        apiGroup: rbac.authorization.k8s.io
        name: admin
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: 'system:image-puller'

A subscription to this repo might look like this:

    apiVersion: v1
    kind: Namespace
    metadata:
      name: acm-install-policies
    ---
    apiVersion: app.k8s.io/v1beta1
    kind: Application
    metadata:
      name: pipelines-install-policy
      namespace: acm-install-policies
    spec:
      componentKinds:
        - group: apps.open-cluster-management.io
          kind: Subscription
      selector:
        matchExpressions:
          - key: app
            operator: In
            values:
              - pipelines-install-policy
    ---
    apiVersion: apps.open-cluster-management.io/v1
    kind: Channel
    metadata:
      annotations:
        apps.open-cluster-management.io/reconcile-rate: medium
      name: pipelines-install-policy-channel
      namespace: acm-install-policies
    spec:
      type: Git
      pathname: https://github.com/skylarscaling/acm-install-openshift-pipelines.git
    ---
    apiVersion: apps.open-cluster-management.io/v1
    kind: Subscription
    metadata:
      annotations:
        apps.open-cluster-management.io/git-branch: main
        # apps.open-cluster-management.io/git-path: N/A
        apps.open-cluster-management.io/reconcile-option: merge
      labels:
        app: pipelines-install-policy
      name: pipelines-install-policy-subscription
      namespace: acm-install-policies
    spec:
      channel: acm-install-policies/pipelines-install-policy-channel
      placement:
        placementRef:
          kind: PlacementRule
          name: pipelines-install-policy-placement
    ---
    apiVersion: apps.open-cluster-management.io/v1
    kind: PlacementRule
    metadata:
      labels:
        app: pipelines-install-policy
      name: pipelines-install-policy-placement
      namespace: acm-install-policies
    spec:
      clusterSelector:
        matchLabels:
          name: local-cluster