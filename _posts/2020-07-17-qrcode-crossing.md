---
layout: post
title:  "QR Code Crossing"
author: violetstair
categories: [ Programming ]
tags: [python, programming]
image: assets/images/fawkes.jpg
description: "QRCode 생성하기"
featured: true
hidden: true
---
QR Code에 이미지를 넣어서 생성하기

그냥 해봄 ...

#### QR Code 생성

QR Core를 생성하기 위해 `qrcode` 라이브러리를 사용

> pip install qrcode

```python
import qrcode

qrcode_data = qrcode.QRCode(error_correction=qrcode.constants.ERROR_CORRECT_H)
qrcode_data.add_data('https://violetstair.github.io/')
qrcode_data.make()

qrcode_image = qrcode_data.make_image()
qrcode_image.save('qrcode_test.png')
```

![qrcode](/assets/images/qrcode/qrcode.png)

#### QR Code 색상 변경

qrcode를 생성할 때 `fill_color`와 `back_color`를 추가해 qrcode의 색상을 변경할 수 있다

```python
qrcode_image = qrcode_data.make_image(fill_color="#B43DD8", back_color="white")
```

![qrcode_color](/assets/images/qrcode/qrcode_color.png)

#### QR Code Crossing

qrcode에 이미지를 넣어 생성해보기

`pillow`를 사용해 이미지를 활용할 수 있다

> pip install pillow

```python
# 이미지 불러오기
crossing_image = Image.open(image_path)

...

# qrcode에 넣을 위치 정하기
position = (
    (qrcode_image.size[0] - crossing_image.size[0]) // 2, # 폭
    (qrcode_image.size[1] - crossing_image.size[1]) // 2  # 높이
)

...

# qrcode에 이미지 붙이기
qrcode_image.paste(crossing_image, pos)
```

이미지를 qrcode의 중간에 넣기 위해 `qrcode size - image size`의 절반을 이미지 시작점으로 잡았다

![qrcode_non_resize](/assets/images/qrcode/qrcode_non_resize.png)

인식 불가 ....

이미지가 너무 커서 qrcode를 다 가려서 안되는 듯 하다

이미지 리사이징

```python
crossing_image = crossing_image.resize((crossing_image.size[0] // 2 , crossing_image.size[1] // 2))
```

이미지를 절반으로 줄인 후 qrcode를 다시 생성해 봤다

![qrcode_resize](/assets/images/qrcode/qrcode_resize.png)

생각보다 간편하게 qrcode 생성이 가능
