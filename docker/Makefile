usr=$(shell cat github-token | tail -n1)
tok=$(shell cat github-token | head -n1)

# REGISTRY=registry.gitlab.uni-bonn.de:5050

REGISTRY=ghcr.io
REGPATH=dluitz/bonn-comp-phys-cxx-dev/bonn-comp-phys-cxx-dev


all: login build push

login:
	docker login -u $(usr) -p $(tok) ${REGISTRY}

build:
	docker build -t ${REGISTRY}/${REGPATH}:latest -f Dockerfile .

push:
	docker push ${REGISTRY}/${REGPATH}
