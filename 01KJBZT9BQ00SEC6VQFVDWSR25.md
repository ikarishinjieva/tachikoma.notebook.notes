---
title: 20250601 - 直播音频生成脚本
confluence_page_id: 3997783
created_at: 2025-06-06T11:43:08+00:00
updated_at: 2025-06-06T11:43:08+00:00
---

# 1.py

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-

# pip install flask websocket-client werkzeug

import os
import json
import time
import base64
import hashlib
import hmac
import datetime
import websocket
import uuid
import ssl
import urllib.parse
from wsgiref.handlers import format_date_time
from datetime import datetime
from time import mktime
import threading
import zipfile
from flask import Flask, request, jsonify, send_file, abort
from werkzeug.serving import run_simple

app = Flask(__name__)

# TTS API 配置
API_CONFIG = {
    'app_id': '2742c6f1',  # 从讯飞开放平台获取
    'api_key': '7a06ab07415862e400a0140cdcf4078e',  # 从讯飞开放平台获取
    'api_secret': 'OTAxMTRkODQ1N2Y1ZmY5MDY0MWIwM2Fj',  # 从讯飞开放平台获取
    'api_url': 'wss://cbm01.cn-huabei-1.xf-yun.com/v1/private/mcd9m97e6'  # 根据实际情况修改
}

# 存储生成的脚本和音频
scripts_storage = []
audios_storage = []
output_directory = 'output_audios'  # 存储音频文件的目录

# 确保输出目录存在
if not os.path.exists(output_directory):
    os.makedirs(output_directory)

class AssembleHeaderException(Exception):
    def __init__(self, msg):
        self.message = msg

class Url:
    def __init__(self, host, path, schema):
        self.host = host
        self.path = path
        self.schema = schema

# 解析 URL
def parse_url(request_url):
    stidx = request_url.index("://")
    host = request_url[stidx + 3:]
    schema = request_url[:stidx + 3]
    edidx = host.index("/")
    if edidx <= 0:
        raise AssembleHeaderException("invalid request url:" + request_url)
    path = host[edidx:]
    host = host[:edidx]
    u = Url(host, path, schema)
    return u

# 生成 WebSocket 认证 URL
def assemble_ws_auth_url(request_url, method="GET", api_key="", api_secret=""):
    u = parse_url(request_url)
    host = u.host
    path = u.path
    now = datetime.now()
    date = format_date_time(mktime(now.timetuple()))
    print(f"Date: {date}")

    signature_origin = f"host: {host}\ndate: {date}\n{method} {path} HTTP/1.1"
    print(f"Signature Origin: {signature_origin}")

    signature_sha = hmac.new(api_secret.encode('utf-8'), signature_origin.encode('utf-8'),
                         digestmod=hashlib.sha256).digest()
    signature_sha_base64 = base64.b64encode(signature_sha).decode(encoding='utf-8')
    print(f"Signature SHA Base64: {signature_sha_base64}")

    authorization_origin = f'api_key="{api_key}", algorithm="hmac-sha256", headers="host date request-line", signature="{signature_sha_base64}"'
    authorization = base64.b64encode(authorization_origin.encode('utf-8')).decode(encoding='utf-8')
    print(f"Authorization: {authorization}")

    url_values = {
        "host": host,
        "date": date,
        "authorization": authorization
    }

    url = f"{request_url}?{urllib.parse.urlencode(url_values)}"
    print(f"Final WebSocket URL: {url}")
    return url

# TTS 转换类
class TtsConverter:
    def __init__(self, app_id, api_key, api_secret, text, voice_type, output_file):
        self.app_id = app_id
        self.api_key = api_key
        self.api_secret = api_secret
        self.text = text
        self.voice_type = voice_type
        self.output_file = output_file
        self.success = False
        self.error_msg = None
        self.finished = False

        # 删除已存在的输出文件
        if os.path.exists(output_file):
            os.remove(output_file)

    def on_message(self, ws, message):
        try:
            print(f"收到消息: {message}")
            message_obj = json.loads(message)
            code = message_obj["header"]["code"]
            sid = message_obj["header"]["sid"]

            if "payload" in message_obj:
                if "audio" in message_obj["payload"]:
                    audio = message_obj["payload"]["audio"].get('audio')
                    if audio:
                        audio_data = base64.b64decode(audio)
                        status = message_obj["payload"]['audio'].get("status", 0)

                        with open(self.output_file, 'ab') as f:
                            f.write(audio_data)

                        if status == 2:
                            print(f"WebSocket for {self.output_file} is closed")
                            self.finished = True
                            self.success = True
                            ws.close()

                if code != 0:
                    error_msg = message_obj.get("message", "Unknown error")
                    print(f"sid:{sid} call error:{error_msg} code is:{code}")
                    self.error_msg = error_msg
                    self.finished = True
                    ws.close()
            else:
                print(f"未找到payload，消息内容: {message}")
        except Exception as e:
            print(f"处理消息时出错: {e}")
            self.error_msg = str(e)
            self.finished = True
            ws.close()

    def on_error(self, ws, error):
        print(f"WebSocket错误: {error}")
        self.error_msg = str(error)
        self.finished = True

    def on_close(self, ws, close_status_code=None, close_msg=None):
        print(f"WebSocket关闭: {close_status_code}, {close_msg}")
        self.finished = True

    def on_open(self, ws):
        def run(*args):
            # 构建请求参数
            common_args = {"app_id": self.app_id, "status": 2}
            business_args = {
                "tts": {
                    "vcn": self.voice_type,  # 发音人
                    "volume": 50,
                    "rhy": 0,
                    "speed": 50,
                    "pitch": 50,
                    "bgs": 0,
                    "reg": 0,
                    "rdn": 0,
                    "audio": {
                        "encoding": "lame",
                        "sample_rate": 24000,
                        "channels": 1,
                        "bit_depth": 16,
                        "frame_size": 0
                    }
                }
            }
            data = {
                "text": {
                    "encoding": "utf8",
                    "compress": "raw",
                    "format": "plain",
                    "status": 2,
                    "seq": 0,
                    "text": str(base64.b64encode(self.text.encode('utf-8')), "UTF8")
                }
            }

            d = {
                "header": common_args,
                "parameter": business_args,
                "payload": data,
            }
            d_json = json.dumps(d)
            print(f"发送数据: {d_json}")
            ws.send(d_json)

        threading.Thread(target=run).start()

    def convert(self):
        # 由于认证问题，我们使用模拟数据进行测试
        # 如下是模拟TTS生成成功的情况
        try:
            ws_url = assemble_ws_auth_url(
                API_CONFIG['api_url'],
                "GET",
                self.api_key,
                self.api_secret
            )

            # 开启WebSocket调试
            websocket.enableTrace(True)

            # 创建WebSocket连接
            ws = websocket.WebSocketApp(
                ws_url,
                on_message=self.on_message,
                on_error=self.on_error,
                on_close=self.on_close
            )
            ws.on_open = self.on_open

            # 运行WebSocket
            ws.run_forever(sslopt={"cert_reqs": ssl.CERT_NONE})

            # 等待处理完成
            while not self.finished:
                time.sleep(0.1)

            # 返回处理结果
            return {
                "success": self.success,
                "error": self.error_msg,
                "output_file": self.output_file if self.success else None
            }
        except Exception as e:
            print(f"转换过程出错: {e}")
            self.error_msg = str(e)
            return {
                "success": False,
                "error": self.error_msg,
                "output_file": None
            }

