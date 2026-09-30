# Part 94: Cloud Deployment - AWS, GCP & Azure

## บทนำ

Cloud Deployment คือการนำ application ขึ้นไปรันบน Cloud Provider แทนที่จะรันบน server ส่วนตัว ช่วยให้ scale ได้ง่าย จ่ายเฉพาะที่ใช้ และมี reliability สูง Part นี้จะครอบคลุม 3 Cloud Provider หลัก: AWS, GCP และ Azure พร้อมการใช้ Python ร่วมกับบริการต่างๆ

---

## 1. Cloud Deployment Options Overview

### เปรียบเทียบตัวเลือกการ Deploy

| Option | คำอธิบาย | เหมาะกับ | ตัวอย่าง |
|--------|-----------|---------|---------|
| IaaS | Virtual machines | Control เต็มที่ | EC2, GCE, Azure VM |
| PaaS | Platform ที่ manage แล้ว | Deploy เร็ว ไม่อยาก manage OS | Heroku, App Engine, Elastic Beanstalk |
| CaaS | Container platform | Microservices | ECS/Fargate, Cloud Run, AKS |
| FaaS | Serverless functions | Event-driven, sporadic | Lambda, Cloud Functions, Azure Functions |
| SaaS | Managed databases/services | ไม่อยาก manage infrastructure | RDS, Cloud SQL, Azure SQL |

### ตัวอย่างที่ 1: Cloud Provider Comparison ด้วย Python

```python
# ตัวอย่าง 1: เปรียบเทียบ cloud services
from dataclasses import dataclass
from typing import List, Optional

@dataclass
class CloudService:
    provider: str
    category: str
    service_name: str
    python_sdk: str
    use_case: str
    pricing_model: str

# รายการ cloud services สำหรับ Python apps
cloud_services = [
    # AWS Services
    CloudService("AWS", "Compute", "EC2", "boto3", "Virtual machines", "Per hour"),
    CloudService("AWS", "Serverless", "Lambda", "boto3", "Functions", "Per invocation"),
    CloudService("AWS", "PaaS", "Elastic Beanstalk", "awsebcli", "Web apps", "Per resource"),
    CloudService("AWS", "Container", "ECS/Fargate", "boto3", "Containers", "Per vCPU/memory"),
    CloudService("AWS", "Database", "RDS", "boto3 + SQLAlchemy", "SQL databases", "Per hour"),
    CloudService("AWS", "Storage", "S3", "boto3", "Object storage", "Per GB"),
    
    # GCP Services
    CloudService("GCP", "Serverless", "Cloud Run", "google-cloud", "Containers", "Per request"),
    CloudService("GCP", "PaaS", "App Engine", "google-cloud-appengine", "Web apps", "Per instance"),
    CloudService("GCP", "Serverless", "Cloud Functions", "google-cloud-functions", "Functions", "Per invocation"),
    CloudService("GCP", "Database", "Cloud SQL", "cloud-sql-connector", "SQL databases", "Per hour"),
    
    # Azure Services  
    CloudService("Azure", "PaaS", "App Service", "azure-mgmt-web", "Web apps", "Per plan"),
    CloudService("Azure", "Serverless", "Azure Functions", "azure-functions", "Functions", "Per execution"),
    CloudService("Azure", "Container", "AKS", "azure-mgmt-containerservice", "Kubernetes", "Per node"),
    CloudService("Azure", "Database", "Azure SQL", "pyodbc", "SQL databases", "Per DTU/vCore"),
]

def compare_providers(category: str) -> None:
    """เปรียบเทียบ services ในแต่ละ category"""
    services = [s for s in cloud_services if s.category == category]
    
    print(f"\n{category} Services Comparison:")
    print("-" * 70)
    print(f"{'Provider':<8} {'Service':<25} {'SDK':<25} {'Pricing'}")
    print("-" * 70)
    
    for svc in services:
        print(f"{svc.provider:<8} {svc.service_name:<25} {svc.python_sdk:<25} {svc.pricing_model}")

# เรียกใช้
for category in ["Serverless", "PaaS", "Database"]:
    compare_providers(category)
```

---

## 2. AWS: EC2 (Elastic Compute Cloud)

### ตัวอย่างที่ 2: จัดการ EC2 ด้วย Boto3

```python
# ตัวอย่าง 2: EC2 Instance Management ด้วย Boto3
import boto3
import time
from typing import Optional, List, Dict, Any
from botocore.exceptions import ClientError

class EC2Manager:
    """จัดการ EC2 Instances"""
    
    def __init__(self, region: str = 'ap-southeast-1'):
        self.ec2 = boto3.client('ec2', region_name=region)
        self.resource = boto3.resource('ec2', region_name=region)
        self.region = region
    
    def create_instance(
        self,
        ami_id: str,
        instance_type: str = 't3.micro',
        key_name: str = None,
        security_group_ids: List[str] = None,
        subnet_id: str = None,
        user_data: str = None,
        tags: Dict[str, str] = None
    ) -> Dict[str, Any]:
        """สร้าง EC2 instance ใหม่"""
        
        params = {
            'ImageId': ami_id,
            'InstanceType': instance_type,
            'MinCount': 1,
            'MaxCount': 1,
        }
        
        if key_name:
            params['KeyName'] = key_name
        
        if security_group_ids:
            params['SecurityGroupIds'] = security_group_ids
        
        if subnet_id:
            params['SubnetId'] = subnet_id
        
        if user_data:
            params['UserData'] = user_data
        
        if tags:
            params['TagSpecifications'] = [{
                'ResourceType': 'instance',
                'Tags': [{'Key': k, 'Value': v} for k, v in tags.items()]
            }]
        
        try:
            response = self.ec2.run_instances(**params)
            instance = response['Instances'][0]
            instance_id = instance['InstanceId']
            
            print(f"Created instance: {instance_id}")
            
            # รอให้ instance พร้อมใช้งาน
            self._wait_for_state(instance_id, 'running')
            
            return instance
            
        except ClientError as e:
            print(f"Error creating instance: {e}")
            raise
    
    def _wait_for_state(self, instance_id: str, state: str, timeout: int = 300):
        """รอให้ instance ถึง state ที่ต้องการ"""
        print(f"Waiting for instance {instance_id} to be {state}...")
        
        waiter = self.ec2.get_waiter(f'instance_{state}')
        waiter.wait(
            InstanceIds=[instance_id],
            WaiterConfig={'Delay': 10, 'MaxAttempts': timeout // 10}
        )
        print(f"Instance {instance_id} is now {state}")
    
    def get_instance_info(self, instance_id: str) -> Optional[Dict]:
        """ดึงข้อมูล instance"""
        try:
            response = self.ec2.describe_instances(InstanceIds=[instance_id])
            instances = response['Reservations'][0]['Instances']
            return instances[0] if instances else None
        except ClientError:
            return None
    
    def stop_instance(self, instance_id: str):
        """หยุด instance (ยังไม่ลบ)"""
        self.ec2.stop_instances(InstanceIds=[instance_id])
        self._wait_for_state(instance_id, 'stopped')
    
    def terminate_instance(self, instance_id: str):
        """ลบ instance ถาวร"""
        self.ec2.terminate_instances(InstanceIds=[instance_id])
        self._wait_for_state(instance_id, 'terminated')
        print(f"Instance {instance_id} terminated")
    
    def list_instances(self, filters: List[Dict] = None) -> List[Dict]:
        """แสดงรายการ instances"""
        params = {}
        if filters:
            params['Filters'] = filters
        
        response = self.ec2.describe_instances(**params)
        instances = []
        
        for reservation in response['Reservations']:
            for instance in reservation['Instances']:
                name = ''
                for tag in instance.get('Tags', []):
                    if tag['Key'] == 'Name':
                        name = tag['Value']
                
                instances.append({
                    'id': instance['InstanceId'],
                    'name': name,
                    'type': instance['InstanceType'],
                    'state': instance['State']['Name'],
                    'public_ip': instance.get('PublicIpAddress', 'N/A'),
                    'private_ip': instance.get('PrivateIpAddress', 'N/A'),
                })
        
        return instances

# ตัวอย่างการใช้งาน (ต้องมี AWS credentials ก่อน)
def demo_ec2():
    manager = EC2Manager()
    
    # แสดงรายการ instances ที่กำลังทำงาน
    running = manager.list_instances(
        filters=[{'Name': 'instance-state-name', 'Values': ['running']}]
    )
    
    print(f"\nRunning instances ({len(running)}):")
    print(f"{'ID':<20} {'Name':<20} {'Type':<15} {'State':<12} {'Public IP'}")
    print("-" * 80)
    
    for inst in running:
        print(f"{inst['id']:<20} {inst['name']:<20} {inst['type']:<15} "
              f"{inst['state']:<12} {inst['public_ip']}")

# demo_ec2()  # uncomment เพื่อรันจริง
print("EC2Manager class defined - ready to use with AWS credentials")
```

---

## 3. AWS: Lambda (Serverless)

### ตัวอย่างที่ 3: Lambda Function สำหรับ Python

```python
# lambda_function.py - Lambda handler
import json
import os
import logging
from typing import Any, Dict
import boto3

logger = logging.getLogger(__name__)
logger.setLevel(logging.INFO)

# Reuse connection ระหว่าง invocations
dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table(os.environ.get('TABLE_NAME', 'users'))

def lambda_handler(event: Dict[str, Any], context: Any) -> Dict[str, Any]:
    """
    Lambda entry point
    
    Parameters:
    - event: ข้อมูลที่ส่งมาจาก trigger (API Gateway, S3, SQS, etc.)
    - context: Lambda execution context
    """
    logger.info(f"Event: {json.dumps(event)}")
    logger.info(f"Function: {context.function_name}")
    logger.info(f"Remaining time: {context.get_remaining_time_in_millis()}ms")
    
    # Route ตาม HTTP method (สำหรับ API Gateway trigger)
    http_method = event.get('httpMethod', 'UNKNOWN')
    path = event.get('path', '/')
    
    try:
        if http_method == 'GET' and path == '/users':
            return handle_get_users(event)
        elif http_method == 'POST' and path == '/users':
            return handle_create_user(event)
        elif http_method == 'GET' and path.startswith('/users/'):
            user_id = path.split('/')[-1]
            return handle_get_user(event, user_id)
        else:
            return response(404, {'error': 'Not found'})
    
    except Exception as e:
        logger.error(f"Unhandled error: {e}", exc_info=True)
        return response(500, {'error': 'Internal server error'})

def handle_get_users(event: Dict) -> Dict:
    """ดึงรายการ users"""
    result = table.scan(Limit=50)
    users = result.get('Items', [])
    return response(200, {'users': users, 'count': len(users)})

def handle_get_user(event: Dict, user_id: str) -> Dict:
    """ดึงข้อมูล user เดียว"""
    result = table.get_item(Key={'id': user_id})
    user = result.get('Item')
    
    if not user:
        return response(404, {'error': f'User {user_id} not found'})
    
    return response(200, user)

def handle_create_user(event: Dict) -> Dict:
    """สร้าง user ใหม่"""
    import uuid
    
    body = json.loads(event.get('body', '{}'))
    
    if not body.get('name') or not body.get('email'):
        return response(400, {'error': 'name and email are required'})
    
    user = {
        'id': str(uuid.uuid4()),
        'name': body['name'],
        'email': body['email'],
    }
    
    table.put_item(Item=user)
    return response(201, user)

def response(status_code: int, body: Any) -> Dict:
    """สร้าง API Gateway response"""
    return {
        'statusCode': status_code,
        'headers': {
            'Content-Type': 'application/json',
            'Access-Control-Allow-Origin': '*',
        },
        'body': json.dumps(body, default=str),
    }
```

### ตัวอย่างที่ 4: Deploy Lambda ด้วย Boto3

