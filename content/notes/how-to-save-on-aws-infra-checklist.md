---
title: how to save on aws infra (checklist)
date: 2026-01-08T19:01:11+09:00
draft: false
summary:
tags:
---
i hate spending unnecessary money, especially on aws, since they are already too rich. so make sure you are doing everything on this list.

here are some of the things i did that saved me thousands of dollars on aws infra, from most savings to least.

1. right-sizing: make sure you do not waste compute or storage. the aws service for this is `compute optimizer`. it can recommend changes for ec2 instances, ebs volumes, lambda functions, and ecs services on fargate.

2. compute savings plans: if your compute usage is predictable, this is one of the easiest wins. you commit to an hourly amount of compute usage for 1 or 3 years in exchange for lower prices. you also do not have to pay everything up front, because aws offers all upfront, partial upfront, and no upfront options.

3. migrate to graviton: graviton is aws's arm64-based processor family. aws lets you compare the price and performance impact of moving supported workloads to graviton-based instances, so if your app and dependencies support arm64, this can cut compute cost fast.

4. automatically shut down staging: if your staging environment is ec2-based, stop leaving it on at night and on weekends when nobody is using it. aws systems manager quick setup can start and stop tagged ec2 instances on a schedule, which is a very boring and very effective way to stop wasting money.

## references

1. right-sizing: [aws compute optimizer supported resources](https://docs.aws.amazon.com/compute-optimizer/latest/ug/supported-resources.html)
2. compute savings plans: [what are savings plans?](https://docs.aws.amazon.com/savingsplans/latest/userguide/what-is-savings-plans.html)
3. graviton: [aws graviton-based instance recommendations](https://docs.aws.amazon.com/compute-optimizer/latest/ug/graviton-recommendations.html)
4. scheduled shutdowns: [stop and start ec2 instances automatically on a schedule using quick setup](https://docs.aws.amazon.com/systems-manager/latest/userguide/quick-setup-scheduler.html)
