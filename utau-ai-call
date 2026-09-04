export default {
  async fetch(request, env) {
    const url = new URL(request.url);

    if (request.method === "POST" && url.pathname === "/api/chat") {
      try {
        const { message, personality, custom } =
          await request.json();

        let prompt = "";

        if (personality === "tanaka") {
          prompt = `
あなたは「田中」という親しみやすいおじいちゃんAIです。
一人称は「ワシ」。
「〜じゃ」「〜じゃぞ」を時々使います。
やさしく、友達のように自然に、短く話してください。
`;
        } else if (personality === "jagajaga") {
          prompt = `
あなたは「じゃがじゃが」という明るく楽しいAIです。
親しみやすく、友達のように自然に短く話してください。
`;
        } else {
          prompt = `
あなたの性格：
${custom || "親しみやすいAI"}

自然で短い会話をしてください。
`;
        }

        const model =
          env.GEMINI_MODEL || "gemini-3.6-flash";

        const r = await fetch(
          `https://generativelanguage.googleapis.com/v1beta/models/${model}:generateContent?key=${env.GEMINI_API_KEY}`,
          {
            method: "POST",
            headers: {
              "Content-Type": "application/json"
            },
            body: JSON.stringify({
              systemInstruction: {
                parts: [{ text: prompt }]
              },
              contents: [{
                role: "user",
                parts: [{ text: message }]
              }],
              generationConfig: {
                temperature: 0.8,
                maxOutputTokens: 200
              }
            })
          }
        );

        const data = await r.json();

        if (!r.ok)
          return Response.json({
            error: data?.error?.message || "Gemini error"
          }, { status: r.status });

        return Response.json({
          reply:
            data?.candidates?.[0]?.content?.parts?.[0]?.text ||
            "うまく聞き取れなかったぞ！"
        });

      } catch (e) {
        return Response.json({
          error: e.message
        }, { status: 500 });
      }
    }

    return new Response(`<!DOCTYPE html>
<html lang="ja">
<meta name="viewport"
content="width=device-width,initial-scale=1">
<title>UTAU AI Call</title>

<style>
body{
background:#202124;color:white;
font-family:sans-serif;
text-align:center;padding:20px
}
button{
font-size:18px;padding:15px;
margin:8px;border:0;border-radius:12px
}
#chat{
text-align:left;margin:20px 0
}
</style>

<h1>📞 UTAU AI Call</h1>

<h2 id="name">AIキャラ</h2>

<button onclick="person('tanaka')">👴 田中</button>
<button onclick="person('jagajaga')">🥔 じゃがじゃが</button>
<button onclick="person('custom')">✏️ 自分で決める</button>

<br>

<textarea id="custom"
placeholder="自分の性格"
style="display:none;width:90%;height:60px">
</textarea>

<div id="chat"></div>

<button onclick="startCall()">📞 通話開始</button>
<button onclick="endCall()">🔴 終了</button>

<script>
let type="";
let active=false;
let rec;

const SR=window.SpeechRecognition||
window.webkitSpeechRecognition;

function person(x){
type=x;
name.textContent=
x==="tanaka"?"田中":
x==="jagajaga"?"じゃがじゃが":"オリジナルAI";
custom.style.display=
x==="custom"?"block":"none";
}

async function startCall(){
if(!type)return alert("性格を選んでください！");
if(!SR)return alert("Chromeで試してください");

active=true;
rec=new SR();
rec.lang="ja-JP";
rec.continuous=false;
rec.interimResults=false;

rec.onresult=e=>{
let text=e.results[0][0].transcript;
add("あなた： "+text);
ai(text);
};

rec.onend=()=>{
if(active)setTimeout(listen,500);
};

listen();
}

function listen(){
if(active)try{rec.start()}catch(e){}
}

async function ai(text){
let r=await fetch("/api/chat",{
method:"POST",
headers:{"Content-Type":"application/json"},
body:JSON.stringify({
message:text,
personality:type,
custom:custom.value
})
});

let d=await r.json();
if(d.error)return add("❌ "+d.error);

add(name.textContent+"： "+d.reply);

speechSynthesis.cancel();
let u=new SpeechSynthesisUtterance(d.reply);
u.lang="ja-JP";
u.onend=()=>{if(active)listen()};
speechSynthesis.speak(u);
}

function add(t){
chat.innerHTML+="<p>"+esc(t)+"</p>";
}

function esc(t){
return t.replace(/&/g,"&amp;")
.replace(/</g,"&lt;")
.replace(/>/g,"&gt;");
}

function endCall(){
active=false;
if(rec)try{rec.stop()}catch(e){}
speechSynthesis.cancel();
}

</script>
</html>`,{
headers:{"Content-Type":"text/html;charset=UTF-8"}
});
  }
};