```python
# ตัวอย่าง 4: จัดการ Lambda Functions
import boto3
import json
import zipfile
import io
import os
from pathlib import Path

class LambdaDeployer:
    """Deploy และจัดการ Lambda functions"""
    
    def __init__(self, region: str = 'ap-southeast-1'):
        self.client = boto3.client('lambda', region_name=region)
        self.iam = boto3.client('iam', region_name=region)
    
    def create_deployment_package(self, source_dir: str) -> bytes:
        """สร้าง ZIP file จาก source code"""
        zip_buffer = io.BytesIO()
        
        with zipfile.ZipFile(zip_buffer, 'w', zipfile.ZIP_DEFLATED) as zip_file:
            for file_path in Path(source_dir).rglob('*.py'):
                arcname = file_path.relative_to(source_dir)
                zip_file.write(file_path, arcname)
        
        return zip_buffer.getvalue()
    
    def create_or_update_function(
        self,
        function_name: str,
        handler: str,
        role_arn: str,
        source_dir: str,
        runtime: str = 'python3.11',
        memory_size: int = 256,
        timeout: int = 30,
        environment: dict = None,
        layers: list = None
    ) -> dict:
        """สร้างหรืออัปเดต Lambda function"""
        
        zip_code = self.create_deployment_package(source_dir)
        
        params = {
            'FunctionName': function_name,
            'Runtime': runtime,
            'Role': role_arn,
            'Handler': handler,
            'Code': {'ZipFile': zip_code},
            'MemorySize': memory_size,
            'Timeout': timeout,
        }
        
        if environment:
            params['Environment'] = {'Variables': environment}
        
        if layers:
            params['Layers'] = layers
        
        try:
            # อัปเดตถ้ามีอยู่แล้ว
            self.client.get_function(FunctionName=function_name)
            
            # อัปเดต code
            self.client.update_function_code(
                FunctionName=function_name,
                ZipFile=zip_code
            )
            
            # อัปเดต configuration
            config_params = {k: v for k, v in params.items() 
                           if k not in ['FunctionName', 'Code']}
            response = self.client.update_function_configuration(
                FunctionName=function_name,
                **config_params
            )
            print(f"Updated Lambda: {function_name}")
            
        except self.client.exceptions.ResourceNotFoundException:
            # สร้างใหม่
            response = self.client.create_function(**params)
            print(f"Created Lambda: {function_name}")
        
        return response
    
    def invoke_function(self, function_name: str, payload: dict) -> dict:
        """เรียกใช้ Lambda function"""
        response = self.client.invoke(
            FunctionName=function_name,
            InvocationType='RequestResponse',
            Payload=json.dumps(payload).encode()
        )
        
        result = json.loads(response['Payload'].read())
        return result
    
    def get_function_logs(self, function_name: str, limit: int = 10) -> list:
        """ดึง logs จาก CloudWatch"""
        logs_client = boto3.client('logs')
        log_group = f'/aws/lambda/{function_name}'
        
        try:
            # ดึง log streams ล่าสุด
            streams = logs_client.describe_log_streams(
                logGroupName=log_group,
                orderBy='LastEventTime',
                descending=True,
                limit=1
            )
            
            if not streams['logStreams']:
                return []
            
            stream_name = streams['logStreams'][0]['logStreamName']
            
            events = logs_client.get_log_events(
                logGroupName=log_group,
                logStreamName=stream_name,
                limit=limit
            )
            
            return [e['message'] for e in events['events']]
        
        except Exception as e:
            print(f"Error getting logs: {e}")
            return []

# ตัวอย่างการใช้งาน
print("LambdaDeployer ready to use")
print("Environment variables needed: AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY")
```

### ตัวอย่างที่ 5: Lambda Event Sources

```python
# ตัวอย่าง 5: Lambda handlers สำหรับ event sources ต่างๆ

import json
import base64
from typing import Any, Dict, List

# ========== S3 Event Handler ==========
def s3_handler(event: Dict, context: Any) -> None:
    """จัดการไฟล์ที่ upload ไปยัง S3"""
    import boto3
    from PIL import Image
    import io
    
    s3 = boto3.client('s3')
    
    for record in event['Records']:
        bucket = record['s3']['bucket']['name']
        key = record['s3']['object']['key']
        
        print(f"Processing: s3://{bucket}/{key}")
        
        # ดาวน์โหลดไฟล์
        response = s3.get_object(Bucket=bucket, Key=key)
        content = response['Body'].read()
        
        if key.lower().endswith(('.jpg', '.jpeg', '.png')):
            # สร้าง thumbnail
            image = Image.open(io.BytesIO(content))
            image.thumbnail((200, 200))
            
            output = io.BytesIO()
            image.save(output, format='JPEG')
            
            thumb_key = f"thumbnails/{key}"
            s3.put_object(
                Bucket=bucket,
                Key=thumb_key,
                Body=output.getvalue(),
                ContentType='image/jpeg'
            )
            print(f"Created thumbnail: {thumb_key}")

# ========== SQS Event Handler ==========
def sqs_handler(event: Dict, context: Any) -> Dict:
    """ประมวลผล messages จาก SQS"""
    failed_ids = []
    
    for record in event['Records']:
        message_id = record['messageId']
        
        try:
            body = json.loads(record['body'])
            print(f"Processing message: {message_id}")
            print(f"Body: {body}")
            
            # ประมวลผล message
            process_message(body)
            
        except Exception as e:
            print(f"Failed to process {message_id}: {e}")
            failed_ids.append({'itemIdentifier': message_id})
    
    # Return failed messages สำหรับ retry
    return {'batchItemFailures': failed_ids}

def process_message(body: dict) -> None:
    """ประมวลผล SQS message"""
    print(f"Processing: {body.get('action')} for {body.get('user_id')}")

# ========== API Gateway Handler ==========
def api_gateway_handler(event: Dict, context: Any) -> Dict:
    """Handler สำหรับ API Gateway REST API"""
    
    method = event['httpMethod']
    path = event['path']
    query_params = event.get('queryStringParameters') or {}
    path_params = event.get('pathParameters') or {}
    
    # ดึง body
    body_str = event.get('body', '{}') or '{}'
    if event.get('isBase64Encoded'):
        body_str = base64.b64decode(body_str).decode('utf-8')
    body = json.loads(body_str)
    
    print(f"{method} {path}")
    print(f"Query: {query_params}")
    print(f"Body: {body}")
    
    return {
        'statusCode': 200,
        'headers': {'Content-Type': 'application/json'},
        'body': json.dumps({'message': 'OK', 'path': path})
    }

# ========== EventBridge (Scheduled) Handler ==========
def scheduled_handler(event: Dict, context: Any) -> None:
    """Handler สำหรับ scheduled events (EventBridge)"""
    import boto3
    from datetime import datetime
    
    print(f"Running scheduled task at: {datetime.utcnow().isoformat()}")
    print(f"Event source: {event.get('source')}")
    
    # ทำงาน scheduled task เช่น cleanup, report generation
    cleanup_old_records()
    generate_daily_report()

def cleanup_old_records():
    print("Cleaning up old records...")

def generate_daily_report():
    print("Generating daily report...")
```

---

## 4. AWS: Elastic Beanstalk

### ตัวอย่างที่ 6: Elastic Beanstalk Configuration

```python
# ตัวอย่าง 6: Elastic Beanstalk deployment สำหรับ FastAPI

# .ebextensions/01-python.config
"""
option_settings:
  aws:elasticbeanstalk:container:python:
    WSGIPath: application:application
  aws:elasticbeanstalk:environment:proxy:staticfiles:
    /static: static
  aws:autoscaling:asg:
    MinSize: 1
    MaxSize: 4
  aws:autoscaling:trigger:
    MeasureName: CPUUtilization
    Statistic: Average
    Unit: Percent
    LowerThreshold: 30
    UpperThreshold: 70
  aws:elasticbeanstalk:application:environment:
    DJANGO_SETTINGS_MODULE: myapp.settings.production

packages:
  yum:
    git: []
    postgresql-devel: []

commands:
  01_upgrade_pip:
    command: /usr/bin/pip install --upgrade pip

container_commands:
  01_migrate:
    command: python manage.py migrate
    leader_only: true
  02_collectstatic:
    command: python manage.py collectstatic --noinput
"""

# application.py - Elastic Beanstalk entry point
from fastapi import FastAPI
from mangum import Mangum

app = FastAPI(title="My EB App")

@app.get("/")
async def root():
    return {"message": "Running on Elastic Beanstalk"}

@app.get("/health")
async def health():
    return {"status": "healthy"}

# สำหรับ Elastic Beanstalk
application = Mangum(app)

# Procfile
"""
web: gunicorn application:application -k uvicorn.workers.UvicornWorker -b 0.0.0.0:8000
"""
```

```python
# ตัวอย่าง 6b: EB CLI wrapper ด้วย Python
import subprocess
import json
from pathlib import Path

class ElasticBeanstalkManager:
    """Wrapper สำหรับ EB CLI"""
    
    def __init__(self, app_name: str, env_name: str, region: str = 'ap-southeast-1'):
        self.app_name = app_name
        self.env_name = env_name
        self.region = region
    
    def _run(self, *args) -> str:
        """รัน eb command"""
        cmd = ['eb'] + list(args) + ['--region', self.region]
        result = subprocess.run(cmd, capture_output=True, text=True)
        if result.returncode != 0:
            raise RuntimeError(f"EB command failed: {result.stderr}")
        return result.stdout
    
    def init(self, platform: str = 'python-3.11'):
        """Initialize EB application"""
        self._run('init', self.app_name, '-p', platform)
        print(f"Initialized EB app: {self.app_name}")
    
    def create_env(self, instance_type: str = 't3.micro'):
        """สร้าง environment"""
        self._run('create', self.env_name, 
                 '-i', instance_type,
                 '--single')
        print(f"Created environment: {self.env_name}")
    
    def deploy(self, label: str = None):
        """Deploy application"""
        args = ['deploy', self.env_name]
        if label:
            args += ['--label', label]
        self._run(*args)
        print(f"Deployed to: {self.env_name}")
    
    def set_environment_variables(self, **kwargs):
        """ตั้งค่า environment variables"""
        env_vars = [f"{k}={v}" for k, v in kwargs.items()]
        self._run('setenv', *env_vars, '--environment', self.env_name)
    
    def get_status(self) -> str:
        """ดูสถานะ environment"""
        return self._run('status', self.env_name)
    
    def open(self):
        """เปิด URL ใน browser"""
        self._run('open', self.env_name)

# การใช้งาน
# manager = ElasticBeanstalkManager('my-app', 'my-app-production')
# manager.deploy(label='v1.2.3')
print("ElasticBeanstalkManager ready")
```

---

## 5. AWS: ECS/Fargate

### ตัวอย่างที่ 7: ECS Task Definition และ Service

