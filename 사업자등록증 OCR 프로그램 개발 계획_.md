

# **사업자등록증 OCR 프로그램 개발 사업계획 수립을 위한 기술 심층 분석 보고서**

## **1\. 기술적 타당성 및 접근 방식 요약**

본 보고서는 PDF 및 JPG 형식의 사업자등록증에서 정형화된 패턴을 인식하여 주요 정보를 추출하고, 이를 워드 또는 엑셀 형식으로 업로드하는 OCR(Optical Character Recognition, 광학 문자 인식) 프로그램 개발에 필요한 기술적 사항을 심층적으로 분석한다. 개발 목표는 외부망 접근이 불가능한 VDI(Virtual Desktop Infrastructure) 환경에 배포 가능하며, 웹뷰(Webview) 형태로 제공되는 사용자 친화적 애플리케이션을 구축하는 것이다.

주요 기술적 결정 사항으로는 Naver Clova의 오픈소스인 CRAFT(Character Region Awareness for Text Detection)를 텍스트 영역 검출에 활용하고, deep-text-recognition-benchmark 저장소에서 제공하는 모델들을 문자 인식에 사용하는 방안이 제시된다.1 이들 기술은 오픈소스 라이선스로 제공되어 자유로운 활용이 가능하며, 한국어 및 특정 문서(사업자등록증)의 구조에 대한 정확도 향상을 위해 필수적인 커스터마이징 및 미세조정(fine-tuning) 과정을 거치게 된다. VDI 배포 환경을 고려하여, PyInstaller와 같은 도구를 사용한 패키징 전략이 권장되며, 웹뷰 기술로는 Tauri 또는 Electron 등이 고려될 수 있으나, 각 기술의 장단점을 면밀히 검토하여 최적의 방안을 선택해야 한다.

핵심 도전 과제는 오프라인 환경에서의 안정적인 배포, 한국어 OCR의 정확성 확보, 그리고 사업자등록증의 정형화된 데이터 추출이다. 이러한 과제 해결을 위해 본 보고서에서는 구체적인 기술 스택, 개발 방법론, 그리고 단계별 이행 계획을 제안한다.

본 프로젝트의 성공적인 완수는 단순히 특정 문서 처리 자동화를 넘어, 유사한 형태의 표준화된 국내 문서 처리를 위한 OCR 솔루션 개발의 기반 기술 및 배포 전략을 확보하는 중요한 의미를 지닌다. 일반적인 목적의 오픈소스 OCR 도구를 특정 국가의 법적 효력을 지닌 문서에 적용하는 과정은 그 자체로 상당한 기술적 노하우 축적을 동반한다. 사업자등록증은 정형화된 패턴을 가지고 있지만 4, 그 안에 포함된 정보의 정확한 추출은 비즈니스 애플리케이션에서 매우 중요하다. 따라서, 범용 데이터셋으로 학습된 모델을 한국어 및 사업자등록증의 고유한 특성에 맞게 조정하는 과정이 프로젝트의 핵심 성공 요인이 될 것이다. 이 과정에서 얻어지는 경험과 자산은 향후 다른 종류의 정형화된 한국어 문서 처리 시스템 개발로 확장될 수 있는 잠재력을 가진다.

## **2\. 사업자등록증 OCR을 위한 기술 심층 분석**

### **2.1. Naver Clova 오픈소스 컴포넌트 활용**

#### **2.1.1. CRAFT (Character Region Awareness for Text Detection)**

CRAFT는 각 문자의 영역과 문자 간의 관계(affinity)를 탐색하여 텍스트 영역을 효과적으로 검출하는 알고리즘이다.2 PyTorch 기반으로 구현되어 있으며 1, 복잡한 레이아웃에서도 강인한 텍스트 검출 성능을 보여 문서 분석에 유리하다.

craft\_mlt\_25k.pth와 같은 사전 학습된 모델은 다국어 및 일반적인 텍스트 검출에 활용될 수 있어 초기 개발의 좋은 출발점을 제공한다.2

그러나 CRAFT의 공식 저장소는 지적재산권 문제로 인해 학습 코드를 제공하지 않는다.2 이는 CRAFT 검출 모델 자체를 직접 미세조정하는 데 제약이 있음을 의미한다. 따라서, 한국어 사업자등록증의 특성에 맞춘 성능 최적화는 CRAFT를 강력한 특징 추출기로 활용하거나, 기존 모델의 일반화 성능에 의존하되, 입력 이미지의 전처리 및 후처리 단계를 강화하는 방향으로 진행될 가능성이 높다.

#### **2.1.2. 텍스트 인식 모델 (예: deep-text-recognition-benchmark 활용)**

deep-text-recognition-benchmark 저장소는 TPS(Thin Plate Spline), ResNet, BiLSTM, Attention 메커니즘 등을 조합한 다양한 텍스트 인식 모델 아키텍처와 학습/평가 프레임워크를 제공하는 핵심 자원이다.1 이 모델들은 CRAFT와 같은 텍스트 검출기에 의해 식별된 문자 영역의 이미지를 입력받아 실제 문자열로 변환하는 역할을 수행한다. 저장소 내

train.py, test.py, demo.py 스크립트는 모델 학습, 평가, 그리고 실제 추론 과정에 필수적으로 사용된다.3

#### **2.1.3. Donut (OCR-free Document Understanding Transformer)의 잠재력**

Donut은 별도의 OCR 파이프라인 없이 문서 이해 작업을 엔드투엔드로 수행하는 Transformer 기반 모델이다.1 특히 구조화된 문서에서 강력한 성능을 보이며, 문서 분류 및 정보 추출 작업에 적합하다. 사업자등록증이 정형화된 구조를 가지고 있다는 점을 고려할 때, Donut은 CRAFT와 인식 모델을 결합하는 전통적인 파이프라인의 대안 또는 보완적인 접근법이 될 수 있다. Donut을 활용하면 텍스트 검출 및 인식 단계를 거치지 않고 직접적으로 필요한 정보를 추출할 가능성도 탐색해볼 수 있다. 다만, 특정 문서 유형과 한국어에 대한 Donut 모델의 학습 및 미세조정 복잡도, 그리고 기존 파이프라인과의 성능 비교 검토가 필요하다. Donut은 MIT 라이선스로 제공된다.1

### **2.2. 한국어 및 문서 구조에 대한 핵심 커스터마이징 및 미세조정**

#### **2.2.1. 특정 레이아웃 및 한국어 텍스트 처리**

가장 큰 도전 과제는 주로 영어 또는 일반 다국어 데이터셋으로 사전 학습된 모델들을 한국어 문자를 정확히 인식하고 사업자등록증의 특정 필드 레이아웃을 이해하도록 조정하는 것이다. 사업자등록증에는 '상호', '대표자 성명', '사업자등록번호', '사업장 소재지', '개업연월일', '사업의 종류(업태, 종목)' 등 정확하게 식별하고 추출해야 하는 다수의 중요 필드가 존재한다.4

#### **2.2.2. 한국어 학습 데이터 적용 전략**

* **커스텀 데이터셋 구축:** deep-text-recognition-benchmark 저장소의 create\_lmdb\_dataset.py 스크립트를 활용하여 LMDB(Lightning Memory-Mapped Database) 형식의 커스텀 데이터셋을 구축한다.3 이는 텍스트 인식 모델 학습에 필요한 표준 데이터 형식이다. 입력 형식은 이미지 폴더와 각 이미지 파일명에 해당하는 한국어 텍스트 전사(transcription)를 매핑한  
  gt.txt 파일로 구성된다.3  
