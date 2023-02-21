# Dynamic-agent-Jenkins

A small project to implement dynamic agents for Jenkins

### Steps:

- **First thing is to set up the dev cluster where we are going to try all the mess. u can also use your own cluster in place of this dev cluster**
    - using Kind let's create the cluster `kind create cluster --config cluster-config.yaml` this will create the cluster.
    - We can check the details of the cluster using kubectl `kubectl cluster-info --context kind-kind`
    - To install reffer [https://kind.sigs.k8s.io/](https://kind.sigs.k8s.io/).

- **Now lets create a namespace in K8s which we are going to use**
    - `kubectl create namespace jenkins` this will create the namespace named jenkins.
    - Make sure to keep a record of the namespace for future ref.

- **We have to create a service account with admin privileges as we are going to use this service account to perform all the required actions on the cluser.**
    - to create the service account just apply the serviceAccount.yaml file on the approprate context and namespace `kubectl apply -f .\serviceAccount.yaml --context kind-kind -n jenkins`
    - this file will create the following:
        1. **`ServiceAccount`** with the name **`jenkins`** in the **`jenkins`** namespace.
        2. **`Secret`** with the name **`jenkins-token`** in the **`jenkins`** namespace. This secret is of type **`kubernetes.io/service-account-token`**, and has an annotation **`kubernetes.io/service-account.name`** with the value **`jenkins`**, which is required for the token to be valid.
        3. **`ClusterRoleBinding`** with the name **`jenkins-cr`**. This binds the **`cluster-admin`** role to the **`jenkins`** service account in the **`jenkins`** namespace.
    
    - once the account is created successfully we can get the token for the jenkins service account using the following command `kubectl get secret jenkins-token --namespace jenkins --context kind-kind -o jsonpath='{.data.token}'` make sure to change the necessary information if u make any changes in the yaml file .
    - this will return a base64 encoded value so make sure to decode it before you use it.

 

- **Installing the K8s plugin in Jenkins**
    - instruction on how to install [https://plugins.jenkins.io/kubernetes/](https://plugins.jenkins.io/kubernetes/)
- **Once we are done installing we can move on to config part of the plugin**
    - go to `Dashboard>Manage Jenkins>Manage nodes and clouds>Configure Clouds` and add a new cloud and select Kubernetes.
    - config the required settings like
        - Name : `kubernetes` or whatever you want to set, just make sure u remember it as this is going to be the name of the agent on which the jobs can be executed.
        - Kubernetes URL : we can get the URL for this from `cluster-info` command
        - Disable https certificate check : in case you don’t have a valid https cert mark it `True`
        - Kubernetes Namespace : the namespace we created `jenkins`
        - Jenkins URL : Change it if u are using [localhost](http://localhost) or you cant route a named entry.
        - WebSocket : mark it `True`
        - Credentials : create a new secret text credential with the value of `jenkins-token` as the id .
    - now we can save it and the connection is ready to run jobs.
    - we can define pod template also in case we need it.
- Creating a job:
    - we can create jobs to run on k8s by setting the agent to `kubernetes`
    - example :
    
    ```yaml
    pipeline {
      agent {
        kubernetes {
          yaml '''
            apiVersion: v1
            kind: Pod
            spec:
              containers:
              - name: alpine
                image: alpine
                tty: true
            '''
        }
      }
      stages {
        stage('test run') {
          steps {
            container('alpine') {
              sh 'echo "hello world" && sleep 120'
            }
          }
        }
      }
    }
    ```
    
    we first define the the pod where we want to run the job then we use the container using the container option with the name of the container as argument.