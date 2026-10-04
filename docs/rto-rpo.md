# RTO and RPO

## Recovery Time Objective

**Target: 1 hour**

The recovery environment should restore the application workload and make it operational within one hour of a major disruption.

### Validation result

The Azure Site Recovery test failover job completed in **21 minutes and 2 seconds**. This result was within the stated RTO.

## Recovery Point Objective

**Target: 15 minutes**

The recovery solution should maintain a recovery point no more than approximately 15 minutes behind the production workload.

### Validation result

Azure Site Recovery reported an observed RPO of **4 minutes** while the protected VM was healthy.

## Interpretation

RTO answers: **How long can the service be unavailable?**

RPO answers: **How much recent data can be lost, measured in time?**
