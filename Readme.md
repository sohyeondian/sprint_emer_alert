# 긴급 상황 알림

-------



### 목적

---

#### 긴급 상황 발생 시 통신 및 센서 제어

- 긴급 상황 : 화재, 전압 등의 센서 측정 중 감지할 수 있는 상황

  ** 긴급 상황을 직접 발생 시키기 어려우므로 Blynk Application의 버튼 사용





### 가상 환경 구축

---

- 가상의 공장 환경 구축
- Pixcel Display를 이용한 컨베이어 벨트 운영 환경
- Position Sensitive Device 센서를 이용 양쪽으로 분류





### 프로그램 환경

---

![image-20210429161743332](C:\Users\PC-03\AppData\Roaming\Typora\typora-user-images\image-20210429161743332.png)

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

![image-20210430103019319](C:\Users\PC-03\AppData\Roaming\Typora\typora-user-images\image-20210430103019319.png)

#### Status Bar

**<span style="color:green">GREEN</span>**

- Device : 가상 공장 운영

- Blynk Application : 센서 측정 값 출력

  - LCD widget : Temperature, Humidity
  - Gauge widget : Cds, Sound

  <img src="C:\Users\PC-03\Downloads\Green.jpeg" alt="Green" style="zoom:50%;" />

  

**<span style="color:red">RED</span>**

- Device

  - MQTT - 긴급상황게시
  - Leds : 8개 모두 깜빡임
  - OLED : "Warning" 글자 출력, 깜빡임
  - PiezoBuzzer :  경고 소리
  - Blynk - LCD : 긴급상황메시지 전송

  <img src="C:\Users\PC-03\Downloads\949BA829-A765-48F0-B01E-17165620BBC0.jpeg" alt="949BA829-A765-48F0-B01E-17165620BBC0" style="zoom: 25%;" />

- Blynk Application 

  - LCD widget : Device 메시지 출력

  <img src="C:\Users\PC-03\Downloads\Red2.jpeg" alt="Red2" style="zoom:50%;" />

- PC

  - MQTT 구독(+/EMERGENCY) 메시지 출력

    



### 라이브러리

---

1. Pop

   ``` python
   import pop
   ```

   - 센서를 이용해 데이터 측정

2. Blynk

   ```python
   import BlynkLib
   ```

   - 센서 측정값을 받아와 출력 ( 온도, 습도 등 )
   - EMERGENCY Button  : 긴급상황 발생 & 종료

   - MQTT

3. mqtt

   ```python
   import paho.mqtt.client
   ```

   - PC로 Topic 발행 : Emergency





### 소스코드

---

#### Factory

**가상 공장 시뮬레이션**







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


  ``` python
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












### 기대효과

---

현재는 버튼으로 조절하는 

연결된 모든 기기에 알림을 보내므로 빠른 대처 가능

PC의 경우, 해당 알림을 받아 바로 신고하는 프로그램을 만드는 등의 **확장 가능**







### 참고

---

https://github.com/planxlabs/toheaven

https://pop-docs.readthedocs.io/en/latest/pop/

https://github.com/blynkkk/lib-python