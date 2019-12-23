---
layout: post
title: "[GCP] AWS / GCP 환경이 아닌 서버에 google-fluentd 설치하는 방법"
date: 2019-12-23 21:50:36
tags: Dev
categories: Dev
---

## google-fluentd (https://github.com/GoogleCloudPlatform/google-fluentd)

올해 회사에서 진행했던 업무 중, `각 서버별 에러성 로그 수집 환경 구축`이 있었다.

GCP에는 [Stackdriver](https://cloud.google.com/stackdriver/?hl=ko) 라는 멋진 서비스가 있고, 해당 제품에서 지원하는 기능 중 `Stackdriver Logging`이 있다.

대부분 GCP에서 사용했을 때 최고의 효과를 발휘할 수 있지만, AWS 연동도 잘 지원하고 있으며 실제로 우리 회사의 경우는 별 문제 없이 잘 쓰고 있다.

GCP / AWS 환경에서 `Stackdriver Logging`을 사용하려면 여러가지 방법이 있으나, 가장 쉬운 방법은 서버에 `google-fluentd`라는 로깅 에이전트를 설치하여 로그를 수집하는 것이다.

[Fluentd](https://www.fluentd.org/)는 [이 글](https://bcho.tistory.com/1115)을 보면 알 수 있지만 분산 로그 수집기다. 상당히 많은 곳에서 사용하고 있으며, 각종 플랫폼에 대한 플러그인도 다양하게 만들어져있다.  
`google-fluentd`의 경우 Fluentd를 이용하여 서버의 로그를 수집하고 Stackdriver Logging에 전송할 수 있도록 되어있는데, [지원되는 환경](https://cloud.google.com/logging/docs/agent/?hl=ko#environments)을 읽어보면 아래와 같다.

* Compute Engine 인스턴스
* Amazon Web Services Elastic Compute Cloud (AWS EC2) 인스턴스

즉, GCP와 AWS를 지원한다고 되어있다. 당시 우리 회사는 AWS뿐만 아니라, 아직 가비아에 입주한 서버들이 남아있었기에 이에 대한 대응이 필요한 상황이었다.

우선은 AWS에 대한 구축부터 먼저 진행해놓고 대책을 찾던 중, 이 상황을 해결하기 위한 방법을 발견할 수 있었다. 여기서는 해당 게시글의 링크와, 내가 세팅한 방법을 공유하고자 한다.

## AWS / GCP 환경이 아닌 서버에 설치하기
https://github.com/GoogleCloudPlatform/fluent-plugin-google-cloud/issues/156

해당 이슈에서, 나와 같은 고민을 하는 사람들의 의견들을 볼 수 있었다.

해결책에 대해서는 `bryanlarsen` 유저가 남긴 댓글을 인용한다.

![comment](/images/google-fluentd-issue.png)

전체 방법을 설명하면 아래와 같다.

참고로 Workspace 생성법이나 그 외 세팅에 대해서는 여기서 언급하지 않을 예정이다.

1. GCP의 IAM 페이지에 들어가서 Stackdriver Logging 사용을 위한 서비스 계정을 1개 생성한다.
2. 서비스 계정 생성 과정에서 인증키를 json 파일로 다운로드 받을 수 있는데, 이 파일의 이름을 `application_default_credentials.json`으로 변경하고 서버에 업로드한다.
3. 서버에 파일이 위치하는 경로는 `/etc/google/auth/application_default_credentials.json`으로 한다.
4. 세팅 완료 후 명령어를 아래와 같이 입력한다.
    ```bash
    export GOOGLE_APPLICATION_CREDENTIALS="/etc/google/auth/application_default_credentials.json"
    sudo chown root:root "$GOOGLE_APPLICATION_CREDENTIALS"
    sudo chmod 0400 "$GOOGLE_APPLICATION_CREDENTIALS"
    ```
    사실 3번에서 파일이 위치하는 경로 및 파일명은 다르게 지정해도 상관 없다. export에 설정만 잘해준다면...
5. 에이전트를 설치한다.
    ```bash
    curl -sSO https://dl.google.com/cloudagents/install-logging-agent.sh
    sudo bash install-logging-agent.sh
    ```
    설치 과정에서 google-fluentd가 기본 지원하는 config를 세팅하고 싶지 않은 경우 `install-logging-agent.sh`를 열어 51번 line을 편집 후 `sudo bash install-logging-agent.sh`를 실행하길 바란다.
    ```bash
    install_catch_all_config="true" // 설치를 원하지 않는 경우 true를 false로 변경후 저장할 것
    if [[ -n "${DO_NOT_INSTALL_CATCH_ALL_CONFIG}" ]]; then
        install_catch_all_config="false"
    fi
    ```
6. 설치 완료 후에는 google-fluentd가 자동으로 실행되며, 아래와 같이 나온다.
    ```bash
    ps -ef | grep google
    root     28053     1  0 22:01 ?        00:00:00 /opt/google-fluentd/embedded/bin/ruby /usr/sbin/google-fluentd --log /var/log/google-fluentd/google-fluentd.log --daemon /var/run/google-fluentd/google-fluentd.pid
    root     28058 28053  0 22:01 ?        00:00:31 /opt/google-fluentd/embedded/bin/ruby -Eascii-8bit:ascii-8bit /usr/sbin/google-fluentd --log /var/log/google-fluentd/google-fluentd.log --daemon /var/run/google-fluentd/google-fluentd.pid --under-supervisor
    ```
7. `/etc/google-fluentd/google-fluentd.conf` 파일을 수정한다.
    ```
    31번 라인 @type google_cloud 아래에 다음과 같이 입력
    project_id {GCP에 만들어놓은 Workspace 이름 입력}
    vm_id {해당 서버 로그인하면 나오는 호스트명 예시) shua-server}
    zone northamerica-northeast1-b
    ```
    위 예시대로 입력하면 아래와 같다. (workspace 이름은 `shua-gcp` 라고 치자)
    ```
    @type google_cloud
    project_id shua-gcp
    vm_id shua-server
    zone northamerica-northeast1-b
    ```
8. 저장 후 google-fluentd 에이전트 재시작
    ```
    service google-fluentd restart
    ```
9. 위 과정을 성공적으로 진행했으면, Stackdriver Logging 페이지에서 Resource를 `GCE VM Instance`로 선택 후 vm_id에 입력한 값이 목록에 나오는지 확인하면 된다.

## 마무리

해당 내용에 대해 한국어로 된 글이 없었던 것을 기억하고 한 번 적어보았다.

추후에 시간이 된다면 스크린샷도 보충을 하고, Stackdriver Logging 세팅에 대한 전반적인 내용을 적어보고 싶다.