* **합성 데이터 생성 (SynthDoG, SynthTIGER):** Naver Clova에서 제공하는 SynthDoG (Donut 프로젝트의 일부) 1 및 SynthTIGER 1와 같은 합성 텍스트 이미지 생성 도구는 한국어 학습 데이터셋을 보강하는 데 매우 유용할 수 있다. 특히 실제 사업자등록증 이미지와 주석(annotation) 데이터가 부족할 경우, 이들 도구를 활용하여 현실적인 한국어 사업자등록증 텍스트 이미지를 생성함으로써 학습 데이터의 양과 다양성을 확보할 수 있다. 이러한 합성 데이터 생성 도구의 활용은 실제 데이터 수집 및 레이블링에 소요되는 시간과 비용을 절감하고, 모델의 일반화 성능을 향상시키는 데 기여할 수 있다.  
* **데이터 증강 (Data Augmentation):** Albumentations와 같은 라이브러리를 사용하여 회전, 크기 조절, 노이즈 추가, 블러 처리 등 표준적인 데이터 증강 기법을 적용하여 모델의 강인성을 향상시킨다.8

#### **2.2.3. 선택 모델에 대한 미세조정 접근법**

주요 미세조정 대상은 deep-text-recognition-benchmark에서 제공하는 텍스트 *인식* 모델이 될 것이다.3 구축된 한국어 LMDB 데이터셋을 사용하여

train.py 스크립트를 통해 미세조정을 진행한다. 이때, \--select\_data, \--batch\_ratio와 같은 학습 데이터 관련 파라미터와 사업자등록증에 나타나는 모든 한국어 문자, 기호, 영문자, 숫자 등을 포함하도록 opt.character 파라미터를 정확히 설정하는 것이 매우 중요하다.3 CRAFT 모델 자체의 학습 코드가 제공되지 않는 상황 2 때문에, 텍스트 검출 단계에서는 사전 학습된 CRAFT 모델의 일반화 성능에 의존하거나, 사업자등록증 이미지에 대한 정교한 전처리 과정을 통해 CRAFT가 한국어 텍스트 영역을 효과적으로 검출하도록 유도해야 한다. 결과적으로, 한국어 특화 및 문서 구조 적응의 부담은 텍스트 인식 모델의 미세조정과 이미지 전처리 단계에 더 크게 집중될 것이다.

### **2.3. 관련 GitHub 저장소 및 오픈소스 활용 사례 분석**

Naver Clova AI Research의 GitHub 1는 CRAFT, Donut,

deep-text-recognition-benchmark, SynthTIGER 등 본 프로젝트의 핵심 기술들의 주요 소스이다. 이들 저장소의 활발한 관리와 커뮤니티 지원은 기술의 품질과 안정성을 뒷받침한다. CRAFT-pytorch 2는 MIT 라이선스로 제공되며,

test.py 스크립트와 관련 인자들(--text\_threshold, \--link\_threshold 등)은 추론 과정 설정에 중요하다. deep-text-recognition-benchmark 3는 Apache-2.0 라이선스로 다양한 인식 모델과 견고한 학습/테스트 스크립트를 제공하며,

demo.py 3는 사전 학습 모델을 사용한 추론 예시를 보여준다.

Naver Cloud Platform에서 제공하는 상용 OCR 서비스 중에는 사업자등록증 정보 추출 API가 존재한다.11 이는 본 프로젝트가 오픈소스 기반으로 개발됨에도 불구하고, Naver가 이미 해당 문제(한국 사업자등록증 OCR)에 대한 해결책과 노하우를 보유하고 있음을 시사한다. 상용 API의 존재는 해당 문제가 기술적으로 해결 가능하며, Naver의 핵심 기술을 활용하여 높은 수준의 정확도를 달성하는 것이 현실적인 목표임을 간접적으로 증명한다. 이는 오픈소스 컴포넌트를 적절히 커스터마이징했을 때 기대할 수 있는 성능 수준의 기준점이 될 수 있다.

다음 표는 본 프로젝트에서 활용될 Naver Clova의 주요 오픈소스 OCR 컴포넌트를 분석한 것이다.

**표 2.1: Naver Clova 오픈소스 OCR 컴포넌트 분석**

| 컴포넌트 | 주요 특징 및 강점 | 한국어 문서 커스터마이징 잠재력 | 주요 GitHub 소스 | 라이선스 |
| :---- | :---- | :---- | :---- | :---- |
| CRAFT | 문자 영역 및 문자 간 관계 기반의 강인한 텍스트 검출, PyTorch 구현 | 사전 학습 모델 활용, 전/후처리로 성능 보강 가능 (학습 코드 미제공으로 직접 미세조정은 어려움) | 1 | MIT |
| deep-text-recognition-benchmark 모델 | 다양한 인식 모델 아키텍처 (TPS, ResNet, BiLSTM, Attn 등) 제공, LMDB 기반 학습/평가 스크립트 | 한국어 데이터셋을 사용한 직접적인 미세조정 가능, opt.character 설정으로 한국어 문자셋 지원 | 1 | Apache-2.0 |
| Donut | OCR-free 엔드투엔드 문서 이해 Transformer, 구조화된 문서에 강점 | 한국어 사업자등록증 데이터로 미세조정 시 정보 직접 추출 가능성, 학습 복잡도 고려 필요 | 1 | MIT |
| SynthTIGER | 합성 텍스트 이미지 생성 도구 | 한국어 텍스트 및 사업자등록증 스타일의 합성 데이터 생성으로 학습 데이터 부족 문제 해결 기여 가능 | 1 | MIT |

## **3\. 시스템 아키텍처 및 워크플로우 설계**

### **3.1. 엔드투엔드 프로세스 흐름도 (개념적)**

사업자등록증 OCR 프로그램의 전체 처리 흐름은 다음과 같이 구상할 수 있다:

1. **입력**: 사용자가 PDF 또는 JPG 형식의 사업자등록증 파일을 시스템에 업로드한다.  
2. **전처리**:  
   * PDF 파일의 경우, 이미지로 변환한다 (예: PyMuPDF 라이브러리 사용). 다중 페이지 PDF는 각 페이지를 개별 이미지로 처리한다.  
   * 이미지 품질 최적화를 위해 크기 조정, 노이즈 제거, 명암 대비 향상, 이진화(binarization), 기울기 보정(deskewing) 등의 작업을 수행한다 (예: OpenCV 라이브러리 활용).  
3. **텍스트 검출 (CRAFT)**: 전처리된 이미지를 CRAFT 모델에 입력하여 텍스트가 존재하는 영역의 경계 상자(bounding box) 좌표를 얻는다.  
4. **텍스트 인식 (커스텀 한국어 모델)**: 검출된 각 텍스트 영역 이미지를 미세조정된 한국어 텍스트 인식 모델(예: deep-text-recognition-benchmark 기반 모델)에 입력하여 실제 문자열을 추출한다.  
5. **후처리 및 정보 추출**:  
   * 인식된 원시 텍스트(raw text)로부터 사업자등록증의 정형화된 필드(예: 상호, 대표자명, 사업자등록번호 등)에 해당하는 정보를 매핑한다. 이는 키워드 검색, 텍스트 블록의 상대적 위치 분석, 정규 표현식 활용 등을 통해 이루어진다.  
   * 추출된 정보에 대해 기본적인 유효성 검사(예: 사업자등록번호 형식, 날짜 형식)를 수행한다.  
   * 필요시 한국어 맞춤법 교정 또는 문자 교정 로직을 적용한다.  
6. **구조화된 데이터 생성**: 추출 및 검증된 정보를 JSON 또는 Python 딕셔너리 형태의 키-값 쌍으로 구조화한다.  
7. **출력**: 구조화된 데이터를 사용자의 선택에 따라 워드(.docx) 또는 엑셀(.xlsx) 파일 형식으로 변환하여 제공한다.

### **3.2. 모듈 1: 입력 처리 및 전처리**

