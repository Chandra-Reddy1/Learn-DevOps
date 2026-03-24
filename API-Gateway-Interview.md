# AWS API Gateway Interview Questions & Answers (Top 20)

---

## 1. What is AWS API Gateway?

AWS API Gateway is a fully managed service to create, publish, and manage APIs at scale.

---

## 2. What are types of APIs in API Gateway?

- REST API
- HTTP API (lightweight, cheaper)
- WebSocket API (real-time communication)

---

## 3. What is the difference between REST API and HTTP API?

- REST API → more features (auth, transformation)
- HTTP API → lower cost, simpler, faster

---

## 4. What is an endpoint type?

- Edge-optimized → global users via CloudFront
- Regional → specific region
- Private → accessible only within VPC

---

## 5. What is integration in API Gateway?

Defines how API Gateway connects to backend:
- Lambda
- HTTP endpoint
- AWS services

---

## 6. What is Lambda proxy integration?

Passes full request to Lambda without transformation.

---

## 7. What is mapping template?

Used to transform request/response before sending to backend.

---

## 8. What is throttling?

Limits number of API requests to prevent overload.

---

## 9. What is rate limit vs burst limit?

- Rate → requests per second
- Burst → short spike capacity

---

## 10. What is API Gateway caching?

Stores responses to reduce backend calls and improve performance.

---

## 11. What is authorization in API Gateway?

- IAM
- Cognito User Pool
- Lambda Authorizer (custom)

---

## 12. What is CORS?

Cross-Origin Resource Sharing allows frontend apps from different domains to call APIs.

---

## 13. What is stage in API Gateway?

Environment like:
- dev
- test
- prod

---

## 14. What is deployment in API Gateway?

Publishing API changes to a stage.

---

## 15. What is API key?

Used to track and control API usage.

---

## 16. What is usage plan?

Defines throttling and quota for API keys.

---

## 17. What is private API?

API accessible only inside VPC using VPC endpoint.

---

## 18. How does API Gateway handle errors?

- Returns HTTP status codes
- Custom error mapping possible

---

## 19. What is CloudWatch role in API Gateway?

- Logs requests/responses
- Monitors metrics

---

## 20. What is request/response transformation?

Modify data using mapping templates before sending/returning.

---

# Bonus (High-Impact DevOps Questions)

---

## 21. How to secure API Gateway?

- Use IAM / Cognito / Lambda Authorizer
- Enable HTTPS
- Use WAF
- Throttle requests

---

## 22. How does API Gateway integrate with Lambda?

- API Gateway receives request
- Invokes Lambda
- Returns Lambda response

---

## 23. How to handle high traffic?

- Enable caching
- Use throttling
- Use CloudFront
- Optimize backend (Lambda concurrency)

---

## 24. Difference between API Gateway and Load Balancer?

- API Gateway → API management, auth, throttling
- ALB → traffic distribution

---

## 25. What is custom domain in API Gateway?

Map your own domain (e.g., api.example.com) to API Gateway.

---

## 26. What is stage variable?

Variables used to configure different environments.

---

## 27. How to deploy API Gateway using DevOps tools?

- Terraform
- CloudFormation
- CI/CD pipelines

---

## 28. What is WebSocket API use case?

Real-time apps (chat, notifications)

---

## 29. What is request validation?

Validates input before sending to backend.

---

## 30. What is integration timeout?

Maximum time API Gateway waits for backend response (~29 sec).

---

# API Gateway + Lambda Flow (Interview Explanation)

---

## Scenario
Build a serverless backend API without managing servers.

---

## Architecture Overview

Client → API Gateway → Lambda → Response

---

## Step-by-Step Flow

### 1. Client Request
- Client (browser/mobile app) sends HTTP request
- Example: GET /users

---

### 2. API Gateway Receives Request
- API Gateway acts as entry point
- Handles:
  - Authentication (IAM / Cognito / Authorizer)
  - Throttling
  - Request validation

---

### 3. API Gateway → Lambda
- API Gateway invokes Lambda function
- Uses:
  - Lambda Proxy Integration (common)
- Entire request (headers, body, params) passed to Lambda

---

### 4. Lambda Execution
- Lambda processes request:
  - Business logic
  - DB calls (RDS/DynamoDB)
  - S3 interaction (if needed)

---

### 5. Lambda Response
- Lambda returns response:
```json
{
  "statusCode": 200,
  "body": "Success"
}
