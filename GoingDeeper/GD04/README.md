# AIFFEL Campus Online Code Peer Review Templete
- 코더 : 코더의 이름을 작성하세요.
- 리뷰어 : 리뷰어의 이름을 작성하세요.


# PRT(Peer Review Template)
- [x]  **1. 주어진 문제를 해결하는 완성된 코드가 제출되었나요?**
    1) Hourglass와 SimpleBaseline을 비교분석한 결과를 체계적으로 정리하였다.
       - loss 곡선, 파라미터 수, 최종 loss, 소요시간, RCKh@0.5 등을 계산하여 정량 비교를 진행하였다.
       - compare_models()로 동일 테스트 이미지에 대한 두 모델의 pose estimation 결과를 나란히 시각화하여 정성 비교하였다.
       - 단순 수치 나열이 아니라 결론을 도출하였고, 한계도 명시하였다.
         
         <img width="1100" height="678" alt="image" src="https://github.com/user-attachments/assets/0ac7a754-df20-4e3f-a15f-981445a55489" />
         <img width="837" height="823" alt="image" src="https://github.com/user-attachments/assets/c8ef7f05-c9a9-40bf-bb72-a6d97eb17518" />
         
    2) simplebaseline 모델을 정상적으로 구현한 후 정상적으로 학습하였다.
       - SimpleBaseline 클래스가 완결된 형태로 구현되어 있다. forward()가 Hourglass와 동일하게 [heatmap] 리스트를 반환하도록 인터페이스를 통일해, 학습 셀에서 model= 인자 한 줄만 바꿔 대체가 되도록 설계했다.
       - epoch 1 train 0.0403 | val 0.0370 → 이후 loss가 꾸준히 감소하며 매 epoch의 체크포인트가 저장된다. 에러 없이 5 epoch 전체가 끝까지 실행됐다.
      
         <img width="690" height="924" alt="image" src="https://github.com/user-attachments/assets/59ff1eba-d7f2-4a29-8051-405f651017ca" />
         <img width="1114" height="280" alt="image" src="https://github.com/user-attachments/assets/45df8387-0b16-4808-8b89-7b073f356f17" />

    3) MPII 데이터셋 기반 1 epoch 30분 이내 학습 가능한 베이스라인을 구축하였다.
       - 2-4 섹션에서 데이터 로딩만으로 1 epoch 예상 시간 1.0분을 측정했고, 실제 학습 로그에서도 epoch당 StackedHourglass 4.7~4.8분, SimpleBaseline 1.6~2.2분으로 30분 기준을 크게 하회함을 실측으로 확인했다.

         <img width="679" height="623" alt="image" src="https://github.com/user-attachments/assets/3f3c2ee4-b34a-4aa6-b815-9e9870dcd6cb" />

    
- [x]  **2. 전체 코드에서 가장 핵심적이거나 가장 복잡하고 이해하기 어려운 부분에 작성된 
주석 또는 doc string을 보고 해당 코드가 잘 이해되었나요?**
    - `SimpleBaseline` 클래스가 가장 핵심적이라고 생각한다. Hourglass와 완전히 다른 구조를 가진 두 모델이 하나의 Trainer를 공유하도록 인터페이스를 맞추었고, pretrained/신규 레이어를 서로 다른 방식으로 초기화하는 부분이 한 클래스 안에 몰려 있어 설계 의도를 모르고 코드만 읽어서는 왜 이렇게 짰는지를 파악하기 어렵기 때문이다.
      - `DeconvHead`: kernel_size=4, stride=2, padding=1을 쓰는 이유를 "출력 = (입력-1)stride - 2padding + kernel = 2*입력" 공식까지 docstring에 적어, 왜 이 조합이 해상도를 정확히 2배로 키우는지, 그래서 3번 통과하면 8×8이 정확히 64×64가 되는지가 계산으로 검증된다.
      - `SimpleBaseline`: return_list 인자를 Hourglass와 동일한 [heatmap] 리스트 반환 규약을 맞추기 위한 스위치라고 명시했다.
      - `forward()`: encoder → neck → decoder → final_layer 각 줄에 텐서 shape을 주석으로 작성하여 실행하지 않고 코드만 읽어도 shape 흐름을 이해할 수 있다.

        <img width="679" height="923" alt="image" src="https://github.com/user-attachments/assets/240138de-b2c3-4184-8e83-02e9de3e0cf6" />

        
