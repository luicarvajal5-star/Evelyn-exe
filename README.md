# Evelyn-exe
Un pequeño universo para mi amor..

<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Evelyn.exe ♡</title>
<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
<style>
*{box-sizing:border-box}
body{margin:0;background:#171522;color:#f8eaf2;font-family:monospace;text-align:center;padding:25px 15px}
main{max-width:650px;margin:auto}
h1{color:#f4a9ce;font-size:30px}
h2{color:#e9a5ca}
p{line-height:1.7;color:#e5d5e1}
button,input{font-family:monospace;border-radius:9px;padding:12px;margin:6px;border:1px solid #b66b9c}
button{background:#a64d83;color:white;cursor:pointer}
input{background:#242033;color:white;max-width:90%}
section{background:#211d30;border:1px solid #543653;border-radius:15px;padding:20px;margin:22px auto}
.hidden{display:none}
.heart{font-size:75px;color:#ed83b8;animation:latir 1.2s infinite}
@keyframes latir{50%{transform:scale(1.18)}}
#terminal{background:#100f19;text-align:left;padding:15px;border-radius:10px;color:#d9a8cf}
#respuesta{color:#f4b9da;min-height:30px}
.grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(145px,1fr));gap:12px;margin-top:15px}
.foto{background:#171522;padding:10px;border-radius:12px;border:1px solid #694466;overflow:hidden}
.foto img{width:100%;height:160px;object-fit:cover;border-radius:8px}
.foto p{font-size:12px;overflow-wrap:anywhere}
small{color:#bca9bd}
</style>
</head>
<body>
<main>
<section id="inicio">
<div class="heart">♥</div>
<h1>Evelyn.exe</h1>
<p>Un pequeño universo creado para ti.</p>
<button onclick="iniciar()">START ♡</button>
</section>

<section id="terminal" class="hidden">
<h2>LOVE_TERMINAL.exe</h2>
<p>Escribe un comando:</p>
<input id="comando" placeholder="te_quiero">
<button onclick="ejecutar()">EJECUTAR</button>
<p id="respuesta"></p>
</section>

<section id="contador" class="hidden">
<h2>TIEMPO JUNTAS ♡</h2>
<p id="tiempo"></p>
<small>Desde el 22 de septiembre de 2026</small>
</section>

<section id="memory" class="hidden">
<h2>MEMORY.exe</h2>
<p>Pequeños momentos que merecen quedarse.</p>
<p id="estado">Cargando recuerdos...</p>
<div id="galeria" class="grid"></div>

<div id="acceso">
<h3>ADMINISTRAR RECUERDOS</h3>
<p>Solo la persona autorizada puede agregar fotografías.</p>
<input id="correo" type="email" placeholder="Tu correo">
<input id="clave" type="password" placeholder="Tu contraseña">
<br>
<button onclick="entrar()">INICIAR SESIÓN</button>
<p id="mensaje"></p>
</div>

<div id="editor" class="hidden">
<h3>ADMINISTRADORA ACTIVADA ♡</h3>
<input id="titulo" placeholder="Título del recuerdo">
<input id="descripcion" placeholder="Un pequeño mensaje">
<input id="archivo" type="file" accept="image/*">
<br>
<button onclick="subirFoto()">GUARDAR RECUERDO</button>
<button onclick="salir()">CERRAR SESIÓN</button>
<p id="resultado"></p>
</div>
</section>

<section id="final" class="hidden">
<h2>Para ti ♡</h2>
<p>Qué bonito coincidir contigo. Gracias por hacer tan especiales mis días. Espero que sigamos creando recuerdos juntas.</p>
<div class="heart">♥</div>
</section>
</main>

<script>
const SUPABASE_URL = "https://aixdfrfezwkdkostcjcl.supabase.co/rest/v1/";
const SUPABASE_KEY = "sb_publishable_s7IDrF4dSatxFUD3miKvqQ_LUyJj9Oa";
const BUCKET = "recuerdos";

const db = supabase.createClient(SUPABASE_URL,SUPABASE_KEY);
let administradora = false;

function iniciar(){
["terminal","contador","memory","final"].forEach(id=>{
document.getElementById(id).classList.remove("hidden");
});
actualizarContador();
cargarFotos();
}

function ejecutar(){
const c=document.getElementById("comando").value.trim().toLowerCase();
const respuestas={
te_quiero:"Proceso confirmado: te quiero muchísimo. ♡",
felicidad:"Tu existencia es una de mis cosas favoritas.",
secreto:"Psst... eres una de mis coincidencias favoritas.",
nuestros_recuerdos:"Abriendo MEMORY.exe...",
amor:"Error 404: no existen palabras suficientes para explicar cuánto te quiero.",
desbloquear:"No necesitas una contraseña para saber que eres especial."
};
document.getElementById("respuesta").textContent=
respuestas[c]||"Comando no reconocido. Intenta de nuevo ♡";
}

function actualizarContador(){
const inicio=new Date(2026,8,22);
const dias=Math.max(0,Math.floor((new Date()-inicio)/86400000));
document.getElementById("tiempo").textContent=dias+" días creando recuerdos juntas ♡";
}

async function entrar(){
const email=document.getElementById("correo").value.trim();
const password=document.getElementById("clave").value;
const msg=document.getElementById("mensaje");
msg.textContent="Verificando...";
const {error}=await db.auth.signInWithPassword({email,password});
if(error){msg.textContent="No se pudo iniciar sesión. Revisa tus datos.";return;}
administradora=true;
document.getElementById("acceso").classList.add("hidden");
document.getElementById("editor").classList.remove("hidden");
cargarFotos();
}

async function salir(){
await db.auth.signOut();
administradora=false;
document.getElementById("editor").classList.add("hidden");
document.getElementById("acceso").classList.remove("hidden");
cargarFotos();
}

function codificar(s){
return encodeURIComponent(s.trim()).replaceAll("_","%5F");
}
function escapar(s){
return s.replace(/[&<>"']/g,c=>({"&":"&amp;","<":"&lt;",">":"&gt;",'"':"&quot;","'":"&#39;"}[c]));
}

async function cargarFotos(){
const estado=document.getElementById("estado");
const galeria=document.getElementById("galeria");
estado.textContent="Cargando recuerdos...";
galeria.innerHTML="";
const {data,error}=await db.storage.from(BUCKET).list("",{
limit:100,sortBy:{column:"created_at",order:"desc"}
});
if(error){estado.textContent="No se pudieron cargar los recuerdos.";return;}
if(!data||data.length===0){estado.textContent="Todavía no hay recuerdos. ♡";return;}
estado.textContent="Nuestros recuerdos ♡";
data.filter(f=>f.name&&!f.name.startsWith(".")).forEach(f=>{
const partes=f.name.split("__");
const titulo=partes.length>1?decodeURIComponent(partes[1]):"Un recuerdo";
const descripcion=partes.length>2?decodeURIComponent(partes[2].replace(/\.[^.]+$/,"")):"";
const {data:urlData}=db.storage.from(BUCKET).getPublicUrl(f.name);
const tarjeta=document.createElement("div");
tarjeta.className="foto";
tarjeta.innerHTML=`<img src="${escapar(urlData.publicUrl)}" alt="Recuerdo"><p><b>${escapar(titulo)}</b></p><p>${escapar(descripcion)}</p>`;
if(administradora){
const borrar=document.createElement("button");
borrar.textContent="ELIMINAR";
borrar.onclick=()=>eliminarFoto(f.name);
tarjeta.appendChild(borrar);
}
galeria.appendChild(tarjeta);
});
}

async function subirFoto(){
const archivo=document.getElementById("archivo").files[0];
const titulo=document.getElementById("titulo").value||"Un recuerdo";
const descripcion=document.getElementById("descripcion").value||"";
const resultado=document.getElementById("resultado");
if(!archivo){resultado.textContent="Selecciona una fotografía.";return;}
if(!archivo.type.startsWith("image/")){resultado.textContent="Selecciona una imagen.";return;}
if(archivo.size>5*1024*1024){resultado.textContent="La imagen debe pesar menos de 5 MB.";return;}
const extension=archivo.name.split(".").pop().toLowerCase().replace(/[^a-z0-9]/g,"")||"jpg";
const nombre=Date.now()+"__"+codificar(titulo)+"__"+codificar(descripcion)+"."+extension;
resultado.textContent="Subiendo recuerdo...";
const {error}=await db.storage.from(BUCKET).upload(nombre,archivo,{upsert:false});
if(error){resultado.textContent="No se pudo guardar. Revisa los permisos.";return;}
resultado.textContent="¡Recuerdo guardado! ♡";
document.getElementById("archivo").value="";
document.getElementById("titulo").value="";
document.getElementById("descripcion").value="";
cargarFotos();
}

async function eliminarFoto(nombre){
if(!confirm("¿Quieres eliminar este recuerdo?"))return;
const {error}=await db.storage.from(BUCKET).remove([nombre]);
if(error){alert("No se pudo eliminar.");return;}
cargarFotos();
}

db.auth.getSession().then(({data})=>{
if(data.session){
administradora=true;
document.getElementById("acceso").classList.add("hidden");
document.getElementById("editor").classList.remove("hidden");
}
});
</script>
</body>
</html>
