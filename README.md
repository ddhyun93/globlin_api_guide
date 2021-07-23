# globlin_api_guide

## 프로젝트 개요 

> [글로블린](https://globlin.io)은 글로벌 마케팅 대행 서비스를 제공하는 [네이콘](https://neicon.net)에서 개발한 "글로벌 인플루언서 마케팅 매니징 툴" 입니다.  
글로블린은 아래의 핵심 기능을 제공하여 인플루언서 마케팅 캠페인을 컨트롤하는 마케터의 업무를 돕습니다.
> 1. 인플루언서 통계 데이터 (조회수, 구독자 등) 관리 시스템 제공 
> 2. 마케팅 캠페인 진행시 인플루언서 콘텐츠에 대한 데일리 리포팅 제공
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



2. AWS ElasticBeanstalk 업로드

* 소스코드에 포함된 Dockerfile을 ElasticBeanstalk이 인식하여 자동으로 배포합니다.


## 데이터베이스 구조

## API 명세서

## AWS 인스턴스 설정 내역
- Docker 플랫폼 기반 ElasticBeanstalk 환경에서 배포할 수 있습니다. (Dockerfile 참조)
- **API 서버**와 **스케쥴러 서버** 모두 단일 인스턴스로 Auto Scailing을 사용하지 않습니다.
- Elastic Beanstalk 환경의 로드밸런서가 아닌 EC2에서의 로드밸런서 설정으로 HTTPS 요청을 처리합니다. 
