# EC2 Auto Scaling with Scheduled and Dynamic Scaling Policies

> This project was completed as a guided hands-on lab in AWS Skill Builder (SimuLearn).
> It demonstrates applied configuration of Amazon EC2 Auto Scaling in a simulated AWS environment,
> including an independent extension task (DIY Goal).

## Overview
This lab implements auto-healing and elastic capacity management for a fleet of game-server EC2
instances using an Auto Scaling group (ASG), a CloudWatch CPU alarm for dynamic scaling, and two
CloudWatch scheduled events for predictable daily demand cycles.

## Problem
A game-server workload experiences (a) unpredictable spikes in CPU load and (b) predictable
daily peak/off-peak usage windows. Static capacity either under-serves peak load or wastes spend
during low-usage hours.

## Architecture
- An Auto Scaling group is configured with **Minimum = 1** and **Maximum = 3** instances, launched
  from a Launch Template built off a custom AMI created from a running "Game server" instance.
- A CloudWatch alarm monitors CPU utilization; when usage stays below the 70% threshold the ASG
  is not scaled further, and above it a new instance is added (`CPU usage < 70%` → maintain, else scale out).
- Two scheduled scaling events adjust desired capacity at fixed UTC times (11:00 PM and 5:00 PM),
  independent of the CPU-based policy.

## AWS Services & Roles
| Service | Role |
|---|---|
| Amazon EC2 Auto Scaling | Maintains 1–3 game-server instances, replaces unhealthy instances |
| Amazon CloudWatch (Alarms) | Triggers dynamic scale-out based on CPU utilization |
| Amazon CloudWatch (Scheduled Actions) | Adjusts capacity at fixed daily times |
| Amazon Machine Image (AMI) | Custom image used to launch consistent game-server instances |
| EC2 Launch Template | Standardizes instance configuration for the ASG |

## Workflow
1. CloudWatch alarm evaluates average CPU utilization across the ASG.
2. If load is high, the ASG launches an instance from the Launch Template (sourced from the AMI).
3. If load is low, the ASG may terminate an instance, subject to Min=1.
4. Independently, scheduled events force desired-capacity changes at 5:00 PM and 11:00 PM UTC,
   overriding organic demand at those two points in the day.

## Reliability and Scalability
- Minimum capacity of 1 guarantees the service is never fully down (auto-healing: a terminated/
  unhealthy instance is automatically replaced up to the Min setting).
- Maximum capacity of 3 bounds cost exposure during demand spikes.
- Scheduled scaling reduces the lag between known demand changes and capacity changes, compared to
  relying on the CPU alarm alone.

## What I Implemented (Guided)
- Created the Auto Scaling group and attached EC2 instances to it.

## What I Implemented (DIY / Unguided)
- Configured an additional scheduled scaling policy to scale the group down to 0 resources at
  01:00 AM daily.

## Limitations / Not Documented
- Multi-AZ placement for the ASG: **Not documented / Requires clarification**.
- Health check type (EC2 vs ELB) and grace period: **Not documented / Requires clarification**.
- No load balancer is shown in this lab; traffic distribution mechanism to the 3 instances is
  **Not documented / Requires clarification**.

## Skills Demonstrated
EC2 Auto Scaling group configuration, Launch Templates, AMI creation, CloudWatch alarms, scheduled
scaling actions, basic capacity planning for variable workloads.

## Future Improvements
- Add an Application Load Balancer + target group health checks (as done in Project 3) to actually
  distribute traffic, not just scale headcount.
- Reproduce this configuration in Terraform in a personal AWS account for a fully IaC-backed repo.