# 生成模拟口播文案
def generate_mock_scripts(data):
    product_name = data.get('productName', '')
    description = data.get('productDescription', '')
    features = data.get('specialFeatures', '')
    audience = data.get('targetAudience', '')
    count = int(data.get('scriptCount', 3))
    length = int(data.get('scriptLength', 500))
    styles = data.getlist('styles') if hasattr(data, 'getlist') else data.get('styles', ['intro'])

    if not styles:
        styles = ['intro']

    # 口播类型模板
    script_types = {
        'intro': {
            'title': "产品介绍",
            'template': "这款{product}采用了{feature}，能够{benefit}。无论您是{audience}，都能体验到它带来的{value}。现在只需{price}元，就能拥有这款{quality}的产品。{product}的设计理念是{concept}，{feature2}让它在同类产品中脱颖而出。它不仅{function}，还能{advantage}，是您{scene}的得力助手。"
        },
        'problem': {
            'title': "问题解决",
            'template': "您是否曾经遇到过{problem}？这款{product}就是为了解决这个问题而设计的。它通过{feature}，有效地{solution}。不再为{pain}而烦恼，让生活更加{better}。很多用户反馈，使用{product}后，{problem2}的问题得到了明显改善，生活质量大大提升。"
        },
        'promo': {
            'title': "促销引导",
            'template': "限时特惠！这款{product}原价{original}元，现在只需{price}元！它不仅{feature}，还能{benefit}。抓紧时间，{action}，数量有限，先到先得！今天下单还送{gift}，超值优惠不容错过。这款产品已经获得了{positive}的好评，现在购买绝对是最明智的选择。"
        },
        'story': {
            'title': "故事情境",
            'template': "记得上个月，我的朋友小李遇到了{problem}的困扰。一次偶然的机会，他开始使用这款{product}，没想到效果出奇的好。他告诉我：'{quote}'。确实，这款产品的{feature}解决了很多人的痛点，让生活变得更加{better}。如果您也面临类似的问题，不妨试试这款神奇的产品。"
        },
        'expert': {
            'title': "专业权威",
            'template': "根据{authority}的研究表明，{statistic}。这款{product}采用了先进的{technology}，能够有效{function}。它的{feature}比市场上同类产品高出{percentage}%，{parameter}指标更是达到了{standard}标准。专家建议，对于{condition}的人群来说，选择这样的产品能够{benefit}，长期使用更是能够{longterm}。"
        },
        'compare': {
            'title': "对比评测",
            'template': "我们对市面上几款主流{category}产品进行了详细测评，结果显示这款{product}在{aspect1}、{aspect2}和{aspect3}方面都具有明显优势。相比竞品A的{competitive1}，我们的产品{advantage1}；相比竞品B的{competitive2}，我们的产品{advantage2}。无论是从性价比还是使用体验来看，这款{product}都是目前市场上的最佳选择。"
        },
        'scene': {
            'title': "场景应用",
            'template': "想象一下，当您{scenario}时，这款{product}会如何改变您的体验？它的{feature}让您能够{benefit}，无论是{use1}还是{use2}，都能带来{advantage}。在{scene1}场景下，您可以通过{method}来使用它；在{scene2}场景中，它则能够通过{function}来帮助您。这款产品真正实现了{slogan}，让您的生活更加便利。"
        },
        'testimony': {
            'title': "用户见证",
            'template': "来自{city}的用户{username}使用这款{product}已经{time}了，他这样评价道：'{comment}'。另一位用户{username2}则分享了自己的使用心得：'{experience}'。这些真实的用户反馈证明了这款产品的实用性和可靠性。如果您也面临{pain}，不妨试试这款已经帮助了众多用户的{product}。"
        }
    }

    # 提取产品特点列表
    feature_list = [f.replace(r'^\d+\.\s*', '') for f in features.split('\n') if f.strip()]
    if not feature_list:
        feature_list = ["高品质", "便携设计", "多功能"]

    # 提取目标受众列表
    audience_list = [a.replace(r'^\d+\.\s*', '') for a in audience.split('\n') if a.strip()]
    if not audience_list:
        audience_list = ["追求生活品质的用户", "现代都市人", "注重健康的消费者"]

    product_price = data.get('productPrice', '299.00')

    scripts = []

    # 根据要求生成指定数量的口播文案
    for i in range(count):
        # 从选定的风格中选择一个
        style = styles[i % len(styles)]
        script_type = script_types.get(style, script_types['intro'])

        # 基于模板生成内容
        template = script_type['template']
        title = script_type['title']

        # 替换模板变量
        content = template
        content = content.replace('{product}', product_name)
        content = content.replace('{feature}', feature_list[i % len(feature_list)])
        content = content.replace('{feature2}', feature_list[(i + 1) % len(feature_list)])
        content = content.replace('{benefit}', "提升您的生活品质")
        content = content.replace('{audience}', audience_list[i % len(audience_list)])
        content = content.replace('{value}', "便利与舒适")
        content = content.replace('{price}', product_price)
        content = content.replace('{quality}', "高品质")
        content = content.replace('{concept}', "优化用户体验")
        content = content.replace('{function}', "净化空气")
        content = content.replace('{advantage}', "改善生活环境")
        content = content.replace('{scene}', "日常生活")
        content = content.replace('{problem}', "空气质量差影响健康")
        content = content.replace('{problem2}', "过敏、哮喘或呼吸不畅")
        content = content.replace('{solution}', "改善室内空气质量")
        content = content.replace('{pain}', "空气污染")
        content = content.replace('{better}', "健康舒适")
        content = content.replace('{original}', str(float(product_price) * 1.3))
        content = content.replace('{action}', "立即下单")
        content = content.replace('{gift}', "专业滤网")
        content = content.replace('{positive}', "98%用户")
        content = content.replace('{quote}', "自从用了这款净化器，我晚上睡觉再也不会因为空气不好而咳嗽了")
        content = content.replace('{authority}', "国家环保研究中心")
        content = content.replace('{statistic}', "80%的呼吸道疾病与室内空气质量有关")
        content = content.replace('{technology}', "HEPA过滤技术")
        content = content.replace('{percentage}', "30")
        content = content.replace('{parameter}', "过滤效率")
        content = content.replace('{standard}', "国际A级")
        content = content.replace('{condition}', "有呼吸道敏感")
        content = content.replace('{longterm}', "降低呼吸系统疾病风险")
        content = content.replace('{category}', "空气净化器")
        content = content.replace('{aspect1}', "过滤效率")
        content = content.replace('{aspect2}', "噪音控制")
        content = content.replace('{aspect3}', "能耗比")
        content = content.replace('{competitive1}', "单层过滤")
        content = content.replace('{advantage1}', "采用五层过滤系统")
        content = content.replace('{competitive2}', "高噪音")
        content = content.replace('{advantage2}', "噪音低至20分贝")
        content = content.replace('{scenario}', "在空气质量较差的环境中")
        content = content.replace('{use1}', "在家中")
        content = content.replace('{use2}', "在办公室")
        content = content.replace('{method}', "一键启动")
        content = content.replace('{scene1}', "家庭")
        content = content.replace('{scene2}', "办公")
        content = content.replace('{slogan}', "随时随地，呼吸清新")
        content = content.replace('{city}', "北京")
        content = content.replace('{username}', "王先生")
        content = content.replace('{username2}', "李女士")
        content = content.replace('{time}', "3个月")
        content = content.replace('{comment}', "这款净化器真的改变了我的生活，家里的空气质量明显提升，孩子的过敏症状也减轻了很多")
        content = content.replace('{experience}', "小巧便携，放在办公桌上一点不占地方，而且效果很好，同事都很羡慕")

        # 确保内容长度符合要求
        if len(content) > length:
            content = content[:length]
            # 确保不会截断句子中间
            last_punctuation = max(
                content.rfind('。'),
                content.rfind('！'),
                content.rfind('？')
            )
            if last_punctuation > 0.9 * length:
                content = content[:last_punctuation + 1]
            else:
                content += "..."
        elif len(content) < length:
            # 自动扩展文本到要求长度
            additional_text = description
            content += " " + additional_text
            # 如果还不够长，再添加一些模板文本
            if len(content) < length:
                more_text = f"这款{product_name}是您生活中不可或缺的好帮手。它的每一个细节都体现了我们对品质的追求。选择它，选择更好的生活。"
                while len(content) < length:
                    content += " " + more_text

            # 最后确保不超过要求长度
            if len(content) > length:
                content = content[:length]

        # 添加脚本
        scripts.append({
            'id': i + 1,
            'title': title,
            'content': content,
            'style': style
        })

    return scripts

# 生成音频文件
def generate_audio_files(scripts, voice_type):
    audios = []

    for script in scripts:
        # 生成唯一文件名
        file_id = uuid.uuid4().hex[:8]
        output_file = os.path.join(output_directory, f"script_{script['id']}_{file_id}.mp3")

        # 创建TTS转换器
        converter = TtsConverter(
            app_id=API_CONFIG['app_id'],
            api_key=API_CONFIG['api_key'],
            api_secret=API_CONFIG['api_secret'],
            text=script['content'],
            voice_type=voice_type,
            output_file=output_file
        )

        # 执行转换
        result = converter.convert()

        if result['success']:
            # 计算时长和文件大小
            file_size = os.path.getsize(output_file)
            # 根据文本长度估算时长，实际应从音频文件中获取
            duration_seconds = len(script['content']) // 4  # 假设每秒4个字

            # 格式化时长显示
            if duration_seconds < 60:
                duration_text = f"{duration_seconds}秒"
            else:
                minutes = duration_seconds // 60
                seconds = duration_seconds % 60
                duration_text = f"{minutes}分{seconds}秒"

            # 格式化文件大小显示
            if file_size < 1024 * 1024:
                file_size_text = f"{file_size / 1024:.1f}KB"
            else:
                file_size_text = f"{file_size / (1024 * 1024):.1f}MB"

            # 构建音频信息
            audio = {
                'id': script['id'],
                'title': script['title'],
                'text': script['content'],
                'style': script['style'],
                'voiceType': voice_type,
                'duration': duration_text,
                'fileSize': file_size_text,
                'audioUrl': f"/audio/{os.path.basename(output_file)}",
                'filePath': output_file
            }

            audios.append(audio)
        else:
            # 处理错误
            print(f"转换失败: {result['error']}")
            # 创建一个模拟的音频条目
            audio = {
                'id': script['id'],
                'title': script['title'],
                'text': script['content'],
                'style': script['style'],
                'voiceType': voice_type,
                'duration': "生成失败",
                'fileSize': "0KB",
                'audioUrl': "",
                'error': result['error']
            }
            audios.append(audio)

    return audios

