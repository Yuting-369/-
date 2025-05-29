<!DOCTYPE html>
<html lang="zh">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>拼音消消乐 - 熊猫主题</title>
<style>
  body {
    font-family: "Microsoft YaHei", Arial, sans-serif;
    background: #e0f7fa;
    margin: 0; padding: 0;
    display: flex;
    flex-direction: column;
    align-items: center;
  }
  header {
    width: 100%;
    background: #80deea;
    padding: 1em;
    text-align: center;
    color: #004d40;
    font-weight: bold;
    font-size: 1.2em;
  }
  #input-area {
    margin: 1em;
    text-align: center;
  }
  #input-area input {
    font-size: 1.2em;
    margin: 0.5em;
    padding: 0.3em;
  }
  #start-btn {
    background: #00796b;
    color: white;
    border: none;
    padding: 0.6em 1.2em;
    border-radius: 8px;
    font-size: 1.2em;
    cursor: pointer;
  }
  #timer {
    margin: 1em 0;
    font-size: 1.5em;
    color: #004d40;
  }
  #game-board {
    width: 360px;
    display: grid;
    grid-template-columns: repeat(6, 1fr);
    gap: 8px;
    margin-bottom: 1em;
  }
  .card {
    width: 56px;
    height: 56px;
    background: #fff;
    border-radius: 10px;
    box-shadow: 0 0 8px #009688;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 18px;
    cursor: pointer;
    user-select: none;
    position: relative;
    color: transparent;
    transition: background-color 0.3s, color 0.3s;
  }
  .card.flipped {
    background: #004d40;
    color: #fff;
  }
  .card.matched {
    background: #a5d6a7;
    color: #004d40;
    cursor: default;
  }
  footer {
    font-size: 1em;
    color: #004d40;
    margin-bottom: 2em;
  }
</style>
</head>
<body>

<header>🐼 拼音消消乐 - 熊猫主题</header>

<div id="input-area">
  <input type="text" id="class-input" placeholder="반 입력 (班级)" />
  <input type="text" id="name-input" placeholder="이름 입력 (姓名)" />
  <button id="start-btn">게임 시작</button>
</div>

<div id="timer" style="display:none;">시간: 0초</div>

<div id="game-board" style="display:none;"></div>

<footer>© 2025 拼音消消乐</footer>

<script>
  const words = [
    "你(nǐ)", "我(wǒ)", "他(tā)", "她(tā)", "你们(nǐmen)", "我们(wǒmen)", "她们(tāmen)", "他们(tāmen)", "您(nín)",
    "早上(zǎoshang)", "晚上(wǎnshang)", "上午(shàngwǔ)", "中午(zhōngwǔ)", "下午(xiàwǔ)",
    "您好(nín hǎo)", "你好(nǐ hǎo)", "老师(lǎoshī)", "大家(dàjiā)",
    "你(nǐ)", "我(wǒ)", "他(tā)", "她(tā)", "你们(nǐmen)", "我们(wǒmen)", "她们(tāmen)", "他们(tāmen)", "您(nín)",
    "早上(zǎoshang)", "晚上(wǎnshang)", "上午(shàngwǔ)", "中午(zhōngwǔ)", "下午(xiàwǔ)",
    "您好(nín hǎo)", "你好(nǐ hǎo)", "老师(lǎoshī)", "大家(dàjiā)"
  ];

  let gameStarted = false;
  let timerInterval = null;
  let timeElapsed = 0;
  let flippedCards = [];
  let matchedCount = 0;

  const startBtn = document.getElementById("start-btn");
  const timerDisplay = document.getElementById("timer");
  const gameBoard = document.getElementById("game-board");
  const classInput = document.getElementById("class-input");
  const nameInput = document.getElementById("name-input");

  function shuffle(array) {
    for (let i = array.length -1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i+1));
      [array[i], array[j]] = [array[j], array[i]];
    }
  }

  function createCards() {
    gameBoard.innerHTML = "";
    const shuffledWords = [...words];
    shuffle(shuffledWords);

    shuffledWords.forEach((word, idx) => {
      const card = document.createElement("div");
      card.classList.add("card");
      card.dataset.word = word;
      card.style.backgroundColor = `hsl(${Math.floor(Math.random() * 360)}, 70%, 80%)`;
      card.textContent = word;
      card.addEventListener("click", () => onCardClick(card));
      gameBoard.appendChild(card);
    });
  }

  function onCardClick(card) {
    if (!gameStarted) return;
    if (flippedCards.length >= 2 || card.classList.contains("flipped") || card.classList.contains("matched")) return;

    card.classList.add("flipped");
    flippedCards.push(card);

    if (flippedCards.length === 2) {
      checkMatch();
    }
  }

  function checkMatch() {
    const [card1, card2] = flippedCards;
    if (card1.dataset.word === card2.dataset.word) {
      card1.classList.add("matched");
      card2.classList.add("matched");
      matchedCount += 2;
      flippedCards = [];

      if (matchedCount === words.length) {
        endGame();
      }
    } else {
      setTimeout(() => {
        card1.classList.remove("flipped");
        card2.classList.remove("flipped");
        flippedCards = [];
      }, 800);
    }
  }

  function startTimer() {
    timeElapsed = 0;
    timerDisplay.textContent = `시간: 0초`;
    timerInterval = setInterval(() => {
      timeElapsed++;
      timerDisplay.textContent = `시간: ${timeElapsed}초`;
    }, 1000);
  }

  function stopTimer() {
    clearInterval(timerInterval);
  }

  function endGame() {
    stopTimer();
    alert(`축하합니다! 게임 완료 시간: ${timeElapsed}초`);
    uploadResult();
  }

  async function uploadResult() {
    const className = classInput.value.trim();
    const studentName = nameInput.value.trim();

    if (!className || !studentName) {
      alert("班级和姓名不能为空！");
      return;
    }

    // 这里是示范的 API 地址，真实部署后替换成你的后端 URL
    const apiUrl = "https://your-backend-url/api/submit";

    try {
      const res = await fetch(apiUrl, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ class: className, name: studentName, time: timeElapsed }),
      });
      if (res.ok) {
        alert("成绩上传成功！");
      } else {
        alert("成绩上传失败！");
      }
    } catch (error) {
      alert("上传错误，请检查网络连接！");
    }
  }

  startBtn.addEventListener("click", () => {
    if (!classInput.value.trim() || !nameInput.value.trim()) {
      alert("请输入班级和姓名（韩语）！");
      return;
    }
    startBtn.style.display = "none";
    classInput.style.display = "none";
    nameInput.style.display = "none";
    timerDisplay.style.display = "block";
    gameBoard.style.display = "grid";

    createCards();
    gameStarted = true;
    startTimer();
  });
</script>

</body>
</html>
