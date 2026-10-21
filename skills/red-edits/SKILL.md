---
name: red-edits
description: 사용자의 문서(docx, hwp/hwpx, pptx, xlsx, html, md 등)를 수정할 때, Claude가 새로 쓰거나 바꾼 부분의 글자를 빨간색(#FF0000)으로 표시한다. 문서 수정·교정·윤문·내용 추가·서식 채우기를 할 때마다 적용하고, 사용자가 "/red-edits"를 입력하거나 "수정한 곳 빨간색으로", "바뀐 부분 표시해줘"라고 할 때도 사용한다.
---

# 수정한 부분은 빨간 글자로

사용자의 문서를 고칠 때는 **Claude가 추가하거나 바꾼 텍스트만** 빨간색(`#FF0000`)으로 칠한다. 손대지 않은 원문은 색과 서식을 그대로 둔다. 사용자가 무엇이 바뀌었는지 한눈에 보고 검토할 수 있게 하려는 것이다.

## 규칙

1. **범위**: 바뀐 단어나 구절만 칠한다. 문장 안에서 한 단어만 고쳤다면 그 단어만 칠하고, 문장 전체를 다시 썼다면 문장 전체를 칠한다. 새로 넣은 문단·표 셀·항목은 전부 칠한다.
2. **삭제한 부분**: 기본적으로 그냥 지운다. 사용자가 "삭제한 것도 보여줘"라고 하면 빨간색 취소선으로 남긴다.
3. **기존 서식 유지**: 글꼴·크기·굵기·기울임은 원문 그대로 두고 **색만** 바꾼다. 원래 run의 서식을 복사한 뒤 색만 덮어쓴다.
4. **누적 수정**: 같은 대화에서 다시 고칠 때 이전 빨간 표시는 그대로 두고 새로 바꾼 부분도 빨간색으로 칠한다. 사용자가 "표시 지워줘"나 "확정"이라고 하면 빨간색을 원래 색(보통 검정/자동)으로 되돌린다.
5. **보고**: 작업이 끝나면 무엇을 바꿨는지 짧게 목록으로 알려준다. 채팅창에서는 ```diff 블록을 쓴다(`-` 줄은 원문, `+` 줄은 수정본). 터미널에서 빨강/초록으로 보인다.

## 형식별 적용 방법

| 형식 | 방법 |
|---|---|
| **docx** | python-docx: `run.font.color.rgb = RGBColor(0xFF, 0x00, 0x00)`. XML을 직접 다룰 때는 `<w:rPr><w:color w:val="FF0000"/></w:rPr>`. 바뀐 부분만 칠하려면 run을 나눈다. |
| **hwpx** | `charPr`에 `textColor="#FF0000"`를 둔 글자 모양을 새로 만들고(원래 charPr를 복사해서 색만 변경), 바뀐 텍스트 run의 `charPrIDRef`를 그 ID로 지정한다. |
| **hwp** | `k-skill-rhwp` CLI에는 색 옵션이 없으므로 Node에서 `@rhwp/core`를 직접 쓴다(`require('k-skill-rhwp/src/wasm-init').getRhwpCore()` → `new core.HwpDocument(bytes)`). 본문은 `applyCharFormat(sec, para, start, end, '{"textColor":"#FF0000"}')`, 표 셀은 `applyCharFormatInCell(sec, parentPara, control, cell, cellPara, start, end, props)`. 셀 칸을 다시 쓸 때는 `deleteTextInCell` → `insertTextInCell` → 전체를 원래 색으로 → 채운 값 범위만 빨강 순서로 하고, `exportHwp()`로 저장한다. 저장한 뒤 `getCellCharPropertiesAt(...).textColor`로 색을 확인한다. |
| **pptx** | python-pptx: `run.font.color.rgb = RGBColor(0xFF, 0x00, 0x00)`. XML은 `<a:solidFill><a:srgbClr val="FF0000"/></a:solidFill>`. |
| **xlsx** | openpyxl: 셀의 기존 Font를 `copy(cell.font, color="FFFF0000")`로 복사해서 바꾼다. 셀 일부만 바꿨다면 rich text(`CellRichText`, `TextBlock`)를 쓴다. |
| **html** | `<span style="color:#FF0000">수정한 텍스트</span>`. 삭제 표시는 `<del style="color:#FF0000">…</del>`. |
| **md** | `<span style="color:#FF0000">수정한 텍스트</span>` (HTML을 렌더링하는 뷰어에서 보인다). |
| **Google Docs / Notion 등 연결된 도구** | 해당 도구가 지원하는 글자 색 서식(Notion은 `red` 색)을 바꾼 부분에 적용한다. |

## 적용하지 않는 경우

- **소스 코드나 설정 파일**(.py, .js, .json, .yaml 등): 글자 색이라는 것이 없다. 파일에는 색 표시를 넣지 말고, 채팅에서 ```diff 블록으로 바뀐 부분을 보여준다.
- 사용자가 "색 표시 없이" 또는 "깔끔하게 최종본으로"라고 요청한 경우.
- 처음부터 새로 만드는 문서: 전부 새 내용이라 칠할 기준이 없다. 단, 사용자가 준 템플릿이나 양식을 채우는 경우에는 채워 넣은 내용을 빨간색으로 칠한다.

## 마무리 확인

저장한 다음 파일을 다시 열어서 다음을 확인한다.
- 바뀐 부분이 실제로 빨간색인지
- 바뀌지 않은 원문의 색과 서식이 그대로인지
