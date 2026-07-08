---
name: coding-ceo-sns-content-pipeline
description: "「코딩하는 CEO」 SNS 콘텐츠 제작 파이프라인. 쇼츠 영상/소재를 받아 자막 작업, 롱폼 확장, 자연스러운 음성 교체, 콘텐츠 전략 리뷰까지 수행한다."
version: 1.0.0
author: Pola for Ninestone
license: MIT
metadata:
  hermes:
    tags: [coding-ceo, sns-content, shorts, reels, youtube, captions, longform, higgsfield, voiceover, promo]
---

# 「코딩하는 CEO」 SNS 콘텐츠 제작 파이프라인

나인스톤이 「코딩하는 CEO」 관련 영상·이미지·카피를 주고 다음 요청을 할 때 사용한다.

- “자막 작업해줘”
- “유튜브 쇼츠에 올릴 거야”
- “이 영상을 롱폼으로 만들어줘”
- “음성을 힉스필드에서 자연스러운 음성으로 바꿔줘”
- “SNS 콘텐츠 만든 과정 리뷰해줘”
- “코딩하는 CEO 콘텐츠 만들어줘”

목표는 단순 설명이 아니라 **실제 업로드 가능한 SNS 콘텐츠 산출물**을 만드는 것이다.

## 핵심 포지셔닝

「코딩하는 CEO」 콘텐츠의 중심 메시지는 다음이다.

> 대표님, 개발자부터 찾지 말고 AI로 먼저 만들고 먼저 검증하세요.

이 콘텐츠는 “대표가 개발자가 되자”가 아니다.

- 대표가 기술을 이해한다.
- 외주/개발팀에 끌려가지 않는다.
- 고객 반응을 먼저 검증한다.
- 개발자 채용 전, 외주 발주 전, AI로 첫 번째 작동물을 만든다.

## 기본 전환 목표

기본 CTA는 다음 중 하나를 사용한다.

- 체험판 60페이지 무료
- 프로필 링크에서 60페이지 체험판 받기
- 개발자 뽑기 전, 이 60페이지부터 보세요
- 대표님용 AI 실행 가이드, 무료로 받아보세요

나인스톤이 별도 CTA를 주면 그걸 우선한다.

## 작업 원칙

1. **비즈니스 메시지 우선**
   - 예쁜 영상보다 리드 전환이 중요하다.
   - 핵심은 “조회수 → 신뢰 → 체험판 다운로드 → 상담/교육 문의” 흐름이다.
2. **쇼츠 자막은 하단 UI를 피한다**
   - 유튜브 쇼츠는 하단 제목/계정/CTA 영역에 자막이 가리기 쉽다.
   - 기본적으로 중상단 또는 상단 안전영역에 배치한다.
   - 영상마다 프레임을 추출해 자막 위치를 판단한다.
3. **한국어 텍스트는 후반 합성으로 통제한다**
   - 생성 모델에 한국어 텍스트를 직접 맡기지 않는다.
   - `ffmpeg` + `.ass` 자막으로 정확하게 합성한다.
4. **검증 없는 납품 금지**
   - `ffprobe`로 해상도/길이/오디오 스트림 확인.
   - 필요 시 중간 프레임을 뽑아 `vision_analyze`로 가독성 확인.
5. **한 소재를 여러 포맷으로 재활용한다**
   - 쇼츠 광고
   - 릴스 광고
   - 가로형 미니 롱폼
   - 랜딩페이지 삽입 영상
   - 강의 홍보 영상

## 1. 쇼츠 자막 작업 절차

### 1) 영상 정보 확인

```bash
ffprobe -v error \
  -show_entries format=duration:stream=width,height,r_frame_rate \
  -of json input.mp4
```

확인할 것:

- 9:16 세로 영상인지
- 길이
- 오디오 스트림 존재 여부

### 2) 음성 인식

한국어 음성은 `faster_whisper`를 우선 사용한다.

```python
from faster_whisper import WhisperModel

path = "input.mp4"
model = WhisperModel("small", device="cpu", compute_type="int8")
segments, info = model.transcribe(path, language="ko", vad_filter=True, beam_size=5)
for s in segments:
    print(f"{s.start:.2f}\t{s.end:.2f}\t{s.text.strip()}")
```

주의:

- 자동 인식 오타는 문맥상 보정한다.
  - 예: `코닝` → `코딩`
- 문장은 SNS 자막용으로 더 짧게 다듬는다.

