<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Gestor De Clientes</title>

<style>
body{font-family:Inter,Arial;background:#020617;color:white;margin:0}
header{padding:16px;background:#0f172a;text-align:center;font-size:22px;font-weight:700}

.container{padding:15px;max-width:800px;margin:auto}

input,button,textarea{
  padding:12px;border-radius:12px;border:none;width:100%;margin-top:8px
}
input,textarea{background:#1e293b;color:white}

button{background:#22c55e;color:white;font-weight:bold}

.card{
  background:#0f172a;
  padding:16px;
  border-radius:16px;
  margin-top:12px;
  box-shadow:0 10px 25px rgba(0,0,0,.4)
}

.nombre{font-size:18px;font-weight:bold}

.actions{display:flex;gap:8px;margin-top:10px}
.actions button{flex:1}

.btn-wsp{background:#22c55e}
.btn-edit{background:#334155}
.btn-del{background:#ef4444}

.fab{
  position:fixed;bottom:20px;right:20px;
  background:#22c55e;width:60px;height:60px;
  border-radius:50%;display:flex;align-items:center;
  justify-content:center;font-size:30px;cursor:pointer
}

.modal{
  display:none;position:fixed;inset:0;
  background:rgba(0,0,0,.7);
  justify-content:center;align-items:center
}

.modal-content{
  background:#0f172a;padding:20px;
  border-radius:16px;width:90%;max-width:400px
}
</style>
</head>
<body>

<header>Gestor De Clientes</header>

<div class="container">
<input id="buscador" placeholder="Buscar..." onkeyup="mostrar()">
<button onclick="abrirImportador()">📥 Importar mensaje</button>
<div id="lista"></div>
</div>

<div class="fab" onclick="abrirModal()">+</div>

<!-- MODAL -->
<div class="modal" id="modal">
<div class="modal-content">
<input id="nombre" placeholder="Nombre">
<input id="telefono" placeholder="Teléfono">
<input id="direccion" placeholder="Dirección">
<button onclick="guardar()">Guardar</button>
<button onclick="cerrarModal()">Cerrar</button>
</div>
</div>

<!-- IMPORTADOR -->
<div class="modal" id="importador">
<div class="modal-content">
<textarea id="textoImportar" placeholder="Pegá mensaje"></textarea>
<button onclick="procesarMensaje()">Procesar</button>
<button onclick="cerrarImportador()">Cerrar</button>
</div>
</div>

<script>
let contactos = JSON.parse(localStorage.getItem("crm")) || [];
let editIndex=null;

function abrirModal(){modal.style.display="flex"}
function cerrarModal(){modal.style.display="none"}
function abrirImportador(){importador.style.display="flex"}
function cerrarImportador(){importador.style.display="none"}

function guardar(){
 let n=nombre.value.trim();
 let t=telefono.value.trim();
 let d=direccion.value.trim();

 if(!n||!t)return alert("Completa datos");

 let data={nombre:n,telefono:t,direccion:d,fecha:new Date().toLocaleDateString()};

 if(editIndex!==null){contactos[editIndex]=data}else{contactos.push(data)}

 localStorage.setItem("crm",JSON.stringify(contactos));
 cerrarModal();
 mostrar();
}

function editar(i){
 let c=contactos[i];
 nombre.value=c.nombre;
 telefono.value=c.telefono;
 direccion.value=c.direccion;
 editIndex=i;
 abrirModal();
}

function eliminar(i){
 if(!confirm("Eliminar cliente?"))return;
 contactos.splice(i,1);
 localStorage.setItem("crm",JSON.stringify(contactos));
 mostrar();
}

function abrirWhatsApp(num){
 let limpio=num.replace(/[^0-9]/g,"");
 window.location.href=`https://wa.me/${limpio}`;
}

// 🔥 IMPORTADOR PRO
function procesarMensaje(){
 let texto=textoImportar.value.toLowerCase();

 // cortar cuando empieza otra cosa
 let limpioTexto=texto.split(/(mi numero|mi número|telefono|teléfono|y mi)/)[0];

 // numero
 let numeroMatch=texto.match(/\d{7,15}/);
 let numero=numeroMatch?numeroMatch[0]:"";

 // nombre inteligente
 let nombre="Cliente";

 let n1=texto.match(/mi nombre es ([a-záéíóúñ]+\s*[a-záéíóúñ]*)/i);
 let n2=texto.match(/soy ([a-záéíóúñ]+\s*[a-záéíóúñ]*)/i);

 if(n1){nombre=capitalizar(n1[1])}
 else if(n2){nombre=capitalizar(n2[1])}

 // direccion inteligente
 let direccion="";

 let m1=limpioTexto.match(/([a-záéíóúñ]+\s*[a-záéíóúñ]*\s*\d+\s*(y|entre)?\s*\d*)/);
 let m2=limpioTexto.match(/(\d+\s*(y|entre)\s*\d+)/);

 if(m1){direccion=m1[0]}
 else if(m2){direccion=m2[0]}

 contactos.push({
  nombre:nombre,
  telefono:numero,
  direccion:direccion,
  fecha:new Date().toLocaleDateString()
 });

 localStorage.setItem("crm",JSON.stringify(contactos));
 cerrarImportador();
 mostrar();
}

function capitalizar(t){
 return t.split(" ").map(p=>p.charAt(0).toUpperCase()+p.slice(1)).join(" ");
}

function mostrar(){
 lista.innerHTML="";
 let filtro=buscador.value.toLowerCase();

 contactos.forEach((c,i)=>{
  if(
   c.nombre.toLowerCase().includes(filtro) ||
   c.telefono.includes(filtro) ||
   (c.direccion||"").toLowerCase().includes(filtro)
  ){
   lista.innerHTML+=`
   <div class="card">
    <div class="nombre">${c.nombre}</div>
    📞 ${c.telefono}<br>
    📍 ${c.direccion || "-"}<br>
    📅 ${c.fecha}

    <div class="actions">
     <button class="btn-wsp" onclick="abrirWhatsApp('${c.telefono}')">💬</button>
     <button class="btn-edit" onclick="editar(${i})">✏️</button>
     <button class="btn-del" onclick="eliminar(${i})">🗑️</button>
    </div>
   </div>`;
  }
 });
}

mostrar();
</script>

</body>
</html>
