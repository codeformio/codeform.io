+++
date = "2019-05-20T10:30:00-04:00"
draft = false
title = "Notes on the Google Cloud Architect Certification"
subtitle = ""
weight = 9997
page = "blog"

author = "Nick Stogner"
description = "The Google Cloud Architect certification is not like other cloud exams. It is intended to validate your experience as a software architect in general while also verifying that you can apply your knowledge to the Google Cloud Platform. Here is the path I took to getting certified."

tags = [
    "gcp",
    "certs",
    "career",
]

categories = [
    "Cloud",
    "Architecture",
]

+++

When Google Cloud launched their Kubernetes platform (GKE) back in 2015 I signed up for the free trial. Today my free credits are long gone, but I still run and launch new clusters on a weekly basis. After putting it off for a while, I recently decided to formalize my experience by getting the GCP Architect certification. In this post I detail some of the resources I used to prepare for the exam.

To my surprise, this exam is not just a test of concepts specific to Google's products. While a thorough understanding of GCP is required, the exam is also designed to assess your proficiency as a software architect in general. This makes giving study advice difficult since the topics are not just restrained to the GCP documentation. With that disclaimer, here is a list of some of the material I used to prepare for the exam.


## General Resources

These overviews and courses are a good starting point.

- [Certification Overview](https://cloud.google.com/certification/cloud-architect) - What are you getting yourself into?
- [Exam Guide/Outline](https://cloud.google.com/certification/guides/professional-cloud-architect/) - Bullet points of topics covered
- [Coursera Specialization](https://www.coursera.org/specializations/gcp-architecture) - Covers various GCP products (>1 month of material)
- [Coursera Exam Prep](https://www.coursera.org/learn/preparing-cloud-professional-cloud-architect-exam) - Gives you an overview of the exam and strategies for additional preparation
- [Practice Exam](https://cloud.google.com/certification/practice-exam/cloud-architect) - After completing, read the explanations to get an idea for the line of reasoning the authors take in their questions and solutions

## Miscellaneous Topics

The following topics are not comprehensive and they are in no particular order. I included them to provide an example of the depth/breadth of knowledge you will need to know.

#### Kubernetes

- [Creating clusters & running workloads](https://cloud.google.com/kubernetes-engine/docs/tutorials/hello-app)
- [Deploying / updating applications using `kubectl`](https://cloud.google.com/kubernetes-engine/docs/concepts/deployment)

#### Networking

- [VPN Concepts](https://cloud.google.com/vpn/docs/concepts/overview) (gateways & network topography)
- [Firewall concepts](https://cloud.google.com/vpc/docs/firewalls)

#### IAM

- How [resource hierarchies](https://cloud.google.com/iam/docs/resource-hierarchy-access-control) work with IAM

#### BigQuery

- Best practices around [schema design](https://cloud.google.com/solutions/bigquery-data-warehouse#designing_schema)
- [Table partitioning](https://cloud.google.com/bigquery/docs/partitioned-tables) and [data expiration](https://cloud.google.com/bigquery/docs/managing-partitioned-tables#partition-expiration)
- [Access control (roles) & how jobs are billed](https://cloud.google.com/bigquery/docs/jobs-overview)

#### Datastore

- [Best practices around querying](https://cloud.google.com/datastore/docs/best-practices) (multiple gets / batch get / query filters)
- [Index management](https://cloud.google.com/datastore/docs/concepts/indexes)

#### Cloud Storage

- [Methods of data transfer](https://cloud.google.com/solutions/transferring-big-data-sets-to-gcp) (what is considered a large dataset?)
- Uploading large amounts of data using a [Transfer Appliance](https://cloud.google.com/transfer-appliance/docs/2.0/overview) (familiarity with the data transfer process)
- [Object lifecycle management](https://cloud.google.com/storage/docs/lifecycle)
- [Using encryption keys](https://cloud.google.com/storage/docs/gsutil/addlhelp/UsingEncryptionKeys)

#### CI/CD

- High level familiarity with [spinnaker](https://www.spinnaker.io/) & [jenkins](https://jenkins.io/)

## Best of Luck!

Like most people in software I find myself a little suspicious of the value of most certifications. However I believe this particular certification is worth the time investment for anyone building on top of GCP. Preparing for the exam forced me fill in my GCP knowledge gaps and provided insight into the various use-cases that Google Cloud products are intended to solve.

<img src="https://api.accredible.com/v1/frontend/credential_website_embed_image/certificate/13043207" >

