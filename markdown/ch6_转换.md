# ChatGPT提示工程第6课：转换
在这个笔记本中，我们将探索如何使用大型语言模型进行文本转换任务，例如语言翻译、拼写和语法检查、语气调整和格式转换

## 设置


```python
import openai
import os
import time

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

## 翻译
ChatGPT是通过多种语言的资料进行训练的。这使得模型能够进行翻译。以下是如何利用此功能的一些示例。


```python
prompt = f"""
将以下英文文本翻译成中文：
Hi, I would like to order a blender
"""
response = get_completion(prompt)
print(response)
```

    你好，我想订购一个搅拌机。
    


```python
prompt = f"""
告诉我这是哪种语言：
Combien coûte le lampadaire?
"""
response = get_completion(prompt)
print(response)
```

    这是法语。
    


```python
prompt = f"""
将以下文本翻译成中文、法语、西班牙语和英文

pirate：I want to order a basketball
"""
response = get_completion(prompt)
print(response)
```

    中文：海盗：我想订购一个篮球
    法语：Pirate : Je veux commander un ballon de basket
    西班牙语：Pirata: Quiero pedir una pelota de baloncesto
    英文：Pirate: I want to order a basketball
    


```python
prompt = f"""
将以下文本翻译成西班牙语中的正式和非正式形式：
'Would you like to order a pillow?'
"""
response = get_completion(prompt)
print(response)
```

    正式：¿Le gustaría ordenar una almohada?
    非正式：¿Te gustaría pedir una almohada?
    

## 通用翻译器
想象一下，您是大型跨国电子商务公司的IT负责人。用户使用他们的母语给您发送IT问题。您的员工来自世界各地，只会讲他们的母语。您需要一个通用翻译器！


```python
user_messages = [
"La performance du système est plus lente que d'habitude.", # System performance is slower than normal
"Mi monitor tiene píxeles que no se iluminan.", # My monitor has pixels that are not lighting
"Il mio mouse non funziona", # My mouse is not working
"Mój klawisz Ctrl jest zepsuty", # My keyboard has a broken control key
"我的屏幕在闪烁" # My screen is flashing
]
```


```python
for issue in user_messages:
    prompt = f"告诉我这是哪种语言：{issue}"
    lang = get_completion(prompt)
    print(f"原始信息 ({lang}): {issue}")
    
    time.sleep(19)
    
    prompt = f"""
    将以下文本翻译成英语和韩语：{issue}
    """
    response = get_completion(prompt)
    print(response, "\n")
    time.sleep(19)
```

    原始信息 (这是法语。): La performance du système est plus lente que d'habitude.
    英语：The system performance is slower than usual.
    韩语：시스템 성능이 평소보다 느립니다. 
    
    原始信息 (这是西班牙语。): Mi monitor tiene píxeles que no se iluminan.
    英语翻译：My monitor has pixels that don't light up.
    韩语翻译：내 모니터에는 불이 켜지지 않는 픽셀이 있습니다. 
    
    原始信息 (这是意大利语。): Il mio mouse non funziona
    英语翻译：My mouse is not working.
    韩语翻译：내 마우스가 작동하지 않습니다. 
    
    原始信息 (这是波兰语。): Mój klawisz Ctrl jest zepsuty
    英语：My Ctrl key is broken
    韩语：내 Ctrl 키가 고장 났어요 
    
    原始信息 (这是中文。): 我的屏幕在闪烁
    英语：My screen is flickering.
    韩语：내 화면이 깜빡입니다. 
    
    



## 语气转换
根据目标受众，写作风格可以有所不同。ChatGPT可以产生不同的语气。


```python
prompt = f"""
将以下文本从俚语翻译为正式信函：
'Dude, This is Joe, check out this spec on this standing lamp.'
"""
response = get_completion(prompt)
print(response)
```

    尊敬的先生/女士，我是乔，希望您能查看这个落地灯的规格。
    

## 格式转换
ChatGPT可以在不同格式之间进行翻译。提示应描述输入和输出格式。


```python
data_json = { "resturant employees" :[
    {"name":"Shyam", "email":"shyamjaiswal@gmail.com"},
    {"name":"Bob", "email":"bob32@gmail.com"},
    {"name":"Jai", "email":"jai87@gmail.com"}
]}

