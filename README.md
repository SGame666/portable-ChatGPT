# portable-ChatGPT

## Product Definition
可攜式的ChatGPT裝置在設計上應該具有以下功能：<br>
1. 語音交互: 能夠透過語音與使用者進行交互，讓使用者能夠自然地與ChatGPT進行對話。
2. 文字輸入: 除了語音交互外，也應該支援文字輸入，以便在需要時進行靜默操作或者在環境不適合語音交互時使用。
3. 智能對話能力: 具有ChatGPT的核心功能，能夠理解和回應使用者的提問、請求、指令等，並提供相應的回答或解決方案。
4. 個性化設定: 允許使用者根據自己的喜好和需求，對ChatGPT的個性、口吻、語調等進行設定，以使對話更加符合使用者的期望。
5. 即時翻譯: 具有即時翻譯功能，能夠將不同語言的對話翻譯成使用者所選擇的目標語言，從而促進跨語言溝通。
6. 智能建議與推薦: 根據使用者的對話內容和需求，提供相應的建議、推薦或資訊，以幫助使用者解決問題或者滿足需求。
7. 本地化: 考慮到不同地區的文化和語言特點，應該支援多種語言和地區的本地化設定，以提供更加貼近當地用戶的使用體驗。
8. 隱私保護: 尊重使用者的隱私，不收集或者僅收集必要的個人信息，並提供選項讓使用者控制其數據的使用和共享。
9. 學習能力: 具有一定程度的學習能力，能夠根據與使用者的互動不斷優化自身的回答和表現，提供更加智能和個性化的服務
10. 連接性: 具有Wi-Fi或藍牙等連接功能，以便與互聯網或其他設備進行連接，從而擴展其功能或者進行遠程更新和管理。

---
## Development Board Introduciton
### AMB82-MINI overview
<p><img width="50%" height="50%" src="https://www.amebaiot.com/wp-content/uploads/2023/03/amb82_mini.png"></p>

[RTL8735B](https://www.amebaiot.com/en/amebapro2/): 32-bit Arm v8M, up to 500MHz, 768KB ROM, 512KB RAM, 16MB Flash (MCM embedded DDR2/DDR3L up to 128MB) 802.11 a/b/g/n WiFi 2.4GHz/5GHz, BLE 5.1, NN Engine 0.4 TOPS, Crypto Engine, Audo Codec, …

* [Ameba Arduino](https://www.amebaiot.com/en/ameba-arduino-summary/)

* [Amebapro2 AMB82-mini Arduino Example Guides](https://www.amebaiot.com/en/amebapro2-amb82-mini-arduino-peripherals-examples)

* [Amebapro2 AMB82-mini Arduino getting started](https://www.amebaiot.com/en/amebapro2-amb82-mini-arduino-getting-started/)

---
## System Diagram
<p><img width="50%" height="50%" src="https://github.com/rkuo2000/portable-ChatGPT/blob/main/assets/Portable-ChatGPT_System_Diagram.png?raw=true"></p>

---
## Implementation
### Server: 
model_name = "microsoft/Phi-3-mini-128k-instruct"<br>
**Code:** [AmebaPro2_Whisper_LLM_server.py](https://github.com/rkuo2000/portable-ChatGPT/blob/main/AmebaPro2_Whisper_LLM_server.py)<br>
```
@app.post("/audio")
async def post_audio(data: Base64Data):
    try:
        #Decode the base64 string
        decoded_data = base64.b64decode(data.base64_string)
        
        # print(decoded_data)
        #Save the decoded data to an MP4 file
        with open("output.mp4", "wb") as f:
            f.write(decoded_data)
      
        # Whisper transcribe
        result = ASR.transcribe("output.mp4",fp16=False)
        header1 = "ASR: "
        print(header1+result["text"])

        # LLM generate
        prompt = result["text"]
        input_ids = tokenizer.encode(prompt, return_tensors="pt").to("cuda")
        output = LLM.generate(input_ids, max_length=128, num_beams=5, no_repeat_ngram_size=2, pad_token_id=tokenizer.eos_token_id, do_sample=True)
        generated_text = tokenizer.decode(output[0], skip_special_tokens=True)
        header2="LLM: "
        print(header2+generated_text) 
        #return Response(header2+generated_text)
        textout = generated_text.split("</s>")[-1] # for Phi-3
        return Response(header2+textout)
        return Response(header2+textout)
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
```

### Client
**Code:** [RecordMP4_HTTP_Post_Audio_TFTLCD.ino](https://github.com/rkuo2000/portable-ChatGPT/blob/main/RecordMP4_HTTP_Post_Audio_TFTLCD.ino)<br>
```
#define TFT_RESET 5
#define TFT_DC    4
#define TFT_CS    SPI_SS

AmebaILI9341 tft = AmebaILI9341(TFT_CS, TFT_DC, TFT_RESET);

#define ILI9341_SPI_FREQUENCY 20000000

#define FILENAME "TestRecordingAudioOnly.mp4"

char ssid[] = "Your SSID";    // your network SSID (Home WiFi or Smartphone Hotspot)
char pass[] = "Your Password";        // your network password
int status = WL_IDLE_STATUS;

char server[] = "123.195.32.57";   // the server IP running HTTP server on PC
#define PORT 5000
```

---
## Demo Video
[![](https://markdown-videos-api.jorgenkh.no/youtube/7rfmXPqyLF0)](https://youtu.be/7rfmXPqyLF0)