### 3) 프레임 추출 후 자막 위치 판단

```bash
ffmpeg -y -ss 3.0 -i input.mp4 -frames:v 1 frame_check.jpg
```

`vision_analyze`로 확인한다.

검수 질문 예시:

> 이 세로 영상 프레임에서 유튜브 쇼츠 하단 기본 UI에 가리지 않으면서 주요 화면 요소를 덜 가리는 자막 위치를 판단해줘.

기본 위치:

- 책/제품이 중앙·하단에 있으면 **상단 안전영역**
- 얼굴이 상단에 있으면 **중상단 또는 중앙 하단보다 위**
- 하단 25~30%는 가능하면 피한다.

### 4) ASS 자막 생성

권장 스타일:

```ass
[Script Info]
ScriptType: v4.00+
PlayResX: 720
PlayResY: 1280
ScaledBorderAndShadow: yes
YCbCr Matrix: TV.709

[V4+ Styles]
Format: Name, Fontname, Fontsize, PrimaryColour, SecondaryColour, OutlineColour, BackColour, Bold, Italic, Underline, StrikeOut, ScaleX, ScaleY, Spacing, Angle, BorderStyle, Outline, Shadow, Alignment, MarginL, MarginR, MarginV, Encoding
Style: ShortsTop,Pretendard ExtraBold,50,&H00FFFFFF,&H000000FF,&H00000000,&H80000000,-1,0,0,0,100,100,-1,0,1,5,2,8,55,55,95,1
```

강조색:

- AI 강조: `&H00FFF04A&` 노랑
- CTA/60페이지 강조: `&H0000E8FF&` 시안
- 기본: 흰색 + 검은 외곽선

자막 문구 예시:

```ass
Dialogue: 0,0:00:00.00,0:00:03.52,ShortsTop,,0,0,0,,{\c&H0000E8FF&}코딩하는 CEO{\c&H00FFFFFF&}\N코딩 배우지 마세요\N{\c&H00FFF04A&}AI에게 시키세요
Dialogue: 0,0:00:03.52,0:00:05.98,ShortsTop,,0,0,0,,개발자 뽑기 전\N대표가 먼저 검증하세요
Dialogue: 0,0:00:05.98,0:00:08.06,ShortsTop,,0,0,0,,체험판 {\c&H0000E8FF&}60페이지 무료{\c&H00FFFFFF&}\N지금 받아보세요
```

### 5) 렌더링

```bash
ffmpeg -y -i input.mp4 \
  -vf "subtitles=captions.ass:fontsdir=/Users/macbook15-platform/Library/Fonts" \
  -c:v libx264 -preset medium -crf 18 \
  -c:a aac -b:a 192k \
  -movflags +faststart \
  output_safe_shorts.mp4
```

### 6) 검증

```bash
ffprobe -v error \
  -show_entries format=duration,size \
  -show_entries stream=codec_type,width,height \
  -of json output_safe_shorts.mp4
```

중간 프레임 추출:

```bash
ffmpeg -y -ss 3.0 -i output_safe_shorts.mp4 -frames:v 1 subtitle_check.jpg
```

`vision_analyze`로 자막 위치/가독성 확인.

## 2. 쇼츠 → 가로형 미니 롱폼 확장

나인스톤이 “롱폼으로 만들어줘”라고 하면 기본값은 다음이다.

- 원본: 8초 내외 세로 쇼츠
- 결과: 1~2분짜리 16:9 가로형 미니 롱폼
- 용도: 유튜브 일반 영상, 랜딩페이지 삽입, 광고 설명 영상

진짜 5~8분 롱폼이 필요하면 별도 대본 확장을 제안한다.

### 롱폼 구성

1. 훅
   - “대표님, 아직도 개발자부터 찾고 계신가요?”
2. 문제 제기
   - 아이디어가 생겼을 때 바로 개발자를 찾는 위험
3. 해결 관점
   - AI로 먼저 만들고 먼저 검증
4. 「코딩하는 CEO」 포지셔닝
   - 개발자가 되자는 게 아니라 대표가 기술을 이해하는 것
5. 구체 예시
   - 랜딩페이지, 자동화, 챗봇, 내부 업무 도구
6. CTA
   - 체험판 60페이지 무료

### 내레이션 대본 예시

