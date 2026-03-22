---
title: SolidWorks Quick Start
sidebar_position: 1
---

# SolidWorks Workflow Quick Start Guide

이 문서는 SolidWorks를 처음 사용하는 팀원이 프로그램 실행부터 환경 설정, 2D 스케치 작성, 3D 모델 변환까지의 기본 워크플로우를 따라갈 수 있도록 만든 퀵 스타트 가이드이다.  

이 문서의 목표는 복잡한 이론 설명이 아니라, 사용자가 **무엇을 클릭하고, 무엇을 선택하며, 왜 그렇게 하는지**를 단계적으로 이해하고 따라할 수 있도록 하는 것이다.

---

# 1. How to Start

## 1.1 SolidWorks 시작하기

SolidWorks를 사용할 때 가장 먼저 해야 할 것은 모델링 전에 **작업 환경을 표준화하는 것**이다.  

단위, 화면 방향, 스케치 시작 위치 등의 기본 설정이 제대로 되어 있지 않으면, 이후 치수 오류나 방향 오류, 조립 문제 등이 발생할 수 있다.

---

## 1.2 새 파일 생성하기

1. SolidWorks 실행  
2. 왼쪽 상단의 **File** 클릭  
3. **New** 선택  
4. 새 문서 창에서 아래 중 하나 선택  

   - **Part**: 단일 부품 생성  
   - **Assembly**: 여러 부품 조립  
   - **Drawing**: 도면 작성  

초보자는 **Part**부터 시작하는 것이 권장된다.

---

## 1.3 단위 설정

모델링 전에 반드시 **단위(Unit)**를 확인해야 한다.  

기계 부품이나 로봇암 프로젝트에서는 보통 **MMGS**를 사용한다:

- **M**illimeter (밀리미터)  
- **G**ram (그램)  
- **S**econd (초)  

### 설정 방법

1. 화면 오른쪽 아래 단위 표시 확인  
2. inch 또는 meter로 되어 있다면 클릭  
3. **MMGS (millimeter, gram, second)** 선택  

### 중요한 이유

단위를 잘못 설정하면 심각한 오류가 발생할 수 있다.  
예: 50mm 부품 → 50inch로 생성되는 문제

---

## 1.4 파일 저장

작업 시작 전에 파일을 먼저 저장하는 것이 중요하다.

### 저장 방법

1. **File → Save As**  
2. 프로젝트 폴더 선택  
3. 파일 이름 입력 후 저장  

### 파일 이름 예시

- `base_plate_v1`  
- `joint_arm_A_v2`  
- `motor_mount_test`  

### 팁

모호한 이름은 피한다:

- `part1` ❌  
- `newnewfinal` ❌  
- `arm_link_01` ⭕

---

## 1.5 인터페이스 이해

SolidWorks 화면은 다음과 같은 주요 영역으로 구성된다.

### (1) FeatureManager Design Tree  
왼쪽 패널에 위치하며, 스케치, 돌출, 컷, 필렛 등의 작업 히스토리를 보여준다.

### (2) Graphics Area  
중앙 작업 공간으로, 모델을 생성하고 확인하는 영역이다.

### (3) CommandManager  
상단 메뉴 탭 영역:

- Features  
- Sketch  
- Evaluate  

### (4) Heads-Up View Toolbar  
그래픽 영역 상단에 위치하며, 확대, 회전, 뷰 변경 등을 담당한다.

---

## 1.6 기본 기준면 이해

SolidWorks에는 세 가지 기본 기준면이 있다:

- **Front Plane**  
- **Top Plane**  
- **Right Plane**  

이 기준면은 스케치를 시작하는 위치를 결정한다.

### 선택 기준

- 위에서 본 형상 → **Top Plane**  
- 정면 기준 → **Front Plane**  
- 측면 기준 → **Right Plane**  

초보자는 **Top Plane 또는 Front Plane**을 주로 사용한다.

---

## 1.7 스케치 시작하기

1. FeatureManager에서 Plane 선택  
   (예: Top Plane)  
2. 마우스 오른쪽 클릭  
3. **Sketch** 선택  

이제 2D 스케치 모드로 진입한다.

### 확인 사항

- 화면이 평면 기준으로 정렬됨  
- 상단 메뉴가 Sketch 도구로 변경됨  

---

## 1.8 화면 정렬 (Normal To)

스케치를 할 때는 반드시 **정면(수직 방향)으로 화면을 맞춰야 한다.**

### 사용 방법

1. Plane 또는 Sketch 선택  
2. **Normal To** 클릭  
   또는  
3. **Ctrl + 8**  

### 이유

화면이 기울어져 있으면 정확한 스케치와 치수 입력이 어렵다.

---

# 2. How to Draw 2D

## 2.1 2D 스케치란?

2D 스케치는 3D 모델의 기초가 되는 평면 도형이다.  

예:
- 원 → 원기둥  
- 사각형 → 블록  

---

## 2.2 Top Plane에서 스케치 시작

예: 원형 베이스 만들기

### 단계