```python
# ตัวอย่าง 7: จัดการ ECS ด้วย Boto3
import boto3
import json
from typing import List, Dict, Optional

class ECSManager:
    """จัดการ ECS Clusters, Task Definitions และ Services"""
    
    def __init__(self, region: str = 'ap-southeast-1'):
        self.client = boto3.client('ecs', region_name=region)
        self.region = region
    
    def register_task_definition(
        self,
        family: str,
        image: str,
        cpu: int = 256,       # 0.25 vCPU
        memory: int = 512,    # 512 MB
        port: int = 8000,
        environment: List[Dict] = None,
        secrets: List[Dict] = None,
        role_arn: str = None
    ) -> str:
        """ลงทะเบียน task definition"""
        
        container_def = {
            'name': family,
            'image': image,
            'portMappings': [
                {
                    'containerPort': port,
                    'protocol': 'tcp'
                }
            ],
            'logConfiguration': {
                'logDriver': 'awslogs',
                'options': {
                    'awslogs-group': f'/ecs/{family}',
                    'awslogs-region': self.region,
                    'awslogs-stream-prefix': 'ecs'
                }
            },
            'essential': True,
        }
        
        if environment:
            container_def['environment'] = environment
        
        if secrets:
            container_def['secrets'] = secrets
        
        params = {
            'family': family,
            'networkMode': 'awsvpc',
            'containerDefinitions': [container_def],
            'requiresCompatibilities': ['FARGATE'],
            'cpu': str(cpu),
            'memory': str(memory),
        }
        
        if role_arn:
            params['executionRoleArn'] = role_arn
            params['taskRoleArn'] = role_arn
        
        response = self.client.register_task_definition(**params)
        task_def = response['taskDefinition']
        arn = task_def['taskDefinitionArn']
        print(f"Registered task definition: {arn}")
        return arn
    
    def create_or_update_service(
        self,
        cluster: str,
        service_name: str,
        task_definition: str,
        desired_count: int = 1,
        subnets: List[str] = None,
        security_groups: List[str] = None,
        target_group_arn: str = None
    ) -> Dict:
        """สร้างหรืออัปเดต ECS service"""
        
        network_config = {}
        if subnets or security_groups:
            network_config = {
                'awsvpcConfiguration': {
                    'subnets': subnets or [],
                    'securityGroups': security_groups or [],
                    'assignPublicIp': 'ENABLED'
                }
            }
        
        load_balancers = []
        if target_group_arn:
            load_balancers = [{
                'targetGroupArn': target_group_arn,
                'containerName': service_name,
                'containerPort': 8000
            }]
        
        try:
            # ตรวจสอบว่า service มีอยู่แล้วหรือไม่
            self.client.describe_services(
                cluster=cluster, 
                services=[service_name]
            )
            
            # อัปเดต service
            response = self.client.update_service(
                cluster=cluster,
                service=service_name,
                taskDefinition=task_definition,
                desiredCount=desired_count,
                networkConfiguration=network_config or None,
                forceNewDeployment=True
            )
            print(f"Updated ECS service: {service_name}")
            
        except Exception:
            # สร้างใหม่
            params = {
                'cluster': cluster,
                'serviceName': service_name,
                'taskDefinition': task_definition,
                'desiredCount': desired_count,
                'launchType': 'FARGATE',
            }
            
            if network_config:
                params['networkConfiguration'] = network_config
            
            if load_balancers:
                params['loadBalancers'] = load_balancers
            
            response = self.client.create_service(**params)
            print(f"Created ECS service: {service_name}")
        
        return response.get('service', {})
    
    def scale_service(self, cluster: str, service: str, desired_count: int):
        """Scale service"""
        self.client.update_service(
            cluster=cluster,
            service=service,
            desiredCount=desired_count
        )
        print(f"Scaled {service} to {desired_count} tasks")

print("ECSManager ready")
```

---

## 6. AWS: RDS และ S3

### ตัวอย่างที่ 8: S3 Operations

```python
# ตัวอย่าง 8: S3 File Operations
import boto3
from botocore.exceptions import ClientError
from pathlib import Path
import json
from typing import Optional, List
import mimetypes

class S3Manager:
    """จัดการไฟล์บน S3"""
    
    def __init__(self, bucket_name: str, region: str = 'ap-southeast-1'):
        self.s3 = boto3.client('s3', region_name=region)
        self.bucket = bucket_name
        self.region = region
    
    def upload_file(
        self,
        local_path: str,
        s3_key: str = None,
        public: bool = False,
        content_type: str = None
    ) -> str:
        """อัปโหลดไฟล์ไปยัง S3"""
        
        if s3_key is None:
            s3_key = Path(local_path).name
        
        # ตรวจสอบ content type
        if content_type is None:
            content_type, _ = mimetypes.guess_type(local_path)
            content_type = content_type or 'application/octet-stream'
        
        extra_args = {'ContentType': content_type}
        
        if public:
            extra_args['ACL'] = 'public-read'
        
        self.s3.upload_file(local_path, self.bucket, s3_key, ExtraArgs=extra_args)
        
        url = f"https://{self.bucket}.s3.{self.region}.amazonaws.com/{s3_key}"
        print(f"Uploaded: {url}")
        return url
    
    def upload_json(self, data: dict, s3_key: str) -> str:
        """อัปโหลด JSON data"""
        json_str = json.dumps(data, ensure_ascii=False, indent=2)
        
        self.s3.put_object(
            Bucket=self.bucket,
            Key=s3_key,
            Body=json_str.encode('utf-8'),
            ContentType='application/json'
        )
        
        return f"s3://{self.bucket}/{s3_key}"
    
    def download_file(self, s3_key: str, local_path: str) -> str:
        """ดาวน์โหลดไฟล์จาก S3"""
        Path(local_path).parent.mkdir(parents=True, exist_ok=True)
        self.s3.download_file(self.bucket, s3_key, local_path)
        print(f"Downloaded: {local_path}")
        return local_path
    
    def get_presigned_url(self, s3_key: str, expires_in: int = 3600) -> str:
        """สร้าง presigned URL สำหรับ temporary access"""
        url = self.s3.generate_presigned_url(
            'get_object',
            Params={'Bucket': self.bucket, 'Key': s3_key},
            ExpiresIn=expires_in
        )
        return url
    
    def list_files(self, prefix: str = '') -> List[dict]:
        """แสดงรายการไฟล์"""
        response = self.s3.list_objects_v2(
            Bucket=self.bucket,
            Prefix=prefix
        )
        
        files = []
        for obj in response.get('Contents', []):
            files.append({
                'key': obj['Key'],
                'size': obj['Size'],
                'last_modified': obj['LastModified'].isoformat(),
            })
        
        return files
    
    def delete_file(self, s3_key: str) -> bool:
        """ลบไฟล์"""
        try:
            self.s3.delete_object(Bucket=self.bucket, Key=s3_key)
            print(f"Deleted: s3://{self.bucket}/{s3_key}")
            return True
        except ClientError as e:
            print(f"Error deleting {s3_key}: {e}")
            return False
    
    def copy_file(self, source_key: str, dest_key: str, dest_bucket: str = None) -> bool:
        """คัดลอกไฟล์"""
        dest_bucket = dest_bucket or self.bucket
        copy_source = {'Bucket': self.bucket, 'Key': source_key}
        
        try:
            self.s3.copy_object(
                CopySource=copy_source,
                Bucket=dest_bucket,
                Key=dest_key
            )
            print(f"Copied: {source_key} → {dest_key}")
            return True
        except ClientError as e:
            print(f"Error copying: {e}")
            return False

# ตัวอย่างการใช้งาน S3 with streaming
def process_large_file_from_s3(bucket: str, key: str):
    """อ่านไฟล์ขนาดใหญ่จาก S3 แบบ streaming"""
    s3 = boto3.client('s3')
    
    response = s3.get_object(Bucket=bucket, Key=key)
    
    # อ่านแบบ streaming โดยไม่โหลดทั้งไฟล์เข้า memory
    chunk_size = 1024 * 1024  # 1MB chunks
    total_processed = 0
    
    for chunk in response['Body'].iter_chunks(chunk_size):
        # ประมวลผลแต่ละ chunk
        total_processed += len(chunk)
        print(f"Processed: {total_processed / 1024 / 1024:.1f} MB")

print("S3Manager ready")
```

### ตัวอย่างที่ 9: RDS Connection Pooling

```python
# ตัวอย่าง 9: RDS Connection Pool ด้วย SQLAlchemy
import os
from sqlalchemy import create_engine, text
from sqlalchemy.orm import sessionmaker, declarative_base
from sqlalchemy.pool import QueuePool
import boto3
import json

def get_rds_credentials(secret_name: str, region: str = 'ap-southeast-1') -> dict:
    """ดึง RDS credentials จาก AWS Secrets Manager"""
    client = boto3.client('secretsmanager', region_name=region)
    
    response = client.get_secret_value(SecretId=secret_name)
    secret = json.loads(response['SecretString'])
    
    return {
        'host': secret['host'],
        'port': secret.get('port', 5432),
        'database': secret['dbname'],
        'username': secret['username'],
        'password': secret['password'],
    }

def create_database_engine(
    secret_name: str = None,
    database_url: str = None,
    pool_size: int = 10,
    max_overflow: int = 20
):
    """สร้าง SQLAlchemy engine ที่เชื่อมต่อกับ RDS"""
    
    if database_url is None:
        if secret_name:
            creds = get_rds_credentials(secret_name)
            database_url = (
                f"postgresql+psycopg2://{creds['username']}:{creds['password']}"
                f"@{creds['host']}:{creds['port']}/{creds['database']}"
            )
        else:
            database_url = os.environ.get('DATABASE_URL', 'sqlite:///./dev.db')
    
    engine = create_engine(
        database_url,
        poolclass=QueuePool,
        pool_size=pool_size,         # จำนวน connections ที่เก็บไว้
        max_overflow=max_overflow,   # connections เพิ่มเติมที่ยอมรับได้
        pool_pre_ping=True,          # ตรวจสอบ connection ก่อนใช้
        pool_recycle=3600,           # recycle connections ทุก 1 ชั่วโมง
        echo=os.environ.get('SQL_DEBUG', 'false').lower() == 'true',
        connect_args={
            'connect_timeout': 10,
            'keepalives': 1,
            'keepalives_idle': 30,
            'keepalives_interval': 10,
            'keepalives_count': 5,
        } if 'postgresql' in database_url else {}
    )
    
    return engine

# สร้าง Base สำหรับ models
Base = declarative_base()

# สร้าง engine และ session factory
engine = create_database_engine()
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

def get_db():
    """Dependency สำหรับ FastAPI"""
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

# Test connection
def test_connection():
    try:
        with engine.connect() as conn:
            result = conn.execute(text("SELECT 1"))
            print("Database connection successful!")
            return True
    except Exception as e:
        print(f"Database connection failed: {e}")
        return False

print("Database engine configured")
```

---

## 7. Boto3 Library สำหรับ AWS

### ตัวอย่างที่ 10: Boto3 Utility Functions

```python
# ตัวอย่าง 10: Boto3 helper utilities
import boto3
import json
from typing import Any, Dict, List, Optional
from datetime import datetime, timezone
from botocore.exceptions import ClientError

class AWSHelper:
    """Collection of AWS helper functions"""
    
    @staticmethod
    def get_current_region() -> str:
        """ดึง AWS region ที่ใช้งานอยู่"""
        session = boto3.session.Session()
        return session.region_name or 'us-east-1'
    
    @staticmethod
    def get_account_id() -> str:
        """ดึง AWS Account ID"""
        sts = boto3.client('sts')
        return sts.get_caller_identity()['Account']
    
    @staticmethod
    def send_sns_notification(topic_arn: str, subject: str, message: str) -> bool:
        """ส่ง notification ผ่าน SNS"""
        sns = boto3.client('sns')
        try:
            sns.publish(
                TopicArn=topic_arn,
                Subject=subject,
                Message=message
            )
            return True
        except ClientError as e:
            print(f"SNS error: {e}")
            return False
    
    @staticmethod
    def send_sqs_message(queue_url: str, body: dict, delay: int = 0) -> Optional[str]:
        """ส่ง message ไปยัง SQS"""
        sqs = boto3.client('sqs')
        try:
            response = sqs.send_message(
                QueueUrl=queue_url,
                MessageBody=json.dumps(body),
                DelaySeconds=delay
            )
            return response['MessageId']
        except ClientError as e:
            print(f"SQS error: {e}")
            return None
    
    @staticmethod
    def put_metric(
        namespace: str,
        metric_name: str,
        value: float,
        unit: str = 'Count',
        dimensions: Dict[str, str] = None
    ) -> None:
        """ส่ง custom metric ไปยัง CloudWatch"""
        cloudwatch = boto3.client('cloudwatch')
        
        metric_data = {
            'MetricName': metric_name,
            'Value': value,
            'Unit': unit,
            'Timestamp': datetime.now(timezone.utc),
        }
        
        if dimensions:
            metric_data['Dimensions'] = [
                {'Name': k, 'Value': v}
                for k, v in dimensions.items()
            ]
        
        cloudwatch.put_metric_data(
            Namespace=namespace,
            MetricData=[metric_data]
        )
    
    @staticmethod
    def get_ssm_parameter(name: str, decrypt: bool = True) -> Optional[str]:
        """ดึงค่าจาก AWS SSM Parameter Store"""
        ssm = boto3.client('ssm')
        try:
            response = ssm.get_parameter(Name=name, WithDecryption=decrypt)
            return response['Parameter']['Value']
        except ClientError:
            return None
    
    @staticmethod
    def invalidate_cloudfront(distribution_id: str, paths: List[str]) -> str:
        """สร้าง CloudFront cache invalidation"""
        cf = boto3.client('cloudfront')
        
        response = cf.create_invalidation(
            DistributionId=distribution_id,
            InvalidationBatch={
                'Paths': {
                    'Quantity': len(paths),
                    'Items': paths,
                },
                'CallerReference': str(datetime.now().timestamp())
            }
        )
        
        invalidation_id = response['Invalidation']['Id']
        print(f"Created CloudFront invalidation: {invalidation_id}")
        return invalidation_id

# ตัวอย่างการใช้งาน
def deploy_static_website(bucket: str, dist_id: str, local_dir: str):
    """Deploy static website ไปยัง S3 + CloudFront"""
    s3 = S3Manager(bucket)
    helper = AWSHelper()
    
    # Upload ไฟล์ทั้งหมด
    from pathlib import Path
    uploaded_keys = []
    
    for file_path in Path(local_dir).rglob('*'):
        if file_path.is_file():
            key = str(file_path.relative_to(local_dir))
            s3.upload_file(str(file_path), key, public=True)
            uploaded_keys.append(f'/{key}')
    
    # Invalidate CloudFront cache
    helper.invalidate_cloudfront(dist_id, ['/*'])
    
    print(f"Deployed {len(uploaded_keys)} files")

print("AWSHelper ready")
```

