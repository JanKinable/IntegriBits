# IntegriBits
Building blocks to compose integration functionality for communciation between **Applications**



# Motivation

## Deployment Cost Comparison: Single Host vs. Per-Subscription Deployment

### 1. Cost Components

#### **Compute (Container App Costs)**
- **Single Host Approach**:  
  - One container instance running (with scaling based on load).
  - **Cost Factors**: vCPU/memory usage, scaling requirements.
- **Per Subscription Deployment**:  
  - 30 separate instances running the same code, each processing the same messages.
  - Each instance incurs a **baseline cost**, even when idle.

💡 **Extra Cost Factor**:  
- Even with minimal CPU/memory allocations per instance, 30x deployments mean 30x the minimum running cost.  
- If using **pay-per-use** billing, each instance incurs cold start times, and the cost grows significantly with **always-on instances**.

#### **Networking (Service Bus & Ingress/Egress Costs)**
- **Service Bus Operations**:
  - **Single Host**: One instance pulls messages, reducing redundant Service Bus operations.
  - **Per Subscription**: 30 instances receiving identical messages → **30x more Service Bus operations**.
- **Outbound Data Transfer**:
  - If messages trigger API calls or data transfers, 30 deployments **multiply egress costs**.

💡 **Extra Cost Factor**:  
- Azure charges per million Service Bus operations. 30 subscriptions processing the same messages **30x the operations cost**.

#### **Storage & Logging Costs**
- **Single Host**: Centralized logs & state storage.
- **Per Subscription**: 30 separate logs, potentially redundant data storage.

💡 **Extra Cost Factor**:  
- More App Insights logs, distributed state storage, and configuration replication.

---

### 2. Estimated Cost Comparison

Assuming:
- **Container App (0.5 vCPU, 1GB RAM)**
- **Single Instance** scales up to 10 replicas (pay-per-use).
- **Per Subscription Deployment** runs 30x 0.5 vCPU instances.

| **Cost Component**   | **Single Host (1 instance, scaling to 10 replicas)** | **Separate Deployments (30 instances, same scaling)** |
|----------------------|-------------------------------------------------|-------------------------------------------------|
| **Compute (Container Apps, Pay-As-You-Go)** | ~$100–200/month (scaled use) | **$3,000+ (30x cost, minimum instance costs even when idle)** |
| **Service Bus Operations (assume $50/month baseline)** | ~$50 | **$1,500 (30x extra operations)** |
| **App Insights & Logs** | ~$30–50 | **$500+ (30x logging instances)** |
| **Total Estimated Cost** | **$180–300/month** | **$5,000+/month** |

---

### 3. Key Takeaways

🚀 **Why the cost is much higher for separate deployments?**
- **Compute baseline costs multiply by 30x**, even if each runs a minimal container.
- **Service Bus operations multiply**, as each instance pulls the same messages.
- **Observability & storage grow exponentially**, as logs/configs scale per instance.

🔧 **Potential Optimizations if Per Subscription Deployment is Required:**
- **Use a single deployment with multiple replica instances** instead of 30 separate ones.
- **Share a single Service Bus connection** (e.g., partition message handling).
- **Use autoscaling aggressively** (minimize always-on instances).
- **Consider Durable Functions instead of separate deployments** if state is required.

#### **Final Verdict**
❌ **A strict "one deployment per subscription" model is expensive and inefficient.**  
✅ **A hybrid model (grouped subscriptions or scaling replicas instead of new instances) balances cost, maintainability, and reliability.**  

## Cost Comparison: Container Apps vs Logic Apps (Standard)

### 1. Azure Logic Apps (Standard)
In the Standard plan, Logic Apps require selecting a **Workflow Standard hosting plan**, which offers different pricing tiers:

- **WS1**: 1 vCPU, 3.5 GB memory
- **WS2**: 2 vCPUs, 7 GB memory
- **WS3**: 4 vCPUs, 14 GB memory

Billing is based on reserved capacity, meaning you're charged for the allocated resources regardless of usage. For instance, in a sample region, the estimated monthly rates are:

- **WS1**: ~$175.16/month
- **WS2**: ~$350.33/month
- **WS3**: ~$700.65/month

These estimates are based on 730 hours per month and regional rates.

### 2. Azure Container Apps

Azure Container Apps offer flexible pricing with a **Dedicated plan**, charging based on vCPU and memory usage:

- **vCPU**: Billed per second
- **Memory**: Billed per GiB-second

Additionally, there's a base price for Dedicated plan management. The platform supports scaling to zero, meaning you only pay for active usage, optimizing costs during idle periods.

### 3. Service Bus Operations

Both solutions would interact with Azure Service Bus. Costs here depend on the number of operations, message sizes, and chosen pricing tier. It's essential to estimate the volume of messages and operations to determine this expense accurately.

### 4. Cost Comparison

Assuming you deploy 4 hosts:

- **Logic Apps (Standard)**: Selecting the WS1 tier for each host results in approximately $700.64/month (4 x $175.16).
- **Container Apps**: Costs depend on actual usage. With the ability to scale to zero, expenses could be lower during idle times. However, continuous high usage might lead to higher costs compared to the fixed pricing of Logic Apps.

### 5. Additional Considerations

- **Development and Maintenance**: Logic Apps offer a low-code environment, potentially reducing development time. Container Apps provide more control, suitable for complex scenarios but may require more maintenance.
  - Pesonal expirience: with Logic Apps the development cost quickly raises when trying to some custom things. When well designed code, Container Apps can reduce even more the development cost and whilst given the flexibility 
- **Scalability**: Container Apps offer flexible scaling, which can be cost-effective for variable workloads. Logic Apps have fixed resource allocations based on the selected tier.
- File size limitations: Logic App has a file size limitation of 100 MB, even with smaller files performance might be limited due to execution speed.

**Conclusion**

If your workloads are consistent and align with the resource allocations of Logic Apps' tiers, the Standard plan provides predictable pricing. For variable workloads requiring dynamic scaling, Azure Container Apps might offer cost benefits, especially with the ability to scale to zero during inactivity. 

Container Apps are the least expensive when well designed, keeping in mind the idle times, frequency calls

It's crucial to analyze your application's usage patterns and resource requirements to choose the most cost-effective solution.


=============

WIP: planning the approach
==========================

## Assumed
- Container Apps
- 
## Start with Receiving
- For sync communication: [integrate with APIM](https://learn.microsoft.com/en-us/azure/api-management/import-container-app-with-oas)
- For async communication: [route to servicebus](https://learn.microsoft.com/en-us/answers/questions/229490/how-to-route-traffic-to-servicebus-service-via-api)
  - handling of messages by container app [read](https://hexmaster.nl/posts/azure-conainer-apps-jobs-demo/)
=> Start working with connectors : servicebus subscriber, storageaccount, FTPs

## Think of type of message and file handlers



