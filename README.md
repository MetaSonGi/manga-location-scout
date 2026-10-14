# 🎬 만화 로케헌 (manga-location-scout) v1.0

실제 지도 + 표고 데이터를 3D 만화 배경(선화·스크린톤)으로 그리는 웹앱.
[どこでもロケハン](https://tools.oyasumi-gamedev.com/rokehan/) 아이디어의 브라우저판.
100% 브라우저에서 동작하며 외부 서버가 없다.

## 사용법

1. **장소 설정**: 장소 이름, 위도/경도 입력 (기본값: 서울 북촌한옥마을 37.5826, 126.9837)
2. **범위 선택**: 200m / 300m / 500m / 1km — 프리셋 4곳(북촌한옥마을·홍대입구·해운대·황리단길)도 지원
3. **불러오기**: 건물 footprint + 표고를 가져와 만화 배경으로 렌더링
4. **스타일**: 스크린톤 on/off, 선 굵기(1~4px), 수평선 on/off, 자동 회전, 회전 리셋
5. **출력**: PNG 다운로드 (1x/2x, `{이름}_manga.png`), 링크 복사 (`#ll=위도,경도&s=범위&yaw=각도&n=이름` 해시 공유)

조작: 드래그로 회전, 마우스 휠 / 두 손가락 핀치로 줌.

## 데이터·기술

- **건물**: Overpass API (`overpass-api.de`)에서 `way["building"]` footprint 조회.
  높이는 `height` 태그 → `building:levels` × 3m → 기본 8m 순으로 결정.
  건물이 1500개를 넘으면 범위 축소 안내 (서버·브라우저 부하 방지).
- **표고**: Terrarium 타일 (`elevation-tiles-prod`, z=14)을 24×24 격자로 샘플링.
  디코드: `h = R×256 + G + B/256 − 32768`.
- **렌더링**: 외부 3D 라이브러리 없이 순수 Canvas 2D.
  yaw 회전(드래그) + 고정 pitch 35° + 약한 투시 투영(`s = f/(f−z)`, f = 화면 크기의 2.5배).
  건물 footprint를 높이만큼 extrude → 면 단위 painter's algorithm(depth sort).
  광원(좌상단)을 등진 벽면에는 하프톤 도트 스크린톤 오버레이.
- **공유 링크**: URL 해시에 장소·범위·회전각을 인코딩 (원본 도구와 같은 형식).

## 제약·주의사항

- **표고 폴백**: 표고 타일 로드 실패(CORS·네트워크·파싱 오류)는 전부 조용히 잡아서 **평지(0m)로 렌더링**한다. 앱이 절대 멈추지 않음. 표고가 없으면 지면은 평지 + 옅은 격자로 표시.
- Overpass API는 공개 API라 간헐적으로 느리거나 타임아웃될 수 있음 (30초 타임아웃 설정).
- 건물 데이터가 없는 지역(산·바다)은 지면만 그려짐.
- `ctx.letterSpacing` 미사용 — 자간 기능 없음.

## 데이터 출처

- 건물: © [OpenStreetMap](https://www.openstreetmap.org/copyright) contributors (via Overpass API)
- 표고: Terrarium 타일 (`s3.amazonaws.com/elevation-tiles-prod`)

## 실행

`index.html`을 브라우저에서 열거나 GitHub Pages로 배포하면 바로 사용 가능.
