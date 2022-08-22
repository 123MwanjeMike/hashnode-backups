## Set up Kubernetes role based access control

Kubernetes(K8s) role-based access control is a powerful tool in restricting access to resources within a Kubernetes cluster. In this post, we shall briefly discuss what role-based access control is, and how to set it up in Kubernetes. I promise, it's not as long a process as you may think.

## What is role-based access control?

Role-based access control(RBAC) is a security model in which users are assigned roles, and can then be granted access to certain resources based on those roles. For example, an organisation can have a role-based access control system that allows DevOps engineers to perform privileged operations within the entire Kubernetes cluster, while restricting the application developers to only viewing information about their applications deployed in a given namespace.

### Definitions

Before setting up RBAC in Kubernetes, let's review some of the key concepts that we are going to be dealing with:

* **Service accounts** are identities that can be used to delegate access to Kubernetes resources within a namespace. They can be used by users or services. We shall do some minor tweaks later on in the article using clusterrolebindings to enable us override the "namespacedness" limitation.

* **Resources** are objects within a Kubernetes cluster that we aim to implement role-based access control around. Examples include pods, deployments, replicasets, configmaps, and services. 

* **Roles** are collections of permissions that are applied within a given namespace. In most cases, these consist of a set of permissions that can be granted to service accounts.

* **Rolebindings** are the association of a role with a service account. In other words, rolebindings are used to assign roles. Think of a rolebinding as a mechanism to plug a set of permissions(a *role*) to a service account.

* **Clusterroles** are roles that can be assigned to service accounts within a Kubernetes cluster. Whereas roles are namespaced, clusterroles apply to the entire cluster. Some default ones in a k8s cluster include view, edit, admin, and cluster-admin.

* **Clusterrolebindings** are associations of a clusterrole with a service account. As you might have guessed, clusterrolebindings enable us assign cluster wide permissions to a service accounts through clusterroles.

Now let us see how we can set up RBAC in Kubernetes.

## How to set up a Kubernetes cluster using RBAC
### Prerequisites

- A working Kubernetes cluster. If you don't have one yet, you might find [this post](https://itnext.io/kubernetes-installation-methods-the-complete-guide-1036c860a2b3) helpful. It lists a couple of ways you could setup a K8s cluster.

