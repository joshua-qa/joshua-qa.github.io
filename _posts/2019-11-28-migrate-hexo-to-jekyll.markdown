---
layout: post
title: "Hexo에서 Jekyll로 블로그 이전하는 방법"
date: 2019-11-28 22:04:48
tags: etc
categories: etc
---

Jekyll에서 Hexo로 넘어가는 경우, [공식문서](https://hexo.io/ko/docs/migration)에 마이그레이션 가이드가 있기 때문에 그 것을 참고하면 된다.

그런데, 반대의 경우에는 가이드를 찾기 어려워서 (검색하면 물론, 한 두개는 나온다) 한 번 적어보려고 한다.

## 무엇이 문제인가?

기존에 썼던 글을 유지한 상태로 블로그를 갈아타려는 경우 가장 문제가 되는 것은 포맷이 다르다는 점이다.

- 파일명
- 게시글 헤더 (layout, title, date, tags, categories, ...)

참고로 Jekyll과 Hexo는 게시글 헤더의 기본 틀이 상당히 비슷하며, 복잡한 세팅을 하지 않았다면 비교적 쉽게 이전할 수 있다. 다만 Jekyll을 사용하려는 경우 파일명을 규칙에 맞게 지정해줄 필요가 있으므로, Hexo에서 작성했던 게시글들의 파일명을 모두 바꿔줘야 한다.

그리고, 각 게시글을 열어서 헤더에 `layout: post`를 추가해줘야 하는 것도 잊지 말자.

## 쉽게 바꾸는 방법

[이 코드](https://gist.github.com/yiyizym/ce6021b89326079920dbef0c81c7bd18)를 참고하여 마이그레이션용 스크립트를 구성했다.

```ruby
# encoding: UTF-8
#!/usr/bin/env ruby
Dir.chdir('_posts') do 
  Dir.glob('*.md') do |filename|

    File.open(filename, mode: 'r+:UTF-8') do |file|
      content = file.read(file.size)
      date = content.match(/date: (\d+-\d+-\d+)/)
      if date.nil?
        puts filename
      else
        purename = filename[0..-4]
        puts purename
        File.rename(filename, "#{date[1]}-#{purename}.markdown")
      end
    end
  end
end
```

원 코드의 하단에는 파일 첫 부분에 `---\nlayout:post\n`를 넣어주는 코드도 있지만 내가 실행해 봤을 때는 잘 동작하지 않아서, gsed를 이용한 방법을 소개하려고 한다.

우선 윗 코드를 `_posts` 폴더의 상위 폴더에 저장한 뒤 실행하면, Hexo로 작성한 게시글 파일명이 아래와 같이 변경된다.

`programmers-lv4-scheduling.md -> 2018-08-25-programmers-lv4-scheduling.markdown`

코드를 간단하게 설명해보면 `게시글 헤더의 date값을 읽어와서 파일명 앞에 붙여주는 코드`이다.  
다른 방법도 충분히 많겠지만, 이렇게 하는 방법도 있다는 것을 알아두면 좋을 것 같다.

`layout:post`를 끼워넣는 작업은 아래와 같이 진행했다. (MacOS 기준)

1) GNU Sed를 설치 (gsed)
2) 게시글 파일이 들어있는 폴더 (_posts)에서 아래와 같은 명령어 실행
```shell
gsed -i '0,/---/s//---\nlayout: post/' 
```

Hexo로 작성한 게시글의 헤더는 보통 아래와 같은 규격을 가진다.

```
---
title: asdf
date: 2019-12-31 23:59:59
categories: asdf
tags: asdf
---

(이후 글 내용)
```

위 명령어를 사용하면 `---가 처음 등장하는 파트`만 `---\nlayout: post`로 치환해준다.  
sed라고 해서 모두 이런 명령어를 사용할 수는 없고, GNU sed에서만 동작한다고 하니 참고하자.
(https://stackoverflow.com/questions/148451/how-to-use-sed-to-replace-only-the-first-occurrence-in-a-file)

이 작업을 하고나면 아래와 같이 바뀌어있다.

```
---
layout: post
title: asdf
date: 2019-12-31 23:59:59
categories: asdf
tags: asdf
---

(이후 글 내용)
```

여기까지 진행하면, Hexo에서 작성했던 게시글을 Jekyll로 마이그레이션 하는 작업이 거의 끝나게 된다.

그 이후에는 Jekyll 세팅을 완료하고 블로그를 배포하면 되나, 아직 한 단계 과정이 남아있다.

## 기존 주소 형식을 유지하고 싶은 경우는?

Jekyll에서 별다른 permalink 설정을 하지 않는 경우, 아래와 같은 세팅이 디폴트다.

`/:categories/:year/:month/:day/:title:output_ext`

출처 : [공식문서](https://jekyllrb-ko.github.io/docs/permalinks/)

즉, 내 블로그를 기준으로 한다면 `http://blog.devjoshua.me/daily/2017/12/28/171228-2017년회고.markdown`이 될 것이다.

이렇게 되는 경우, Hexo로 블로그를 운영하던 시절의 게시글 링크들을 사용할 수 없다.

이게 맘에 들지 않는 경우 아래와 같이 `_config.yml`에서 permalink를 지정해주면 된다.

`permalink: /:year/:month/:day/:title/`
