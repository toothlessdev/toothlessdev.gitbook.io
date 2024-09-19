# 서버리스 프레임워크 (AWS Lambda, DynamoDB)

## 📖 Serverless 란?

서버리스 (Serverless) 는 서버가 없다는 뜻이 아니라, 사용자가 서버를 직접 관리할 필요가 없다는 개념입니다.

서버의 프로비저닝, 패치, 모니터링 등을 클라우드 제공자 (AWS, GCP ...) 가 자동으로 처리하고, 사용자는 애플리케이션 코드를 작성하고 서비스 실행에만 집중 할 수 있습니다.

> ❓프로비저닝&#x20;
>
> 사용자가 요청한 IT 자원을 사용할 수 있는 상태로 준비하는 것을 말합니다

### ✏️ 장점

1. **Auto Scailing**

한글로 번역하면 '자동으로 크기를 조절해주는' 입니다.\
일반적으로 트래픽에 따라 자동으로 서버의 개수를 조절해주는 기능을 의미합니다.

AWS EC2 와 같은 컴퓨팅 서비스를 사용하면, 인스턴스를 위한 AMI 를 설정하고, \
AWS Elastic Load Balancer 와 AWS CloudWatch 를 사용해 다양한 지표에 따라 인스턴스 크기를 조절 하도록 Auto Scaling Group 을 설정 하는 등 복잡한 과정이 필요합니다.

> 반면, 서버리스는 트래픽이나 요청이 증가할때 자동으로 확장되고, 트래픽이 줄어들면 자동으로 축소됩니다. 이를 통해 서버 자원을 효율적으로 사용할 수 있습니다.

2. **사용량 기반 비용 책정**

서버리스는 요청이 들어올 때만 함수 인스턴스가 생성되고, 응답을 반환하면 함수 인스턴스는 종료되고, 자원이 해제됩니다.

> 따라서 AWS Lambda 와 같은 서버리스 함수는,\
> 요청이 들어와서 함수가 실행되는 동안에만 비용이 발생하며, 요청이 없는 시간에는 비용이 발생하지 않습니다.



### ✏️ 단점

1. **Cold Start**

서버리스 환경에서는 EC2 와 같은 컴퓨팅 서비스처럼 항상 인스턴스가 켜져 있는 것이 아니라고 했습니다.\
따라서 함수를 처음 호출하거나, 오랫동안 호출되지 않았을때 서버가 준비되는 시간이 필요합니다.

{% embed url="https://maxday.github.io/lambda-perf/" %}

2. **실행 시간의 제한**

AWS Lambda 는 메모리 최대 10GB, 처리시간 최대 15분 으로 제한되어 있습니다.\
따라서 많은 메모리나 실행시간을 필요로 하는 작업을 처리하는 데에는 적합하지 않습니다





## 📖 Serverless Framework

Serverless Framework 는 클라우드 서비스와 연동하여&#x20;

create, deploy, update, manage 가 쉬움, lambda functions, services 에 대해

cicd 가 쉬움

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>



## 📖 개발 환경설정

* NodeJS
* AWS CLI
* IAM User
* npm i -g serverless

1. IAM user 설정

* administrator access 권한 추가
* access, secret

2. aws cli

aws configure --profile NAME

access, secret , region 설정

export AWS\_PROFILE=NAME

env | grep AWS 로 설정 확이ㅏㄴ

aws s3 ls

3. npm i -g serverless



sls create





local 에서 테스트

sls invoke local --function FUNC\_NAME

FUNC\_NAME 은 serverless.yml handler.FUNC\_NAME

sls invoke local --function FUNC\_NAME --data { JSON }



