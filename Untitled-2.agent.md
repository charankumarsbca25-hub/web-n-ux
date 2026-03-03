<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>SkillSwap Pro</title>

<style>
*{
  margin:0;
  padding:0;
  box-sizing:border-box;
  font-family:Arial, sans-serif;
}

body{
  background:#0f172a;
  color:white;
}

/* Header */
header{
  background:royalblue;
  padding:20px 50px;
  display:flex;
  justify-content:space-between;
  align-items:center;
  box-shadow:0 4px 15px rgba(0,0,0,0.4);
}

header h1 span{
  color:#ffd700;
}

button{
  padding:8px 15px;
  border:none;
  border-radius:6px;
  cursor:pointer;
  font-weight:bold;
  background:#1e3a8a;
  color:white;
}

button:hover{
  background:#2563eb;
}

/* Hero Section */
.hero{
  text-align:center;
  padding:70px 20px;
  background:linear-gradient(135deg,#1e3a8a,#0f172a);
}

.hero h2{
  font-size:42px;
  margin-bottom:10px;
}

.hero p{
  color:#cbd5e1;
}

/* Skills Section */
.skills-section{
  padding:40px;
  text-align:center;
}

input{
  padding:10px;
  width:250px;
  margin:10px;
  border-radius:5px;
  border:none;
}

.skill-card{
  background:#1e293b;
  margin:10px auto;
  padding:15px;
  width:350px;
  border-radius:8px;
  box-shadow:0 5px 15px rgba(0,0,0,0.5);
}

small{
  color:#94a3b8;
}

/* Modal */
.modal{
  display:none;
  position:fixed;
  width:100%;
  height:100%;
  background:rgba(0,0,0,0.7);
}

.modal-content{
  background:#1e293b;
  width:300px;
  margin:10% auto;
  padding:20px;
  border-radius:10px;
  text-align:center;
}

.modal-content input{
  width:90%;
  margin:8px 0;
}
</style>
</head>

<body>

<header>
  <h1>SkillSwap <span>Pro</span></h1>
  <div id="authArea">
    <button onclick="openModal(true)">Login</button>
    <button onclick="openModal(false)">Register</button>
  </div>
  <button id="logoutBtn" onclick="logout()" style="display:none;">Logout</button>
</header>

<section class="hero">
  <h2>Exchange Skills. Grow Together.</h2>
  <p>Learn from others. Teach what you know. Completely Free.</p>
</section>

<section class="skills-section">

  <h3>Add New Skill</h3>
  <input type="text" id="skillInput" placeholder="Enter your skill">
  <button onclick="addSkill()">Add Skill</button>

  <h3 style="margin-top:40px;">Available Skills</h3>
  <div id="skillsContainer"></div>

</section>

<!-- Modal -->
<div class="modal" id="authModal">
  <div class="modal-content">
    <h3 id="modalTitle"></h3>
    <input type="email" id="email" placeholder="Email">
    <input type="password" id="password" placeholder="Password">
    <button onclick="handleAuth()" id="authBtn"></button>
    <br><br>
    <button onclick="closeModal()">Close</button>
  </div>
</div>

<script type="module">

import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
import { 
  getAuth, createUserWithEmailAndPassword,
  signInWithEmailAndPassword, signOut,
  onAuthStateChanged 
} from "https://www.gstatic.com/firebasejs/10.7.1/firebase-auth.js";
import { 
  getFirestore, collection, addDoc,
  query, orderBy, onSnapshot 
} from "https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js";

// 🔥 Replace with your Firebase config
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_AUTH_DOMAIN",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_BUCKET",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_APP_ID"
};

const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);

let isLogin = true;

// Modal controls
window.openModal = (loginMode)=>{
  isLogin = loginMode;
  document.getElementById("modalTitle").innerText = loginMode ? "Login" : "Register";
  document.getElementById("authBtn").innerText = loginMode ? "Login" : "Register";
  document.getElementById("authModal").style.display="block";
}

window.closeModal = ()=>{
  document.getElementById("authModal").style.display="none";
}

// Register/Login
window.handleAuth = async ()=>{
  const email=document.getElementById("email").value;
  const password=document.getElementById("password").value;

  try{
    if(isLogin){
      await signInWithEmailAndPassword(auth,email,password);
      alert("Login Successful");
    }else{
      await createUserWithEmailAndPassword(auth,email,password);
      alert("Registration Successful");
    }
    closeModal();
  }catch(error){
    alert(error.message);
  }
}

// Logout
window.logout = async ()=>{
  await signOut(auth);
}

// Auth State
onAuthStateChanged(auth,user=>{
  if(user){
    document.getElementById("logoutBtn").style.display="inline-block";
    document.getElementById("authArea").style.display="none";
  }else{
    document.getElementById("logoutBtn").style.display="none";
    document.getElementById("authArea").style.display="block";
  }
});

// Add Skill
window.addSkill = async ()=>{
  const skill=document.getElementById("skillInput").value;
  const user=auth.currentUser;

  if(!skill.trim()) return alert("Enter skill");
  if(!user) return alert("Login first");

  await addDoc(collection(db,"skills"),{
    skill:skill,
    userEmail:user.email,
    createdAt:new Date()
  });

  document.getElementById("skillInput").value="";
}

// Real-time Skills Display
const q = query(collection(db,"skills"), orderBy("createdAt","desc"));

onSnapshot(q,snapshot=>{
  const container=document.getElementById("skillsContainer");
  container.innerHTML="";

  snapshot.forEach(doc=>{
    const data=doc.data();
    const div=document.createElement("div");
    div.className="skill-card";
    div.innerHTML="<strong>"+data.skill+"</strong><br><small>Shared by: "+data.userEmail+"</small>";
    container.appendChild(div);
  });
});

</script>

</body>
</html>