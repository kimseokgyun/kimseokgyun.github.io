---
layout: post
title: Nav2_Costmap2d
date: 2024-02-14 20:32 +0900
last_modified_at: 2024-02-14 20:32 +0900
tags: [Robot ,ROS2]
toc:  true
---
## Conclusion
ROS2 NAV2 패키지에서 Costmap2d 패키지 아키텍처에대한 설명
...2024-06-23 작성진행중


## Meaning
- CostMap이란 주변 환경정보를 효과적으로 표현하고 경로 계획에 활용하기 위한 개념
- 센서 데이터를 기반으로 장애물 표현하기 수월
- Navigation 알고리즘에서 이동 비용 할당하기에 수월

<!-- ![placeholder](http://placehold.it/800x400 "Large example image") -->

## Concept
![placeholder](/upload_image/nav2_costmap/costmap1.png "Large example image")


- Nav2 Costmap은 Grid Map 기반으로 각 셀 0~254값 (1byte)로 할당된 Map을 관리하는 객체
    - Nav2 기본 아키텍처는 global_costmap, local_costmap 이름을 가진 Costmap2D 객체를 관리한다.
        - global Costmap은 planner_server에서 객체 생성,관리
        - local Costmap은 controller_server에서 객체 생성,관리
    - 한개의 Costmap은 Plugin별로 각 Layer를 관리 
        - obstacle_layerm, voxel_layer,inflation_layer ...etc
        - 각 plugin에서의 입력값 (Sensor data / map data) 를 받아서 raytrace,marking,clearing과 같은 API정의
    - 최종 Costmap 객체인 Layered_costmap은 Thread 실행에서 자원 관리차 2개의 객체로 관리된다
        - primary_costmap : 기존 Plugin별로 관리된 costmap 객체
        - combined_costmap : primary_costmap에서 filter 알고리즘이 적용된 costmap 





### 구현

...2024-06-23 작성진행중