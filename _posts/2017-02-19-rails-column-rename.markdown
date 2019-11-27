---
layout: post
title: "[RoR] Rails에서 DB컬럼명을 잘못 입력해서 변경하고 싶을 때"
date: 2017-02-19 01:25:02
tags:
- rails
- ruby
categories:
- Dev
---

## 발생한 상황

* 레일즈에서 Model을 생성할 때 컬럼명에 model_name을 포함시켰는데, `model_name is defined by Active Record. Check to make sure that you don't have an attribute or method with the same name.` 오류가 발생해서 변경이 필요했다.
* 그래서 컬럼명을 다른 걸로 변경할 필요가 있었다.



## 해결 방법

* 생각 이상으로 간단하게 해결할 수 있었는데, migration을 생성해주고 거기에 변경 필요한 내용을 작성해서 db:migrate 해주면 되었다.

* rails generate migration '이름' 형식으로 콘솔에 써주면 migration 파일을 만들어준다. 나는 FixColumnName으로 써서 만들었다.

* YYYYMMDDhhmmss_fix_column_name.rb 이런식으로 파일이 생성되는데 편집기로 열어서 다음과 같이 써주면 된다.

  	class FixColumnName < ActiveRecord::Migration[5.0]
  		def change
  			rename_column :테이블이름, :기존컬럼명, :바꿀컬럼명
  	 	end
  	end
  편집한 뒤에 콘솔에서 `rails db:migrate` 해주면 컬럼명이 변경된다.
