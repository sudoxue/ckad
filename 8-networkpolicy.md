# Network Policy (8%)
## Switch namespace to `ckad-network-policy`
## Ingress and Egress policy



1. Create a network policy to allow traffic from the 'Internal' application only to the 'payroll-service' and 'db-service'
Use the spec given on the right. You might want to enable ingress traffic to the pod to test your rules in the UI.

Policy Name: internal-policy
Policy Types: Egress
Egress Allow: payroll
Payroll Port: 8080
Egress Allow: mysql
MYSQL Port: 3306

2. Network policy question: On my exam, this is a troubleshooting problem. The pod cannot access service because of created network policy. So you might describe network policy then the correct label of the pods to fix it. If you understand the network policy, very quickly to finish the problem. Read more the task Declare Network Policy.


<details><summary>Show Solution</summary>
<p>
￼
```

```
</p>
</details>