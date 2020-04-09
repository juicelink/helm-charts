default: test

include ../../../helpers/examples.mk

RELEASE := elasticsearch
CLUSTER_NAME := elasticsearch
NAMESPACE := elasticstack
ELASTICSEARCH_IMAGE := docker.elastic.co/elasticsearch/elasticsearch:$(STACK_VERSION)

install:
	helm upgrade --namespace=$(NAMESPACE) --wait --timeout=600 --install --values ./security.yml $(RELEASE) ../../
	# helm install --namespace=$(NAMESPACE) --wait --timeout=600 --name $(RELEASE) --values ./security.yml ../../
purge:
	kubectl delete secrets elastic-credentials elastic-certificates elastic-certificate-pem -n $(NAMESPACE) || true
	# helm del --purge $(RELEASE)

test: secrets install goss

pull-elasticsearch-image:
	docker pull $(ELASTICSEARCH_IMAGE)

secrets:
	docker rm -f elastic-helm-charts-certs || true
	rm -f elastic-certificates.p12 elastic-certificate.pem elastic-stack-ca.p12 || true
	password=$$([ ! -z "$$ELASTIC_PASSWORD" ] && echo $$ELASTIC_PASSWORD || echo $$(docker run --rm $(ELASTICSEARCH_IMAGE) /bin/sh -c "< /dev/urandom tr -cd '[:alnum:]' | head -c20")) && \
	docker run --name elastic-helm-charts-certs -i -w /app \
		$(ELASTICSEARCH_IMAGE) \
		/bin/sh -c " \
			elasticsearch-certutil ca --out /app/elastic-stack-ca.p12 --pass '' && \
			elasticsearch-certutil cert --name $(CLUSTER_NAME)-master --dns $(CLUSTER_NAME)-master --ca /app/elastic-stack-ca.p12 --pass '' --ca-pass '' --out /app/elastic-certificates.p12" && \
	docker cp elastic-helm-charts-certs:/app/elastic-certificates.p12 ./ && \
	docker rm -f elastic-helm-charts-certs && \
	openssl pkcs12 -nodes -passin pass:'' -in elastic-certificates.p12 -out elastic-certificate.pem && \
	kubectl create secret generic elastic-certificates -n $(NAMESPACE) --from-file=elastic-certificates.p12 && \
	kubectl create secret generic elastic-certificate-pem -n $(NAMESPACE) --from-file=elastic-certificate.pem && \
	kubectl create secret generic elastic-credentials  -n $(NAMESPACE) --from-literal=password=$$password --from-literal=username=elastic && \
	rm -f elastic-certificates.p12 elastic-certificate.pem elastic-stack-ca.p12
