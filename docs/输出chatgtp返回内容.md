#如何利用streamlit构建属于你的Chat-GTP

###实现效果
![](https://foruda.gitee.com/images/1686036034232104459/983df910_5094274.gif)


###环境准备

本地安装`Python 3.10`

登录 [OpenAI官网](https://chat.openai.com/)左侧页面中，找到API Keys，创建属于你的的API Key。

比如我的是（只是举例 ㊙️）

	sk-myNds72DQ22BVwcwqhfWjkashdouymxSnChJhPpN7OQ4vWi9


![](https://img-blog.csdnimg.cn/img_convert/06142e119aa0473a972e67a7844ce565.png)
###导入库
	pip install streamlit openai streamlit-pills

```
Collecting pyrsistent!=0.17.0,!=0.17.1,!=0.17.2,>=0.14.0
  Downloading pyrsistent-0.19.3-cp310-cp310-macosx_10_9_universal2.whl (82 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 82.5/82.5 kB 912.5 kB/s eta 0:00:00
Collecting mdurl~=0.1
  Using cached mdurl-0.1.2-py3-none-any.whl (10.0 kB)
Building wheels for collected packages: validators
  Building wheel for validators (setup.py) ... done
  Created wheel for validators: filename=validators-0.20.0-py3-none-any.whl size=19581 sha256=bcb2dee716ebd8a524893c86c70c006ccf5c7f4ae511f367c6c1fde82dc96e81
  Stored in directory: /Users/hk/Library/Caches/pip/wheels/2d/55/25/123071088f4e466746cbadc923b1a31e08cea99ea9ef6bb35e
Successfully built validators
Installing collected packages: pytz, zipp, urllib3, tzdata, typing-extensions, tqdm, tornado, toolz, toml, tenacity, smmap, six, pyrsistent, pympler, pygments, protobuf, pillow, packaging, numpy, multidict, mdurl, MarkupSafe, idna, frozenlist, decorator, click, charset-normalizer, certifi, cachetools, blinker, attrs, async-timeout, yarl, validators, requests, pytz-deprecation-shim, python-dateutil, pyarrow, markdown-it-py, jsonschema, jinja2, importlib-metadata, gitdb, aiosignal, tzlocal, rich, pydeck, pandas, gitpython, aiohttp, openai, altair, streamlit, streamlit-pills
Successfully installed MarkupSafe-2.1.2 aiohttp-3.8.4 aiosignal-1.3.1 altair-5.0.1 async-timeout-4.0.2 attrs-23.1.0 blinker-1.6.2 cachetools-5.3.1 certifi-2023.5.7 charset-normalizer-3.1.0 click-8.1.3 decorator-5.1.1 frozenlist-1.3.3 gitdb-4.0.10 gitpython-3.1.31 idna-3.4 importlib-metadata-6.6.0 jinja2-3.1.2 jsonschema-4.17.3 markdown-it-py-2.2.0 mdurl-0.1.2 multidict-6.0.4 numpy-1.24.3 openai-0.27.7 packaging-23.1 pandas-2.0.2 pillow-9.5.0 protobuf-4.23.2 pyarrow-12.0.0 pydeck-0.8.1b0 pygments-2.15.1 pympler-1.0.1 pyrsistent-0.19.3 python-dateutil-2.8.2 pytz-2023.3 pytz-deprecation-shim-0.1.0.post0 requests-2.31.0 rich-13.4.1 six-1.16.0 smmap-5.0.0 streamlit-1.23.1 streamlit-pills-0.3.0 tenacity-8.2.2 toml-0.10.2 toolz-0.12.0 tornado-6.3.2 tqdm-4.65.0 typing-extensions-4.6.3 tzdata-2023.3 tzlocal-4.3 urllib3-2.0.2 validators-0.20.0 yarl-1.9.2 zipp-3.15.0
```
ChatGtp调用

`model`：对应的模型

`prompt`：请求gtp的内容

`temperature`：控制生成文本的随机程度的指数

`stream`：是否流式响应

`max_tokens`：gtp返回的最大token数量



```
user_input="你好 Chatgtp"
openai.api_key = "sk-myNds72DQ22BVwcwqhfWjkashdouymxSnChJhPpN7OQ4vWi9"
response =openai.Completion.create(model='text-davinci-003',
                                         prompt=user_input,
                                         max_tokens=4000,
                                         stream=True,
                                         temperature=0.5)
# 输出chatgtp返回内容
print(response.choices[0].text)
```
###完整代码

```
import openai
import streamlit as st

# Set the title and body of your Streamlit app
st.set_page_config(page_title="Open AI", page_icon="🤖")

st.title("Open Ai🤪")

st.subheader("Ask me anything using the OpenAI API! 😄")

# Use radio buttons to ask if user has an OpenAI API key
has_key = st.radio(label="Do you have any OpenAI API key?", options=["No", "Yes"])

if has_key == "Yes":
    # Create input box for OpenAI API key
    openai.api_key = st.text_input("Enter your OpenAI API key:", key="openai_key")
else:
    # Set default OpenAI API key
    openai.api_key = "sk-myNds72DQ22BVwcwqhfWjkashdouymxSnChJhPpN7OQ4vWi9
"

# Create the input box for user question
user_input = st.text_input("Enter your question:", key="input")

# If the user clicks the 'Ask' button, generate and display an answer
if st.button("Ask", key="ask_button"):
    st.markdown("---")
    res_box = st.empty()
    report = []
    # Looping over the response
    for completions in openai.Completion.create(model='text-davinci-003',
                                         prompt=user_input,
                                         max_tokens=4000,
                                         temperature=0.5,
                                         stream=True):
        report.append(completions.choices[0].text)
        res_box.markdown(f'{"".join(report).strip()}')
    st.markdown("---")

# Add some 'about' information and a link to the OpenAI API documentation
st.subheader("About")
st.write("This application uses the OpenAI API to generate answers to user questions.")
st.write("For more information, see the [OpenAI API documentation](https://beta.openai.com/docs/api-reference).")

# Add some footer information for your application
st.markdown("---")
st.write("Created by  tz", "https://chrisjxp.github.io/docx/")

```