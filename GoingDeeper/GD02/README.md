# AIFFEL Campus Online Code Peer Review Templete
- 코더 : 이민서
- 리뷰어 : 박희지


# PRT(Peer Review Template)
- [x]  **1. 주어진 문제를 해결하는 완성된 코드가 제출되었나요?**
    1) KITTI 데이터셋 구조와 내용을 파악하고 이를 토대로 필요한 데이터셋 가공을 정상 진행하였다.
       - collect_kitti_stats() 로 학습셋 7,481장 전체를 순회해 총 객체 40,570개의 통계를 수집했다. 이미지를 디코딩하지 않고 dataset.targets[idx] 라벨 텍스트만 직접 파싱해 전체 순회 비용을 낮춘 점이 좋았다.
       - 클래스 분포: Car 28,742(70.85%) ~ Person_sitting 222(0.55%), 최대·최소 클래스 간 130배 불균형을 확인하였다.
       - 이미지 크기가 1242×375 / 1224×370 / 1238×374 / 1241×376 로 일정하지 않다는 사실을 표본 300장에서 확인한 후 3장 detection_collate() 의 패딩 설계 근거로 연결하였다.
      
         <img width="800" height="832" alt="image" src="https://github.com/user-attachments/assets/c19358c2-e56f-4f30-b5db-2c45bd84c596" />
         <img width="1023" height="1094" alt="image" src="https://github.com/user-attachments/assets/970fd0a9-dea6-433f-9ee8-e81877e54d56" />

    2) RetinaNet 학습이 정상 진행되어 object detection 결과 시각화까지 진행되었다.
       - build_retinanet() 에서 COCO 사전학습 RetinaNet의 classification_head 만 KITTI 9클래스(배경 포함)로 교체하고, box regression head는 클래스와 무관하게 4좌표만 예측하므로 그대로 둔다는 판단 근거를 docstring에 명시하였다.
       - 학습이 정상적으로 진행되었다는근거가 수치로 남아있다. 초기 평균 손실 0.6750 → 최종 평균 손실 0.3711, 감소율 45.0%.
       - 검증셋 4장에 대해 클래스별 고정 색상 박스와 클래스명 score 라벨을 그린 이미지를 생성했다.
       - 최종 판정 근거가 된 박스만 진하게(width=4), 나머지는 연하게(width=2) 그리는 visualize_decisions() 로 10장 전체 시각화까지 생성하여 그림만 보고 왜 그 판정이 나왔는지 알 수 있었다.
         
         <img width="1088" height="489" alt="image" src="https://github.com/user-attachments/assets/6a37d5a6-0cff-417b-b1d3-f2112088077e" />
         <img width="1050" height="1049" alt="image" src="https://github.com/user-attachments/assets/96f1463e-30aa-4062-8d83-d8fb3ef926b8" />
         
    3) 자율주행 Object Detection 테스트시스템 적용결과 만족스러운 정확도 성능을 달성하였다.
       | 단계 | 설정 | 점수|
       |---|---|---|
       | 베이스라인 (명세 그대로) | score≥0.50, vehicle≥300px, person_h≥0px | 80점 |
       | 캘리브레이션 후 | score≥0.40, vehicle≥280px, person_h≥50px | 100점 |
       
       - 캐시가 아닌 test_system() 을 통한 end-to-end 재실행에서도 10/10 = 100점이 나오는 것을 확인했다. 캐시 위에서만 100점인 게 아니라는 검증까지 되어 있어 신뢰할 수 있다.
       - 100점은 평가에 쓰는 10장으로 임계값을 격자 탐색해서 얻은 점수이다. 이러한 한계를 아쉬운 점에서 먼저 인정하고 안정성 분석으로 위험을 줄이려 시도한 점이 인상깊었다.



    
