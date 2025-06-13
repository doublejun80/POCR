# 오프라인 한글 OCR 구축 안내 (CRAFT + deep-text-recognition-benchmark)

## 1. 개요
- 본 프로젝트는 VDI 등 오프라인 환경에서 동작하도록 CRAFT(텍스트 영역 검출)와 deep-text-recognition-benchmark(문자 인식) 오픈소스 모델을 활용해 등기부등본/한국 문서 OCR을 구현합니다.
- 외부 API/클라우드 미사용, 모든 코드/모델/가중치는 내부에 직접 배치

## 2. 폴더 구조 예시
```
SKAXPOCR/
├─ sample_1.jpg
├─ sample_2.jpg
├─ requirements.txt
├─ craft/                  # CRAFT 코드 및 모델
├─ recognition/            # deep-text-recognition-benchmark 코드 및 모델
├─ run_ocr.py              # 전체 실행 스크립트
```

## 3. 오픈소스 코드/모델 다운로드 안내
- [CRAFT (official)](https://github.com/clovaai/CRAFT-pytorch)
    - 코드 전체 다운로드
    - 모델: craft_mlt_25k.pth (Release/Google Drive)
- [deep-text-recognition-benchmark (official)](https://github.com/clovaai/deep-text-recognition-benchmark)
    - 코드 전체 다운로드
    - 모델: 한글 지원 사전학습 가중치(공식/커뮤니티)

> **참고:**
> - 각 모델의 weights(.pth)는 인터넷 연결 가능한 PC에서 미리 받아 VDI로 옮겨야 함
> - 각 오픈소스의 requirements.txt도 참고하여 의존성 충돌 없도록 설치

## 4. 샘플 실행 코드 (run_ocr.py)
- 샘플 이미지를 대상으로 CRAFT로 텍스트 박스 검출 → recognition으로 한글 인식 → 결과 JSON/CSV 저장
- 실제 코드는 craft/, recognition/ 폴더 내 모듈 import 방식으로 작성

## 5. 추가 참고
- PDF 지원은 pdf2image로 이미지 변환 후 동일 방식 적용
- 저화질/삐뚤어진 이미지: opencv 등으로 전처리(회전, 노이즈 제거 등) 추가 가능

---
자세한 실행 예제 및 코드 작성은 다음 단계에서 진행합니다.
