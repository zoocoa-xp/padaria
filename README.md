<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Padaria</title>
<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js"></script><style>
body{
  font-family: 'Segoe UI', sans-serif;
  margin:0;
  background:#fff8f0;
}

header{
  background: linear-gradient(135deg,#ff8c00,#ffb347);
  color:white;
  padding:20px;
  text-align:center;
}

.container{
  padding:20px;
  max-width:1000px;
  margin:auto;
}

input{
  padding:10px;
  margin:5px;
  border-radius:6px;
  border:1px solid #ccc;
}

button{
  padding:10px 15px;
  border:none;
  border-radius:6px;
  background:#ff8c00;
  color:white;
  cursor:pointer;
  transition:0.3s;
}

button:hover{
  background:#e67600;
}

.grid{
  display:grid;
  grid-template-columns:repeat(auto-fill,minmax(200px,1fr));
  gap:15px;
}

.card{
  background:white;
  padding:15px;
  border-radius:12px;
  box-shadow:0 4px 10px rgba(0,0,0,0.1);
  transition:0.2s;
}

.card:hover{
  transform:scale(1.03);
}

.price{
  color:#ff8c00;
  font-weight:bold;
}

.cart-box{
  background:white;
  padding:15px;
  border-radius:12px;
  box-shadow:0 4px 10px rgba(0,0,0,0.1);
  margin-top:20px;
}

.total{
  font-weight:bold;
  margin-top:10px;
}

</style></head>
<body><header>
<h1>🥖 Padaria Online</h1>
<p>Produtos fresquinhos todos os dias</p>
</header><div class="container"><!-- LOGIN --><input id="email" placeholder="Email">
<input id="senha" type="password" placeholder="Senha">
<button onclick="login()">Entrar</button>
<button onclick="register()">Cadastrar</button><h2>🍞 Produtos</h2>
<div id="produtos" class="grid"></div><!-- CARRINHO DENTRO DA PÁGINA --><div class="cart-box">
<h3>🛒 Carrinho</h3>
<ul id="cartItems"></ul>
<div class="total">Total: R$ <span id="total">0</span></div>
<button onclick="finalizar()">Finalizar Pedido</button>
</div></div><script>
const { createClient } = supabase;

const supabaseClient = createClient(
  'https://xxxx.supabase.co',
  'SUA_KEY_AQUI'
);

let carrinho = [];

// LOGIN
async function login(){
  const emailVal = document.getElementById('email').value;
  const senhaVal = document.getElementById('senha').value;

  const { error } = await supabaseClient.auth.signInWithPassword({
    email: emailVal,
    password: senhaVal
  });

  if(error){
    alert(error.message);
  } else {
    alert('Logado!');
  }
}

// CADASTRO
async function register(){
  const emailVal = document.getElementById('email').value;
  const senhaVal = document.getElementById('senha').value;

  const { error } = await supabaseClient.auth.signUp({
    email: emailVal,
    password: senhaVal
  });

  if(error){
    alert(error.message);
  } else {
    alert('Cadastrado!');
  }
}

// PRODUTOS
async function carregarProdutos(){
  const { data, error } = await supabaseClient.from('produtos').select('*');

  if(error){
    alert(error.message);
    return;
  }

  const produtosDiv = document.getElementById('produtos');
  produtosDiv.innerHTML='';

  data.forEach(p=>{
    produtosDiv.innerHTML += `
      <div class="card">
        <h3>${p.nome}</h3>
        <p class="price">R$ ${p.preco}</p>
        <button onclick="add('${p.id}','${p.nome}',${p.preco})">Comprar</button>
      </div>
    `;
  });
}

// CARRINHO
function add(id,nome,preco){
  carrinho.push({id,nome,preco});
  renderCart();
}

function renderCart(){
  const cartItems = document.getElementById('cartItems');
  const totalSpan = document.getElementById('total');

  cartItems.innerHTML='';
  let total=0;

  carrinho.forEach(i=>{
    total+=i.preco;
    cartItems.innerHTML += `<li>${i.nome}</li>`;
  });

  totalSpan.textContent = total;
}

// FINALIZAR
async function finalizar(){
  const { data: userData } = await supabaseClient.auth.getUser();
  const user = userData.user;

  if(!user){
    alert('Faça login primeiro!');
    return;
  }

  const total = document.getElementById('total').textContent;

  const { data: pedido } = await supabaseClient
    .from('pedidos')
    .insert({
      usuario_id:user.id,
      total: total
    })
    .select()
    .single();

  for(let item of carrinho){
    await supabaseClient.from('itens_pedido').insert({
      pedido_id:pedido.id,
      produto_id:item.id,
      preco_unitario:item.preco
    });
  }

  alert('Pedido feito!');
  carrinho=[];
  renderCart();
}

// INIT
carregarProdutos();
</script></body>