```text
대표님, 아직도 개발자부터 찾고 계신가요?
많은 대표님들이 아이디어가 생기면 가장 먼저 개발자를 찾습니다.
하지만 문제는 개발자를 뽑기 전에, 이 아이디어가 정말 고객에게 필요한지부터 검증해야 한다는 겁니다.

이제는 코딩을 처음부터 깊게 배우지 않아도 됩니다.
AI에게 시키면 됩니다.
중요한 건 대표가 직접 문제를 정의하고, AI와 함께 빠르게 만들어보고, 시장에서 먼저 반응을 확인하는 것입니다.

코딩하는 CEO는 개발자가 되자는 이야기가 아닙니다.
대표가 기술을 이해하고, 외주와 개발팀에 끌려가지 않고, 먼저 만들고 먼저 검증하는 실행력을 갖추자는 이야기입니다.

작은 랜딩페이지, 간단한 자동화, 고객 응대 챗봇, 내부 업무 도구.
이런 것들을 대표가 직접 프로토타입으로 만들어볼 수 있다면 의사결정 속도가 달라집니다.

체험판 60페이지를 무료로 받아보세요.
AI 시대의 대표는 개발자를 기다리지 않습니다.
먼저 만들고, 먼저 검증합니다.
```

### 가로형 슬라이드 영상 레이아웃

권장 1920×1080 구조:

- 왼쪽: 큰 메시지 텍스트
- 오른쪽: 원본 세로 영상을 휴대폰 mockup처럼 배치
- 배경: 원본 프레임을 블러 처리
- 하단: “먼저 만들고 · 먼저 검증하세요” + CTA

권장 슬라이드:

1. 대표님, 아직도 개발자부터 찾으세요?
2. 개발자 뽑기 전 대표가 먼저 검증하세요
3. 코딩을 배우는 게 아니라 AI에게 시키는 시대
4. 코딩하는 CEO는 개발자가 되자는 말이 아닙니다
5. 작게 만들고 빠르게 확인하세요
6. 완벽한 서비스보다 첫 번째 작동물이 먼저입니다
7. AI 시대의 대표는 기다리지 않습니다
8. 체험판 60페이지 무료

검증:

```bash
ffprobe -v error \
  -show_entries format=duration,size \
  -show_entries stream=codec_type,width,height \
  -of json longform.mp4
```

## 3. 자연스러운 음성 교체 — Higgsfield

나인스톤이 “힉스필드에서 자연스러운 음성으로 바꿔줘”라고 하면 Higgsfield `voice_change`를 사용한다.

> 음성 출처/사용 보이스를 나중에 물어보면, 새로 업로드된 파일명이 아니라 **이전 납품본과의 해시 매칭 및 오디오 보존 여부**로 확인한다. 자세한 검증 패턴은 `references/voice-provenance-and-output-matching.md`를 참고한다.

### 1) Higgsfield 사용 가능 확인

```bash
command -v higgsfield
higgsfield --help
```

### 2) 보이스 목록 확인

```bash
higgsfield voices list --size 50 --json
```

「코딩하는 CEO」 기본 추천:

- 남성 비즈니스 톤: `Mark`
- 차분한 전문가 톤: `Roman`, `Sterling`, `Julian` 등 후보 검토

예시 Mark voice id:

```text
27c04473-84a9-4b60-a41f-c8e8458bd4f1
```

### 3) 원본 롱폼 영상 업로드

```bash
higgsfield upload create longform.mp4 --json
```

반환 예시:

```json
{
  "id": "...",
  "type": "video",
  "url": "..."
}
```

### 4) voice_change 실행

주의: 업로드된 입력 비디오는 `type`을 `video_input`으로 넘긴다.

```bash
higgsfield generate create voice_change \
  --input_video '{"id":"UPLOAD_ID","type":"video_input"}' \
  --voice_id 27c04473-84a9-4b60-a41f-c8e8458bd4f1 \
  --voice_type preset \
  --wait --wait-timeout 10m --wait-interval 10s \
  --json
```

완료되면 `result_url`을 다운로드한다.

```bash
curl -L 'RESULT_URL' -o output_higgsfield_voice.mp4
```

### 5) 음성 검증

```bash
ffprobe -v error \
  -show_entries format=duration,size \
  -show_entries stream=index,codec_type,codec_name,width,height \
  -of json output_higgsfield_voice.mp4
```

음량 확인:

```bash
ffmpeg -i output_higgsfield_voice.mp4 \
  -af volumedetect -vn -sn -dn -f null - 2>&1 | tail -20
```

원본 음성과 달라졌는지 확인하려면 오디오 추출 후 해시 비교:

