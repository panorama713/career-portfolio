# 북메이트 — 도서관 무인 대출·추천 키오스크 플랫폼

> **2024.01 ~ 재직 중** | **공동 개발 (백엔드 2인)** | 8개 저장소 / 본인 커밋 약 430건 (전체의 약 40%)
> **도입 30개 도서관 · 키오스크 65대 · 일평균 약 2,700건 처리**

## 담당 범위 (본인)

전체 8개 저장소 중 **키오스크 클라이언트(약 190/300 커밋)** 를 주도했고, 서버에서는 **문학상 수상작 도메인, 도서관 통계, 키오스크 관리, SNS 연동** 영역을 담당했습니다. 인프라·CI/CD·게이트웨이 및 서지 데이터 구축 배치는 동료(팀 리드)가 주도했으며, 본인은 기능 단위로 참여했습니다.

## 주요 작업

- **문학상 수상작 도메인 개발** — 최근 3개년 문학상 수상작을 수집·제공하는 기능을 API·서비스·문서 계층까지 담당하고, 해당 기능의 **테스트 코드를 함께 작성**
- **도서관 이용 통계 수집** — 인터셉터 기반으로 도서관별 일일 이용 통계를 수집하는 구조를 구현. 비즈니스 로직에 집계 코드를 섞지 않고 횡단 관심사로 분리
- **키오스크 관리 기능** — 키오스크 등록·조회 API 및 도서관별 장비 관리. 도서관 코드·키오스크 키 기반으로 장비별 설정을 분리하고, 설정 누락 시 경고하도록 처리해 **65대 규모 배포에서 설정 오류를 예방**
- **SNS 연동 및 외부 연계** — SNS 연동 기능 개발, 알라딘 서지 API·정보나루(공공 도서관 데이터) 등 외부 소스 연동 및 **외부 연계 서비스 테스트 코드 작성**
- **다국어 메시지 체계 관리** — 메시지 프로퍼티 기반 응답 메시지 일원화
- **운영 이슈 상시 대응** — 외부 API 스펙 변경, SSL 인증서 문제, 배포 환경 이슈 등

## 기술적 선택과 근거

**민감정보 암호화 계층 분리** — 알림톡 발송에는 이용자 휴대폰 번호가 필요한데 평문 저장은 개인정보 리스크가 큽니다. **RSA 비대칭키를 적용해 저장은 공개키, 발송 시점 복호화는 비밀키로 분리**해 DB 접근만으로는 번호를 복원할 수 없도록 했습니다. DB 비밀번호·외부 API 키 등은 **Jasypt로 암호화**해 저장소에 평문이 남지 않게 했습니다.

> 아래 코드는 실제 구현을 단순화해 재구성한 예시입니다.

```java
// 저장 시: 공개키로 암호화 (앱 서버 어디서든 가능)
public String encryptPhoneNumber(String rawPhoneNumber) {
    Cipher cipher = Cipher.getInstance("RSA/ECB/PKCS1Padding");
    cipher.init(Cipher.ENCRYPT_MODE, publicKey);
    byte[] encrypted = cipher.doFinal(rawPhoneNumber.getBytes(StandardCharsets.UTF_8));
    return Base64.getEncoder().encodeToString(encrypted);
}

// 발송 시점: 비밀키로 복호화 (비밀키는 발송 처리 서버에만 존재)
public String decryptPhoneNumber(String encryptedPhoneNumber) {
    Cipher cipher = Cipher.getInstance("RSA/ECB/PKCS1Padding");
    cipher.init(Cipher.DECRYPT_MODE, privateKey);
    byte[] decrypted = cipher.doFinal(Base64.getDecoder().decode(encryptedPhoneNumber));
    return new String(decrypted, StandardCharsets.UTF_8);
}
```

**캐시 도입 시 개발 환경 편의성 보존** — 도서 검색에 Redis 캐시를 적용하면서, 로컬에서는 Redis 없이도 기동 가능하도록 `spring.cache.type=none` 분기를 제공해 **온보딩 비용을 낮췄습니다.**

```yaml
# application-local.yml — Redis 없이 로컬 기동
spring:
  cache:
    type: none
---
# application-prod.yml
spring:
  cache:
    type: redis
```

## 현장 설치 자동화

이 프로젝트에서 처음 설계한 설치 자동화 체계를 이후 3개 제품에 추가로 적용했습니다. 자세한 내용은 [현장 설치·배포 자동화](05-install-automation.md) 문서를 참고해주세요.