이 모듈은 다양한 형식의 입력 파일을 수용하고 OCR 정확도를 높이기 위한 이미지 최적화 작업을 담당한다. PDF 파일 처리를 위해서는 PyMuPDF와 같은 라이브러리를, JPG 파일 처리를 위해서는 OpenCV 또는 Pillow 라이브러리를 사용할 수 있다. 이미지 최적화는 OCR 성능에 직접적인 영향을 미치므로, 다양한 기법(예: 가우시안 블러, 오츠 이진화, 적응형 스레시홀딩)을 실험하고 문서 특성에 맞는 최적의 파이프라인을 구축하는 것이 중요하다. 사업자등록증의 레이아웃이 매우 일관적이라면, 특정 정보가 위치할 가능성이 높은 관심 영역(Region of Interest, ROI)을 우선적으로 식별하여 처리 효율성과 정확도를 높이는 방안도 고려할 수 있다.

### **3.3. 모듈 2: 핵심 OCR 엔진**

#### **3.3.1. 텍스트 검출 서브모듈 (CRAFT 구현)**

사전 학습된 CRAFT 모델(craft\_mlt\_25k.pth 등)을 로드하여 사용한다.2 입력 이미지는 CRAFT 모델의 요구사항에 맞게 크기 조정(

canvas\_size, mag\_ratio 파라미터 활용) 등의 전처리 과정을 거친다.2 모델 추론을 통해 문자 영역 점수(region score)와 문자 연결 점수(affinity score)를 얻고, 이를 후처리하여 개별 단어 또는 텍스트 라인에 대한 경계 상자 좌표를 생성한다. 이 후처리 로직은 CRAFT-pytorch 저장소의

test.py 내 getDetBoxes 함수 또는 유사한 로직을 참조하여 구현할 수 있다.13

#### **3.3.2. 텍스트 인식 서브모듈 (커스텀 한국어 모델)**

한국어 사업자등록증 데이터로 미세조정된 텍스트 인식 모델(예: TPS-ResNet-BiLSTM-Attn 아키텍처 기반)을 로드한다.3 CRAFT가 검출한 경계 상자를 이용해 원본 이미지에서 텍스트 영역 이미지를 잘라내고, 이를 인식 모델에 입력하여 한국어 문자열을 얻는다.

#### **3.3.3. 후처리 및 정보 추출**

이 단계는 OCR 시스템의 실질적인 가치를 결정하는 매우 중요한 부분이다. 단순히 문자를 인식하는 것을 넘어, 인식된 텍스트가 사업자등록증의 어떤 필드에 해당하는지를 정확히 연결해야 한다. 사업자등록증은 "상호:", "대표자:", "등록번호:" 와 같이 특정 레이블과 함께 정보가 기재되는 정형화된 패턴을 가지고 있다.4 이러한 특성을 활용하여, 키워드 기반 검색 (예: "상호"라는 문자열을 찾고 그 주변의 텍스트를 회사명으로 인식), 인식된 텍스트 블록들의 상대적 위치 관계 분석, 정규 표현식을 이용한 특정 패턴(예: 사업자등록번호 형식 000-00-00000) 추출 등의 규칙 기반 또는 템플릿 기반 접근 방식을 적용할 수 있다.

최종적으로 추출된 정보는 {"상호": "주식회사 네이버", "대표자": "최수연",...}과 같은 Python 딕셔너리 또는 JSON 객체로 구조화된다. 이 구조화된 데이터의 정확성은 OCR 문자 인식 정확도뿐만 아니라, 이 정보 추출 로직의 정교함에 크게 좌우된다. 만약 필드 매핑 과정에서 오류가 발생한다면, 개별 문자가 아무리 정확하게 인식되었더라도 최종 결과물의 유용성은 현저히 저하될 것이다. 따라서 사업자등록증의 다양한 양식을 분석하고, 예외 케이스를 처리할 수 있는 견고한 정보 추출 로직 개발에 상당한 노력을 기울여야 한다.

### **3.4. 모듈 3: 데이터 출력 및 통합 (워드/엑셀 내보내기)**

구조화된 OCR 결과는 사용자의 업무 활용도를 높이기 위해 워드 또는 엑셀 파일로 내보내져야 한다.

* **워드(.docx) 내보내기**: python-docx 라이브러리를 사용하여 워드 문서를 생성하고, 추출된 데이터를 표 형태로 삽입하거나 텍스트 형식으로 정리하여 채워 넣을 수 있다.14  
* **엑셀(.xlsx) 내보내기**: openpyxl 라이브러리 또는 pandas DataFrame의 to\_excel 기능을 활용하여 엑셀 스프레드시트를 생성한다.16 각 사업자등록증의 추출 정보는 한 행(row)으로, 각 필드는 열(column)로 구성하여 데이터를 체계적으로 관리할 수 있도록 한다.

모듈화된 아키텍처 설계는 각 구성 요소의 독립적인 개발, 테스트, 그리고 향후 개선을 용이하게 한다. 예를 들어, 더 향상된 OCR 인식 모델이 개발될 경우, 입력 처리 모듈이나 출력 모듈의 큰 변경 없이 OCR 엔진 모듈만 교체하여 시스템 전체 성능을 업그레이드할 수 있다.

다음 표는 사업자등록증에서 추출해야 할 주요 표준 필드들을 정의한 것이다. 이는 정보 추출 모듈 개발의 기초 자료로 활용된다.

**표 3.1: 한국 사업자등록증 표준 필드 정보**

| 필드명 (국문) | 필드명 (영문 번역 예시) | 일반적 데이터 유형/형식 | 예시 | 추출 고려사항/키워드 |
| :---- | :---- | :---- | :---- | :---- |
| 등록번호 | Business Registration Number | \#\#\#-\#\#-\#\#\#\#\# (숫자, 하이픈) | 123-45-67890 | "등록번호", "사업자등록번호" 키워드, 특정 숫자 패턴 |
| 상호 (법인명) | Company Name (Corporate Name) | 텍스트 | (주)가나다라상사 | "상호", "법인명" 키워드 |
| 대표자 (성명) | Representative (Name) | 텍스트 | 홍길동 | "대표자", "성명" 키워드 |
| 사업장 소재지 | Business Location Address | 텍스트 | 서울특별시 강남구 테헤란로 123 | "사업장소재지", "주소" 키워드 |
| 본점 소재지 | Head Office Address | 텍스트 | (법인의 경우) | "본점소재지" 키워드 (법인인 경우 사업장 소재지와 다를 수 있음) |
| 개업연월일 | Business Commencement Date | YYYY년 MM월 DD일 (날짜) | 2023년 01월 15일 | "개업연월일" 키워드, 날짜 패턴 |
| 법인등록번호 | Corporate Registration Number | 숫자, 하이픈 | (법인의 경우) 110111-\*\*\*\*\*\*\* | "법인등록번호" 키워드 (법인인 경우) |
| 사업의 종류 | Type of Business | 텍스트 | 업태: 도소매, 종목: 전자기기 | "사업의 종류", "업태", "종목" 키워드, 복수 항목 가능성 |
| 발급 세무서 | Issuing Tax Office | 텍스트 | 강남세무서장 | 문서 하단 발급기관명 |
| 공동사업자 정보 | Joint Business Owners Info | 텍스트 (표 형식 가능) | (해당하는 경우) | "공동사업자" 키워드, 별도 섹션 또는 표 형태로 존재 가능성 |

데이터 출처: 4 및 일반적인 사업자등록증 양식 기반

## **4\. VDI (오프라인 환경) 배포 전략**

### **4.1. 네트워크 제약 사항 해결**

