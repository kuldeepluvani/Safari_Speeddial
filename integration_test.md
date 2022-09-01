# integration test setup
---
* Install minikube
* bootstrap argocd
  * Create namespace `argocd` for the argocd app. `kubectl create namespace argocd`
  * Install Argocd: `kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml`
  * Get secret for login: `kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d` (make sure to copy the secret key)
  * Kubernetes port forward: `kubectl port-forward svc/argocd-server -n argocd 8080:443`
  * Argocd Login: `argocd login localhost:8080`
  * Install app/apps pointer: `argocd repo add https://github.com/ZEFR-INC/apps --username <username> --password <PAT token>`
* Create an argocd app:
  `argocd app create integration-test \
  --repo https://github.com/ZEFR-INC/apps/integration-test.git \
  --path integration-test/overlays/stage \
  --dest-namespace integration-test \
  --dest-server https://kubernetes.default.svc \
  --sync-policy automated \
  --auto-prune \
  --revision integration_test \
  --port-forward`

* Test if Kafka is running:
  * Create `cp-kafka` namespace for kafka broker: `kubectl create namespace cp-kafka`
  * Consumer: `kubectl exec -c cp-kafka-broker -it kafka-cp-kafka-0 -- /bin/bash  /usr/bin/kafka-console-consumer --bootstrap-server localhost:9092 --topic test --from-beginning`
  * Producer: `kubectl exec -c cp-kafka-broker -it kafka-cp-kafka-0 -- /bin/bash /usr/bin/kafka-console-producer --broker-list localhost:9092 --topic test`


* Enable registry to pull image from ECR:
`minikube addons enable registry
    ╭──────────────────────────────────────────────────────────────────────────────────────────────╮
💡  │Registry addon with docker driver uses port 59387 please use that instead of default port 5000`



TODO:
* Create a Publisher and Consumer for local minikube development env. (Generic one which takes topic names as an input)
  * Try creating single topic local image for consumer Producer
  * Test Publisher and consumer for one topic
  * Create a Publisher to pulish multiple topics if needed.


Questions:
- Will it be separate docker container?
- Are we using the localdev from app? How are we planning to fetch the image?

-> Life cycle hook: sync and updated to verify if we are ready to run test. (could be wait, sleep or status check)
-> Run test and output log