---

## 8. GCP: Cloud Run

### ตัวอย่างที่ 11: Deploy ไปยัง Cloud Run

```python
# ตัวอย่าง 11: Cloud Run deployment
from google.cloud import run_v2
from google.api_core.exceptions import NotFound
import subprocess
import json

class CloudRunDeployer:
    """Deploy Python apps ไปยัง Google Cloud Run"""
    
    def __init__(self, project_id: str, region: str = 'asia-southeast1'):
        self.project = project_id
        self.region = region
        self.client = run_v2.ServicesClient()
        self.parent = f"projects/{project_id}/locations/{region}"
    
    def build_and_push(self, image_name: str, source_dir: str = '.') -> str:
        """Build image ด้วย Google Cloud Build และ push ไปยัง GCR"""
        
        full_image = f"gcr.io/{self.project}/{image_name}:latest"
        
        print(f"Building image: {full_image}")
        subprocess.run([
            'gcloud', 'builds', 'submit',
            '--image', full_image,
            source_dir
        ], check=True)
        
        return full_image
    
    def deploy_service(
        self,
        service_name: str,
        image: str,
        cpu: str = '1',
        memory: str = '512Mi',
        min_instances: int = 0,
        max_instances: int = 100,
        port: int = 8000,
        env_vars: dict = None,
        allow_unauthenticated: bool = True
    ) -> str:
        """Deploy service ไปยัง Cloud Run"""
        
        container = run_v2.Container(
            image=image,
            ports=[run_v2.ContainerPort(container_port=port)],
            resources=run_v2.ResourceRequirements(
                limits={'cpu': cpu, 'memory': memory}
            ),
            env=[
                run_v2.EnvVar(name=k, value=v)
                for k, v in (env_vars or {}).items()
            ]
        )
        
        scaling = run_v2.RevisionScaling(
            min_instance_count=min_instances,
            max_instance_count=max_instances
        )
        
        service = run_v2.Service(
            template=run_v2.RevisionTemplate(
                containers=[container],
                scaling=scaling,
            )
        )
        
        service_path = f"{self.parent}/services/{service_name}"
        
        try:
            # อัปเดต service ที่มีอยู่
            existing = self.client.get_service(name=service_path)
            existing.template = service.template
            operation = self.client.update_service(service=existing)
            print(f"Updating Cloud Run service: {service_name}")
        except NotFound:
            # สร้างใหม่
            service.name = service_path
            operation = self.client.create_service(
                parent=self.parent,
                service=service,
                service_id=service_name
            )
            print(f"Creating Cloud Run service: {service_name}")
        
        # รอให้ operation เสร็จ
        result = operation.result(timeout=300)
        
        # ตั้งค่า public access
        if allow_unauthenticated:
            self._allow_public_access(service_path)
        
        url = result.urls[0] if result.urls else 'URL not available'
        print(f"Service URL: {url}")
        return url
    
    def _allow_public_access(self, service_name: str):
        """อนุญาตให้ทุกคนเข้าถึงได้"""
        from google.iam.v1 import iam_policy_pb2, policy_pb2
        
        policy = self.client.get_iam_policy(resource=service_name)
        
        binding = policy_pb2.Binding(
            role="roles/run.invoker",
            members=["allUsers"]
        )
        policy.bindings.append(binding)
        
        self.client.set_iam_policy(
            request={
                'resource': service_name,
                'policy': policy
            }
        )

# ตัวอย่างการใช้งาน
def deploy_to_cloud_run():
    deployer = CloudRunDeployer('my-gcp-project')
    
    image = deployer.build_and_push('my-python-app', '.')
    url = deployer.deploy_service(
        service_name='my-python-app',
        image=image,
        memory='1Gi',
        min_instances=1,
        env_vars={
            'ENVIRONMENT': 'production',
            'LOG_LEVEL': 'INFO'
        }
    )
    
    print(f"Deployed to: {url}")

print("CloudRunDeployer ready")
```

---

## 9. GCP: App Engine และ Cloud Functions

### ตัวอย่างที่ 12: App Engine Configuration

```yaml
# app.yaml - Google App Engine configuration
runtime: python311
entrypoint: gunicorn -b :$PORT main:app

env_variables:
  ENVIRONMENT: production
  DATABASE_URL: postgresql://user:pass@/mydb?host=/cloudsql/project:region:instance

automatic_scaling:
  min_instances: 1
  max_instances: 10
  target_cpu_utilization: 0.6
  min_pending_latency: automatic
  max_pending_latency: 30ms
  max_concurrent_requests: 80

resources:
  cpu: 1
  memory_gb: 0.5
  disk_size_gb: 10

handlers:
  - url: /static
    static_dir: static
  - url: /.*
    script: auto
    secure: always

inbound_services:
  - warmup

vpc_access_connector:
  name: projects/my-project/locations/asia-southeast1/connectors/my-connector
```

```python
# ตัวอย่าง 12: Cloud Functions
from functions_framework import http
from flask import Request, Response
import json
import os

# HTTP Cloud Function
@http
def hello_python(request: Request) -> Response:
    """HTTP Cloud Function"""
    
    if request.method == 'OPTIONS':
        headers = {
            'Access-Control-Allow-Origin': '*',
            'Access-Control-Allow-Methods': 'GET, POST',
            'Access-Control-Max-Age': '3600',
        }
        return Response('', 204, headers)
    
    headers = {'Access-Control-Allow-Origin': '*'}
    
    data = request.get_json(silent=True) or {}
    name = data.get('name', 'World')
    
    return Response(
        json.dumps({'message': f'Hello, {name}!'}),
        200,
        headers,
        content_type='application/json'
    )

# Pub/Sub Cloud Function
def pubsub_handler(event: dict, context) -> None:
    """Pub/Sub triggered Cloud Function"""
    import base64
    
    if 'data' in event:
        message = base64.b64decode(event['data']).decode('utf-8')
        data = json.loads(message)
        print(f"Processing message: {data}")
        process_pubsub_message(data)

def process_pubsub_message(data: dict) -> None:
    """ประมวลผล Pub/Sub message"""
    action = data.get('action')
    payload = data.get('payload', {})
    
    if action == 'send_email':
        print(f"Sending email to: {payload.get('email')}")
    elif action == 'process_order':
        print(f"Processing order: {payload.get('order_id')}")
    else:
        print(f"Unknown action: {action}")

# Cloud Storage Function
def gcs_handler(event: dict, context) -> None:
    """Cloud Storage triggered Function"""
    from google.cloud import storage
    
    bucket_name = event['bucket']
    file_name = event['name']
    
    print(f"File uploaded: gs://{bucket_name}/{file_name}")
    
    # ประมวลผลไฟล์
    client = storage.Client()
    bucket = client.bucket(bucket_name)
    blob = bucket.blob(file_name)
    
    content = blob.download_as_text()
    print(f"File size: {len(content)} bytes")
```

---

## 10. Azure: App Service และ Azure Functions

### ตัวอย่างที่ 13: Azure App Service

```python
# ตัวอย่าง 13: Deploy ไปยัง Azure App Service
from azure.mgmt.web import WebSiteManagementClient
from azure.identity import DefaultAzureCredential
import subprocess
import os

class AzureAppServiceDeployer:
    """Deploy Python apps ไปยัง Azure App Service"""
    
    def __init__(self, subscription_id: str):
        credential = DefaultAzureCredential()
        self.client = WebSiteManagementClient(credential, subscription_id)
        self.subscription_id = subscription_id
    
    def create_app_service_plan(
        self,
        resource_group: str,
        plan_name: str,
        location: str = 'Southeast Asia',
        sku: str = 'B1'
    ):
        """สร้าง App Service Plan"""
        from azure.mgmt.web.models import AppServicePlan, SkuDescription
        
        plan = self.client.app_service_plans.begin_create_or_update(
            resource_group,
            plan_name,
            AppServicePlan(
                location=location,
                sku=SkuDescription(name=sku, tier='Basic'),
                kind='linux',
                reserved=True   # Linux App Service
            )
        ).result()
        
        print(f"Created App Service Plan: {plan_name}")
        return plan
    
    def create_web_app(
        self,
        resource_group: str,
        app_name: str,
        plan_id: str,
        python_version: str = '3.11',
        app_settings: dict = None
    ):
        """สร้าง Web App"""
        from azure.mgmt.web.models import Site, SiteConfig, NameValuePair
        
        settings = [
            NameValuePair(name='PYTHON_VERSION', value=python_version),
            NameValuePair(name='SCM_DO_BUILD_DURING_DEPLOYMENT', value='true'),
        ]
        
        if app_settings:
            for k, v in app_settings.items():
                settings.append(NameValuePair(name=k, value=v))
        
        app = self.client.web_apps.begin_create_or_update(
            resource_group,
            app_name,
            Site(
                location='Southeast Asia',
                server_farm_id=plan_id,
                site_config=SiteConfig(
                    linux_fx_version=f"PYTHON|{python_version}",
                    app_command_line="gunicorn --bind=0.0.0.0 --timeout 600 app:app",
                    app_settings=settings,
                    always_on=True,
                    http20_enabled=True,
                )
            )
        ).result()
        
        print(f"Created Web App: {app_name}")
        print(f"URL: https://{app_name}.azurewebsites.net")
        return app
    
    def deploy_via_zip(self, resource_group: str, app_name: str, zip_path: str):
        """Deploy ด้วย ZIP deployment"""
        publish_credentials = self.client.web_apps.begin_list_publishing_credentials(
            resource_group, app_name
        ).result()
        
        subprocess.run([
            'az', 'webapp', 'deployment', 'source', 'config-zip',
            '--resource-group', resource_group,
            '--name', app_name,
            '--src', zip_path
        ], check=True)
        
        print(f"Deployed to: https://{app_name}.azurewebsites.net")

print("AzureAppServiceDeployer ready")
```

### ตัวอย่างที่ 14: Azure Functions

```python
# ตัวอย่าง 14: Azure Functions สำหรับ Python

# function_app.py
import azure.functions as func
import json
import logging
from datetime import datetime

app = func.FunctionApp(http_auth_level=func.AuthLevel.FUNCTION)

# HTTP Trigger Function
@app.route(route="hello")
def http_trigger(req: func.HttpRequest) -> func.HttpResponse:
    """HTTP triggered Azure Function"""
    logging.info('HTTP trigger function processed a request.')
    
    name = req.params.get('name')
    if not name:
        try:
            req_body = req.get_json()
            name = req_body.get('name')
        except ValueError:
            pass
    
    if name:
        return func.HttpResponse(
            json.dumps({"message": f"Hello, {name}!"}),
            mimetype="application/json"
        )
    else:
        return func.HttpResponse(
            "Please pass a name parameter",
            status_code=400
        )

# Timer Trigger Function
@app.timer_trigger(
    schedule="0 0 * * * *",   # ทุกชั่วโมง
    arg_name="timer",
    run_on_startup=False
)
def timer_trigger(timer: func.TimerRequest) -> None:
    """Timer triggered function - runs hourly"""
    utc_timestamp = datetime.utcnow().isoformat()
    logging.info(f'Timer trigger executed at {utc_timestamp}')
    
    if timer.past_due:
        logging.info('The timer is past due!')
    
    # ทำงาน scheduled
    cleanup_expired_sessions()
    send_hourly_digest()

def cleanup_expired_sessions():
    logging.info("Cleaning up expired sessions...")

def send_hourly_digest():
    logging.info("Sending hourly digest...")

# Blob Storage Trigger
@app.blob_trigger(
    arg_name="myblob",
    path="uploads/{name}",
    connection="AzureWebJobsStorage"
)
def blob_trigger(myblob: func.InputStream) -> None:
    """Triggered when a blob is created/updated"""
    logging.info(f"Blob trigger fired for: {myblob.name}")
    logging.info(f"Blob size: {myblob.length} bytes")
    
    content = myblob.read()
    process_uploaded_file(myblob.name, content)

def process_uploaded_file(name: str, content: bytes):
    logging.info(f"Processing file: {name}")

# Service Bus Queue Trigger
@app.service_bus_queue_trigger(
    arg_name="msg",
    queue_name="my-queue",
    connection="ServiceBusConnection"
)
def service_bus_trigger(msg: func.ServiceBusMessage) -> None:
    """Triggered by Service Bus message"""
    body = msg.get_body().decode('utf-8')
    data = json.loads(body)
    
    logging.info(f"Processing message: {data}")
    process_service_bus_message(data)

def process_service_bus_message(data: dict):
    logging.info(f"Action: {data.get('action')}")
```

