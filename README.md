<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>커피 추출 수율 계산기</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <style>
        body {
            background: #fafafa;
            font-family: 'Pretendard', 'Noto Sans KR', sans-serif;
            margin: 0;
            padding: 0;
        }
        .cafename {
            text-align: center;
            font-size: 2.0rem;
            font-weight: bold;
            color: #795548;
            margin-top: 38px;
            margin-bottom: 10px;
            letter-spacing: 2px;
        }
        .container {
            max-width: 430px;
            margin: 16px auto 40px auto;
            background: #fff;
            border-radius: 16px;
            box-shadow: 0 4px 20px rgba(0,0,0,0.07);
            padding: 32px 18px 24px 18px;
        }
        h1 {
            text-align: center;
            font-size: 1.3rem;
            margin-bottom: 22px;
        }
        label {
            display: block;
            margin: 11px 0 6px 1px;
            font-size: 1.02rem;
        }
        input {
            width: 100%;
            font-size: 1.03rem;
            padding: 7px 12px;
            border-radius: 7px;
            border: 1px solid #ddd;
            margin-bottom: 4px;
            box-sizing: border-box;
        }
        button {
            width: 100%;
            background: #5c524e;
            color: #fff;
            font-size: 1.1rem;
            border: none;
            border-radius: 7px;
            padding: 11px 0;
            margin-top: 18px;
            cursor: pointer;
            transition: background 0.23s;
        }
        button:hover {
            background: #3c3735;
        }
        .result-card {
            background: #f9f6f4;
            border-radius: 10px;
            padding: 18px 11px 5px 11px;
            margin-top: 24px;
            box-shadow: 0 2px 8px rgba(80,80,80,0.08);
        }
        .result-title {
            font-weight: 600;
            font-size: 1.11rem;
            margin-bottom: 7px;
        }
        .score {
            font-size: 1.05rem;
            margin-bottom: 13px;
        }
        .feedback {
            font-size: 1.02rem;
            margin-bottom: 9px;
        }
        .improve {
            font-size: 1.02rem;
            color: #926c3e;
            margin-bottom: 7px;
        }
        .byline {
            text-align: right;
            color: #bbb;
            font-size: 0.85rem;
            margin-top: 12px;
        }
        .auto-note {
            font-size: 0.97rem;
            color: #8a7f74;
            margin-bottom: 8px;
        }
    </style>
