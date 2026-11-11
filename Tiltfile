# Deploy PostgreSQL and Catalog Service to Kubernetes
k8s_yaml('kubernetes/postgres.yml')
k8s_yaml('kubernetes/catalog-service.yml')

# Access Catalog Service at localhost:9001
k8s_resource('catalog-service', port_forwards=9001)