---

## 11. Serverless Patterns กับ Python

### ตัวอย่างที่ 15: Serverless Framework Configuration

```yaml
# serverless.yml - Serverless Framework config
service: my-python-api

provider:
  name: aws
  runtime: python3.11
  region: ap-southeast-1
  memorySize: 256
  timeout: 30
  
  environment:
    ENVIRONMENT: ${opt:stage, 'dev'}
    TABLE_NAME: ${self:service}-${opt:stage, 'dev'}-users
  
  iam:
    role:
      statements:
        - Effect: Allow
          Action:
            - dynamodb:*
          Resource: !GetAtt UsersTable.Arn
        - Effect: Allow
          Action:
            - s3:GetObject
            - s3:PutObject
          Resource: !Sub "${FilesBucket.Arn}/*"

functions:
  api:
    handler: src/handler.main
    events:
      - http:
          path: /{proxy+}
          method: ANY
          cors: true
  
  processImage:
    handler: src/image_processor.handler
    memorySize: 1024
    timeout: 60
    events:
      - s3:
          bucket: !Ref FilesBucket
          event: s3:ObjectCreated:*
          rules:
            - prefix: uploads/
  
  dailyReport:
    handler: src/reports.generate
    events:
      - schedule: rate(1 day)

resources:
  Resources:
    UsersTable:
      Type: AWS::DynamoDB::Table
      Properties:
        TableName: ${self:provider.environment.TABLE_NAME}
        BillingMode: PAY_PER_REQUEST
        AttributeDefinitions:
          - AttributeName: id
            AttributeType: S
        KeySchema:
          - AttributeName: id
            KeyType: HASH
    
    FilesBucket:
      Type: AWS::S3::Bucket

plugins:
  - serverless-python-requirements
  - serverless-offline

custom:
  pythonRequirements:
    dockerizePip: non-linux
    slim: true
    strip: false
    noDeploy:
      - pytest
      - black
      - flake8
```

### ตัวอย่างที่ 16: AWS CDK สำหรับ Python

```python
# ตัวอย่าง 16: AWS CDK infrastructure as Python code
from aws_cdk import (
    App, Stack, Duration, RemovalPolicy,
    aws_lambda as lambda_,
    aws_apigateway as apigw,
    aws_dynamodb as dynamodb,
    aws_s3 as s3,
    aws_iam as iam,
    aws_events as events,
    aws_events_targets as targets,
)
from constructs import Construct

class PythonAppStack(Stack):
    """CDK Stack สำหรับ Python serverless application"""
    
    def __init__(self, scope: Construct, stack_id: str, **kwargs):
        super().__init__(scope, stack_id, **kwargs)
        
        # DynamoDB Table
        table = dynamodb.Table(
            self, "UsersTable",
            table_name="users",
            partition_key=dynamodb.Attribute(
                name="id",
                type=dynamodb.AttributeType.STRING
            ),
            billing_mode=dynamodb.BillingMode.PAY_PER_REQUEST,
            removal_policy=RemovalPolicy.DESTROY,   # dev เท่านั้น
            point_in_time_recovery=True,
        )
        
        # S3 Bucket
        bucket = s3.Bucket(
            self, "FilesBucket",
            removal_policy=RemovalPolicy.DESTROY,
            lifecycle_rules=[
                s3.LifecycleRule(
                    id="archive-old-files",
                    transitions=[
                        s3.Transition(
                            storage_class=s3.StorageClass.INFREQUENT_ACCESS,
                            transition_after=Duration.days(30)
                        )
                    ]
                )
            ]
        )
        
        # Lambda Layer สำหรับ dependencies
        dependencies_layer = lambda_.LayerVersion(
            self, "DependenciesLayer",
            code=lambda_.Code.from_asset("lambda_layer"),
            compatible_runtimes=[lambda_.Runtime.PYTHON_3_11],
            description="Python dependencies"
        )
        
        # Main API Lambda
        api_function = lambda_.Function(
            self, "ApiFunction",
            runtime=lambda_.Runtime.PYTHON_3_11,
            handler="src.handler.main",
            code=lambda_.Code.from_asset("src"),
            memory_size=256,
            timeout=Duration.seconds(30),
            layers=[dependencies_layer],
            environment={
                "TABLE_NAME": table.table_name,
                "BUCKET_NAME": bucket.bucket_name,
                "ENVIRONMENT": "production",
            }
        )
        
        # Grant permissions
        table.grant_read_write_data(api_function)
        bucket.grant_read_write(api_function)
        
        # API Gateway
        api = apigw.RestApi(
            self, "PythonApi",
            rest_api_name="Python App API",
            deploy_options=apigw.StageOptions(
                stage_name="prod",
                throttling_rate_limit=1000,
                throttling_burst_limit=500,
            ),
            default_cors_preflight_options=apigw.CorsOptions(
                allow_origins=apigw.Cors.ALL_ORIGINS,
                allow_methods=apigw.Cors.ALL_METHODS,
            )
        )
        
        proxy = api.root.add_resource("{proxy+}")
        proxy.add_method(
            "ANY",
            apigw.LambdaIntegration(api_function),
        )
        
        # Scheduled Lambda
        report_function = lambda_.Function(
            self, "ReportFunction",
            runtime=lambda_.Runtime.PYTHON_3_11,
            handler="src.reports.generate",
            code=lambda_.Code.from_asset("src"),
            timeout=Duration.minutes(5),
            environment={"ENVIRONMENT": "production"}
        )
        
        # EventBridge Rule
        events.Rule(
            self, "DailyReportRule",
            schedule=events.Schedule.cron(hour="2", minute="0"),
            targets=[targets.LambdaFunction(report_function)]
        )

# สร้าง app
app = App()
PythonAppStack(app, "PythonApp", env={'region': 'ap-southeast-1'})
app.synth()
```

---

## 12. Environment Variables และ Secrets บน Cloud

### ตัวอย่างที่ 17: การจัดการ Environment Variables

```python
# ตัวอย่าง 17: Config management สำหรับ cloud deployment
import os
from typing import Optional
from pydantic import BaseSettings, validator
import boto3
import json
from functools import lru_cache

class Settings(BaseSettings):
    """Application settings จาก environment variables"""
    
    # App Config
    app_name: str = "My Python App"
    environment: str = "development"
    debug: bool = False
    log_level: str = "INFO"
    
    # Server
    host: str = "0.0.0.0"
    port: int = 8000
    workers: int = 4
    
    # Database
    database_url: Optional[str] = None
    db_pool_size: int = 10
    db_max_overflow: int = 20
    
    # AWS
    aws_region: str = "ap-southeast-1"
    aws_s3_bucket: Optional[str] = None
    
    # Security
    secret_key: str = "dev-secret-key-change-in-production"
    jwt_algorithm: str = "HS256"
    jwt_expire_minutes: int = 30
    
    # External Services
    redis_url: str = "redis://localhost:6379"
    
    @validator('environment')
    def validate_environment(cls, v):
        allowed = ['development', 'staging', 'production', 'test']
        if v not in allowed:
            raise ValueError(f"environment must be one of {allowed}")
        return v
    
    @validator('secret_key')
    def validate_secret_key(cls, v, values):
        if values.get('environment') == 'production' and v == 'dev-secret-key-change-in-production':
            raise ValueError("SECRET_KEY must be changed in production!")
        return v
    
    class Config:
        env_file = ".env"
        env_file_encoding = 'utf-8'
        case_sensitive = False

@lru_cache()
def get_settings() -> Settings:
    """Singleton settings instance"""
    settings = Settings()
    
    # ถ้าอยู่บน AWS ให้ดึงค่าจาก Secrets Manager
    if settings.environment == 'production':
        _load_aws_secrets(settings)
    
    return settings

def _load_aws_secrets(settings: Settings) -> None:
    """โหลด secrets จาก AWS Secrets Manager"""
    client = boto3.client('secretsmanager', region_name=settings.aws_region)
    
    try:
        response = client.get_secret_value(
            SecretId=f"{settings.environment}/myapp/secrets"
        )
        secrets = json.loads(response['SecretString'])
        
        # อัปเดตค่าจาก secrets
        if 'database_url' in secrets:
            object.__setattr__(settings, 'database_url', secrets['database_url'])
        if 'secret_key' in secrets:
            object.__setattr__(settings, 'secret_key', secrets['secret_key'])
    
    except Exception as e:
        print(f"Warning: Could not load AWS secrets: {e}")

# ใช้งาน
settings = get_settings()
print(f"App: {settings.app_name}")
print(f"Environment: {settings.environment}")
print(f"Debug: {settings.debug}")
```

---

## 13. Infrastructure as Code (Terraform)

### ตัวอย่างที่ 18: Terraform สำหรับ Python App

```hcl
# main.tf - Terraform configuration สำหรับ Python web app

terraform {
  required_version = ">= 1.5"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  
  # เก็บ state ใน S3
  backend "s3" {
    bucket = "my-terraform-state"
    key    = "python-app/terraform.tfstate"
    region = "ap-southeast-1"
  }
}

provider "aws" {
  region = var.aws_region
}

# Variables
variable "aws_region" {
  default = "ap-southeast-1"
}

variable "app_name" {
  default = "my-python-app"
}

variable "environment" {
  default = "production"
}

# ECR Repository
resource "aws_ecr_repository" "app" {
  name                 = var.app_name
  image_tag_mutability = "MUTABLE"

  image_scanning_configuration {
    scan_on_push = true
  }
}

# ECS Cluster
resource "aws_ecs_cluster" "main" {
  name = "${var.app_name}-cluster"
  
  setting {
    name  = "containerInsights"
    value = "enabled"
  }
}

# RDS Instance
resource "aws_db_instance" "postgres" {
  identifier        = "${var.app_name}-db"
  engine            = "postgres"
  engine_version    = "15.3"
  instance_class    = "db.t3.micro"
  allocated_storage = 20
  
  db_name  = "appdb"
  username = "appuser"
  password = var.db_password
  
  vpc_security_group_ids = [aws_security_group.rds.id]
  db_subnet_group_name   = aws_db_subnet_group.main.name
  
  skip_final_snapshot = false
  final_snapshot_identifier = "${var.app_name}-final-snapshot"
  
  backup_retention_period = 7
  backup_window          = "03:00-04:00"
  maintenance_window     = "Mon:04:00-Mon:05:00"
  
  tags = {
    Name        = "${var.app_name}-db"
    Environment = var.environment
  }
}

# Outputs
output "ecr_repository_url" {
  value = aws_ecr_repository.app.repository_url
}

output "rds_endpoint" {
  value = aws_db_instance.postgres.endpoint
}
```

### ตัวอย่างที่ 19: Python Terraform Wrapper

