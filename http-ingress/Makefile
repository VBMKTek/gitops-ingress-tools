# Makefile for HTTP Ingress Tools
CHART_NAME=http-ingress
RELEASE_NAME=http-ingress
NAMESPACE=common-tools

.PHONY: help lint template install upgrade uninstall status logs

help: ## Display this help message
	@echo "Available commands:"
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | awk 'BEGIN {FS = ":.*?## "}; {printf "  %-15s %s\n", $$1, $$2}'

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
	@echo "Ingress resources:"
	kubectl get ingress -n $(NAMESPACE)
	@echo ""
	@echo "Services in common-tools namespace:"
	kubectl get svc -n common-tools

test-connection: ## Test ingress connectivity
	@echo "Testing Keycloak connection via ingress..."
	@echo "URL: http://localhost/admin"
	@echo "Make sure your ingress controller is configured properly"
	@echo ""
	@echo "If using port-forward for testing:"
	@echo "kubectl port-forward -n ingress-nginx service/ingress-nginx-controller 80:80"
	@echo ""
	@echo "Or test with curl:"
	@echo "curl -H \"Host: localhost\" http://<INGRESS_IP>/admin"

logs: ## View ingress controller logs
	@echo "Viewing ingress controller logs..."
	kubectl logs -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx --tail=50

check-ingress-controller: ## Check if ingress controller is running
	@echo "Checking ingress controller status..."
	kubectl get pods -n ingress-nginx
	@echo ""
	kubectl get svc -n ingress-nginx

dry-run: ## Perform a dry-run installation
	@echo "Performing dry-run..."
	helm install $(RELEASE_NAME) . --namespace $(NAMESPACE) --dry-run --debug
