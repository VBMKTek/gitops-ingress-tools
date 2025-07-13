# HTTP Ingress for Common Tools

Helm chart này cung cấp HTTP ingress routing cho các công cụ common tools như Keycloak, Grafana, Kibana, v.v.

## Yêu cầu

- Kubernetes cluster với ingress controller đã được cài đặt (nginx, traefik, haproxy, v.v.)
- Helm 3.x
- Các service đích đã được deploy trong namespace `common-tools`

## Cài đặt Ingress Controller

Nếu chưa có ingress controller, cài đặt nginx ingress controller:

```bash
# Cài đặt nginx ingress controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml

# Hoặc với Helm
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace
```

## Cấu hình

### values.yaml

Chỉnh sửa `values.yaml` để cấu hình:

```yaml
ingress:
  host: your-domain.com  # Thay đổi thành domain của bạn
  className: "nginx"     # Thay đổi tùy ingress controller

keycloak:
  enabled: true          # Bật/tắt routing cho Keycloak
  path: "/admin"         # Path để truy cập Keycloak
```

## Deployment

```bash
# Lint chart
make lint

# Xem template được generate
make template

# Cài đặt
make install

# Upgrade
make upgrade

# Kiểm tra status
make status

# Test connection
make test-connection
```

## Truy cập Services

Sau khi deploy, các service có thể truy cập qua:

### Keycloak
- URL: `http://localhost/admin` (local development)
- URL: `http://your-domain.com/admin` (production)

### Các service khác (khi enable)
- Grafana: `http://your-domain.com/grafana`
- Kibana: `http://your-domain.com/kibana`

## Testing

### Local Development

1. Port-forward ingress controller:
```bash
kubectl port-forward -n ingress-nginx service/ingress-nginx-controller 80:80
```

2. Truy cập Keycloak:
```bash
curl -H "Host: localhost" http://localhost/admin
# Hoặc mở browser: http://localhost/admin
```

### Production

1. Cấu hình DNS để point domain đến ingress controller
2. Truy cập trực tiếp qua domain

## Troubleshooting

### Kiểm tra ingress controller
```bash
make check-ingress-controller
```

### Xem logs
```bash
make logs
```

### Kiểm tra ingress resource
```bash
kubectl get ingress -n common-tools
kubectl describe ingress http-ingress -n common-tools
```

### Kiểm tra service endpoints
```bash
kubectl get endpoints -n common-tools
```

## Cấu hình TLS (HTTPS)

Để enable HTTPS, cập nhật `values.yaml`:

```yaml
ingress:
  tls:
    enabled: true
    secretName: tls-secret
```

Tạo TLS secret:
```bash
kubectl create secret tls tls-secret \
  --cert=path/to/tls.crt \
  --key=path/to/tls.key \
  -n common-tools
```

## Thêm Service Mới

Để thêm routing cho service mới, cập nhật `values.yaml`:

```yaml
new-service:
  enabled: true
  path: "/new-service"
  pathType: Prefix
  service:
    name: new-service-name
    port: 8080
    namespace: common-tools
```

Và thêm vào template `ingress.yaml`:
```yaml
{{- if .Values.new-service.enabled }}
- path: {{ .Values.new-service.path }}
  pathType: {{ .Values.new-service.pathType }}
  backend:
    service:
      name: {{ .Values.new-service.service.name }}
      port:
        number: {{ .Values.new-service.service.port }}
{{- end }}
```

## Uninstall

```bash
make uninstall
```
