# ChatGPT提示工程第5课：推理
在这篇课程中，您将会推断产品评论和新闻文章中的情感和主题。

## 设置


```python
import openai
import os

from dotenv import load_dotenv, find_dotenv
_ = load_dotenv(find_dotenv())
# 在项目根目录的.env文件中填写你的OpenAI API Key
# 可以在kudaohang.com上获取测试用的key
openai.api_key  = os.getenv('OPENAI_API_KEY')


def get_completion(prompt, model="gpt-3.5-turbo"):
    messages = [{"role": "user", "content": prompt}]
    response = openai.ChatCompletion.create(
        model=model,
        messages=messages,
        temperature=0, # this is the degree of randomness of the model's output
    )
    return response.choices[0].message["content"]
```


```python
## 产品评论
```


```python
lamp_review = """
我需要一盏漂亮的卧室灯，这款灯具不仅有额外的存储空间，
价格也不高。物流速度快，可运输过程中我们的拉线断了，
但公司很愉快地发送了一根新的。也很快就到货了。
它很容易安装。我有一个缺失的零件，所以联系他们的客服，
他们非常快地给我补发了缺失的零件！Lumina 
对我来说是一家非常注重客户和产品的优秀公司！！
"""
```

## 情感（积极/消极）


```python
提示 =f"""
以下产品评论的情感是什么，其中使用三个反引号进行分隔？

评论文本：'' '{lamp_review}' ''
"""
回复=get_completion(提示)
print(回复)
```

    积极的情感。
    


```python
提示 =f"""
以下产品评论的情感是什么，其中使用三个反引号进行分隔？

请以一个单词回答，要么是“positive”（积极），
要么是“negative”（消极）。

评论文本：'' '{lamp_review}' ''
"""
回复=get_completion(提示)
print(回复)
```

    positive
    

## 识别情感类型


```python
提示 = f"""
识别以下评论作者表达的情感类型列表。 列表中的项目不得超过五项。
格式化您的答案为由逗号分隔的小写单词列表。

评论文本：'''{lamp_review}'''
"""
响应= get_completion(提示)
print(响应)
```

    满意, 赞扬, 信任, 愉快, 感激
    

## 识别愤怒


```python
提示 = f"""
以下评论的作者是否表达了愤怒？
评论使用三个重音符号分隔。
以yes或no的形式回答。

评论文本：'''{lamp_review}'''
"""
响应= get_completion(提示)
print(响应)
```

    no
    

## 提取顾客评价中的产品和公司名称


```python
提示 = f"""
从评价文本中识别以下项目：

评价者购买的物品
制造该物品的公司
评价文本使用三个反引号进行分隔。
将您的响应格式化为具有“项目”和“品牌”键的JSON对象。
如果信息不存在，请将“未知”作为值使用。
使您的响应尽可能简短。

评价文本：'''{lamp_review}'''
"""
响应 = get_completion(提示)
print(响应)
```

    {
      "项目": "卧室灯",
      "品牌": "Lumina"
    }
    

## 在单个提示中进行多个任务


```python
提示 = f"""
从评论文本中识别以下项目：
- 情感（正面或负面）
- 评论者是否表达了愤怒？（真或假）
- 评论者购买的商品
- 制造商品的公司

评论用三个反引号分隔。
将您的回答格式化为一个JSON对象，其中
“情感”、“愤怒”、“商品”和“品牌”作为键。
如果信息不存在，请使用“未知”
作为值。
尽可能缩短您的回答。
将愤怒值格式化为布尔值。

评论文本：'''{lamp_review}'''
"""
response = get_completion(提示)
print(response)
```

    {
      "情感": "正面",
      "愤怒": false,
      "商品": "卧室灯",
      "品牌": "Lumina"
    }
    

## 推断话题


```python
story = """ 在政府最近进行的一项调查中， 
公共部门的员工被要求对他们所在的部门的满意度进行评价。
结果显示，NASA是最受欢迎的 部门，满意度达到了95%。
一位NASA的员工，约翰·史密斯，对调查结果发表了评论， 
他说，“我并不惊讶NASA排名第一。 这是一个工作环境很好，
有着优秀的人才和 难以置信的机会的地方。
我为能成为这样一个创新组织的一员而感到自豪。”

NASA的管理团队也对结果表示欢迎， 主任汤姆·约翰逊说，
“我们很高兴 听到我们的员工对他们在NASA的工作感到满意。 
我们有一支才华横溢、敬业的团队，他们不知疲倦地为实现我
们的目标而努力，看到他们的 辛勤工作得到回报真是太棒了。”
调查还显示，社会保障局有着最低的满意度， 只有45%的员工
表示他们对自己的工作感到满意。政府已经承诺 解决调查中
员工提出的问题，并 致力于提高所有部门的工作满意度。 """ 
```


```python

prompt = f""" 确定以下文本中讨论的五个话题，文本用三个反引号分隔。

每个项目用一到两个词表示。

将您的回答格式化为用逗号分隔的项目列表。

文本样本：'''{story}'''
"""

response = get_completion(prompt) 
print(response)

response.split(sep=',')
```

    NASA, 员工满意度调查, 政府部门, 社会保障局, 工作环境
    


```python
topic_list = [ "nasa", "地方政府", "工程", "员工满意度", "联邦政府" ]

提示 = f"""
确定以下主题列表中的每个项目是否是以下用三个反引号分隔的文本中的主题。

按照每个主题给出0或1的列表作为您的答案。
主题列表：{",".join(topic_list)}

文本示例：'''{story}'''
"""
response = get_completion(提示)
print(response)
```

    nasa: 1
    地方政府: 0
    工程: 0
    员工满意度: 1
    联邦政府: 1
    


```python
topic_dict = {i.split(': ')[0]: int(i.split(': ')[1]) for i in response.split(sep='\n')}
if topic_dict['nasa'] == 1:
    print("ALERT: New NASA story!")
```

    ALERT: New NASA story!
    

公众号“开放人工智能”，后台回复“会说话的椰子”获取提示工程体验卡。
