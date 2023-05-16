# ChatGPT提示工程第8课：聊天格式

在这份笔记本中，您将探索如何利用聊天格式与个性化或专门针对特定任务或行为的聊天机器人进行延伸对话。


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

def get_completion_from_messages(messages, model="gpt-3.5-turbo", temperature=0):
    response = openai.ChatCompletion.create(
        model=model,
        messages=messages,
        temperature=temperature, # this is the degree of randomness of the model's output
    )
#     print(str(response.choices[0].message))
    return response.choices[0].message["content"]
```


```python
messages = [  
{'role':'system', 'content':'你是一个说莎士比亚语的助手.'},    
{'role':'user', 'content':'给我讲个笑话'},   
{'role':'assistant', 'content':'为什么鸡要过马路'},   
{'role':'user', 'content':"我不知道"}
]
response = get_completion_from_messages(messages, temperature=1)
print(response)
```

    为了到对面的餐厅吃炸鸡!
    


```python
messages =  [  
{'role':'system', 'content':'你是一个友好的聊天机器人。'},    
{'role':'user', 'content':'嗨，我是张三'}  ]
response = get_completion_from_messages(messages, temperature=1)
print(response)
```

    你好，张三！很高兴见到你！有什么我可以帮助你的吗？
    


```python
messages =  [  
{'role':'system', 'content':'你是一个友好的聊天机器人。'},    
{'role':'user', 'content':'是的，你能提醒我，我叫什么名字吗？'}  ]
response = get_completion_from_messages(messages, temperature=1)
print(response)
```

    您没有告诉我您的名字。
    


```python
messages =  [  
{'role':'system', 'content':'你是一个友好的聊天机器人。'},
{'role':'user', 'content':'嗨，我是张三'},
{'role':'assistant', 'content': "你好张三！很高兴见到你。有什么我可以帮助你的吗？"},
{'role':'user', 'content':'是的，你能提醒我，我叫什么名字吗？'}  ]
response = get_completion_from_messages(messages, temperature=1)
print(response)
```

    您刚刚告诉我您的名字是张三。
    

## 订单机器人
我们可以自动收集用户提示和助手回复，来构建一个OrderBot。OrderBot将在一家披萨店接收订单。


```python
def collect_messages(_):
    prompt = inp.value_input
    inp.value = ''
    context.append({'role':'user', 'content':f"{prompt}"})
    response = get_completion_from_messages(context) 
    context.append({'role':'assistant', 'content':f"{response}"})
    panels.append(
        pn.Row('User:', pn.pane.Markdown(prompt, width=600)))
    panels.append(
        pn.Row('Assistant:', pn.pane.Markdown(response, width=600, style={'background-color': '#F6F6F6'})))
 
    return pn.Column(*panels)

import panel as pn  
pn.extension()

panels = [] 

context = [ {'role':'system', 'content':"""
你是OrderBot，一个自动化服务，用于收集比萨餐厅的订单。\
你首先向客户问候，然后收集订单，\
然后询问它是取货还是送货。\
您等待收集整个订单，然后概括并最后检查一次\
是否有其他东西可以添加。\
如果这是一次交付，则要求提供地址。\
最后，您收集付款。\
确保澄清所有选项，额外选项和规格，以唯一地\
从菜单中识别该项目。\
您以简短，非常对话友好的风格回答。\
该菜单包括\
意大利辣香肠比萨12.95元，10.00元，7.00元\
芝士披萨10.95元，9.25元，6.50元\
茄子比萨11.95元、9.75元、6.75元\
薯条4.50元、3.50元\
希腊沙拉7.25元\
佐料：\
额外芝士2.00元、\
蘑菇1.50元\
香肠3.00元\
加拿大熏肉3.50元\
人工智能酱1.50元\
辣椒1.00元\
饮料：\
可乐3.00元，2.00元，1.00元\
雪碧3.00元，2.00元，1.00元\
瓶装水5.00元\
"""} ]  

inp = pn.widgets.TextInput(value="你好", placeholder='在这里输入文本...')

button_conversation = pn.widgets.Button(name="聊天！")

interactive_conversation = pn.bind(collect_messages, button_conversation)

dashboard = pn.Column(
    inp,
    pn.Row(button_conversation),
    pn.panel(interactive_conversation, loading_indicator=True, height=300),
)

dashboard
```






<style>.bk-root, .bk-root .bk:before, .bk-root .bk:after {
  font-family: var(--jp-ui-font-size1);
  font-size: var(--jp-ui-font-size1);
  color: var(--jp-ui-font-color1);
}
</style>





    BokehModel(render_bundle={'docs_json': {'d0df2972-6589-4420-a2d4-a27f15e57570': {'defs': [{'extends': None, 'm…




```python
messages =  context.copy()
messages.append(
{'role':'system', 'content':'创建先前食品订单的json汇总。列出每个项目的价格和描述信息\
字段应为1）披萨，包括大小2）配菜列表3）饮料列表，包括大小，4）备选菜单列表包括大小5）总价格 '},    
)

response = get_completion_from_messages(messages, temperature=0)
print(response)
```

    {
      "order": {
        "pizzas": [
          {
            "description": "意大利辣香肠比萨",
            "prices": {
              "small": 7.00,
              "medium": 10.00,
              "large": 12.95
            },
            "size": "medium"
          },
          {
            "description": "芝士披萨",
            "prices": {
              "small": 6.50,
              "medium": 9.25,
              "large": 10.95
            },
            "size": "large"
          },
          {
            "description": "茄子比萨",
            "prices": {
              "small": 6.75,
              "medium": 9.75,
              "large": 11.95
            },
            "size": "small"
          }
        ],
        "sides": [
          {
            "description": "薯条",
            "prices": {
              "small": 3.50,
              "large": 4.50
            },
            "size": "large"
          },
          {
            "description": "希腊沙拉",
            "prices": {
              "small": 7.25
            },
            "size": "small"
          }
        ],
        "drinks": [
          {
            "description": "可乐",
            "prices": {
              "small": 1.00,
              "medium": 2.00,
              "large": 3.00
            },
            "size": "medium"
          },
          {
            "description": "雪碧",
            "prices": {
              "small": 1.00,
              "medium": 2.00,
              "large": 3.00
            },
            "size": "large"
          },
          {
            "description": "瓶装水",
            "prices": {
              "small": 5.00
            },
            "size": "small"
          }
        ],
        "menu_items": [
          {
            "description": "额外芝士",
            "price": 2.00
          },
          {
            "description": "蘑菇",
            "price": 1.50
          },
          {
            "description": "香肠",
            "price": 3.00
          },
          {
            "description": "加拿大熏肉",
            "price": 3.50
          },
          {
            "description": "人工智能酱",
            "price": 1.50
          },
          {
            "description": "辣椒",
            "price": 1.00
          }
        ],
        "total_price": 29.45
      }
    }
    

公众号“开放人工智能”，后台回复“会说话的椰子”获取提示工程体验卡。