개발된 OCR 프로그램은 "외부망 접근이 불가한 환경"에 배포되어야 하므로, 애플리케이션과 모든 의존성(Python 인터프리터, 라이브러리, 머신러닝 모델 파일 등)이 포함된 단일 실행 패키지 형태로 제공되어야 한다. 이는 애플리케이션 실행 시 어떠한 외부 네트워크 통신도 시도하지 않아야 함을 의미한다.

### **4.2. 독립 실행형 애플리케이션 번들 생성 기법**

* **PyInstaller 활용**: Python으로 개발된 백엔드 및 OCR 로직을 패키징하는 데 PyInstaller는 강력한 후보이다.18 PyInstaller는 Python 스크립트를 분석하여 의존성을 찾아내고, 이를 단일 실행 파일(  
  \--onefile 옵션) 또는 단일 디렉터리(--onedir 옵션) 형태로 번들링한다. 초기 VDI 환경 디버깅에는 \--onedir 모드가 유리할 수 있으며, 최종 배포에는 사용자 편의성을 위해 \--onefile 모드가 선호될 수 있다.18 PyInstaller 사용 시 PyTorch, OpenCV와 같은 복잡한 라이브러리나  
  .pth 모델 파일, 설정 파일 등이 누락되거나 경로 문제가 발생할 수 있으므로, 철저한 테스트와 필요시 hook 파일 작성 또는 .spec 파일 수정이 요구된다. 이러한 패키징 과정의 성공 여부는 VDI 배포의 핵심 요소이다. Python 인터프리터 자체도 번들에 포함되므로 VDI 환경에 별도의 Python 설치가 필요 없다는 장점이 있다.  
* **모델 파일 및 라이브러리 포함**: CRAFT 및 텍스트 인식 모델의 .pth 파일, 설정 파일, 그리고 기타 필요한 모든 자원(asset)들이 번들 내에 정확히 포함되고, 프로그램 내에서 상대 경로를 통해 올바르게 참조될 수 있도록 경로 관리에 유의해야 한다.

### **4.3. 로컬 접근성 보장**

애플리케이션은 모델 파일 다운로드, 자동 업데이트, 외부 API 호출 등 일체의 네트워크 연결 시도를 하지 않도록 설계되어야 한다. 모든 설정은 번들에 포함된 로컬 설정 파일(예: JSON, YAML 형식)을 통해 관리 가능해야 한다.

### **4.4. VDI 리소스 고려사항**

VDI 환경은 일반적으로 물리적 머신에 비해 CPU, 메모리, 디스크 공간 등의 자원이 제한적일 수 있다. 따라서 최종 번들 크기와 실행 시 메모리/CPU 사용량을 최소화하도록 모델 및 코드를 최적화하는 것이 중요하다. 이는 효율적인 모델 구조 선택, 경량화된 웹뷰 프레임워크 고려 등 기술 선택 단계에서부터 반영되어야 한다.

오프라인 VDI 배포 제약은 웹뷰 기술 선택에도 영향을 미친다. 시스템에 기본 설치된 웹 엔진의 버전이 낮거나 호환성 문제가 있을 수 있고, 오프라인 환경에서는 이를 업데이트할 수 없기 때문이다. 따라서 웹뷰 기술이 자체적으로 웹 렌더링 엔진을 포함하거나(예: Electron), VDI 환경의 웹뷰가 안정적이고 최신 버전임이 보장될 때 적합한 기술(예: Tauri의 기본 모드)을 신중히 선택해야 한다. 성공적인 오프라인 패키징 전략은 초기 VDI 배포 목표를 넘어, 다른 제한된 환경으로의 애플리케이션 이식성을 높이는 부수적인 효과도 가져올 수 있다.

## **5\. 애플리케이션 제공 형태: 최적의 웹뷰 프레임워크 선정**

### **5.1. 비교 분석: Electron vs. Tauri vs. 개별 웹 서버 \+ 웹뷰**

사용자 인터페이스(UI)를 제공하고 OCR 백엔드와 상호작용하기 위한 웹뷰 기술 선택은 중요한 결정 사항이다.

* **Electron**:  
  * **장점**: 성숙한 생태계, 방대한 커뮤니티, Chromium 엔진 내장으로 인한 플랫폼 간 일관된 렌더링 보장 20, Node.js API 접근 용이성 (단,  
    nodeIntegration은 기본적으로 비활성화됨 20). 오프라인 환경에서의 강력한 독립 실행 능력.  
  * **단점**: 상대적으로 큰 번들 크기 21, 높은 메모리 사용량 21, "무겁다(bloat)"는 인식.  
  * **VDI 적합성**: 내장된 Chromium은 통제된 VDI 환경에서 일관된 UI를 제공하므로 안정적이다.  
* **Tauri**:  
  * **장점**: 현저히 작은 번들 크기 21, 낮은 메모리 사용량 21, Rust 기반 백엔드(Python과 사이드카 또는 IPC로 연동 가능), 보안에 중점.  
  * **단점**: 시스템 웹뷰(WebView)에 의존하므로 20, VDI 환경의 웹뷰가 오래되었거나 제한적일 경우 UI 렌더링 불일치 또는 기능 제약 발생 가능. Rust 백엔드가 주력이므로 Python 중심 팀에게는 학습 곡선이 있을 수 있음 (핵심 로직은 Python으로 유지 가능). 초기 빌드 시간이 상대적으로 길 수 있음.21  
  * **VDI 적합성**: 작은 설치 공간은 VDI 환경에 매우 매력적이다. 오프라인 기능은 우수하나, 대상 VDI의 시스템 웹뷰 호환성 및 안정성 검증이 필수적이다.  
* **개별 웹 서버 (예: Flask/Django) \+ PyWebview/Pyloid**:  
  * **장점**: 백엔드를 Python으로 완전 제어 가능. PyWebview 22 사용 시 시스템 네이티브 웹뷰를 활용하여 매우 가벼운 GUI 구현 가능. Pyloid 22는 QtWebEngine 기반으로 Python 백엔드에 최적화된 Electron/Tauri 대안을 목표로 하며, 웹 기술(HTML/CSS/JS)을 사용한 정교한 UI 개발을 지원한다.  
  * **단점**: 데스크톱 애플리케이션과 유사한 경험을 제공하기 위한 추가적인 개발 작업 필요. PyWebview는 Tauri와 유사하게 시스템 웹뷰에 의존. Pyloid는 QtWebEngine을 번들링하므로 Electron과 유사하게 웹 엔진 의존성 문제를 해결하지만, Python 중심적이다.  
  * **VDI 적합성**: Python 기반 웹 서버를 PyInstaller로 OCR 백엔드와 함께 번들링하고, PyWebview 또는 Pyloid를 사용하면 실행 가능한 오프라인 솔루션이 될 수 있다. PyWebview(시스템 엔진)와 Pyloid(번들된 QtWebEngine) 간의 선택은 웹 엔진 의존성에 대한 Tauri/Electron 선택과 유사한 고려사항을 가진다.

### **5.2. 최적 웹뷰 기술 추천**

VDI의 오프라인 제약과 독립 실행형 애플리케이션 요구를 고려할 때, **Electron**은 내장된 Chromium으로 일관된 실행 환경을 보장하여 초기에는 더 안전한 선택일 수 있다. 이는 특히 VDI 환경의 시스템 웹뷰가 구형이거나 예측 불가능할 경우 중요하다.

그러나, 대상 VDI 환경의 시스템 웹뷰가 충분히 현대적이고 안정적임이 확인된다면, **Tauri**가 현저히 작은 번들 크기와 메모리 점유율 덕분에 훨씬 더 매력적인 선택이 될 것이다.21 이는 VDI 리소스 절약에 크게 기여할 수 있으므로, 대상 VDI 환경에 대한 사전 조사가 필수적이다.

