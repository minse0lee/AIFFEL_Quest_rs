# AIFFEL Campus Online Code Peer Review Template

- 코더 : 이민서
- 리뷰어 : 강지수

# PRT(Peer Review Template)

- [x] **1. 주어진 문제를 해결하는 완성된 코드가 제출되었나요?**
    - Stanford Dogs 데이터셋을 이용해 ResNet50 + GAP + Dense 구조를 구성하고,
      CAM과 Grad-CAM을 모두 구현한 뒤 Bounding Box와 IoU까지 계산하여
      프로젝트에서 요구한 전체 흐름이 치밀하게 진행됩니다. 
    - 5 epoch 학습 후 top-1 accuracy 0.8816, top-5 accuracy 0.9893을 기록했고,
      200개 테스트 샘플에 대해 CAM과 Grad-CAM의 localization 성능을
      동일 조건에서 통제 하에 잘 비교했습니다.
    - 특히 단일 이미지 시각화에 그치지 않고 mean IoU, median IoU,
      GT-Known Localization, Top-1 Localization까지 정량화한 점이 좋았습니다.

<img width="731" height="473" alt="스크린샷 2026-08-28 오후 5 13 06" src="https://github.com/user-attachments/assets/d30424bc-5984-4fe8-ac28-14c7490512b5" />


- [x] **2. 전체 코드에서 가장 핵심적이거나 가장 복잡하고 이해하기 어려운 부분에 작성된
주석 또는 doc string을 보고 해당 코드가 잘 이해되었나요?**
    - `ActivationMapExtractor`를 부모 클래스로 두고 CAM과 Grad-CAM의 공통 로직을
      하나로 묶은 부분과 여러 주석들 덕분에 흐름을 이해할 수 있었습니다.

<img width="994" height="464" alt="스크린샷 2026-08-28 오후 5 14 03" src="https://github.com/user-attachments/assets/19fa527d-2463-412d-8de7-df622f2ff36e" />
<img width="772" height="533" alt="스크린샷 2026-08-28 오후 5 15 41" src="https://github.com/user-attachments/assets/568a153c-7755-480e-9550-4889a607313b" />


- [x] **3. 에러가 난 부분을 디버깅하여 문제를 해결한 기록을 남겼거나
새로운 시도 또는 추가 실험을 수행해봤나요?**
    - 별도의 `디버깅 로그`를 두어 IoU 좌표계 불일치, OpenCV BGR/RGB 문제,
      Grad-CAM gradient hook 문제 등을 증상 → 원인 → 해결 순서로 기록했습니다.
<img width="1007" height="592" alt="스크린샷 2026-08-28 오후 5 17 25" src="https://github.com/user-attachments/assets/6dffee27-5e9a-44ff-b433-6bff88e38915" />

    - 추가 실험도 threshold 민감도, CAM과 Grad-CAM의 이론적 등가성,
      레이어별 Grad-CAM IoU, 오분류 샘플 분석까지 총 4종을 수행했습니다.
    - 특히 처음 예상했던 "깊은 layer일수록 localization이 좋을 것"이라는 가설과
      실제 결과가 다르자, 활성 영역과 예측 Bounding Box의 크기를 추가로 측정하여
      원인을 다시 분석한 과정이 남달랐습니다.
      


- [x] **4. 회고를 잘 작성했나요?**
    - 프로젝트 처음에 전체 실행 흐름을 도식으로 제시하여
      데이터 → 학습 → CAM/Grad-CAM → BBox → IoU의 관계를 이해하기 쉬웠습니다.
    - 회고에서도 단순히 결과를 정리하는 데 그치지 않고,
      CAM의 의미, Grad-CAM에서 공간축과 채널축을 다루는 방식,
      weakly supervised localization의 의미 등을 구현 경험과 연결해 정리했습니다.
<img width="1073" height="609" alt="스크린샷 2026-08-28 오후 5 18 00" src="https://github.com/user-attachments/assets/2e259fbc-7fcf-41fc-9baa-7002e1c45fd6" />




- [x] **5. 코드가 간결하고 효율적인가요?**
    - `Config` dataclass를 이용해 하이퍼파라미터와 경로를 한 곳에서 관리하고,
      학습/검증을 `run_one_epoch()`로 통합하는 등 중복 코드가 잘 정리되어 있습니다.
    - CAM과 Grad-CAM 또한 공통 기능을 `ActivationMapExtractor`에 모으고
      가중치 계산 방식만 상속을 통해 분리해 범용적으로 사용할 수 있도록 설계했습니다.
<img width="1070" height="599" alt="스크린샷 2026-08-28 오후 5 18 44" src="https://github.com/user-attachments/assets/252970ef-252d-48ab-b098-f50575598df6" />


# 회고(참고 링크 및 코드 개선)

이번 딥러닝해커톤 때에도 느꼈지만, 문서 전개가 탄탄하고 연구자의 자세가 돋보입니다. 

결과가 예상과 다르게 나왔을 때 단순히 모델 성능의 문제로 결론내리지 않고, 새로운 측정값을 추가하여 원인을 추적하는 구조를 논리적으로 전개합니다.

layer4가 가장 의미적인 feature를 갖고 있으므로 localization에도 가장 유리할 것이라는 초기 가설과 달리 layer2의 IoU가 더 높게 나온 결과를,
활성 영역의 면적뿐 아니라 예측 Bounding Box의 공간적 퍼짐까지 측정하며 분석하는 모습에  구글 코랩 자원 한계만 탓하던 제 연구 설계가 실로 조촐하게 느껴졌습니다. 

전체적으로 구현, 정량평가, 추가 실험, 디버깅, 회고가 하나의 연구 흐름으로 연결되어 있어 감탄할 부분이 많은 프로젝트였습니다.
