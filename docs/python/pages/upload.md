# 文件或图片上传
> 文件或图片上传

## 一、单文件上传文件

### 1.1. 使用`MultipartEncoder`上传文件
```python
import requests
from requests_toolbelt import MultipartEncoder

def upload_multipart_encoder(file_path, file_type = "png/jpg", server_uri = "http://upload.xxxxx.com"):
    # from_data上传文件，注意参数名propertyMessageXml
    payload = MultipartEncoder(fields={'propertyMessageXml': ('filename', open(local, 'rb'), file_type)})
    headers = { 'Content-Type': payload.content_type }
    respoonse = requests.post(server_uri, data = payload, headers = headers).json()
    return respoonse

if __name__=='__main__':
    res = upload_multipart_encoder('data/123.png') # 上传文件 (默认为png图片)
    res = upload_multipart_encoder('data/123.jpeg', "jpeg/jpg") # 上传jpeg
    res = upload_multipart_encoder('data/123.xml', "text/xml") # 上传xml文件
    print(res)
```

### 1.2. 使用`raw`上传文件
```python
import requests

def upload_raw(file_path, file_type = "png/jpg", server_uri = "http://upload.xxxxx.com"):
    headers = { 'Content-Type': file_type }
    file = open(file_path,'rb')
    requests.post(server_uri, data=file.read(), headers = headers).json()
    return respoonse

if __name__=='__main__':
    res = upload_raw('data/123.png') # 上传文件 (默认为png图片)
    res = upload_raw('data/123.jpeg', "jpeg/jpg") # 上传jpeg
    res = upload_raw('data/123.xml', "text/xml") # 上传xml文件
    print(res)
```

### 1.3. 使用`binary`上传文件
```python
import requests

def upload_binary(file_path, server_uri = "http://upload.xxxxx.com"):
    headers = { 'Content-Type': "binary" }
    files = {
        'file':open(file_path,'rb')
    }
    requests.post(server_uri, files = files, headers = headers).json()
    return respoonse

if __name__=='__main__':
    res = upload_binary('data/123.png') # 上传png
    res = upload_binary('data/123.jpeg') # 上传jpeg
    res = upload_binary('data/123.xml') # 上传xml文件
    print(res)
```

## 二、批量上传
```python
import requests, glob
from urllib3 import encode_multipart_formdata

def upload_file(url = None, path = None, file_path = None):
    if path:
        for file_path in glob.glob(path + '\*'): #批量文件
            data = {}
            data['file'] = (file_path.split("/")[-1], open(file_path, 'rb').read())  # 名称，读文件
            encode_data = encode_multipart_formdata(data)
            res = requests.post(url, headers = {'Content-Type':encode_data[1]},data=encode_data[0])
            return res.text

    if file_path:
        data = {}
        data['file'] = (file_path.split("/")[-1], open(file_path, 'rb').read())  # 名称，读文件
        encode_data = encode_multipart_formdata(data)
        res = requests.post(url, headers={'Content-Type': encode_data[1]}, data=encode_data[0])
        return res.text
```

## 三、其他上传方式

### 3.1. 图片上传
```python
import requests

def sendImg(img_path, img_name, img_type='image/jpeg'):
    """
    :param img_path:图片的路径
    :param img_name:图片的名称
    :param img_type:图片的类型,这里写的是image/jpeg，也可以是png/jpg
    """
    url = 'https://www.xxxxxxxxxx.com' # 自己想要请求的接口地址

    with open(img_path + img_name, "rb")as f_abs：# 以2进制方式打开图片
    body = {
        # 有些上传图片时可能会有其他字段,比如图片的时间什么的，这个根据自己的需要

        'camera_code': (None, "摄像头1"), 

        'image_face': (img_name, f_abs, img_type)
        # 图片的名称、图片的绝对路径、图片的类型（就是后缀）

        "time":(None, "2019-01-01 10:00:00")
    }
    # 上传图片的时候，不使用data和json，用files
    response = requests.post(url=url, files=body).json()
    return response
if __name__=='__main__':
    # 上传图片
    res = sendImg(img_path, img_name) # 调用sendImg方法
    print(res)
```


### 3.2. 文件上传
```python
from urllib3 import encode_multipart_formdata
import requests

def sendFile(filename, file_path):
    """
    :param filename：文件的名称
    :param file_path：文件的绝对路径
    """
    url = "https://www.xxxxxxx.com" # 请求的接口地址
    with open(file_path, mode="r", encoding="utf8")as f: # 打开文件
    file = {
        "file": (filename, f.read()),# 引号的file是接口的字段，后面的是文件的名称、文件的内容
        "key": "value", # 如果接口中有其他字段也可以加上
    } 

    encode_data = encode_multipart_formdata(file)

    file_data = encode_data[0] 
    # b'--c0c46a5929c2ce4c935c9cff85bf11d4\r\nContent-Disposition: form-data; name="file"; filename="1.txt"\r\nContent-Type: text/plain\r\n\r\n...........--c0c46a5929c2ce4c935c9cff85bf11d4--\r\n

    headers_from_data = {
        "Content-Type": encode_data[1], 
        "Authorization": "Bearer {0}".format(token)
    } 
    # token是登陆后给的值，如果你的接口中头部不需要上传字段，就不用写，只要前面的就可以
    # 'Content-Type': 'multipart/form-data; boundary=c0c46a5929c2ce4c935c9cff85bf11d4'，这里上传文件用的是form-data,不能用json

    response = requests.post(url=url, headers=headers_from_data, data=file_data).json()
    return response

if __name__=='__main__':
    # 上传文件
    res = sendFile(filename, file_path) # 调用sendFile方法
    print(res)
```