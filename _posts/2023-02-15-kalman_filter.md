---
layout: post
title: 선형칼만필터
date: 2023-02-15 20:32 +0900
last_modified_at: 2023-02-15 20:32 +0900
tags: [robot , filter]
toc:  true
---


## 역사

- 루돌프 E.칼만
- 항공 우주산업에서 유도 항법 시스템 정확도를 높이기위해 개발

- EKF
    - 선형화 칼만필터와 비슷한개념 But EKF는 선형칼만필터
    - 알고리즘이 발산할 위험 (대부분의 원인은 선형화에서발생)
    
- UKF
    - UT 알고리즘
    - 시그마포인트
    
- Particle Filter
    - Particle init
    - Particle Prediction
    - Update Particle Weight
    - Resampling
    

---

### 사용사례

object Tracking using Kalmanfilter :https://www.youtube.com/watch?v=V8ElWjlD-n8

https://www.youtube.com/watch?v=MxwVwCuBEDA

데이터 Hz상승용

센서퓨전

---

## **칼만필터의 특징**

- 선형모델 : 선형 시스템 모델에 기반한 데이터 추정
- 비선형시스템까지 확장 가능
- 재귀모델: 상태 추정을 위해 이전 시간 단계 정보만 이용하여 현재 상태 추정
- 가우시안 분포 모델을 이용 : 많은 실세계 시스템에서의 잡음은 가우시안 분포를 따르는 경향
    - 가우시안 분포 모델을 사용할때의 이점
        - 수학적 편리
        - 중심 극한 정리 : 독립확률변수 n이 표본이 충분히 커진다면, 표본평균의 분포는 정규분포를 따른다
        

## 정립된 칼만필터를 이용할 개발자 관점

- 시스템 모델 확립
    - 실제로 가장 어려운작업 , 또한 칼만필터의 성능이 좌지우지

---

