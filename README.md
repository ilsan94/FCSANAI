<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>FC사나이 풋살 매치 밸런서</title>
    <style>
        :root {
            --bg-main: #0f172a;
            --bg-card: #1e293b;
            --bg-input: #334155;
            --primary: #38bdf8;
            --accent: #f59e0b;
            --danger: #ef4444;
            --text-light: #f8fafc;
            --text-muted: #94a3b8;
        }
        * { box-sizing: border-box; margin: 0; padding: 0; font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif; }
        body { background-color: var(--bg-main); color: var(--text-light); padding: 16px; display: flex; justify-content: center; }
        .app-container { width: 100%; max-width: 600px; display: flex; flex-direction: column; gap: 16px; }
        h1 { font-size: 1.5rem; text-align: center; color: var(--primary); display: flex; align-items: center; justify-content: center; gap: 8px; }
        .card { background: var(--bg-card); border-radius: 12px; padding: 14px; box-shadow: 0 4px 12px rgba(0,0,0,0.3); }
        .card-header { font-size: 1rem; font-weight: bold; margin-bottom: 10px; display: flex; justify-content: space-between; align-items: center; color: var(--primary); }
        
        /* 탭 버튼 */
        .tabs { display: flex; gap: 8px; margin-bottom: 8px; }
        .tab-btn { flex: 1; padding: 10px; background: var(--bg-card); border: 1px solid var(--bg-input); color: var(--text-muted); border-radius: 8px; cursor: pointer; font-weight: bold; font-size: 0.9rem; }
        .tab-btn.active { background: var(--primary); color: #0f172a; border-color: var(--primary); }

        /* 선수 관리 & 출석 리스트 */
        .player-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(130px, 1fr)); gap: 8px; }
        .player-item { background: var(--bg-input); padding: 8px; border-radius: 8px; display: flex; flex-direction: column; gap: 4px; font-size: 0.85rem; }
        .attendance-item { display: flex; align-items: center; gap: 6px; padding: 8px; background: var(--bg-input); border-radius: 6px; cursor: pointer; user-select: none; font-size: 0.85rem; }
        .attendance-item.checked { background: rgba(56, 189, 248, 0.2); border: 1px solid var(--primary); }

        /* 인풋 폼 */
        .form-row { display: flex; gap: 6px; margin-bottom: 8px; }
        input, select { background: var(--bg-input); border: none; color: white; padding: 8px; border-radius: 6px; font-size: 0.85rem; }
        input[type="text"] { flex: 1; }
        button.action-btn { background: var(--primary); color: #0f172a; border: none; padding: 10px; border-radius: 8px; font-weight: bold; cursor: pointer; width: 100%; font-size: 0.95rem; }
        button.action-btn:active { opacity: 0.8; }
        button.sub-btn { background: transparent; border: 1px solid var(--bg-input); color: var(--text-muted); padding: 6px 10px; border-radius: 6px; cursor: pointer; font-size: 0.75rem; }

        /* 팀 배정 결과 */
        .match-teams { display: flex; flex-direction: column; gap: 12px; margin-top: 10px; }
        .team-box { background: rgba(255,255,255,0.03); border-radius: 8px; padding: 10px; border-left: 4px solid var(--primary); }
        .team-title { font-weight: bold; display: flex; justify-content: space-between; font-size: 0.9rem; margin-bottom: 6px; }
        .team-members { display: flex; flex-wrap: wrap; gap: 6px; }
        .member-badge { background: var(--bg-input); padding: 4px 8px; border-radius: 4px; font-size: 0.8rem; }
    </style>
</head>
<body>

<div class="app-container">
    <h1>⚽ FC사나이 팀 밸런서 ⚽</h1>

    <div class="tabs">
        <button class="tab-btn active" onclick="switchTab('match')">매치 편성</button>
        <button class="tab-btn" onclick="switchTab('roster')">21인 명단 관리</button>
    </div>

    <!-- 탭 1: 매치 편성 -->
    <div id="tab-match">
        <div class="card">
            <div class="card-header">
                <span>1. 참석 인원 선택 (<span id="attendee-count">0</span>명)</span>
                <div>
                    <button class="sub-btn" onclick="toggleAllAttendance(true)">전체선택</button>
                    <button class="sub-btn" onclick="toggleAllAttendance(false)">해제</button>
                </div>
            </div>
            <div class="player-grid" id="attendance-list"></div>
        </div>

        <div class="card" style="margin-top: 12px;">
            <div class="card-header">
                <span>2. 팀 편성 옵션</span>
            </div>
            <div style="display:flex; gap:8px; align-items:center; margin-bottom: 12px;">
                <label style="font-size: 0.85rem;">팀 개수:</label>
                <select id="team-count" style="flex:1;">
                    <option value="2">2개 팀 (12인 매치)</option>
                    <option value="3">3개 팀 (18인 매치 / 3파전)</option>
                </select>
            </div>
            <button class="action-btn" onclick="generateTeams()">밸런스 매칭 실행</button>
        </div>

        <div class="card" id="result-card" style="display:none; margin-top: 12px;">
            <div class="card-header">
                <span>최적 편성 결과</span>
                <button class="sub-btn" onclick="copyResult()">결과 복사</button>
            </div>
            <div id="match-teams" class="match-teams"></div>
            <button class="action-btn" style="margin-top:12px; background:var(--accent);" onclick="confirmMatch()">이 팀 조합 확정 (기록 저장)</button>
        </div>
    </div>

    <!-- 탭 2: 선수 명단 관리 -->
    <div id="tab-roster" style="display:none;">
        <div class="card">
            <div class="card-header">선수 등록 / 추가</div>
            <div class="form-row">
                <input type="text" id="new-name" placeholder="이름 (예: 홍길동)">
                <select id="new-pos">
                    <option value="FW">공격(FW)</option>
                    <option value="MF" selected>미드(MF)</option>
                    <option value="DF">수비(DF)</option>
                    <option value="GK">골키퍼(GK)</option>
                </select>
                <select id="new-tier">
                    <option value="5">5점 (에이스)</option>
                    <option value="4">4점 (상)</option>
                    <option value="3" selected>3점 (중)</option>
                    <option value="2">2점 (하)</option>
                    <option value="1">1점 (비기너)</option>
                </select>
                <button class="sub-btn" style="background:var(--primary); color:#0f172a;" onclick="addPlayer()">추가</button>
            </div>
            <button class="sub-btn" style="margin-top:4px;" onclick="loadDefault21()">21명 기본 더미데이터 넣기</button>
            <button class="sub-btn" style="margin-top:4px; color:var(--danger);" onclick="resetHistory()">매칭 히스토리 초기화</button>
        </div>

        <div class="card" style="margin-top: 12px;">
            <div class="card-header">등록된 선수 (<span id="total-player-count">0</span>/21)</div>
            <div class="player-grid" id="roster-list"></div>
        </div>
    </div>
</div>

<script>
    // LocalStorage Key
    const STORAGE_KEY_PLAYERS = 'fcsanai_players';
    const STORAGE_KEY_HISTORY = 'fcsanai_team_history';

    let players = JSON.parse(localStorage.getItem(STORAGE_KEY_PLAYERS)) || [];
    let historyMatrix = JSON.parse(localStorage.getItem(STORAGE_KEY_HISTORY)) || {};
    let currentGeneratedTeams = [];

    function savePlayers() {
        localStorage.setItem(STORAGE_KEY_PLAYERS, JSON.stringify(players));
        renderRoster();
        renderAttendance();
    }

    function saveHistory() {
        localStorage.setItem(STORAGE_KEY_HISTORY, JSON.stringify(historyMatrix));
    }

    function switchTab(tab) {
        document.getElementById('tab-match').style.display = tab === 'match' ? 'block' : 'none';
        document.getElementById('tab-roster').style.display = tab === 'roster' ? 'block' : 'none';
        document.querySelectorAll('.tab-btn').forEach((btn, idx) => {
            btn.classList.toggle('active', (tab === 'match' && idx === 0) || (tab === 'roster' && idx === 1));
        });
    }

    // 21명 기본 세팅
    function loadDefault21() {
        if (!confirm('기본 21명 로스터로 교체하시겠습니까?')) return;
        players = [];
        const positions = ['FW', 'MF', 'MF', 'DF', 'DF', 'GK'];
        for (let i = 1; i <= 21; i++) {
            players.push({
                id: 'p_' + i,
                name: '선수 ' + i,
                pos: positions[(i - 1) % positions.length],
                tier: ((i % 5) + 1),
                checked: i <= 12
            });
        }
        savePlayers();
    }

    function addPlayer() {
        const nameInput = document.getElementById('new-name');
        const name = nameInput.value.trim();
        if (!name) return alert('이름을 입력하세요.');
        players.push({
            id: 'p_' + Date.now(),
            name: name,
            pos: document.getElementById('new-pos').value,
            tier: parseInt(document.getElementById('new-tier').value),
            checked: true
        });
        nameInput.value = '';
        savePlayers();
    }

    function removePlayer(id) {
        players = players.filter(p => p.id !== id);
        savePlayers();
    }

    function renderRoster() {
        const list = document.getElementById('roster-list');
        document.getElementById('total-player-count').innerText = players.length;
        list.innerHTML = players.map(p => `
            <div class="player-item">
                <div style="font-weight:bold;">${p.name}</div>
                <div style="color:var(--text-muted); font-size:0.75rem;">${p.pos} | ${p.tier}점</div>
                <button class="sub-btn" style="color:var(--danger); align-self:flex-end;" onclick="removePlayer('${p.id}')">삭제</button>
            </div>
        `).join('');
    }

    function renderAttendance() {
        const list = document.getElementById('attendance-list');
        let checkedCount = 0;
        list.innerHTML = players.map(p => {
            if (p.checked) checkedCount++;
            return `
                <div class="attendance-item ${p.checked ? 'checked' : ''}" onclick="toggleAttendance('${p.id}')">
                    <input type="checkbox" ${p.checked ? 'checked' : ''} style="pointer-events:none;">
                    <span>${p.name} (${p.tier}점)</span>
                </div>
            `;
        }).join('');
        document.getElementById('attendee-count').innerText = checkedCount;
    }

    function toggleAttendance(id) {
        const p = players.find(x => x.id === id);
        if (p) p.checked = !p.checked;
        savePlayers();
    }

    function toggleAllAttendance(status) {
        players.forEach(p => p.checked = status);
        savePlayers();
    }

    function getTogetherCount(id1, id2) {
        const key = [id1, id2].sort().join('_');
        return historyMatrix[key] || 0;
    }

    function addTogetherCount(id1, id2) {
        const key = [id1, id2].sort().join('_');
        historyMatrix[key] = (historyMatrix[key] || 0) + 1;
    }

    // 밸런스 매칭 알고리즘 (2회 이상 중복 방지 + 실력 차이 최소화)
    function generateTeams() {
        const attendees = players.filter(p => p.checked);
        const k = parseInt(document.getElementById('team-count').value);
        if (attendees.length < k * 2) {
            return alert(`인원이 부족합니다. 최소 ${k * 2}명 이상 참석 체크해주세요.`);
        }

        let bestTeams = null;
        let bestCost = Infinity;

        // 몬테카를로 최적화 (2,000회 시뮬레이션으로 최적 조합 탐색)
        for (let iter = 0; iter < 2000; iter++) {
            const shuffled = [...attendees].sort(() => Math.random() - 0.5);
            const teams = Array.from({ length: k }, () => []);

            shuffled.forEach((p, idx) => {
                teams[idx % k].push(p);
            });

            // 점수 계산 (비용 함수)
            let cost = 0;
            const teamScores = teams.map(t => t.reduce((sum, p) => sum + p.tier, 0));
            const maxScore = Math.max(...teamScores);
            const minScore = Math.min(...teamScores);
            
            // 1. 실력 편차 페널티
            cost += (maxScore - minScore) * 10;

            // 2. 히스토리 페널티 (2회 이상 같은 편 절대 회피)
            for (let t of teams) {
                for (let i = 0; i < t.length; i++) {
                    for (let j = i + 1; j < t.length; j++) {
                        const count = getTogetherCount(t[i].id, t[j].id);
                        if (count >= 2) {
                            cost += 5000 * count; // 2회 이상 중복 시 초고강도 페널티
                        } else if (count === 1) {
                            cost += 15; // 1회 중복 약한 페널티
                        }
                    }
                }
            }

            if (cost < bestCost) {
                bestCost = cost;
                bestTeams = teams;
            }
        }

        currentGeneratedTeams = bestTeams;
        renderResult(bestTeams);
    }

    function renderResult(teams) {
        const container = document.getElementById('match-teams');
        container.innerHTML = teams.map((team, idx) => {
            const totalScore = team.reduce((sum, p) => sum + p.tier, 0);
            return `
                <div class="team-box">
                    <div class="team-title">
                        <span>팀 ${String.fromCharCode(65 + idx)} (${team.length}명)</span>
                        <span style="color:var(--accent);">총 능력치: ${totalScore}점</span>
                    </div>
                    <div class="team-members">
                        ${team.map(p => `<span class="member-badge">${p.name} [${p.pos}/${p.tier}]</span>`).join('')}
                    </div>
                </div>
            `;
        }).join('');
        document.getElementById('result-card').style.display = 'block';
        window.scrollTo({ top: document.body.scrollHeight, behavior: 'smooth' });
    }

    function confirmMatch() {
        if (!currentGeneratedTeams.length) return;
        if (!confirm('이 편성을 확정하고 같은 팀 횟수를 누적 저장하시겠습니까?')) return;

        currentGeneratedTeams.forEach(team => {
            for (let i = 0; i < team.length; i++) {
                for (let j = i + 1; j < team.length; j++) {
                    addTogetherCount(team[i].id, team[j].id);
                }
            }
        });

        saveHistory();
        alert('매치 기록이 반영되었습니다! 다음 매칭 시 이번 조합이 중복되지 않도록 계산됩니다.');
    }

    function copyResult() {
        if (!currentGeneratedTeams.length) return;
        let text = `⚽ FC사나이 목요 풋살 팀 편성 결과 ⚽\n\n`;
        currentGeneratedTeams.forEach((team, idx) => {
            const score = team.reduce((sum, p) => sum + p.tier, 0);
            text += `[ 팀 ${String.fromCharCode(65 + idx)} ] (점수합: ${score}점)\n`;
            text += team.map(p => `${p.name}(${p.pos})`).join(', ') + '\n\n';
        });

        navigator.clipboard.writeText(text).then(() => {
            alert('카톡방 공유용 텍스트가 클립보드에 복사되었습니다!');
        });
    }

    function resetHistory() {
        if (confirm('모든 이전 매칭 기록(누적 횟수)을 초기화하시겠습니까?')) {
            historyMatrix = {};
            saveHistory();
            alert('히스토리가 초기화되었습니다.');
        }
    }

    // 초기 로드
    if (players.length === 0) {
        loadDefault21();
    } else {
        renderRoster();
        renderAttendance();
    }
</script>
</body>
</html>
