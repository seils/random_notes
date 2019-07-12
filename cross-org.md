This example creates a DNS peering zone that forwards queries to a private zone in a separate organization and project.

Let's set some variables to start:

```
CONSUMER_PROJECT="dns-consumer"
CONSUMER_SA="dns-peer-consumer"
CONSUMER_VPC="consumer-vpc"
PRODUCER_PROJECT="dns-producer"
PRODUCER_VPC="producer-vpc"
```

The producer org/project is configured with two private zones:

```
$ gcloud beta dns managed-zones list
NAME              DNS_NAME                   DESCRIPTION  VISIBILITY
producer-gcp      producer.gcp.                           private
ptr-producer-gcp  50.10.10.10.in-addr.arpa.               private
```

First, we need to create a service account in the consumer org/project.  This is the account we'll have the producer org/project grant permissions to create the DNS peering relationship.

```
# Make sure we're in the right project
$ gcloud config set project $CONSUMER_PROJECT
Updated property [core/project].

$ gcloud iam service-accounts create $CONSUMER_SA
Created service account [dns-peering-consumer].
```

To save some typing later, let's capture to email address associated with the new service account:

```
CONSUMER_SA_EMAIL=$(gcloud iam service-accounts list --filter name:$CONSUMER_SA --format "value(email)")
```

Now we need to assign the service account the `DNS Admin` role in the consumer org/project.  This is required to create the peering zone in the consumer org/project.

```
$ gcloud beta projects add-iam-policy-binding $CONSUMER_PROJECT \
> --member serviceAccount:$CONSUMER_SA_EMAIL \
> --role "roles/dns.admin"

Updated IAM policy for project [dns-consumer].
bindings:
- members:
  - serviceAccount:service-111111111111@compute-system.iam.gserviceaccount.com
  role: roles/compute.serviceAgent
- members:
  - serviceAccount:dns-peer-consumer@dns-consumer.iam.gserviceaccount.com
  role: roles/dns.admin
- members:
  - serviceAccount:111111111111-compute@developer.gserviceaccount.com
  - serviceAccount:111111111111@cloudservices.gserviceaccount.com
  role: roles/editor
- members:
  - user:zachseils@google.com
  role: roles/owner
etag: BwWMUYp5JuY=
version: 1
```

At this point, provide the service account email to the producer org/project so they can grant the account the `DNS Peer` role in the org/project with the target private zones, e.g.:

```
# Switch to the producer project (obviously if the producer is really a different org, 
# someone on the producer side is executing this step
$ gcloud config set project $PRODUCER_PROJECT
Updated property [core/project].

$ gcloud beta projects add-iam-policy-binding $PRODUCER_PROJECT \
> --member serviceAccount:$CONSUMER_SA_EMAIL \
> --role "roles/dns.peer"
Updated IAM policy for project [dns-producer].
bindings:
- members:
  - serviceAccount:service-222222222222@compute-system.iam.gserviceaccount.com
  role: roles/compute.serviceAgent
- members:
  - serviceAccount:dns-peering-consumer@dns-consumer.iam.gserviceaccount.com
  role: roles/dns.peer
- members:
  - serviceAccount:222222222222-compute@developer.gserviceaccount.com
  - serviceAccount:222222222222@cloudservices.gserviceaccount.com
  role: roles/editor
- members:
  - user:zachseils@google.com
  role: roles/owner
etag: BwWMUZd764A=
version: 1
```

Once the producer org/project has granted the service account the `DNS Peer` role, you can create the peering zone in the consumer org/project.

```
# Make sure you are in the right project
$ gcloud config set project $CONSUMER_PROJECT
Updated property [core/project].

# Create peering zone in dns-consumer project
WARNING: This command is using service account impersonation. All API calls will be executed as [dns-peer-consumer@dns-consumer.iam.gserviceaccount.com].
Created [https://dns.googleapis.com/dns/v1beta2/projects/dns-consumer/managedZones/peer-producer-gcp].

```
