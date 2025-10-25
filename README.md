# #12 CSV 변환기

> ⚠️ **개발 시작 전 필독!**
> 전역 개발 가이드: [`../_template/README.md`](../_template/README.md)
> Phase 1 버그, 체크리스트, 크로스 프로모션 도구 버튼 구현 확인 필수!

**URL:** csv.baal.co.kr

## 서비스 내용

CSV↔JSON↔Excel 변환. 구분자 설정, 미리보기

## 기능 요구사항

- [ ] CSV 파일 업로드 (드래그 앤 드롭)
- [ ] CSV → JSON 변환
- [ ] JSON → CSV 변환
- [ ] CSV → Excel (XLSX) 변환
- [ ] Excel → CSV 변환
- [ ] 구분자 선택 (쉼표, 탭, 세미콜론, 파이프 등)
- [ ] 첫 행을 헤더로 사용 옵션
- [ ] 데이터 미리보기 (테이블)
- [ ] 인코딩 선택 (UTF-8, EUC-KR)
- [ ] 다운로드 (CSV, JSON, XLSX)
- [ ] 텍스트 직접 입력

## 경쟁사 분석 (2025년 기준)

### 인기 사이트 TOP 5

1. **ConvertCSV** - 가장 인기 있는 CSV 변환 사이트
   - 강점: 20+ 포맷 지원, CSV → JSON/XML/SQL 등
   - 약점: UI 복잡, 광고 많음

2. **CSV to JSON Converter** - 간단한 UI
   - 강점: 빠르고 간단
   - 약점: CSV/JSON만 지원

3. **Mr. Data Converter** - 개발자 전문
   - 강점: 다양한 포맷 (HTML, MySQL, PHP 배열 등)
   - 약점: 디자인 구식

4. **Aconvert** - 통합 변환 도구
   - 강점: 문서/이미지/비디오 등 모든 변환
   - 약점: CSV 기능 제한적

5. **Online CSV Tools** - CSV 전문
   - 강점: CSV 특화 도구 모음
   - 약점: 복잡한 UI

### 우리의 차별화 전략

- ✅ **3가지 포맷** - CSV, JSON, Excel 양방향 변환
- ✅ **실시간 미리보기** - 테이블로 결과 확인
- ✅ **구분자 자동 감지** - 쉼표, 탭, 세미콜론 자동 인식
- ✅ **한글 인코딩 지원** - UTF-8, EUC-KR
- ✅ **다크모드** 지원
- ✅ **한/영 전환**
- ✅ **완전 무료** - 광고 없음

## 주요 라이브러리

### 옵션 1: PapaParse (CSV 파싱 - 추천!)

가장 강력한 CSV 파서

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.4.1/papaparse.min.js"></script>
```

```javascript
// CSV → JSON
function csvToJson(csvText, delimiter = ',') {
  const result = Papa.parse(csvText, {
    header: true,           // 첫 행을 헤더로
    delimiter: delimiter,   // 구분자
    skipEmptyLines: true,   // 빈 줄 건너뛰기
    dynamicTyping: true,    // 자동 타입 변환 (숫자, 불린 등)
    encoding: 'UTF-8'
  });

  if (result.errors.length > 0) {
    console.error('CSV 파싱 에러:', result.errors);
  }

  return result.data; // JSON 배열
}

// 사용 예시
const csv = `name,age,email
John Doe,30,john@example.com
Jane Smith,25,jane@example.com`;

const json = csvToJson(csv);
console.log(json);
// [
//   { name: 'John Doe', age: 30, email: 'john@example.com' },
//   { name: 'Jane Smith', age: 25, email: 'jane@example.com' }
// ]
```

### JSON → CSV

```javascript
function jsonToCsv(jsonArray, delimiter = ',') {
  const csv = Papa.unparse(jsonArray, {
    delimiter: delimiter,
    header: true,
    skipEmptyLines: true,
    quotes: true  // 따옴표로 감싸기
  });

  return csv;
}