- [x]  **3. 에러가 난 부분을 디버깅하여 문제를 해결한 기록을 남겼거나
새로운 시도 또는 추가 실험을 수행해봤나요?**
    - `6. 디버깅 기록 및 추가 실험` 섹션에 4건이 "발견 경위 → 원인 → 수정" 형식으로 정리되어 있다.
      1) `[버그]` heatmap→좌표 복원 시 x/y 축이 뒤바뀌는 문제 — H==W(정사각형 heatmap)일 때만 우연히 맞던 인덱스 복원 오류를 발견하고 축별로 나누도록 수정.
      2) `[버그]` 랜덤 margin으로 인해 crop 밖으로 나간 keypoint가 잘못된 위치에 정답 heatmap으로 찍히는 문제 — visibility를 0으로 강등하는 방식으로 수정.
      3) `[성능]` 실습 원본의 불필요한 JPEG 인코딩→디코딩 왕복 제거 — 대조군(LegacyPreprocessor)을 직접 만들어 19.5ms → 12.2ms/sample(37.2% 단축) 로 정량 측정까지 수행했다. 단순 서술이 아니라 코드로 재현·측정한 점이 좋다.
      4) `[추가 실험]` Mixed Precision(AMP) 적용. Hourglass가 SimpleBaseline보다 파라미터는 1.67배 많은데 학습 시간은 2.80배 더 걸린다고 분석한 부분과 맞물려, AMP 도입의 필요성이 근거를 갖고 설명된다.
     
         <img width="1095" height="866" alt="image" src="https://github.com/user-attachments/assets/2e06b91a-197d-43ec-a372-ad946bdda43e" />
         <img width="1108" height="640" alt="image" src="https://github.com/user-attachments/assets/916bc3f2-0ea5-47fa-94e8-be7124e8575c" />


        
- [x]  **4. 회고를 잘 작성했나요?**
    - 마지막 셀에 배운 점 / 아쉬운 점 / 느낀 점 / 다음에 해볼 것 네 항목으로 구성되어 있고, 각 항목이 막연한 감상이 아니라 구체적 근거를 동반한다.
    - 전체 실행 플로우 다이어그래도 있다. 데이터 다운로드→전처리→Trainer→두 모델→평가까지 전체 흐름을 빠짐없이 보여주고 있어 이해에 도움이 되었다.
     
      <img width="1107" height="777" alt="image" src="https://github.com/user-attachments/assets/83963388-83d4-4ce9-aaf4-b8acaae0fae2" />
      <img width="676" height="661" alt="image" src="https://github.com/user-attachments/assets/6f2cf554-5518-4abe-890d-8520ece2a66d" />


        
- [x]  **5. 코드가 간결하고 효율적인가요?**
    - `Preprocessor`, `MPIIDataset`, `build_dataloader`, `Trainer/History`, `build_backbone`/`build_simplebaseline`/`build_hourglass` 팩토리 함수, `decode_heatmap`/`predict`/`draw_pose`/`compare_models`/`plot_history` 등 재사용 단위로 깔끔히 분리되어 있다.
    - 특히 두 모델이 완전히 동일한 Trainer, 데이터로더, 평가 함수를 공유하도록 `forward()` 반환 형식을 통일해, 모델 비교라는 요구사항 자체를 코드 구조로 보장한 설계가 가장 잘 구현된 부분이라고 생각한다.
     
      <img width="643" height="1037" alt="image" src="https://github.com/user-attachments/assets/12efb0bf-ebf3-4c2c-891d-69eb887483c8" />



# 회고(참고 링크 및 코드 개선)
```
StackedHourglass와 SimpleBaseline이 동일한 Trainer·데이터로더·평가 함수를 공유하도록 forward() 반환 형식을 리스트로 통일해둔 설계가 인상적이었습니다.
덕분에 두 모델 비교라는 과제 목표가 코드 구조로 자연스럽게 보장되는 것 같습니다.
또한 JPEG 왕복 제거 효과를 대조군까지 만들어 직접 측정하신 부분처럼, 실험 설계와 디버깅 기록을 꼼꼼하게 정리하시는 것 같습니다.
많이 배웠습니다.
```
