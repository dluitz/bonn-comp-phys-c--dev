usr=$(shell cat gitlab-token | tail -n1)
tok=$(shell cat gitlab-token | head -n1)

REGISTRY=registry.gitlab.uni-bonn.de:5050
REGPATH=dluitz/bonn-comp-phys-cxx-dev


all: login build push

login:
	docker login -u $(usr) -p $(tok) ${REGISTRY}

build:
	docker build -t ${REGISTRY}/${REGPATH}:latest -f Dockerfile .

push:
	docker push ${REGISTRY}/${REGPATH}
