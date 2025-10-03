# saju-webapp-3
# ----------------------------------------------------
# Sajuhub-Pro: 완전 자동화 운세 홈페이지 프로젝트
# ----------------------------------------------------

# 1. 프로젝트 폴더 만들기
New-Item -ItemType Directory -Path .\sajuhub-pro
Set-Location .\sajuhub-pro

# 2. index.html 생성 (오늘 운세 + 사주 + 궁합)
@'
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>사주 허브 프로</title>
  <script src="js/main.js" defer></script>
  <link href="css/tailwind.css" rel="stylesheet">
</head>
<body class="bg-gray-50">
  <header class="p-4 bg-yellow-200 text-center font-bold text-xl">사주 허브 프로</header>

  <main class="max-w-3xl mx-auto p-4">
    <!-- 오늘 운세 -->
    <section class="card p-4 bg-white rounded shadow mt-4">
      <h2 class="font-semibold mb-2">오늘의 운세</h2>
      <button onclick="showTodayFortune()" class="bg-yellow-400 px-3 py-1 rounded">오늘 운세 보기</button>
      <div id="todayResult" class="mt-2"></div>
    </section>

    <!-- 단일 사주 -->
    <section class="card p-4 bg-white rounded shadow mt-4">
      <h2 class="font-semibold mb-2">단일 사주 보기</h2>
      <label>이름: <input id="name" class="border px-2 py-1 rounded"></label>
      <label>생년월일: <input type="date" id="birthday1" class="border px-2 py-1 rounded"></label>
      <label>달력:
        <select id="calendar1" class="border px-2 py-1 rounded">
          <option value="solar">양력</option>
          <option value="lunar">음력</option>
        </select>
      </label>
      <button onclick="getSaju()" class="bg-yellow-400 px-3 py-1 rounded mt-2">계산하기</button>
      <div id="result1" class="mt-2"></div>
    </section>

    <!-- 궁합 -->
    <section class="card p-4 bg-white rounded shadow mt-4">
      <h2 class="font-semibold mb-2">두 사람 궁합</h2>
      <label>사람1 생년월일: <input type="date" id="p1" class="border px-2 py-1 rounded"></label>
      <label>달력:
        <select id="p1cal" class="border px-2 py-1 rounded">
          <option value="solar">양력</option>
          <option value="lunar">음력</option>
        </select>
      </label>
      <label>사람2 생년월일: <input type="date" id="p2" class="border px-2 py-1 rounded"></label>
      <label>달력:
        <select id="p2cal" class="border px-2 py-1 rounded">
          <option value="solar">양력</option>
          <option value="lunar">음력</option>
        </select>
      </label>
      <button onclick="getCompatibility()" class="bg-yellow-400 px-3 py-1 rounded mt-2">궁합 확인</button>
      <div id="result2" class="mt-2"></div>
    </section>
  </main>
</body>
</html>
'@ | Out-File -Encoding UTF8 index.html

# 3. Tailwind CSS placeholder 생성
New-Item -ItemType Directory -Path .\css
@'/* Tailwind CSS 빌드 파일 placeholder */'@ | Out-File -Encoding UTF8 .\css\tailwind.css

# 4. JS 공용 파일 생성
New-Item -ItemType Directory -Path .\js
@'
async function getSaju() {
  const name = document.getElementById("name").value;
  const date = document.getElementById("birthday1").value;
  const calendar = document.getElementById("calendar1").value;
  const res = await fetch(`/api/saju?date=${date}&calendar=${calendar}`);
  const data = await res.json();
  document.getElementById("result1").innerHTML = `<p>${name}님 생일(${calendar}): ${data.solar}</p><p>간지: ${data.ganji}</p>`;
}

async function getCompatibility() {
  const p1 = document.getElementById("p1").value;
  const c1 = document.getElementById("p1cal").value;
  const p2 = document.getElementById("p2").value;
  const c2 = document.getElementById("p2cal").value;
  const res = await fetch(`/api/compatibility?person1=${p1}&calendar1=${c1}&person2=${p2}&calendar2=${c2}`);
  const data = await res.json();
  document.getElementById("result2").innerHTML =
    `<p>사람1: ${data.person1.solar} (${data.person1.ganji})</p>
     <p>사람2: ${data.person2.solar} (${data.person2.ganji})</p>
     <p>점수: ${data.compatibilityScore}</p>
     <p>결과: ${data.result}</p>`;
}

