<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<title>Happy Birthday Masha - Interactive Album</title>
<style>
html, body {
  margin:0; padding:0;
  overflow:hidden;
  background:black;
  font-family:Arial;
  color:white;
  height:100vh;
}

/* MATRIX */
canvas{
  position:fixed;
  top:0;
  left:0;
  width:100%;
  height:100%;
  z-index:0;
}

/* CENTER TEXT */
#centerText{
  position:absolute;
  top:50%;
  left:50%;
  transform:translate(-50%,-50%);
  text-align:center;
  z-index:2;
}

.big{
  font-size:120px;
  color:#ff4da6;
  text-shadow:0 0 25px #ff4da6;
}

.word{
  font-size:70px;
  display:none;
  color:#ff66cc;
  text-shadow:0 0 20px #ff66cc;
}

#newGif{
  display:none;
  margin-top:20px;
  animation:pop 0.8s ease;
}

/* ALBUM */
#album {
  width: 400px;
  height: 250px;
  perspective: 1500px;
  cursor: pointer;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  display: none;
}

/* COVER (слева направо) */
#cover {
  width: 100%;
  height: 100%;
  background:#ff4da6;
  border-radius:12px;
  display:flex;
  align-items:center;
  justify-content:center;
  font-size:24px;
  color:white;
  transform-origin: left center; /* Поворот слева направо */
  transition: transform 1s ease;
  position:absolute;
  top:0;
  left:0;
  z-index:3;
  box-shadow: 0 10px 40px rgba(0,0,0,0.5);
}

/* LEFT PAGE */
#leftHalf {
  width: 50%;
  height: 100%;
  background:white;
  border-radius:0 0 0 12px;
  display:flex;
  justify-content:center;
  align-items:center;
  overflow: hidden;
  transform: rotateY(0deg);
  transform-origin: left center;
  transition: transform 1s ease;
  z-index:2;
  box-shadow: inset -5px 0 15px rgba(0,0,0,0.2);
}

/* RIGHT PAGE */
#rightHalf {
  width: 50%;
  height: 100%;
  background:white;
  border-radius:0 0 12px 0;
  display:flex;
  justify-content:center;
  align-items:center;
  overflow: hidden;
  transform: rotateY(0deg);
  transform-origin: right center;
  transition: transform 1s ease;
  z-index:1;
  box-shadow: inset 5px 0 15px rgba(0,0,0,0.2);
}

/* Скрываем фото до открытия */
#leftHalf img,
#rightHalf img {
  width:100%;
  height:100%;
  object-fit:cover;
  opacity:0;
  display:block;
  transition: opacity 0.5s ease;
}

/* OPEN STATE */
#album.open #cover {
  transform: rotateY(-180deg);
}

#album.open #leftHalf img {
  opacity:1;
  transition-delay:1s; /* Ждем пока обложка повернется */
}

#album.open #rightHalf img {
  opacity:1;
  transition-delay:1s; /* Ждем пока обложка повернется */
}
</style>
</head>
<body>

<canvas id="matrix"></canvas>

<div id="centerText">
  <div id="count" class="big" style="display:none;">3</div>
  <div id="w1" class="word">Happy</div>
  <div id="w2" class="word">Birthday</div>
  <div id="w3" class="word">to</div>
  <div id="w4" class="word">you</div>
  <div id="heart" class="word">❤️</div>

  <img id="newGif"
       src="https://media0.giphy.com/media/v1.Y2lkPTc5MGI3NjExaGtyYmsxMTM0YjB4NDQ5Y3dwd3NsN3JtcjQ2eHN0aHJ4bGthcDN3MyZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9cw/COxdWdYHHtPgWvDb5k/giphy.gif"
       width="250">
</div>

<div id="album">
  <div id="cover">Click to open 🤍</div>
  <div id="leftHalf">
    <img src="https://picsum.photos/200/250?1" alt="Left Photo">
  </div>
  <div id="rightHalf">
    <img src="https://picsum.photos/200/250?2" alt="Right Photo">
  </div>
</div>

<script>
/* MATRIX */
const canvas=document.getElementById("matrix");
const ctx=canvas.getContext("2d");

function resize(){
  canvas.width=window.innerWidth;
  canvas.height=window.innerHeight;
}
resize();
window.addEventListener("resize",resize);

const letters="0123456789";
const fontSize=16;
const columns=Math.floor(canvas.width/fontSize);
const drops=[]; for(let i=0;i<columns;i++) drops[i]=1;

function draw(){
  ctx.fillStyle="rgba(0,0,0,0.05)";
  ctx.fillRect(0,0,canvas.width,canvas.height);
  ctx.fillStyle="#ff4da6";
  ctx.font=fontSize+"px monospace";
  for(let i=0;i<drops.length;i++){
    let text=letters[Math.floor(Math.random()*letters.length)];
    ctx.fillText(text,i*fontSize,drops[i]*fontSize);
    if(drops[i]*fontSize>canvas.height && Math.random()>0.975) drops[i]=0;
    drops[i]++;
  }
}
setInterval(draw,33);

/* COUNTDOWN */
let count = 3;
const c = document.getElementById("count");
setTimeout(() => {
  c.style.display = "block";
  const timer = setInterval(() => {
    count--;
    if (count > 0) {
      c.innerText = count;
    } else {
      clearInterval(timer);
      c.style.display = "none";
      showWords();
    }
  }, 1000);
}, 3000);

/* WORDS AND GIF */
const w1=document.getElementById("w1");
const w2=document.getElementById("w2");
const w3=document.getElementById("w3");
const w4=document.getElementById("w4");
const heart=document.getElementById("heart");
const newGif=document.getElementById("newGif");
const album=document.getElementById("album");

function showWords(){
  setTimeout(()=>{ w1.style.display="block"; },500);
  setTimeout(()=>{ w1.style.display="none"; w2.style.display="block"; },2000);
  setTimeout(()=>{ w2.style.display="none"; w3.style.display="block"; },3500);
  setTimeout(()=>{ w3.style.display="none"; w4.style.display="block"; },5000);
  setTimeout(()=>{ heart.style.display="block"; },6500);
  setTimeout(()=>{
    w4.style.display="none";
    heart.style.display="none";
    newGif.style.display="block";
  },8500);
  setTimeout(() => {
    newGif.style.display = "none";
    album.style.display = "flex";
  }, 10500);
}

/* ALBUM CLICK */
const cover = document.getElementById('cover');
cover.onclick = () => {
  album.classList.add('open');
};
</script>

</body>
</html>
