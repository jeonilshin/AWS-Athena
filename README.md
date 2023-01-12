# <h1 align="center"><img src="https://user-images.githubusercontent.com/86287920/209415861-25c0652b-e801-4cce-82ae-ddbc0c7d36c3.png" width="200"></h1>

### 중요! : SQL 공부 필수 입니다.

Amazon Athena는 표준 SQL을 사용하여 Amazon S3(Amazon Simple Storage Service)에 있는 데이터를 직접 간편하게 분석할 수 있는 대화형 쿼리 서비스입니다. AWS Management Console에서 몇 가지 작업을 수행하면 Athena에서 Amazon S3에 저장된 데이터를 지정하고 표준 SQL을 사용하여 임시 쿼리를 실행하여 몇 초 안에 결과를 얻을 수 있습니다.

Athena는 서버리스 서비스이므로 설정하거나 관리할 인프라가 없으며 실행한 쿼리에 대해서만 비용을 지불하면 됩니다. Athena는 자동으로 확장되어 쿼리를 병렬로 실행하여 대규모 데이터 집합과 복잡한 쿼리에서도 빠르게 결과를 얻을 수 있습니다.

### Amazon Athena를 사용하여 Application Load Balancer 액세스 로그를 분석하려면 어떻게 해야 하나요?


기본적으로 Elastic Load Balancing은 액세스 로깅을 활성화하지 않습니다. [액세스 로깅을 활성화]할 때 Amazon Simple Storage Service(Amazon S3) 버킷을 지정합니다. 모든 Application Load Balancer 및 Classic Load Balancer 액세스 로그는 해당 Amazon S3 버킷에 저장됩니다. 로드 밸런서의 문제를 해결하거나 성능을 분석하려는 경우 Athena를 사용하여 S3의 액세스 로그를 분석하면 됩니다.

**참고:** Athena를 사용하여 Application Load Balancer와 Classic Load Balancer에 대한 액세스 로그를 모두 분석할 수 있지만 이 문서에서는 [Application Load Balancers]만 다룹니다.

### 해결 방법
**Application Load Balancer** 로그에 대한 데이터베이스 및 테이블 생성

Athena에서 액세스 로그를 분석하려면 다음을 수행하여 데이터베이스와 테이블을 생성합니다.

1. [Athena] 콘솔을 엽니다.

2. 이전 단계에서 생성한 데이터베이스에서 Application Load Balancer에 대한 **alb_log** 테이블을 만듭니다. 자세한 내용은 [Application Load Balancer 로그에 대한 테이블 생성]을 참조하세요.
**참고:** 쿼리 성능 향상을 위해 파티션 프로젝션을 사용하여 테이블을 생성하도록 선택할 수 있습니다. 파티션 프로젝션에서 파티션 값과 위치는 AWS Glue 데이터 카탈로그와 같은 리포지토리에서 읽지 않고 구성에서 계산됩니다. 자세한 내용은 [Amazon Athena와 파티션 프로젝션]을 참조하세요.
```
CREATE EXTERNAL TABLE IF NOT EXISTS alb_logs (
            type string,
            time string,
            elb string,
            client_ip string,
            client_port int,
            target_ip string,
            target_port int,
            request_processing_time double,
            target_processing_time double,
            response_processing_time double,
            elb_status_code int,
            target_status_code string,
            received_bytes bigint,
            sent_bytes bigint,
            request_verb string,
            request_url string,
            request_proto string,
            user_agent string,
            ssl_cipher string,
            ssl_protocol string,
            target_group_arn string,
            trace_id string,
            domain_name string,
            chosen_cert_arn string,
            matched_rule_priority string,
            request_creation_time string,
            actions_executed string,
            redirect_url string,
            lambda_error_reason string,
            target_port_list string,
            target_status_code_list string,
            classification string,
            classification_reason string
            )
            ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.RegexSerDe'
            WITH SERDEPROPERTIES (
            'serialization.format' = '1',
            'input.regex' = 
        '([^ ]*) ([^ ]*) ([^ ]*) ([^ ]*):([0-9]*) ([^ ]*)[:-]([0-9]*) ([-.0-9]*) ([-.0-9]*) ([-.0-9]*) (|[-0-9]*) (-|[-0-9]*) ([-0-9]*) ([-0-9]*) \"([^ ]*) (.*) (- |[^ ]*)\" \"([^\"]*)\" ([A-Z0-9-_]+) ([A-Za-z0-9.-]*) ([^ ]*) \"([^\"]*)\" \"([^\"]*)\" \"([^\"]*)\" ([-.0-9]*) ([^ ]*) \"([^\"]*)\" \"([^\"]*)\" \"([^ ]*)\" \"([^\s]+?)\" \"([^\s]+)\" \"([^ ]*)\" \"([^ ]*)\"')
            LOCATION 's3://alb-logs-2113/logs/';
```
사용 사례에 따라 테이블 이름과 S3 위치를 바꿔야 합니다.

3. 탐색 창의 **[테이블(Tables)]** 에서 테이블 이름 옆에 있는 메뉴 버튼에서 **[테이블 미리 보기(Preview table)]** 를 선택합니다. Application Load Balancer 액세스 로그의 데이터는 **[결과(Results)]** 창에 표시됩니다.

4. **쿼리 편집기**를 사용하여 테이블에서 SQL 문을 실행합니다. 쿼리를 저장하고, 이전 쿼리를 보거나 쿼리 결과를 CSV 형식으로 다운로드할 수 있습니다.

## 쿼리의 예
다음 예에서 쿼리에 맞게 테이블 이름, 열 값 및 기타 변수를 수정하세요.

**사진 입니다** 
명령어 사용하고 싶다면 AWS Athena 사이트에서 참고해주세요: https://aws.amazon.com/ko/premiumsupport/knowledge-center/athena-analyze-access-logs/

![image](https://user-images.githubusercontent.com/86287920/209416848-bbcc34a0-a3db-44b5-8826-59cc6b173f91.png)
![image](https://user-images.githubusercontent.com/86287920/209416867-218b6804-7186-45eb-a2e6-180ce57f72e4.png)
![image](https://user-images.githubusercontent.com/86287920/209416929-b4b88c6c-4423-4f6b-a13a-f56e4f046f16.png)

## 직접 사용했던 명령어 예시:
```
SELECT COUNT(request_verb) AS
 count,
 client_ip
FROM alb_log
WHERE elb_status_code = 302
GROUP BY client_ip
```
```
SELECT COUNT(request_verb) AS count, request_verb, request_url FROM alb_log
WHERE request_verb = 'GET'
GROUP BY request_verb, request_url
```
```
SELECT COUNT(request_verb) AS count, request_verb, request_url, client_ip FROM alb_log
WHERE request_verb = 'POST'
GROUP BY request_verb, request_url, client_ip
```

---
[액세스 로깅을 활성화]: https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-access-logs.html#enable-access-logging
[Application Load Balancers]: https://docs.aws.amazon.com/elasticloadbalancing/latest/application/application-load-balancers.html
[Athena 콘솔]: https://console.aws.amazon.com/athena/
[데이터베이스를 생성]: https://docs.aws.amazon.com/athena/latest/ug/create-database.html
[Application Load Balancer 로그에 대한 테이블 생성]: https://docs.aws.amazon.com/athena/latest/ug/application-load-balancer-logs.html#create-alb-table
[Amazon Athena와 파티션 프로젝션]: https://docs.aws.amazon.com/athena/latest/ug/partition-projection.html






