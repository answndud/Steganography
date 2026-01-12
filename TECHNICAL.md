# 🔐 Steganography Vault - 기술 문서

이 문서는 웹 기반 이미지 스테가노그래피 도구의 구현 원리와 기술적 세부사항을 심층적으로 설명합니다. LSB(Least Significant Bit) 스테가노그래피 알고리즘, AES-256 암호화, Canvas API를 활용한 픽셀 조작 등 핵심 기술들을 다룹니다.

---

## 📚 목차

1. [스테가노그래피 이론](#1-스테가노그래피-이론)
2. [LSB 알고리즘 상세](#2-lsb-알고리즘-상세)
3. [이진 데이터 표현](#3-이진-데이터-표현)
4. [AES-256 암호화](#4-aes-256-암호화)
5. [Canvas API 픽셀 조작](#5-canvas-api-픽셀-조작)
6. [데이터 인코딩 프로토콜](#6-데이터-인코딩-프로토콜)
7. [디코딩 프로세스](#7-디코딩-프로세스)
8. [보안 분석](#8-보안-분석)
9. [성능 최적화](#9-성능-최적화)
10. [한계점 및 개선 방향](#10-한계점-및-개선-방향)

---

## 1. 스테가노그래피 이론

### 1.1 스테가노그래피란?

**스테가노그래피(Steganography)**는 그리스어로 "덮여진 글쓰기(covered writing)"를 의미하며, 비밀 메시지의 **존재 자체**를 숨기는 기술입니다. 암호화(Cryptography)와의 핵심 차이점:

| 구분 | 암호화 (Cryptography) | 스테가노그래피 (Steganography) |
|------|----------------------|-------------------------------|
| **목적** | 메시지 내용을 읽을 수 없게 함 | 메시지 존재 자체를 숨김 |
| **가시성** | 암호문이 보임 | 캐리어 미디어만 보임 |
| **의심 가능성** | 암호화된 것이 명확 | 일반 파일로 보임 |
| **공격 대상** | 암호 해독 시도 | 숨겨진 데이터 탐지 시도 |

### 1.2 이미지 스테가노그래피의 원리

디지털 이미지는 픽셀의 2차원 배열로 구성됩니다. 각 픽셀은 색상 값을 가지며, 이 값들의 미세한 변경은 **인간의 시각 시스템(HVS, Human Visual System)**으로는 감지할 수 없습니다.

```
원본 이미지        스테가노 이미지
[픽셀1][픽셀2]     [픽셀1'][픽셀2']
[픽셀3][픽셀4]  →  [픽셀3'][픽셀4']

RGB(128, 64, 255) → RGB(129, 64, 254)
육안으로는 동일하게 보임
```

### 1.3 캐리어, 페이로드, 스테고

스테가노그래피의 세 가지 핵심 요소:

1. **캐리어(Carrier/Cover)**: 비밀 데이터를 숨기는 호스트 미디어 (이미지)
2. **페이로드(Payload)**: 숨기려는 비밀 메시지
3. **스테고(Stego)**: 페이로드가 삽입된 결과물

```
Cover Image + Secret Message = Stego Image
    (캐리어)     (페이로드)      (스테고)
```

---

## 2. LSB 알고리즘 상세

### 2.1 LSB(Least Significant Bit) 개념

디지털 이미지의 각 픽셀 채널(R, G, B)은 8비트(0-255)로 표현됩니다. LSB는 이 8비트 중 **가장 오른쪽 비트(최하위 비트)**를 의미합니다.

```
십진수 128의 이진 표현:
┌───┬───┬───┬───┬───┬───┬───┬───┐
│ 1 │ 0 │ 0 │ 0 │ 0 │ 0 │ 0 │ 0 │
└───┴───┴───┴───┴───┴───┴───┴───┘
  ↑                           ↑
 MSB                         LSB
(128)                        (1)
```

### 2.2 LSB 변경의 영향

LSB를 변경하면 값이 최대 ±1만 변합니다:

| 원본 값 | 원본 이진 | 변경 후 이진 | 변경 후 값 | 차이 |
|---------|-----------|--------------|------------|------|
| 128 | 10000000 | 10000001 | 129 | +1 |
| 255 | 11111111 | 11111110 | 254 | -1 |
| 100 | 01100100 | 01100101 | 101 | +1 |

**색상 차이 시각화**:
```
원본 RGB(128, 64, 200)     ████████
변경 RGB(129, 65, 201)     ████████
                          ↑ 구분 불가능
```

### 2.3 LSB 삽입 알고리즘

```javascript
/**
 * LSB 삽입의 핵심 연산
 * 
 * 비트 연산 설명:
 * - AND 0xFE (11111110): LSB를 0으로 클리어
 * - OR bit: 원하는 비트값(0 또는 1)을 LSB에 설정
 */
function setLSB(pixelValue, bit) {
    return (pixelValue & 0xFE) | bit;
}

// 예시
// 원본: 128 (10000000)
// bit: 1
// 결과: (10000000 & 11111110) | 1 = 10000000 | 1 = 10000001 = 129
```

### 2.4 비트 연산 상세

#### AND 연산 (& 0xFE)
```
  10000111  (135)
& 11111110  (0xFE = 254)
──────────
  10000110  (134) ← LSB가 0으로 클리어됨
```

#### OR 연산 (| bit)
```
  10000110  (134)  ← LSB 클리어된 상태
| 00000001  (1)    ← 삽입할 비트
──────────
  10000111  (135)  ← 비트 1이 LSB에 삽입됨
```

### 2.5 구현 코드

```javascript
/**
 * 이미지 데이터에 비밀 데이터를 LSB 방식으로 삽입
 * 
 * @param {ImageData} imageData - Canvas에서 가져온 픽셀 데이터
 * @param {string} data - 숨길 데이터 (이미 암호화된 상태)
 * @returns {ImageData} - 수정된 이미지 데이터
 */
function encodeDataInImage(imageData, data) {
    const pixels = imageData.data;  // Uint8ClampedArray [R,G,B,A,R,G,B,A,...]
    const binary = stringToBinary(data + DELIMITER);
    
    // 32비트 헤더: 데이터 길이 저장
    const lengthBinary = binary.length.toString(2).padStart(32, '0');
    const fullData = lengthBinary + binary;
    
    // 용량 검사
    const maxBits = Math.floor(pixels.length * 0.75); // Alpha 채널 제외
    if (fullData.length > maxBits) {
        throw new Error('데이터가 이미지 용량을 초과합니다.');
    }
    
    let bitIndex = 0;
    
    for (let i = 0; i < pixels.length && bitIndex < fullData.length; i++) {
        // Alpha 채널(투명도)은 건너뜀 - 매 4번째 바이트
        if ((i + 1) % 4 === 0) continue;
        
        // LSB 수정
        const bit = parseInt(fullData[bitIndex], 10);
        pixels[i] = (pixels[i] & 0xFE) | bit;
        bitIndex++;
    }
    
    return imageData;
}
```

### 2.6 왜 Alpha 채널을 사용하지 않는가?

```javascript
// ImageData.data 구조: [R, G, B, A, R, G, B, A, ...]
//                       0  1  2  3  4  5  6  7  ...

// Alpha(투명도) 채널을 수정하지 않는 이유:
// 1. PNG의 투명도 처리 방식에 따라 데이터 손실 가능
// 2. 투명도가 있는 이미지에서 시각적 차이 발생 가능
// 3. 일부 뷰어/편집기가 Alpha 값을 정규화함
```

---

## 3. 이진 데이터 표현

### 3.1 문자열 → 이진 변환

JavaScript에서 문자열을 이진 데이터로 변환하는 과정:

```javascript
/**
 * 문자열을 UTF-8 이진 표현으로 변환
 * 
 * UTF-8 인코딩 규칙:
 * - ASCII (0-127): 1바이트
 * - 한글, 이모지 등: 2-4바이트
 */
function stringToBinary(str) {
    const encoder = new TextEncoder();  // UTF-8 인코더
    const bytes = encoder.encode(str);  // Uint8Array
    
    let binary = '';
    for (const byte of bytes) {
        // 각 바이트를 8비트 이진 문자열로 변환
        binary += byte.toString(2).padStart(8, '0');
    }
    return binary;
}

// 예시: "Hi" → UTF-8 바이트 [72, 105] → "0100100001101001"
// 예시: "안" → UTF-8 바이트 [236, 149, 136] → "111011001001010110001000"
```

### 3.2 이진 → 문자열 변환

```javascript
/**
 * 이진 문자열을 원본 텍스트로 복원
 */
function binaryToString(binary) {
    const bytes = [];
    
    // 8비트씩 분할하여 바이트 배열 생성
    for (let i = 0; i < binary.length; i += 8) {
        const byte = binary.slice(i, i + 8);
        if (byte.length === 8) {
            bytes.push(parseInt(byte, 2));
        }
    }
    
    const decoder = new TextDecoder();  // UTF-8 디코더
    return decoder.decode(new Uint8Array(bytes));
}
```

### 3.3 UTF-8 인코딩 상세

```
문자 'A' (ASCII):
코드포인트: U+0041 (65)
UTF-8:     01000001 (1바이트)

문자 '한':
코드포인트: U+D55C (54620)
UTF-8:     11101101 10010101 10011100 (3바이트)

문자 '😀':
코드포인트: U+1F600 (128512)
UTF-8:     11110000 10011111 10011000 10000000 (4바이트)
```

---

## 4. AES-256 암호화

### 4.1 AES(Advanced Encryption Standard) 개요

AES는 미국 국가표준기술연구소(NIST)가 채택한 대칭키 블록 암호 알고리즘입니다.

| 속성 | 값 |
|------|-----|
| 블록 크기 | 128비트 (16바이트) |
| 키 길이 | 128/192/256비트 |
| 라운드 수 | 10/12/14 (키 길이에 따라) |
| 알고리즘 유형 | 대칭키 블록 암호 |

### 4.2 CryptoJS를 통한 AES 구현

```javascript
/**
 * AES-256 암호화
 * 
 * CryptoJS 내부 동작:
 * 1. 키 파생: PBKDF2 알고리즘으로 비밀번호에서 256비트 키 생성
 * 2. IV 생성: 랜덤 초기화 벡터 생성
 * 3. 패딩: PKCS7 패딩으로 블록 크기 맞춤
 * 4. 암호화: CBC 모드로 AES 암호화 수행
 */
function encryptAES(plainText, password) {
    // CryptoJS가 내부적으로 처리:
    // - salt 생성 및 저장
    // - PBKDF2로 키 파생
    // - 랜덤 IV 생성
    // - CBC 모드 암호화
    return CryptoJS.AES.encrypt(plainText, password).toString();
}

// 결과 형식 (Base64 인코딩):
// "U2FsdGVkX1+..." 
// ↑ "Salted__" 매직 바이트 + salt + 암호문
```

### 4.3 키 파생 과정

```javascript
/*
 * PBKDF2 (Password-Based Key Derivation Function 2)
 * 
 * 비밀번호 → 암호화 키 변환 과정
 * 
 * 입력:
 *   - password: 사용자 입력 비밀번호
 *   - salt: 8바이트 랜덤 값 (레인보우 테이블 공격 방지)
 *   - iterations: 반복 횟수 (기본값: 1000)
 *   - keySize: 출력 키 크기 (256비트)
 * 
 * 출력:
 *   - 256비트 AES 키
 *   - 128비트 IV (초기화 벡터)
 */

// 간략화된 PBKDF2 의사 코드
function deriveKey(password, salt) {
    let key = password + salt;
    for (let i = 0; i < 1000; i++) {
        key = HMAC-SHA256(key, password);
    }
    return key.slice(0, 32);  // 256비트
}
```

### 4.4 CBC 모드 동작

```
평문 블록:   P1    P2    P3    P4
             ↓     ↓     ↓     ↓
XOR with:   IV ←─ C1 ←─ C2 ←─ C3
             ↓     ↓     ↓     ↓
AES 암호화:  E     E     E     E
             ↓     ↓     ↓     ↓
암호문:     C1    C2    C3    C4

CBC = Cipher Block Chaining
각 블록이 이전 암호문 블록과 XOR되어 체인 형성
→ 동일한 평문 블록도 다른 암호문 생성
```

### 4.5 복호화 과정

```javascript
/**
 * AES-256 복호화
 */
function decryptAES(cipherText, password) {
    try {
        const bytes = CryptoJS.AES.decrypt(cipherText, password);
        const decrypted = bytes.toString(CryptoJS.enc.Utf8);
        
        if (!decrypted) {
            throw new Error('복호화 실패');
        }
        
        return decrypted;
    } catch (e) {
        throw new Error('잘못된 키이거나 데이터가 손상되었습니다.');
    }
}
```

### 4.6 암호화 강도 분석

```
AES-256 무차별 대입 공격(Brute Force):
- 가능한 키 조합: 2^256 ≈ 1.16 × 10^77
- 초당 10^18 (엑사) 번 시도 시: 3.67 × 10^51 년 소요
- 우주 나이(138억 년) × 10^42 배

→ 현재 기술로는 실질적으로 해독 불가능
```

---

## 5. Canvas API 픽셀 조작

### 5.1 Canvas 2D Context 설정

```javascript
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d', { 
    willReadFrequently: true  // 픽셀 읽기 최적화 힌트
});
```

### 5.2 이미지 로딩 및 픽셀 데이터 추출

```javascript
/**
 * 이미지를 Canvas에 그리고 픽셀 데이터 추출
 */
function loadImageData(image) {
    // Canvas 크기를 이미지에 맞춤
    canvas.width = image.width;
    canvas.height = image.height;
    
    // 이미지 그리기
    ctx.drawImage(image, 0, 0);
    
    // 픽셀 데이터 추출
    // ImageData { width, height, data: Uint8ClampedArray }
    return ctx.getImageData(0, 0, canvas.width, canvas.height);
}
```

### 5.3 ImageData 구조

```javascript
/*
 * ImageData.data는 1차원 Uint8ClampedArray
 * 
 * 배열 구조:
 * [R₀, G₀, B₀, A₀, R₁, G₁, B₁, A₁, R₂, G₂, B₂, A₂, ...]
 *  ↑───픽셀 0───↑   ↑───픽셀 1───↑   ↑───픽셀 2───↑
 * 
 * 인덱스 계산:
 * - 픽셀 (x, y)의 R: (y * width + x) * 4 + 0
 * - 픽셀 (x, y)의 G: (y * width + x) * 4 + 1
 * - 픽셀 (x, y)의 B: (y * width + x) * 4 + 2
 * - 픽셀 (x, y)의 A: (y * width + x) * 4 + 3
 */

// 예: 800x600 이미지
// - 총 픽셀 수: 480,000
// - 배열 길이: 1,920,000 (480,000 × 4)
// - 저장 가능 비트: 1,440,000 (Alpha 제외, 480,000 × 3)
// - 저장 가능 바이트: ~180,000 (180KB)
```

### 5.4 픽셀 수정 및 적용

```javascript
// 수정된 데이터를 Canvas에 다시 적용
ctx.putImageData(modifiedImageData, 0, 0);

// Canvas를 PNG Blob으로 변환
canvas.toBlob((blob) => {
    // Blob 처리
}, 'image/png');

// 또는 Data URL로 변환
const dataURL = canvas.toDataURL('image/png');
// "data:image/png;base64,iVBORw0KGgo..."
```

### 5.5 왜 PNG를 사용하는가?

```
JPEG (손실 압축):
┌─────────────────────────────────────────────────┐
│ 원본 픽셀 → DCT 변환 → 양자화(손실) → 허프만 코딩 │
└─────────────────────────────────────────────────┘
  ↓
  LSB 데이터가 양자화 과정에서 손상됨

PNG (무손실 압축):
┌─────────────────────────────────────────────────┐
│ 원본 픽셀 → 필터링 → DEFLATE 압축 (무손실)      │
└─────────────────────────────────────────────────┘
  ↓
  LSB 데이터가 완벽하게 보존됨
```

---

## 6. 데이터 인코딩 프로토콜

### 6.1 데이터 구조 설계

```
┌──────────────────────────────────────────────────────────────┐
│                    스테가노 데이터 구조                        │
├────────────────┬─────────────────────────┬──────────────────┤
│   헤더 (32bit) │    암호화된 페이로드     │   종료 구분자     │
│  데이터 길이    │  AES-256 암호문 (Base64) │ "###STEGO_END###"│
└────────────────┴─────────────────────────┴──────────────────┘

총 비트 = 32 + (암호문 길이 × 8) + (구분자 길이 × 8)
```

### 6.1.1 메타데이터 헤더 (v1)

현재 구현은 페이로드 앞에 간단한 메타데이터 헤더를 추가합니다. 디코딩 시 숨김 설정(LSB, 채널, 랜덤 여부)을 확인하는 용도로 사용됩니다.

```
STEGO1|{"v":1,"bits":1,"channels":"RGB","randomize":false}|<암호문>
```

> 랜덤 삽입 시드는 보안상 저장하지 않으며, 디코딩 시 수동 입력이 필요합니다.

### 6.2 헤더 설계

```javascript
// 32비트 헤더로 최대 2^32 = 약 40억 비트 = 약 500MB 표현 가능
const HEADER_LENGTH = 32;

function createHeader(dataLength) {
    // 데이터 길이를 32비트 이진 문자열로 변환
    return dataLength.toString(2).padStart(HEADER_LENGTH, '0');
}

// 예: 1000비트 데이터
// → "00000000000000000000001111101000"
```

### 6.3 종료 구분자 (Delimiter)

```javascript
const DELIMITER = '###STEGO_END###';

// 구분자의 역할:
// 1. 데이터 끝을 명확히 표시
// 2. 디코딩 시 유효한 데이터 범위 확인
// 3. 손상된 데이터 탐지

// 구분자가 없으면?
// → 이미지의 모든 픽셀에서 LSB를 읽어야 함
// → 노이즈 데이터가 포함될 수 있음
```

### 6.4 전체 인코딩 흐름

```javascript
async function encodeMessage(image, message, password) {
    // 1. 메시지 암호화
    const encrypted = CryptoJS.AES.encrypt(message, password).toString();
    // 예: "U2FsdGVkX1+8vQ4Z7..."
    
    // 2. 페이로드 생성 (암호문 + 구분자)
    const payload = encrypted + DELIMITER;
    
    // 3. 이진 변환
    const binaryPayload = stringToBinary(payload);
    // 예: "01010101001100..."
    
    // 4. 헤더 생성
    const header = binaryPayload.length.toString(2).padStart(32, '0');
    
    // 5. 최종 데이터
    const fullData = header + binaryPayload;
    
    // 6. 이미지에 삽입
    const imageData = getImageData(image);
    embedDataInPixels(imageData, fullData);
    
    return imageData;
}
```

### 6.5 픽셀 순회 및 데이터 삽입

```javascript
function embedDataInPixels(imageData, binaryData, settings) {
    const pixels = imageData.data;
    let bitIndex = 0;
    
    // 픽셀 순회 패턴 시각화 (왼쪽→오른쪽, 위→아래)
    // ┌─────────────────────┐
    // │ P0→ P1→ P2→ P3→ P4│ 행 0
    // │ P5→ P6→ P7→ P8→ P9│ 행 1
    // │ ...                 │
    // └─────────────────────┘
    
    // settings.bitsPerChannel, settings.channels, settings.randomize에 따라
    // 채널 선택/비트 수/삽입 순서를 변경할 수 있음
}
```

---

## 7. 디코딩 프로세스

### 7.1 전체 디코딩 흐름

```javascript
async function decodeMessage(image, password) {
    // 1. 이미지에서 픽셀 데이터 추출
    const imageData = getImageData(image);
    const pixels = imageData.data;
    
    // 2. 헤더 읽기 (처음 32비트)
    const header = extractBits(pixels, 0, 32);
    const dataLength = parseInt(header, 2);
    
    // 3. 페이로드 읽기
    const payload = extractBits(pixels, 32, dataLength);
    
    // 4. 이진 → 문자열 변환
    const encrypted = binaryToString(payload);
    
    // 5. 구분자 확인 및 제거
    const delimiterIndex = encrypted.indexOf(DELIMITER);
    if (delimiterIndex === -1) {
        throw new Error('유효한 스테가노 데이터가 아닙니다.');
    }
    const cipherText = encrypted.substring(0, delimiterIndex);
    
    // 6. 복호화
    const plainText = decryptAES(cipherText, password);
    
    return plainText;
}
```

### 7.2 비트 추출 함수

```javascript
function extractBits(pixels, startBit, count) {
    let binary = '';
    let pixelIndex = 0;
    let bitsRead = 0;
    let bitsSkipped = 0;
    
    // startBit까지 건너뛰기
    while (bitsSkipped < startBit) {
        if ((pixelIndex + 1) % 4 !== 0) {  // Alpha 채널 제외
            bitsSkipped++;
        }
        pixelIndex++;
    }
    
    // count개의 비트 읽기
    while (bitsRead < count && pixelIndex < pixels.length) {
        if ((pixelIndex + 1) % 4 !== 0) {
            binary += (pixels[pixelIndex] & 1).toString();
            bitsRead++;
        }
        pixelIndex++;
    }
    
    return binary;
}
```

### 7.3 LSB 추출 연산

```javascript
// LSB 추출: AND 연산으로 마지막 비트만 가져옴
const lsb = pixelValue & 1;

// 예시:
// 129 (10000001) & 1 (00000001) = 1
// 128 (10000000) & 1 (00000001) = 0
```

---

## 8. 보안 분석

### 8.1 보안 계층 구조

```
┌─────────────────────────────────────────────────────────┐
│                    보안 계층                            │
├─────────────────────────────────────────────────────────┤
│ Layer 3: 스테가노그래피 (데이터 존재 은닉)              │
│   - LSB 삽입으로 시각적 감지 방지                       │
│   - 통계적 분석 없이는 데이터 존재 확인 어려움          │
├─────────────────────────────────────────────────────────┤
│ Layer 2: AES-256 암호화 (데이터 내용 보호)              │
│   - 무차별 대입 실질적 불가능                           │
│   - 키 없이 내용 해독 불가                              │
├─────────────────────────────────────────────────────────┤
│ Layer 1: 클라이언트 사이드 처리 (전송 보안)             │
│   - 서버로 데이터 전송 없음                             │
│   - 네트워크 감청 위험 없음                             │
└─────────────────────────────────────────────────────────┘
```

### 8.2 공격 벡터 분석

#### 1) 시각적 공격 (Visual Attack)
```
공격: 원본과 스테고 이미지를 시각적으로 비교
방어: LSB 변경은 ±1의 미세한 차이 → 육안 구분 불가
위험도: ★☆☆☆☆ (매우 낮음)
```

#### 2) 통계적 공격 (Statistical Attack)
```
공격: 픽셀 값 분포의 통계적 이상 탐지
  - Chi-square 분석
  - RS 스테가노분석
  - 샘플 페어 분석

방어 한계: 대량의 데이터 삽입 시 통계적 이상 발생 가능
권장: 이미지 용량의 50% 이하만 사용

위험도: ★★★☆☆ (중간)
```

#### 3) 암호 분석 공격 (Cryptanalysis)
```
공격: AES-256 암호화 해독 시도
방어: 
  - 2^256 키 공간
  - PBKDF2 키 파생으로 약한 비밀번호도 강화
위험도: ★☆☆☆☆ (매우 낮음, 강력한 키 사용 시)
```

#### 4) 사회공학적 공격
```
공격: 사용자를 속여 키를 획득
방어: 기술적 보호 범위 외
권장: 키 관리 교육, 키 공유 시 별도 채널 사용
위험도: ★★★★☆ (높음)
```

### 8.3 키 강도 분석

```javascript
function analyzePasswordStrength(password) {
    let score = 0;
    
    // 길이 점수
    if (password.length >= 8) score++;
    if (password.length >= 12) score++;
    if (password.length >= 16) score++;
    
    // 복잡성 점수
    if (/[a-z]/.test(password)) score++;
    if (/[A-Z]/.test(password)) score++;
    if (/[0-9]/.test(password)) score++;
    if (/[^a-zA-Z0-9]/.test(password)) score++;
    
    // 엔트로피 추정
    let charsetSize = 0;
    if (/[a-z]/.test(password)) charsetSize += 26;
    if (/[A-Z]/.test(password)) charsetSize += 26;
    if (/[0-9]/.test(password)) charsetSize += 10;
    if (/[^a-zA-Z0-9]/.test(password)) charsetSize += 32;
    
    const entropy = Math.log2(Math.pow(charsetSize, password.length));
    
    return { score, entropy };
}

/*
 * 엔트로피 해석:
 * - < 28 비트: 매우 약함
 * - 28-35 비트: 약함
 * - 36-59 비트: 적당함
 * - 60-127 비트: 강함
 * - 128+ 비트: 매우 강함
 */
```

### 8.4 랜덤 키 생성

```javascript
/**
 * 암호학적으로 안전한 랜덤 키 생성
 * Web Crypto API 사용 (Math.random()보다 안전)
 */
function generateSecureKey(length = 16) {
    const array = new Uint8Array(length);
    window.crypto.getRandomValues(array);
    
    // 16진수 문자열로 변환 (32자 = 128비트)
    return Array.from(array, byte => 
        byte.toString(16).padStart(2, '0')
    ).join('');
}

// 예: "a7f3c92e1d8b4502f6e9c3a8d1b7e4f2"
```

---

## 9. 성능 최적화

### 9.1 대용량 이미지 처리

```javascript
// 문제: 큰 이미지에서 getImageData가 메인 스레드를 블록

// 해결 1: 청크 단위 처리
async function processInChunks(imageData, chunkSize = 100000) {
    const pixels = imageData.data;
    const totalChunks = Math.ceil(pixels.length / chunkSize);
    
    for (let i = 0; i < totalChunks; i++) {
        const start = i * chunkSize;
        const end = Math.min(start + chunkSize, pixels.length);
        
        // 청크 처리
        processChunk(pixels, start, end);
        
        // UI 업데이트를 위한 양보
        await new Promise(resolve => setTimeout(resolve, 0));
        
        // 진행률 표시
        updateProgress((i + 1) / totalChunks * 100);
    }
}
```

### 9.2 Web Worker 활용 (고급)

```javascript
// main.js
const worker = new Worker('stego-worker.js');

worker.postMessage({
    type: 'encode',
    imageData: imageData,
    data: encryptedData
});

worker.onmessage = (e) => {
    const modifiedImageData = e.data;
    ctx.putImageData(modifiedImageData, 0, 0);
};

// stego-worker.js
self.onmessage = (e) => {
    const { type, imageData, data } = e.data;
    
    if (type === 'encode') {
        const result = encodeDataInImage(imageData, data);
        self.postMessage(result);
    }
};
```

### 9.3 메모리 관리

```javascript
// Canvas 크기 제한
const MAX_DIMENSION = 4096;

function resizeIfNeeded(image) {
    const scale = Math.min(
        MAX_DIMENSION / image.width,
        MAX_DIMENSION / image.height,
        1  // 원본보다 크게 하지 않음
    );
    
    if (scale < 1) {
        canvas.width = image.width * scale;
        canvas.height = image.height * scale;
        ctx.drawImage(image, 0, 0, canvas.width, canvas.height);
    }
}
```

### 9.4 성능 측정

```javascript
function measurePerformance(operation, ...args) {
    const start = performance.now();
    const result = operation(...args);
    const end = performance.now();
    
    console.log(`${operation.name}: ${(end - start).toFixed(2)}ms`);
    return result;
}

// 사용
measurePerformance(encodeDataInImage, imageData, binaryData);
```

---

## 10. 한계점 및 개선 방향

### 10.1 현재 구현의 한계

| 한계 | 설명 | 개선 방안 |
|------|------|----------|
| **통계적 탐지** | 대량 데이터 삽입 시 탐지 가능 | 랜덤 픽셀 선택, 노이즈 추가 |
| **JPEG 지원** | 손실 압축으로 데이터 손상 | PNG 전용 또는 JPEG 저항 기법 |
| **용량 제한** | 이미지 크기에 의존 | 다중 이미지 분산 저장 |
| **단일 채널** | RGB만 사용 | DCT, DWT 기반 방법 |

### 10.2 고급 스테가노그래피 기법

#### 1) 랜덤 픽셀 선택
```javascript
// 의사 난수 생성기로 픽셀 순서 섞기
function getPixelOrder(seed, count) {
    const order = Array.from({ length: count }, (_, i) => i);
    const rng = new SeededRandom(seed);
    
    // Fisher-Yates 셔플
    for (let i = order.length - 1; i > 0; i--) {
        const j = Math.floor(rng.next() * (i + 1));
        [order[i], order[j]] = [order[j], order[i]];
    }
    
    return order;
}
```

#### 2) 다중 비트 삽입
```javascript
// 2-LSB 삽입 (용량 2배, 탐지 위험 증가)
function set2LSB(value, bits) {
    return (value & 0xFC) | bits;  // 마지막 2비트 수정
}
```

#### 3) 주파수 영역 스테가노그래피
```
공간 영역 (현재): 픽셀 값 직접 수정
주파수 영역: DCT/DWT 계수 수정 → JPEG 저항성 향상
```

### 10.3 향후 개선 사항

```
┌─────────────────────────────────────────────────────────┐
│                     로드맵                              │
├─────────────────────────────────────────────────────────┤
│ v1.1: 다중 이미지 분산 저장                             │
│ v1.2: 키 파일 지원 (비밀번호 + 키 파일)                 │
│ v1.3: 오디오/비디오 스테가노그래피                      │
│ v2.0: 주파수 영역 기반 JPEG 지원                        │
│ v2.1: 스테가노분석 탐지 회피 기법                       │
└─────────────────────────────────────────────────────────┘
```

---

## 부록: 주요 상수 및 설정

```javascript
// 상수 정의
const DELIMITER = '###STEGO_END###';     // 데이터 종료 마커
const HEADER_LENGTH = 32;                 // 길이 헤더 비트 수
const MAX_FILE_SIZE = 10 * 1024 * 1024;   // 최대 파일 크기 (10MB)
const CAPACITY_SAFETY = 0.7;              // 안전 용량 비율 (70%)

// 용량 계산
function calculateCapacity(width, height) {
    const totalPixels = width * height;
    const bitsAvailable = totalPixels * 3 * CAPACITY_SAFETY; // RGB만
    const bytesAvailable = Math.floor(bitsAvailable / 8);
    return bytesAvailable;
}

// 예: 1920×1080 이미지
// → 2,073,600 픽셀 × 3 채널 × 0.7 = 4,354,560 비트
// → 544,320 바이트 ≈ 531 KB
```

---

## 참고 자료

1. **Steganography and Digital Watermarking** - Katzenbeisser & Petitcolas
2. **Information Hiding: Steganography and Watermarking** - Wayner
3. **CryptoJS Documentation** - https://cryptojs.gitbook.io/docs/
4. **Canvas API - MDN** - https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API
5. **AES Specification (FIPS 197)** - NIST

---

*이 문서는 Steganography Vault 프로젝트의 기술적 구현 세부사항을 설명합니다.*
*보안 관련 결정은 전문가와 상담하시기 바랍니다.*
