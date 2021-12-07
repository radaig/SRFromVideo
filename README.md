# 用Python解决女朋友看电影没字幕的需求

2020-10-262020-10-26 15:05:52阅读 3000

# 用Python解决女朋友看电影没字幕的需求

### 文章目录

- 用Python解决女朋友看电影没字幕的需求
- - 一、故事情节
    - 二、开发前的准备工作
    - 三、开发过程详细介绍
    - - （一）接口规范说明
        - （二）项目开发
        - - 1、界面部分的实现
            - 2、处理音视频功能开发
            - 3、发送数据翻译功能的开发
    - 四、效果展示
    - 五、总结

## 一、故事情节

是这样子的，女朋友晚上突然翻到了自己喜欢看的一个电影，但是没有字幕，这让她很苦恼。

我急中生智，紧急的解决了我女朋友的需求。

想到了使用Python做一个可以识别语音，然后翻译出来文字的软件。

![](https://ask.qcloudimg.com/http-save/yehe-1150278/or6awg78x8.jpeg?imageView2/2/w/1620)

如下图就是本片文章所要完成的效果，哈哈，是不是还不错，很棒的样子。

如果有兴趣可以给我点个赞，之后带来更多好玩、有趣的demo和实现的教程。

《甄嬛传》第一集的某一小段：

![](https://ask.qcloudimg.com/http-save/yehe-1150278/mtz9f5d0se.png?imageView2/2/w/1620)

**其实，是这样子的：**

最近剧荒，偶然翻出了曾经下载的电视剧回味一番，经典就是经典，不论是剧情还是台词，都那么有魅力，咦？等等，台词，台词……作为一个IT从业者，我忽然灵光一现——现在[语音识别](https://cloud.tencent.com/product/asr?from=10680)技术这么发达，能否有什么办法能帮我保存下一些精彩桥段的台词呢？或许我也可以是个野生字幕君:p ,似乎也可以在此基础上顺手再翻译一下个别难懂的台词！

略加思索，我大概有了个想法——做个视频中提取音频的程序，而后去请求一个开放的语音识别API来帮我把语音转为文字。鉴于之前调用有道智云的愉快经验，我决定再次拿来为我所用，很快做出了这个[demo](https://github.com/LemonQH/SRFromVideo)（请忽略这丑丑的界面布局，能用就行……）。

![](https://ask.qcloudimg.com/http-save/yehe-1150278/gr9z4ac63r.png?imageView2/2/w/1620)

欢迎关注我，`一块来履行我之前的承诺`，`连更一个月之内`，把几篇写完。

文本翻译，单文本翻译，批量翻译demo。
OCR-demo，完成批量上传识别;在一个demo中可选择不同类型的OCR识别《包含手写体/印刷体/身份证/表格/整题/名片），然后调用平台能力，具体实现步骤等。
语音识别demo，demo中上传—段视频，并截取视频中短语音识别-demo的一段音频进行短语音识别



## 二、开发前的准备工作

首先，是需要在有道智云的个人页面上创建实例、创建应用、绑定应用和实例，获取调用接口用到的应用的id和密钥。具体个人注册的过程和应用创建过程详见文章[不到100行代码搞定Python做OCR识别身份证，文字等各种字体](https://blog.csdn.net/qq_17623363/article/details/108588629)

![](https://ask.qcloudimg.com/http-save/yehe-1150278/30upsj8r94.png?imageView2/2/w/1620)

## 三、开发过程详细介绍

下面介绍具体的代码开发过程。

### （一）接口规范说明

首先分析有道智云的API输入输出规范。根据[文档](https://ai.youdao.com/DOCSIRMA/html/%E8%AF%AD%E9%9F%B3%E8%AF%86%E5%88%ABASR/API%E6%96%87%E6%A1%A3/%E7%9F%AD%E8%AF%AD%E9%9F%B3%E8%AF%86%E5%88%AB%E6%9C%8D%E5%8A%A1/%E7%9F%AD%E8%AF%AD%E9%9F%B3%E8%AF%86%E5%88%AB%E6%9C%8D%E5%8A%A1-API%E6%96%87%E6%A1%A3.html)来看，调用接口格式如下：

**有道语音识别API HTTPS地址：**

https://openapi.youdao.com/asrapi

**接口调用参数:**
|字段名|类型|含义|必填|备注|
|---|---|---|---|---|
|q|text|要翻译的音频文件的Base64编码字符串|True|必须是Base64编码|
|langType|text|源语言|True|支持语言|
|appKey|text|应用ID|True|可在应用管理查看|
|salt|text|UUID|True|UUID|
|curtime|text|时间戳（秒）|true|秒数|
|sign|text|签名，通过md5(应用ID+q+salt+curTime+密钥)生成|True|应用ID+q+salt+curTime+密钥的MD5值|
|signType|text|签名版本|True|v2|
|format|text|语音文件的格式，wav|true|wav|
|rate|text|采样率，推荐16000采用率|true|16000|
|channel|text|声道数，仅支持单声道，请填写固定值1|true|1|
|type|text|上传类型，仅支持base64上传，请填写固定值1|true|1|其中q为base64编码的待识别音频文件，“上传的文件时长不能超过120s，文件大小不能超过10M”，这点需要注意一下。
API的**返回内容**较为简单：
|字段|含义|
|---|---|
|errorCode|识别结果错误码，一定存在。详细信息参加错误代码列表|
|result|识别结果，识别成功一定存在|

### （二）项目开发

这个项目使用`python3`开发，包括`maindow.py`，`videoprocess.py`，`srbynetease.py`三个文件。

`界面`部分，使用python自带的`tkinter库`，提供视频文件选择、时间输入框和确认按钮；

`videoprocess.py`:来实现在视频的指定时间区间提取音频和处理API返回信息的功能；

`srbynetease.py`:将处理好的音频发送到短语音识别API并返回结果。

#### 1、界面部分的实现

界面部分代码如下，比较简单。

root\=tk.Tk()
root.title("netease youdao sr test")
frm \= tk.Frame(root)
frm.grid(padx\='50', pady\='50')

btn\_get\_file \= tk.Button(frm, text\='选择待识别视频', command\=get\_file)
btn\_get\_file.grid(row\=0, column\=0,  padx\='10', pady\='20')
path\_text \= tk.Entry(frm, width\='40')
path\_text.grid(row\=0, column\=1)

start\_label\=tk.Label(frm,text\='开始时刻：')
start\_label.grid(row\=1,column\=0)
start\_input\=tk.Entry(frm)
start\_input.grid(row\=1,column\=1)

end\_label\=tk.Label(frm,text\='结束时刻：')
end\_label.grid(row\=2,column\=0)
end\_input\=tk.Entry(frm)
end\_input.grid(row\=2,column\=1)

sure\_btn\=tk.Button(frm, text\='开始识别', command\=start\_sr)
sure\_btn.grid(row\=3,column\=0,columnspan\=3)
root.mainloop()

其中sure\_btn的绑定事件start\_sr()做了简单的异常处理，并通过弹窗打印最终的识别结果:

def start\_sr():
    print(video.video\_full\_path)
 if len(path\_text.get())\==0:
        sr\_result \= '未选择文件'
    else:
        video.start\_time \= int(start\_input.get())
        video.end\_time \= int(end\_input.get())
        sr\_result\=video.do\_sr()

    tk.messagebox.showinfo("识别结果", sr\_result)

#### 2、处理音视频功能开发

（1）在videoprocess.py中，我用到了python的moviepy库来处理视频，按指定起止时间截取视频，提取音频，并按API要求转为base64编码形式：

def get\_audio\_base64(self):
    video\_clip\=VideoFileClip(self.video\_full\_path).subclip(self.start\_time,self.end\_time)
    audio\=video\_clip.audio
    result\_path\=self.video\_full\_path.split('.')\[0\]+'\_clip.mp3'
    audio.write\_audiofile(result\_path)
    audio\_base64 \= base64.b64encode(open(result\_path,'rb').read()).decode('utf-8')
    return audio\_base64

（2）处理好的音频文件编码传到封装好的有道智云API调用方法中：

def do\_sr(self):
    audio\_base64\=self.get\_audio\_base64()
    sr\_result\=srbynetease.connect(audio\_base64)
    print(sr\_result)
    if sr\_result\['errorCode'\]\=='0':
        return sr\_result\['result'\]
    else:
        return "Something wrong , errorCode:"+sr\_result\['errorCode'\]

#### 3、发送数据翻译功能的开发

srbynetease.py中封装的调用方法比较简单，按API文档“组装”好data{}发送即可：

def connect(audio\_base64):
    data \= {}
    curtime \= str(int(time.time()))
    data\['curtime'\] \= curtime
    salt \= str(uuid.uuid1())
    signStr \= APP\_KEY + truncate(audio\_base64) + salt + curtime + APP\_SECRET
    sign \= encrypt(signStr)
    data\['appKey'\] \= APP\_KEY
    data\['q'\] \= audio\_base64
    data\['salt'\] \= salt
    data\['sign'\] \= sign
    data\['signType'\] \= "v2"
    data\['langType'\] \= 'zh-CHS'
    data\['rate'\] \= 16000
    data\['format'\] \= 'mp3'
    data\['channel'\] \= 1
    data\['type'\] \= 1

    response \= do\_request(data)

    return json.loads(str(response.content,'utf-8'))

## 四、效果展示

随手打开《甄嬛传》第一集的某一小段试试：

![](https://ask.qcloudimg.com/http-save/yehe-1150278/hw6r7gnk46.png?imageView2/2/w/1620)

效果可以，断句的一点小瑕疵可以忽略。没想到这短语音识别API博古通今，古文语音识别也这么溜，厉害厉害！

## 五、总结

一番尝试带我打开了新世界的大门，从今天开始我可以是一个不打字却能搬运字幕的野生字幕君了，后面再有时间可以试试识别完翻译成其他语言的操作，嗯，是技术的力量！

项目地址：https://github.com/LemonQH/SRFromVideo
