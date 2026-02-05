
# Option 1 - Cloud NAT based egress, Akka Management access via public ingress with IP whitelisting

- Akka VPC `public egress` via Cloud NAT and Internet GW
  - Your Akka Services
  - Akka Management egress to Akka federation plane
- Akka VPC `public ingress`
  - Akka federation plane to Akka Management via public endpoint and IP whitelisting
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
            AKKA_MGMT[Akka Management]
        end
        CLOUD_NAT[Cloud NAT]
        PUBLIC_LB[Public IP/LB <br>IP whitelisting]
        INT_GW[Internet GW<br>0.0.0.0/0]
        AKKA_PEERING[Peering]
    end

   CLOUD_NAT --> |public<br>egress| INT_GW
   INT_GW --> |public<br>egress| AKKA_FED_PLAN
   INT_GW --> |public<br>egress| INT_SRVS
   SRVS --> |public<br>egress| CLOUD_NAT
   AKKA_MGMT --> |public<br>egress| CLOUD_NAT

   SRVS --> |private<br>egress| AKKA_PEERING
   AKKA_PEERING <--> CUST_PEERING

   ADMIN --> |public<br>ingress|HUB
   AKKA_FED_PLAN --> |public<br>ingress|PUBLIC_LB
   PUBLIC_LB --> |public<br>ingress|AKKA_MGMT

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
  - Your Akka Services and Akka Management with reverse tunneling
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
            AKKA_MGMT[Akka Management]
        end
        CLOUD_NAT[Cloud NAT]
        INT_GW[Internet GW<br>0.0.0.0/0]
        AKKA_PEERING[Peering]
    end

    AKKA_MGMT <--> |public<br>egress/ingress| CLOUD_NAT
    CLOUD_NAT --> |public<br>egress| INT_GW
    INT_GW --> |public<br>egress/ingress| AKKA_FED_PLAN
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

- **NO** Akka VPC `public egress`
- **NO** Akka VPC `public ingress`
- Your VPC Centralised HUB based `public ingress`
  - Akka CLI, External Users and Services, Akka federation plane
- Akka VPC `private egress` to Your VPC via peering
  - to internal services, internet and Akka federation plane
- Akka VPC `private ingress` from Your VPC via peering
  - from internal services/users and Akka federation plane



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
            AKKA_MGMT[Akka Management]
        end
        AKKA_PEERING[Peering]
    end

    AKKA_MGMT <-->  AKKA_PEERING
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
