kops / kubectl - how do i import state created on a another server?
https://stackoverflow.com/questions/48084506/kops-kubectl-how-do-i-import-state-created-on-a-another-server

To run kubectl command, you will need the cluster's apiServer URL and related credentials for authentication. Those data are by convention stored in ~/.kube/config file. You may also view it via kubectl config view command.

In order to run kubectl on your CI server, you need to make sure the ~/.kube/config file contains all the information that kubectl client needs.

With kops, a simple naive solution is to:

1) install kops, kubectl on your CI server

2) config the AWS access credential on your CI server (either via IAM Role or simply env vars), make sure it has access to your s3 state store path

3) set env var for kops to access your cluster:

  export NAME=${YOUR_CLUSTER_NAME}
  export KOPS_STATE_STORE=s3://${YOUR_CLUSTER_KOPS_STATE_STORE}
4) Use kops export command to get the kubecfg needed for running kubectl

  kops export kubecfg ${YOUR_CLUSTER_NAME}
see https://github.com/kubernetes/kops/blob/master/docs/cli/kops_export.md

Now the ~/.kube/config file on your CI server should contain all the information kubectl needs to access your cluster.

Note that this will use the default admin account on your CI server. To implement a more secure CI/CD environment, you should create a service account bind to a required permission scope (a namespace or type or resources for example), and place its credential on your CI server machine.