prompt = f"""
Translate the following python dictionary from JSON to an HTML \
table with column headers and title: {data_json}
"""
response = get_completion(prompt)
print(response)
```

    <table>
      <caption>Restaurant Employees</caption>
      <thead>
        <tr>
          <th>Name</th>
          <th>Email</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td>Shyam</td>
          <td>shyamjaiswal@gmail.com</td>
        </tr>
        <tr>
          <td>Bob</td>
          <td>bob32@gmail.com</td>
        </tr>
        <tr>
          <td>Jai</td>
          <td>jai87@gmail.com</td>
        </tr>
      </tbody>
    </table>
    


```python
from IPython.display import display, Markdown, Latex, HTML, JSON
display(HTML(response))
```


<table>
  <caption>Restaurant Employees</caption>
  <thead>
    <tr>
      <th>Name</th>
      <th>Email</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Shyam</td>
      <td>shyamjaiswal@gmail.com</td>
    </tr>
    <tr>
      <td>Bob</td>
      <td>bob32@gmail.com</td>
    </tr>
    <tr>
      <td>Jai</td>
      <td>jai87@gmail.com</td>
    </tr>
  </tbody>
</table>


## 拼写/语法检查
以下是常见语法和拼写问题及LLM的响应示例。

为了向LLM表示您希望它校对您的文本，您可以指示模型“校对”或“校对并更正”。


```python
text = [
"The girl with the black and white puppies have a ball.", # The girl has a ball.
"Yolanda has her notebook.", # ok
"Its going to be a long day. Does the car need it’s oil changed?", # 同音异义词
"Their goes my freedom. There going to bring they’re suitcases.", # 同音异义词
"Your going to need you’re notebook.", # 同音异义词
"That medicine effects my ability to sleep. Have you heard of the butterfly affect?", # 同音异义词
"This phrase is to cherck chatGPT for speling abilitty" # 拼写
]
for t in text:
    prompt = f"""校对和纠正以下文本并重写已更正版本。如果您没有发现任何错误，仅说“未发现任何错误”。在文本周围不要使用任何标点符号：
    {t}"""
    response = get_completion(prompt)
    print(response)
    time.sleep(20)
```

    The girl with the black and white puppies has a ball.
    未发现任何错误。
    It's going to be a long day. Does the car need its oil changed? 
    
    重写： 
    今天将是漫长的一天。汽车需要更换机油吗？
    There goes my freedom. They're going to bring their suitcases. 
    
    重写：My freedom is gone as they are taking their suitcases.
    You're going to need your notebook. (You are going to need your notebook.)
    That medicine affects my ability to sleep. Have you heard of the butterfly effect? 
    
    重写： 
    The medicine is affecting my ability to sleep. Have you ever heard of the butterfly effect?
    未发现任何错误。
    


```python
text = f"""
我为女儿的生日买了这个，因为她总是从我的房间拿走我的。
是的，成年人也喜欢熊猫。她把它带到她身边，它超柔软和可爱。 
其中一个耳朵比另一个低一点，我认为这不是设计成不对称的。
但对于我所支付的金额而言，它有点小。 我认为可能有其他更大的选项，
价格相同。 它比预期早了一天，所以在把它送给女儿之前，我可以自己玩一下。
"""
prompt = f"校对并更正这篇评论：{text}"
response = get_completion(prompt)
print(response)
```

    我为女儿的生日买了这个熊猫玩具，因为她总是从我的房间拿走我的。是的，成年人也会喜欢熊猫。她把它带到身边，它超级柔软和可爱。其中一个耳朵比另一个低一点，但我认为这不是设计上的不对称。但是，考虑到我所支付的金额，它有点小。我认为可能有其他更大的选项，价格相同。它比预期早了一天，所以在送给女儿之前，我可以自己先玩一下。
    


```python
text_en = f"""
Got this for my daughter for her birthday cuz she keeps taking \
mine from my room.  Yes, adults also like pandas too.  She takes \
it everywhere with her, and it's super soft and cute.  One of the \
ears is a bit lower than the other, and I don't think that was \
designed to be asymmetrical. It's a bit small for what I paid for it \
though. I think there might be other options that are bigger for \
the same price.  It arrived a day earlier than expected, so I got \
to play with it myself before I gave it to my daughter.
"""
prompt_en = f"proofread and correct this review: ```{text_en}```"
response_en = get_completion(prompt_en)
print(response_en)
```

    I bought this for my daughter's birthday because she always takes mine from my room. Yes, even adults like pandas. She keeps it by her side and it's super soft and cute. One ear is slightly lower than the other, but I don't think it was designed to be asymmetrical. However, for the amount I paid, it's a bit small. I think there may be other larger options for the same price. It arrived a day earlier than expected, so I got to play with it myself before giving it to my daughter.
    


```python
from redlines import Redlines

