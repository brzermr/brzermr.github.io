> 基于kubesphere的devops组件(jenkins)

`Jenkinsfile`

```
pipeline {

  agent {
    node {label 'base'}
  }
  
  parameters {
    string(name: 'TAG_NAME', defaultValue: "v1.0.0", description: '')
  }

  environment {
    IMAGE_REGISTRY = ''
    NEXUS_CREDENTIAL_ID = ''
    GITLAB_CREDENTIAL_ID = ''
    KUBECONFIG_CREDENTIAL_ID = ''
    CURRENT_BUILD_NUMBER = "$currentBuild.number"
  }
  
  stages {
    
    stage('checkout scm') {
      steps {
        checkout(scm)
      }
    }
    
    stage('build') {
      steps {
        container('base') {
          sh 'docker build -t $IMAGE_REGISTRY/to/app:$TAG_NAME .'
          withCredentials([usernamePassword(passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME', credentialsId: "$NEXUS_CREDENTIAL_ID")]) {
            sh 'echo "$DOCKER_PASSWORD" | docker login $IMAGE_REGISTRY -u "$DOCKER_USERNAME" --password-stdin'
            sh 'docker push $IMAGE_REGISTRY/to/app:$TAG_NAME'
          }
        }
      }
    }
    
    stage('deploy') {
      when {
        allOf {
          expression {
            return params.TAG_NAME =~ /v.*/
          }
        }
      }
      steps {
        kubernetesDeploy(configs: "deployment.yaml", enableConfigSubstitution: true, kubeconfigId: "$KUBECONFIG_CREDENTIAL_ID")
      }
    }
    
  }

}
```

`Dockerfile`

```
FROM python
RUN mkdir /app && apt-get update
RUN ping mirrors.aliyun.com -c4
COPY .  /app
RUN pip install -r /app/requirements.txt -i https://mirrors.aliyun.com/pypi/simple/
WORKDIR /app
CMD ["python","/app/app.py"]
```

`deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: test
  name: apptest
spec:
  progressDeadlineSeconds: 600
  replicas: 2
  selector:
    matchLabels:
      app: apptest
  template:
    metadata:
      name: apptest
      labels:
        app: apptest
        version: $TAG_NAME
        buildNumber: $CURRENT_BUILD_NUMBER
    spec:
      imagePullSecrets:
        - name: nexusKey
      containers:
        - name: apptest
          image: $IMAGE_REGISTRY/to/app:$TAG_NAME
          imagePullPolicy: Always
          terminationMessagePath: /var/log/termination-log
          terminationMessagePolicy: File
          ports:
            - containerPort: 6666
              protocol: TCP
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
---
apiVersion: v1
kind: Service
metadata:
  name: apptest
  namespace: test
  labels:
    app: apptest
spec:
  sessionAffinity: None
  ports:
    - name: http
      port: 6666
      protocol: TCP
      targetPort: 6666
  selector:
    app: apptest
  type: NodePort
  externalTrafficPolicy: Cluster
```
