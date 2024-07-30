---
layout: post
title: 모터 엔코더
date: 2024-07-30 09:00 +0900
last_modified_at: 2024-07-30 09:00 +0900
tags: [Embedded system ,Robot]
toc:  true
---
## Conclusion
산업에서 사용하는 모터의 이해

<br></br>

## Concept

### 증분형 엔코더

블로그에 소개될때는, 증분형엔코더는 속도값이 절대엔코더는 위치값이 나오는 엔코더로 소개하는데 피곤하게 따지게보면 결국 raw data는 위치값이 나오는것 아닌가?

<center>
  <img src="/upload_image/motor_encoder/incremetal_encoder.png" alt="Large example image" width="500"/>
</center>

- 한개의 값만으로는 정방향/역방향 감지를 못하므로 두개의 emitters/detectors 단을 구성한다
- 1 데이터가 몇개 나왔는지로 얼마나 회전함을 확인할수있다.
- 한개의 detector단에서 (Rising/Falling) interrupt시에  다른한개의 detector의 위상으로 정.역방향 감지
- 한개의 detector 에서 한개의 interrupt만 사용한다면 1체배 , 두개의 interrupt를 사용한다면 2채배, 두 detector의 모든 interrupt를 사용한다면 4체배가된다
- CPR (Cycles Per Revolution) : 한 회전당 발생하는 신호 주기 수 → 원판에 구멍뚫어놓은 개수
- 엔코더의 분해능은 resolution으로 표현하게되고,  4체배로 사용한다면 CPR*4 가 해당 모터의 분해능이된다.
- 추가적으로 기어드 모터라면, 기어비*resolution이 되기때문에 더 정밀한 각도 측정이 가능하다.


<br></br>

### 절대 엔코더

<center>
  <img src="/upload_image/motor_encoder/absolute_encoder.png" alt="Large example image" width="500"/>
</center>

- 기존 증분형엔코더는 빛을 통과할수있는 hall 개수를 몇개 지나갔느냐로 판단을했다면, 절대엔코더는 각 위치마다 고유의 디지털 코드를 가지고있다.
- 시스템이 재시작된 경우에도 현재 위치를 정확하게 파악 가능하다
- 각 Hall은 이진코드(Gray Code)를 출력한다
- 예를들어, 12비트 엔코더 resolution 4096인 엔코더는 4096개의 각기 다른 값을 나타냄으로써, 이전의정보없이(전원이 재인가되어도) 현재 위치를 바로 파악가능하다.

<br></br>

### 기어드 모터라면 얘기가 다름

- 멀티턴 절대 엔코더 ( Multi-turn Absolute Encoder) 총 몇바퀴도는지 알수있는 엔코더를 의미
- 예를들어 실제 모터가 9번돌아야 링크가 1번도는 기어드모터라면 모터입장에서 내가 지금 몇바퀴를 돌았는지 기억할 필요가있음
- 당연히, 그냥 프로세서측에서 기억하고있으면 되지않겠나하지만 예를들어 로봇팔 같은경우에, 비상시 전원이 꺼졌다 켜진경우에도, 원래상태로 복귀하거나,  하던일을 마저 동작하게하기위해선 단순히 펌웨어단에서 기억하고있는건 좋지않는 방법
- 멀티턴이가능한 절대엔코더라고 소개하는 대부분은 MCU ROM에다가 회전정보를 저장해놓는 경우가 대부분인듯