</head>
<body>
    <div class="cafename">BCL Roastery Cafe</div>
    <div class="container">
        <h1>커피 추출 수율 계산기</h1>
        <label for="coffeeInput">투입 커피량 (g)</label>
        <input type="number" step="0.1" id="coffeeInput" placeholder="예: 14.0">
        <label for="waterInput">물 사용량 (ml)</label>
        <input type="number" step="1" id="waterInput" placeholder="예: 220">
        <label for="brewInput">실제 추출된 커피량 (ml) <span style="color:#888;font-size:0.93em;">(빈칸이면 자동계산)</span></label>
        <input type="number" step="1" id="brewInput" placeholder="예: 180">
        <label for="tdsInput">추출 농도 (TDS, %)</label>
        <input type="number" step="0.01" id="tdsInput" placeholder="예: 1.35">
        <button type="button" onclick="calculateYield()">계산하기</button>
        <div id="result"></div>
        <div class="byline">SCA 브루잉 차트 기반 자동 피드백</div>
    </div>
    <script>
      // 더 풍성해진 맛 평가 및 개선 팁
      function getTasteAndAdvice(tds, extractionYield) {
        let taste = '';
        let improve = '';
        // 이상적 범위
        if (tds >= 1.20 && tds <= 1.45 && extractionYield >= 18 && extractionYield <= 22) {
          taste = "이상적인 맛과 밸런스! 산미, 단맛, 바디감, 쓴맛이 조화롭게 느껴집니다.";
          improve = "취향에 따라 소폭 분쇄도나 추출시간만 조절해보세요.";
        }
        // 약하고 언더익스트랙션
        else if (tds < 1.2 && extractionYield < 18) {
          taste = "아주 약하고 심심하며, 본연의 풍미와 단맛이 거의 느껴지지 않습니다. 산미나 바디감이 부족합니다.";
          improve = "더 가늘게 분쇄, 추출 시간을 늘리거나, 커피 양을 늘려주세요. 물 온도를 올려도 좋습니다.";
        }
        // 강하고 오버익스트랙션
        else if (tds > 1.45 && extractionYield > 22) {
          taste = "쓴맛, 떫은맛이 강하며 입안이 텁텁하고, 긁는 느낌이 날 수 있습니다. 과다 추출입니다.";
          improve = "분쇄를 굵게 하거나, 추출 시간을 줄이고, 물 온도를 낮추세요. 커피양을 줄이거나 물양을 늘리는 것도 방법입니다.";
        }
        // 약하지만 오버익스트랙션
        else if (tds < 1.2 && extractionYield > 22) {
          taste = "연하지만 쓴맛과 떫은맛이 남아 밍밍하면서도 불쾌한 뒷맛이 남습니다.";
          improve = "커피양을 늘리거나 분쇄를 가늘게, 추출시간을 짧게 조정해보세요.";
        }
        // 강하지만 언더익스트랙션
        else if (tds > 1.45 && extractionYield < 18) {
          taste = "진하지만 풍미가 부족하고, 날카로운 신맛 혹은 쓴맛이 날 수 있습니다.";
          improve = "분쇄를 조금 더 가늘게, 추출시간을 늘려 풍미를 충분히 끌어내세요.";
        }
        // 약함(농도만 낮음)
        else if (tds < 1.2) {
          taste = "약하고 밍밍하며, 커피 맛이 희미합니다.";
          improve = "분쇄를 가늘게, 추출 시간을 늘리거나, 커피양을 늘리세요.";
        }
        // 강함(농도만 높음)
        else if (tds > 1.45) {
          taste = "진하지만 쓴맛이나 떫은맛이 강하게 느껴집니다.";
          improve = "분쇄를 굵게, 추출 시간을 줄이거나, 물양을 늘리세요.";
        }
        // 언더익스트랙션(수율만 낮음)
        else if (extractionYield < 18) {
          taste = "산미가 강하거나, 밝고 가벼운 느낌. 본연의 풍미가 다 나오지 않았을 수 있습니다.";
          improve = "분쇄를 가늘게, 추출 시간을 늘리거나, 물 온도를 높이세요.";
        }
        // 오버익스트랙션(수율만 높음)
        else if (extractionYield > 22) {
          taste = "쓴맛, 떫은맛이 늘어나 균형이 무너집니다.";
          improve = "분쇄를 굵게, 추출 시간을 줄이거나, 물 온도를 낮추세요.";
        }
        // 그 외 애매한 경우
        else {
          taste = "밸런스가 약간 어긋나 있을 수 있습니다.";
          improve = "분쇄도, 추출시간, 커피/물 비율을 조절하여 본인 취향에 맞게 미세 조정해보세요.";
        }
        return {taste, improve};
      }

      function calculateYield() {
        const coffee = parseFloat(document.getElementById('coffeeInput').value);
        const water = parseFloat(document.getElementById('waterInput').value);
        let brew = parseFloat(document.getElementById('brewInput').value);
        const tds = parseFloat(document.getElementById('tdsInput').value);

        // 입력 검증
        if (isNaN(coffee) || coffee <= 0 || isNaN(water) || water <= 0 || isNaN(tds) || tds <= 0) {
          document.getElementById('result').innerHTML = `<div class="result-card"><span style="color:red;">모든 값을 올바르게 입력해주세요.<br>(커피량, 물양, 농도는 필수입니다)</span></div>`;
          return;
        }

        let autoNote = '';
        // 실제 추출량이 비어있으면 자동 계산 (물 사용량 - 투입 커피량 x 2)
        if (isNaN(brew) || brew <= 0) {
          brew = water - coffee * 2;
          if (brew <= 0) {
            brew = 0;
            autoNote = `<div class="auto-note">※ 자동 계산된 추출량이 0 이하로 나와 계산할 수 없습니다.<br>입력값을 확인해주세요.</div>`;
          } else {
            autoNote = `<div class="auto-note">※ 추출된 커피량이 입력되지 않아, 물 사용량 - 투입 커피량×2 (${water} - ${coffee * 2} = ${brew}ml)로 자동 계산합니다.</div>`;
          }
        }

        if (brew <= 0) {
          document.getElementById('result').innerHTML = `<div class="result-card">${autoNote}<span style="color:red;">추출 커피량이 0 이하로 계산되어 수율을 산출할 수 없습니다. 입력값을 확인해주세요.</span></div>`;
          return;
        }

        // 커피에 남아있는 수분량 계산 (커피 투입량 x 2.1) - 참고용
        // const retainedWater = coffee * 2.1;

        // 수율 계산 공식: (추출량 × 농도) / 투입량
        const extractionYield = ((brew * tds) / coffee).toFixed(2);

        // 평가 및 피드백
        const fb = getTasteAndAdvice(tds, extractionYield);

        document.getElementById('result').innerHTML = `
          <div class="result-card">
            ${autoNote}
            <div class="result-title">수율(Extraction Yield): <span style="color:#926c3e">${extractionYield}%</span></div>
            <div class="result-title">농도(Strength, TDS): <span style="color:#926c3e">${tds}%</span></div>
            <div class="score">예상 맛 평가: <b>${fb.taste}</b></div>
            <div class="improve">개선 방법: ${fb.improve}</div>
            <div class="feedback">※ SCA 기준: <br>수율 18~22%, 농도 1.20~1.45%가 ‘최적 밸런스’입니다.</div>
          </div>
        `;
      }
    </script>
</body>
</html>
