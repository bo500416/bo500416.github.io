---
title: "跨标签页倒计时提醒" # in any language you want
layout: "count" # is necessary
# url: "/count"
# description: "Description for Search"
summary: "count"
---

<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">    
    <style>
        :root {
            --primary: #2563eb;
            --bg: #f8fafc;
            --card: #ffffff;
            --text: #1e293b;
        }

        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
            background-color: var(--bg);
            color: var(--text);
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
        }

        .card {
            background: var(--card);
            padding: 2.5rem;
            border-radius: 1.5rem;
            box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.1);
            text-align: center;
            width: 350px;
        }

        h1 { font-size: 1.5rem; margin-bottom: 1.5rem; }

        .inputs {
            display: flex;
            gap: 10px;
            justify-content: center;
            margin-bottom: 1.5rem;
        }

        .input-box {
            display: flex;
            flex-direction: column;
        }

        .input-box label {
            font-size: 0.75rem;
            color: #64748b;
            margin-bottom: 4px;
        }

        input {
            width: 60px;
            padding: 10px;
            border: 1px solid #e2e8f0;
            border-radius: 0.5rem;
            text-align: center;
            font-size: 1.2rem;
        }

        .display {
            font-size: 3.5rem;
            font-weight: 700;
            font-family: monospace;
            margin: 1.5rem 0;
            color: var(--primary);
        }

        .btn-group {
            display: flex;
            gap: 10px;
        }

        button {
            flex: 1;
            padding: 12px;
            border: none;
            border-radius: 0.5rem;
            font-weight: 600;
            cursor: pointer;
            transition: all 0.2s;
        }

        #startBtn { background: var(--primary); color: white; }
        #startBtn:hover { background: #1d4ed8; }
        #resetBtn { background: #e2e8f0; color: #475569; }
        #resetBtn:hover { background: #cbd5e1; }
        button:disabled { opacity: 0.5; cursor: not-allowed; }

        .status {
            margin-top: 1rem;
            font-size: 0.85rem;
            color: #ef4444;
            height: 1.2rem;
        }
    </style>
</head>
<body>

<div class="card">
    <h1>倒计时提醒</h1>
    
    <div class="inputs">
        <div class="input-box">
            <label>时</label>
            <input type="number" id="h" min="0" value="0">
        </div>
        <div class="input-box">
            <label>分</label>
            <input type="number" id="m" min="0" max="59" value="0">
        </div>
        <div class="input-box">
            <label>秒</label>
            <input type="number" id="s" min="0" max="59" value="0">
        </div>
    </div>

    <div class="display" id="timerDisplay">00:00:00</div>

    <div class="btn-group">
        <button id="startBtn" onclick="startTimer()">开始倒计时</button>
        <button id="resetBtn" onclick="resetTimer()">重置</button>
    </div>

    <div class="status" id="statusMsg"></div>
</div>

<script>
    let countdownInterval;
    let targetTime; // 目标结束的时间戳

    // 1. 请求通知权限
    function requestPermission() {
        if (!("Notification" in window)) {
            document.getElementById('statusMsg').innerText = "此浏览器不支持通知功能";
            return false;
        }
        if (Notification.permission !== "granted") {
            Notification.requestPermission().then(permission => {
                if (permission !== "granted") {
                    document.getElementById('statusMsg').innerText = "请允许通知权限以实现后台提醒";
                }
            });
        }
        return true;
    }

    function startTimer() {
        // 每次开始前先请求权限
        if (!requestPermission()) return;

        const h = parseInt(document.getElementById('h').value) || 0;
        const m = parseInt(document.getElementById('m').value) || 0;
        const s = parseInt(document.getElementById('s').value) || 0;

        const totalSeconds = h * 3600 + m * 60 + s;

        if (totalSeconds <= 0) {
            document.getElementById('statusMsg').innerText = "请输入有效时间";
            return;
        }

        document.getElementById('statusMsg').innerText = "";
        document.getElementById('startBtn').disabled = true;

        // 核心逻辑：计算目标结束的时间点（Date.now + 毫秒数）
        // 这样即便标签页进入后台被浏览器“休眠”，时间计算依然是准确的
        targetTime = Date.now() + (totalSeconds * 1000);

        // 清除旧的定时器
        clearInterval(countdownInterval);

        // 启动定时器
        countdownInterval = setInterval(updateTimer, 1000);
    }

    function updateTimer() {
        const now = Date.now();
        const remaining = targetTime - now;

        if (remaining <= 0) {
            // 时间到了
            clearInterval(countdownInterval);
            finishTimer();
            return;
        }

        // 更新显示
        const h = Math.floor(remaining / 3600000);
        const m = Math.floor((remaining % 3600000) / 60000);
        const s = Math.floor((remaining % 60000) / 1000);
        
        document.getElementById('timerDisplay').textContent = 
            `${String(h).padStart(2, '0')}:${String(m).padStart(2, '0')}:${String(s).padStart(2, '0')}`;
    }

    function finishTimer() {
        document.getElementById('timerDisplay').textContent = "00:00:00";
        document.getElementById('startBtn').disabled = false;

        // 触发系统级通知 (跨标签页提醒)
        if (Notification.permission === "granted") {
            new Notification("倒计时结束", {
                body: "您设置的时间已经到了！",
                icon: "https://cdn-icons-png.flaticon.com/512/3602/3602145.png" // 一个闹钟图标
            });
        } else {
            // 如果没给权限，只能在当前页面弹窗
            alert("时间到！(由于未开启通知权限，无法进行系统提醒)");
        }
    }

    function resetTimer() {
        clearInterval(countdownInterval);
        document.getElementById('startBtn').disabled = false;
        document.getElementById('timerDisplay').textContent = "00:00:00";
        document.getElementById('h').value = 0;
        document.getElementById('m').value = 0;
        document.getElementById('s').value = 0;
        document.getElementById('statusMsg').innerText = "";
    }
</script>

</body>
</html>
