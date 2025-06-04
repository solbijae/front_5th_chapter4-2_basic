# 바닐라 JS 프로젝트 성능 개선
- 배포 url: https://front-5th-chapter4-2-basic.netlify.app/

## 성능 개선 보고서
### 초기 성능
- **PageSpeed Insights 보고서 링크:** 
  - **모바일:** https://pagespeed.web.dev/analysis/https-front-5th-chapter4-2-basic-netlify-app/mjziitvm5k?hl=ko&form_factor=mobile
  - **데스크톱:** https://pagespeed.web.dev/analysis/https-front-5th-chapter4-2-basic-netlify-app/mjziitvm5k?hl=ko&form_factor=desktop
- **보고서 전체 스크린샷**
  - **모바일**
    ![Image](https://github.com/user-attachments/assets/2d9b2d72-c845-47f6-9488-8f6f5ba8e191)
  - **데스크톱**
    ![Image](https://github.com/user-attachments/assets/4c27af8c-a720-42ad-880d-b654524749d3)

### 개선 후 성능
- **PageSpeed Insights 보고서 링크:** 
  - **모바일:** https://pagespeed.web.dev/analysis/https-front-5th-chapter4-2-basic-netlify-app/aucafrqeym?hl=ko&form_factor=mobile
  - **데스크톱:** https://pagespeed.web.dev/analysis/https-front-5th-chapter4-2-basic-netlify-app/aucafrqeym?hl=ko&form_factor=desktop
- **보고서 전체 스크린샷**
  - **모바일**
    ![Image](https://github.com/user-attachments/assets/6a26d2b8-8220-4eb5-9ddf-2f0e68c5e619)
  - **데스크톱**
    ![Image](https://github.com/user-attachments/assets/cb144af9-afb9-457e-b91e-5c6b913a174d)
- **개선 결과**
  **[모바일]**

  | 항목 | 개선 전 | 개선 후 | 개선 결과(%) |
  | --- | --- | --- | --- |
  | 성능 | 49 | 92 | 88% |
  | 접근성 | 81 | 95 | 17% |
  | 권장사항 | 93 | 93 | - |
  | 검색엔진 최적화 | 82 | 91 | 11% |

  **[데스크톱]**

  | 항목 | 개선 전 | 개선 후 | 개선 결과(%) |
  | --- | --- | --- | --- |
  | 성능 | 62 | 97 | 56% |
  | 접근성 | 81 | 95 | 17% |
  | 권장사항 | 96 | 96 | - |
  | 검색엔진 최적화 | 82 | 91 | 11% |

--- 
### 개선 세부 내용
#### 이미지 jpg/png → webp
- **cwebp** (Google의 공식 WebP 인코더)
    - 가장 안정적이고 널리 사용됨
    - 다양한 품질 설정 옵션
    - 명령줄 인터페이스로 쉬운 사용
- **사용법**
    - `brew install webp`
    - `cd images && for file in *.jpg; do cwebp -q 80 -m 4 "$file" -o "${file%.jpg}.webp"; done`
- **결과**
    
    
    | 이미지 | 변환 전 크기 | 변환 후 크기 | 감소 비율 |
    | --- | --- | --- | --- |
    | Hero_Desktop.jpg | 1.0MB | 188KB | 81% |
    | Hero_Mobile.jpg | 405KB | 63KB | 84% |
    | Hero_Tablet.jpg | 769KB | 123KB | 84% |
    | vr1.jpg | 52KB | 7.4KB | 86% |
    | vr2.jpg | 89KB | 14KB | 84% |
    | vr3.jpg | 75KB | 9.5KB | 87% |

#### lazy loading 적용
- **예시**
    
    ```jsx
    <picture>
    	<source srcset="images/vr2.webp" type="image/webp">
    	<img src="images/vr2.jpg" alt="product: Noise Cancelling Headphone" loading="lazy">
    </picture>
    ```
    
- **결과**
    1. 화면에 보이지 않는 이미지들은 지연 로딩
    2. 초기 페이지 로딩 시간이 개선
    3. 대역폭 사용량이 줄어듬

#### LCP 성능 개선
1. **LCP(Largest Contentful Paint)란?**
   - 페이지 로드 중에 뷰포트(화면)에 표시되는 가장 큰 콘텐츠 요소가 그려지는 시간
   - 사용자가 페이지의 주요 콘텐츠를 볼 수 있는 시점을 의미
   - 현재 사이트에서는 모바일 Hero 이미지가 LCP 요소로 확인됨

2. **현재 문제 상황**

   - LCP 시간: 5,020ms (목표: 2.5초 이내)
   - 주요 지연 요소:
   - 로드 지연: 2,290ms (46%)
   - 로드 시간: 1,500ms (30%)
   - 렌더링 지연: 480ms (10%)
   - TTFB: 750ms (15%)

3. **개선 작업**

    ```jsx
    <!-- 모바일 이미지 (LCP 요소) -->
    <picture>
        <source srcset="images/Hero_Mobile.webp" type="image/webp">
        <img class="mobile" src="images/Hero_Mobile.jpg" fetchpriority="high">
    </picture>

    <!-- 데스크톱/태블릿 이미지 -->
    <picture>
        <source srcset="images/Hero_Desktop.webp" type="image/webp">
        <img class="desktop" src="images/Hero_Desktop.jpg" loading="lazy" fetchpriority="low">
    </picture>
    <picture>
        <source srcset="images/Hero_Tablet.webp" type="image/webp">
        <img class="tablet" src="images/Hero_Tablet.jpg" loading="lazy" fetchpriority="low">
    </picture>
    ```

4. **최적화 이유와 방법**

   1. **모바일 이미지 최적화**
       - 이유: LCP 요소이므로 가장 먼저 로드되어야 함
       - 방법:
       - fetchpriority="high" 추가로 우선 로딩
       - loading="lazy" 제거로 지연 로딩 방지
   2. **데스크톱/태블릿 이미지 최적화**
       - 이유: 화면에 보이지 않을 수 있어 우선순위가 낮아도 됨
       - 방법:
       - fetchpriority="low" 추가로 우선순위 낮춤
       - loading="lazy" 유지로 필요할 때 로딩

#### 이미지 크기 적절하게 설정하기
- **방법**
    
    ```jsx
    cwebp -q 80 -m 4 -resize 1440 670 Hero_Desktop.jpg -o Hero_Desktop.webp && cwebp -q 80 -m 4 -resize 960 770 Hero_Tablet.jpg -o Hero_Tablet.webp && cwebp -q 80 -m 4 -resize 576 576 Hero_Mobile.jpg -o Hero_Mobile.webp && cwebp -q 80 -m 4 -resize 500 500 vr1.jpg -o vr1.webp && cwebp -q 80 -m 4 -resize 500 500 vr2.jpg -o vr2.webp && cwebp -q 80 -m 4 -resize 500 500 vr3.jpg -o vr3.webp
    ```
    
- **결과**
    
    
    | 이미지 | 변환 전 크기 | 변환 전 크기 | 감소 비율 |
    | --- | --- | --- | --- |
    | Hero_Desktop.jpg | 188KB | 113.2KB | 38% |
    | Hero_Mobile.jpg | 63KB | 37.9KB | 40% |
    | Hero_Tablet.jpg | 123KB | 74.3KB | 38% |
    | vr1.jpg | 7.4KB | 4.7KB | 36% |
    | vr2.jpg | 14KB | 9.0KB | 37% |
    | vr3.jpg | 9.5KB | 5.7KB | 40% |

#### 이미지 접근성 개선을 위한 alt 속성 추가
#### 폰트 로딩 최적화
#### 쿠키 동의 배너 스크립트 에러 해결
- **문제 상황**
    - 에러 메시지: `Cannot read properties of null (reading 'insertAdjacentElement')`
        - 쿠키 동의 배너가 정상적으로 표시되지 않는 문제 발생
- **원인 분석**
    - 쿠키 동의 스크립트가 DOM이 완전히 로드되기 전에 실행되어 발생
    - 스크립트가 head 태그 내에서 실행되어 DOM 요소에 접근할 수 없었음
- **해결 방법**
    1. 쿠키 동의 스크립트를 DOMContentLoaded 이벤트 이후에 실행되도록 수정
    2. 스크립트 위치를 body 태그 끝부분으로 이동
- **결과**
    - 쿠키 동의 배너가 정상적으로 표시됨
    - 스크립트 에러가 해결됨