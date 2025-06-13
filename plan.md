# 사업자등록증 OCR 프로그램 개발 Plan

## Notes
- 프로젝트는 한국 등기부등본의 주요 필드 OCR 추출 및 단일 포맷 내보내기를 목표로 함
- Naver Clova 오픈소스(CRAFT, deep-text-recognition-benchmark) 활용, 상용 API 미사용
- VDI 환경(외부망 제한, 리소스 제약)에서의 안정성, 배포, 패키징 중요
- 데이터셋 품질 확보, 정보 추출 후처리, UI 설계, 오픈소스 유지보수 책임 강조
- 프런트엔드(OcrMaster) UI는 React 기반 컴포넌트 구조로 Python OCR 엔진과 옵션/포맷/실행 등 결합이 매우 용이함
- Tauri 기반의 가벼운 데스크탑 GUI(프런트엔드)와 Python OCR 엔진(백엔드) 구조로 개발, Node.js는 배포/실행 시 필요 없음
- Tauri 패키징 및 폴더 복사만으로 배포/실행 가능, 실제 폴더 구조/실행 스크립트(run_gui.bat) 설계 및 테스트 필요
- Python OCR 엔진(run_ocr.py) 기본 구조 및 샘플 코드 작성 완료 (CRAFT/Recognition 추상화, CLI 인자 기반)
- Tauri 프론트엔드 폴더/파일 구조, 샘플 코드(App.tsx, main.rs, index.html 등), 연동 방식(서브프로세스 Rust Command) 및 자동화 스크립트(run_gui.bat) 코드화 및 저장 완료
- CRAFT 및 deep-text-recognition-benchmark 모델, 코드, 가중치 등 모든 구성요소를 VDI 내부에 직접 배치하여 완전 오프라인(외부 API 미사용) 환경에서 동작하도록 구현
- 실제 CRAFT/Recognition 추론 로직은 아직 run_ocr.py에 미구현(플레이스홀더 상태)
- 대상 문서는 등기부등본이며, PDF 내 특정 영역의 정보 추출이 목적임
- 출력 포맷으로 JSON(좌표, 신뢰도 등 메타데이터 포함), CSV 지원 필요
- 문서 레이아웃 분석 및 표/양식/비정형 문서 처리 범위 명확화(템플릿 기반/비기반)
- 실시간 및 대규모 처리, 확장성, 클라우드/분산처리 옵션 고려
- OCR 후처리(오류 교정, NLP 기반 문맥 보정) 및 사용자 검증 UI 제공 필요

## Task List
- [ ] 고품질 등기부등본 데이터셋 확보 및 전처리
- [ ] 샘플 이미지/문서 분석 및 좌표 지정 방식 설계
- [ ] PDF/이미지 특정 영역 crop 및 OCR 추출 로직 구현
- [ ] CRAFT 기반 텍스트 검출 모델 도입 및 한글 최적화
- [ ] deep-text-recognition-benchmark 기반 텍스트 인식 모델 도입 및 한글 최적화
- [ ] OCR 결과 후처리 및 주요 필드 정보 추출 로직 구현
- [ ] JSON(좌표/신뢰도 등 포함), CSV 등 다양한 출력 포맷 지원
- [ ] VDI 환경 대응 배포/패키징(Tauri 기반 GUI + Python 엔진, 설치 없이 폴더 복사만으로 동작)
- [x] Python OCR 엔진(run_ocr.py) 기본 구조 코드화 및 저장
- [x] Tauri 기반 GUI 설계 및 Python OCR 엔진 연동 방식 구체화 및 코드화
- [x] Tauri 프론트엔드 초기 설계(예시 화면, 기능 정의) 및 샘플 코드 저장
- [x] 연동 방식(서브프로세스/HTTP API) 코드화 및 샘플 작성
- [x] 샘플 코드 및 배포 자동화 스크립트 작성
- [ ] CRAFT/Recognition 실제 모델 연동 및 추론 코드 구현
- [ ] 기능/입력/출력 포맷 옵션화 및 사용자 선택지 제공
- [ ] 프런트엔드-백엔드 옵션/엔진 선택 연동 구현
- [ ] Tauri 최종 패키징/폴더 복사 실행 테스트
- [ ] 사용자 피드백 수집 및 기능 개선(확장)
- [ ] 문서 레이아웃 분석, 템플릿/비템플릿 처리 전략 수립 및 구현
- [ ] 실시간/대규모 처리 및 확장성 검증
- [ ] 후처리(오류 교정, NLP), 사용자 검증 UI 개발

## Current Goal
CRAFT/Recognition 실제 모델 연동 및 기능/입력/출력 포맷 옵션화