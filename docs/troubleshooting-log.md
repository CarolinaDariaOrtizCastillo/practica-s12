# Bitácora de Troubleshooting y Resolución de Problemas

**Proyecto:** Despliegue Cloud, Seguridad y Troubleshooting  
**Namespace:** `practica-s12`  
**Entorno:** Amazon Web Services (EKS / Elastic Kubernetes Service)  

---

## Escenario A: Falla de Configuración y Hardening (CrashLoopBackOff)
- **Descripción del problema:** Al desplegar inicialmente la aplicación aplicando las directivas de seguridad `runAsUser: 10001` y `readOnlyRootFilesystem: true`, el Pod `app-deployment` entró en estado continuo de `CrashLoopBackOff` (0/1 Ready).
- **Diagnóstico:**
  - Comandos ejecutados:
    ```bash
    kubectl get pods -n practica-s12
    kubectl describe pod app-deployment-699678f99f-lslwg -n practica-s12
    kubectl logs -n practica-s12 -l app=app-backend
    ```
  - Hallazgos: La imagen estándar requería privilegios root para escribir en `/var/cache/nginx`, `/var/run` y enlazar el puerto privileged 80. Con el sistema de archivos de solo lectura y el UID `10001`, el proceso se bloqueaba por falta de permisos.
- **Solución aplicada:**
  1. Se utilizó una imagen no privilegiada (`nginxinc/nginx-unprivileged:alpine`).
  2. Se configuró un volumen temporal tipo `emptyDir` montado en `/tmp` para permitir la escritura en runtime requerida por la aplicación.
  3. Se ajustó el puerto interno del contenedor a `8080`.
- **Resultado:** El Pod inició inmediatamente y pasó a estado `Running` (1/1) sin reinicios.

---

## Escenario B: Falla de Red y Conectividad (Backend ↔ Servicio)
- **Descripción del problema:** Tras mover el contenedor al puerto no privilegiado `8080`, el servicio balanceador `app-service` no lograba enviar tráfico a los pods backend.
- **Diagnóstico:**
  - Comandos ejecutados:
    ```bash
    kubectl describe svc app-service -n practica-s12
    ```
  - Hallazgos: El `Service` mantenía como `targetPort: 80`, el cual ya no escuchaba tráfico dentro del contenedor seguro.
- **Solución aplicada:**
  - Se modificó `04-app-service.yaml` actualizando el `targetPort: 8080` para enrutar el tráfico externo del puerto 80 del LoadBalancer hacia el puerto interno real del Pod.
- **Resultado:** Conectividad restablecida exitosamente y tráfico enrutado correctamente.

---

## Escenario C: Falla en Sondas de Salud (Liveness y Readiness Probes)
- **Descripción del problema:** Riesgo de reinicios constantes si las sondas `livenessProbe` y `readinessProbe` apuntan al puerto erróneo.
- **Diagnóstico:**
  - Comandos ejecutados:
    ```bash
    kubectl describe pod -n practica-s12
    ```
  - Hallazgos: Las sondas HTTP continuaban sondeando el puerto 80, provocando fallas de chequeo de salud y demoras en el estado `Ready`.
- **Solución aplicada:**
  - Se ajustaron `livenessProbe` y `readinessProbe` en `03-app-deployment.yaml` para realizar el `httpGet` en la ruta `/` en el puerto `8080` con tiempos `initialDelaySeconds: 5` y `periodSeconds: 10`.
- **Resultado:** Sondas respondiendo con código HTTP 200, validando el pod como `Ready` de forma estable.