Python 중심의 스택을 선호한다면, **Pyloid** 22와 같은 프레임워크가 OCR 백엔드와 함께 PyInstaller로 잘 패키징될 수 있다면 좋은 균형점을 제공할 수 있다. Pyloid는 QtWebEngine을 사용하여 웹 엔진을 번들링하므로 시스템 웹뷰 의존성 문제를 피하면서 Python 생태계 내에서 개발을 유지할 수 있다는 장점이 있다. Pyloid는 Python 개발자에게 Electron이나 Tauri보다 쉬운 통합과 Python 라이브러리 활용성을 제공하며, 특히 AI 기반 데스크톱 애플리케이션 구축에 최적화되어 있다고 주장한다.22

궁극적인 선택은 VDI 환경의 구체적인 사양, 개발팀의 기술 숙련도, 그리고 성능과 번들 크기 간의 트레이드오프를 종합적으로 고려하여 내려져야 한다. 현재로서는 VDI 시스템 웹뷰의 신뢰성이 확보된다면 Tauri를, 그렇지 않다면 Electron 또는 Pyloid(성숙도 및 패키징 용이성 검증 후)를 우선 고려하는 것이 합리적이다. 이러한 웹뷰 기술 선택은 단순히 UI를 결정하는 것을 넘어, 개발 워크플로우, 빌드 프로세스, 최종 사용자 경험, 그리고 VDI 환경에서의 애플리케이션 안정성에 큰 영향을 미치는 핵심 아키텍처 결정 사항이다.

다음 표는 VDI 배포를 위한 주요 웹뷰 기술들을 비교 분석한 것이다.

**표 5.1: VDI 배포를 위한 웹뷰 기술 비교**

| 기술 | 백엔드 언어 | 웹 엔진 | 번들 크기 (추정) | 메모리 사용량 (추정) | 오프라인 실행 | VDI 시스템 웹뷰 의존성 | 개발 노력/복잡도 | 장점 | 단점 |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| Electron | JavaScript (Node.js) | Chromium (내장) | 큼 (예: \~244MB 21) | 높음 (예: \~409MB 21) | 우수 | 없음 | 중간 | 성숙한 생태계, 플랫폼 간 일관성, 강력한 기능 | 번들 크기, 메모리 사용량, "무거움" |
| Tauri | Rust (주력) | 시스템 웹뷰 (OS 제공) | 매우 작음 (예: \~8.6MB 21) | 낮음 (예: \~172MB 21) | 우수 | 높음 | 중간\~높음 | 작은 번들, 낮은 리소스 사용, 보안 중점, Rust의 성능 활용 가능 | 시스템 웹뷰 호환성 문제, Rust 학습 곡선, 초기 빌드 시간 |
| Pyloid (+Python 서버) | Python | QtWebEngine (PySide6, 내장 가능) | 중간 | 중간 | 우수 | 낮음 (번들 시) | 중간 | Python 중심 개발, 웹 기술 UI, Electron/Tauri의 Python 대안 22 | 상대적으로 새로운 프레임워크, 커뮤니티/자료 부족 가능성, Qt 의존성 |
| PyWebview (+Python 서버) | Python | 시스템 웹뷰 (OS 제공) | 매우 작음 | 낮음 | 우수 | 높음 | 낮음\~중간 | 매우 가벼움, Python과의 간단한 통합 | 시스템 웹뷰 호환성 문제, 제한된 기능, 정교한 UI 개발 어려움 |

데이터 출처: 20 및 일반적인 기술 특성 기반

## **6\. Windsurf AI IDE를 활용한 종합 개발 로드맵**

### **6.1. 단계별 개발 접근 방식**

1. **0단계: 환경 설정 및 기술 숙지**  
   * Windsurf AI IDE 내 Python 개발 환경 구성.  
   * Python, PyTorch, OpenCV, Pillow, lmdb, natsort 등 핵심 라이브러리 설치.3  
   * Naver Clova 관련 GitHub 저장소(CRAFT 2,  
     deep-text-recognition-benchmark 3) 클론.  
   * 제공되는 데모 스크립트(demo.py 3,  
     test.py 2)를 사전 학습된 모델로 실행하여 기본 설정 및 사용법 검증.  
2. **1단계: 데이터셋 확보 및 준비**  
   * 다양한 형태의 한국 사업자등록증 샘플(PDF, JPG) 수집.  
   * PDF를 고품질 이미지로 변환하는 스크립트 개발.  
   * 이미지 어노테이션: 사업자등록증 내 모든 한국어 텍스트 또는 주요 필드 텍스트 전사.  
   * 어노테이션 결과를 create\_lmdb\_dataset.py 3 요구 형식(  
     gt.txt)에 맞춰 포맷팅.  
   * 학습 및 검증용 LMDB 데이터셋 생성.  
   * (선택 사항, 권장) 실제 데이터가 부족할 경우, SynthTIGER 1 또는 Donut의 SynthDoG 1를 활용한 한국어 합성 학습 데이터 생성 실험.  
3. **2단계: 기준 OCR 모델 구현 및 평가**  
   * 사전 학습된 CRAFT 모델을 사용한 텍스트 검출 파이프라인 구현.2  
   * deep-text-recognition-benchmark의 사전 학습된 모델(예: TPS-ResNet-BiLSTM-Attn)을 사용한 텍스트 인식 파이프라인 구현.3  
   * 검출 및 인식 모듈 통합.  
   * 준비된 한국 사업자등록증 테스트셋에 대한 기준 성능 평가 (원시 텍스트에 대한 문자 오류율(CER), 단어 오류율(WER) 측정).  
4. **3단계: 반복적 미세조정 및 성능 최적화**  
   * 선택된 텍스트 인식 모델을 구축된 커스텀 한국어 LMDB 데이터셋으로 미세조정 (train.py 활용 3).  
   * opt.character 파라미터를 사업자등록증에 나타나는 모든 필수 한국어 문자, 기호, 영문자, 숫자 등을 포함하도록 조정.  
   * 기준 성능이 부족할 경우, 벤치마크 저장소 내 다른 모델 아키텍처 실험.  
   * 반복적인 평가 및 개선. 이미지 전처리 기법(예: 명암 조절, 샤프닝, 이진화 최적화) 고도화.  
5. **4단계: 데이터 구조화 및 출력 모듈 개발**  
   * 인식된 원시 텍스트를 사업자등록증의 구조화된 필드(표 3.1 참조)로 매핑하는 후처리 로직 설계 및 구현.  
   * 데이터 유효성 검사 규칙 개발 (예: 사업자등록번호 형식 검사).  
   * 워드(python-docx 14) 및 엑셀(  
     openpyxl 또는 pandas 16) 내보내기 기능 구현.  
6. **5단계: 프론트엔드/웹뷰 통합**  
   * 사용자 인터페이스 개발 (기본 기능: 파일 업로드, OCR 처리 시작, 결과 표시, 내보내기 버튼).  
   * Python OCR 백엔드를 선택된 웹뷰 프레임워크(Electron, Tauri, 또는 Python 기반 솔루션)와 통합. 프론트엔드와 백엔드가 별도 프로세스일 경우 IPC(Inter-Process Communication) 설정 필요.  
7. **6단계: VDI 환경을 위한 패키징, 테스트 및 배포**  
   * PyInstaller 18 또는 선택된 대안을 사용하여 전체 애플리케이션(Python 백엔드, 모델, 웹뷰 컴포넌트)을 독립 실행형 번들로 패키징.  
   * VDI 환경을 모방한 클린 환경에서 번들된 애플리케이션 철저히 테스트.  
   * 실제 VDI 환경에 배포하고, 발생 가능한 경로, 권한, 의존성 누락 문제 해결 및 최종 테스트.

