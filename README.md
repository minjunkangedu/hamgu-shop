<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <title>햄구신교 상점</title>
  <style>
    body { font-family: 'Arial', sans-serif; background: #fffaf0; color: #333; text-align: center; padding: 20px; }
    h1 { font-size: 2.5em; margin-bottom: 10px; }
    .item, .gift, .history, .leaderboard { border: 2px solid #ccc; padding: 15px; margin: 10px auto; width: 300px; border-radius: 10px; background: #fdfdfd; }
    .price { font-weight: bold; color: #d2691e; }
    input, button { padding: 8px; margin-top: 5px; }
    .small { font-size: 0.9em; color: #888; }
  </style>
</head>
<body>

  <h1>✨ 햄구신교 상점 ✨</h1>
  <p>Hamgu는 거룩할지어다!</p>

  <div class="item">
    <h2>🍎 황금사과</h2>
    <p class="price">가격: 2 HBC</p>
    <button onclick="buyItem('황금사과', 2)">구매</button>
  </div>

  <div class="item">
    <h2>🍀 럭잼 (10~30%)</h2>
    <p class="price">가격: 4 HBC</p>
    <button onclick="buyItem('럭잼', 4)">구매</button>
  </div>

  <div class="item">
    <h2>🧪 경험치 병</h2>
    <p class="price">가격: 10 HBC</p>
    <button onclick="buyItem('경험치 병', 10)">구매</button>
  </div>

  <div class="item">
    <h2>🎉 이벤트 티켓</h2>
    <p class="price">가격: 20 HBC</p>
    <button onclick="buyItem('이벤트 티켓', 20)">구매</button>
  </div>

  <div class="item">
    <h2>🎲 랜덤 아이템 상자</h2>
    <p>무작위로 하나의 아이템을 받습니다!</p>
    <p class="price">가격: 3 HBC</p>
    <button onclick="openRandomBox()">상자 열기</button>
  </div>

  <div class="gift">
    <h2>🎁 코인 선물</h2>
    <input id="targetUser" placeholder="상대 이름">
    <input id="amount" type="number" placeholder="수량">
    <button onclick="gift()">선물</button>
  </div>

  <div class="gift">
    <h2>🔧 관리자 지급</h2>
    <input id="adminTarget" placeholder="유저 이름">
    <input id="adminAmount" type="number" placeholder="코인 수량 (예: 10)">
    <button onclick="adminGiveCoin()">코인 지급</button>
    <button onclick="adminGiveBox()">랜덤상자 지급</button>
    <input id="adminPass" type="password" placeholder="관리자 비밀번호">
  </div>

  <div class="history">
    <h2>📜 구매 내역</h2>
    <pre id="historyLog"></pre>
  </div>

  <div class="leaderboard">
    <h2>🏆 코인 보유 순위</h2>
    <pre id="leaderboardLog"></pre>
  </div>

  <div>
    <p class="small">현재 사용자: <span id="username"></span></p>
    <p class="small">잔액: <span id="balance"></span> HBC</p>
  </div>

  <div class="history">
    <h2>🗞 공용 상점 로그</h2>
    <pre id="globalLog"></pre>
  </div>

  <script>
    const PASSWORD = "komq3244";
    const storage = window.localStorage;
    
    function getUser() {
      let name = prompt("당신의 이름을 입력하세요:");
      if (!name) name = "이름없음";
      document.getElementById("username").innerText = name;
      return name;
    }

    function getBalance(user) {
      return parseFloat(storage.getItem(user + "_balance") || "1");
    }

    function setBalance(user, amount) {
      storage.setItem(user + "_balance", amount.toString());
    }

    function logPurchase(user, item, cost) {
      let history = storage.getItem(user + "_history") || "";
      history += `✅ ${item} (${cost} HBC)\n`;
      storage.setItem(user + "_history", history);
    }

    function logGlobal(message) {
      const now = new Date().toLocaleString(); // 시간 형식: YYYY-MM-DD HH:MM:SS
      let log = storage.getItem("global_log") || "";
      let logLines = log.split("\n").filter(line => line.trim() !== "");
      logLines.push(`[${now}] ${message}`);

      // 최대 50줄 유지
      if (logLines.length > 50) {
        logLines = logLines.slice(logLines.length - 50);
      }

      storage.setItem("global_log", logLines.join("\n"));
    }

    function showHistory() {
      const user = getUser();
      const history = storage.getItem(user + "_history") || "(없음)";
      document.getElementById("historyLog").innerText = history;
    }

    function showGlobalLog() {
      document.getElementById("globalLog").innerText = storage.getItem("global_log") || "(아직 로그 없음)";
    }

    function updateDisplay() {
      const user = getUser();
      document.getElementById("balance").innerText = getBalance(user);
    }

    function updateLeaderboard() {
      const keys = Object.keys(storage);
      const users = keys.filter(k => k.endsWith("_balance"))
        .map(k => ({
          name: k.replace("_balance", ""),
          bal: parseFloat(storage.getItem(k))
        }))
        .sort((a, b) => b.bal - a.bal)
        .slice(0, 5);
      document.getElementById("leaderboardLog").innerText = users.map(u => `${u.name}: ${u.bal} HBC`).join("\n");
    }

    function buyItem(item, cost) {
      const user = getUser();
      let balance = getBalance(user);
      if (balance >= cost) {
        setBalance(user, balance - cost);
        logPurchase(user, item, cost);
        logGlobal(`🛒 ${user}님이 '${item}'을(를) ${cost} HBC에 구매했습니다.`);
        alert(`${item}을(를) 구매했습니다.`);
        updateDisplay();
        showHistory();
        showGlobalLog();
        updateLeaderboard();
      } else {
        alert("HBC가 부족합니다.");
      }
    }

    function gift() {
      const user = getUser();
      const target = document.getElementById("targetUser").value;
      const amount = parseFloat(document.getElementById("amount").value);
      if (!target || isNaN(amount) || amount <= 0) return alert("올바르게 입력해주세요.");
      if (getBalance(user) < amount) return alert("잔액 부족!");

      setBalance(user, getBalance(user) - amount);
      setBalance(target, getBalance(target) + amount);
      alert(`${target}에게 ${amount} HBC를 보냈습니다.`);
      updateDisplay();
      showGlobalLog();
      updateLeaderboard();
    }

    function adminAuth() {
      return document.getElementById("adminPass").value === PASSWORD;
    }

    function adminGiveCoin() {
      if (!adminAuth()) return alert("관리자 비밀번호가 틀렸습니다.");

      const target = document.getElementById("adminTarget").value;
      const amount = parseFloat(document.getElementById("adminAmount").value);

      if (!target || isNaN(amount) || amount <= 0) return alert("입력 오류");

      setBalance(target, getBalance(target) + amount);
      logGlobal(`🔧 관리자님이 ${target}에게 ${amount} HBC를 지급했습니다.`);
      alert(`${target}에게 ${amount} HBC 지급 완료`);
      updateLeaderboard();
    }

    function adminGiveBox() {
      if (!adminAuth()) return alert("관리자 비밀번호가 틀렸습니다.");

      const target = document.getElementById("adminTarget").value;
      if (!target) return alert("유저 이름을 입력해주세요.");

      logGlobal(`🔧 관리자님이 ${target}에게 랜덤상자를 지급했습니다.`);

      let history = storage.getItem(target + "_history") || "";
      history += `✅ 관리자 지급 → 랜덤상자\n`;
      storage.setItem(target + "_history", history);

      alert(`${target}에게 랜덤상자 지급 완료`);
    }

    function openRandomBox() {
      const user = getUser();
      const balance = getBalance(user);
      if (balance < 3) {
        alert("HBC가 부족합니다.");
        return;
      }
      setBalance(user, balance - 3);

      const rewards = [
        { item: '꽝', weight: 30 },
        { item: '₩10', won: 10, weight: 10 },
        { item: '₩100', won: 100, weight: 5 },
        { item: '₩1000', won: 1000, weight: 2 },
        { item: '₩5000', won: 5000, weight: 1 },
        { item: '황금사과', weight: 3 },
        { item: '황금당근', weight: 2 },
        { item: '고양이 스폰알', weight: 5 },
        { item: '말 스폰알', weight: 1 },
        { item: '낙타 스폰알', weight: 1 }
      ];

      const totalWeight = rewards.reduce((sum, r) => sum + r.weight, 0);
      let rand = Math.random() * totalWeight;
      let chosen = null;

      for (const reward of rewards) {
        if (rand < reward.weight) {
          chosen = reward;
          break;
        }
        rand -= reward.weight;
      }

      if (!chosen) chosen = { item: '꽝' };

      if (