![KakaoTalk_20240207_153816919.jpg](https://prod-files-secure.s3.us-west-2.amazonaws.com/5d057f3b-7018-4c7f-8dc5-82288f5780ba/bba165fc-b534-493c-a352-46100e76221d/KakaoTalk_20240207_153816919.jpg)

칼만필터

$$
\hat{x_k}=\hat{x_k^-} + K_k(z_k -H\hat{x_k^-})
$$

1차 LowPass Filter

$$
\hat{x_k} = \alpha\hat{x_{k-1}} + (1-\alpha) *x_k
$$

---

## 칼만필터 시스템

$$
x_{k+1} = Ax_k + w_k

$$

$$
x_k :상태변수 \\z_k : 측정값 \ (센서 \ 측정값)\\ A :  동적  \ 모델 \ (Dynamic\ Model)\\ H: 측정모델 \ (Measurement \ Model) \\ w_k : 프로세스\ 노이즈 \\v_k : 측정값 \ 노이즈
$$

### 상태변수

<aside>
✏️ 거리,속도,가속도,위치,무게 등 칼만필터를 통해 결과적으로 정제하거나, 얻고싶은 변수

</aside>

> Example1
> 

$$
X = \begin{bmatrix} P_x \\ V_x
  \end{bmatrix}
$$

> Example2
> 

$$
X=\begin{bmatrix} roll\\ pitch \\yaw
  \end{bmatrix}
$$

### 시스템모델 or 예측모델 (A or F)

<aside>
✏️ 동적모델을 시스템모델이라고 설명하는곳도있고, Prediction Matrix 라고 설명하는 곳도 있음

변수의 상관관계를 이용해서 시스템모델을 작성하면 Prediction Matrix 라고함

시스템모델은, 선형모델이여야만함

</aside>

> Exmaple1
> 

$$
\begin{bmatrix} P_x \\ V_x
  \end{bmatrix} = \begin{bmatrix} 1&dt \\ 0 &1  \end{bmatrix}\begin{bmatrix}  P_{x-1} \\ V_{x-1}\end{bmatrix}
$$

> Example2
> 

$$
\begin{bmatrix} x \\ y \\z \\w
  \end{bmatrix} = I+ dt*\frac{1}{2}\begin{bmatrix} 0&-p&-q&r \\ p &0&r&-q \\q&-r&0&p\\r&q&-p&0 \end{bmatrix}\begin{bmatrix}  P_{x-1} \\ V_{x-1}\end{bmatrix}
$$

> Example3
> 

$$
\begin{bmatrix} x\\ y \\ z \\roll\\pitch\\yaw\\vx\\vy\\vz\\vroll\\vpitch\\vyaw\\ax\\ay\\az
  \end{bmatrix}
$$

### 입력모델 (B)

Control Matrix 도 존재할수있다 [B]

$$
x_{k+1} = Ax_k + Bu_k + w_k
$$

> Example1
> 

$$
u_k = a_x 
$$

$$
A = \begin{bmatrix} 1&dt \\ 0 &1  \end{bmatrix}
$$

$$
B= \begin{bmatrix} \frac{dt^2}{2}\\ dt  \end{bmatrix}
$$

$$
\begin{bmatrix} P_x \\ V_x
  \end{bmatrix} = \begin{bmatrix} 1&dt \\ 0 &1  \end{bmatrix}\begin{bmatrix}  P_{x-1} \\ V_{x-1}\end{bmatrix} + \begin{bmatrix}\frac{dt^2}{2}\\ dt \end{bmatrix}u_k
$$

### 관측모델 (H)

<aside>
✏️ 내가 선정한 상태변수와 , 내가 사용하는 센서 측정값이 일치하지 않을수있다.

Kalman Gain을 계산하기위해서 , 내가 선정한 상태변수 예측값과 측정값을 비교할수있도록 선정해야한다.

</aside>

> example1
> 

$$
z_k = [v_x]
$$

$$
\begin{bmatrix} p_x & v_x
  \end{bmatrix}
$$

$$
H = \begin{bmatrix} 0 \\ 1
  \end{bmatrix}
$$

$$
\hat{x_k}=\hat{x_k^-} + K_k(z_k -H\hat{x_k^-})
$$

$$
[v_{zk}]-\begin{bmatrix} 0 \\ 1
  \end{bmatrix}  \begin{bmatrix} \hat{p_x} & \hat{v_x}
  \end{bmatrix}
$$

> example2
> 

실제론 속도값만 센싱되지만, 관측값을 k-1시점의 위치값도 사용하고싶을때

위와같은 시스템행렬[A]일때는 관측값과 예측값이 같아서 의미가 없겠지만,

[A]가 수시로 변하는 복잡한 형태일때는 유의미함.

$$
z_k = \begin{bmatrix}p_{k-1}\\  v_{zk} \end{bmatrix}
$$

$$
\begin{bmatrix} p_x \\ v_x
  \end{bmatrix}
$$

$$
H= \begin{bmatrix} 1 &  0 \\ 0 &1
  \end{bmatrix}
$$

$$
\hat{x_k}=\hat{x_k^-} + K_k(z_k -H\hat{x_k^-})
$$

$$
\begin{bmatrix}p_{zk}\\  v_{zk} \end{bmatrix}-\begin{bmatrix} 1 & 0 \\ 0 & 1
  \end{bmatrix}  \begin{bmatrix} \hat{p_x} \\ \hat{v_x}
  \end{bmatrix}
$$

> example3  - Failed  —> 이러한 문제점을 해결하기위해 EKF
> 

$$
z_k = \begin{bmatrix}x\\ y\\z\\w \end{bmatrix}
$$

$$
\begin{bmatrix} roll\\ pitch \\yaw
  \end{bmatrix}
$$

$$
H= \begin{bmatrix} ?\end{bmatrix}
$$

$$
\text{Yaw} (\psi) = \text{atan2}\left(2(xw + yz), 1 - 2(x^2 + y^2)\right)
$$

$$
\text{Pitch} (\theta) = \text{asin}\left(2(xy - zw)\right)
$$

$$
\text{Roll} (\phi) = \text{atan2}\left(2(xz + yw), 1 - 2(y^2 + z^2)\right)
$$

관측값은 쿼터니안으로 들어오지만 , 상태변수는 오일러각으로 되어있다.

H행렬을 맞춰줄수없음

But → 야코비안으로는 해결가능 ( EKF) 

이런 변수의 상관관계를 이용하는 칼만필터 특성때문에, 측정데이터와다른 차원의 변수도 예측가능—> 그래서 센서퓨전에 많이 사용

---

## 칼만필터 진행순서

### 1. 초기값

$$
초기값 \ \hat{x_0}  ,  P_0 
$$

### 2. 예측

- 예측모델을 통한 추정값과 **오차 공분산 예측** ( - 붙어있는건 사전예측값 : 측정되기전)

$$
\hat{x_k^-} = A\hat{x}_{k-1} 
$$

$$
P_k^- = AP_{k-1}A^T +Q
$$

사전 공분산 계산

k-1단계에서의 공분산을 A(시스템모델)로 어느정도 바뀔지 예상 

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/5d057f3b-7018-4c7f-8dc5-82288f5780ba/c327ad32-9337-4267-b16d-791f2f94da46/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/5d057f3b-7018-4c7f-8dc5-82288f5780ba/12c44ada-ae20-4df4-88d5-331fff68308f/Untitled.png)