머신러닝 프로젝트의 특성상, 특히 새로운 언어와 문서 조합에 대한 OCR 성능은 초기에 예측하기 어렵다. 따라서 폭포수 모델보다는 위와 같은 단계별 접근과 반복적인 미세조정 및 평가가 필수적이다. 이는 불확실성을 관리하고 모델이 한국어 문서에 대한 정확도 요구사항을 충족하도록 보장하는 데 중요하다. 초기 단계에서 기준 모델 성능을 확인하고, 이를 바탕으로 목표 성능 달성을 위한 구체적인 개선 방향을 설정할 수 있다.

### **6.2. 주요 개발 언어 및 프레임워크**

* **백엔드 및 OCR**: Python (주 언어), PyTorch (Clova 모델 실행), OpenCV (이미지 처리), NumPy.  
* **프론트엔드 (웹뷰 UI)**: HTML, CSS, JavaScript. Electron/Tauri 사용 시 React, Vue, Svelte와 같은 JS 프레임워크를 고려할 수 있으나, 기능의 복잡도에 따라 순수 JavaScript로도 충분할 수 있다.  
* **패키징**: PyInstaller.

### **6.3. AI 지원 개발 (Windsurf AI IDE)을 위한 예시 프롬프트**

Windsurf AI IDE와 같은 AI 지원 개발 환경을 활용할 경우, 다음과 같은 구체적인 프롬프트를 통해 개발 효율성을 높일 수 있다. AI 도구의 효과는 프롬프트의 구체성과 AI가 Naver Clova와 같은 특수 라이브러리 및 모델의 컨텍스트를 얼마나 잘 이해하는지에 따라 달라질 수 있다. 개발자는 AI가 생성한 코드를 검토하고 수정할 수 있는 깊이 있는 이해를 갖추어야 한다.

* *"PyMuPDF 라이브러리를 사용하여 pdf\_path 변수에 지정된 PDF 파일의 첫 페이지를 PNG 이미지로 변환하고 output\_image\_path로 저장하는 Python 코드를 생성해 줘."*  
* *"OpenCV를 사용하여 이미지 경로를 입력받아 이미지를 로드하고, 5x5 커널로 가우시안 블러를 적용한 후 오츠 이진화를 수행하는 Python 함수를 만들어 줘. 이진화된 이미지를 반환해야 해."*  
* "clovaai/CRAFT-pytorch GitHub 저장소(2)의 코드 구조를 참조하여, 'craft\_mlt\_25k.pth' 경로의 사전 학습된 CRAFT 모델과 'craft\_refiner\_CTW1500.pth' 경로의 LinkRefiner 모델을 로드하고, 'input.jpg' 이미지에 대해 텍스트 검출을 수행한 후 경계 상자가 표시된 결과 이미지를 'output\_craft.jpg'로 저장하는 Python 스크립트를 작성해 줘."  
* "extracted\_data \= {'company\_name': '주식회사 예시', 'ceo\_name': '홍길동', 'address': '서울시 강남구'}와 같은 Python 딕셔너리와 {{COMPANY\_NAME}}, {{CEO\_NAME}}과 같은 플레이스홀더가 있는 'template.docx' 워드 템플릿 문서가 주어졌을 때, python-docx 라이브러리(14)를 사용하여 플레이스홀더를 딕셔너리 값으로 대체하고 'output.docx'로 저장하는 Python 코드를 생성해 줘."  
* *"'index.html'을 로드하는 브라우저 창을 생성하고, 'ocr-process-file' 및 'ocr-get-result' 이벤트에 대한 IPC 리스너를 설정하는 기본 Electron main.js 스크립트를 생성해 줘."*

### **6.4. OCR 데이터 추출 로직 상세 흐름도 (개념적)**

이미지 입력부터 구조화된 JSON/딕셔너리 출력까지의 OCR 데이터 추출 로직은 다음과 같은 세부 단계를 포함할 수 있다:

1. 입력 이미지 수신  
2. 이미지 전처리 (기울기 보정, 노이즈 제거, 이진화 등)  
3. CRAFT 모델을 이용한 텍스트 영역 검출 \-\> 경계 상자 목록 획득  
4. 각 경계 상자에 대해:  
   a. 원본 이미지에서 텍스트 영역 이미지 자르기  
   b. 텍스트 인식 모델로 자른 이미지 전달 \-\> 인식된 텍스트 문자열 획득  
5. 모든 인식된 텍스트 문자열과 해당 위치 정보(경계 상자) 수집  
6. 정보 추출 로직 적용:  
   a. 키워드(예: "상호", "대표자") 및 위치 기반 필드 식별  
   b. 정규 표현식을 이용한 특정 패턴(예: 사업자등록번호, 날짜) 추출  
   c. 추출된 정보와 사업자등록증 필드 매핑  
   d. 필요시 신뢰도 점수 기반 필터링 또는 보정  
7. 데이터 유효성 검사 (형식, 값의 범위 등)  
8. 오류 처리 및 로깅  
9. 최종 구조화된 데이터(JSON/딕셔너리) 생성

### **6.5. 자원 및 일정 고려사항 (개략적)**

본 프로젝트는 Python, 머신러닝/컴퓨터 비전, 그리고 잠재적으로 프론트엔드 기술에 능숙한 개발 인력을 필요로 하는 복잡한 과제이다. 단계별 예상 소요 기간은 팀 규모와 전문성에 따라 달라질 수 있으나, 대략적으로 다음과 같이 추정할 수 있다:

* 데이터셋 구축 및 준비: 1\~2개월  
* 기준 OCR 모델 구현: 1개월  
* 미세조정 및 성능 최적화: 2\~3개월  
* 데이터 구조화 및 출력 모듈 개발: 1개월  
* 웹뷰 통합 및 패키징: 1\~2개월  
* 테스트 및 안정화: 1개월

이러한 상세한 개발 로드맵은 본 프로젝트뿐만 아니라, 향후 유사한 문서 OCR 과제를 오픈소스 도구로 해결하고자 할 때 유용한 참조 템플릿이 될 수 있다.

다음 표는 프로젝트에 필요한 핵심 소프트웨어 의존성 및 라이브러리를 요약한 것이다.

**표 6.1: 핵심 소프트웨어 의존성 및 라이브러리**

| 라이브러리/프레임워크 | 프로젝트 내 목적 | 주요 버전 (중요시) | 라이선스 | 오프라인 사용 고려사항 |
| :---- | :---- | :---- | :---- | :---- |
| Python | 주 개발 언어 | 3.8+ 권장 | PSF License | PyInstaller로 번들링 시 포함 |
| PyTorch | Naver Clova 모델(CRAFT, 인식 모델) 실행 및 미세조정 | 최신 안정 버전 | BSD-style (확인 필요) | 모델 파일과 함께 PyInstaller로 번들링 필요, 크기 주의 |
| OpenCV (cv2) | 이미지 전처리, 후처리, 이미지 입출력 | 최신 안정 버전 | Apache 2.0 | PyInstaller로 번들링 시 의존성 포함 확인 필요 |
| Pillow (PIL Fork) | 이미지 처리 보조 | 최신 안정 버전 | HPND License | PyInstaller로 번들링 용이 |
| python-docx | 워드(.docx) 파일 생성 및 수정 | 최신 안정 버전 | MIT | PyInstaller로 번들링 용이 |
| openpyxl (또는 pandas) | 엑셀(.xlsx) 파일 생성 및 수정 | 최신 안정 버전 | MIT (openpyxl), BSD 3-Clause (pandas) | PyInstaller로 번들링 용이 |
| PyInstaller | Python 애플리케이션 패키징 (독립 실행 파일 생성) | 최신 안정 버전 | GPL (일부 조건부 상업적 허용, 확인 필요) | 오프라인 배포의 핵심 도구 |
| Electron / Tauri / Pyloid | 웹뷰 기반 데스크톱 UI 제공 | 최신 안정 버전 | MIT (Electron, Tauri), 확인 필요 (Pyloid) | Electron/Pyloid(Qt)은 엔진 포함, Tauri는 시스템 의존 |
| lmdb, natsort, nltk | deep-text-recognition-benchmark 의존 라이브러리 3 | 명시된 버전 확인 | 다양 (OpenLDAP, MIT 등) | PyInstaller로 번들링 시 포함 확인 필요 |

