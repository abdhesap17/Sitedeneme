<!DOCTYPE html>
<html lang="tr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Sohbet</title>

<style>
*{box-sizing:border-box}
body{margin:0;font-family:system-ui;background:#020617;color:#e5e7eb}
#app{max-width:480px;margin:auto;height:100vh;display:flex;flex-direction:column;position:relative}

/* TOP */
.top{height:48px;display:flex;align-items:center;padding:0 14px;border-bottom:1px solid #1e293b}
.dot{font-size:22px;cursor:pointer}

/* SIDE */
.side{position:fixed;top:0;left:-260px;width:240px;height:100%;
background:#020617;border-right:1px solid #1e293b;
transition:.25s;z-index:50;padding:12px}
.side.open{left:0}
.room{padding:10px;border-radius:10px;margin-bottom:6px;
background:#0f172a;display:flex;justify-content:space-between;align-items:center}
.room.active{background:#2563eb}
.room span{cursor:pointer}
.roomBtns{display:flex;gap:6px}

/* PINNED */
.pinnedBar{background:#020617;border-bottom:1px solid #1e293b;
padding:8px 12px;font-size:13px;color:#93c5fd;display:none;cursor:pointer}
.pinned{background:#0f172a;border:1px solid #2563eb}
.pinBadge{position:absolute;top:-6px;left:12px;font-size:11px;color:#60a5fa}

/* MESSAGES */
.messages{flex:1;padding:14px;overflow-y:auto}
.msg{background:#1e293b;border-radius:14px;padding:12px 44px 24px 12px;
max-width:80%;margin-left:auto;margin-bottom:18px;position:relative}
.msgHeader{display:flex;align-items:center;gap:6px;font-size:13px;margin-bottom:6px}
.avatar{width:26px;height:26px;border-radius:50%;background:#2563eb;
display:flex;align-items:center;justify-content:center;font-weight:bold}
.text{font-size:14px}
.time{position:absolute;bottom:4px;right:8px;font-size:11px;color:#94a3b8}
.read{position:absolute;bottom:4px;left:12px;font-size:11px;color:#22c55e}

/* MESSAGE MENU */
.menu{position:absolute;top:6px;right:6px;width:28px;height:28px;
display:flex;align-items:center;justify-content:center;cursor:pointer}
.msgMenu{position:absolute;top:34px;right:6px;background:#020617;
border:1px solid #1e293b;border-radius:10px;overflow:hidden;display:none;z-index:10}
.msgMenu div{padding:10px 14px;font-size:14px;cursor:pointer}
.msgMenu div:hover{background:#1e293b}

/* QUOTE */
.quote{background:#020617;border-left:3px solid #2563eb;
padding:6px 8px;font-size:13px;margin-bottom:6px;color:#94a3b8;
cursor:pointer}
.quoteBar{background:#020617;border-left:4px solid #2563eb;
padding:8px 10px;font-size:13px;color:#94a3b8;
display:none;justify-content:space-between;align-items:center}

/* SYSTEM */
.system{background:#0f172a;border:1px dashed #1e293b;
color:#94a3b8;text-align:center;font-size:13px;
padding:8px;border-radius:12px;margin:14px auto;max-width:90%}

/* INPUT */
.input{border-top:1px solid #1e293b}
.inputBar{display:flex}
.input input{flex:1;padding:14px;background:none;border:none;color:#fff}
.input button{padding:14px 18px;background:#2563eb;border:none;color:#fff}

/* MODAL */
.modal{position:fixed;inset:0;background:#0008;display:none;
align-items:center;justify-content:center;z-index:100}
.modalBox{background:#020617;border:1px solid #1e293b;
border-radius:14px;padding:16px;width:80%;max-width:300px}
.modalBox h3{margin:0 0 8px 0;font-size:16px}
.modalBox input{width:100%;padding:10px;background:#0f172a;
border:none;color:#fff;border-radius:8px;margin-bottom:10px}
.modalBox button{width:100%;padding:10px;border:none;
border-radius:8px;background:#2563eb;color:#fff;margin-top:6px}
.danger{background:#dc2626}
</style>
</head>

<body>

<!-- USER LOGIN -->
<div class="modal" id="userModal" style="display:flex">
  <div class="modalBox">
    <h3>Kullanıcı Adı</h3>
    <input id="userInput" placeholder="Adını gir">
    <button onclick="setUser()">Devam</button>
  </div>
</div>

<div id="app">
  <div class="top">
    <div class="dot" onclick="toggleMenu()">⋮</div>
    <div id="roomTitle" style="margin-left:10px"></div>
  </div>

  <div id="pinnedBar" class="pinnedBar"></div>

  <div class="messages" id="messages"></div>

  <div class="quoteBar" id="quoteBar">
    <span id="quoteText"></span>
    <span style="cursor:pointer" onclick="closeQuote()">✕</span>
  </div>

  <div class="input">
    <div class="inputBar">
      <input id="msgInput" placeholder="Mesaj yaz">
      <button onclick="send()">Gönder</button>
    </div>
  </div>
</div>

<!-- SIDE -->
<div class="side" id="side">
  <button onclick="openEdit('roomAdd')" style="width:100%">+ Oda</button>
  <div id="rooms"></div>
</div>

<!-- EDIT MODAL -->
<div class="modal" id="editModal">
  <div class="modalBox">
    <h3 id="editTitle"></h3>
    <input id="editInput">
    <button onclick="saveEdit()">Kaydet</button>
    <button onclick="closeEdit()">İptal</button>
  </div>
</div>

<!-- DELETE MODAL -->
<div class="modal" id="deleteModal">
  <div class="modalBox">
    <h3>Sil</h3>
    <p style="font-size:13px;color:#94a3b8">Bu işlem geri alınamaz</p>
    <button class="danger" onclick="confirmDelete()">Sil</button>
    <button onclick="closeDelete()">İptal</button>
  </div>
</div>

<script>
const $=q=>document.querySelector(q)
const messages=$("#messages")
const roomsDiv=$("#rooms")
const input=$("#msgInput")
const roomTitle=$("#roomTitle")
const quoteBar=$("#quoteBar")
const quoteText=$("#quoteText")
const pinnedBar=$("#pinnedBar")

let user=""
let rooms=[{id:1,name:"Genel",msgs:[],pinned:null}]
let currentRoom=1
let editMode=null,editTarget=null
let deleteTarget=null
let quote=null

const time=()=>new Date().toLocaleTimeString("tr-TR",{hour:"2-digit",minute:"2-digit"})

function setUser(){
const v=$("#userInput").value.trim()
if(!v)return
user=v
$("#userModal").style.display="none"
systemMsg(`${user} sohbete katıldı`)
renderRooms()
}

function toggleMenu(){ $("#side").classList.toggle("open") }

function renderRooms(){
roomsDiv.innerHTML=""
rooms.forEach(r=>{
const d=document.createElement("div")
d.className="room"+(r.id===currentRoom?" active":"")
d.innerHTML=`
<span onclick="selectRoom(${r.id})">${r.name}</span>
<div class="roomBtns">
<span onclick="openEdit('roomEdit',${r.id})">✎</span>
<span onclick="openDelete(${r.id},'room')">🗑</span>
</div>`
roomsDiv.appendChild(d)
})
roomTitle.textContent=rooms.find(r=>r.id===currentRoom).name
renderPinned()
}

function selectRoom(id){
currentRoom=id
messages.innerHTML=""
renderRooms()
rooms.find(r=>r.id===id).msgs.forEach(drawMsg)
toggleMenu()
}

function systemMsg(t){
const m={sys:1,text:t}
rooms.find(r=>r.id===currentRoom).msgs.push(m)
drawMsg(m)
}

function renderPinned(){
const p=rooms.find(r=>r.id===currentRoom).pinned
if(!p){pinnedBar.style.display="none";return}
pinnedBar.textContent="📌 "+p.text
pinnedBar.style.display="block"
pinnedBar.onclick=()=>{
const t=document.querySelector(`.msg[data-id="${p.id}"]`)
if(t)t.scrollIntoView({behavior:"smooth"})
}
}

function drawMsg(m){
if(m.sys){
const s=document.createElement("div")
s.className="system"
s.textContent=m.text
messages.appendChild(s)
return
}
const d=document.createElement("div")
d.className="msg"+(rooms.find(r=>r.id===currentRoom).pinned?.id===m.id?" pinned":"")
d.dataset.id=m.id
let quoteHTML=""
if(m.quote){
quoteHTML=`<div class="quote" data-ref="${m.quote.id}">${m.quote.text}</div>`
}
d.innerHTML=`
${rooms.find(r=>r.id===currentRoom).pinned?.id===m.id?'<div class="pinBadge">📌 Sabitlendi</div>':''}
<div class="menu" onclick="toggleMsgMenu(this)">⋮</div>
<div class="msgMenu">
<div onclick="setQuote(${m.id})">Alıntıla</div>
<div onclick="pinMsg(${m.id})">Sabitle</div>
<div onclick="editMsg(${m.id})">Düzenle</div>
<div onclick="openDelete(${m.id},'msg')">Sil</div>
</div>
${quoteHTML}
<div class="msgHeader"><div class="avatar">${user[0]}</div>${user}</div>
<div class="text">${m.text}</div>
<div class="read">✓✓</div>
<div class="time">${m.time}</div>`
messages.appendChild(d)
messages.scrollTop=messages.scrollHeight
}

function pinMsg(id){
const r=rooms.find(r=>r.id===currentRoom)
r.pinned=r.msgs.find(m=>m.id===id)
selectRoom(currentRoom)
}

function toggleMsgMenu(e){
document.querySelectorAll(".msgMenu").forEach(x=>x.style.display="none")
e.nextElementSibling.style.display="block"
}

function send(){
if(!input.value||!user)return
const m={id:Date.now(),text:input.value,time:time(),quote:quote}
rooms.find(r=>r.id===currentRoom).msgs.push(m)
drawMsg(m)
input.value=""
quote=null
quoteBar.style.display="none"
}

function setQuote(id){
const m=rooms.find(r=>r.id===currentRoom).msgs.find(x=>x.id===id)
quote={id:m.id,text:m.text}
quoteText.textContent=m.text
quoteBar.style.display="flex"
}

function closeQuote(){ quote=null; quoteBar.style.display="none" }

document.body.onclick=e=>{
if(e.target.classList.contains("quote")){
const ref=e.target.dataset.ref
const t=document.querySelector(`.msg[data-id="${ref}"]`)
if(t)t.scrollIntoView({behavior:"smooth"})
}
}

function openEdit(mode,id){
editMode=mode; editTarget=id
$("#editModal").style.display="flex"
$("#editTitle").textContent=
mode==="roomAdd"?"Oda Ekle":
mode==="roomEdit"?"Oda Adı Düzenle":"Mesaj Düzenle"
$("#editInput").value=
mode==="roomEdit"?rooms.find(r=>r.id===id).name:
mode==="msgEdit"?rooms.find(r=>r.id===currentRoom).msgs.find(m=>m.id===id).text:""
}

function saveEdit(){
const v=$("#editInput").value
if(!v)return
if(editMode==="roomAdd"){
const id=Date.now()
rooms.push({id,name:v,msgs:[],pinned:null})
renderRooms()
}
if(editMode==="roomEdit"){
rooms.find(r=>r.id===editTarget).name=v
systemMsg("Grup adı değiştirildi")
renderRooms()
}
if(editMode==="msgEdit"){
rooms.find(r=>r.id===currentRoom).msgs.find(m=>m.id===editTarget).text=v
selectRoom(currentRoom)
}
closeEdit()
}

function closeEdit(){ $("#editModal").style.display="none" }

function editMsg(id){ openEdit("msgEdit",id) }

function openDelete(id,type){
deleteTarget={id,type}
$("#deleteModal").style.display="flex"
}

function confirmDelete(){
if(deleteTarget.type==="room"){
rooms=rooms.filter(r=>r.id!==deleteTarget.id)
currentRoom=rooms[0].id
renderRooms()
selectRoom(currentRoom)
}
if(deleteTarget.type==="msg"){
rooms.find(r=>r.id===currentRoom).msgs=
rooms.find(r=>r.id===currentRoom).msgs.filter(m=>m.id!==deleteTarget.id)
rooms.find(r=>r.id===currentRoom).pinned=null
selectRoom(currentRoom)
}
closeDelete()
}

function closeDelete(){ $("#deleteModal").style.display="none" }
</script>
</body>
</html>