async function showTodayFortune() {
  const today = new Date().toISOString().slice(0,10);
  const res = await fetch(`/api/fortune?date=${today}`);
  const data = await res.json();
  document.getElementById("todayResult").innerHTML = `<p>오늘 운세: ${data.message}</p>`;
}
'@ | Out-File -Encoding UTF8 .\js\main.js

# 5. API 폴더와 파일 생성
New-Item -ItemType Directory -Path .\api

# a) saju.js
@'
import { KoreanLunarCalendar } from "korean-lunar-calendar";

export default function handler(req, res) {
  const { date, calendar } = req.query;
  const [year, month, day] = date.split("-").map(Number);
  const cal = new KoreanLunarCalendar();
  if(calendar==="lunar"){ cal.setLunar(year,month,day,false); }
  else{ cal.setSolar(year,month,day); }
  res.status(200).json({ solar:`${cal.solarYear}-${cal.solarMonth}-${cal.solarDay}`, lunar:`${cal.lunarYear}-${cal.lunarMonth}-${cal.lunarDay}`, ganji:cal.getGapJaString() });
}
'@ | Out-File -Encoding UTF8 .\api\saju.js

# b) compatibility.js
@'
import { KoreanLunarCalendar } from "korean-lunar-calendar";

function getSaju(date, calendar){
  const [year,month,day] = date.split("-").map(Number);
  const cal = new KoreanLunarCalendar();
  if(calendar==="lunar"){ cal.setLunar(year,month,day,false); }
  else{ cal.setSolar(year,month,day); }
  return { solar:`${cal.solarYear}-${cal.solarMonth}-${cal.solarDay}`, lunar:`${cal.lunarYear}-${cal.lunarMonth}-${cal.lunarDay}`, ganji:cal.getGapJaString() };
}

export default function handler(req,res){
  const { person1,calendar1="solar",person2,calendar2="solar"} = req.query;
  const saju1 = getSaju(person1,calendar1);
  const saju2 = getSaju(person2,calendar2);
  const score=Math.floor(Math.random()*100);
  let message="무난한 궁합입니다.";
  if(score>70) message="아주 좋은 궁합이에요!";
  if(score<40) message="노력과 이해가 필요합니다.";
  res.status(200).json({ person1:saju1, person2:saju2, compatibilityScore:score, result:message });
}
'@ | Out-File -Encoding UTF8 .\api\compatibility.js

# c) fortune.js
@'
export default function handler(req,res){
  const messages=["행운이 가득한 하루입니다!","조금 주의가 필요한 하루입니다.","오늘은 중요한 결정을 미루는 것이 좋습니다."];
  const msg = messages[Math.floor(Math.random()*messages.length)];
  res.status(200).json({ message: msg });
}
'@ | Out-File -Encoding UTF8 .\api\fortune.js

# 6. package.json 생성
@'
{
  "name":"sajuhub-pro",
  "version":"1.0.0",
  "dependencies":{
    "korean-lunar-calendar":"^1.0.0"
  }
}
'@ | Out-File -Encoding UTF8 package.json

# 7. vercel.json (자동 라우팅 포함)
@'
{
  "version":2,
  "builds":[
    { "src":"index.html","use":"@vercel/static" },
    { "src":"api/*.js","use":"@vercel/node" }
  ],
  "routes":[
    { "src":"/fortune","dest":"/api/fortune" },
    { "src":"/today","dest":"/index.html" },
    { "src":"/compatibility","dest":"/api/compatibility" }
  ]
}
'@ | Out-File -Encoding UTF8 vercel.json

# 8. README.md 생성
@'
# Sajuhub-Pro

한국형 종합 운세 홈페이지
- 오늘 운세 / 단일 사주 / 두 사람 궁합
- 향후 띠별, 별자리, 혈액형 운세 확장 가능
'@ | Out-File -Encoding UTF8 README.md

# 9. Git 초기화 + 커밋
git init
git add .
git commit -m "Initial commit: Sajuhub-Pro 완전 자동화 프로젝트"
git branch -M main

Write-Host "⚠️ GitHub 원격 주소를 추가하고 git push 하세요:"
Write-Host "git remote add origin https://github.com/YOUR_USERNAME/sajuhub-pro.git"
Write-Host "git push -u origin main"