- 분산 계산식 정의
    
    $$
    오차의정의 \ e = Value_{true} - Value_{expected}\\변수 x에대한 \ 분산 =E[e\ {e}^T]= P(x)\\분산 = E[오차^2]
    $$
    
    $$
    P_k^-=E[(x_k -\hat{x}_k^-)(x_k -\hat{x}_k^-)^T] \\ \because x_k-\hat{x}^-_k = A(x_{k-1} - \hat{x}_{k-1}) + w_k \\=E[A(x_{k-1}-\hat{x}_{k-1})(x_{k-1}-\hat{x}_{k-1})^TA^T+w_{k-1}w_{k-1}^T]\\\because w(process \ noise) 와 \ x_{n}와는 \ indepedent \\ P_k^- = AP_{k-1}A^T+Q_{k-1}
    $$
    
    $$
    x_k = Ax_{k-1}+w_k \\ \hat{x}_k^- =A\hat{k}_{k-1}
    $$
    

### 3. 관찰

- 칼만 이득 계산

$$
K_k = P_k^- H^T(HP_k^-H^T+R)^{-1}
$$

Observation 모델의 분산과 Prediction 모델의 분산을통해 K (칼만게인) 을 구한다.

- **추정값 계산 (사후측정값)**

$$
\hat{x_k}=\hat{x_k^-} + K_k(z_k -H\hat{x_k^-})
$$

관측값과 사전 예값중에 어느것을 더 가중치를 둘지에 대해 **Adaptive Gain 이 K**

$$
z_k = \begin{bmatrix}p_{k-1}\\  v_{zk} \end{bmatrix}
$$

$$
\begin{bmatrix} p_x \\ v_x
  \end{bmatrix}
$$

$$
H= \begin{bmatrix} 1 &  0 \\ 0 &1
  \end{bmatrix}
$$

$$
\hat{x_k}=\hat{x_k^-} + K_k(z_k -H\hat{x_k^-})
$$

$$
\begin{bmatrix}p_{zk}\\  v_{zk} \end{bmatrix}-\begin{bmatrix} 1 & 0 \\ 0 & 1
  \end{bmatrix}  \begin{bmatrix} \hat{p_x} \\ \hat{v_x}
  \end{bmatrix}
$$

$$
\hat{x_k} = \hat{x_k^-}+ K*e
$$

- **오차 공분산 계산**

$$
P_k = P_k^- -K_kHP_K^-
$$

---

### Kalman_animation

