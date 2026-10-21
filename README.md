# skillforme

개인용 에이전트 스킬 모음입니다. [skills.sh](https://skills.sh) 형식으로 배포됩니다.

## 설치

```bash
# 전체 스킬 설치
npx skills add shain1912/skillforme

# 특정 스킬만 설치
npx skills add shain1912/skillforme --skill red-edits
```

## 스킬 목록

| 스킬 | 설명 |
|---|---|
| [red-edits](skills/red-edits/SKILL.md) | 문서(docx, hwp/hwpx, pptx, xlsx, html, md 등)를 수정하거나 양식을 채울 때, 에이전트가 추가·변경한 글자만 빨간색(#FF0000)으로 표시합니다. |

## 사용 예

- "이 신청서 채워줘" → 채운 내용만 빨간 글자로 저장
- "이 보고서 교정해줘" → 고친 단어·문장만 빨간 글자로 저장
- "표시 지워줘" / "확정" → 빨간색을 원래 색으로 되돌림
