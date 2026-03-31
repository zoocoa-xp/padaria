<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Padaria</title>

<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js"></script>

<style>
body{font-family:Arial}
.card{border:1px solid #ccc;margin:10px;padding:10px}
#cart{position:fixed;right:0;top:0;width:250px;background:#eee;height:100%}
</style>

</head>
<body>

<h1>Padaria Online</h1>

<!-- LOGIN -->
<input id="email" placeholder="Email">
<input id="senha" type="password" placeholder="Senha">
<button onclick="login()">Login</button>
<button onclick="register()">Cadastrar</button>

<h2>Produtos</h2>
<div id="produtos"></div>

<div id="cart">
<h3>Carrinho</h3>
<ul id="cartItems"></ul>
<p>Total: R$ <span id="total">0</span></p>
<button onclick="finalizar()">Finalizar</button>
</div>

<script>
// 🔴 COLE AQUI SUAS CHAVES
const supabaseClient = supabase.createClient(
  'SUA_URL_AQUI',
  'SUA_KEY_AQUI'
);

let carrinho = [];

// LOGIN
async function login(){
  await supabaseClient.auth.signInWithPassword({
    email: email.value,
    password: senha.value
  });
  alert('Logado');
}

// CADASTRO
async function register(){
  await supabaseClient.auth.signUp({
    email: email.value,
    password: senha.value
  });
  alert('Cadastrado');
}

// CARREGAR PRODUTOS
async function carregarProdutos(){
  const {data} = await supabaseClient.from('produtos').select('*');

  produtos.innerHTML='';
  data.forEach(p=>{
    produtos.innerHTML += `
      <div class="card">
        ${p.nome} - R$ ${p.preco}
        <button onclick="add('${p.id}','${p.nome}',${p.preco})">Comprar</button>
      </div>
    `;
  });
}

// ADD CARRINHO
function add(id,nome,preco){
  carrinho.push({id,nome,preco});
  renderCart();
}

// MOSTRAR CARRINHO
function renderCart(){
  cartItems.innerHTML='';
  let total=0;

  carrinho.forEach(i=>{
    total+=i.preco;
    cartItems.innerHTML += `<li>${i.nome}</li>`;
  });

  document.getElementById('total').textContent = total;
}

// FINALIZAR PEDIDO
async function finalizar(){
  const user = (await supabaseClient.auth.getUser()).data.user;

  const {data:pedido} = await supabaseClient
    .from('pedidos')
    .insert({
      usuario_id:user.id,
      total: document.getElementById('total').textContent
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

carregarProdutos();
</script>

</body>
</html>
