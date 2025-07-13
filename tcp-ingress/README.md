# TCP Ingress for Database Connections

Helm chart này cung cấp TCP ingress routing cho các database như PostgreSQL, MongoDB, Redis thông qua Nginx Ingress Controller.

## Yêu cầu

- Kubernetes cluster với nginx ingress controller
- Helm 3.x
- Các database services đã được deploy trong namespace `core-infra`

## Cấu hình

### values.yaml

```yaml
ingress:
  enabled: true
  className: "nginx"
  externalIP: "119.9.70.83"  # IP của nginx ingress controller

postgresql:
  enabled: true
  externalPort: 32432        # Port để connect từ bên ngoài
  service:
    name: postgres
    port: 5432
    namespace: core-infra

mongodb:
  enabled: true
  externalPort: 32017
  service:
    name: mongodb
    port: 27017
    namespace: core-infra
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

# Test connections
make test-connections
```

## Database Connections

Sau khi deploy, có thể connect vào databases qua:

### PostgreSQL
```bash
# Command line
psql -h 119.9.70.83 -p 32432 -U postgres -d postgres

# DBeaver/DataGrip
Host: 119.9.70.83
Port: 32432
Database: postgres
Username: postgres
Password: [password]
```

### MongoDB
```bash
# Command line
mongosh mongodb://119.9.70.83:32017/admin

# MongoDB Compass
mongodb://119.9.70.83:32017

# DBeaver
Host: 119.9.70.83
Port: 32017
Database: admin
Username: [username]
Password: [password]
```

## Cách hoạt động

TCP Ingress sử dụng 2 thành phần chính:

1. **ConfigMap `tcp-services`**: Định nghĩa mapping giữa external ports và backend services
2. **LoadBalancer Service**: Expose TCP ports ra ngoài với external IP

### ConfigMap tcp-services
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: tcp-services
  namespace: ingress-nginx
data:
  "32432": "core-infra/postgres:5432"
  "32017": "core-infra/mongodb:27017"
```

### LoadBalancer Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: tcp-ingress-tcp
  namespace: ingress-nginx
spec:
  type: LoadBalancer
  loadBalancerIP: "119.9.70.83"
  ports:
  - name: postgresql
    port: 32432
    targetPort: 32432
    protocol: TCP
  - name: mongodb
    port: 32017
    targetPort: 32017
    protocol: TCP
```

## Testing

### Kiểm tra ConfigMap
```bash
kubectl get configmap tcp-services -n ingress-nginx -o yaml
```

### Kiểm tra Service
```bash
kubectl get svc -n ingress-nginx | grep tcp
```

### Test PostgreSQL connection
```bash
# Từ trong cluster
kubectl exec -it postgres-xxx -n core-infra -- psql -U postgres -d postgres

# Từ bên ngoài (sau khi deploy tcp-ingress)
psql -h 119.9.70.83 -p 32432 -U postgres -d postgres
```

### Test MongoDB connection
```bash
# Từ trong cluster
kubectl exec -it mongodb-xxx -n core-infra -- mongosh

# Từ bên ngoài (sau khi deploy tcp-ingress)
mongosh mongodb://119.9.70.83:32017/admin
```

## Troubleshooting

### Common Issues

1. **Connection refused**
   - Kiểm tra nginx ingress controller có đang chạy không
   - Verify external IP đúng
   - Check backend services có available không

2. **Port conflicts**
   - Ensure external ports không conflict với services khác
   - Check firewall rules

3. **DNS resolution**
   - Verify service names và namespaces đúng
   - Check backend services có endpoints không

### Debug Commands

```bash
# Check nginx ingress controller
make check-ingress-controller

# Check backend services
make check-backend-services

# View nginx logs
make logs

# Check ConfigMap
kubectl describe configmap tcp-services -n ingress-nginx

# Check Service
kubectl describe svc tcp-ingress-tcp -n ingress-nginx
```

## Security Considerations

1. **Firewall Rules**: Ensure external ports chỉ accessible từ trusted networks
2. **Authentication**: Configure database authentication properly
3. **Encryption**: Consider TLS/SSL cho database connections
4. **Network Policies**: Implement network policies để restrict traffic

## Production Deployment

### Best Practices

1. **Use specific external IPs**: Đừng dùng ephemeral IPs
2. **Monitor connections**: Set up monitoring cho database connections
3. **Backup và restore**: Ensure backup procedures work với external access
4. **Resource limits**: Set appropriate resource limits cho nginx controller

### Recommended Configuration

```yaml
ingress:
  externalIP: "your-static-ip"
  
postgresql:
  externalPort: 5432  # Use standard ports in production
  
mongodb:
  externalPort: 27017
```

## Monitoring

Integrate với Grafana để monitor:
- Connection counts
- Query performance
- Error rates
- Resource usage

Xem thêm tại [Grafana Dashboard section](../grafana/README.md)