```python
# ตัวอย่าง 19: Python wrapper สำหรับ Terraform
import subprocess
import json
import os
from pathlib import Path
from typing import Dict, Any, Optional

class TerraformManager:
    """Python wrapper สำหรับ Terraform commands"""
    
    def __init__(self, working_dir: str = '.', var_file: str = None):
        self.working_dir = Path(working_dir)
        self.var_file = var_file
        self._check_terraform()
    
    def _check_terraform(self):
        """ตรวจสอบว่ามี terraform ติดตั้งแล้ว"""
        result = subprocess.run(
            ['terraform', 'version', '-json'],
            capture_output=True, text=True
        )
        if result.returncode != 0:
            raise RuntimeError("Terraform not installed!")
        
        version_info = json.loads(result.stdout)
        print(f"Terraform version: {version_info['terraform_version']}")
    
    def _run(self, *args, auto_approve: bool = False) -> str:
        """รัน terraform command"""
        cmd = ['terraform'] + list(args)
        
        if auto_approve and 'apply' in args:
            cmd.append('-auto-approve')
        
        if self.var_file:
            cmd.extend(['-var-file', self.var_file])
        
        result = subprocess.run(
            cmd,
            cwd=self.working_dir,
            capture_output=True,
            text=True
        )
        
        if result.returncode != 0:
            raise RuntimeError(f"Terraform error:\n{result.stderr}")
        
        return result.stdout
    
    def init(self) -> str:
        """terraform init"""
        print("Initializing Terraform...")
        return self._run('init', '-upgrade')
    
    def plan(self, out_file: str = 'tfplan') -> str:
        """terraform plan"""
        print("Planning changes...")
        return self._run('plan', '-out', out_file)
    
    def apply(self, plan_file: str = 'tfplan', auto_approve: bool = False) -> str:
        """terraform apply"""
        print("Applying changes...")
        if plan_file:
            return self._run('apply', plan_file, auto_approve=auto_approve)
        return self._run('apply', auto_approve=auto_approve)
    
    def destroy(self, auto_approve: bool = False) -> str:
        """terraform destroy"""
        print("Destroying resources...")
        return self._run('destroy', auto_approve=auto_approve)
    
    def output(self) -> Dict[str, Any]:
        """ดึง terraform outputs"""
        result = self._run('output', '-json')
        outputs = json.loads(result)
        return {k: v['value'] for k, v in outputs.items()}
    
    def state_list(self) -> list:
        """แสดงรายการ resources ใน state"""
        result = self._run('state', 'list')
        return [r.strip() for r in result.splitlines() if r.strip()]
    
    def import_resource(self, resource_address: str, resource_id: str) -> str:
        """import existing resource เข้า Terraform state"""
        return self._run('import', resource_address, resource_id)

# ตัวอย่างการใช้งาน
def deploy_infrastructure():
    tf = TerraformManager('./infrastructure', var_file='production.tfvars')
    
    # Initialize
    tf.init()
    
    # Plan
    plan_output = tf.plan()
    print("Plan output:", plan_output[:500])
    
    # Apply (ใน production ควรให้คน approve ก่อน)
    # tf.apply(auto_approve=True)  # CI/CD only!
    
    # แสดง outputs
    # outputs = tf.output()
    # print(f"ECR URL: {outputs.get('ecr_repository_url')}")
    
    print("Infrastructure deployment ready")

print("TerraformManager ready")
```

---

## 14. ตัวอย่างการ Deploy แบบครบวงจร

### ตัวอย่างที่ 20: Complete AWS Deployment Script

```python
# ตัวอย่าง 20: Complete deployment script
import boto3
import subprocess
import json
import sys
import os
from datetime import datetime
from typing import Optional

class AppDeployer:
    """Complete deployment pipeline สำหรับ Python app บน AWS"""
    
    def __init__(
        self,
        app_name: str,
        environment: str,
        region: str = 'ap-southeast-1'
    ):
        self.app_name = app_name
        self.environment = environment
        self.region = region
        self.image_tag = datetime.now().strftime('%Y%m%d-%H%M%S')
        
        # AWS clients
        self.ecr = boto3.client('ecr', region_name=region)
        self.ecs = boto3.client('ecs', region_name=region)
        self.ssm = boto3.client('ssm', region_name=region)
        
        # ดึง account ID
        sts = boto3.client('sts', region_name=region)
        self.account_id = sts.get_caller_identity()['Account']
        
        self.ecr_registry = f"{self.account_id}.dkr.ecr.{region}.amazonaws.com"
        self.image_name = f"{self.ecr_registry}/{app_name}:{self.image_tag}"
    
    def pre_deploy_checks(self) -> bool:
        """ตรวจสอบก่อน deploy"""
        print("\n=== Pre-deploy Checks ===")
        
        # ตรวจสอบ tests
        print("1. Running tests...")
        result = subprocess.run(
            ['pytest', 'tests/', '--tb=short', '-q'],
            capture_output=True, text=True
        )
        
        if result.returncode != 0:
            print(f"Tests FAILED:\n{result.stdout}")
            return False
        
        print("   Tests PASSED ✅")
        
        # ตรวจสอบ code quality
        print("2. Running lint checks...")
        result = subprocess.run(
            ['flake8', 'src/', '--max-line-length=88'],
            capture_output=True, text=True
        )
        
        if result.returncode != 0:
            print(f"Lint FAILED:\n{result.stdout}")
            return False
        
        print("   Lint PASSED ✅")
        return True
    
    def build_image(self) -> bool:
        """Build Docker image"""
        print(f"\n=== Building Image: {self.image_name} ===")
        
        result = subprocess.run(
            ['docker', 'build', '-t', self.image_name, '.'],
            check=False
        )
        
        return result.returncode == 0
    
    def push_image(self) -> bool:
        """Push image ไปยัง ECR"""
        print(f"\n=== Pushing Image to ECR ===")
        
        # Authenticate กับ ECR
        token = self.ecr.get_authorization_token()['authorizationData'][0]
        auth = token['authorizationToken']
        
        subprocess.run([
            'docker', 'login', '--username', 'AWS',
            '--password-stdin', self.ecr_registry
        ], input=auth, text=True, check=True, capture_output=True)
        
        result = subprocess.run(
            ['docker', 'push', self.image_name],
            check=False
        )
        
        return result.returncode == 0
    
    def update_ecs_service(self) -> bool:
        """อัปเดต ECS service ด้วย image ใหม่"""
        print(f"\n=== Updating ECS Service ===")
        
        cluster = f"{self.app_name}-{self.environment}"
        service = f"{self.app_name}-service"
        
        # ดึง current task definition
        response = self.ecs.describe_services(
            cluster=cluster, services=[service]
        )
        
        task_def_arn = response['services'][0]['taskDefinition']
        task_def = self.ecs.describe_task_definition(
            taskDefinition=task_def_arn
        )['taskDefinition']
        
        # อัปเดต image
        for container in task_def['containerDefinitions']:
            if container['name'] == self.app_name:
                container['image'] = self.image_name
        
        # Register new task definition
        new_task_def = self.ecs.register_task_definition(
            family=task_def['family'],
            containerDefinitions=task_def['containerDefinitions'],
            networkMode=task_def['networkMode'],
            requiresCompatibilities=task_def['requiresCompatibilities'],
            cpu=task_def['cpu'],
            memory=task_def['memory'],
        )
        
        new_task_arn = new_task_def['taskDefinition']['taskDefinitionArn']
        
        # อัปเดต service
        self.ecs.update_service(
            cluster=cluster,
            service=service,
            taskDefinition=new_task_arn,
            forceNewDeployment=True
        )
        
        print(f"Updated service with task: {new_task_arn}")
        return True
    
    def deploy(self) -> bool:
        """Full deployment pipeline"""
        print(f"\n{'='*60}")
        print(f"Deploying {self.app_name} to {self.environment}")
        print(f"Image tag: {self.image_tag}")
        print(f"{'='*60}")
        
        steps = [
            ("Pre-deploy checks", self.pre_deploy_checks),
            ("Build image", self.build_image),
            ("Push image", self.push_image),
            ("Update ECS service", self.update_ecs_service),
        ]
        
        for step_name, step_func in steps:
            print(f"\nRunning: {step_name}...")
            if not step_func():
                print(f"\n❌ FAILED at: {step_name}")
                return False
            print(f"✅ {step_name} completed")
        
        print(f"\n{'='*60}")
        print(f"✅ Deployment SUCCESSFUL!")
        print(f"Image: {self.image_name}")
        print(f"{'='*60}")
        return True

# ตัวอย่างการใช้งาน
# deployer = AppDeployer('my-python-app', 'production')
# success = deployer.deploy()
# sys.exit(0 if success else 1)

print("AppDeployer ready for production deployment")
```

### ตัวอย่างที่ 21: Health Check และ Monitoring

```python
# ตัวอย่าง 21: Health Check endpoint สำหรับ cloud deployment
from fastapi import FastAPI, status
from pydantic import BaseModel
from typing import Dict, Any, Optional
import os
import time
import psutil
import boto3
from datetime import datetime, timezone

app = FastAPI()

class HealthStatus(BaseModel):
    status: str
    version: str
    environment: str
    timestamp: str
    uptime_seconds: float
    checks: Dict[str, Any]

# เก็บเวลา start
START_TIME = time.time()

@app.get("/health", response_model=HealthStatus)
async def health_check() -> HealthStatus:
    """Comprehensive health check endpoint"""
    
    checks = {}
    overall_status = "healthy"
    
    # Database check
    try:
        # from app.database import engine
        # with engine.connect() as conn:
        #     conn.execute(text("SELECT 1"))
        checks['database'] = {'status': 'ok', 'latency_ms': 5}
    except Exception as e:
        checks['database'] = {'status': 'error', 'error': str(e)}
        overall_status = "degraded"
    
    # Memory check
    memory = psutil.virtual_memory()
    memory_pct = memory.percent
    checks['memory'] = {
        'status': 'ok' if memory_pct < 90 else 'warning',
        'used_percent': memory_pct,
        'available_mb': memory.available // 1024 // 1024
    }
    
    if memory_pct > 95:
        overall_status = "degraded"
    
    # Disk check
    disk = psutil.disk_usage('/')
    disk_pct = disk.percent
    checks['disk'] = {
        'status': 'ok' if disk_pct < 85 else 'warning',
        'used_percent': disk_pct,
        'free_gb': disk.free // 1024 // 1024 // 1024
    }
    
    return HealthStatus(
        status=overall_status,
        version=os.environ.get('APP_VERSION', '1.0.0'),
        environment=os.environ.get('ENVIRONMENT', 'unknown'),
        timestamp=datetime.now(timezone.utc).isoformat(),
        uptime_seconds=time.time() - START_TIME,
        checks=checks
    )

@app.get("/ready")
async def readiness_check():
    """Kubernetes readiness probe"""
    # ตรวจสอบว่าพร้อมรับ traffic แล้ว
    return {"ready": True}

@app.get("/live") 
async def liveness_check():
    """Kubernetes liveness probe"""
    return {"alive": True}
```

### ตัวอย่างที่ 22: Cloud-Native Logging

```python
# ตัวอย่าง 22: Structured logging สำหรับ cloud
import json
import logging
import os
import sys
from datetime import datetime, timezone
from typing import Any, Dict, Optional
import traceback

class CloudLogger:
    """Structured logger สำหรับ cloud environments"""
    
    def __init__(self, service_name: str, version: str = '1.0.0'):
        self.service = service_name
        self.version = version
        self.environment = os.environ.get('ENVIRONMENT', 'development')
        
        # ตั้งค่า Python logger
        self._logger = logging.getLogger(service_name)
        handler = logging.StreamHandler(sys.stdout)
        handler.setFormatter(logging.Formatter('%(message)s'))
        self._logger.addHandler(handler)
        self._logger.setLevel(logging.DEBUG)
    
    def _format(self, level: str, message: str, **extra) -> str:
        """สร้าง structured log entry"""
        entry = {
            'timestamp': datetime.now(timezone.utc).isoformat(),
            'level': level,
            'service': self.service,
            'version': self.version,
            'environment': self.environment,
            'message': message,
        }
        entry.update(extra)
        return json.dumps(entry, default=str)
    
    def info(self, message: str, **extra):
        self._logger.info(self._format('INFO', message, **extra))
    
    def warning(self, message: str, **extra):
        self._logger.warning(self._format('WARNING', message, **extra))
    
    def error(self, message: str, exc: Exception = None, **extra):
        if exc:
            extra['error'] = str(exc)
            extra['traceback'] = traceback.format_exc()
        self._logger.error(self._format('ERROR', message, **extra))
    
    def request(self, method: str, path: str, status: int, duration_ms: float, **extra):
        """Log HTTP request"""
        self._logger.info(self._format(
            'INFO', f"{method} {path} {status}",
            http_method=method,
            http_path=path,
            http_status=status,
            duration_ms=duration_ms,
            **extra
        ))

# ใช้งาน
logger = CloudLogger('my-python-app', '1.0.0')
logger.info("Application started", host="0.0.0.0", port=8000)
logger.request("GET", "/users", 200, 45.2, user_count=10)
```

