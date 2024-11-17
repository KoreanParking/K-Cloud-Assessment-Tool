# Terraform으로 Prowler 환경 배포하기

Terraform을 사용하여 AWS에 Prowler 환경을 배포하기 위해 아래의 **필수 요구 사항**을 충족해야 합니다.

## 요구 사항

1. **AWS CLI 로그인**  
   Terraform이 자원을 생성 및 관리하기 위해, 아래 권한이 포함된 IAM 계정으로 AWS CLI에 로그인되어 있어야 합니다. 로그인되지 않은 경우, Terraform은 AWS 자원에 접근할 수 없습니다.

2. **IAM 권한**  
   Terraform을 실행할 IAM 계정에는 아래의 권한이 필요합니다. 이는 Terraform이 ECS 클러스터와 서비스 생성, IAM 역할 및 정책 설정, Prowler 결과의 S3 업로드, 그리고 CloudWatch Logs 구성을 수행하기 위함입니다.

   - **IAM 권한**
     - `iam:CreateRole`, `iam:AttachRolePolicy`, `iam:CreatePolicy`: 새로운 역할과 정책을 생성하는 데 필요합니다.
     - `iam:GetRole`, `iam:ListRolePolicies`, `iam:GetPolicyVersion`, `iam:GetPolicy`: 기존 역할과 정책을 확인하는 데 필요합니다.
     - `iam:PassRole`: ECS에 `ecs_task_execution_role` 역할을 전달하는 데 필요합니다.
     - `iam:ListInstanceProfilesForRole`, `iam:ListPolicyVersions`, `iam:ListAttachedRolePolicies`, `iam:ListRoles`, `iam:GetRolePolicy`: IAM 역할에 연결된 인스턴스 프로파일 목록을 조회하고, 정책 버전 및 역할 목록을 조회하는 데 필요합니다.
     - `iam:DeleteRole`, `iam:DeletePolicy`, `iam:DetachRolePolicy`, `iam:GenerateServiceLastAccessedDetails`: Prowler 실행 완료 후 IAM 역할과 정책을 삭제하거나 서비스 접근 내역을 생성하는 데 필요합니다.

   - **ECS 권한**
     - `ecs:CreateCluster`, `ecs:RegisterTaskDefinition`, `ecs:CreateService`: ECS 클러스터 및 서비스를 생성하는 데 필요합니다.
     - `ecs:DescribeClusters`: 기존 ECS 클러스터 정보를 조회하는 데 필요합니다.
     - `ecs:UpdateService`: Prowler 실행 후 서비스의 `desired-count`를 0으로 설정하는 데 필요합니다.
     - `ecs:DeleteCluster`, `ecs:DescribeTaskDefinition`, `ecs:DescribeTasks`, `ecs:DescribeServices`, `ecs:ListTasks`, `ecs:DeleteService`, `ecs:DeregisterTaskDefinition`: Prowler 실행 후 ECS 클러스터와 서비스 관련 자원을 삭제하는 데 필요합니다.

   - **S3 권한**
     - `s3:ListBucket`, `s3:GetObject`, `s3:PutObject`, `s3:PutObjectAcl` : Prowler 결과를 S3 버킷에 업로드하고 객체에 접근하는 데 필요합니다.
     - `s3:CreateBucket`, `s3:PutBucketAcl`, `s3:DeleteBucket` : Prowler 결과를 전송하기위한 S3 버킷을 생성하고 삭제하는 데 필요합니다.

   - **CloudWatch Logs 권한**
     - `logs:CreateLogGroup`, `logs:CreateLogStream`, `logs:PutLogEvents`, `logs:DescribeLogGroups`, `logs:ListTagsForResource`, `logs:DeleteLogGroup`, `logs:PutRetentionPolicy`, `logs:DeleteLogStream`, `logs:DescribeLogStreams`: CloudWatch에 로그를 기록하고 관리하는 데 필요한 권한입니다.
     
     
  - **awscli.json 파일에 모든 필요한 권한이 JSON 형식으로 저장되었습니다. 이 파일을 참조하여 IAM 정책을 생성할 수 있습니다.  `iam:CreateServiceLinkedRole`,`iam:PutRolePolicy` 권한은 AWSServiceRoleForECS가 있다면 필요하지 않습니다.**


3. **Terraform 설치**  
   Terraform이 로컬 환경에 설치되어 있어야 합니다. 설치가 필요한 경우, [Terraform 공식 웹사이트](https://www.terraform.io/downloads)에서 OS에 맞는 설치 파일을 다운로드하고 설치 가이드를 참조해주시기 바랍니다.


4. **환경 변수 파일 작성 (`terraform.tfvars`)**  
   설정할 값들을 `terraform.tfvars` 파일에 입력해야 합니다. 이 파일에는 S3 버킷 이름(버킷은 이름을 입력하면 자동으로 생성되니 미리 생성하지 마세요), AWS 리전 등 필요한 변수 값이 포함되어야 하며, 이를 제외한 나머지 자원은 Terraform을 통해 자동으로 생성됩니다.



## 환경 구성 및 시작 (terraform_start.sh)

Prowler 환경을 배포하려면 `terraform_start.sh` 스크립트를 실행하세요. 이 스크립트는 필요한 IAM 역할과 정책이 이미 존재하는지 확인하고, 없을 경우 생성합니다. 그 후, Terraform을 통해 ECS 및 필요한 자원을 설정합니다.



## 환경 삭제 (terraform_stop_destroy.sh)

배포된 Prowler 환경을 삭제하려면 `terraform_stop_destroy.sh` 스크립트를 실행하세요. 이 스크립트는 배포된 ECS 서비스와 자원을 삭제합니다. 단, S3 버킷은 삭제되지 않습니다.


---

위 요구 사항을 충족하면 Terraform을 통해 Prowler 환경을 안전하게 배포할 수 있습니다.

