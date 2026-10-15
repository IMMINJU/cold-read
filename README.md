# cold-read

긴 작업 세션을 문서로 옮길 때, 세션을 안 본 사람이 그 문서를 읽을 수 있는지 검사하는 Claude Code 스킬.

> A Claude Code skill that checks whether a document written at the end of a long session
> can be read by someone who wasn't in it. A fresh subagent with no session context reads
> the document, paraphrases each section, and lists undefined terms and guesses.
> The main agent compares those paraphrases with what it meant to say.

## 왜 필요한가

세션에서 일한 Claude가 쓴 문서는 세션을 본 사람에게만 읽힌다. 문장이 틀려서가 아니다.
독자가 갖지 못한 맥락을 전제로 쓰기 때문이다. 쓴 쪽은 그 맥락을 지울 수 없어서 다시 읽어도 못 찾는다.

그래서 검사는 맥락이 없는 서브에이전트가 한다.

## 무엇을 잡나

```
세션에서 만든 이름      "B안으로 갔다"           → B안이 무엇인지 처음 나올 때 적는다
설명 대신 쓴 비유       "밑 빠진 독이었다"        → 재시도할 때마다 큐에 같은 작업이 다시 쌓였다
세션 속을 가리키는 지시어 "앞서 본 문제"          → 문서 앞쪽에 그 문제가 없으면 다시 적는다
생략된 중간 단계        A에서 C로 바로 감         → B를 거친 이유, 버린 안을 버린 이유
화살표 체인            "가설→기각→재설계"         → 단계마다 무슨 일이 있었는지 문장으로
출처 없는 수치          "302→278"               → p95 응답시간 302ms→278ms, 100rps 10분 부하 테스트
```

## 어떻게 검사하나

1. 문서를 쓸 때 위 여섯 가지를 처음부터 피한다.
2. 대화 맥락을 물려받지 않은 새 서브에이전트를 띄워 문서 파일 하나만 읽힌다.
   서브에이전트는 절마다 자기 말로 한 줄씩 바꿔 쓰고, 정의 없이 나온 용어와 추측으로 넘어간 곳을 돌려준다.
3. 바꿔 쓴 한 줄을 쓴 쪽의 의도와 맞춰 본다. "이해 안 됨"보다 잘못 이해한 채 넘어가는 경우가 많고,
   그건 바꿔 쓴 문장을 의도와 비교해야만 드러난다.
4. 오독 · 막힘 · 미정의 용어 · 추측 · 통과로 나눠 보고한다. 고쳐 쓴 전문은 내지 않는다.

독자는 둘 중에서 고른다.

- **동료 (기본)**: 같은 팀이고 이 세션은 안 봤다. 프로젝트 이름과 팀 용어는 안다.
- **고객사 담당자**: 문서가 회사 밖으로 나갈 때. 우리 내부 구성요소 이름은 모른다.
  밖으로 나가면 되돌리기 어려워서 독자를 두 번 띄우고, 두 번 다 나온 항목을 먼저 고친다.

## 무엇을 안 잡나

문체, 맞춤법, 문장 길이. 문장 수사는 [plain-ko](https://github.com/IMMINJU/plain-ko) 몫이다.
cold-read로 빈틈을 메운 뒤 plain-ko로 다듬는 순서를 권한다.

## 설치

```bash
git clone https://github.com/IMMINJU/cold-read
cp -r cold-read/skills/cold-read ~/.claude/skills/
```

프로젝트에만 쓰려면 `.claude/skills/` 아래에 둔다. 서브에이전트를 띄울 수 있는 환경(Agent 도구)이 필요하다.

## 쓰는 법

부르지 않아도 "문서로 정리해줘", "취합해줘", "공유용으로 써줘", "고객사 안내문 써줘"처럼
세션 내용을 문서로 옮기는 요청이면 발동한다. 이미 있는 문서를 검사하려면 이렇게 부른다.

```
이 문서 cold-read로 읽히나 봐줘
```

1,500자 미만 메모나 본인만 볼 메모에는 발동하지 않는다.

## 라이선스

MIT