// 사용 예시
const json = [
  { name: 'John Doe', age: 30, email: 'john@example.com' },
  { name: 'Jane Smith', age: 25, email: 'jane@example.com' }
];

const csv = jsonToCsv(json);
console.log(csv);
// "name","age","email"
// "John Doe",30,"john@example.com"
// "Jane Smith",25,"jane@example.com"
```

### 파일에서 CSV 읽기

```javascript
function parseCSVFile(file) {
  return new Promise((resolve, reject) => {
    Papa.parse(file, {
      header: true,
      dynamicTyping: true,
      skipEmptyLines: true,
      complete: (result) => {
        resolve(result.data);
      },
      error: (error) => {
        reject(error);
      }
    });
  });
}

// 사용
const file = document.querySelector('input[type="file"]').files[0];
const data = await parseCSVFile(file);
console.log(data);
```

### 구분자 자동 감지

```javascript
function detectDelimiter(csvText) {
  const delimiters = [',', '\t', ';', '|'];
  let maxColumns = 0;
  let bestDelimiter = ',';

  delimiters.forEach(delimiter => {
    const result = Papa.parse(csvText, {
      delimiter: delimiter,
      preview: 1  // 첫 줄만 미리보기
    });

    const columnCount = result.data[0]?.length || 0;

    if (columnCount > maxColumns) {
      maxColumns = columnCount;
      bestDelimiter = delimiter;
    }
  });

  return bestDelimiter;
}

// 사용
const delimiter = detectDelimiter(csvText);
console.log(`감지된 구분자: ${delimiter}`);
```

### 옵션 2: SheetJS (Excel 변환)

Excel XLSX 파일 읽기/쓰기

```html
<script src="https://cdn.sheetjs.com/xlsx-0.20.1/package/dist/xlsx.full.min.js"></script>
```

```javascript
// CSV → Excel (XLSX)
function csvToExcel(csvText, filename = 'output.xlsx') {
  // CSV 파싱
  const data = Papa.parse(csvText, { header: true }).data;

  // JSON → Worksheet
  const worksheet = XLSX.utils.json_to_sheet(data);

  // Workbook 생성
  const workbook = XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(workbook, worksheet, 'Sheet1');

  // 다운로드
  XLSX.writeFile(workbook, filename);
}

// Excel → CSV
async function excelToCsv(file) {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();

    reader.onload = (e) => {
      try {
        // ArrayBuffer → Workbook
        const data = new Uint8Array(e.target.result);
        const workbook = XLSX.read(data, { type: 'array' });

        // 첫 번째 시트 가져오기
        const firstSheet = workbook.Sheets[workbook.SheetNames[0]];

        // Worksheet → CSV
        const csv = XLSX.utils.sheet_to_csv(firstSheet);

        resolve(csv);
      } catch (error) {
        reject(error);
      }
    };

    reader.onerror = reject;
    reader.readAsArrayBuffer(file);
  });
}

