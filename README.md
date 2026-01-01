# 🔐 Steganography Vault

이미지 속에 비밀 메시지를 숨기고 추출하는 웹 기반 스테가노그래피 도구입니다. AES-256 암호화와 LSB(Least Significant Bit) 스테가노그래피 기법을 결합하여 강력한 보안을 제공합니다.

![Steganography Vault](https://img.shields.io/badge/Security-AES--256-green)
![LSB](https://img.shields.io/badge/Steganography-LSB-blue)
![Client Side](https://img.shields.io/badge/Processing-Client--Side-orange)

## ✨ 주요 기능

### 🔒 메시지 숨기기 (Encoding)
- 이미지에 비밀 메시지를 보이지 않게 삽입
- AES-256 암호화로 메시지 보호
- 원본과 구분 불가능한 스테가노 이미지 생성
- PNG 포맷으로 무손실 저장

### 🔓 메시지 추출 (Decoding)
- 스테가노 이미지에서 숨겨진 메시지 추출
- 올바른 암호화 키로만 복호화 가능
- 추출된 메시지 클립보드 복사

### 🛡️ 보안 기능
- **AES-256 암호화**: 군사급 암호화 알고리즘
- **클라이언트 사이드 처리**: 모든 데이터가 브라우저 내에서 처리
- **서버 전송 없음**: 이미지와 메시지가 외부로 전송되지 않음
- **랜덤 키 생성**: 암호학적으로 안전한 랜덤 키 생성기

## 🚀 사용 방법

### 메시지 숨기기
1. **이미지 업로드**: PNG, JPEG, WebP 이미지를 업로드합니다
2. **메시지 입력**: 숨기고 싶은 비밀 메시지를 입력합니다
3. **암호화 키 설정**: 암호화에 사용할 키를 입력하거나 랜덤 생성합니다
4. **인코딩**: "메시지 숨기기" 버튼을 클릭합니다
5. **다운로드**: 생성된 스테가노 이미지를 PNG로 다운로드합니다

### 메시지 추출
1. **스테가노 이미지 업로드**: 메시지가 숨겨진 이미지를 업로드합니다
2. **복호화 키 입력**: 암호화 시 사용한 동일한 키를 입력합니다
3. **디코딩**: "메시지 추출하기" 버튼을 클릭합니다
4. **메시지 확인**: 추출된 원본 메시지를 확인합니다

## 🔬 작동 원리

### LSB (Least Significant Bit) 스테가노그래피
- 각 픽셀의 RGB 값은 0-255 사이의 정수입니다
- 최하위 비트(LSB)를 수정해도 색상 변화는 육안으로 감지 불가능합니다
- 예: `RGB(128, 64, 255)` → `RGB(129, 64, 254)` (거의 동일하게 보임)
- 각 픽셀당 3비트(R, G, B)의 데이터를 저장할 수 있습니다

### 데이터 구조
```
[32비트 헤더: 데이터 길이] + [암호화된 메시지] + [종료 구분자]
```

### 암호화 흐름
```
원본 메시지 → AES-256 암호화 → 이진 변환 → LSB 삽입 → PNG 저장
```

### 복호화 흐름
```
PNG 로드 → LSB 추출 → 이진 → 문자열 변환 → AES-256 복호화 → 원본 메시지
```

## ⚠️ 주의사항

### 이미지 포맷
- **PNG 권장**: 무손실 압축으로 데이터 보존
- **JPEG 주의**: 손실 압축으로 데이터 손상 가능
- **자동 변환**: JPEG 업로드 시 PNG로 저장됨

### 저장 용량
- 이미지 크기에 따라 저장 가능한 메시지 용량이 달라집니다
- 대략 `(가로 × 세로 × 3 × 0.7) / 8` 바이트의 데이터 저장 가능
- 예: 1920×1080 이미지 ≈ 약 500KB의 텍스트 저장 가능

### 보안
- 암호화 키를 분실하면 메시지 복구 불가능
- 강력한 암호화 키 사용 권장 (12자 이상, 특수문자 포함)
- 스테가노 이미지를 편집하면 숨겨진 데이터가 손상됨

## 🛠️ 기술 스택

| 분류 | 기술 |
|------|------|
| **프론트엔드** | HTML5, CSS3, Vanilla JavaScript |
| **암호화** | CryptoJS (AES-256) |
| **이미지 처리** | HTML5 Canvas API |
| **폰트** | Outfit, JetBrains Mono |
| **배포** | GitHub Pages |

## 📁 프로젝트 구조

```
Steganography/
├── index.html      # 메인 애플리케이션 (HTML/CSS/JS 통합)
├── README.md       # 프로젝트 문서
└── TECHNICAL.md    # 기술 상세 문서
```

## 🌐 브라우저 호환성

| 브라우저 | 지원 |
|----------|------|
| Chrome | ✅ |
| Firefox | ✅ |
| Safari | ✅ |
| Edge | ✅ |

## 📜 라이선스

MIT License - 자유롭게 사용, 수정, 배포 가능합니다.

## 🔗 관련 링크

- [CryptoJS Documentation](https://cryptojs.gitbook.io/docs/)
- [Canvas API - MDN](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API)
- [Web Crypto API - MDN](https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API)

---

Made with 💚 for privacy enthusiasts

