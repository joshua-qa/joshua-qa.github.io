---
layout: post
title: "[Outlook] 이 개체는 Outlook 프로그램을 사용하여 만들었습니다"
date: 2017-06-16 12:06:41
categories:
- etc
tags:
- office
---

## Outlook 오류 메시지 : 이 개체는 Outlook 프로그램을 사용하여 만들었습니다.

아버지의 요청으로 원격 지원을 할 일이 생겼는데, 매일같이 사용하시던 Outlook에서 갑자기 오류가 나면서 첨부파일 (엑셀 파일)이 열리지 않는다고 하셨다.

들어가서 오류 메시지를 확인해보니 다음과 같은 오류와 함께 첨부된 엑셀 파일이 열리지 않았다.

`이 개체는 Outlook 프로그램을 사용하여 만들었습니다.`

비슷한 류의 질문이 한국 MS 페이지에 올라와 있었지만 답변 내용이 크게 도움을 주진 못했고.. 오류 메시지를 영문으로 번역해서 구글링을 해봤다.

미국 MS 페이지에서 어제 날짜로 올라온 질문 및 답변을 확인하여 이슈를 해결할 수 있었고, 일단 이 포스팅이 구글 검색 결과에 노출될지 모르겠지만...

분명 2017년 6월 14일~15일쯤에 윈도우 업데이트를 하신분이 있다면 비슷한 오류를 겪으실거라고 생각한다. 그래서 간단하게 적어봅니다.

[https://answers.microsoft.com/en-us/msoffice/forum/msoffice_outlook-mso_win10/outlook-2007-error-the-program-used-to-create-this/a1c82585-eec0-43bb-bccc-6a92f49d716f](https://answers.microsoft.com/en-us/msoffice/forum/msoffice_outlook-mso_win10/outlook-2007-error-the-program-used-to-create-this/a1c82585-eec0-43bb-bccc-6a92f49d716f)



이 링크를 들어가보면, 다음과 같은 업데이트를 삭제해보라고 안내해주고 있다.

`KB3191898, KB3203467`

3191898은 오피스 2007 유저를 위한 보안 업데이트, 3203467은 오피스 2010 유저를 위한 보안 업데이트인 것 같다.

`프로그램 제거 또는 변경`에 들어가셔서 `설치된 업데이트 보기` 눌러보시면 설치한 업데이트 목록이 나오는데, 여기서 위에 적어놓은 업데이트 번호를 찾아서 삭제하시면 정상적으로 기능이 동작함을 확인하실 수 있습니다.
