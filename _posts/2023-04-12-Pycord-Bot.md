---
title: "[Record] Pycord로 동아리 물품 대여 Bot 만들기 with Notion API"
date: 2023-04-12 00:00:00
categories: [Record, etc]
tags: [python, discord, discord.py, pycord, notion]
published: False
---

# # Intro
동아리 ..

<br>


# # Pycord 설치

> 자세한 내용은 [pycord 공식 guide](https://guide.pycord.dev/installation)를 참고
{: .prompt-info}

1. Python 설치
    Pycord 공식 document에 의하면 Pycord가 v2.0으로 업데이트 되며 **Python3.8** 이상의 버전을 요구하게 되었다. 각자 환경에 맞게 버전을 설치하자.

2. discord.py 제거
    기존에 discord.py가 설치되어 있었다면 제거해야 한다.

    ```shell
    pip uninstall discord.py
    ```
3. py-cord 설치

    ```shell
    pip install py-cord
    ```

<br>

# # Bot Development
