# Backend best practices & recommendations

## Logging policy
1. Backend logs any error that eventually causes 5xx response status (500, 503, etc).
2. Backend never logs errors that caused 4xx response statuses.
   1. `Why:` to decrease log service costs and decrease amount of logs that causes a lot of noise (401).