# Makefile for TCP Ingress Tools
CHART_NAME=tcp-ingress
RELEASE_NAME=tcp-ingress
NAMESPACE=ingress-nginx

.PHONY: help lint template install upgrade uninstall status test-connections

help: ## Display this help message
	@echo "Available commands:"
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | awk 'BEGIN {FS = ":.*?## "}; {printf "  %-20s %s\n", $$1, $$2}'

lint: ## Lint the Helm chart
	@echo "Linting Helm chart..."
	helm lint .

template: ## Generate Kubernetes manifests without installing
	@echo "Generating templates..."
	helm template $(RELEASE_NAME) . --namespace $(NAMESPACE)

install: ## Install the Helm chart
	@echo "Installing $(CHART_NAME)..."
	helm install $(RELEASE_NAME) . --namespace $(NAMESPACE) --create-namespace

upgrade: ## Upgrade the Helm chart
	@echo "Upgrading $(CHART_NAME)..."
	helm upgrade $(RELEASE_NAME) . --namespace $(NAMESPACE)

uninstall: ## Uninstall the Helm chart
	@echo "Uninstalling $(CHART_NAME)..."
	helm uninstall $(RELEASE_NAME) --namespace $(NAMESPACE)

status: ## Check the status of the deployment
	@echo "Checking status..."
	helm status $(RELEASE_NAME) --namespace $(NAMESPACE)
	@echo ""
	@echo "TCP Services ConfigMap:"
	kubectl get configmap tcp-services -n ingress-nginx -o yaml
	@echo ""
	@echo "TCP Ingress Service:"
	kubectl get svc -n ingress-nginx | grep tcp

test-connections: ## Test database connections via TCP ingress
	@echo "Testing database connections..."
	@echo ""
	@echo "PostgreSQL connection test:"
	@echo "psql -h 119.9.70.83 -p 32432 -U postgres -d postgres"
	@echo ""
	@echo "MongoDB connection test:"
	@echo "mongosh mongodb://119.9.70.83:32017/admin"
	@echo ""
	@echo "DBeaver connections:"
	@echo "PostgreSQL: 119.9.70.83:32432"
	@echo "MongoDB: 119.9.70.83:32017"

check-ingress-controller: ## Check nginx ingress controller configuration
	@echo "Checking nginx ingress controller..."
	kubectl get pods -n ingress-nginx
	@echo ""
	kubectl get svc -n ingress-nginx
	@echo ""
	kubectl describe deployment nginx-ingress-controller -n ingress-nginx

check-backend-services: ## Check backend database services
	@echo "Checking backend services..."
	@echo "PostgreSQL:"
	kubectl get svc postgres -n core-infra
	kubectl get pods -n core-infra | grep postgres
	@echo ""
	@echo "MongoDB:"
	kubectl get svc mongodb -n core-infra
	kubectl get pods -n core-infra | grep mongodb

logs: ## View nginx ingress controller logs
	@echo "Viewing nginx ingress controller logs..."
	kubectl logs -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx --tail=50

dry-run: ## Perform a dry-run installation
	@echo "Performing dry-run..."
	helm install $(RELEASE_NAME) . --namespace $(NAMESPACE) --dry-run --debug