```bash
ffmpeg -y -i original.mp4 -vn -ac 1 -ar 16000 /tmp/orig.wav
ffmpeg -y -i output_higgsfield_voice.mp4 -vn -ac 1 -ar 16000 /tmp/higgs.wav
shasum -a 256 /tmp/orig.wav /tmp/higgs.wav
```

## 4. 콘텐츠 리뷰 프레임

나인스톤이 “과정 리뷰해줘”라고 하면 다음 구조로 간결히 리뷰한다.

### 1) 전체 흐름

| 단계 | 작업 | 결과 |
|---|---|---|
| 1 | 쇼츠 영상 확보 | 8초 내외 세로 영상 |
| 2 | 음성 인식/대본 추출 | 한국어 문장 추출 |
| 3 | 쇼츠 자막 삽입 | 하단 UI 회피 |
| 4 | 자막 스타일링 | 핵심어 컬러 강조 |
| 5 | 롱폼 변환 | 16:9 미니 롱폼 |
| 6 | 내레이션 생성 | 설명형 음성 |
| 7 | 힉스필드 음성 교체 | 자연스러운 보이스 |

### 2) 잘한 점

- 핵심 메시지가 명확했다.
- 쇼츠 하단 UI를 피한 자막 배치가 좋았다.
- 숏폼 → 롱폼 재활용 가능성을 확인했다.
- 음성 품질을 개선해 광고 영상 느낌을 높였다.

### 3) 아쉬운 점

- 1분대 영상은 진짜 롱폼보다는 미니 롱폼/광고 설명 영상에 가깝다.
- 사례가 부족하면 메시지가 추상적으로 보일 수 있다.
- CTA는 “프로필 링크에서 받기”처럼 더 행동 중심으로 구체화하면 좋다.

### 4) 다음 콘텐츠 방향

A/B/C 3계열로 나눈다.

| 계열 | 목적 | 예시 훅 |
|---|---|---|
| A 문제 자극형 | 공감/정지 | 대표님, 개발자부터 찾으면 늦습니다 |
| B 해결 제안형 | 방법론 각인 | AI로 먼저 만들고 시장에서 먼저 검증하세요 |
| C 사례 증명형 | 신뢰/전환 | 랜딩페이지, 챗봇, 자동화까지 대표가 먼저 만들 수 있습니다 |

## 5. 납품 메시지 템플릿

### 쇼츠 자막 납품

```text
나인스톤, 자막 작업 완료했어요.

- 쇼츠용 세로비율 유지: 720×1280
- 자막 위치: 상단/중상단 안전영역
- 유튜브 쇼츠 하단 UI 회피
- 출력 검증 완료: [길이] / mp4 정상 생성

MEDIA:/path/to/output.mp4
```

### 롱폼 납품

```text
나인스톤, 롱폼 버전 초안 만들어뒀어요.

- 원본: 8초 쇼츠형 세로 영상
- 변환본: [길이] / 유튜브 롱폼 16:9
- 구성: 훅 → 문제 제기 → AI 검증 메시지 → 「코딩하는 CEO」 포지셔닝 → CTA
- 검증: 1920×1080 / 오디오 포함 / mp4 정상 생성

MEDIA:/path/to/longform.mp4
```

### 힉스필드 음성 납품

```text
나인스톤, 힉스필드 보이스로 자연스럽게 바꾼 버전 만들었어요.

- 처리 방식: Higgsfield voice_change
- 보이스: Mark 계열 남성 보이스
- 길이: [길이]
- 형식: 1920×1080 / mp4
- 검증: 오디오 스트림 정상, 음량 정상

MEDIA:/path/to/higgsfield_voice.mp4
```

## 6. 다음 액션 추천

복수 콘텐츠를 만들었으면 다음으로 제안한다.

> 「코딩하는 CEO」 7일치 SNS 콘텐츠 캘린더를 만들고, 각 날짜마다 쇼츠 문구/영상 콘셉트/CTA를 정리하자.

기본 7일 콘텐츠 축:

1. 대표님, 개발자부터 찾으면 늦습니다.
2. 코딩 배우지 말고 AI에게 시키세요.
3. 외주 맡기기 전 MVP부터 직접 만들어보세요.
4. 랜딩페이지 하나로 먼저 검증하세요.
5. 챗봇/자동화로 내부 업무부터 바꿔보세요.
6. AI 시대 대표의 역할은 문제 정의입니다.
7. 체험판 60페이지 무료 CTA.
