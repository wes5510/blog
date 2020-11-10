# KOREA DevOps meetup '19 후기 및 요약

**아래의 글은 세션 발표를 그대로 쓴게 아니라 주관적인 생각이 들어간 요약이니 주의하시고 읽어보시기 바랍니다.**

## 키노트

- [Cloud Native Computing Foundation](https://www.cncf.io/)에서 DevOps의 주요 플로우를 볼 수 있음.
- 아래는 CNCF Annual Report 2018의 [ECOSYSTEM TOOLS](https://www.cncf.io/cncf-annual-report-2018/#ecosystem-tools)의 한 부분이다

![https://www.cncf.io/wp-content/uploads/2019/02/landscape.png](https://www.cncf.io/wp-content/uploads/2019/02/landscape.png)

![https://www.cncf.io/wp-content/uploads/2019/02/CNCF_TrailMap_latest.png](https://www.cncf.io/wp-content/uploads/2019/02/CNCF_TrailMap_latest.png)

## 메가존&글로벌 클라우드 교육 프로그램 소개

- 메가존에서 실시하는 알리바바 클라우드 교육 프로그램 소개
    - [LabEx](https://labex.io/alibaba)에서 클라우드 교육 제공
    - 메가존 교육 세미나

## DevOps의 인문학적 접근

- 오픈소스 소프트웨어
    - 소프트웨어의 엔트로피는 점점 높아질 것이기 때문에 오픈 소스 문화(시장:Bazaar)가 중요하다.
    - 중앙 집중 vs 비 중앙 집중
- DevOps
    - 협업의 효율
    - 반복의 기민함
    - 비 중앙 집중에 걸맞는 방법론 같다.
    - 넷플릭스
        - 모든 개발자가 OP에 접근가능
        - 좋은 개발자를 뽑아서 방목
    - Tools
        - Nexus, ...
        - GitLab은 DevOps에 모든 사이클에 대한 도구를 제공한다.
- 좋은 기업은 개발자가 아무런 절차, 룰에 상관없이 창의력을 펼칠 수 있는 곳(Zero Configuration, ...)
- 노자가 서술한 책 읽어보자

## 자동화, 어디부터 어디까지 가능한가?

- 할 수 있는 모든 걸 자동화하자
- 자동화 타이밍
    - 같은 작업을 5번 이상 반복해야 할 때, 비생산적일, ...
- 자동화 이유
    - 시간 단축, 프로세스 개선, 생산성 증가, 스킬 향상
- 자동화 언어와 도구들
    - Bash, Node.js, RegEx, RPA, IaaC(Terraform, Ansible), Kubernetes

## 최소한의 비용과 시간을 갈아넣어 만든 Kubernetes + CI/CD...

- Devops 도구(jenkins, ...)는 AWS lightsail
- kops
    - kubctl과 비슷
    - Kubernetes 관리
- CI/CD 파이프라인
    - Bitbucket(코드 관) → Jenkins, Sonarqube(빌드) → portur(컨테이너 빌드) → Kubernetes(배포)
- 다차원 분석(OLAP)
    - ClickHouse, Druid, Pinot
- 모니터링
    - Prometheus + Grafana
        - 장점: 사용 쉬움, 왠만한 대시보드 다 있음
    - 현재는 Kibana으로 넘어가려함
- 꿀팁
    - ACM/lets encrypt 로 꽁짜로 ssh 인증서 사용
    - Lightsail: 온디맨드에서는 가장 싸다, vpc peering도 됨
    - direnv 디렉토리만 바꾸면 환경설정을 바꿔준다

## 후기

비중앙집중이라는 말이 핵심으로 느껴졌다. 비중앙집중은 Git, DevOps, ...등에 많은 영향을 끼친거 같다. Git은 분산 버전 관리 시스템, DevOps는 개발과 운영을 통합하므로서 비중앙집중을 실현한다.

하지만 모든 책임을 모든이가 나누는 건 한계가 있을 수도 있다. 모든 사람이 모든 걸 알아야하는 건 비효율적이기 때문이다. 어느 정도 틀안에서 실행해야지 효과가 있을 것이라 생각한다.

결론으로 비중앙집중은 이와 같은 효과를 가져다 줄 거 같다.

> 비중앙집중 → 업무 공유(문서화, 리팩토링, 프로세스 개선, 오픈소스화, ...) → 책임, 권한의 분산 → 모든 참여자의 주인의식 증진 → 성공적인 프로젝트

그리고 자동화 세션을 들었을 때 이때까지 무식하게 일을 해왔던 나에게 반성하는 계기가 됐다. 별거 아닌 불편함에 적응되버린 것들을 하나씩 해결해야겠다.

아 경품으로 주신 데브옵스 책 감사합니다.🙇

## 세션 자료

- 운영팀 없는 스타트업에서 개발자가 알아야 하는 서비스 운영 지식/이동인

    [스타트업 개발자가 알아야할 서비스 운영지식](https://www.slideshare.net/WhaTapIO/ss-149039496?fbclid=IwAR2NMGKJUgXQ8SYtumq0MdDdXLkmVw8ZHOS_u-UDK14Q-zBvCGWA53PbPLQ)

- Opencensus with Prometheus and Kubernetes/김진웅

    [OpenCensus with Prometheus and Kubernetes](https://www.slideshare.net/JinwoongKim8/opencensus-with-prometheus-and-kubernetes?fbclid=IwAR26CA3OqmhVl7_fKIY8LwuALqcDY-tU5K-bL5etPoZjQ3u7fKkM8Jxaeio)

- 그들이 AWS 위에서 Data Pipeline 을 관리하는 방법/박훈

    [그들이 AWS 위에서 데이터 파이프라인을 운영하는법 (2019)](https://docs.google.com/presentation/d/11C_BKio0DZIop_ZjJk7ogxQtWV5qHIr-hHjw277z64k/edit?fbclid=IwAR1UAINLBpa8TZSbtx-ROWtHYEg4s_e6UrA5JCBfSPdUufVVSh1JlnAtJJM#slide=id.g5adadfee39_0_43)

- 최소한의 비용과 시간을 갈아넣어 만든 Kubernetes/독고혁

    [시간과비용을갈아넣은 kubernetes](https://www.slideshare.net/secret/rAVb26qXn71icl?fbclid=IwAR1zpFG9VbyaYfoTTd37iJ9NO6DdVjH3ls3KiDkc1FUGGOeFCCl38Znx6c8)
