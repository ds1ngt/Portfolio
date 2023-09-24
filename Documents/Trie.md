# 검색 (Trie)
## 개요
Trie기반 문자열 검색 도구
## 기능
- 일반 검색
- 초성 검색 (한글)
- 부분 검색 (특정 문자열의 일부분에 대한 검색)
- 검색 결과로 객체 저장
## API
### Trie
API | 설명
--- | ---
static Trie\<T> **CreateNew**(TrieSettings settings, params Pair[] pairs) | 새로운 Trie 생성
void **Insert**(Pair pair) | 검색 대상 추가
List\<T> **FindAll**(string key) | 검색
void **Clear**() | Trie 제거
### Trie.TrieSettings
Trie 설정 구조체

옵션 | 설명
UsePartialSearch | 부분 검색 활성화
UseConsonantSearch | 초성 검색 활성화

## 사용 예시
``` csharp
// 초기화
var trie = Trie<string>.CreateNew(Trie<string>.TrieSettings.Default);

// 검색 대상 추가
var items = new string[] {"홍길동", "김철수"};
foreach (var item in items)
 trie.Insert(new Trie<string>.Pair { Key = item, value = item} );

// 검색
var result = trie.FindAll("ㅎㄱㄷ");    // result = List<string> {"홍길동"};
```

## 제약 사항
- 초성 + 문자열 검색 불가  
ex) ㅎ길동 (x)