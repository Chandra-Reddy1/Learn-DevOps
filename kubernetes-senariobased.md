# Kubernetes Interview Scenarios (Top 20)

---

## 1. Pod stuck in Pending
- Check: kubectl describe pod
- Reason: insufficient resources / node selector mismatch

---

## 2. CrashLoopBackOff
- Check logs: kubectl logs <pod>
- Fix: application error / wrong command

---

## 3. ImagePullBackOff
- Cause: wrong image / auth issue
- Fix: correct image or add imagePullSecrets

---

## 4. Pod running but not accessible
- Check Service + Endpoints
- Verify labels match

---

## 5. Service not routing traffic
- kubectl get endpoints
- No endpoints = selector mismatch

---

## 6. Ingress not working
- Check Ingress controller
- Verify rules + host

---

## 7. DNS resolution failure
- Test inside pod: nslookup service-name
- Check CoreDNS

---

## 8. Rollout stuck
- kubectl rollout status deployment
- Cause: readiness probe failure

---

## 9. High CPU but no scaling
- Check HPA config
- Metrics server issue

---

## 10. Pod OOMKilled
- Increase memory limits
- Optimize app usage

---

## 11. Node NotReady
- kubectl describe node
- Check kubelet / disk / network

---

## 12. Config not updating
- Restart pod
- Check ConfigMap mount

---

## 13. Secret not working
- Verify base64 encoding
- Check env reference

---

## 14. Wrong container port
- Service targetPort mismatch
- Fix port mapping

---

## 15. Multiple pods but uneven traffic
- Check kube-proxy mode
- Session affinity issue

---

## 16. Deployment rollback needed
- kubectl rollout undo deployment

---

## 17. Pod scheduling issue
- Check taints/tolerations
- Check affinity rules

---

## 18. Storage not mounting
- Check PVC status
- Verify storage class

---

## 19. Slow application
- Check resource limits
- Check node pressure

---

## 20. Full app flow debugging

Flow:
User → DNS → LoadBalancer → Ingress → Service → Pod

Check each layer step-by-step
