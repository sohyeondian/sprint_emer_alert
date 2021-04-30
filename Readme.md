# 긴급 상황 알림



### 목적

---

#### 여러 센서 모니터링 및 작업 환경에서 긴급 상황 발생 시 통신 및 센서 제어

- 긴급 상황 : 화재, 전압 등의 센서 측정 중 감지할 수 있는 상황






### 가상 환경 구축

---

**가상 공장 시뮬레이션**

- 가상의 공장 환경 구축
- Pixcel Display를 이용한 컨베이어 벨트 운영 환경
- Pixcel을 하나의 상품으로 가정
- Psd(Position Sensitive Device)센서 이용

![factory](https://user-images.githubusercontent.com/81665465/116645826-9b047c00-a9b1-11eb-9225-588e693e0f93.png)

1. 거리 센서로 상품의 사이즈 측정
2. 크기 별로 나누어 적재 





### 프로그램 환경

---

![network](https://user-images.githubusercontent.com/81665465/116642954-ec5d3d00-a9aa-11eb-9d42-805feb56234a.png)

- Device : PyC Basic
  - Blynk for Python v0.2.0(Linux)
- UI : Blink Application(v2.26.6)
- PC : Windows 10 



**Code Editor**

- VS Code





### 사용한 센서 모듈

---

- Pixel Display
- Psd
- Leds
- Cds
- Sound
- Oled
- PiezoBuzzer
- Temp/Humi



### 프로그램 구조

---

**긴급 상황을 직접 발생 시키기 어려우므로 Blynk Application의 버튼 사용**

![program](https://user-images.githubusercontent.com/81665465/116642968-f41ce180-a9aa-11eb-81e7-74570467e6c0.png)

#### Status Bar

**<span style="color:green">GREEN</span>**

- Device : 가상 공장 운영

  ![Pixel Display](https://user-images.githubusercontent.com/81665465/116645829-9c35a900-a9b1-11eb-8759-4bd15cbac01b.png)



- Blynk Application : 센서 측정 값 출력

  - LCD widget : Temperature, Humidity
  - Gauge widget : Cds, Sound

  ![Green](https://user-images.githubusercontent.com/81665465/116642941-e4050200-a9aa-11eb-90c6-65eff32f4889.jpeg)

  

**<span style="color:red">RED</span>**

- Device

  - MQTT - 긴급상황게시
  - Leds : 8개 모두 깜빡임
  - OLED : "Warning" 글자 출력, 깜빡임
  - PiezoBuzzer :  경고 소리
  - Blynk - LCD : 긴급상황메시지 전송

  ![emer](https://user-images.githubusercontent.com/81665465/116642917-da7b9a00-a9aa-11eb-99cd-77b1176b0889.jpeg)

- Blynk Application 

  - LCD widget : Device 메시지 출력

  ![Red2](https://user-images.githubusercontent.com/81665465/116642981-fa12c280-a9aa-11eb-967b-ecf81907abbb.jpeg)

- PC

  - MQTT 구독(+/EMERGENCY) 메시지 출력

    



### 라이브러리

---

1. [Pop](https://pop-docs.readthedocs.io/en/latest/pop/)

   ``` python
   import pop
   ```

   - 센서를 이용해 데이터 측정

2. [Blynk](https://github.com/blynkkk/lib-python)

   ```python
   import BlynkLib
   ```

   - 센서 측정값을 받아와 출력 ( 온도, 습도 등 )
   - EMERGENCY Button  : 긴급상황 발생 & 종료

   - MQTT

3. [mqtt](https://github.com/eclipse/paho.mqtt.python)

   ```python
   import paho.mqtt.client
   ```

   - PC로 Topic 발행 : Emergency





### 소스코드

---

#### Virtual Factory

**Factory.py**

- Thread : 온,습도 및 기타 센서와 별도로 실행되어야 함

- big_box , small_box : 상품을 두 사이즈로 분류

  ```python
  class Factory(PopThread):
      def __init__(self):
          super().__init__()
          self.px=PixelDisplay()
          self.psd=Psd()
          self.big_box = []
          self.small_box = []
  ```

  

- 상품 사이즈 별 분류

  - 센서와의 거리가 가까울 수록 물체의 크기가 큰 것으로 간주
  - 테스트 환경에서 확인하기 위해 임의로 기준 값 30 설정
  - Product_size : psd 센서 값

  ```python
  if Product_size <= 30:
  ```

- 상품 적재 : 픽셀 디스플레이 첫 행, 마지막 행에 표시
  - 최대 적재량  : 픽셀 디스플레이(8*8) 중 하나의 행, 8로 설정

  - 상품이 적재될 수 있는 범위 만큼 차면 RGB 값을 0으로 만들어 픽셀 디스플레이 리셋

  - 분류된 상품은 big_box , small_box 리스트가 가지고 있음

  ```python
  if big == 7:
     for i in range(8):
        self.px.setColor(i, 0, [0,0,0])
  ```

    



#### Emergency

- **send.py**

  Topic 발행 및 보내기 위한 클래스 생성

  ```python
  class Send():
      def __init__(self, company, location, topic, addr):
          self.client = mqtt.Client()
          self.client.connect(addr)
          self.company = company
          self.location = location
          self.topic = topic
  ```
  
  Topic 구조 : [회사이름] / [지역 or 공장이름] / [기타]
  
  ```python
  def send(self):
      self.client.publish(self.company+"/"+self.location+"/"+self.topic, self.msg)
  ```
  
  [기타] 토픽 및 메시지 수정으로 긴급 상황이 아니더라도 이용 가능
  
  ```python
  def set_topic(self, topic):
      self.topic = topic
      
  def set_message(self, msg):
      self.msg = msg
  ```
  
  

* **emer.py**

  PC로의 전송 및 경고 센서 제어 클래스

  * send.py 의 Send 클래스 사용

    IP 주소 , 토픽 : "EMERGENCY", 기본메시지 : "Emergency, Help" 으로 초기화

    ```python
    self.__send = Send(company, location, "EMERGENCY", addr)
    self.__send.set_message("EMERGENCY, HELP")
    ```

    

  * 긴급 상황 시 내부 경고를 위한 센서 제어

    Leds : 깜빡이기

    PiezoBuzzer : 소리내기

    Oled : "WARNING" 출력
    
    ```python
    def run(self):
        self.__oled.clearDisplay()
        self.__oled.setCursor(40,25)
        self.__leds.allOff()
        time.sleep(.5)
        self.__leds.allOn()
        self.__oled.print("WARNING")
        self.__pb.tone(3,2,1)
        time.sleep(.5)
    ```



PC

- pc_client.py

  - 회사 모든 지역의 긴급상황을 받을 수 있게 구독 환경 설정

    ```python
    client.subscribe("SODA/+/EMERGENCY")
    ```

  - 날짜, 지역, 필요한 메시지만 출력

    ```python
    def do_message(client, userdata, message):
        date = time.strftime('%Y-%m-%d %X',time.localtime(time.time()))
        topic = message.topic.split("/")
        print(f"[{date}] {topic[1]} : {message.payload.decode()}")
    ```

    




### 기대효과

---

연결된 모든 기기에 알림을 보내므로 빠른 대처 가능

PC의 경우, 해당 알림을 받아 바로 신고하는 프로그램을 만드는 등의 **확장 가능**

현재는 버튼을 이용해서 긴급 상황을 가정했지만 온도, 가스 및 기타 센서 사용하여 다양한 가정 및 산업현장에 접목 가능







### 참고

---

https://github.com/planxlabs/toheaven

https://pop-docs.readthedocs.io/en/latest/pop/

https://github.com/blynkkk/lib-python