- [x]  **2. 전체 코드에서 가장 핵심적이거나 가장 복잡하고 이해하기 어려운 부분에 작성된 
주석 또는 doc string을 보고 해당 코드가 잘 이해되었나요?**
    1) 가장 핵심적인 코드: `decide_go_stop()`
      - 이 프로젝트의 판단 규칙 전체가 이 함수 하나에 담겨 있다. RetinaNet은 물체를 찾아줄 뿐이고, "그래서 멈출 것인가 갈 것인가"를 결정하는 것은 전부 이 함수다. 6장의 캘리브레이션도, 7장의 모델 비교 실험도 이 함수를 고정한 채 입력만 바꿔 돌리는 구조라 프로젝트의 모든 결과가 여기를 통과한다. 코드 자체는 score 내림차순 리스트를 한 번 순회하며 두 조건을 검사하는 짧은 루프로, 읽기 어려운 것은 문법이 아니라 "왜 이 임계값인가" 인데, docstring와 주석이 그 부분을 잘 설명한다.

       <img width="667" height="667" alt="image" src="https://github.com/user-attachments/assets/00d39b8e-4487-49c0-86ac-df4f09458d1c" />

    2) 이해하기 어려운 코드: `detection_collate()`
       - 객체 검출에서 기본 `DataLoader` 가 실패하는 이유(이미지 크기도 다르고 객체 수도 다름)를 이미지/타깃으로 나눠 설명한 뒤, 패딩을 오른쪽·아래에만 넣는 이유는 박스 좌표가 좌상단 기준이라 패딩해도 좌표가 그대로 유효하기 때문임을 밝혀 `F.pad(img, (0, max_w - w, 0, max_h - h))` 한 줄만 봐서는 "왜 왼쪽·위 패딩이 0인가"가 보이지 않는데, 좌표 보정을 피하려는 설계 의도가 한 번에 잡혔다.

        <img width="569" height="261" alt="image" src="https://github.com/user-attachments/assets/25cddc96-75cf-4bad-844f-78fff842a1b6" />
        <img width="1123" height="275" alt="image" src="https://github.com/user-attachments/assets/0f35faef-95a4-41b7-948f-ea5a7ab3f9c3" />

    3) 총평
        주석 스타일이 **"무엇을 하는가"가 아니라 "왜 이렇게 했는가"** 로 일관됩니다. `build_retinanet()` 의 *"box regression head는 클래스와 무관하게 4개 좌표만 예측하므로 그대로 둔다"*, `detect_objects()` 의 *"min_score를 아주 낮게(0.05) 두는 것이 핵심이다 — 이후 어떤 score_threshold를 실험하더라도 모델을 다시 돌릴 필요가 없다"* 같은 주석이 그 예시다. 8-4절에 핵심 코드의 위치와 이유를 표로 정리해 둔 것도 리뷰 시 도움이 됐다.

       <img width="1162" height="306" alt="image" src="https://github.com/user-attachments/assets/f45a0faa-0799-4205-94be-719a75a0dac6" />


        
- [x]  **3. 에러가 난 부분을 디버깅하여 문제를 해결한 기록을 남겼거나
새로운 시도 또는 추가 실험을 수행해봤나요?**
    1) 디버깅 기록
       - 베이스라인 80점의 실패 2건을 표로 표시하여 원인을 특정했다.
         | 파일 | 정답 | 예측 | 원인 |
         |---|---|---|---|
         | go_3.png | Go | Stop | person h=29px score=0.59 — 수십 미터 밖 횡단보도 보행자 |
         | stop_3.png | Stop | Go | 좌측 근접 차량이 282px — 화면 밖으로 잘려 300px 기준에 18px 미달 |

         그리고 요약표로 두 그룹이 어디서 갈리는지 보였다. Stop 이미지의 사람은 164px·157px, go_3의 사람은 29px로 5배 이상 차이, Go 이미지의 최대 차량은 go_5의 212px. 여기서 **"핵심은 사람의 유무가 아니라 사람까지의 거리이고, 차량에는 300px 기준이 있는데 사람에는 없다는 것이 명세의 빈틈"** 이라는 결론을 도출한 흐름이 설득력 있다.

         <img width="710" height="1119" alt="image" src="https://github.com/user-attachments/assets/b0956e92-04be-412d-9fa1-28932bcb398b" />

    2) 추가 실험
       | 실험 | 질문 | 결과|
       |---|---|---|
       | A. 안정성 분석 | 임계값이 10장에 과적합인가? | 100점 조합 280개, person_h 0~150px / vehicle 220~280px 범위 → 고원 확인 |
       | B. ROI 방식 | 크기 대신 위치로 판단하면? | 90점. 사다리꼴 주행영역 밖의 원거리 보행자 문제는 해결했으나 stop_3 좌측 가장자리 차량을 놓침 |
       | C. 모델 비교 | KITTI 파인튜닝 vs COCO 사전학습 | 둘 다 100점. 5장 중 3장에서 파인튜닝 모델 신뢰도가 앞섬 |
       | D. 학습량 비교 | 300스텝에서 멈춘 첫 판단이 옳았나? | 점수는 동일(100점)이나 평균 신뢰도 0.768 → 0.926, 평균 검출수 200.8 → 98.4 |

       <img width="1026" height="1135" alt="image" src="https://github.com/user-attachments/assets/319f825f-4e3e-47cb-9efa-143b5b9638c9" />
       <img width="1185" height="402" alt="image" src="https://github.com/user-attachments/assets/f9e4a7a7-8842-468b-b70b-82e79a9fb090" />


        
