# Dynatrace Interview Questions (Concept + Scenario Based)

---

## 1. What is Dynatrace?
- Full-stack monitoring tool (APM, Infra, Logs, RUM)

---

## 2. What is OneAgent?
- Single agent that auto-discovers services, processes, dependencies

---

## 3. What is PurePath?
- End-to-end transaction tracing across services

---

## 4. What is Smartscape?
- Real-time topology visualization of apps, services, infra

---

## 5. What is Davis AI?
- AI engine for root cause analysis and anomaly detection

---

## 6. What is a Service in Dynatrace?
- Logical unit of application (API, backend, etc.)

---

## 7. What is a Process Group?
- Group of processes running same service

---

## 8. What is RUM?
- Real User Monitoring (frontend user experience)

---

## 9. What is Synthetic Monitoring?
- Simulated user tests (API/browser)

---

## 10. What are Metrics vs Logs vs Traces?
- Metrics → numbers (CPU, memory)
- Logs → events
- Traces → request flow

---

# 🔥 Scenario-Based Questions

---

## 11. Application is slow — how do you debug?
- Check service flow (PurePath)
- Identify slow DB/API calls
- Use Davis AI root cause

---

## 12. High CPU usage — what will you do?
- Identify process
- Check service consuming CPU
- Analyze code-level hotspots

---

## 13. One service is down — how to find root cause?
- Use Smartscape
- Check dependencies
- Davis AI problem analysis

---

## 14. API latency increased suddenly
- Compare baseline vs current
- Check backend services
- DB queries or external calls

---

## 15. Users complaining about slow UI
- Check RUM
- Identify slow frontend resources
- Check backend dependencies

---

## 16. No data in Dynatrace
- Check OneAgent status
- Verify network connectivity
- Check process detection

---

## 17. Kubernetes monitoring issue
- Check OneAgent DaemonSet
- Validate pod injection
- Check namespaces

---

## 18. Pod crash issue
- Check logs + metrics
- Analyze memory/CPU
- Use trace for failure

---

## 19. DB slow queries
- Use service flow
- Identify DB calls
- Optimize queries

---

## 20. Memory leak detection
- Monitor heap usage
- Analyze long-running processes
- Use code-level diagnostics

---

## 21. Deployment caused issue
- Compare before/after metrics
- Check version changes
- Rollback if needed

---

## 22. External API failure
- Trace outgoing calls
- Check response time/errors
- Validate endpoint availability

---

## 23. Alert tuning
- Adjust thresholds
- Use custom metrics
- Avoid false positives

---

## 24. SLA breach
- Analyze response time
- Check error rate
- Identify bottleneck service

---

## 25. Multi-tier app debugging
- Follow PurePath trace
- Identify slow layer
- Fix dependency issue

---

# 🔥 Important Areas

- Dynatrace dashboards
- Service flow
- Problems & events
- Metrics explorer
- Logs & traces

---

# 🚀 Interview One-Liner

"Dynatrace helps monitor applications end-to-end using OneAgent, provides full visibility with PurePath tracing, and uses Davis AI for automatic root cause analysis."
