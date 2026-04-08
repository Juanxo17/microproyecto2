# Computacion en la Nube - Parcial 2

Juan Manuel Lame / Juan Felipe Plata
Universidad Autonoma de Occidente

---

## Descripcion

Aplicacion de e-commerce basada en microservicios desplegada en tres entornos: infraestructura con Vagrant + Terraform + Ansible, balanceo de carga con HAProxy, y orquestacion con Kubernetes (Minikube).

Microservicios:
- users-service (puerto 5002, expuesto en 3001)
- products-service (puerto 5003, expuesto en 3002)
- orders-service (puerto 5004, expuesto en 3003)

Imagenes en Docker Hub: juan1722/users-service, juan1722/products-service, juan1722/orders-service

---

## Estructura

parcial_cloud/
├── Vagrantfile                      # Define las 3 VMs
├── main.tf                          # Terraform: provisiona vm-haproxy y vm-microservices
├── inventory.ini                    # Ansible: IPs y llaves SSH de los nodos target
├── playbook-haproxy.yml             # Ansible: instala y configura HAProxy en vm-haproxy
├── playbook-microservices.yml       # Ansible: instala Docker y levanta los contenedores
├── haproxy.cfg                      # Configuracion de HAProxy con 3 backends
├── consul.yml                       # K8s: Deployment + Service de Consul
├── databases.yml                    # K8s: Deployments + Services de MariaDB (x3)
├── users.yml                        # K8s: Deployment + Service de users-service
├── products.yml                     # K8s: Deployment + Service de products-service
├── orders.yml                       # K8s: Deployment + Service de orders-service
└── ingress.yml                      # K8s: Ingress con enrutamiento por ruta

---

## Infraestructura (Problemas 1 y 2)

| VM | IP | Rol |
|---|---|---|
| control-node | 192.168.56.10 | Ejecuta Terraform y Ansible |
| vm-haproxy | 192.168.56.11 | Corre HAProxy en puerto 80 |
| vm-microservices | 192.168.56.12 | Corre los contenedores Docker |

### Inicializacion

1. Levantar las VMs:

    vagrant up

2. Entrar al control-node y aprovisionar:

    vagrant ssh control-node
    cd ~/infra && terraform init && terraform apply
    ansible-playbook -i ansible/inventory.ini ansible/playbook-microservices.yml
    ansible-playbook -i ansible/inventory.ini ansible/playbook-haproxy.yml

### HAProxy

- Frontend en puerto 80, enruta por prefijo de ruta
- Backend users: 192.168.56.12:3001 y :3011
- Backend products: 192.168.56.12:3002 y :3012
- Backend orders: 192.168.56.12:3003 y :3013
- Dashboard: http://192.168.56.11:8080 (admin / admin123)

---

## Kubernetes - Minikube (Problema 3)

Namespace: microapp

Deployments:
- users-deployment: 2 replicas
- products-deployment: 2 replicas
- orders-deployment: 4 replicas
- db-users, db-products, db-orders: 1 replica cada uno
- consul: 1 replica

### Inicializacion

    kubectl apply -f databases.yml
    kubectl apply -f consul.yml
    kubectl apply -f users.yml
    kubectl apply -f products.yml
    kubectl apply -f orders.yml
    kubectl apply -f ingress.yml

Acceso externo (requiere minikube tunnel en otra terminal):

    curl http://127.0.0.1/api/users
    curl http://127.0.0.1/api/products
    curl http://127.0.0.1/api/orders
