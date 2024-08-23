# IBM Maximo Application Suite and Cloud Pak for Data Deployment Tools

You can deploy IBM [Maximo Application Suite](https://www.ibm.com/docs/en/mas-cd/continuous-delivery?topic=overview-deployment-options-maximo-application-suite) (MAS) and [Cloud Pak for Data](https://www.ibm.com/docs/en/cloud-paks/cp-data/5.0.x?topic=installing) (CP4D) using Ansible Playbooks and CLI. 

This document describes a few custom tools that you can use during MAS and CP4D deployment.

## List MAS superuser account

You can look up the default MAS admin account by search the superuser secret in OpenShift. Optionally, you can run the [playbook](docs/list_mas_account.yml) to get the admin account after logging in to the OpenShift cluster.

```
oc login --token=xxx --server=xxx:6443

export MAS_INSTANCE_ID=inst1
export PROJECT_CPD_INST_OPERANDS=ibm-cpd

ansible-playbook list_mas_account.yml
```

The output looks like below.

```
TASK [Maximo Application Suite Authentication Summary:] ****************************************************************
ok: [localhost] => {
    "msg": [
        "Maximo Application Suite is Ready, use the superuser credentials to authenticate",
        "Admin Dashboard ... https://admin.inst1.apps.xxx.com",
        "Username .......... fBmYeTEy3tgY5h7730XE7qBpLk52guse",
        "Password .......... TU6ap2fTh1n7YN2KsHwLm1DuRx8rbr31"
    ]
}
```

## List MAS application resources

When you deploy MAS applications such as MAS Manage with playbooks, you wait for application and workspace to be ready. If you are curious of what application resources are and what each resource status is, you can use the [playbook](docs/list_mas_status.yml), `list_mas_status` to obtain the information.

```
# change mas_devops_path to point to your local mas devops repo in the yaml file
export MAS_INSTANCE_ID=inst1
export MAS_APP_ID=manage
export MAS_WORKSPACE_ID=masdev

ansible-playbook list_mas_status.yml
```

The output looks like below. The text in the `conditions` line with values like `'status': 'True', 'type': 'Ready'` is important because the playbooks evaluate them.

```
TASK [Show mas app resource status] ************************************************************************************
ok: [localhost] => 
{
    "msg": 
    [
        "CP4D is Ready, use the admin credentials to authenticate",
        "Retries ... 30",
        "Delays ... 120",
        "Resources ... 1",
        "Resources ... 
        {
        'components': {'manage': {'enabled': True, 'version': '9.0.0'}}, 
        'conditions': [
            {'lastTransitionTime': '2024-08-22T03:23:17Z', 'message': 'Controller updated primary entity manager to supported version (9.0.0)', 'reason': 'VersionUpdateCompleted', 'status': 'True', 'type': 'ControllerHealth'}, {'lastTransitionTime': '2024-08-22T03:23:17Z', 'message': '', 'reason': '', 'status': 'False', 'type': 'Failure'}, {'lastTransitionTime': '2024-08-22T03:32:23Z', 'message': 'Ready to enable workspace', 'reason': 'Ready', 'status': 'True', 'type': 'Ready'}, 
            {'ansibleResult': 
             {'changed': 0, 'completion': '2024-08-23T15:47:38.225116', 'failures': 0, 'ok': 68, 'skipped': 21}, 'lastTransitionTime': '2024-08-22T03:22:29Z', 'message': 'Awaiting next reconciliation', 'reason': 'Successful', 'status': 'True', 'type': 'Running'}, 
            {'lastTransitionTime': '2024-08-23T15:47:39Z', 'message': 'Last reconciliation succeeded', 'reason': 'Successful', 'status': 'True', 'type': 'Successful'}
        ], 
        'milestones': {'installed': 
                       {'anonymousId': '77098a3f-29d3-4222-bf2c-ff0d62e00c4e', 'timestamp': '2024-08-22T03:23:12.962064+00:00', 'transactionId': 'TransactionID Not Applicable for DRO'}
                       }, 
        'podTemplates': [], 
        'version': {'reconciled': ''}, 
        'versions': {'controller': '9.0.0', 
                     'isRollbackRunning': False, 
                     'reconciled': '9.0.0', 
                     'supported': ['8.7.7', '9.0.0']
                     }
        }"
    ]
}
```

## List CP4D account

Similar to MAS deployment, you can look up the default CP4D admin account in OpenShift. Optionally, you can run the [playbook](docs/list_cp4d_account.yml) to get the admin or cpadmin credentials.

```
export PROJECT_CPD_INST_OPERANDS=ibm-cpd
export CPD_SECRET_NAM=admin-user-details
export CPD_ROUTE_NAME=cpd
export CPD_ADMIN_USERNAME=admin

ansible-playbook list_cp4d_account.yml
```

The output looks like the below. Notice that the playbook shows the admin account twice. The reason is that when cpadmin is the default account, you can ignore the first part and use the second part. 

```
TASK [CP4D Authentication Summary - When IAM Is Enabled:] **************************************************************
ok: [localhost] => {
    "msg": [
        "CP4D is Ready, use the admin credentials to authenticate",
        "Admin Dashboard ... https://cpd-ibm-cpd.xxx.com",
        "Username .......... admin",
        "Password .......... lArDJ0RcbhGX"
    ]
}
```

## Check CP4D versions

How do you find out the exact version of CP4D that has been deployed? You can log in to CP4D and check the version in the About menu at the upper right corner. But you can only find the major version.

```
IBM Cloud Pak for Data
Version
v4.8
```

To find the precise version of CP4D, you can download and run the [cpd-cli](https://github.com/IBM/cpd-cli/releases) tool, after logging in with `ocp-cli manage login-to-ocp` command line, not `oc login`.

```
#oc login --token=xxx --server=xxx:6443

export OCP_URL=https://api.xxx.com:6443
export OCP_TOKEN=xxx
export SERVER_ARGUMENTS="--server=${OCP_URL}"
export LOGIN_ARGUMENTS="--token=${OCP_TOKEN}"

./cpd-cli manage login-to-ocp ${SERVER_ARGUMENTS} ${LOGIN_ARGUMENTS}"

./cpd-cli version

cpd-cli
	Version: 14.0.1
	Build Date: 2024-07-24T14:46:43
	Build Number: 396
	CPD Release Version: 5.0.1
```

To find more details on the versions, run the command line below.

```
oc login --token=xxx --server=xxx:6443
export PROJECT_CPD_INST_OPERANDS=ibm-cpd

./cpd-cli manage get-cr-status \ --cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS}

[INFO] Output the result in the below chart:

Component         CR-kind        CR-name         Namespace    Status     Version    Creationtimestamp     Reconciled-version    Operator-info
----------------  -------------  --------------  -----------  ---------  ---------  --------------------  --------------------  -----------------------------------
cpfs              CommonService  common-service  ibm-cpd      Succeeded  N/A        2024-08-08T20:42:02Z  N/A                   N/A
zen               ZenService     lite-cr         ibm-cpd      Completed  5.1.0      2024-08-08T20:47:29Z  5.1.0                 zen operator 5.1.0 build 126
cpd_platform      Ibmcpd         ibmcpd-cr       ibm-cpd      Completed  4.8.0      2024-08-08T20:45:23Z  N/A                   cpdPlatform operator 5.0.0 build 29
ccs               CCS            ccs-cr          ibm-cpd      Completed  8.0.0      2024-08-08T21:37:16Z  8.0.0                 427
cognos_analytics  CAService      ca-addon-cr     ibm-cpd      Completed  25.0.0     2024-08-08T21:36:36Z  25.0.0                25.0.0+20231101.160318.567 

```

