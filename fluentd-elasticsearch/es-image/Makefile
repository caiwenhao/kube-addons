.PHONY:	build push

IMAGE = elasticsearch
TAG = 5.0.1

build:	
	docker build -t 10.10.185.222/lifesense/$(IMAGE):$(TAG) .
push:
	docker push 10.10.185.222/lifesense/$(IMAGE):$(TAG)