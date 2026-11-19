# Proyecto Práctico: Despliegue Cloud, Seguridad y Troubleshooting

Este repositorio contiene los manifiestos de Kubernetes y la documentación técnica para el despliegue en Amazon Web Services (AWS), aplicando directivas de Hardening de Seguridad y resolución de incidencias.

---

## Estructura del Repositorio

```text
/
├── k8s/
│   ├── 00-namespace.yaml             # Definición del Namespace practica-s12
│   ├── 01-security-configmap.yaml    # ConfigMap de la aplicación
│   ├── 02-db-secret.yaml             # Secret con credenciales en Base64
│   ├── 03-app-deployment.yaml        # Deployment con SecurityContext y Sondas
│   └── 04-app-service.yaml           # Servicio expuesto (LoadBalancer / ELB)
├── docs/
│   └── troubleshooting-log.md        # Bitácora detallada de errores y soluciones
└── README.md                         # Documentación general del proyecto
```

---

## Instrucciones de Despliegue

1. **Crear Namespace:**
   ```bash
   kubectl apply -f k8s/00-namespace.yaml
   ```

2. **Aplicar Configuraciones y Secretos:**
   ```bash
   kubectl apply -f k8s/01-security-configmap.yaml
   kubectl apply -f k8s/02-db-secret.yaml
   ```

3. **Desplegar la Aplicación:**
   ```bash
   kubectl apply -f k8s/03-app-deployment.yaml
   ```

4. **Exponer el Servicio:**
   ```bash
   kubectl apply -f k8s/04-app-service.yaml
   ```

---

## Verificaciones de Seguridad y Salud

- **Verificar Pods:**
  ```bash
  kubectl get pods -n practica-s12
  ```

- **Verificar Usuario No Privilegiado (non-root):**
  ```bash
  kubectl exec -it <pod-name> -n practica-s12 -- whoami
  kubectl exec -it <pod-name> -n practica-s12 -- id
  ```

- **Obtener Endpoint Público:**
  ```bash
  kubectl get svc app-service -n practica-s12
  ```