- Kubectl. You can install kubectl following [this guide](https://kubernetes.io/docs/tasks/tools/#kubectl).

- Kubernetes 1.20 or later. Earlier versions of Kubernetes are no longer actively maintained and RBAC in those versions is not as stable. [This documentation](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/) can guide you through the process.

- [The Kubernetes Dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/) if you intend on using it in the [Access the Kubernetes dashboard](#heading-access-the-kubernetes-dashboard) section later on.

### Setting up RBAC within a namespace

First, let us create a namespace which we are going to be working with. Let's call it ***dev-env***. 
```
kubectl create namespace dev-env
```
*Output:*
![Output of  kubectl create command namespace](https://cdn.hashnode.com/res/hashnode/image/upload/v1660247153561/cQe5gGH0H.png align="left")

Next, create a service account called ***app-dev*** which we shall use all through out. 
```
kubectl create serviceaccount app-dev --namespace dev-env
```
*Output:*
![Output of  kubectl create command serviceaccount](https://cdn.hashnode.com/res/hashnode/image/upload/v1660247550731/NHgtbaLGv.png align="left")

We shall follow this up by creating an admin role that has all rights within the `dev-env` namespace. Below is its manifest, which is also available [on GitHub](https://github.com/123MwanjeMike/k8s-rbac/blob/main/admin-role.yaml).

```
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: admin
  namespace: dev-env
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
``` 
Apply the file to create the ***admin*** role.
```
kubectl apply -f https://raw.githubusercontent.com/123MwanjeMike/k8s-rbac/main/admin-role.yaml
``` 
*Output:*
![Output after creating role](https://cdn.hashnode.com/res/hashnode/image/upload/v1660251942764/5Qpd7n8Tz.png align="left")

***Note:*** *We shall be creating the rest of the Kubernetes resources in this tutorial from the pre-written manifest files in [this repository](https://github.com/123MwanjeMike/k8s-rbac) as we have done above. However, you may also edit these manifests if you wish to create K8s resources tailored to your needs.*

%[https://github.com/123MwanjeMike/k8s-rbac]
<br>
Next,create a rolebinding, ***app-dev-rolebinding***, of the `admin` role to the `app-dev` service account. Below is the manifest. 
```
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: app-dev-rolebinding
  namespace: dev-env
subjects:
- kind: ServiceAccount
  name: app-dev
  namespace: dev-env
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: admin
``` 
Run the command below to create it.
```
kubectl apply -f https://raw.githubusercontent.com/123MwanjeMike/k8s-rbac/main/rolebinding.yaml
``` 
*Output:*
![Output of creating a rolebinding](https://cdn.hashnode.com/res/hashnode/image/upload/v1660333008852/wt0J1nH5e.png align="left")

Our service account is now an admin within the 'dev-env` namespace. We can even go further and define access rights for it across the entire cluster. Let's say, basing on our DevOps engineer and application developer example we had at the start, that we want our application developers to view any resource cluster wide and even to use the Kubernetes dashboard. How we can we do this?

### Setting up RBAC for the entire cluster
For the first part, we're in luck! Kubernetes has a default view clusterrole which can be used for viewing of all resources in a cluster; and so we shall just leverage this. We shall create a clusterrolebinding, `app-dev-view`, that does just that.

```
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: app-dev-view
subjects:
- kind: ServiceAccount
  name: app-dev
  namespace: dev-env
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: view
```
Create the clusterrolebinding
```
kubectl apply -f https://raw.githubusercontent.com/123MwanjeMike/k8s-rbac/main/clusterrolebindings/view.yaml
```
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1660983996844/OYnYOQv_9.png align="left")

Now, for `app-dev` to access the Kubernetes dashboard, we could easily just create a bind the `edit` default clusterrole to it since it has the access rights needed for this. However, this would also give the service account more permissions than it should have and so we want to use a more fine-grained clusterrole in accordance to the principle of least privileges.

So, we shall create a custom clusterrole with only the required permissions needed to access the K8s dashboard on top of those already held by the service account through default `view` clusterrole.
```
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: view-addition-for-k8s-dashboard-access
rules:
- apiGroups: [""]
  resources: ["pods/attach", "pods/exec", "pods/portforward", "pods/proxy", "services/proxy"]
  verbs: ["get", "list", "watch", "create"]
- apiGroups: [""]
  resources: ["services"]
  verbs: ["proxy"]
```
Run the command below to create the `view-addition-for-k8s-dashboard-access` clusterrole with the above definition.
```
kubectl apply -f https://raw.githubusercontent.com/123MwanjeMike/k8s-rbac/main/custom-clusterrole.yaml
```
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1660988651204/toageyghC.png align="left")

We can now bind `view-addition-for-k8s-dashboard-access` clusterrole to the `app-dev` service account for it access the K8s dashboard. Let's have a look at the clusterrolebinding manifest for this.
```
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: app-dev-use-k8s-dashboard
subjects:
- kind: ServiceAccount
  name: app-dev
  namespace: dev-env
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: view-addition-for-k8s-dashboard-access
```
Create the `app-dev-use-k8s-dashboard` clusterrolebinding
```
kubectl apply -f https://raw.githubusercontent.com/123MwanjeMike/k8s-rbac/main/clusterrolebindings/use-k8s-dashboard.yaml
```
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1660989964648/FL4jrUjbc.png align="left")

Our `app-dev` service account can now be used by our "application developers" to create applications in the `dev-env` namespace and conveniently access the k8s dashboard.

## Putting it to use
### Create a kubectl configuration
We are going to need a token for the `app-dev` service account and so to get it, run the commands below
```
export NAMESPACE="dev-env"
export SERVICE_ACCOUNT="app-dev"
kubectl -n ${NAMESPACE} describe secret $(kubectl -n ${NAMESPACE} get secret | (grep ${SERVICE_ACCOUNT} || echo "$_") | awk '{print $1}') | grep token: | awk '{print $2}'\n
```
Your output will be some long string. It should start with something similar to what I have below. Save that **token** somewhere temporarily.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1661021009694/Vq5dbO3gW.png align="left")

Next, get the certificate data by running the command below in the same terminal where you had defined the *NAMESPACE* and *SERVICE_ACCOUNT* variables.
```
kubectl  -n ${NAMESPACE} get secret `kubectl -n ${NAMESPACE} get secret | (grep ${SERVICE_ACCOUNT} || echo "$_") | awk '{print $1}'` -o "jsonpath={.data['ca\.crt']}"
```
Your output will be another long string whose beginning is something similar to what I have below. Save that **certificate data** somewhere temporarily.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1661021487335/MAfiYFNYJ.png align="left")

With these two, we let's create our kubectl config file. You might already have a *.kube/config *which you might still need. So, we shall back it up as .kube/config.bak and create a new one for the `app-dev`. Run the commands below to do precisely that.
```
cd
mv .kube/config .kube/config.bak
nano .kube/config
```
This will open the nano text editor. Fill in the template below paste in the editor. Make sure to substitute the *certificate-data*, *server-ip-address*, *cluster-name*, and *token* with the correct values before saving the changes. Remember to delete the token and certificate data from where you had temporarily saved them.
```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data:  <certificate-data>
    server: https://<server-ip-address>:6443
  name: <cluster-name>
contexts:
- context:
    cluster: <cluster-name>
    namespace: dev-env
    user: app-dev
  name: <cluster-name>
current-context: dev-env
kind: Config
preferences: {}
users:
- name: app-dev
  user:
    client-key-data:  <certificate-data>
    token:  <token>
```
### Access the Kubernetes dashboard
1. Run `kubectl proxy` and click [here](http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/) to access the dashboard.
2. Sign in with the .kube/config we created in the previous step.
3. Attempt to create a user role in the dev-env namespace with the manifest below. All should go on well.

```
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: test-admin
  namespace: dev-env
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
```

![test-admin role creation successful in dev-env namespace](https://cdn.hashnode.com/res/hashnode/image/upload/v1661025164815/2Rl9dZStI.png align="left")
4. Attempt to create the user role in the kube-public namespace with the manifest below.

```
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: test-admin
  namespace: kube-public
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
```
Tadaa!
![test-admin role creation failed in kube-public namespace](https://cdn.hashnode.com/res/hashnode/image/upload/v1661023906267/ynJGj_q5q.png align="left")

We did it. `app-dev` has no such rights outside the `dev-env` namespace. All application developers can now use that .kube/config file to gain controlled access to the cluster

## Conclusion
In this post, we looked at how we can implement RBAC not only at the namespace level, but also at the cluster level. We specifically used a service account given that it is managed by Kubernetes taking away this task from us. It is also important to note that Kubernetes also user accounts as much as it does service accounts. One key difference between the two is that user accounts represent actual Kubernetes users unlike the service accounts.

I hope you enjoyed this article and that you'll show support of some sort as I'd greatly appreciate that. And if you have any thoughts or questions in relation to this, feel free to drop them via the comments section. I'd love to hear from you.

*Happy hacking*


### Additional resources
1. [Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
2. [Kubernetes default clusterroles definitions](https://github.com/kubernetes/kubernetes/blob/master/plugin/pkg/auth/authorizer/rbac/bootstrappolicy/testdata/cluster-roles.yaml)
3. [The principle of least privileges](https://en.wikipedia.org/wiki/Principle_of_least_privilege)