import streamlit as st

# 1. 添加这行配置，禁止浏览器对 Streamlit DOM 节点进行自动翻译冲突
st.set_page_config(
    page_title="微信流水贷前风控分析系统",
    layout="centered"
)

# 2. 注入 CSS 规避某些翻译插件对 DOM 的侵入
st.markdown('<html lang="zh-CN" class="notranslate" translate="no">', unsafe_allow_html=True)

import pdfplumber
import re

st.title("微信流水贷前风控分析系统")
# ... 下面保持您原来的代码不变 ...

import streamlit as st
import pdfplumber
import re

st.title("微信流水贷前风控分析系统")

uploaded_file = st.file_uploader("上传微信支付流水 PDF", type=["pdf"])

if uploaded_file is not None:
    txs = []
    with pdfplumber.open(uploaded_file) as pdf:
        for page in pdf.pages:
            text = page.extract_text()
            if text:
                for line in text.split('\n'):
                    match = re.search(r'(\d{4}-\d{2}-\d{2}\s+\d{2}:\d{2}:\d{2})\s*\|\s*([^|]+)\s*\|\s*(收入|支出)\s*\|\s*([^|]*)\s*\|\s*([\d\.]+)\s*\|\s*(.*)', line)
                    if match:
                        txs.append({
                            "time": match.group(1),
                            "type": match.group(2),
                            "dir": match.group(3),
                            "amount": float(match.group(5)),
                            "party": match.group(6)
                        })
    
    biz_income = sum(t["amount"] for t in txs if t["dir"] == "收入" and t["type"] == "二维码收款")
    total_in = sum(t["amount"] for t in txs if t["dir"] == "收入")
    total_out = sum(t["amount"] for t in txs if t["dir"] == "支出")
    
    st.success("解析完成！")
    col1, col2, col3 = st.columns(3)
    col1.metric("总收入", f"¥{total_in:.2f}")
    col2.metric("真实经营收入", f"¥{biz_income:.2f}")
    col3.metric("总支出", f"¥{total_out:.2f}")
    
    st.subheader("建议授信额度")
    st.write(f"**¥ {max(biz_income * 3.5, 10000.0):,.2f} 元**")
