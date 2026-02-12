# WPM-Test
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Ultimate WPM Platform</title>
<style>
body{background:linear-gradient(135deg,#0f0f1a,#1a1a3a);color:white;font-family:Segoe UI;text-align:center;margin:0;padding:0;}
h1{background:linear-gradient(90deg,cyan,magenta);-webkit-background-clip:text;color:transparent;}
input,textarea,select,button{margin:5px;padding:10px;border-radius:8px;border:none;font-size:15px;}
button{background:cyan;font-weight:bold;cursor:pointer;}button:hover{background:magenta;color:white;}
textarea{width:80%;height:100px;font-size:18px;}
#textDisplay{margin:20px;font-size:22px;background:#222244;padding:15px;border-radius:10px;}
.stats{font-size:20px;margin:15px;}
.level-card{background:#1a1a2e;padding:10px;margin:10px;border-radius:10px;cursor:pointer;}
.tabs{display:flex;justify-content:center;margin:20px;}
.tab{padding:12px 20px;cursor:pointer;background:#222244;margin:5px;border-radius:10px;}
.tab.active{background:cyan;color:black;}
.section{display:none;padding:20px;}
.section.active{display:block;}
</style>
</head>
<body>

<h1>🌍 Ultimate WPM Platform</h1>

<!-- AUTH -->
<div id="authSection">
  <div id="signupForm">
    <h3>Sign Up</h3>
    <input id="username" placeholder="Username">
    <input id="password" type="password" placeholder="Password">
    <br>
    <select id="signupLanguage">
      <option value="english">English</option>
      <option value="french">French</option>
      <option value="arabic">Arabic</option>
      <option value="german">German</option>
      <option value="spanish">Spanish</option>
      <option value="japanese">Japanese</option>
      <option value="korean">Korean</option>
    </select>
    <br>
    <button onclick="signup()">Sign Up</button>
    <p>Already have an account? <a href="#" onclick="showLogin()">Login</a></p>
  </div>

  <div id="loginForm" style="display:none;">
    <h3>Login</h3>
    <input id="loginUsername" placeholder="Username">
    <input id="loginPassword" type="password" placeholder="Password">
    <br>
    <button onclick="login()">Login</button>
    <p>Don't have an account? <a href="#" onclick="showSignup()">Sign Up</a></p>
  </div>

  <hr>
  <button onclick="playAsGuest()">Play as Guest</button>
</div>

<!-- MAIN WPM SECTION -->
<div id="mainSection" style="display:none;">
<span id="welcomeUser"></span>
<button onclick="logout()">Logout</button>

<div class="tabs">
<div class="tab active" onclick="switchTab('ai')">🤖 AI Mode</div>
<div class="tab" onclick="switchTab('custom')">✍️ Custom Mode</div>
<div class="tab" onclick="switchTab('community')">📂 Community Levels</div>
<div class="tab" onclick="switchTab('leaderboard')">🏆 Leaderboard</div>
</div>

<div id="ai" class="section active">
<select id="aiLanguage">
<option value="english">English</option>
<option value="french">French</option>
<option value="arabic">Arabic</option>
<option value="german">German</option>
<option value="spanish">Spanish</option>
<option value="japanese">Japanese</option>
<option value="korean">Korean</option>
</select>
<select id="aiTime"><option value="30">30s</option><option value="60" selected>60s</option><option value="120">120s</option></select>
<button onclick="startAI()">Start AI Test</button>
</div>

<div id="custom" class="section">
<textarea id="customText" placeholder="Write your own level..."></textarea>
<br>
<label><input type="checkbox" id="useTimer"> Use Timer</label>
<select id="customTime"><option value="30">30s</option><option value="60">60s</option><option value="120">120s</option></select>
<br>
<button onclick="startCustom()">Start Custom Test</button>
<button onclick="submitCommunity()">Save as Community Level</button>
</div>

<div id="community" class="section">
<select id="communityLanguage">
<option value="all">All Languages</option>
<option value="english">English</option>
<option value="french">French</option>
<option value="arabic">Arabic</option>
<option value="german">German</option>
<option value="spanish">Spanish</option>
<option value="japanese">Japanese</option>
<option value="korean">Korean</option>
</select>
<button onclick="loadCommunity()">Load Levels</button>
<div id="communityList"></div>
</div>

<div id="leaderboard" class="section">
<h2>XP Leaderboard</h2>
<ul id="leaderboardList"></ul>
</div>

<hr>
<div id="textDisplay"></div>
<textarea id="input" disabled oninput="updateStats()"></textarea>
<div class="stats">
⏳ Time: <span id="time">0</span> |
🚀 WPM: <span id="wpm">0</span> |
🎯 Accuracy: <span id="accuracy">100</span>% |
⭐ XP: <span id="xp">0</span> |
⬆ Level: <span id="level">0</span>
</div>

<script>
// --- LOCAL STORAGE USER MANAGEMENT ---
let currentUser=null;
let guestCounter=1;
let currentText="";
let startTime, timer, timeLeft, timerEnabled=true, userXP=0, userLevel=0;
let users=JSON.parse(localStorage.getItem("users")||"{}");

function showLogin(){document.getElementById("signupForm").style.display="none";document.getElementById("loginForm").style.display="block";}
function showSignup(){document.getElementById("signupForm").style.display="block";document.getElementById("loginForm").style.display="none";}

function signup(){
  let u=document.getElementById("username").value.trim();
  let p=document.getElementById("password").value;
  let l=document.getElementById("signupLanguage").value;
  if(!u||!p)return alert("Fill all fields");
  if(users[u]) return alert("Username exists");
  users[u]={password:p, language:l, xp:0, level:0};
  localStorage.setItem("users", JSON.stringify(users));
  currentUser=u;
  goToWPM();
}

function login(){
  let u=document.getElementById("loginUsername").value.trim();
  let p=document.getElementById("loginPassword").value;
  if(!users[u] || users[u].password!==p) return alert("Invalid login");
  currentUser=u;
  goToWPM();
}

function playAsGuest(){
  currentUser="logged_out_"+guestCounter; guestCounter++; goToWPM();
}

function logout(){
  currentUser=null;
  document.getElementById("mainSection").style.display="none";
  document.getElementById("authSection").style.display="block";
}

function goToWPM(){
  document.getElementById("authSection").style.display="none";
  document.getElementById("mainSection").style.display="block";
  document.getElementById("welcomeUser").innerText="Welcome "+currentUser;
  loadLeaderboard();
}

// --- TABS ---
function switchTab(id){document.querySelectorAll(".tab").forEach(t=>t.classList.remove("active"));document.querySelectorAll(".section").forEach(s=>s.classList.remove("active"));event.target.classList.add("active");document.getElementById(id).classList.add("active");}

// --- AI GENERATOR ---
function generateAI(){let w=["future","technology","world","competition","accuracy","speed","victory","challenge","innovation","intelligence"],s="";for(let i=0;i<15;i++){s+=w[Math.floor(Math.random()*w.length)]+" ";}return s;}
function startAI(){currentText=generateAI();textDisplay.innerText=currentText;startTest(document.getElementById("aiTime").value,true);}

// --- CUSTOM ---
function startCustom(){currentText=document.getElementById("customText").value;timerEnabled=document.getElementById("useTimer").checked;startTest(document.getElementById("customTime").value,timerEnabled);}
function submitCommunity(){let text=document.getElementById("customText").value;if(!text)return alert("Type text");let community=JSON.parse(localStorage.getItem("community")||"[]");community.push({creator:currentUser,text});localStorage.setItem("community",JSON.stringify(community));alert("Saved!");}
function loadCommunity(){let lang=document.getElementById("communityLanguage").value;let community=JSON.parse(localStorage.getItem("community")||"[]");communityList.innerHTML="";community.forEach((lvl,i)=>{communityList.innerHTML+=`<div class="level-card" onclick="playCommunity(${i})">Level ${i+1} by ${lvl.creator}</div>`;});}
function playCommunity(i){currentText=JSON.parse(localStorage.getItem("community"))[i].text;textDisplay.innerText=currentText;startTest(60,true);}

// --- TEST LOGIC ---
function startTest(duration,useTimer){
  input.disabled=false; input.value=""; input.focus(); startTime=new Date(); timerEnabled=useTimer; clearInterval(timer);
  if(useTimer){timeLeft=parseInt(duration); time.innerText=timeLeft; timer=setInterval(()=>{timeLeft--; time.innerText=timeLeft;if(timeLeft<=0){clearInterval(timer);endTest();}},1000);}else{time.innerText="∞";}
}

function updateStats(){
  let typed=input.value, mins=(new Date()-startTime)/60000, words=typed.trim().split(/\s+/).length;
  let wpm=Math.round(words/mins)||0; document.getElementById("wpm").innerText=wpm;
  let correct=0; for(let i=0;i<typed.length;i++){if(typed[i]===currentText[i]) correct++;}
  let acc=Math.round((correct/typed.length)*100)||100; document.getElementById("accuracy").innerText=acc;
  userXP=words*10+acc; userLevel=Math.floor(userXP/1000); document.getElementById("xp").innerText=userXP; document.getElementById("level").innerText=userLevel;
}

function endTest(){input.disabled=true; saveUserData();}

function saveUserData(){if(users[currentUser]){users[currentUser].xp=userXP; users[currentUser].level=userLevel; localStorage.setItem("users", JSON.stringify(users));}}

// --- LEADERBOARD ---
function loadLeaderboard(){leaderboardList.innerHTML="";Object.keys(users).sort((a,b)=>users[b].xp-users[a].xp).forEach(u=>leaderboardList.innerHTML+=`<li>${u} - XP:${users[u].xp} Level:${users[u].level}</li>`);}
</script>
</body>
</html>
