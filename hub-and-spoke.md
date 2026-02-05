
# Option 1 - Cloud NAT based egress, Akka's K8S API access (for Akka's federation plane) via public ingress with IP whitelisting

- Akka VPC `public egress` via Cloud NAT and Internet GW
  - Your Akka Services and Akka's K8S API egress to Akka's federation plane
- Akka VPC `public ingress`
  - Akka's federation plane to Akka's K8S API with public endpoint and IP whitelisting
- Your VPC Centralised HUB based `public ingress`
  - Akka CLI, External Users and Services
- Akka VPC `private egress` to Your VPC via peering
  - to internal services
- Akka VPC `private ingress` from Your VPC via peering
  - from internal services/users

```mermaid
---
config:
  theme: mc
  look: classic
---
graph TB
    
    subgraph CUSTOMER_VPC[Your VPC]
        CUST_PEERING[Peering]
        HUB[Centralised HUB]
    end

    subgraph INT[Internet]
        AKKA_FED_PLAN[Akka Federation Plane]
        INT_SRVS[External<br>Services]
        USER[External<br>Users]
        ADMIN[Akka CLI]
    end

    subgraph AKKA_VPC[Akka VPC]
        subgraph AAO[Akka AAO]
            SRVS[Your Akka Services]
            K8S_API[Akka K8S API <br> public endpoint]
        end
        CLOUD_NAT[Cloud NAT]
        PUBLIC_LB[IP white listing]
        INT_GW[Internet GW<br>0.0.0.0/0]
        AKKA_PEERING[Peering]
    end

   CLOUD_NAT --> |public<br>egress| INT_GW
   INT_GW --> |public<br>egress| AKKA_FED_PLAN
   INT_GW --> |public<br>egress| INT_SRVS
   SRVS --> |public<br>egress| CLOUD_NAT
   K8S_API --> |public<br>egress| CLOUD_NAT

   SRVS --> |private<br>egress| AKKA_PEERING
   AKKA_PEERING <--> CUST_PEERING

   ADMIN --> |public<br>ingress|HUB
   AKKA_FED_PLAN --> |public<br>ingress|PUBLIC_LB
   PUBLIC_LB --> |public<br>ingress|K8S_API

   INT_SRVS --> |public<br>ingress|HUB
   USER --> |public<br>ingress|HUB
   HUB --> |private<br>ingress|CUST_PEERING
   AKKA_PEERING --> |private<br>ingress|SRVS

    
    style INT fill:lightblue,stroke:lightgray,stroke-width:3px,color:black
    style AKKA_VPC fill:lightgreen,stroke:lightgray,stroke-width:3px,color:black
    style CUSTOMER_VPC fill:lightsalmon,stroke:lightgray,stroke-width:3px,color:black
```

# Option 2 - Cloud NAT based egress, NO Akka VPC public ingress 

- Akka VPC `public egress` via Cloud NAT and Internet GW
  - Your Akka Services and Teleport gateway for Akka's K8S API (reverse tunneling)
- **NO** Akka VPC `public ingress`
- Your VPC Centralised HUB based `public ingress`
  - Akka CLI, External Users and Services
- Akka VPC `private egress` to Your VPC via peering
  - to internal services
- Akka VPC `private ingress` from Your VPC via peering
  - from internal services/users


```mermaid
---
config:
  theme: mc
  look: classic
---
graph TB

    subgraph CUSTOMER_VPC[Your VPC]
        CUST_PEERING[Peering]
        HUB[Centralised HUB]
    end

    subgraph INT[Internet]
        AKKA_FED_PLAN[Akka Federation Plane]
        INT_SRVS[External<br>Services]
        USER[External<br>Users]
        ADMIN[Akka CLI]
    end

    subgraph AKKA_VPC[Akka VPC]
        subgraph AAO[Akka AAO]
            SRVS[Your Akka Services]
            K8S_API[Akka K8S API <br> private endpoint]
        end
        CLOUD_NAT[Cloud NAT]
        TELEPORT[Teleport]
        INT_GW[Internet GW<br>0.0.0.0/0]
        AKKA_PEERING[Peering]
    end

    K8S_API <--> |public<br>egress/ingress| TELEPORT
    TELEPORT --> |public<br>egress| CLOUD_NAT
    CLOUD_NAT --> |public<br>egress| INT_GW
    INT_GW --> |public<br>egress| AKKA_FED_PLAN
    INT_GW --> |public<br>egress| INT_SRVS
    SRVS --> |public<br>egress| CLOUD_NAT

    SRVS --> |private<br>egress| AKKA_PEERING
    AKKA_PEERING <--> CUST_PEERING

    ADMIN --> |public<br>ingress|HUB

    INT_SRVS --> |public<br>ingress|HUB
    USER --> |public<br>ingress|HUB
    HUB --> |private<br>ingress|CUST_PEERING
    AKKA_PEERING --> |private<br>ingress|SRVS

    style INT fill:lightblue,stroke:lightgray,stroke-width:3px,color:black
    style AKKA_VPC fill:lightgreen,stroke:lightgray,stroke-width:3px,color:black
    style CUSTOMER_VPC fill:lightsalmon,stroke:lightgray,stroke-width:3px,color:black
```

# Option 3 - No Public egress or ingress

- Akka VPC `public egress` via Your VPC Centralised HUB and peering
- Akka VPC `public ingress`
    - External Users and Services via Your VPC Centralised HUB and peering
- Akka VPC `private egress` to Your VPC via peering
- Akka VPC `private ingress` from Your VPC via peering

- **NO** Akka VPC `public egress`
- **NO** Akka VPC `public ingress`
- Your VPC Centralised HUB based `public ingress`
  - Akka CLI, External Users and Services, Akka's Federation Plane
- Akka VPC `private egress` to Your VPC via peering
  - to internal services, internet and Akka's Federation Plane
- Akka VPC `private ingress` from Your VPC via peering
  - from internal services/users and Akka's Federation Plane



```mermaid
---
config:
  theme: mc
  look: classic
---
graph TB

    subgraph CUSTOMER_VPC[Your VPC]
        CUST_PEERING[Peering]
        HUB[Centralised HUB]
    end

    subgraph INT[Internet]
        AKKA_FED_PLAN[Akka Federation Plane]
        INT_SRVS[External<br>Services]
        USER[External<br>Users]
        ADMIN[Akka CLI]
    end

    subgraph AKKA_VPC[Akka VPC]
        subgraph AAO[Akka AAO]
            SRVS[Your Akka Services]
            K8S_API[Akka K8S API <br> private endpoint]
        end
        AKKA_PEERING[Peering]
    end

    K8S_API <-->  AKKA_PEERING
    HUB <--> AKKA_FED_PLAN
    HUB <--> INT_SRVS

    SRVS <--> AKKA_PEERING
    AKKA_PEERING <--> CUST_PEERING

    ADMIN --> HUB

    USER --> HUB
    HUB <--> CUST_PEERING

    style INT fill:lightblue,stroke:lightgray,stroke-width:3px,color:black
    style AKKA_VPC fill:lightgreen,stroke:lightgray,stroke-width:3px,color:black
    style CUSTOMER_VPC fill:lightsalmon,stroke:lightgray,stroke-width:3px,color:black
```
