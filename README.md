# globlin_api_guide

## 프로젝트 개요 

> [글로블린](https://globlin.io)은 글로벌 마케팅 대행 서비스를 제공하는 [네이콘](https://neicon.net)에서 개발한 "글로벌 인플루언서 마케팅 매니징 툴" 입니다.  
> 자세한 툴 사용에 대한 가이드는 [링크](https://github.com/ddhyun93/globlin_guide) 를 참조해주세요.

> 글로블린은 아래의 핵심 기능을 제공하여 인플루언서 마케팅 캠페인을 컨트롤하는 마케터의 업무를 돕습니다.
> 1. 인플루언서 통계 데이터 (조회수, 구독자 등) 관리 시스템 제공 
>   - 내부 스케쥴러를 통한 인플루언서 메타데이터의 주기적 업데이트 기능 제공
>   - 내부 로직 및 유트브 API 통신을 통한 인플루언서의 마케팅 캠페인 예상 수치 제공
>   - 엑셀 파일 입출력 제공
> 2. 마케팅 캠페인 진행시 인플루언서 콘텐츠에 대한 데일리 리포팅 제공
>   - 내부 스케쥴러를 통한 콘텐츠의 일간 데이터 기록
>   - 콘텐츠 Raw Data의 엑셀 출력 제공 
> 3. 키워드 기반 특정 권역의 인플루언서 대량 크롤링 기능 제공

>[글로블린](https://globlin.io)의 API 서버는 아래의 태크 스택을 기반으로 만들어졌습니다.
> 
> - 개발 언어: *python 3.8*
> - 관계형 데이터베이스: *MySQL*
> - 어플리케이션 관리 : *Docker*
> - 클라우드 환경: *AWS*
>   - *EC2*
>   - *Elastic Beanstalk*
>   - *RDS*
>   - *Route53*
>   - *S3*
>   - *boto3*
>- 기타:
>   - WebFramework: [FastAPI](https://fastapi.tiangolo.com/) 
>   - ORM: [SQLAlchemy](https://www.sqlalchemy.org/)
>   - DB Migration: [Alembic](https://alembic.sqlalchemy.org/en/latest/)
>   - Background Scheduler: [APScheduler](https://apscheduler.readthedocs.io/en/stable/)
>   - Login: [JWT](https://jwt.io/)
>   - Excel IO: [openpyxl](https://openpyxl.readthedocs.io/en/stable/)

> [글로블린](https://globlin.io)은 [**REST API 서버**]와 스케쥴링 잡을 관리하는 [**스케쥴러 서버**] 2개의 인스턴스로 운영됩니다.  


## 배포 가이드


1. 셸 스크립트로 소스코드 압축  
    `sh zip.sh`
    
    * 소스코드 파일이 추가된 경우 zip.sh 파일을 수정하세요.
        ```shell
       #!/bin/sh
        zip -r "globlin_api-$(date +"%Y-%m-%d").zip" .env ./__init__.py ./controller ./database.py ./Dockerfile ./Dockerrun.aws.json ./routers ./main.py ./requirements.txt ./models ./schema ./[template]influencer_list.xlsx ./[template]crawled_influencer.xlsx
        exit 0
        >>> .env 이하로 추가된 소스코드를 추가하면 됩니다.
        ```

2. 환경에 따른 상수 변경 
    - 배포 환경에 따라, `database.py`의 데이터 베이스 경로, `controller/schedule_controller.py`의 스케쥴러 서버 경로를 수정한 후 배포해야 합니다. 
    

3. AWS ElasticBeanstalk 업로드

* 소스코드에 포함된 Dockerfile을 ElasticBeanstalk이 배포합니다.


## 데이터베이스 구조
![erd](https://user-images.githubusercontent.com/58629967/127252906-e5e1b78e-cf91-4fca-a993-ed0340ed9c45.png)
- (정성적으로) 데이터 베이스는 크게 3 부분으로 나눌 수 있습니다.
    1. 유튜브 채널들과, 그 채널들이 진행하는 광고  캠페인으로 나누어진 **인플루언서** 관련 부분 
       
        |테이블명|설명|
        |--------|---|
        |***influencers***|인플루언서의 기본 정보를 기록합니다.|
        |***campaigns***|진행되는 광고 캠페인의 기본 정보를 기록합니다.|
        |***campaign_influencer***|캠페인과 인플루언서의 N:M 구조를 위한 관계 테이블입니다. campaign_id와 influencer_id 정보를 기록합니다.|
        |***videos***|인플루언서가 광고 캠페인을 위해 등록한 비디오에 대한 정보를 기록합니다.|
        |***crawled_data***|매일 오전 7시 광고 캠페인을 위해 등록된 영상을 서버가 백그라운드에서 체크하여, 해당일의 영상 정보를 기록합니다.|   
    2. 플랫폼을 구성하는 각종 데이터를 담은 **메타데이터** 관련 부분으로, 다른 테이블의 컬럼에서 **Foreign Key**로 사용됩니다.
       
        |테이블명|설명|사용처|
        |--------|-----|------|
        |***agencies***|인플루언서들의 소속사 정보를 기록합니다.|***influencer***테이블의 agency 컬럼|
        |***tags***|인플루언서를 대표하는 태그 정보를 기록합니다.|***influencer***테이블의 tag1~tag5 컬럼|
        |***categories***|캠페인과 인플루언서가의 카테고리 정보를 기록합니다.|***influencer/campaigns***테이블의 category 컬럼|
        |***languages***|광고 캠페인의 언어와, 인플루언서의 사용언어를 기록합니다. 어떤 언어인지를 표현하는 **name**컬럼과 해당 언어의 ISO639-1 코드를 기록하는 **code**컬럼이 존재합니다.|***influencer/campaigns***테이블의 language 컬럼|
        |***regions***|광고 캠페인의 지역과, 인플루언서의 유튜브상 프로필에 붙는 등록 권역들을 기록합니다. 어떤 권역인지를 표현하는 **name**컬럼과 해당 권역의 ISO 3166-1 alpha-2 코드를 기록하는 **code**컬럼이 존재합니다.|***influencer/campaigns***테이블의 region 컬럼|
        |***platforms***|인플루언서의 활동 플랫폼을 기록합니다. 유튜브만 지원하는 현재 버전에서는 유의미하게 사용되지 않습니다.|***influencer***테이블의 platform 컬럼|
        
    3. 유튜브 API와 통신하여 특정 키워드와 매칭되는 인플루언서를 크롤링해는 **크롤링** 관련 부분 
    
        |테이블명|설명|
        |-------|----|
        |***influencer_crawl_jobs***|apscheduler의 백그라운드 작업 객체의 메타데이터를 저장합니다.|
        |***crawled_influencer***|크롤링된 인플루언서 리스트를 저장합니다.|
    
    4. 기타
    
        |테이블명|설명|
        |-------|----|
        |***user***|회원 데이터를 저장합니다.

## API 명세서
- API는 크게 4개의 도메인 단위로 나뉩니다.
- 자세한 API 명세는 [openapi_swagger](http://dev-api.globlin.io/docs) 를 통해 확인할 수 있으며, 본 README에서는 대략적인 소개만 다룹니다.

    1. 로그인/인증 관련 : /user
    2. 캠페인 관련 : /campaign
    3. 인플루언서 관련 : /influencer
    4. 플랫폼 내 사용되는 필드 관련 : /field
    5. 기타 등등 : heathchecker 및 인플루언서 데이터 갱신 시간 체크 등

## 주요 기능 설명    

- ***로그인 로직***
    - JWT 형식의 access_token과 refresh_token을 발급합니다.
    - 클라이언트 사이드에서 브라우저 스토리지에 토큰값을 저장합니다. 
    - 서버에서는 클라이언트가 요청헤더에 토큰을 실어 보내면 이를 검증합니다. 
        - `/controller/user_controller.py` 및 `/routers/user.py` 참조
    - access_token은 15분의 유효기간이 지나면 만료됩니다.
    - access_token이 만료되면 /user/refresh 에 refresh_token을 헤더에 실어 요청을 보내 새로운 access_token을 발급받을 수 있습니다.
    - refresh_token은 30일 후 만료됩니다.


- ***캠페인 로직***
    - 캠페인을 생성하면, DB에 등록된 인플루언서와 해당 캠페인을 N:M 관계로 맵핑할 수 있게 됩니다. 
        - `[POST] /campaign/{campaign_id}/influencer`
    - 캠페인에 인플루언서의 영상이 등록되면, 서버에서 자동으로 익일부터의 데이터를 수집합니다.
    - DB서버 사양을 고려하여 매일매일의 영상 데이터 수집은 매일 오전 7시 ~ 7시 10분 사이(KST기준)에 랜덤하게 진행됩니다.  
        - `스케쥴러 서버의 소스코드 참조`
    - 리포팅은 설정해둔 캠페인 진행 기간동안 진행됩니다.
    - 오전 7시경 진행중인 캠페인에 대한 영상 데이터 수집이 끝나면 오전 8시 엑셀파일이 렌더링 되며, boto3를 통해 연결된 s3 버킷에 이 엑셀파일이 업로드 됩니다.
    - 클라이언트 사이드에선 정해진 패턴의 엑셀 파일명을 포함한 s3 버킷 경로로 엑셀 파일 다운로드 요청을 보냅니다.
    - 캠페인에는 한 명의 인플루언서가 여러 영상을 올리는 경우를 상정하여, 이 경우 캠페인에 영상을 등록한 순서대로 해당 영상에 order_number를 부여합니다. 


- ***인플루언서 로직***
    - 인플루언서를 등록하면 인플루언서의 메타데이터 (평균 조회수, 구독자 등)이 자동으로 계산되어 등록됩니다.
    - 해당 메타데이터는 유튜브 API Key의 잔여 사용량에 따라, 하루 약 50명씩 새로 계산되어, 최신 데이터를 유지하도록 하였습니다. 
    - 기존 인플루언서 마케팅 업체에서 사용하던 인플루언서 제안서 양식에 맞게 엑셀 템플릿을 구축해놔, 광고주에게 제안하고자 하는 인플루언서 리스트를 excel 파일로 다운받을 수 있게 하였습니다.
    - 인플루언서를 등록하기 위해서 직접 인플루언서의 데이터를 입력하는 방식 외에도 "파일을 통한 등록" 방식도 지원하고 있습니다.
    - 인플루언서 파일 등록의 경우 아래의 로직으로 처리됩니다.
        1. 클라이언트가 업로드한 엑셀 파일을 openpyxl 의 sheet 객체로 변환
        2. sheet를 해석하여 인플루언서의 channel_id 를 추출
        4. 표현식 활용하여 유효한 channel_id 양식인지 확인  
        5. 인플루언서 id와 등록된 인플루언서 DB를 비교하여 이미 등록된 인플루언서인지 확인
        6. 문제 없을 경우 query_list에 append, 문제 있는 경우 해당 셀 번호를 error_list에 append
        7. sheet 내 다른 데이터 (입력한 태그값 등)가 유효한지 확인 
        8. 유효한 데이터이면 pass, 무효한 데이터이면 해당 셀 번호를 error_list에 append
        9. error_list의 length가 0이면 무결한 엑셀파일로 간주하고, query_list에 있는 인플루언서 channel_id로 인플루언서 등록 시작
        10. error_list의 length가 1보다 크면 error_list에 저장된 "문제있는 셀 번호"를 리턴함  

- ***유튜버 크롤링 로직***
    - Youtube Search API 를 통해 키워드와 권역을 기반으로 인플루언서를 검색합니다.
    - 소스코드 대부분은 [개인프로젝트](https://github.com/ddhyun93/youtube_channel_search) 에서 참고한 부분이 많으니 자세한 내용은 소스코드를 참조해주세요.

## AWS 인스턴스 설정 내역
- Docker 플랫폼 기반 ElasticBeanstalk 환경에서 배포할 수 있습니다. (Dockerfile 참조)
- **API 서버**와 **스케쥴러 서버** 모두 단일 인스턴스로 Auto Scailing을 사용하지 않습니다.
- Elastic Beanstalk 환경의 로드밸런서가 아닌 EC2에서의 로드밸런서 설정으로 HTTPS 요청을 처리합니다. 
