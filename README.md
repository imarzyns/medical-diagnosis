# Medical Diagnosis Validated Pattern

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[Live build status](https://validatedpatterns.io/ci/?pattern=medicaldiag)


## On-Prem customization

Instead of external S3 bucket we set up ceph rgw object storage. To communicate with its API user can utilize aws-cli. Installation instruction: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html#getting-started-install-instructions

Create initial object bucket to store xray images and copy images from 'base_images' to the bucket: 
```
export AWS_ACCESS_KEY_ID=xxxx
export AWS_SECRET_ACCESS_KEY=xxx

aws --endpoint CEPH_RGW_ENDPOINT --no-verify-ssl s3api create-bucket --bucket CEPH_BUCKET_NAME
aws --endpoint CEPH_RGW_ENDPOINT --no-verify-ssl s3 cp base_images/ s3://CEPH_BUCKET_NAME/ --recursive
aws --endpoint CEPH_RGW_ENDPOINT --no-verify-ssl s3api put-bucket-policy --bucket CEPH_BUCKET_NAME --policy file://PATH_TO_BUCKET_POLICY/bucket-policy.json
```

User can get CEPH_RGW_ENDPOINT by executing command:
```
oc -n openshift-storage get route ocs-storagecluster-cephobjectstore -ojson | jq '.spec.host'
```

Ceph rgw bucket needs specific bucket policy to be applied 'bucket-policy.json':
```
{
"Statement": [
                {
                "Sid": "listobjs",
                "Effect": "Allow",
                "Principal": "*",
                "Action": "s3:*",
                "Resource": "*"
                }
                ]
}
```

Base images can be found on original repo: https://github.com/red-hat-data-services/jumpstart-library/tree/main/demo1-xray-pipeline/base_elements/containers/image-init/base_images
AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY can be found in OCS Secret: 'rook-ceph-object-user-ocs-storagecluster-cephobjectstore-xraylab-1' in 'openshift-storage' namespace.

Required fields to fill are in 'values-global.yaml' set as blank fields: "".


### Cluster HW requirements

On-prem version was tested with 3 master and 3 worker nodes: 
* 12 vCPU each (for future 36 vCPU per worker may be required)
* 40 GB memory
* 100 GB disk space for OS
* Storage: 400 GB



## XRay analysis automated pipeline

This Validated Pattern is based on a demo implemetation of an automated data pipeline for chest Xray
analysis previously developed by Red Hat.  The original demo can be found [here](https://github.com/red-hat-data-services/jumpstart-library]).

The Validated Pattern includes the same functionality as the original demonstration.  The difference is
that we use the *GitOps* to deploy most of the components which includes operators, creation of namespaces,
and cluster configuration.

The Validated Pattern includes:

* Ingest chest Xrays into an object store based on Ceph.
* The Object store sends notifications to a Kafka topic.
* A KNative Eventing Listener to the topic triggers a KNative Serving function.
* An ML-trained model running in a container makes a risk of Pneumonia assessment for incoming images.
* A Grafana dashboard displays the pipeline in real time, along with images incoming, processed and anonymized, as well as full metrics.

This pipeline is showcased [in this video](https://www.youtube.com/watch?v=zja83FVsm14).

![Pipeline dashboard](doc/dashboard.png)

## Check the values files before deployment

You can run a check before deployment to make sure that you have the required variables to deploy the
Medical Diagnosis Validated Pattern.

You can run `make predeploy` to check your values. This will allow you to review your values and changed them in
the case there are typos or old values.  The values files that should be reviewed prior to deploying the
Medical Diagnosis Validated Pattern are:

| Values File | Description |
| ----------- | ----------- |
| values-secret.yaml | This is the values file that will include the xraylab section with all the database secrets |
| values-global.yaml | File that is used to contain all the global values used by Helm |

Make sure you have the correct domain, clustername, externalUrl, targetBucket and bucketSource values.

[![asciicast](https://github.com/claudiol/medical-diagnosis/blob/claudiol-xray-deployment/doc/predeploy.svg)](https://github.com/claudiol/medical-diagnosis/blob/claudiol-xray-deployment/doc/predeploy.svg)

Then you can run `make install` to deploy the Medical Diagnosis Validated Pattern.

[![asciicast](https://github.com/claudiol/medical-diagnosis/blob/claudiol-xray-deployment/doc/xray-deployment.svg)](https://github.com/claudiol/medical-diagnosis/blob/claudiol-xray-deployment/doc/xray-deployment.svg)

This validated pattern is still being developed.  More to come in the next few weeks. Any questions or concerns
please contact [Jonny Rickard](jrickard@redhat.com) or [Lester Claudio](claudiol@redhat.com).
