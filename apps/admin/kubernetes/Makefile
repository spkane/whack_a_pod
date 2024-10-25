# Copyright 2017 Google Inc. All Rights Reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
BASEDIR = $(shell pwd)

include ../../../Makefile.properties


reset: clean.deployment app
	say "App Reset"

reset.safe: 
	CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o "$(BASEDIR)/../containers/main" "$(BASEDIR)/../containers/main.go" "$(BASEDIR)/../containers/kubernetes.go"
	gcloud container builds submit "$(BASEDIR)/../containers/." --tag=$(DOCKERREPO)/admin
	-kubectl delete --ignore-not-found=true deployment admin-deployment
	-kubectl create deployment admin-deployment --replicas=1 --image=$(DOCKERREPO):admin --port=8080
	kubectl set env deployment admin-deployment APIIMAGE=$(DOCKERREPO):api
	kubectl label deployment --overwrite admin-deployment app=admin
	say "app refresh complete"		

app: build deploy

main:
	pwd
	CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o "$(BASEDIR)/../containers/main" "$(BASEDIR)/../containers/main.go" "$(BASEDIR)/../containers/kubernetes.go"	

build: env main
	gcloud container builds submit "$(BASEDIR)/../containers/." --tag=$(DOCKERREPO)/admin

build.dockerhub: main
	docker build "$(BASEDIR)/../containers/." --tag=$(DOCKERREPO):admin
	docker push $(DOCKERREPO):admin

build.generic: main
	docker build "$(BASEDIR)/../containers/." --tag=$(DOCKERREPO)/whackapod-admin
	docker push $(DOCKERREPO)/whackapod-admin

deploy: env creds deployment service

deploy.minikube: deployment service
	
deploy.minikube.dockerhub:
	kubectl create deployment admin-deployment --replicas=1 --image=$(DOCKERREPO):admin --port=8080
	kubectl set env deployment admin-deployment APIIMAGE=$(DOCKERREPO):api
	kubectl label deployment --overwrite admin-deployment app=admin
	kubectl expose deployment admin-deployment --name=admin --target-port=8080  --type=NodePort --labels="app=admin"
	kubectl create serviceaccount wap-admin
	kubectl create clusterrolebinding wap-admin --clusterrole=cluster-admin --serviceaccount=default:wap-admin
	kubectl set serviceaccount deployment admin-deployment wap-admin

deploy.generic:
	kubectl create deployment admin-deployment --replicas=1 --image=$(DOCKERREPO):admin --port=8080
	kubectl set env deployment admin-deployment APIIMAGE=$(DOCKERREPO):api
	kubectl label deployment --overwrite admin-deployment app=admin
	kubectl expose deployment admin-deployment --name=admin --target-port=8080  --type=NodePort --labels="app=admin"
	kubectl create serviceaccount wap-admin	
	kubectl create clusterrolebinding wap-admin --clusterrole=cluster-admin --serviceaccount=$$(kubectl get sa -o=jsonpath="{.items[0]..metadata.namespace}"):wap-admin
	kubectl set serviceaccount deployment admin-deployment wap-admin

test: 
	cd 	"$(BASEDIR)/../containers" && go test

deployment:
	kubectl create deployment admin-deployment --replicas=1 --image=$(DOCKERREPO):admin --port=8080
	kubectl set env deployment admin-deployment APIIMAGE=$(DOCKERREPO):api
	kubectl label deployment --overwrite admin-deployment app=admin

service:
	kubectl expose deployment admin-deployment --name=admin --target-port=8080  --type=NodePort --labels="app=admin"

clean: env creds clean.deployment clean.service 

clean.minikube: clean.deployment clean.service

clean.minikube.dockerhub: clean.deployment clean.service
	-kubectl delete --ignore-not-found=true serviceaccount wap-admin
	-kubectl delete --ignore-not-found=true clusterrolebinding wap-admin 

clean.deployment: 
	-kubectl delete --ignore-not-found=true deployment admin-deployment
	
clean.service: 
	-kubectl delete --ignore-not-found=true service admin

clean.generic: clean.deployment clean.service
	-kubectl delete --ignore-not-found=true serviceaccount wap-admin
	-kubectl delete --ignore-not-found=true clusterrolebinding wap-admin 

retry: clean build deploy

config: env 
	echo "No custom config needed for admin"