1. **Top Plane 선택**  
2. 오른쪽 클릭 → Sketch  
3. **Normal To** 적용  
4. 스케치 시작  

---

## 2.3 Line 그리기

### 위치

Sketch → Line

### 단계

1. Line 클릭  
2. 시작점 클릭  
3. 마우스 이동  
4. 끝점 클릭  
5. Enter 또는 Esc로 종료  

### 용도

외곽선, 구조선 등 기본 형상 생성

---

## 2.4 Circle 그리기

### 위치

Sketch → Circle

### 단계

1. Circle 클릭  
2. 중심점 클릭  
3. 바깥으로 드래그  
4. 클릭하여 완료  

### 팁

이후 Smart Dimension으로 정확한 크기 지정

---

## 2.5 Rectangle 그리기

### 단계

1. Rectangle 클릭  
2. 첫 꼭짓점 클릭  
3. 반대 꼭짓점 클릭  
4. 치수 입력  

### 옵션

Center Rectangle → 중심 기준 생성

---

## 2.6 Centerline 사용

Centerline은 실제 형상이 아닌 **기준선**이다.

### 단계

1. Centerline 클릭  
2. 일반 선처럼 작성  

### 용도

- 중심 정렬  
- 대칭 구조  
- 회전 축  

---

## 2.7 Smart Dimension 적용

### 단계

1. Smart Dimension 클릭  
2. 요소 선택  
3. 위치 클릭  
4. 값 입력  

### 예시

- 선 길이: 100mm  
- 원 지름: 40mm  

### 중요성

치수 없이는 정확한 설계가 불가능하다.

---

## 2.8 Sketch Relations

자동 또는 수동으로 관계 설정:

- Horizontal  
- Vertical  
- Coincident  
- Midpoint  
- Tangent  

스케치 안정성을 높인다.

---

## 2.9 Fully Defined 상태

- 검정색: 완전 정의  
- 파란색: 미완 정의  

### 방법

- 치수 추가  
- 관계 추가  

### 중요성

미완 정의 상태는 형상이 변형될 수 있다.

---

## 2.10 Trim 사용

### 단계

1. Trim Entities 클릭  
2. 불필요한 부분 제거  

---

## 2.11 Offset Entities

기존 선과 평행한 새로운 선 생성

### 용도

- 두께 생성  
- 내부/외부 경계  

---

## 2.12 홀 위치 설정

### 단계

1. 기본 형상 생성  
2. 기준 치수 설정  
3. 원 생성  
4. 위치 및 크기 정의  

---

# 3. Detail Sketch Functions

## 3.1 홀 스케치 생성

1. 기본 형상 작성  
2. 위치 기준 설정  
3. 원 생성  
4. 치수 입력  

---

## 3.2 기준선 활용

복잡한 형상에서는 기준선을 먼저 생성:

- 중심선  
- 대칭선  
- 중간점 기준  

---

## 3.3 대칭 구조 생성

1. 중심선 작성  
2. 반쪽 형상 작성  
3. Mirror 적용  

---

## 3.4 슬롯 및 고급 형상

사용 가능한 도구:

- Slot  
- Arc  
- Spline  
- Chamfer (스케치)

---

# 4. How to Draw 3D

## 4.1 2D → 3D 변환

스케치 완료 후 3D로 변환한다.  
가장 기본 기능은 **Extruded Boss/Base**이다.

---

## 4.2 돌출 (Extrude)

### 단계

1. 닫힌 스케치 선택  
2. Extruded Boss/Base 클릭  
3. 높이 입력  
4. 확인  

---

## 4.3 컷 (Extruded Cut)

### 단계

1. 면 선택  
2. 스케치 생성  
3. 형상 작성  
4. Extruded Cut  
5. 깊이 설정 또는 Through All  
6. 확인  

---

## 4.4 Hole Wizard

### 용도

- 볼트 구멍  
- 카운터보어  
- 탭  

### 단계

1. Hole Wizard 선택  
2. 타입 설정  
3. 크기 설정  
4. 위치 지정  
5. 확인  

---

## 4.5 Fillet

모서리를 둥글게 처리

### 단계

1. Fillet 선택  
2. 모서리 선택  
3. 반지름 입력  
4. 확인  

---

## 4.6 Chamfer

모서리를 각지게 처리

### 단계

1. Chamfer 선택  
2. 모서리 선택  
3. 거리 또는 각도 입력  
4. 확인  

---

## 4.7 Revolve

### 단계

1. 단면 스케치  
2. 중심선 생성  
3. Revolve 실행  
4. 축 선택  
5. 각도 입력  

---

## 4.8 화면 조작

- 마우스 휠: 확대/축소  
- 드래그: 회전  
- Shift + 드래그: 이동  

---

## 4.9 자주 하는 실수

1. 스케치가 닫혀 있지 않음 → 돌출 불가  
2. 단위 오류 → 스케일 문제  
3. Fully Defined 아님 → 불안정  
4. Plane 선택 오류 → 방향 문제  
5. 저장 안 함 → 데이터 손실  