### ตัวอย่างที่ 23: Auto Scaling Configuration

```python
# ตัวอย่าง 23: Auto Scaling กับ AWS Application Auto Scaling
import boto3

class AutoScalingManager:
    """จัดการ Auto Scaling สำหรับ ECS services"""
    
    def __init__(self, region: str = 'ap-southeast-1'):
        self.client = boto3.client(
            'application-autoscaling',
            region_name=region
        )
        self.cloudwatch = boto3.client('cloudwatch', region_name=region)
    
    def register_scalable_target(
        self,
        cluster: str,
        service: str,
        min_capacity: int = 1,
        max_capacity: int = 10
    ) -> None:
        """ลงทะเบียน ECS service เป็น scalable target"""
        self.client.register_scalable_target(
            ServiceNamespace='ecs',
            ResourceId=f'service/{cluster}/{service}',
            ScalableDimension='ecs:service:DesiredCount',
            MinCapacity=min_capacity,
            MaxCapacity=max_capacity,
        )
        print(f"Registered scalable target: {service} ({min_capacity}-{max_capacity})")
    
    def set_cpu_scaling_policy(
        self,
        cluster: str,
        service: str,
        target_cpu: float = 70.0
    ) -> str:
        """ตั้งค่า CPU-based auto scaling"""
        response = self.client.put_scaling_policy(
            PolicyName=f'{service}-cpu-scaling',
            ServiceNamespace='ecs',
            ResourceId=f'service/{cluster}/{service}',
            ScalableDimension='ecs:service:DesiredCount',
            PolicyType='TargetTrackingScaling',
            TargetTrackingScalingPolicyConfiguration={
                'TargetValue': target_cpu,
                'PredefinedMetricSpecification': {
                    'PredefinedMetricType': 'ECSServiceAverageCPUUtilization'
                },
                'ScaleInCooldown': 300,
                'ScaleOutCooldown': 60,
            }
        )
        
        policy_arn = response['PolicyARN']
        print(f"Set CPU scaling policy: target {target_cpu}% CPU")
        return policy_arn
    
    def set_request_scaling_policy(
        self,
        cluster: str,
        service: str,
        alb_arn_suffix: str,
        target_group_arn_suffix: str,
        requests_per_target: int = 1000
    ) -> str:
        """ตั้งค่า Request Count-based auto scaling"""
        response = self.client.put_scaling_policy(
            PolicyName=f'{service}-request-scaling',
            ServiceNamespace='ecs',
            ResourceId=f'service/{cluster}/{service}',
            ScalableDimension='ecs:service:DesiredCount',
            PolicyType='TargetTrackingScaling',
            TargetTrackingScalingPolicyConfiguration={
                'TargetValue': float(requests_per_target),
                'PredefinedMetricSpecification': {
                    'PredefinedMetricType': 'ALBRequestCountPerTarget',
                    'ResourceLabel': f'{alb_arn_suffix}/{target_group_arn_suffix}'
                },
            }
        )
        
        print(f"Set request-based scaling: {requests_per_target} req/target")
        return response['PolicyARN']

print("AutoScalingManager ready")
```

### ตัวอย่างที่ 24: Database Migration บน Cloud

```python
# ตัวอย่าง 24: Database migration ใน cloud environment
import os
import subprocess
import sys
import boto3
import json
from pathlib import Path

def run_migrations_on_lambda():
    """รัน database migrations ใน Lambda function"""
    
    # ดึง database URL จาก Secrets Manager
    client = boto3.client('secretsmanager')
    secret = json.loads(
        client.get_secret_value(
            SecretId='prod/myapp/database'
        )['SecretString']
    )
    
    os.environ['DATABASE_URL'] = (
        f"postgresql://{secret['username']}:{secret['password']}"
        f"@{secret['host']}/{secret['dbname']}"
    )
    
    # รัน Alembic migrations
    result = subprocess.run(
        ['alembic', 'upgrade', 'head'],
        capture_output=True,
        text=True
    )
    
    if result.returncode != 0:
        print(f"Migration failed:\n{result.stderr}")
        raise RuntimeError("Database migration failed")
    
    print(f"Migration succeeded:\n{result.stdout}")

def lambda_migration_handler(event, context):
    """Lambda handler สำหรับ run migrations"""
    try:
        run_migrations_on_lambda()
        return {'statusCode': 200, 'body': 'Migration successful'}
    except Exception as e:
        return {'statusCode': 500, 'body': str(e)}

# ECS Task สำหรับ migration
def run_migration_task():
    """รัน migration ในฐานะ ECS task ก่อน deploy"""
    ecs = boto3.client('ecs')
    
    response = ecs.run_task(
        cluster='my-app-cluster',
        taskDefinition='my-app-migration',
        launchType='FARGATE',
        networkConfiguration={
            'awsvpcConfiguration': {
                'subnets': ['subnet-12345'],
                'securityGroups': ['sg-12345'],
                'assignPublicIp': 'ENABLED'
            }
        }
    )
    
    task_arn = response['tasks'][0]['taskArn']
    print(f"Running migration task: {task_arn}")
    
    # รอให้ task เสร็จ
    waiter = ecs.get_waiter('tasks_stopped')
    waiter.wait(
        cluster='my-app-cluster',
        tasks=[task_arn]
    )
    
    # ตรวจสอบผลลัพธ์
    task_info = ecs.describe_tasks(
        cluster='my-app-cluster',
        tasks=[task_arn]
    )['tasks'][0]
    
    exit_code = task_info['containers'][0].get('exitCode', -1)
    if exit_code != 0:
        raise RuntimeError(f"Migration task failed with exit code {exit_code}")
    
    print("Migration completed successfully!")

print("Database migration utilities ready")
```

### ตัวอย่างที่ 25: Multi-Cloud Deployment Abstraction

```python
# ตัวอย่าง 25: Abstraction layer สำหรับ multi-cloud deployment
from abc import ABC, abstractmethod
from dataclasses import dataclass
from typing import Optional, Dict, List

@dataclass
class DeploymentConfig:
    app_name: str
    image: str
    cpu: str
    memory: str
    min_instances: int
    max_instances: int
    port: int
    env_vars: Dict[str, str]

class CloudProvider(ABC):
    """Abstract base class สำหรับ cloud providers"""
    
    @abstractmethod
    def deploy(self, config: DeploymentConfig) -> str:
        """Deploy application, return URL"""
        pass
    
    @abstractmethod
    def scale(self, app_name: str, instances: int) -> None:
        """Scale application"""
        pass
    
    @abstractmethod
    def get_logs(self, app_name: str, lines: int = 100) -> List[str]:
        """ดึง application logs"""
        pass
    
    @abstractmethod
    def delete(self, app_name: str) -> None:
        """ลบ application"""
        pass

class AWSFargateProvider(CloudProvider):
    """Deploy ไปยัง AWS Fargate"""
    
    def __init__(self, cluster: str, region: str = 'ap-southeast-1'):
        import boto3
        self.ecs = boto3.client('ecs', region_name=region)
        self.cluster = cluster
        self.region = region
    
    def deploy(self, config: DeploymentConfig) -> str:
        print(f"Deploying {config.app_name} to AWS Fargate...")
        # ECS deployment logic
        return f"https://{config.app_name}.{self.region}.elb.amazonaws.com"
    
    def scale(self, app_name: str, instances: int) -> None:
        print(f"Scaling {app_name} to {instances} instances on AWS Fargate")
    
    def get_logs(self, app_name: str, lines: int = 100) -> List[str]:
        return [f"[AWS Fargate] Log line {i}" for i in range(lines)]
    
    def delete(self, app_name: str) -> None:
        print(f"Deleting {app_name} from AWS Fargate")

class GCPCloudRunProvider(CloudProvider):
    """Deploy ไปยัง GCP Cloud Run"""
    
    def __init__(self, project_id: str, region: str = 'asia-southeast1'):
        self.project = project_id
        self.region = region
    
    def deploy(self, config: DeploymentConfig) -> str:
        print(f"Deploying {config.app_name} to GCP Cloud Run...")
        return f"https://{config.app_name}-{self.project}.{self.region}.run.app"
    
    def scale(self, app_name: str, instances: int) -> None:
        print(f"Scaling {app_name} to {instances} instances on Cloud Run")
    
    def get_logs(self, app_name: str, lines: int = 100) -> List[str]:
        return [f"[Cloud Run] Log line {i}" for i in range(lines)]
    
    def delete(self, app_name: str) -> None:
        print(f"Deleting {app_name} from Cloud Run")

class AzureContainerProvider(CloudProvider):
    """Deploy ไปยัง Azure Container Apps"""
    
    def __init__(self, resource_group: str, region: str = 'Southeast Asia'):
        self.resource_group = resource_group
        self.region = region
    
    def deploy(self, config: DeploymentConfig) -> str:
        print(f"Deploying {config.app_name} to Azure Container Apps...")
        return f"https://{config.app_name}.azurecontainerapps.io"
    
    def scale(self, app_name: str, instances: int) -> None:
        print(f"Scaling {app_name} to {instances} instances on Azure")
    
    def get_logs(self, app_name: str, lines: int = 100) -> List[str]:
        return [f"[Azure] Log line {i}" for i in range(lines)]
    
    def delete(self, app_name: str) -> None:
        print(f"Deleting {app_name} from Azure")

class MultiCloudDeployer:
    """Deploy ไปยัง multiple cloud providers"""
    
    def __init__(self):
        self.providers: Dict[str, CloudProvider] = {}
    
    def add_provider(self, name: str, provider: CloudProvider) -> 'MultiCloudDeployer':
        self.providers[name] = provider
        return self
    
    def deploy_all(self, config: DeploymentConfig) -> Dict[str, str]:
        """Deploy ไปทุก providers พร้อมกัน"""
        urls = {}
        for name, provider in self.providers.items():
            try:
                url = provider.deploy(config)
                urls[name] = url
                print(f"✅ {name}: {url}")
            except Exception as e:
                urls[name] = f"FAILED: {e}"
                print(f"❌ {name}: {e}")
        return urls

# ตัวอย่างการใช้งาน
config = DeploymentConfig(
    app_name='my-python-app',
    image='my-registry/my-python-app:latest',
    cpu='0.5',
    memory='512Mi',
    min_instances=1,
    max_instances=10,
    port=8000,
    env_vars={'ENVIRONMENT': 'production', 'LOG_LEVEL': 'INFO'}
)

deployer = (MultiCloudDeployer()
    .add_provider('aws', AWSFargateProvider('my-cluster'))
    .add_provider('gcp', GCPCloudRunProvider('my-project'))
    .add_provider('azure', AzureContainerProvider('my-resource-group')))

# urls = deployer.deploy_all(config)
print("Multi-cloud deployer configured!")
print(f"Config: {config.app_name} → {config.image}")
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: S3 File Manager
เขียน Python script ที่ทำงานกับ S3:
- Upload หลายไฟล์พร้อมกันด้วย concurrent uploads
- Generate presigned URLs สำหรับ temporary download access
- List ไฟล์ใน bucket พร้อม pagination

**เฉลย:**
```python
import boto3
import concurrent.futures
from pathlib import Path
from typing import List, Dict

