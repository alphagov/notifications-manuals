## WIP
This page should not be considered usable until the WIP marker above has been reviewed. 
This content is part of an on-going ticket and is incomplete and unreviewed. 

---
This page will cover the 3 likeliest scenarios that Concourse would need to be recovered:
1. AWS region is down
2. Concourse app itself is compromised
3. Concourse update destroys our settings and we cannot roll back


This document only covers deploying Concourse in the case of an `eu-west-2` outage, not other Notify infra, even though changes are required in `notifications-aws` too.
This sections assumes you know about `notifications-concourse-deployment` and `notifications-concourse`; how to use them and their purpose. A primer can be found here ([1](https://github.com/alphagov/notifications-manuals/wiki/Concourse#background), [2](https://github.com/alphagov/notifications-manuals/wiki/Concourse#background)).

## AWS Region is down
> **_WARNING:_**  If `us-east-1` is down then there is nothing we can do if changes are required for [various AWS services](https://docs.aws.amazon.com/whitepapers/latest/aws-fault-isolation-boundaries/global-services.html) which use `us-east-1`'s [control plane](https://docs.aws.amazon.com/whitepapers/latest/aws-fault-isolation-boundaries/global-services.html). This is a known and accepted risk.

Concourse instances run in `eu-west-2` by default and the buckets containing their state does too. Most of Notify's infra runs in this `eu-west-1`. 
It is assumed you already know the new region you will deploy into. The examples below assume you have chosen to deploy into `eu-west-1`.
### Create bootstrap infra
The first step is to manually create an S3 bucket to store terraform state in the `notify-deploy` account. This should be called `notify-deploy-tfstate-<new region-name>-<6 random characters>

Then update this value in both [notifications-concourse-deployment](https://github.com/alphagov/notifications-concourse-deployment/blob/59b0c3f1635d25b25e7184b2ebd352ca1bb280c9/terraform/deployments/concourse/site.tf#L29) and [notifications-concourse](https://github.com/alphagov/notifications-concourse/blob/a90e2b89ae20a0a7eef0baa083d7037fe841e8b1/pipelines/concourse-deployer.vars.yml#L41), where the current bucket name, `notify-deploy-tfstate`, is set. In `notifications-concourse-deployment` you must also update the region to your new one e.g. `eu-west-1`. 

```
backend "s3" {
    bucket = "notify-deploy-tfstate"
    # should be <deployment_name>.tfstate
    key    = "concourse.tfstate"
    region = "eu-west-2".  <- This becomes "eu-west-1"
  }
```


### notifications-concourse
You must update the region where it is set to `eu-west-2` in all locations.  

Apart from locations where the region is a comment e.g. `pipelines/concourse-deployer.vars.yml line 38`

You should merge these changes first.

### notifications-concourse-deployment
You must update the region where it is set to `eu-west-2` in all locations. 
You must not update the region for `resource "aws_ecr_replication_configuration" "cross_region_replication"`.

You must run `scripts/upload-credentials/upload-credentials.sh` so the expected credentials exist in the concourse env's (i.e. `concourse` or `staging-concourse`) SSM parameter store in the new region.

You should merge these changes second.

Once merged you should then manually create infra from the `notifications-concourse-deployment` repo with a second pair of eyes. 