// 사용
const file = document.querySelector('input[type="file"]').files[0];
const csv = await excelToCsv(file);
console.log(csv);
```

### JSON → Excel

```javascript
function jsonToExcel(jsonArray, filename = 'output.xlsx') {
  // JSON → Worksheet
  const worksheet = XLSX.utils.json_to_sheet(jsonArray);

  // 열 너비 자동 조정
  const columnWidths = [];
  const range = XLSX.utils.decode_range(worksheet['!ref']);

  for (let C = range.s.c; C <= range.e.c; ++C) {
    let maxWidth = 10;

    for (let R = range.s.r; R <= range.e.r; ++R) {
      const cellAddress = XLSX.utils.encode_cell({ r: R, c: C });
      const cell = worksheet[cellAddress];

      if (cell && cell.v) {
        const cellWidth = cell.v.toString().length;
        maxWidth = Math.max(maxWidth, cellWidth);
      }
    }

    columnWidths.push({ wch: maxWidth + 2 });
  }

  worksheet['!cols'] = columnWidths;

  // Workbook 생성
  const workbook = XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(workbook, worksheet, 'Sheet1');

  // 다운로드
  XLSX.writeFile(workbook, filename);
}
```

## UI/UX 디자인 패턴

### 화면 구성

```
┌─────────────────────────────────────────────┐
│  CSV 변환기 (CSV Converter)                  │
│  CSV, JSON, Excel 파일을 변환하세요          │
├─────────────────────────────────────────────┤
│  입력: [CSV] [JSON] [Excel]                 │
│  출력: [CSV] [JSON] [Excel]                 │
├─────────────────────────────────────────────┤
│  ┌─ 입력 ─────────────────────────────┐    │
│  │                                     │    │
│  │  [파일 업로드] [텍스트 직접 입력]    │    │
│  │                                     │    │
│  │  🗂️ 드래그 앤 드롭 또는 클릭         │    │
│  │  또는 텍스트 붙여넣기                │    │
│  │                                     │    │
│  │  ┌───────────────────────────┐     │    │
│  │  │ name,age,email            │     │    │
│  │  │ John,30,john@example.com  │     │    │
│  │  │ Jane,25,jane@example.com  │     │    │
│  │  └───────────────────────────┘     │    │
│  │                                     │    │
│  │  옵션:                               │    │
│  │  구분자: [,] [Tab] [;] [|] [자동]   │    │
│  │  ☑ 첫 행을 헤더로 사용               │    │
│  │  인코딩: [UTF-8] [EUC-KR]           │    │
│  │                                     │    │
│  │  [변환하기]                          │    │
│  └─────────────────────────────────────┘    │
│                                             │
│  ┌─ 미리보기 ─────────────────────────┐    │
│  │  ┌─────────┬─────┬────────────┐   │    │
│  │  │ name    │ age │ email      │   │    │
│  │  ├─────────┼─────┼────────────┤   │    │
│  │  │ John    │ 30  │ john@...   │   │    │
│  │  │ Jane    │ 25  │ jane@...   │   │    │
│  │  └─────────┴─────┴────────────┘   │    │
│  │  총 2행 3열                         │    │
│  └─────────────────────────────────────┘    │
│                                             │
│  ┌─ 결과 ─────────────────────────────┐    │
│  │  ┌───────────────────────────┐     │    │
│  │  │ [                          │     │    │
│  │  │   {                        │     │    │
│  │  │     "name": "John",        │     │    │
│  │  │     "age": 30,             │     │    │
│  │  │     "email": "john@..."    │     │    │
│  │  │   },                       │     │    │
│  │  │   ...                      │     │    │
│  │  │ ]                          │     │    │
│  │  └───────────────────────────┘     │    │
│  │                                     │    │
│  │  [복사] [CSV 다운로드] [JSON 다운로드] [Excel 다운로드] │
│  └─────────────────────────────────────┘    │
└─────────────────────────────────────────────┘
```

### 변환 경로

```
     CSV ──────→ JSON ──────→ Excel
      ↑           ↓              ↓
      └───────────←──────────────┘
