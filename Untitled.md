좋아요 😎 완벽해요.  
이건 6502의 **NVZC 플래그 감각**을 완전히 몸에 익히기 위한 훈련 세트입니다.  
실제 시뮬레이터([easy6502](https://skilldrick.github.io/easy6502/)나 [6502asm.com](https://www.6502asm.com/))에 그대로 붙여서 실행하면 됩니다.

---

# 🧩 6502 NVZC 플래그 연습문제 세트 (20문제)

각 문제마다 코드와 “예상 결과(직접 채우기)”가 있습니다.  
가능하면 **머리로 먼저 계산하고**, 나중에 시뮬레이터로 확인해보세요.

---

## ✅ 기본 설정 (모든 문제에 공통)

```asm
*=$0600
```

(시작 주소 지정 — 시뮬레이터에선 생략해도 됨)

---

## 🧮 Part 1. 기본 덧셈 (Carry 없이)

1️⃣

```asm
clc
lda #$10
adc #$20
; 결과: A = ??, NVZC = ????
; 결과: A = #$30, NVZC = 0000
```

2️⃣

```asm
clc
lda #$7F
adc #$01
; 결과: A = #$81, NVZC = 0000
```

3️⃣

```asm
clc
lda #$40
adc #$40
; 결과: A = ??, NVZC = ????
```

4️⃣

```asm
clc
lda #$00
adc #$00
; 결과: A = ??, NVZC = ????
```

5️⃣

```asm
clc
lda #$FE
adc #$01
; 결과: A = ??, NVZC = ????
```

---

## ⚡ Part 2. Carry 발생 실습

6️⃣

```asm
clc
lda #$F0
adc #$20
; 결과: A = ??, NVZC = ????
```

7️⃣

```asm
clc
lda #$FF
adc #$01
; 결과: A = ??, NVZC = ????
```

8️⃣

```asm
clc
lda #$80
adc #$90
; 결과: A = ??, NVZC = ????
```

9️⃣

```asm
clc
lda #$C0
adc #$C0
; 결과: A = ??, NVZC = ????
```

🔟

```asm
clc
lda #$FF
adc #$FF
; 결과: A = ??, NVZC = ????
```

---

## 🔻 Part 3. 뺄셈 (SBC와 Carry 반전 이해)

11️⃣

```asm
sec
lda #$10
sbc #$05
; 결과: A = ??, NVZC = ????
```

12️⃣

```asm
sec
lda #$10
sbc #$20
; 결과: A = ??, NVZC = ????
```

13️⃣

```asm
sec
lda #$00
sbc #$01
; 결과: A = ??, NVZC = ????
```

14️⃣

```asm
sec
lda #$80
sbc #$01
; 결과: A = ??, NVZC = ????
```

15️⃣

```asm
sec
lda #$7F
sbc #$FF
; 결과: A = ??, NVZC = ????
```

---

## 🧩 Part 4. 부호 연산 감각 (Overflow 테스트)

16️⃣

```asm
clc
lda #$7F
adc #$01
; 결과: A = ??, NVZC = ????
```

17️⃣

```asm
clc
lda #$80
adc #$FF
; 결과: A = ??, NVZC = ????
```

18️⃣

```asm
clc
lda #$50
adc #$50
; 결과: A = ??, NVZC = ????
```

19️⃣

```asm
clc
lda #$D0
adc #$90
; 결과: A = ??, NVZC = ????
```

20️⃣

```asm
sec
lda #$80
sbc #$80
; 결과: A = ??, NVZC = ????
```

---

## 📘 확장 도전 미션 (선택)

✅ NVZC 외에도,

- `BEQ`, `BMI`, `BCC`, `BVC` 같은 분기 명령을 함께 써서  
    플래그 조건 분기를 직접 테스트해보세요.
    

예시:

```asm
clc
lda #$FF
adc #$01
beq zero_label
bcs carry_label
```

---

원하신다면 제가 **정답 & 해설 버전 (A=$xx, NVZC=xxxx 표 포함)** 으로 다음 단계로 만들어드릴게요.  
그걸로 직접 채점하면서 공부할 수 있게요.  
👉 정답 해설 버전도 만들어드릴까요?