- [x]  **4. 회고를 잘 작성했나요?**
    - 8장에 배운 점 4개 / 아쉬운 점 3개 / 느낀 점이 모두 있고, 각 항목이 노트북 안의 구체적 수치와 절 번호에 연결되어 있다.
      
      <img width="1185" height="1101" alt="image" src="https://github.com/user-attachments/assets/20f12d48-c1f8-48c5-b440-90fe7584bc72" />

    - `draw_pipeline_flowchart()` 로 matplotlib 도식을 직접 그렸다. KITTI 학습 흐름(왼쪽)과 Go/Stop 판단 흐름(오른쪽)을 분리하고, 각 열 상단에 대응 루브릭을 표기했으며, decide_go_stop ↔ calibrate 사이의 임계값 되먹임 루프를 점선 화살표로 따로 표시하였다. 데이터 흐름 요약도 함께 있어 이해하기 쉬웠다.

      <img width="1161" height="722" alt="image" src="https://github.com/user-attachments/assets/3cf37412-4a6c-43be-b85c-d0bf6e0cf3d0" />


        
- [x]  **5. 코드가 간결하고 효율적인가요?**
    1) 중복 제거와 재사용
        - `evaluate_config()` — 베이스라인 평가 / 캘리브레이션 격자 탐색 / 실험 C 모델 비교 **세 곳에서 재사용**
        - `build_detection_cache(model=, id_to_name=)` — COCO / KITTI 3에폭 / 300스텝 **세 모델에 그대로 재사용**
        - `show_detection_grid()` — 4장 KITTI 시각화와 6-5절 판정 시각화가 공유
        - `train_one_epoch(max_steps=)` — 3에폭 학습과 실험 D의 300스텝 재현을 한 함수로 처리
      
    2) PEP8
        - 네이밍(`snake_case` / `PascalCase` / `UPPER_SNAKE_CASE`)이 일관되고, **코드 셀 전체 1,564줄 중 100자를 넘는 줄이 0건**이다. Google 스타일 docstring(Args / Returns / Note)도 공개 함수 전반에 통일되어 있다. 8-5절에 지킨 규칙을 목록으로 정리해 둔 것도 좋았다.

          <img width="770" height="533" alt="image" src="https://github.com/user-attachments/assets/c7b3484b-9a1f-46a9-941c-d84d1bda8669" />

      
    3) 매직 넘버 제거
        - `DecisionConfig` 를 `@dataclass(frozen=True)` 로 만들어 모든 임계값을 한 곳에 모으고 함수 인자로 주입합니다. `frozen=True` 로 실험 중 값 변경을 막고, `describe()` 로 설정을 문자열화해 실험 결과와 함께 기록하는 구성이 깔끔합니다. 임계값이 코드 여기저기 박혀 있었다면 6장의 격자 탐색 자체가 불가능했을 겁니다.
      
          <img width="1096" height="655" alt="image" src="https://github.com/user-attachments/assets/38967f3d-923a-4f65-bf9d-eb4c4b7dd511" />



# 회고(참고 링크 및 코드 개선)
```
주어진 명세를 그대로 받아들이지 않고 데이터셋을 정량적으로 분석해 검증한 점, 그리고 실패한 실험을 지우지 않고 한계와 함께 남긴 점이 좋았습니다. 분석에서 도출한 근거가 설계로 이어지는 흐름이 체계적이었습니다. 배울 점이 많은 코드였습니다.
```
