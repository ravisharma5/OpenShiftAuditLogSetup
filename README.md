# OpenShiftAuditLogSetup
Repository with instructions on how to set Audit log forwarding to S3 from OpenShift

We will use LokiStack with S3 for storing logs from OCP.

First, we will configure the logging stack in OpenShift with LokiStack:
- ClusterLogging CR
- LokiStack

Then we will configure the Log retention policy as well as Log forwarding using:
- ClusterLogForwarder

### Steps:

#### 1. Assign your user to cluster-admin or dedicated-admin group
This step is required even if you are an Admin user to see logs in the console. Without this step, you might see an issue mentioned here: https://access.redhat.com/solutions/7018952

```
$ oc adm groups new dedicated-admin
$ oc adm groups add-users dedicated-admin username
```
#### 2. Deploy RedHat OpenShift Logging Operator
This will create an `openshift-logging` project for you. Once this operator is installed we will create an Instance of `ClusterLogging` CR with LokiStack in step #4.

Under the `Procedure` follow subsection #2 here:https://docs.openshift.com/container-platform/latest/logging/cluster-logging-deploying.html#cluster-logging-deploy-console_cluster-logging-deploying
#### 3. Create a S3 bucket
S3 Bucket which is going to store your logs.
```
$ S3_NAME="auditlog-bucket"
$ AWS_REGION="us-east-2"
$ AWS_KEY=$(aws configure get aws_access_key_id)
$ AWS_SECRET=$(aws configure get aws_secret_access_key)

$ aws s3api create-bucket --bucket $S3_NAME  --region $AWS_REGION --create-bucket-configuration LocationConstraint=$AWS_REGION
```
#### 4. Install LokiStack and ClusterLogging CR
Steps to install LokiStack and ClusterLogging CR are described here: https://docs.openshift.com/container-platform/latest/logging/cluster-logging-loki.html#logging-loki-deploy_cluster-logging-loki

Once you have followed all the steps you should be able to see `Logs` in the console under the `Observe` section.

**NOTE**: This step will enable Infrastructure and Application logs, but you will need to follow step #5 for the Audit log.
#### 5. Forwarding Audit logs using ClusterLogForwarder CR
Use the steps described here to forward Audit logs: https://docs.openshift.com/container-platform/latest/logging/config/cluster-logging-log-store.html#cluster-logging-elasticsearch-audit_cluster-logging-store

Since our default or internal log store is LokiStack, all the logs described in the pipeline will get forwarded to the Loki instance.

```yaml
apiVersion: logging.openshift.io/v1
kind: ClusterLogForwarder
metadata:
  name: instance
  namespace: openshift-logging
spec:
  pipelines: 
  - name: all-to-default
    inputRefs:
    - infrastructure
    - application
    - audit
    outputRefs:
    - default
```

#### 6. Filtering Audit Logs
This is an example of how to enable filtering for Audit logs: https://github.com/alanconway/cluster-logging-operator/blob/api-audit-filter/docs/features/logforwarding/filters/api-audit-filter.adoc