```

## 난이도 & 예상 기간

- **난이도:** 쉬움
- **예상 기간:** 2일
- **실제 기간:** (작업 후 기록)

## 개발 일정

- [ ] Day 1 오전: UI 구성, 파일 업로드
- [ ] Day 1 오후: CSV ↔ JSON 변환 (PapaParse)
- [ ] Day 2 오전: Excel 변환 (SheetJS), 미리보기 테이블
- [ ] Day 2 오후: 다운로드 기능, 구분자 자동 감지, 최적화

## 트래픽 예상

⭐⭐⭐ 높음 - 데이터 분석가, 개발자 타겟

## SEO 키워드

- CSV JSON 변환
- CSV 변환기
- CSV to JSON
- Excel 변환
- 데이터 변환
- JSON to CSV
- CSV to Excel
- 엑셀 변환기

## 이슈 & 해결방안

### 실제 문제점 (경쟁사 분석 & 실무 이슈 기반)

1. **한글 CSV 파일 인코딩 깨짐 (EUC-KR)**
   - 원인: UTF-8로 읽어서 한글 깨짐
   - 해결: 인코딩 자동 감지 또는 선택
   - 코드:
     ```javascript
     function readFileWithEncoding(file, encoding = 'UTF-8') {
       return new Promise((resolve, reject) => {
         const reader = new FileReader();

         reader.onload = (e) => {
           resolve(e.target.result);
         };

         reader.onerror = reject;

         // 인코딩 지정
         reader.readAsText(file, encoding);
       });
     }

     // 사용
     const csvText = await readFileWithEncoding(file, 'EUC-KR');
     const data = Papa.parse(csvText, { header: true }).data;
     ```

2. **CSV 내 쉼표나 줄바꿈 처리 미흡**
   - 원인: 데이터에 구분자가 포함된 경우
   - 해결: PapaParse는 RFC 4180 표준 준수 (따옴표 처리)
   - 코드:
     ```javascript
     // PapaParse는 자동으로 처리
     const csv = `name,description
     "John Doe","He said, ""Hello World""."
     "Jane","Multi-line
     description"`;

     const result = Papa.parse(csv, { header: true });
     console.log(result.data);
     // [
     //   { name: 'John Doe', description: 'He said, "Hello World".' },
     //   { name: 'Jane', description: 'Multi-line\ndescription' }
     // ]
     ```

3. **대용량 CSV 파일 처리 시 브라우저 멈춤**
   - 원인: 수만 행 데이터 처리
   - 해결: 스트리밍 파싱, 청크 단위 처리
   - 코드:
     ```javascript
     function parseCSVStream(file, onData, onComplete) {
       Papa.parse(file, {
         header: true,
         chunk: (results, parser) => {
           // 청크별 처리 (1000행씩)
           onData(results.data);

           // 필요 시 일시 정지
           // parser.pause();
         },
         complete: () => {
           onComplete();
         },
         error: (error) => {
           console.error('CSV 파싱 에러:', error);
         }
       });
     }

     // 사용
     let allData = [];
     parseCSVStream(
       file,
       (chunk) => {
         allData = allData.concat(chunk);
         console.log(`현재까지 ${allData.length}행 로드됨`);
       },
       () => {
         console.log('파싱 완료:', allData.length);
       }
     );
     ```

4. **JSON 형식 검증 부족 (잘못된 JSON 입력)**
   - 원인: 사용자가 잘못된 JSON 입력
   - 해결: JSON.parse() 에러 처리
   - 코드:
     ```javascript
     function validateAndParseJSON(jsonText) {
       try {
         const data = JSON.parse(jsonText);

         // 배열인지 확인
         if (!Array.isArray(data)) {
           throw new Error('JSON은 배열 형식이어야 합니다.');
         }

         // 비어있는지 확인
         if (data.length === 0) {
           throw new Error('JSON 배열이 비어있습니다.');
         }

         // 객체 배열인지 확인
         if (typeof data[0] !== 'object') {
           throw new Error('JSON 배열의 각 요소는 객체여야 합니다.');
         }

         return { success: true, data };
       } catch (error) {
         return {
           success: false,
           error: error.message
         };
       }
     }

     // 사용
     const result = validateAndParseJSON(jsonText);
     if (result.success) {
       const csv = jsonToCsv(result.data);
       showResult(csv);
     } else {
       showError(result.error);
     }
     ```

5. **Excel 파일 크기 제한**
   - 원인: SheetJS는 대용량 파일 느림
   - 해결: 파일 크기 제한 (10MB)
   - 코드:
     ```javascript
     const MAX_FILE_SIZE = 10 * 1024 * 1024; // 10MB

     function validateFileSize(file) {
       if (file.size > MAX_FILE_SIZE) {
         showError(`파일이 너무 큽니다. (최대 10MB, 현재 ${(file.size / 1024 / 1024).toFixed(2)}MB)`);
         return false;
       }
       return true;
     }

     fileInput.addEventListener('change', (e) => {
       const file = e.target.files[0];
       if (!file) return;

       if (!validateFileSize(file)) {
         fileInput.value = '';
         return;
       }

       processFile(file);
     });
     ```

6. **CSV 다운로드 시 한글 파일명 깨짐**
   - 원인: 브라우저 인코딩 문제
   - 해결: encodeURIComponent 사용
   - 코드:
     ```javascript
     function downloadCSV(csvText, filename = 'data.csv') {
       // BOM 추가 (Excel에서 한글 깨짐 방지)
       const BOM = '\uFEFF';
       const blob = new Blob([BOM + csvText], { type: 'text/csv;charset=utf-8;' });

       const link = document.createElement('a');
       link.href = URL.createObjectURL(blob);

       // 한글 파일명 인코딩
       link.download = encodeURIComponent(filename).replace(/%/g, '_');

       link.click();
       URL.revokeObjectURL(link.href);
     }

     // 사용
     downloadCSV(csvText, '데이터.csv');
     ```

7. **테이블 미리보기 성능 저하 (대용량 데이터)**
   - 원인: 수만 행을 DOM에 렌더링
   - 해결: 가상 스크롤 또는 페이지네이션
   - 코드:
     ```javascript
     function renderTablePreview(data, maxRows = 100) {
       const preview = data.slice(0, maxRows);

       let html = '<table><thead><tr>';

       // 헤더
       const headers = Object.keys(preview[0]);
       headers.forEach(header => {
         html += `<th>${escapeHtml(header)}</th>`;
       });

       html += '</tr></thead><tbody>';

       // 행
       preview.forEach(row => {
         html += '<tr>';
         headers.forEach(header => {
           html += `<td>${escapeHtml(row[header] || '')}</td>`;
         });
         html += '</tr>';
       });

       html += '</tbody></table>';

       if (data.length > maxRows) {
         html += `<p>총 ${data.length}행 중 ${maxRows}행만 표시됨</p>`;
       }

       return html;
     }

     function escapeHtml(text) {
       const div = document.createElement('div');
       div.textContent = String(text);
       return div.innerHTML;
     }
     ```

## 개발 로그

### 2025-10-25
- 프로젝트 폴더 생성
- **경쟁사 분석 완료:**
  - ConvertCSV, CSV to JSON Converter, Mr. Data Converter 조사
  - 대부분 광고 많음, 복잡한 UI
  - 차별화: 간단한 UI, 3가지 포맷, 실시간 미리보기
- **라이브러리 조사 완료:**
  - PapaParse (CSV 파싱 - 강력, RFC 4180 준수)
  - SheetJS (Excel 읽기/쓰기)
  - Best practices: 스트리밍 파싱, 인코딩 처리, BOM 추가
- **실제 이슈 파악:**
  - 한글 인코딩 깨짐 (EUC-KR)
  - 대용량 파일 처리
  - JSON 검증 부족
  - Excel 한글 파일명 깨짐
- **UI/UX 패턴:**
  - 3단 구성 (입력 → 미리보기 → 결과)
  - 파일 업로드 + 텍스트 직접 입력
  - 테이블 미리보기
  - 구분자 자동 감지

## 참고 자료

- [PapaParse](https://www.papaparse.com/)
- [SheetJS](https://docs.sheetjs.com/)
- [RFC 4180 - CSV Format](https://tools.ietf.org/html/rfc4180)
- [FileReader API - MDN](https://developer.mozilla.org/en-US/docs/Web/API/FileReader)
- [Blob - MDN](https://developer.mozilla.org/en-US/docs/Web/API/Blob)
