---
title: HTTP Protocol, TCP and Packets
date: "2024-01-28T15:42:42.746Z"
description: "web development secrets 2024 - section2 강의 내용 정리"
---

**브라우저의 주요 기능**

`a program used to display files and navigate web`

web 혹은 world wide web이란

인터넷에 연결된 사용자들이 서로의 정보를 공유할 수 있는 공간

## W3C

- w3c란
    - world wide web consortium, 국제 웹 표준 기구
    - '웹 표준'을 게시하여 브라우저가 수행할 수 있는 작업을 정의
    - 브라우저 기업, 웹 개발자 등 전세계 곳곳에 존재하는 직원 혹은 회원들로 구성된 가상의 조직
- **무엇을 정의하는지? 예시.**
    - **Defining the markup languages that browsers can understand.** For example, the W3C defines the HTML, CSS, and XML markup languages.
    - **Defining the APIs that browsers should expose to web developers.** For example, the File API, DOM API and … you guess it, the Fetch API are a few. There are ton of others.
    - **Defining the various events that browsers can generate.** The W3C defines a variety of events that browsers can generate in response to user actions. For example, the `click` event is generated when a user clicks on a web element.
- 반드시 w3c에서 정의된 표준을 지켜야 하는가?
    - no
    - 실제로 모든 표준을 지키고 있는 브라우저는 없음
        - 복잡성, 성능, 경쟁령 등의 이유로
        - IE는 w3c표준 준수를 중단함
    - 하지만 w3c 표준을 최대한 지키려고 하고 있음, 지켜야 함, 표준이 업데이트됨에 따라 기업은 브라우저를 업데이트해야 함
        - 표준을 지키지 않으면 사용자는 제대로 동작하지 않는 웹페이지를 보게 되며
        - 웹 개발자들은 여러 브라우저에 대해 일관성 있는 개발이 어려움