# API路由：生成脚本
@app.route('/generate_scripts', methods=['POST'])
def api_generate_scripts():
    try:
        # 从请求中获取表单数据
        data = request.form if request.form else request.json

        # 生成脚本
        scripts = generate_mock_scripts(data)

        # 保存到全局存储
        global scripts_storage
        scripts_storage = scripts

        return jsonify({
            "success": True,
            "scripts": scripts
        })
    except Exception as e:
        print(f"生成脚本出错: {e}")
        return jsonify({
            "success": False,
            "error": str(e)
        }), 500

# API路由：更新脚本
@app.route('/update_script', methods=['POST'])
def api_update_script():
    try:
        data = request.json
        index = data.get('index')
        title = data.get('title')
        content = data.get('content')

        if index is None or index < 0 or index >= len(scripts_storage):
            return jsonify({
                "success": False,
                "error": "无效的脚本索引"
            }), 400

        # 更新脚本
        scripts_storage[index]['title'] = title
        scripts_storage[index]['content'] = content

        return jsonify({
            "success": True
        })
    except Exception as e:
        print(f"更新脚本出错: {e}")
        return jsonify({
            "success": False,
            "error": str(e)
        }), 500

# API路由：生成音频
@app.route('/generate_audios', methods=['POST'])
def api_generate_audios():
    try:
        data = request.json
        scripts = data.get('scripts', scripts_storage)
        voice_type = data.get('voiceType', 'x2_yifei')

        # 生成音频文件
        audios = generate_audio_files(scripts, voice_type)

        # 保存到全局存储
        global audios_storage
        audios_storage = audios

        return jsonify({
            "success": True,
            "audios": audios
        })
    except Exception as e:
        print(f"生成音频出错: {e}")
        return jsonify({
            "success": False,
            "error": str(e)
        }), 500

# API路由：获取音频文件
@app.route('/audio/<filename>', methods=['GET'])
def get_audio(filename):
    try:
        file_path = os.path.join(output_directory, filename)
        if not os.path.exists(file_path):
            abort(404)

        return send_file(file_path, as_attachment=False, mimetype='audio/mp3')
    except Exception as e:
        print(f"获取音频文件出错: {e}")
        abort(500)

# API路由：导出所有音频
@app.route('/export_audios', methods=['POST'])
def export_audios():
    try:
        data = request.json
        audio_ids = data.get('audioIds', [])

        # 过滤并获取需要导出的音频
        export_audios = [audio for audio in audios_storage if audio.get('id') in audio_ids]

        if not export_audios:
            return jsonify({
                "success": False,
                "error": "没有可导出的音频"
            }), 400

        # 创建ZIP文件
        zip_filename = os.path.join(output_directory, f"audios_export_{uuid.uuid4().hex[:6]}.zip")
        with zipfile.ZipFile(zip_filename, 'w') as zipf:
            for audio in export_audios:
                if 'filePath' in audio and os.path.exists(audio['filePath']):
                    zipf.write(
                        audio['filePath'],
                        f"口播{audio['id']}_{audio['title']}.mp3"
                    )

        return send_file(zip_filename, as_attachment=True, mimetype='application/zip')
    except Exception as e:
        print(f"导出音频出错: {e}")
        return jsonify({
            "success": False,
            "error": str(e)
        }), 500

# 主页
@app.route('/')
def index():
    return send_file('index.html')

# 测试 API 凭证
@app.route('/test_credentials', methods=['GET'])
def test_credentials():
    try:
        # 构建测试URL
        ws_url = assemble_ws_auth_url(
            API_CONFIG['api_url'],
            "GET",
            API_CONFIG['api_key'],
            API_CONFIG['api_secret']
        )

        return jsonify({
            "success": True,
            "message": "认证URL生成成功",
            "url": ws_url
        })
    except Exception as e:
        return jsonify({
            "success": False,
            "error": str(e)
        }), 500

if __name__ == '__main__':
    print("口播生成助手服务已启动...")
    print(f"API配置: app_id={API_CONFIG['app_id']}, api_key={API_CONFIG['api_key'][:5]}..., url={API_CONFIG['api_url']}")

    # 尝试测试API凭证
    try:
        test_url = assemble_ws_auth_url(
            API_CONFIG['api_url'],
            "GET",
            API_CONFIG['api_key'],
            API_CONFIG['api_secret']
        )
        print(f"认证URL生成成功：{test_url}")
    except Exception as e:
        print(f"认证URL生成失败: {e}")

    # 使用Flask开发服务器运行，生产环境应改用正式WSGI服务器
    run_simple('0.0.0.0', 5432, app, use_reloader=True, use_debugger=True)
``` 

# index.html

```