- Modeling
    - velocity = 1.0 m ,
    - std_dev = 0.1 ( 실제 속도 측정 센서 노이즈)
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/5d057f3b-7018-4c7f-8dc5-82288f5780ba/802ab798-89d9-45ff-bbca-d55ff222f029/Untitled.png)
    
    - 초기 공분산
    
    $$
    P_0 = \begin{bmatrix} 10 &0 \\0 & 10
      \end{bmatrix}
    $$
    
    - Q : process-Noise ( Prediction -Noise)
    
    $$
    Q = \begin{bmatrix} 1 &0 \\0 & 1
      \end{bmatrix}
    $$
    
    - R : Observation-Noise ( Sensor`s covariance )
    
    $$
    R = \begin{bmatrix} 10 &0 \\0 & 10
      \end{bmatrix}
    $$
    
    - 시스템 모델
    
    $$
    A =  \begin{bmatrix} 1&dt \\0 & 1
      \end{bmatrix}
    $$
    
    - 상태 변수
    
    $$
    X = \begin{bmatrix} P_x \\ V_x
      \end{bmatrix}
    $$
    
    $$
    V_0 = 1\ m/s \\P_0 = 0 \ m
    $$
    
    $$
     
    $$
    
    Velocity 의 Kalman Gain 은 빠르게 수렴하는모습 
    
    예측모델가중치가 더 크게 들어가게끔 Gain 수렴
    
    공분산을 줄이거나,늘리는 형태로 수렴을 얼마나 빠르게하나, 발산하냐의 차이로 이해하면 편할듯
    
     
    
    True 값에 잘 따라가지못함.
    
    R = 관측 노이즈가 너무큼
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/5d057f3b-7018-4c7f-8dc5-82288f5780ba/3ed25851-7ae0-4478-b921-5377e310731a/Untitled.png)
    
    - 초기 공분산
    
    $$
    P_0 = \begin{bmatrix} 10 &0 \\0 & 10
      \end{bmatrix}
    $$
    
    - Q : process-Noise ( Prediction -Noise)
    
    $$
    Q = \begin{bmatrix} 0.5 &0 \\0 & 0.5
      \end{bmatrix}
    $$
    
    - R : Observation-Noise ( Sensor`s covariance )
    
    $$
    R = \begin{bmatrix} 1 &0 \\0 & 1
      \end{bmatrix}
    $$
    
    - 상태 변수
    
    $$
    X = \begin{bmatrix} P_x \\ V_x
      \end{bmatrix}
    $$
    
    $$
    V_0 = 1\ m/s \\P_0 = 0 \ m
    $$
    
    Obsercvation Covariance ( 관측 공분산) 을 줄이고 , Preediction Covariance 도 튜닝하여,
    
    관측모델과, 측정모델의 가중치가 각각 0.5 수준으로 Kalman filter 를 수렴시킨모습 
    
    ---
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/5d057f3b-7018-4c7f-8dc5-82288f5780ba/5f772d98-240b-4fb4-bfe1-a00372fcf8b7/Untitled.png)
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/5d057f3b-7018-4c7f-8dc5-82288f5780ba/6d59dae4-93b2-4a71-8891-d6596a47b45b/Untitled.png)
    
    - 초기 공분산
    
    $$
    P_0 = \begin{bmatrix} 10 &0 \\0 & 10
      \end{bmatrix}
    $$
    
    - Q : process-Noise ( Prediction -Noise)
    
    $$
    Q = \begin{bmatrix} \sigma^2dt^2 &\sigma^2*dt \\\sigma^2*dt & \sigma^2
      \end{bmatrix}
    $$
    
    - R : Observation-Noise ( Sensor`s covariance )
    
    $$
    R = \begin{bmatrix} 1 &0 \\0 & 1
      \end{bmatrix}
    $$
    
    - 상태 변수
    
    $$
    X = \begin{bmatrix} P_x \\ V_x
      \end{bmatrix}
    $$
    
    $$
    V_0 = 1\ m/s \\P_0 = 0 \ m
    $$
    
    속도와 위치의 관계식이 생겨서 , Position의 확률밀도가 변한 모습을 볼수있음
    
    https://www.youtube.com/watch?v=PVCGDoTZHaI
    
    ---
    
    ### Why we use EKF ?
    
    1.필터는 결국 이산시스템이라는 문제점
    
    $$
    \begin{bmatrix} P_x \\ V_x
      \end{bmatrix} = \begin{bmatrix} 1&dt \\ 0 &1  \end{bmatrix}\begin{bmatrix}  P_{x-1} \\ V_{x-1}\end{bmatrix}
    $$
    
    dt동안에 V_{x-1} 로 일정하게 움직였다고 가정을하고 P_x를 예측함.
    
    2.선형화 시키기 어려운 시스템 모델
    

$$
P_x = V_{x-1}*dt + P_{x-1}
$$

$$
\begin{bmatrix} P_x \\ V_x
  \end{bmatrix} = \begin{bmatrix} 1&dt \\ 0 &1  \end{bmatrix}\begin{bmatrix}  P_{x-1} \\ V_{x-1}\end{bmatrix}
$$

여기서, 위치와 속도로 상태변수를 두었을때는,

(위치의미분)속도*dt + 전의 위치 = 예측위치

로 표현이 가능하다. 그이유는 상태변수에 p의 미분형태인 속도 v가 존재하기때문.

하지만 아래와같은 경우는 roll pitch yaw는 , 

선형화 (roll의속도) *dt 로 나타낼수있는 변수가 상태변수에 존재하지않는다.

이럴때 EKF를 사용해야한다.

$$
\begin{bmatrix} roll\\ pitch \\ yaw
  \end{bmatrix} = [I+\begin{bmatrix} f(p,q,r,dt)\\ g(p,q,r,dt)\\ h(p,q,r,dt)
  \end{bmatrix}] \begin{bmatrix} roll\\ pitch \\ yaw
  \end{bmatrix}
$$

$$
\hat{\begin{bmatrix} roll\\ pitch \\ yaw
  \end{bmatrix}} = \begin{bmatrix} p+qsin(\psi)tan(\theta)+rcos(\psi)tan(\theta) \\ qcos(\psi)-rsin(\psi)\\qsin(\psi)sec(\theta)+rcos(\psi)sec(\theta)
  \end{bmatrix}
$$

$$
\hat{\begin{bmatrix} roll\\ pitch \\ yaw
  \end{bmatrix}} = \begin{bmatrix} f1 \\f2\\f3
  \end{bmatrix}
$$

$$
\begin{bmatrix} roll\\ pitch \\ yaw
  \end{bmatrix}=[I+\begin{bmatrix}  \frac{\partial f_1}{\partial \phi} & \frac{\partial f_1}{\partial \theta} &\frac{\partial f_1}{\partial \psi}

\\ \frac{\partial f_2}{\partial \phi} & \frac{\partial f_2}{\partial \theta} &\frac{\partial f_2}{\partial \psi}

\\\frac{\partial f_3}{\partial \phi} & \frac{\partial f_3}{\partial \theta} &\frac{\partial f_3}{\partial \psi}

  \end{bmatrix} *dt] \begin{bmatrix} roll\\ pitch \\ yaw
  \end{bmatrix}
$$

—> but 해당 모델과같은경우 상태모델을 쿼터니안으로 변경하여 선형화하여 사용가능

1. 사용하기 편함

$$
\begin{bmatrix} x\\ y \\ z \\roll\\pitch\\yaw\\vx\\vy\\vz\\vroll\\vpitch\\vyaw\\ax\\ay\\az
  \end{bmatrix}
$$

$$
\dot{x} = v_x*cos(yaw) - v_y*sin(yaw)=f1
$$

$$
\dot{y} = v_x*sin(yaw) + v_y*cos(yaw)=f2
$$

$$
\begin{bmatrix} x\\ y
  \end{bmatrix}=[I+\begin{bmatrix}  \frac{\partial f_1}{\partial x} & \frac{\partial f_1}{\partial y}

\\ \frac{\partial f_2}{\partial x} & \frac{\partial f_2}{\partial y}

  \end{bmatrix} *dt] \begin{bmatrix} x\\ y 
  \end{bmatrix}
$$

<aside>
✏️ 야코비안?

</aside>

$$
(x,y)  \rightarrow (r,\theta)\\J = \begin{vmatrix}\frac{\partial X}{\partial r} & \frac{\partial X}{\partial \theta}

\\ \frac{\partial Y}{\partial r} & \frac{\partial Y}{\partial \theta} \end{vmatrix} = \begin{vmatrix} cos\theta & -rsin\theta \\sin\theta & rcos\theta\end{vmatrix}= r
$$