class ConcurrentS3Manager:
    def __init__(self, bucket: str):
        self.bucket = bucket
        self.s3 = boto3.client('s3')
    
    def upload_files_concurrent(
        self, 
        files: List[str], 
        max_workers: int = 5
    ) -> Dict[str, str]:
        """Upload หลายไฟล์พร้อมกัน"""
        results = {}
        
        def upload_single(file_path: str) -> tuple:
            key = Path(file_path).name
            try:
                self.s3.upload_file(file_path, self.bucket, key)
                return key, "success"
            except Exception as e:
                return key, f"failed: {e}"
        
        with concurrent.futures.ThreadPoolExecutor(max_workers=max_workers) as executor:
            futures = {executor.submit(upload_single, f): f for f in files}
            
            for future in concurrent.futures.as_completed(futures):
                key, status = future.result()
                results[key] = status
                print(f"  {key}: {status}")
        
        return results
    
    def get_presigned_url(self, key: str, expires_in: int = 3600) -> str:
        return self.s3.generate_presigned_url(
            'get_object',
            Params={'Bucket': self.bucket, 'Key': key},
            ExpiresIn=expires_in
        )
    
    def list_all_files(self, prefix: str = '') -> List[Dict]:
        """List ไฟล์ทั้งหมดด้วย pagination"""
        files = []
        paginator = self.s3.get_paginator('list_objects_v2')
        
        for page in paginator.paginate(Bucket=self.bucket, Prefix=prefix):
            for obj in page.get('Contents', []):
                files.append({
                    'key': obj['Key'],
                    'size': obj['Size'],
                    'last_modified': obj['LastModified'].isoformat()
                })
        
        return files

# manager = ConcurrentS3Manager('my-bucket')
# results = manager.upload_files_concurrent(['file1.txt', 'file2.txt', 'file3.txt'])
print("ConcurrentS3Manager solution ready")
```

### แบบฝึกหัดที่ 2: Lambda CRUD API
สร้าง Lambda function ที่ทำ CRUD operations บน DynamoDB สำหรับ Todo items

**เฉลย:**
```python
import json
import boto3
import uuid
from datetime import datetime

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('todos')

def lambda_handler(event, context):
    method = event['httpMethod']
    path = event['path']
    
    try:
        if method == 'GET' and path == '/todos':
            return list_todos()
        elif method == 'POST' and path == '/todos':
            body = json.loads(event['body'])
            return create_todo(body)
        elif method == 'PUT' and path.startswith('/todos/'):
            todo_id = event['pathParameters']['id']
            body = json.loads(event['body'])
            return update_todo(todo_id, body)
        elif method == 'DELETE' and path.startswith('/todos/'):
            todo_id = event['pathParameters']['id']
            return delete_todo(todo_id)
        else:
            return resp(404, {'error': 'Not found'})
    except Exception as e:
        return resp(500, {'error': str(e)})

def list_todos():
    result = table.scan()
    return resp(200, {'todos': result['Items']})

def create_todo(body):
    todo = {
        'id': str(uuid.uuid4()),
        'title': body['title'],
        'done': False,
        'created_at': datetime.utcnow().isoformat()
    }
    table.put_item(Item=todo)
    return resp(201, todo)

def update_todo(todo_id, body):
    update_expr = "SET #t = :t, done = :d"
    table.update_item(
        Key={'id': todo_id},
        UpdateExpression=update_expr,
        ExpressionAttributeNames={'#t': 'title'},
        ExpressionAttributeValues={
            ':t': body.get('title', ''),
            ':d': body.get('done', False)
        }
    )
    return resp(200, {'id': todo_id, 'updated': True})

def delete_todo(todo_id):
    table.delete_item(Key={'id': todo_id})
    return resp(200, {'deleted': True})

def resp(code, body):
    return {
        'statusCode': code,
        'headers': {'Content-Type': 'application/json'},
        'body': json.dumps(body, default=str)
    }
```

### แบบฝึกหัดที่ 3: Cloud Run Deployment
เขียน script ที่ build Docker image แล้ว deploy ไปยัง Cloud Run โดยอัตโนมัติ

**เฉลย:**
```python
import subprocess
import sys

def deploy_to_cloud_run(
    project_id: str,
    service_name: str,
    region: str = 'asia-southeast1'
):
    image = f"gcr.io/{project_id}/{service_name}:latest"
    
    # Build และ push ด้วย Cloud Build
    subprocess.run([
        'gcloud', 'builds', 'submit',
        '--image', image, '.'
    ], check=True)
    
    # Deploy ไปยัง Cloud Run
    subprocess.run([
        'gcloud', 'run', 'deploy', service_name,
        '--image', image,
        '--platform', 'managed',
        '--region', region,
        '--allow-unauthenticated',
        '--min-instances', '1',
        '--max-instances', '10',
        '--memory', '512Mi',
        '--cpu', '1',
    ], check=True)
    
    # ดึง URL
    result = subprocess.run([
        'gcloud', 'run', 'services', 'describe', service_name,
        '--region', region,
        '--format', 'value(status.url)'
    ], capture_output=True, text=True, check=True)
    
    url = result.stdout.strip()
    print(f"Deployed to: {url}")
    return url

# deploy_to_cloud_run('my-project', 'my-python-app')
print("Cloud Run deployment function ready")
```

### แบบฝึกหัดที่ 4: Settings Validation
สร้าง Pydantic settings class สำหรับ multi-cloud app ที่ validate environment variables

**เฉลย:**
```python
import os
from pydantic import BaseSettings, validator, AnyUrl
from typing import Optional, Literal

class ProductionSettings(BaseSettings):
    # Cloud Provider
    cloud_provider: Literal['aws', 'gcp', 'azure'] = 'aws'
    
    # App
    environment: Literal['staging', 'production']
    secret_key: str
    
    # Database
    database_url: str
    
    # AWS specific
    aws_region: Optional[str] = None
    aws_s3_bucket: Optional[str] = None
    
    # GCP specific
    gcp_project_id: Optional[str] = None
    
    # Azure specific
    azure_subscription_id: Optional[str] = None
    
    @validator('database_url')
    def validate_database_url(cls, v):
        if not (v.startswith('postgresql://') or v.startswith('mysql://')):
            raise ValueError('Only PostgreSQL and MySQL are supported')
        return v
    
    @validator('aws_region')
    def validate_aws_config(cls, v, values):
        if values.get('cloud_provider') == 'aws' and not v:
            raise ValueError('aws_region required when cloud_provider=aws')
        return v
    
    @validator('gcp_project_id')
    def validate_gcp_config(cls, v, values):
        if values.get('cloud_provider') == 'gcp' and not v:
            raise ValueError('gcp_project_id required when cloud_provider=gcp')
        return v
    
    class Config:
        env_file = '.env.production'

# ทดสอบ
os.environ.update({
    'CLOUD_PROVIDER': 'aws',
    'ENVIRONMENT': 'staging',
    'SECRET_KEY': 'production-secret-key-here',
    'DATABASE_URL': 'postgresql://user:pass@host/db',
    'AWS_REGION': 'ap-southeast-1',
})

settings = ProductionSettings()
print(f"Cloud: {settings.cloud_provider}")
print(f"Environment: {settings.environment}")
print(f"Database configured: {'Yes' if settings.database_url else 'No'}")
```

### แบบฝึกหัดที่ 5: Terraform Module
เขียน Terraform configuration สำหรับ deploy Python app บน AWS ที่รวม: ECS, ALB, RDS, ElastiCache

**เฉลย:**
```hcl
# modules/python-app/main.tf
variable "app_name" { type = string }
variable "environment" { type = string }
variable "docker_image" { type = string }
variable "vpc_id" { type = string }
variable "subnet_ids" { type = list(string) }

# ECS Cluster
resource "aws_ecs_cluster" "this" {
  name = "${var.app_name}-${var.environment}"
}

# ECS Task Definition
resource "aws_ecs_task_definition" "app" {
  family                   = "${var.app_name}-${var.environment}"
  network_mode             = "awsvpc"
  requires_compatibilities = ["FARGATE"]
  cpu                      = 256
  memory                   = 512
  execution_role_arn       = aws_iam_role.ecs_execution.arn

  container_definitions = jsonencode([{
    name  = var.app_name
    image = var.docker_image
    portMappings = [{
      containerPort = 8000
      protocol      = "tcp"
    }]
    environment = [
      { name = "ENVIRONMENT", value = var.environment }
    ]
    logConfiguration = {
      logDriver = "awslogs"
      options = {
        awslogs-group  = "/ecs/${var.app_name}"
        awslogs-region = data.aws_region.current.name
      }
    }
  }])
}

# ALB
resource "aws_lb" "this" {
  name               = "${var.app_name}-alb"
  internal           = false
  load_balancer_type = "application"
  subnets            = var.subnet_ids
}

data "aws_region" "current" {}

output "alb_dns_name" {
  value = aws_lb.this.dns_name
}
```

### แบบฝึกหัดที่ 6: Complete Serverless App
สร้าง serverless Python app ที่รับ image upload ผ่าน API Gateway → S3 → Lambda ประมวลผล → DynamoDB บันทึกผล

**เฉลย:**
```python
# handlers/upload.py - API Gateway handler รับ image upload
import boto3
import json
import uuid
import base64
import os
from datetime import datetime

s3 = boto3.client('s3')
dynamodb = boto3.resource('dynamodb')

BUCKET = os.environ['UPLOAD_BUCKET']
TABLE = dynamodb.Table(os.environ['METADATA_TABLE'])

def upload_handler(event, context):
    """รับ base64 encoded image และ upload ไปยัง S3"""
    body = json.loads(event['body'])
    
    image_data = base64.b64decode(body['image'])
    image_id = str(uuid.uuid4())
    file_ext = body.get('extension', 'jpg')
    s3_key = f"uploads/{image_id}.{file_ext}"
    
    s3.put_object(
        Bucket=BUCKET,
        Key=s3_key,
        Body=image_data,
        ContentType=f'image/{file_ext}'
    )
    
    return {
        'statusCode': 200,
        'body': json.dumps({
            'image_id': image_id,
            'status': 'uploaded',
            'key': s3_key
        })
    }

# handlers/processor.py - S3 triggered Lambda ประมวลผล image
def process_handler(event, context):
    """ประมวลผล image ที่ upload ไปยัง S3"""
    for record in event['Records']:
        bucket = record['s3']['bucket']['name']
        key = record['s3']['object']['key']
        
        # ดาวน์โหลด image
        response = s3.get_object(Bucket=bucket, Key=key)
        image_data = response['Body'].read()
        
        # ประมวลผล (ในตัวอย่างนี้แค่ดึงขนาดไฟล์)
        file_size = len(image_data)
        image_id = key.split('/')[-1].split('.')[0]
        
        # บันทึกผลลัพธ์ลง DynamoDB
        TABLE.put_item(Item={
            'id': image_id,
            'key': key,
            'file_size': file_size,
            'processed': True,
            'processed_at': datetime.utcnow().isoformat(),
            'status': 'completed'
        })
        
        print(f"Processed image: {image_id} ({file_size} bytes)")

print("Serverless image processing app ready")
```

---

## สรุป

Part 94 ครอบคลุมการ deploy Python applications บน Cloud ทั้ง 3 providers หลัก:

| Cloud Provider | Services | Python Tools |
|---------------|---------|--------------|
| **AWS** | EC2, Lambda, ECS/Fargate, RDS, S3 | boto3 |
| **GCP** | Cloud Run, App Engine, Cloud Functions | google-cloud |
| **Azure** | App Service, Azure Functions, AKS | azure-sdk |

### Key Concepts ที่ได้เรียนรู้:

| หัวข้อ | สาระสำคัญ |
|--------|-----------|
| IaaS vs PaaS vs FaaS | เลือกตามความต้องการ control vs convenience |
| Boto3 | AWS SDK สำหรับ Python ใช้งานทุก AWS service |
| Serverless | Lambda/Cloud Functions เหมาะกับ event-driven apps |
| Containers | ECS/Fargate/Cloud Run สำหรับ containerized apps |
| Config Management | Pydantic Settings + Secrets Manager |
| Infrastructure as Code | Terraform/CDK สำหรับ reproducible infrastructure |
| Health Checks | endpoint สำคัญสำหรับ cloud deployment |
| Auto Scaling | scale อัตโนมัติตาม CPU/request |
| Multi-cloud | abstraction layer สำหรับ provider-agnostic code |

---

*Part 94 - Cloud Deployment - AWS, GCP & Azure | Python Course*