<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>口播生成助手</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: "PingFang SC", "Hiragino Sans GB", "Microsoft YaHei", "WenQuanYi Micro Hei", sans-serif;
        }

        body {
            background-color: #f0f2f5;
            color: #333;
            font-size: 16px;
            line-height: 1.5;
            background-image: linear-gradient(45deg, #f5f7fa 0%, #c3cfe2 100%);
            background-attachment: fixed;
        }

        .app-container {
            max-width: 100%;
            margin: 0 auto;
            min-height: 100vh;
            position: relative;
            padding-bottom: 80px;
        }

        .app-header {
            background: #fff;
            color: #333;
            padding: 18px 15px;
            text-align: center;
            font-size: 1.3rem;
            font-weight: 600;
            position: sticky;
            top: 0;
            z-index: 100;
            box-shadow: 0 4px 15px rgba(0,0,0,0.08);
            border-bottom: 1px solid rgba(0,0,0,0.05);
            background-image: linear-gradient(to bottom, #ffffff, #f8f9fa);
        }

        .form-container {
            padding: 20px 15px;
        }

        .form-group {
            margin-bottom: 25px;
            background: #fff;
            border-radius: 15px;
            padding: 18px;
            box-shadow: 0 8px 20px rgba(0,0,0,0.08);
            border: 1px solid rgba(0,0,0,0.04);
            position: relative;
            overflow: hidden;
        }

        .form-group::after {
            content: '';
            position: absolute;
            bottom: 0;
            left: 0;
            right: 0;
            height: 5px;
            background: linear-gradient(to right, #4facfe, #00f2fe);
            opacity: 0.8;
        }

        .form-label {
            display: block;
            margin-bottom: 12px;
            font-weight: 600;
            color: #333;
            font-size: 1rem;
            text-shadow: 0 1px 1px rgba(255,255,255,0.8);
        }

        .form-input {
            width: 100%;
            padding: 14px 18px;
            border: 1px solid #ddd;
            border-radius: 12px;
            font-size: 16px;
            transition: all 0.3s;
            background-color: #fff;
            box-shadow: inset 0 2px 5px rgba(0,0,0,0.05);
        }

        .form-input:focus {
            border-color: #4facfe;
            outline: none;
            box-shadow: inset 0 2px 5px rgba(0,0,0,0.05), 0 0 0 3px rgba(79,172,254,0.1);
        }

        textarea.form-input {
            min-height: 120px;
            resize: vertical;
        }

        .voice-selection {
            display: flex;
            justify-content: space-between;
            margin-top: 15px;
            flex-wrap: wrap;
        }

        .voice-option {
            flex: 1;
            margin: 0 5px 10px;
            min-width: calc(33.333% - 10px);
            text-align: center;
        }

        .voice-option input[type="radio"] {
            display: none;
        }

        .voice-option label {
            display: flex;
            flex-direction: column;
            align-items: center;
            padding: 18px 10px;
            border: 1px solid #ddd;
            border-radius: 12px;
            cursor: pointer;
            transition: all 0.3s;
            background: linear-gradient(to bottom, #ffffff, #f5f5f5);
            box-shadow: 0 4px 8px rgba(0,0,0,0.05);
        }

        .voice-option input[type="radio"]:checked + label {
            border-color: #4facfe;
            background: linear-gradient(to bottom, #f0f9ff, #e1f5fe);
            box-shadow: 0 4px 12px rgba(79,172,254,0.15);
            transform: translateY(-3px);
        }

        .voice-icon {
            width: 50px;
            height: 50px;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            margin-bottom: 12px;
            font-size: 22px;
            box-shadow: 0 5px 15px rgba(0,0,0,0.1);
        }

        .male-icon {
            background: linear-gradient(45deg, #4776E6, #8E54E9);
            color: white;
        }

        .female-icon {
            background: linear-gradient(45deg, #FF5858, #F09819);
            color: white;
        }

        .neutral-icon {
            background: linear-gradient(45deg, #2c3e50, #4CA1AF);
            color: white;
        }

        .count-slider-container {
            margin-top: 20px;
        }

        .slider-container {
            display: flex;
            align-items: center;
            margin-top: 15px;
            background: #f9f9f9;
            padding: 12px;
            border-radius: 12px;
            box-shadow: inset 0 2px 5px rgba(0,0,0,0.05);
        }

        .slider-label {
            width: 100px;
            font-weight: 500;
            color: #555;
        }

        .slider {
            flex: 1;
            margin-right: 15px;
            position: relative;
        }

        .slider-marks {
            display: flex;
            justify-content: space-between;
            font-size: 12px;
            color: #999;
            position: absolute;
            width: 100%;
            top: 15px;
        }

        input[type="range"] {
            -webkit-appearance: none;
            width: 100%;
            height: 10px;
            border-radius: 5px;
            background: #ddd;
            outline: none;
            box-shadow: inset 0 2px 5px rgba(0,0,0,0.1);
        }

        input[type="range"]::-webkit-slider-thumb {
            -webkit-appearance: none;
            appearance: none;
            width: 26px;
            height: 26px;
            border-radius: 50%;
            background: linear-gradient(45deg, #4facfe, #00f2fe);
            cursor: pointer;
            box-shadow: 0 3px 10px rgba(0,0,0,0.2);
            border: 2px solid white;
        }

        input[type="range"]::-moz-range-thumb {
            width: 26px;
            height: 26px;
            border-radius: 50%;
            background: linear-gradient(45deg, #4facfe, #00f2fe);
            cursor: pointer;
            box-shadow: 0 3px 10px rgba(0,0,0,0.2);
            border: 2px solid white;
        }

        .slider-value {
            width: 80px;
            text-align: center;
            font-weight: 600;
            font-size: 18px;
            color: #4facfe;
            text-shadow: 0 1px 1px rgba(255,255,255,0.8);
        }

        .slider-input {
            width: 80px;
            text-align: center;
            font-weight: 600;
            font-size: 16px;
            color: #4facfe;
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 5px;
            margin-left: 10px;
        }

        .slider-input:focus {
            border-color: #4facfe;
            outline: none;
        }

        .submit-btn {
            display: block;
            width: 90%;
            padding: 18px;
            background: linear-gradient(45deg, #4facfe, #00f2fe);
            color: white;
            border: none;
            border-radius: 50px;
            font-size: 18px;
            font-weight: 600;
            cursor: pointer;
            margin: 40px auto 20px;
            transition: all 0.3s;
            box-shadow: 0 8px 25px rgba(79,172,254,0.3);
            text-shadow: 0 1px 2px rgba(0,0,0,0.1);
        }

        .submit-btn:hover, .submit-btn:active {
            transform: translateY(-2px);
            box-shadow: 0 10px 30px rgba(79,172,254,0.4);
        }

        .submit-btn:active {
            transform: translateY(1px);
        }

        .footer {
            padding: 20px;
            text-align: center;
            color: rgba(0,0,0,0.5);
            font-size: 0.85rem;
            position: absolute;
            bottom: 0;
            left: 0;
            right: 0;
            background-image: linear-gradient(to top, rgba(255,255,255,0.8), transparent);
            padding-top: 40px;
        }

        /* 口播风格选择 */
        .style-selection {
            display: flex;
            flex-wrap: wrap;
            margin-top: 15px;
            gap: 10px;
        }

        .style-option {
            position: relative;
            flex: 0 0 calc(50% - 10px);
        }

        @media (min-width: 768px) {
            .style-option {
                flex: 0 0 calc(33.333% - 10px);
            }
        }

        .style-option input[type="checkbox"] {
            display: none;
        }

        .style-option label {
            display: flex;
            flex-direction: column;
            padding: 15px;
            border: 1px solid #ddd;
            border-radius: 12px;
            cursor: pointer;
            transition: all 0.3s;
            background: linear-gradient(to bottom, #ffffff, #f5f5f5);
            box-shadow: 0 4px 8px rgba(0,0,0,0.05);
            height: 100%;
        }

        .style-option input[type="checkbox"]:checked + label {
            border-color: #4facfe;
            background: linear-gradient(to bottom, #f0f9ff, #e1f5fe);
            box-shadow: 0 4px 12px rgba(79,172,254,0.15);
            transform: translateY(-3px);
        }

        .style-title {
            font-weight: 600;
            margin-bottom: 8px;
            color: #333;
            display: flex;
            align-items: center;
        }

        .style-icon {
            width: 24px;
            height: 24px;
            border-radius: 50%;
            background: linear-gradient(45deg, #4facfe, #00f2fe);
            color: white;
            display: flex;
            align-items: center;
            justify-content: center;
            margin-right: 8px;
            font-size: 14px;
        }

        .style-description {
            font-size: 13px;
            color: #666;
            line-height: 1.4;
        }

        /* 口播预览页面 */
        .preview-container {
            display: none;
            min-height: 100vh;
            background-color: #f0f2f5;
            padding-bottom: 70px;
        }

        .preview-header {
            background: #fff;
            color: #333;
            padding: 15px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            box-shadow: 0 4px 15px rgba(0,0,0,0.08);
            position: sticky;
            top: 0;
            z-index: 100;
            background-image: linear-gradient(to bottom, #ffffff, #f8f9fa);
        }

        .preview-title {
            font-size: 1.2rem;
            font-weight: 600;
            text-align: center;
            flex: 1;
        }

        .back-btn {
            background: none;
            border: none;
            font-size: 16px;
            display: flex;
            align-items: center;
            justify-content: center;
            cursor: pointer;
            color: #666;
            width: 36px;
            height: 36px;
            border-radius: 50%;
            transition: all 0.2s;
            background: #f0f0f0;
            box-shadow: 0 2px 8px rgba(0,0,0,0.1);
        }

        .back-btn:hover {
            background: #e5e5e5;
        }

        .back-btn:active {
            transform: scale(0.95);
        }

        .product-summary {
            background: #fff;
            padding: 15px;
            margin: 15px;
            border-radius: 12px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.08);
        }

        .product-name {
            font-weight: 600;
            font-size: 18px;
            margin-bottom: 5px;
        }

        .product-meta {
            display: flex;
            justify-content: space-between;
            color: #888;
            font-size: 14px;
        }

        .scripts-container {
            margin: 15px;
        }

        .script-card {
            background: #fff;
            border-radius: 12px;
            margin-bottom: 20px;
            overflow: hidden;
            box-shadow: 0 4px 15px rgba(0,0,0,0.08);
        }

        .script-header {
            padding: 12px 15px;
            background: linear-gradient(to right, #4facfe, #00f2fe);
            color: white;
            font-weight: 600;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .script-number {
            background: rgba(255,255,255,0.2);
            padding: 2px 8px;
            border-radius: 10px;
            font-size: 14px;
        }

        .script-content {
            padding: 15px;
            line-height: 1.6;
            color: #333;
        }

        .script-actions {
            padding: 12px 15px;
            display: flex;
            justify-content: space-between;
            background: #f9f9f9;
            border-top: 1px solid #eee;
        }

        .script-meta {
            color: #888;
            font-size: 14px;
        }

        .edit-btn {
            color: #4facfe;
            cursor: pointer;
            font-weight: 500;
        }

        .preview-footer {
            position: fixed;
            bottom: 0;
            left: 0;
            right: 0;
            padding: 12px 15px;
            background: rgba(255,255,255,0.9);
            backdrop-filter: blur(10px);
            -webkit-backdrop-filter: blur(10px);
            display: flex;
            justify-content: space-between;
            box-shadow: 0 -2px 10px rgba(0,0,0,0.05);
            z-index: 100;
        }

        .footer-btn {
            flex: 1;
            padding: 12px;
            border: none;
            border-radius: 12px;
            font-weight: 600;
            font-size: 15px;
            cursor: pointer;
            transition: all 0.2s;
            box-shadow: 0 3px 10px rgba(0,0,0,0.1);
        }

        .edit-all-btn {
            background: white;
            color: #555;
            margin-right: 10px;
            border: 1px solid #ddd;
        }

        .edit-all-btn:hover {
            background: #f5f5f5;
        }

        .generate-audio-btn {
            background: linear-gradient(45deg, #4facfe, #00f2fe);
            color: white;
            text-shadow: 0 1px 2px rgba(0,0,0,0.1);
        }

        .generate-audio-btn:hover {
            box-shadow: 0 5px 15px rgba(79,172,254,0.3);
            transform: translateY(-2px);
        }

        .generate-audio-btn:active {
            transform: translateY(1px);
            box-shadow: 0 2px 8px rgba(79,172,254,0.3);
        }

        /* 音频播放页面 */
        .audio-container {
            display: none;
            min-height: 100vh;
            background-color: #f0f2f5;
            padding-bottom: 70px;
        }

        .audio-cards-container {
            margin: 15px;
        }

        .audio-card {
            background: #fff;
            border-radius: 12px;
            margin-bottom: 20px;
            overflow: hidden;
            box-shadow: 0 4px 15px rgba(0,0,0,0.08);
        }

        .audio-header {
            padding: 12px 15px;
            background: linear-gradient(to right, #4facfe, #00f2fe);
            color: white;
            font-weight: 600;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .audio-number {
            background: rgba(255,255,255,0.2);
            padding: 2px 8px;
            border-radius: 10px;
            font-size: 14px;
        }

        .audio-content {
            padding: 15px;
        }

        .audio-text {
            margin-bottom: 15px;
            line-height: 1.6;
            color: #333;
            max-height: 100px;
            overflow-y: auto;
            padding: 10px;
            background: #f9f9f9;
            border-radius: 8px;
            font-size: 15px;
        }

        .audio-player-container {
            display: flex;
            align-items: center;
            margin-bottom: 15px;
        }

        .audio-player {
            flex: 1;
        }

        .audio-controls {
            display: flex;
            justify-content: flex-end;
        }

        .audio-btn {
            display: flex;
            align-items: center;
            justify-content: center;
            width: 40px;
            height: 40px;
            border-radius: 50%;
            background: #f0f0f0;
            border: none;
            margin-left: 10px;
            cursor: pointer;
            transition: all 0.2s;
            box-shadow: 0 3px 8px rgba(0,0,0,0.1);
        }

        .audio-btn:hover {
            background: #e5e5e5;
            box-shadow: 0 4px 12px rgba(0,0,0,0.15);
        }

        .audio-btn:active {
            transform: scale(0.95);
        }

        .download-btn {
            background: linear-gradient(45deg, #4facfe, #00f2fe);
            color: white;
        }

        .download-btn:hover {
            background: linear-gradient(45deg, #3d86ca, #00d4e0);
        }

        .audio-meta {
            display: flex;
            justify-content: space-between;
            color: #888;
            font-size: 14px;
            padding-top: 10px;
            border-top: 1px solid #eee;
        }

        .audio-voice-type {
            display: flex;
            align-items: center;
        }

        .voice-indicator {
            width: 12px;
            height: 12px;
            border-radius: 50%;
            margin-right: 5px;
        }

        .male-indicator {
            background: linear-gradient(45deg, #4776E6, #8E54E9);
        }

        .female-indicator {
            background: linear-gradient(45deg, #FF5858, #F09819);
        }

        .neutral-indicator {
            background: linear-gradient(45deg, #2c3e50, #4CA1AF);
        }

        .audio-duration {
            margin-left: 15px;
        }

        .audio-file-size {
            margin-left: auto;
        }

        .export-all-btn {
            display: block;
            width: 90%;
            padding: 15px;
            background: linear-gradient(45deg, #4facfe, #00f2fe);
            color: white;
            border: none;
            border-radius: 12px;
            font-size: 16px;
            font-weight: 600;
            cursor: pointer;
            margin: 30px auto 20px;
            transition: all 0.3s;
            box-shadow: 0 8px 25px rgba(79,172,254,0.3);
            text-shadow: 0 1px 2px rgba(0,0,0,0.1);
            text-align: center;
        }

        .export-all-btn:hover {
            transform: translateY(-2px);
            box-shadow: 0 10px 30px rgba(79,172,254,0.4);
        }

        .export-all-btn:active {
            transform: translateY(1px);
        }

        /* 加载动画 */
        .loading-overlay {
            position: fixed;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background: rgba(255,255,255,0.8);
            display: flex;
            justify-content: center;
            align-items: center;
            z-index: 1000;
            backdrop-filter: blur(5px);
            -webkit-backdrop-filter: blur(5px);
            display: none;
        }

        .loader {
            width: 80px;
            height: 80px;
            border: 8px solid #f3f3f3;
            border-radius: 50%;
            border-top: 8px solid #4facfe;
            -webkit-animation: spin 1s linear infinite;
            animation: spin 1s linear infinite;
        }

        @-webkit-keyframes spin {
            0% { -webkit-transform: rotate(0deg); }
            100% { -webkit-transform: rotate(360deg); }
        }

        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }

        .loading-text {
            position: absolute;
            bottom: -40px;
            font-size: 16px;
            font-weight: 500;
            color: #4facfe;
        }

        /* 响应式调整 */
        @media (max-width: 768px) {
            .voice-selection {
                flex-wrap: wrap;
            }

            .voice-option {
                flex: 0 0 calc(50% - 10px);
                margin-bottom: 10px;
            }

            .form-group {
                padding: 15px;
            }

            .submit-btn {
                width: 100%;
            }
        }

        /* 编辑对话框 */
        .dialog-overlay {
            position: fixed;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background: rgba(0,0,0,0.5);
            display: none;
            justify-content: center;
            align-items: center;
            z-index: 1000;
        }

        .dialog {
            width: 90%;
            max-width: 500px;
            background: white;
            border-radius: 15px;
            overflow: hidden;
            box-shadow: 0 10px 30px rgba(0,0,0,0.2);
        }

        .dialog-header {
            padding: 15px;
            background: linear-gradient(to right, #4facfe, #00f2fe);
            color: white;
            font-weight: 600;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .close-dialog {
            background: none;
            border: none;
            color: white;
            font-size: 20px;
            cursor: pointer;
        }

        .dialog-content {
            padding: 20px;
        }

        .dialog-footer {
            padding: 15px;
            display: flex;
            justify-content: flex-end;
            border-top: 1px solid #eee;
        }

        .dialog-btn {
            padding: 10px 20px;
            border-radius: 5px;
            margin-left: 10px;
            cursor: pointer;
            transition: all 0.2s;
            font-weight: 500;
        }

        .cancel-btn {
            background: #f0f0f0;
            border: none;
            color: #555;
        }

        .save-btn {
            background: linear-gradient(45deg, #4facfe, #00f2fe);
            border: none;
            color: white;
        }

        .dialog-form-group {
            margin-bottom: 20px;
        }

        .dialog-label {
            display: block;
            margin-bottom: 8px;
            font-weight: 500;
            color: #333;
        }

        .dialog-input {
            width: 100%;
            padding: 12px;
            border: 1px solid #ddd;
            border-radius: 8px;
            font-size: 16px;
        }

        .dialog-textarea {
            width: 100%;
            padding: 12px;
            border: 1px solid #ddd;
            border-radius: 8px;
            font-size: 16px;
            min-height: 150px;
            resize: vertical;
        }

        /* 高级字数设置样式 */
        .advanced-word-count {
            display: flex;
            flex-wrap: wrap;
            align-items: center;
            margin-top: 10px;
            gap: 8px;
        }

        .count-preset {
            padding: 5px 10px;
            background: #f0f0f0;
            border-radius: 15px;
            font-size: 14px;
            color: #555;
            cursor: pointer;
            transition: all 0.2s;
        }

        .count-preset:hover {
            background: #e0e0e0;
        }

        .count-preset.active {
            background: #4facfe;
            color: white;
        }

        /* 字数和时长计算帮助 */
        .length-guide {
            margin-top: 10px;
            padding: 10px;
            background: #f0f8ff;
            border-radius: 8px;
            font-size: 13px;
            color: #555;
            display: flex;
            align-items: center;
        }

        .info-icon {
            margin-right: 10px;
            color: #4facfe;
            font-size: 18px;
        }

        /* 标签样式 */
        .tag-chip {
            display: inline-block;
            padding: 2px 8px;
            background: #e1f5fe;
            color: #0288d1;
            border-radius: 12px;
            font-size: 12px;
            margin-right: 5px;
            margin-bottom: 5px;
        }

        /* 进度提示和警告样式 */
        .progress-container {
            padding: 15px;
            background: #f9f9f9;
            border-radius: 8px;
            margin: 15px;
            display: none;
        }

        .progress-title {
            font-weight: 600;
            margin-bottom: 10px;
            display: flex;
            align-items: center;
            justify-content: space-between;
        }

        .progress-bar-container {
            height: 8px;
            background: #e0e0e0;
            border-radius: 4px;
            margin-bottom: 10px;
            overflow: hidden;
        }

        .progress-bar {
            height: 100%;
            background: linear-gradient(to right, #4facfe, #00f2fe);
            width: 0%;
            border-radius: 4px;
            transition: width 0.3s;
        }

        .progress-status {
            display: flex;
            justify-content: space-between;
            color: #666;
            font-size: 13px;
        }

        .status-icon {
            display: inline-block;
            width: 18px;
            height: 18px;
            border-radius: 50%;
            text-align: center;
            line-height: 18px;
            font-size: 12px;
            margin-right: 5px;
        }

        .status-pending {
            background: #ffd54f;
            color: #fff;
        }

        .status-processing {
            background: #4facfe;
            color: #fff;
            animation: pulse 1.5s infinite;
        }

        .status-completed {
            background: #66bb6a;
            color: #fff;
        }

        .status-error {
            background: #ef5350;
            color: #fff;
        }

        @keyframes pulse {
            0% {
                opacity: 1;
            }
            50% {
                opacity: 0.6;
            }
            100% {
                opacity: 1;
            }
        }

        /* 警告消息 */
        .warning-message {
            padding: 12px 15px;
            background: #fff3e0;
            border-left: 4px solid #ff9800;
            color: #e65100;
            border-radius: 4px;
            margin: 15px;
            font-size: 14px;
            display: flex;
            align-items: center;
            display: none;
        }

        .warning-icon {
            margin-right: 10px;
            font-size: 20px;
        }
    </style>
</head>
<body>
    <div class="app-container" id="formPage">
        <header class="app-header">
            口播生成助手
        </header>

        <div class="form-container">
            <form id="productForm" action="/generate_scripts" method="post">
                <div class="form-group">
                    <label class="form-label" for="productName">商品名称</label>
                    <input type="text" id="productName" name="productName" class="form-input" placeholder="请输入商品名称" value="便携式空气净化器" required>
                </div>

                <div class="form-group">
                    <label class="form-label" for="productDescription">商品介绍</label>
                    <textarea id="productDescription" name="productDescription" class="form-input" placeholder="请输入商品详细介绍" required>这款便携式空气净化器采用HEPA高效过滤技术，能过滤99.97%的PM2.5颗粒物和有害气体。超静音设计，续航时间长达12小时，适合家庭、办公室和旅行使用。机身小巧轻便，操作简单，一键启动，实时显示空气质量。</textarea>
                </div>

                <div class="form-group">
                    <label class="form-label" for="productPrice">商品价格</label>
                    <input type="number" id="productPrice" name="productPrice" class="form-input" placeholder="请输入商品价格" step="0.01" value="299.00" required>
                </div>

                <div class="form-group">
                    <label class="form-label" for="specialFeatures">产品卖点</label>
                    <textarea id="specialFeatures" name="specialFeatures" class="form-input" placeholder="请输入产品的主要卖点">1. HEPA高效过滤技术，过滤率达99.97%
2. 超静音设计，噪音低至20分贝
3. 12小时超长续航，满足一天使用需求
4. 实时空气质量监测显示
5. 便携小巧，重量仅300克
6. 适用范围20平方米，适合各种场景使用</textarea>
                </div>

                <div class="form-group">
                    <label class="form-label" for="targetAudience">目标受众</label>
                    <textarea id="targetAudience" name="targetAudience" class="form-input" placeholder="描述您的目标客户群体">1. 有孩子和老人的家庭用户
2. 关注空气质量和健康的年轻白领
3. 经常出差旅行的商务人士
4. 有轻度过敏或呼吸系统问题的人群
5. 追求品质生活的都市人群</textarea>
                </div>

                <!-- 口播风格选择 -->
                <div class="form-group">
                    <label class="form-label">选择口播风格（可多选）</label>
                    <div class="style-selection">
                        <div class="style-option">
                            <input type="checkbox" id="style-intro" name="styles" value="intro" checked>
                            <label for="style-intro">
                                <div class="style-title">
                                    <div class="style-icon">介</div>
                                    产品介绍
                                </div>
                                <div class="style-description">详细介绍产品的特点、功能和用途，突出产品的优势和特色。</div>
                            </label>
                        </div>

                        <div class="style-option">
                            <input type="checkbox" id="style-problem" name="styles" value="problem" checked>
                            <label for="style-problem">
                                <div class="style-title">
                                    <div class="style-icon">解</div>
                                    问题解决
                                </div>
                                <div class="style-description">针对用户痛点，讲述产品如何解决特定问题，强调解决方案。</div>
                            </label>
                        </div>

                        <div class="style-option">
                            <input type="checkbox" id="style-promo" name="styles" value="promo" checked>
                            <label for="style-promo">
                                <div class="style-title">
                                    <div class="style-icon">促</div>
                                    促销引导
                                </div>
                                <div class="style-description">强调价格优势和限时促销，创造购买紧迫感，引导下单。</div>
                            </label>
                        </div>

                        <div class="style-option">
                            <input type="checkbox" id="style-story" name="styles" value="story">
                            <label for="style-story">
                                <div class="style-title">
                                    <div class="style-icon">故</div>
                                    故事情境
                                </div>
                                <div class="style-description">通过讲述故事或创设情境，让用户身临其境，产生共鸣。</div>
                            </label>
                        </div>

                        <div class="style-option">
                            <input type="checkbox" id="style-expert" name="styles" value="expert">
                            <label for="style-expert">
                                <div class="style-title">
                                    <div class="style-icon">专</div>
                                    专业权威
                                </div>
                                <div class="style-description">以专业角度分析产品价值，引用数据和专家观点，增强说服力。</div>
                            </label>
                        </div>

                        <div class="style-option">
                            <input type="checkbox" id="style-compare" name="styles" value="compare">
                            <label for="style-compare">
                                <div class="style-title">
                                    <div class="style-icon">比</div>
                                    对比评测
                                </div>
                                <div class="style-description">将产品与同类产品进行对比，突出本产品的性价比和优势。</div>
                            </label>
                        </div>

                        <div class="style-option">
                            <input type="checkbox" id="style-scene" name="styles" value="scene">
                            <label for="style-scene">
                                <div class="style-title">
                                    <div class="style-icon">场</div>
                                    场景应用
                                </div>
                                <div class="style-description">详细描述产品在各种场景下的应用方式和效果。</div>
                            </label>
                        </div>

                        <div class="style-option">
                            <input type="checkbox" id="style-testimony" name="styles" value="testimony">
                            <label for="style-testimony">
                                <div class="style-title">
                                    <div class="style-icon">证</div>
                                    用户见证
                                </div>
                                <div class="style-description">分享真实用户使用体验和评价，增强产品可信度。</div>
                            </label>
                        </div>
                    </div>
                </div>

                <div class="form-group">
                    <label class="form-label">选择音频角色</label>
                    <div class="voice-selection">
                        <div class="voice-option">
                            <input type="radio" id="male" name="voiceType" value="x5_lingfeiyi_flow" checked required>
                            <label for="male">
                                <div class="voice-icon male-icon">男</div>
                                成熟男声
                            </label>
                        </div>

                        <div class="voice-option">
                            <input type="radio" id="female" name="voiceType" value="x4_lingxiaoxuan_oral">
                            <label for="female">
                                <div class="voice-icon female-icon">女</div>
                                甜美女声
                            </label>
                        </div>
                    </div>
                </div>

                <div class="form-group count-slider-container">
                    <label class="form-label">口播参数设置</label>

                    <div class="slider-container">
                        <div class="slider-label">口播数量</div>
                        <div class="slider">
                            <input type="range" id="scriptCountSlider" name="scriptCount" min="1" max="10" value="1" step="1">
                        </div>
                        <div class="slider-value" id="scriptCountValue">1个</div>
                    </div>

                    <div class="slider-container">
                        <div class="slider-label">口播字数</div>
                        <div class="slider">
                            <input type="range" id="scriptLengthSlider" min="100" max="10000" value="500" step="100">
                            <div class="slider-marks">
                                <span>100</span>
                                <span>5000</span>
                                <span>10000</span>
                            </div>
                        </div>
                        <input type="number" id="scriptLengthInput" name="scriptLength" class="slider-input" value="500" min="100" max="10000">
                    </div>

                    <div class="advanced-word-count">
                        <span class="count-preset" data-value="200">短篇</span>
                        <span class="count-preset" data-value="500">中篇</span>
                        <span class="count-preset" data-value="1000">长篇</span>
                        <span class="count-preset" data-value="3000">详细</span>
                        <span class="count-preset" data-value="5000">完整</span>
                    </div>

                    <div class="length-guide">
                        <div class="info-icon">ℹ️</div>
                        <div>
                            参考：200字约30秒，500字约1.5分钟，1000字约3分钟，3000字约9分钟，5000字约15分钟
                        </div>
                    </div>
                </div>

                <button type="submit" class="submit-btn" id="generateBtn">生成口播文案</button>
            </form>
        </div>

        <div class="footer">
            © 2025 口播生成助手 | 版本 1.0.0
        </div>
    </div>

    <!-- 口播预览页面 -->
    <div class="preview-container" id="previewPage">
        <header class="preview-header">
            <button class="back-btn" id="backBtn">←</button>
            <div class="preview-title">口播文案预览</div>
            <div style="width: 36px;"></div>
        </header>

        <div class="product-summary">
            <div class="product-name" id="previewProductName">便携式空气净化器</div>
            <div class="product-meta">
                <div id="previewProductPrice">¥299.00</div>
                <div id="previewScriptCount">共3个口播</div>
            </div>
        </div>

        <div class="scripts-container" id="scriptsContainer">
            <!-- 口播文案将在这里动态生成 -->
        </div>

        <div class="preview-footer">
            <button class="footer-btn edit-all-btn" id="editScriptsBtn">修改文案</button>
            <button class="footer-btn generate-audio-btn" id="toAudioBtn">生成音频</button>
        </div>
    </div>

    <!-- 音频播放页面 -->
    <div class="audio-container" id="audioPage">
        <header class="preview-header">
            <button class="back-btn" id="backToPreviewBtn">←</button>
            <div class="preview-title">口播音频</div>
            <div style="width: 36px;"></div>
        </header>

        <div class="product-summary">
            <div class="product-name" id="audioProductName">便携式空气净化器</div>
            <div class="product-meta">
                <div id="audioProductPrice">¥299.00</div>
                <div id="audioVoiceType">成熟男声</div>
            </div>
        </div>

        <!-- 音频生成进度 -->
        <div class="progress-container" id="progressContainer">
            <div class="progress-title">
                <span>音频生成进度</span>
                <span id="progressPercent">0%</span>
            </div>
            <div class="progress-bar-container">
                <div class="progress-bar" id="progressBar"></div>
            </div>
            <div class="progress-status">
                <div>已完成: <span id="completedCount">0</span>/<span id="totalCount">3</span></div>
                <div>状态: <span id="currentStatus">等待中</span></div>
            </div>
        </div>

        <div class="warning-message" id="warningMessage">
            <div class="warning-icon">⚠️</div>
            <div>音频生成可能需要几分钟时间，请耐心等待。</div>
        </div>

        <div class="audio-cards-container" id="audioCardsContainer">
            <!-- 音频卡片将在这里动态生成 -->
        </div>

        <div class="export-all-btn" id="exportAllBtn">
            导出所有音频
        </div>

        <div class="preview-footer">
            <button class="footer-btn edit-all-btn" id="regenerateBtn">重新生成</button>
            <button class="footer-btn generate-audio-btn" id="shareBtn">分享文件</button>
        </div>
    </div>

    <!-- 编辑对话框 -->
    <div class="dialog-overlay" id="editDialog">
        <div class="dialog">
            <div class="dialog-header">
                <div>编辑口播文案</div>
                <button class="close-dialog" id="closeDialog">&times;</button>
            </div>
            <div class="dialog-content">
                <div class="dialog-form-group">
                    <label class="dialog-label" for="editScriptTitle">标题</label>
                    <input type="text" id="editScriptTitle" class="dialog-input" placeholder="输入标题">
                </div>
                <div class="dialog-form-group">
                    <label class="dialog-label" for="editScriptContent">内容</label>
                    <textarea id="editScriptContent" class="dialog-textarea" placeholder="输入口播内容"></textarea>
                </div>
            </div>
            <div class="dialog-footer">
                <button class="dialog-btn cancel-btn" id="cancelEditBtn">取消</button>
                <button class="dialog-btn save-btn" id="saveEditBtn">保存</button>
            </div>
        </div>
    </div>

    <!-- 加载动画 -->
    <div class="loading-overlay" id="loadingOverlay">
        <div class="loader"></div>
        <div class="loading-text" id="loadingText">生成中...</div>
    </div>

    <script>
        // 页面元素
        const formPage = document.getElementById('formPage');
        const previewPage = document.getElementById('previewPage');
        const audioPage = document.getElementById('audioPage');
        const generateBtn = document.getElementById('generateBtn');
        const backBtn = document.getElementById('backBtn');
        const backToPreviewBtn = document.getElementById('backToPreviewBtn');
        const toAudioBtn = document.getElementById('toAudioBtn');
        const editScriptsBtn = document.getElementById('editScriptsBtn');
        const regenerateBtn = document.getElementById('regenerateBtn');
        const shareBtn = document.getElementById('shareBtn');
        const exportAllBtn = document.getElementById('exportAllBtn');
        const editDialog = document.getElementById('editDialog');
        const closeDialog = document.getElementById('closeDialog');
        const cancelEditBtn = document.getElementById('cancelEditBtn');
        const saveEditBtn = document.getElementById('saveEditBtn');
        const loadingOverlay = document.getElementById('loadingOverlay');
        const warningMessage = document.getElementById('warningMessage');
        const progressContainer = document.getElementById('progressContainer');
        const progressBar = document.getElementById('progressBar');
        const progressPercent = document.getElementById('progressPercent');
        const completedCount = document.getElementById('completedCount');
        const totalCount = document.getElementById('totalCount');
        const currentStatus = document.getElementById('currentStatus');

        // 滑块元素
        const scriptCountSlider = document.getElementById('scriptCountSlider');
        const scriptCountValue = document.getElementById('scriptCountValue');
        const scriptLengthSlider = document.getElementById('scriptLengthSlider');
        const scriptLengthInput = document.getElementById('scriptLengthInput');

        // 字数预设按钮
        const countPresets = document.querySelectorAll('.count-preset');

        // 存储生成的口播文案
        let generatedScripts = [];
        // 存储生成的音频数据
        let generatedAudios = [];
        // 当前编辑的脚本索引
        let currentEditIndex = -1;

        // 初始化滑块显示
        scriptCountSlider.addEventListener('input', function() {
            scriptCountValue.textContent = this.value + '个';
        });

        // 字数滑块和输入框双向绑定
        scriptLengthSlider.addEventListener('input', function() {
            scriptLengthInput.value = this.value;
        });

        scriptLengthInput.addEventListener('input', function() {
            let value = parseInt(this.value);

            // 验证输入范围
            if (isNaN(value)) {
                value = 500;
            } else if (value < 100) {
                value = 100;
            } else if (value > 10000) {
                value = 10000;
            }

            this.value = value;
            scriptLengthSlider.value = value;

            // 清除所有预设按钮的活跃状态
            countPresets.forEach(preset => {
                preset.classList.remove('active');
            });
        });

        // 字数预设按钮事件
        countPresets.forEach(preset => {
            preset.addEventListener('click', function() {
                const value = parseInt(this.dataset.value);

                // 更新滑块和输入框
                scriptLengthSlider.value = value;
                scriptLengthInput.value = value;

                // 设置活跃状态
                countPresets.forEach(p => {
                    p.classList.remove('active');
                });
                this.classList.add('active');
            });
        });

        // 表单提交处理
        document.getElementById('productForm').addEventListener('submit', function(e) {
            e.preventDefault();

            // 显示加载动画
            loadingOverlay.style.display = 'flex';
            document.getElementById('loadingText').textContent = '正在生成口播文案...';

            // 获取表单数据
            const formData = new FormData(this);

            // 添加选中的风格
            const selectedStyles = [];
            document.querySelectorAll('input[name="styles"]:checked').forEach(style => {
                selectedStyles.push(style.value);
            });

            // 发送AJAX请求到后端生成脚本
            fetch('/generate_scripts', {
                method: 'POST',
                body: formData
            })
            .then(response => response.json())
            .then(data => {
                generatedScripts = data.scripts;

                // 更新预览页数据
                document.getElementById('previewProductName').textContent = formData.get('productName');
                document.getElementById('previewProductPrice').textContent = '¥' + formData.get('productPrice');
                document.getElementById('previewScriptCount').textContent = '共' + generatedScripts.length + '个口播';

                // 渲染口播文案
                renderScripts();

                // 隐藏加载动画并显示预览页
                loadingOverlay.style.display = 'none';
                formPage.style.display = 'none';
                previewPage.style.display = 'block';

                // 滚动到顶部
                window.scrollTo(0, 0);
            })
            .catch(error => {
                console.error('生成脚本出错:', error);
                alert('生成口播文案时出错，请重试');
                loadingOverlay.style.display = 'none';
            });
        });

        // 返回表单页
        backBtn.addEventListener('click', function() {
            previewPage.style.display = 'none';
            formPage.style.display = 'block';
            window.scrollTo(0, 0);
        });

        // 返回预览页
        backToPreviewBtn.addEventListener('click', function() {
            audioPage.style.display = 'none';
            previewPage.style.display = 'block';
            window.scrollTo(0, 0);
        });

        // 前往音频生成页
        toAudioBtn.addEventListener('click', function() {
            // 显示加载动画
            loadingOverlay.style.display = 'flex';
            document.getElementById('loadingText').textContent = '准备生成音频...';

            const productName = document.getElementById('previewProductName').textContent;
            const productPrice = document.getElementById('previewProductPrice').textContent;
            const voiceType = document.querySelector('input[name="voiceType"]:checked').value;

            // 更新音频页数据
            document.getElementById('audioProductName').textContent = productName;
            document.getElementById('audioProductPrice').textContent = productPrice;

            let voiceTypeText = '成熟男声';
            if (voiceType === 'x4_lingxiaoxuan_oral') {
                voiceTypeText = '甜美女声';
            } else if (voiceType === 'x2_xiaobao') {
                voiceTypeText = '主播音色';
            }
            document.getElementById('audioVoiceType').textContent = voiceTypeText;

            // 准备进度显示
            const total = generatedScripts.length;
            totalCount.textContent = total;
            completedCount.textContent = '0';
            progressBar.style.width = '0%';
            progressPercent.textContent = '0%';
            currentStatus.textContent = '准备中';

            // 显示进度条和警告
            progressContainer.style.display = 'block';
            warningMessage.style.display = 'flex';

            // 清空音频容器
            const audioCardsContainer = document.getElementById('audioCardsContainer');
            audioCardsContainer.innerHTML = '';

            // 隐藏加载动画并显示音频页
            loadingOverlay.style.display = 'none';
            previewPage.style.display = 'none';
            audioPage.style.display = 'block';

            // 滚动到顶部
            window.scrollTo(0, 0);

            // 发送请求生成音频
            generateAudios(generatedScripts, voiceType);
        });

        // 请求生成音频
        function generateAudios(scripts, voiceType) {
            const total = scripts.length;
            generatedAudios = [];

            // 更新状态
            currentStatus.textContent = '生成中';

            // 发送请求到后端生成音频
            fetch('/generate_audios', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({
                    scripts: scripts,
                    voiceType: voiceType
                })
            })
            .then(response => response.json())
            .then(data => {
                // 处理返回的音频数据
                generatedAudios = data.audios;

                // 更新进度
                updateProgress(total, total);

                // 渲染音频卡片
                generatedAudios.forEach(audio => {
                    renderAudioCard(audio);
                });

                // 隐藏警告消息
                warningMessage.style.display = 'none';
            })
            .catch(error => {
                console.error('生成音频出错:', error);
                currentStatus.textContent = '生成失败';
                alert('音频生成时出错，请重试');
            });

            // 模拟进度更新（实际项目中会由后端通过WebSocket推送进度）
            let completed = 0;
            const interval = setInterval(() => {
                completed++;
                updateProgress(completed, total);

                if (completed >= total) {
                    clearInterval(interval);
                }
            }, 2000);
        }

        // 更新进度显示
        function updateProgress(completed, total) {
            const percent = Math.floor((completed / total) * 100);
            progressBar.style.width = percent + '%';
            progressPercent.textContent = percent + '%';
            completedCount.textContent = completed;

            if (completed === total) {
                currentStatus.textContent = '已完成';
            }
        }

        // 渲染单个音频卡片
        function renderAudioCard(audio) {
            const audioCardsContainer = document.getElementById('audioCardsContainer');
            const audioCard = document.createElement('div');
            audioCard.className = 'audio-card';

            // 确定音频角色的样式
            let voiceIndicatorClass = 'male-indicator';
            if (audio.voiceType === 'x4_lingxiaoxuan_oral') {
                voiceIndicatorClass = 'female-indicator';
            } else if (audio.voiceType === 'x2_xiaobao') {
                voiceIndicatorClass = 'neutral-indicator';
            }

            // 准备标签HTML
            let tagsHtml = '';
            if (audio.style) {
                const styleLabel = getStyleLabel(audio.style);
                tagsHtml = `<div class="tag-chip">${styleLabel}</div>`;
            }

            audioCard.innerHTML = `
                <div class="audio-header">
                    <div>${audio.title}</div>
                    <div class="audio-number">口播 ${audio.id}</div>
                </div>
                <div class="audio-content">
                    <div class="audio-text">${audio.text}</div>
                    <div class="audio-player-container">
                        <audio class="audio-player" controls>
                            <source src="${audio.audioUrl}" type="audio/mp3">
                            您的浏览器不支持音频播放
                        </audio>
                    </div>
                    <div class="audio-controls">
                        <button class="audio-btn play-btn" data-index="${audio.id - 1}">
                            <svg width="16" height="16" viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
                                <path d="M3 2.5V13.5L13 8L3 2.5Z" fill="#333"/>
                            </svg>
                        </button>
                        <button class="audio-btn download-btn" data-index="${audio.id - 1}">
                            <svg width="16" height="16" viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
                                <path d="M8 11L3 6L4.4 4.6L7 7.175V1H9V7.175L11.6 4.6L13 6L8 11Z" fill="white"/>
                                <path d="M13 14H3V12H1V14C1 15.1 1.9 16 3 16H13C14.1 16 15 15.1 15 14V12H13V14Z" fill="white"/>
                            </svg>
                        </button>
                    </div>
                    <div class="audio-meta">
                        <div class="audio-voice-type">
                            <div class="voice-indicator ${voiceIndicatorClass}"></div>
                            ${audio.voiceType === 'x2_yifei' ? '男声' : (audio.voiceType === 'x4_lingxiaoxuan_oral' ? '女声' : '主播')}
                            ${tagsHtml}
                        </div>
                        <div class="audio-duration">${audio.duration}</div>
                        <div class="audio-file-size">${audio.fileSize}</div>
                    </div>
                </div>
            `;

            audioCardsContainer.appendChild(audioCard);

            // 添加播放按钮事件
            const playBtn = audioCard.querySelector('.play-btn');
            playBtn.addEventListener('click', function() {
                const audio = audioCard.querySelector('audio');
                if (audio.paused) {
                    audio.play();
                } else {
                    audio.pause();
                }
            });

            // 添加下载按钮事件
            const downloadBtn = audioCard.querySelector('.download-btn');
            downloadBtn.addEventListener('click', function() {
                const audioIndex = parseInt(this.dataset.index);
                const audioFile = generatedAudios[audioIndex];

                // 创建下载链接
                const a = document.createElement('a');
                a.href = audioFile.audioUrl;
                a.download = `口播${audioFile.id}_${audioFile.title}.mp3`;
                document.body.appendChild(a);
                a.click();
                document.body.removeChild(a);
            });
        }

        // 编辑所有文案
        editScriptsBtn.addEventListener('click', function() {
            previewPage.style.display = 'none';
            formPage.style.display = 'block';
            window.scrollTo(0, 0);
        });

        // 重新生成
        regenerateBtn.addEventListener('click', function() {
            audioPage.style.display = 'none';
            formPage.style.display = 'block';
            window.scrollTo(0, 0);
        });

        // 分享文件
        shareBtn.addEventListener('click', function() {
            alert('分享功能将在后续版本中推出');
        });

        // 导出所有音频
        exportAllBtn.addEventListener('click', function() {
            // 请求后端创建ZIP文件
            fetch('/export_audios', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({
                    audioIds: generatedAudios.map(a => a.id)
                })
            })
            .then(response => response.blob())
            .then(blob => {
                // 创建下载链接
                const url = window.URL.createObjectURL(blob);
                const a = document.createElement('a');
                a.href = url;
                a.download = '口播音频合集.zip';
                document.body.appendChild(a);
                a.click();
                window.URL.revokeObjectURL(url);
                document.body.removeChild(a);
            })
            .catch(error => {
                console.error('导出音频出错:', error);
                alert('导出音频时出错，请重试');
            });
        });

        // 关闭编辑对话框
        closeDialog.addEventListener('click', function() {
            editDialog.style.display = 'none';
        });

        // 取消编辑
        cancelEditBtn.addEventListener('click', function() {
            editDialog.style.display = 'none';
        });

        // 保存编辑
        saveEditBtn.addEventListener('click', function() {
            if (currentEditIndex >= 0 && currentEditIndex < generatedScripts.length) {
                const newTitle = document.getElementById('editScriptTitle').value;
                const newContent = document.getElementById('editScriptContent').value;

                generatedScripts[currentEditIndex].title = newTitle;
                generatedScripts[currentEditIndex].content = newContent;

                // 更新后端数据
                fetch('/update_script', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                    },
                    body: JSON.stringify({
                        index: currentEditIndex,
                        title: newTitle,
                        content: newContent
                    })
                })
                .then(response => response.json())
                .then(data => {
                    if (data.success) {
                        // 重新渲染文案
                        renderScripts();
                    } else {
                        alert('更新文案失败，请重试');
                    }
                })
                .catch(error => {
                    console.error('更新文案出错:', error);
                    alert('更新文案时出错，请重试');
                });
            }

            editDialog.style.display = 'none';
        });

        // 当点击对话框外部时关闭
        editDialog.addEventListener('click', function(e) {
            if (e.target === this) {
                this.style.display = 'none';
            }
        });

        // 渲染口播文案
        function renderScripts() {
            const scriptsContainer = document.getElementById('scriptsContainer');
            scriptsContainer.innerHTML = '';

            generatedScripts.forEach((script, index) => {
                const scriptCard = document.createElement('div');
                scriptCard.className = 'script-card';

                // 准备标签HTML
                let tagsHtml = '';
                if (script.style) {
                    const styleLabel = getStyleLabel(script.style);
                    tagsHtml = `<span class="tag-chip">${styleLabel}</span>`;
                }

                // 计算大约时长
                const durationInSeconds = Math.floor(script.content.length / 4);
                let durationText = '';
                if (durationInSeconds < 60) {
                    durationText = `${durationInSeconds}秒`;
                } else {
                    const minutes = Math.floor(durationInSeconds / 60);
                    const seconds = durationInSeconds % 60;
                    durationText = `${minutes}分${seconds}秒`;
                }

                scriptCard.innerHTML = `
                    <div class="script-header">
                        <div>${script.title}</div>
                        <div class="script-number">口播 ${index + 1}</div>
                    </div>
                    <div class="script-content">${script.content}</div>
                    <div class="script-actions">
                        <div class="script-meta">
                            ${tagsHtml}
                            ${script.content.length}字 · 约${durationText}
                        </div>
                        <div class="edit-btn" data-index="${index}">编辑</div>
                    </div>
                `;

                scriptsContainer.appendChild(scriptCard);

                // 添加编辑按钮事件
                const editBtn = scriptCard.querySelector('.edit-btn');
                editBtn.addEventListener('click', function() {
                    const scriptIndex = parseInt(this.dataset.index);
                    const script = generatedScripts[scriptIndex];

                    document.getElementById('editScriptTitle').value = script.title;
                    document.getElementById('editScriptContent').value = script.content;

                    currentEditIndex = scriptIndex;
                    editDialog.style.display = 'flex';
                });
            });
        }

        // 获取口播风格标签名称
        function getStyleLabel(style) {
            const styleLabels = {
                'intro': '产品介绍',
                'problem': '问题解决',
                'promo': '促销引导',
                'story': '故事情境',
                'expert': '专业权威',
                'compare': '对比评测',
                'scene': '场景应用',
                'testimony': '用户见证'
            };

            return styleLabels[style] || style;
        }

        // 初始化页面显示
        window.addEventListener('load', function() {
            formPage.style.display = 'block';
            previewPage.style.display = 'none';
            audioPage.style.display = 'none';

            // 初始化预设按钮的第一个为激活状态
            countPresets[1].classList.add('active');
        });
    </script>
</body>
</html>
```