데이터 출처: 2 및 각 라이브러리 공식 문서 참조

## **7\. 핵심 성공 요인 및 전략적 권고 사항**

### **7.1. 한국어 텍스트 및 문서 레이아웃에 대한 정확성 및 강인성**

이는 프로젝트의 가장 중요한 성공 지표이다. 엄격한 테스트, 포괄적인 한국어 문자셋 처리(특수문자, 한자 포함 가능성 고려), 그리고 사업자등록증의 다양한 변형(오래된 양식, 스캔 품질 저하 등)에도 강인한 필드 매핑 로직 확보가 필수적이다.

* **권고 사항**: 고품질의 다양한 한국 사업자등록증 이미지 데이터셋 구축에 집중적인 투자가 필요하다. 필요하다면 전문적인 데이터 어노테이션 서비스를 활용하는 방안도 고려해야 한다.

### **7.2. VDI 환경에서의 성능**

OCR 처리는 자원 소모적인 작업일 수 있다. 모델과 코드를 속도 및 메모리 효율성 측면에서 최적화해야 한다.

* **권고 사항**: 목표 VDI와 유사한 환경에서 애플리케이션의 성능 프로파일링을 광범위하게 수행해야 한다. 웹뷰 프레임워크 및 기타 구성 요소 선택 시 VDI의 자원 한계를 염두에 두어야 한다.

### **7.3. 데이터 보안 및 개인 정보 보호**

사업자등록증에는 민감한 정보가 포함되어 있다. 오프라인 VDI 애플리케이션이므로 네트워크를 통한 데이터 유출 위험은 낮지만, 임시 파일 생성 및 로컬 데이터 처리 과정에서의 보안을 철저히 관리해야 한다.

* **권고 사항**: OCR 처리 중 생성되는 모든 임시 파일은 작업 완료 후 안전하게 삭제되도록 보장해야 한다.

### **7.4. 유지보수성 및 향후 확장성**

모듈화되고 문서화가 잘 된 코드는 장기적인 유지보수 및 기능 확장을 용이하게 한다.

* **권고 사항**: 모든 코드, 학습된 모델, 데이터셋에 대해 버전 관리를 철저히 수행해야 한다. 향후 OCR 모델 변경이나 특정 모듈의 업그레이드가 용이하도록 시스템을 설계해야 한다.

### **7.5. 사용자 경험 (UX)**

기반 기술이 복잡하더라도 최종 사용자가 접하는 애플리케이션은 사용하기 쉬워야 한다. 명확한 사용 안내, 직관적인 오류 메시지, 간결한 UI가 중요하다.

* **권고 사항**: 가능하다면 웹뷰 개발 초기 단계부터 최종 사용자 또는 대리 사용자를 참여시켜 사용성 테스트를 진행하는 것이 바람직하다.

### **7.6. 전략적 권고: 최소 기능 제품(MVP) 우선 개발**

초기에는 핵심 기능(일반적인 사업자등록증 양식의 주요 필드 OCR, 단일 포맷으로의 내보내기) 구현에 집중하여 최소 기능 제품(Minimum Viable Product, MVP)을 우선적으로 제공하는 전략이 유효하다. 이를 통해 프로젝트 위험을 관리하고 초기 사용자 피드백을 신속하게 반영하여 점진적으로 기능을 확장(다양한 PDF 변형 지원, 추가 필드 인식, 워드 *및* 엑셀 동시 지원 등)해 나갈 수 있다.

본 프로젝트의 성공은 Naver Clova의 최첨단 AI 기술 활용 능력과 데이터 준비, 후처리, VDI 패키징, UI 설계 등 세심한 엔지니어링 역량 간의 균형에 달려있다. 어느 한쪽도 소홀히 할 수 없다. 또한, Naver Cloud의 상용 API 대신 오픈소스 11를 사용하는 결정은 비용 절감과 완전한 통제권을 제공하지만, 개발, 유지보수, 정확도 개선의 모든 책임을 개발팀이 부담해야 함을 의미한다. 이는 사업 계획 수립 시 장기적인 운영 비용 및 자원 배분 관점에서 신중하게 고려되어야 할 전략적 선택이다.

## **8\. 결론 및 제언**

본 보고서는 Naver Clova 오픈소스 기술을 활용하여 한국 사업자등록증 OCR 프로그램을 개발하는 데 필요한 핵심 기술 요소, 개발 방법론, 배포 전략, 그리고 단계별 이행 계획을 심층적으로 분석하였다. 제안된 기술 스택과 개발 로드맵은 외부망 접근이 제한된 VDI 환경에서 안정적으로 운영될 수 있는 고품질 OCR 솔루션 구축을 목표로 한다.

**주요 결론은 다음과 같다:**

1. **기술적 실현 가능성**: Naver Clova의 CRAFT (텍스트 검출) 및 deep-text-recognition-benchmark (텍스트 인식) 오픈소스 모델은 본 프로젝트의 핵심 OCR 엔진 구축에 적합한 기반을 제공한다.1 이들 기술은 한국어 및 사업자등록증의 특정 레이아웃에 대한 커스터마이징 및 미세조정을 통해 높은 정확도를 달성할 잠재력을 가지고 있다.  
2. **커스터마이징의 중요성**: 범용 OCR 모델을 특정 문서(사업자등록증)와 특정 언어(한국어)에 최적화하는 과정이 프로젝트 성공의 관건이다. 이를 위해 고품질의 한국어 사업자등록증 데이터셋 구축, 합성 데이터 생성 도구(예: SynthTIGER 1) 활용, 그리고 텍스트 인식 모델의 정교한 미세조정이 필수적이다.  
3. **오프라인 VDI 배포**: PyInstaller와 같은 패키징 도구를 사용하여 모든 의존성을 포함하는 독립 실행형 애플리케이션을 생성함으로써 외부망 접근이 불가능한 VDI 환경에 성공적으로 배포할 수 있다.18 이 과정에서 복잡한 라이브러리(PyTorch, OpenCV) 및 모델 파일의 정확한 번들링이 중요하다.  
4. **웹뷰 기술 선택**: 사용자 인터페이스 제공을 위한 웹뷰 기술로는 Electron, Tauri, Pyloid 등이 고려될 수 있다.20 VDI 환경의 특성(시스템 웹뷰 안정성, 리소스 제약)과 개발팀의 기술 숙련도를 종합적으로 고려하여 최적의 솔루션을 선택해야 한다. 시스템 웹뷰의 신뢰성이 높다면 Tauri가, 그렇지 않다면 Electron 또는 Pyloid가 유리할 수 있다.  
5. **정보 추출 로직의 핵심 역할**: OCR을 통해 인식된 텍스트를 사업자등록증의 정형화된 필드에 정확히 매핑하는 후처리 및 정보 추출 로직의 완성도가 최종 결과물의 품질을 좌우한다. 사업자등록증의 표준화된 패턴 4은 규칙 기반 또는 템플릿 기반 접근을 효과적으로 만들 수 있다.

**전략적 제언은 다음과 같다:**

