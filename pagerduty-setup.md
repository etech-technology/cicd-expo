# Configuring PagerDuty for Incident Management

## **Step 1: Create a PagerDuty Account**
1. Go to [PagerDuty's website](https://www.pagerduty.com/) and sign up.
2. Complete the initial setup by defining your **team, services, and escalation policies**.

---

## **Step 2: Set Up a Service in PagerDuty**
A service in PagerDuty represents an application or infrastructure component you want to monitor.

1. Navigate to **Services** → **Service Directory**.
2. Click **+ New Service**.
3. Provide a **Service Name** (e.g., "Production Monitoring").
4. Select an **Integration Type** (e.g., Kubernetes, AWS CloudWatch, Prometheus, Zabbix).
5. Set an **Escalation Policy** (who gets alerted first, and who gets escalated).
6. Click **Create Service**.

---

## **Step 3: Add Integrations**
1. Select the newly created service.
2. Click **Integrations** → **Add Integration**.
3. Choose an integration (e.g., Kubernetes, Prometheus, AWS CloudWatch, Nagios).
4. Copy the **Integration Key** provided.

---

## **Step 4: Configure Your Application Pods for PagerDuty Alerts**
### **Using Kubernetes with PagerDuty**
To integrate PagerDuty directly with your Kubernetes application pods, follow these steps:

1. **Deploy the PagerDuty Agent** in your cluster:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pagerduty-agent
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: pagerduty-agent
  template:
    metadata:
      labels:
        app: pagerduty-agent
    spec:
      containers:
      - name: pagerduty-agent
        image: pagerduty/agent:latest
        env:
        - name: PAGERDUTY_INTEGRATION_KEY
          value: "<YOUR_INTEGRATION_KEY>"
```
2. **Update Your Application Deployment to Include Alert Annotations:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app-pod
  annotations:
    pagerduty.com/alert: "true"
    pagerduty.com/integration-key: "<YOUR_INTEGRATION_KEY>"
spec:
  containers:
  - name: my-app
    image: my-app-image:latest
```
3. **Deploy and Verify the Integration:**
   ```sh
   kubectl apply -f pagerduty-agent.yaml
   kubectl apply -f my-app-deployment.yaml
   ```
4. **Trigger a Test Alert from the Application Logs or Errors**
   ```sh
   kubectl logs my-app-pod | grep "error"
   ```

---

## **Step 5: Configure On-Call Schedules**
1. Go to **People** → **On-Call Schedules**.
2. Click **+ New Schedule**.
3. Add team members and define rotation times.
4. Save and assign the schedule to an escalation policy.

---

## **Step 6: Test and Validate Alerts**
1. Trigger an alert from your application logs or errors.
2. Check if PagerDuty receives it and notifies the right person.
3. Acknowledge, escalate, or resolve the incident via PagerDuty’s dashboard or mobile app.

---

## **Best Practices**
- Use **Slack or Microsoft Teams** integration for real-time collaboration.
- Set up **Auto-Resolution** for alerts to close incidents automatically.
- Configure **Runbooks** so responders know how to troubleshoot.
- Use **Event Rules** to filter and categorize alerts efficiently.

---

By following these steps, you can efficiently integrate PagerDuty directly with your Kubernetes application pods, ensuring smooth and effective alert management.