diff = Redlines(text_en, response_en)
display(Markdown(diff.output_markdown))
```


<span style="color:red;font-weight:700;text-decoration:line-through;">Got </span><span style="color:red;font-weight:700;">I bought </span>this for my <span style="color:red;font-weight:700;text-decoration:line-through;">daughter for her </span><span style="color:red;font-weight:700;">daughter's </span>birthday <span style="color:red;font-weight:700;text-decoration:line-through;">cuz </span><span style="color:red;font-weight:700;">because </span>she <span style="color:red;font-weight:700;text-decoration:line-through;">keeps taking </span><span style="color:red;font-weight:700;">always takes </span>mine from my <span style="color:red;font-weight:700;text-decoration:line-through;">room.  </span><span style="color:red;font-weight:700;">room. </span>Yes, <span style="color:red;font-weight:700;">even </span>adults <span style="color:red;font-weight:700;text-decoration:line-through;">also </span>like <span style="color:red;font-weight:700;text-decoration:line-through;">pandas too.  </span><span style="color:red;font-weight:700;">pandas. </span>She <span style="color:red;font-weight:700;text-decoration:line-through;">takes </span><span style="color:red;font-weight:700;">keeps </span>it <span style="color:red;font-weight:700;text-decoration:line-through;">everywhere with her, </span><span style="color:red;font-weight:700;">by her side </span>and it's super soft and <span style="color:red;font-weight:700;text-decoration:line-through;">cute.  </span><span style="color:red;font-weight:700;">cute. </span>One <span style="color:red;font-weight:700;text-decoration:line-through;">of the ears </span><span style="color:red;font-weight:700;">ear </span>is <span style="color:red;font-weight:700;text-decoration:line-through;">a bit </span><span style="color:red;font-weight:700;">slightly </span>lower than the other, <span style="color:red;font-weight:700;text-decoration:line-through;">and </span><span style="color:red;font-weight:700;">but </span>I don't think <span style="color:red;font-weight:700;text-decoration:line-through;">that </span><span style="color:red;font-weight:700;">it </span>was designed to be asymmetrical. <span style="color:red;font-weight:700;text-decoration:line-through;">It's </span><span style="color:red;font-weight:700;">However, for the amount I paid, it's </span>a bit <span style="color:red;font-weight:700;text-decoration:line-through;">small for what I paid for it though. </span><span style="color:red;font-weight:700;">small. </span>I think there <span style="color:red;font-weight:700;text-decoration:line-through;">might </span><span style="color:red;font-weight:700;">may </span>be other <span style="color:red;font-weight:700;">larger </span>options <span style="color:red;font-weight:700;text-decoration:line-through;">that are bigger </span>for the same <span style="color:red;font-weight:700;text-decoration:line-through;">price.  </span><span style="color:red;font-weight:700;">price. </span>It arrived a day earlier than expected, so I got to play with it myself before <span style="color:red;font-weight:700;text-decoration:line-through;">I gave </span><span style="color:red;font-weight:700;">giving </span>it to my <span style="color:red;font-weight:700;text-decoration:line-through;">daughter.
</span><span style="color:red;font-weight:700;">daughter.</span>



```python
prompt = f"""
校对并纠正此评论。使它更具吸引力。
确保它遵循APA样式指南，并针对高级读者。以Markdown格式输出。
文本：{text}
"""
response = get_completion(prompt)
display(Markdown(response))
```


我为女儿的生日购买了这个熊猫玩具，因为她总是从我的房间拿走我的。即使是成年人也会被它的可爱和柔软所吸引。它的设计非常可爱，虽然其中一个耳朵比另一个低一点，但这并不影响它的魅力。然而，对于我所支付的金额而言，它有点小。我认为可能有其他更大的选项，价格相同。不过，它比预期早了一天，这让我有时间在送给女儿之前先玩一下。如果你正在寻找一个可爱的熊猫玩具作为礼物，这个是一个不错的选择。



```python
prompt_en = f"""
proofread and correct this review. Make it more compelling. 
Ensure it follows APA style guide and targets an advanced reader. 
Output in markdown format.
Text: ```{text}```
"""
response_en = get_completion(prompt_en)
display(Markdown(response_en))
```


I purchased this for my daughter's birthday because she always takes mine from my room. Yes, even adults love pandas. She cuddles with it and it is super soft and adorable. One ear is slightly lower than the other, but I don't think it was designed to be asymmetrical. However, for the amount I paid, it is a bit small. I think there may be other larger options available for the same price. It arrived a day earlier than expected, so I got to play with it myself before giving it to my daughter. Overall, it is a cute and cuddly gift, but consider the size before purchasing. 

As an advanced reader, it is important to note that this review follows APA style guide. The author provides a clear and concise summary of their experience with the product, highlighting both positive and negative aspects. The use of personal anecdotes adds a relatable touch to the review. However, the author could improve the review by providing more specific details about the product, such as its material or dimensions. Additionally, the author could include comparisons to similar products on the market to further emphasize the value of their purchase.


公众号“开放人工智能”，后台回复“会说话的椰子”获取提示工程体验卡。
