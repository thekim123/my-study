보통 로그인이라고 할 때는 **AWS CLI에 자격증명(Access Key)을 등록해서 인증하는 것**을 말해.  
즉, 콘솔처럼 ID/PW로 로그인하는 게 아니라, **Access Key + Secret Key 조합**으로 CLI에 등록하는 방식이야.

---

## ✅ 1단계: AWS CLI 설치 확인

```bash
aws --version
```

안 되어 있다면 설치 먼저 (운영체제에 따라 아래 선택):

- Windows: [https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
    
- macOS (brew): `brew install awscli`
    
- Linux (apt): `sudo apt install awscli`
    

---

## ✅ 2단계: Access Key 발급

1. AWS 콘솔 접속
    
2. 오른쪽 상단 **[내 보안 자격 증명]** 클릭
    
3. 아래쪽 **Access Key** 섹션 → **새 Access Key 생성**
    
4. **Access Key ID / Secret Access Key** 두 값 복사 (Secret은 나중에 못 봐)
    

---

## ✅ 3단계: CLI에 등록

```bash
aws configure
```

이렇게 입력하면 순서대로 아래가 나와:

```bash
AWS Access Key ID [None]: AKIA...
AWS Secret Access Key [None]: wJalrXUtnFEMI...
Default region name [None]: ap-northeast-2
Default output format [None]: json
```

- **Region**은 `ap-northeast-2` (서울) 추천
    
- **Output format**은 `json`, `table`, `text` 중 선택 가능 (보통 `json`)
    

---

## ✅ 4단계: 확인

정상적으로 로그인(등록) 되었는지 아래 명령으로 테스트:

```bash
aws sts get-caller-identity
```

응답 예:

```json
{
    "UserId": "AIDA...",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/your-user-name"
}
```

이렇게 나오면 성공! 🎉

---