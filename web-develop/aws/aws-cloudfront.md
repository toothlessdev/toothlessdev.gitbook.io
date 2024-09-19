# AWS Cloudfront

CDN, content delivery network

컨텐츠를 전세계로 빠르게 전송하는것

콘텐츠 전송 네트워크



이미지, html 을 캐싱하여 성능 가속

전세계 수많은 edge location 으로 컨텐츠를 빠르게 전세계로 전송

aws 서비스 <> cloudfront 데이터 전송 무료

ddos 방어 무료 제공 (aws shield standard : L3, L4 디도스 보호)



aws 글로벌 인프라

31개 리전, 99개 가용영역, 400개 이상 엣지 로케이션



용어

distribution 배포

cloudfront 의 단위. 각 배포는 고유 도메인을 가짐

route53 으로 구입한 도메인에 연결가능



origin

원본 파일을 가져오는 위치

기본은 s3. 커스텀으로 ec2, elb, 외부서버에서 가져올 수 이승ㅁ



client <> cloudfront(cdn) <> s3(origin)



s3 버킷 생성

리전 (글로벌) : s3 에서는 리전을 선택할 피룡없음. 버킷에 대한 리전만 설정하면 됨



버킷생성

1. 버킷이름 : dns 형식으로 전세계에서 유일해야함]
2. 객체 소유권 : acl 비활성화됨
3. 퍼블릭 엑세스 차단 : 외부에서 버킷 객체 접근 불가
4. 버킷 만들기



버킷에 파일 업로드

1. 파일 추가
2. 파일 업로드



cloudfront 배포 생성

1. cloudfront 배포 생성
2. 원본 도메인 선택 (origin 선택) : s3 버킷 선택
3. 원본 액세스 제어 설정(권장)&#x20;
   1. 이름 변경
   2. 설정
4. 기본 캐시동작
5. WAF (유료)
6. 기본값 루트 객체 : 배포 url 로 접속시 나올 객체
7. 배포생성 (시간 좀 걸림)
8. s3 버킷으로부터 객체 접근하도록 s3 정책 업데이트
9. 정책 복사, s3 버킷 정책으로 이동
10. 정책 편집기에서 복사한 정책 붙여넣기 ([https://docs.aws.amazon.com/ko\_kr/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html](https://docs.aws.amazon.com/ko\_kr/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html))
11. s3 arn (Resource) 에 붙여넣기.
12. cloudfront 배포 상세정보 arn 붙여넣기
13. 정책 적용



배포 삭제

1. cloudfront 배포 비활성화
2. 배포 삭제





dns : domain name system

도메인네임을 ip 주소로 바꿔주는 시스템

레코드 : 도메인네임 - ip 매핑된것





Route53

cloud dns : aws 서비스와 연동하여 사용하는 dns

클라이언트 요청을 ec2, elb, s3, cloudfront ... 연결가능 + 외부 인프라 연결



1. 호스팅 영역 (hosted zone)&#x20;

레ㅔ코드의 컨테이너로서  두 유형 존재

public : 인터넷에서 트래픽 라우팅 하고자하는 방법을 지정하는 레코드

private : vpc 에서 트래픽을 라우팅하고자하는 방법을 지정하는 레코드

2. record :&#x20;

도메인과 연결된 ip

특정도메인과 서브도메인의 트래픽을 라우팅하는 방식에 대한 정보

호스팅영역과 해당 도메인 이름은 동일



레코드 유형

a : ipv4

aaaa : ipv6

cname: canonical name, 별칭 레코드 (docs.domain.com -> document.domain.com)

ns : 네임서버 주소를 담은 레코드. 자동생성

soa : 권한 시작 레코드. 도메인에 대한 dns 정보 식별



{% embed url="https://docs.aws.amazon.com/ko_kr/Route53/latest/DeveloperGuide/ResourceRecordTypes.html" %}



라우팅 방식

1. 지연 시간 기반라우팅 latency based routing



2. weighted based routing 가중치 기반 라우팅



3.