1. **데이터 중심 접근**: 고품질의 레이블된 한국 사업자등록증 데이터셋 확보에 최우선 순위를 두어야 한다. 데이터의 양과 질이 모델 성능을 결정짓는 가장 중요한 요소이다.  
2. **단계적 개발 및 MVP 우선**: 전체 기능을 한 번에 개발하기보다는 핵심 기능을 포함하는 MVP(최소 기능 제품)를 우선 개발하여 기술적 위험을 조기에 검증하고 사용자 피드백을 반영하는 반복적 개발 방식을 채택할 것을 권고한다.  
3. **철저한 테스트 및 성능 검증**: 개발 초기 단계부터 VDI 환경을 모방한 테스트 환경을 구축하고, 각 모듈 및 통합 시스템에 대한 철저한 기능 및 성능 테스트를 수행해야 한다. 특히 OCR 정확도, 처리 속도, 리소스 사용량 등을 중점적으로 관리해야 한다.  
4. **오픈소스 활용의 양면성 인지**: 오픈소스 기술 활용은 비용 절감과 유연성 확보라는 장점을 제공하지만, 유지보수, 업데이트, 기술 지원의 책임을 내부화해야 한다. 장기적인 관점에서 이러한 운영 부담을 사업 계획에 반영해야 한다.  
5. **Windsurf AI IDE 적극 활용**: AI 지원 개발 도구를 활용하여 코드 생성, 디버깅, 문서화 등의 개발 생산성을 향상시키되, 생성된 결과물에 대한 개발자의 면밀한 검토와 이해가 수반되어야 한다.

본 분석 보고서가 성공적인 사업자등록증 OCR 프로그램 개발 사업 계획 수립에 기여하고, 실질적인 개발 착수를 위한 명확한 기술적 방향을 제시할 수 있기를 기대한다.

#### **참고 자료**

1. Clova AI Research \- GitHub, 6월 13, 2025에 액세스, [https://github.com/clovaai](https://github.com/clovaai)  
2. yakhyo/clovaai-craft: CRAFT-PyTorch official ... \- GitHub, 6월 13, 2025에 액세스, [https://github.com/yakhyo/clovaai-craft](https://github.com/yakhyo/clovaai-craft)  
3. clovaai/deep-text-recognition-benchmark: Text recognition ... \- GitHub, 6월 13, 2025에 액세스, [https://github.com/clovaai/deep-text-recognition-benchmark](https://github.com/clovaai/deep-text-recognition-benchmark)  
4. South Korea Company Documents \- Systemday.com, 6월 13, 2025에 액세스, [https://www.systemday.com/south-korea-company-documents/](https://www.systemday.com/south-korea-company-documents/)  
5. Korean Business Registration Certificate: 4 Key Points to Master \- Behalf Korea, 6월 13, 2025에 액세스, [https://behalfkr.com/korean-business-registration-certificate/](https://behalfkr.com/korean-business-registration-certificate/)  
6. 8 Top Open-Source OCR Models Compared: A Complete Guide | Modal Blog, 6월 13, 2025에 액세스, [https://modal.com/blog/8-top-open-source-ocr-models-compared](https://modal.com/blog/8-top-open-source-ocr-models-compared)  
7. 나홀로 사업자 등록 신청서(개인사업자) 작성하기(+양식) \- 네이버 블로그, 6월 13, 2025에 액세스, [https://m.blog.naver.com/centraltax0/222929340943](https://m.blog.naver.com/centraltax0/222929340943)  
8. albumentations/albumentations/augmentations/transforms.py at main · albumentations-team/albumentations \- GitHub, 6월 13, 2025에 액세스, [https://github.com/albumentations-team/albumentations/blob/main/albumentations/augmentations/transforms.py](https://github.com/albumentations-team/albumentations/blob/main/albumentations/augmentations/transforms.py)  
9. xReniar/OCR-Dataset-Generator: Training data generator for Text Detection and Text Recognition for docTR, EasyOCR, MMOCR, PaddleOCR and other OCR tools. Offers support for data augmentation and label drawing. \- GitHub, 6월 13, 2025에 액세스, [https://github.com/xReniar/OCR-Dataset-Generator](https://github.com/xReniar/OCR-Dataset-Generator)  
10. deep-text-recognition-benchmark.ipynb \- Colab \- Google, 6월 13, 2025에 액세스, [https://colab.research.google.com/github/tgalkovskyi/deep-text-recognition-benchmark/blob/master/demo.ipynb](https://colab.research.google.com/github/tgalkovskyi/deep-text-recognition-benchmark/blob/master/demo.ipynb)  
11. CLOVA OCR overview, 6월 13, 2025에 액세스, [https://api.ncloud-docs.com/docs/en/ai-application-service-ocr](https://api.ncloud-docs.com/docs/en/ai-application-service-ocr)  
12. Hotsse/NAVER\_OCR\_API: 네이버 OCR API 연동 서비스 파일럿 프로젝트 \- GitHub, 6월 13, 2025에 액세스, [https://github.com/Hotsse/NAVER\_OCR\_API](https://github.com/Hotsse/NAVER_OCR_API)  
13. clovaai/CRAFT-pytorch: Official implementation of ... \- GitHub, 6월 13, 2025에 액세스, [https://github.com/clovaai/CRAFT-pytorch](https://github.com/clovaai/CRAFT-pytorch)  
14. Working with Tables — python-docx 1.1.2 documentation, 6월 13, 2025에 액세스, [https://python-docx.readthedocs.io/en/latest/user/tables.html](https://python-docx.readthedocs.io/en/latest/user/tables.html)  
15. Parsing a table data in dictionary format using docx \- Stack Overflow, 6월 13, 2025에 액세스, [https://stackoverflow.com/questions/41052551/parsing-a-table-data-in-dictionary-format-using-docx](https://stackoverflow.com/questions/41052551/parsing-a-table-data-in-dictionary-format-using-docx)  
16. pandas.DataFrame.to\_excel — pandas 2.3.0 documentation \- PyData |, 6월 13, 2025에 액세스, [https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.to\_excel.html](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.to_excel.html)  
17. Write a Pandas DataFrame to Excel using openpyxl \- GitHub Gist, 6월 13, 2025에 액세스, [https://gist.github.com/summerofgeorge/30e384da21665674c0d1b3dc38bafe4a](https://gist.github.com/summerofgeorge/30e384da21665674c0d1b3dc38bafe4a)  
18. Using PyInstaller to Easily Distribute Python Applications \- Real Python, 6월 13, 2025에 액세스, [https://realpython.com/pyinstaller-python/](https://realpython.com/pyinstaller-python/)  
19. Convert python script into an executable (.exe file) with pyinstaller module \- YouTube, 6월 13, 2025에 액세스, [https://www.youtube.com/watch?v=kxGXvpg0Zno](https://www.youtube.com/watch?v=kxGXvpg0Zno)  
20. Tauri vs. Electron – Real world application \- Hacker News, 6월 13, 2025에 액세스, [https://news.ycombinator.com/item?id=32550267](https://news.ycombinator.com/item?id=32550267)  
21. Tauri vs. Electron: performance, bundle size, and the real trade-offs \- Hopp, 6월 13, 2025에 액세스, [https://gethopp.app/blog/tauri-vs-electron](https://gethopp.app/blog/tauri-vs-electron)  
22. Pyloid: A Web-Based Desktop App Framwork \- Python Backend \- v0.14.2 : r/electronjs, 6월 13, 2025에 액세스, [https://www.reddit.com/r/electronjs/comments/1g9iamw/pyloid\_a\_webbased\_desktop\_app\_framwork\_python/](https://www.reddit.com/r/electronjs/comments/1g9iamw/pyloid_a_webbased_desktop_app_framwork_python/)  
23. Will Tauri feel snappier than an Electron app? Benchmarks are confusing me \- Reddit, 6월 13, 2025에 액세스, [https://www.reddit.com/r/tauri/comments/1kg5zb8/will\_tauri\_feel\_snappier\_than\_an\_electron\_app/](https://www.reddit.com/r/tauri/comments/1kg5zb8/will_tauri_feel_snappier_than_an_electron_app/)