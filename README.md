<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Flipkart-like Store — Demo</title>
  <style>
    :root{--brand:#2874f0;--muted:#666;--card:#fff}
    *{box-sizing:border-box}
    body{font-family:Inter,system-ui,Segoe UI,Roboto,'Poppins',sans-serif;margin:0;background:#f2f4f7;color:#111}
    header{background:linear-gradient(90deg,var(--brand),#1e60d1);color:#fff;padding:12px 18px;display:flex;align-items:center;gap:12px}
    .logo{font-weight:700;font-size:20px}
    .search{flex:1;display:flex}
    .search input{flex:1;padding:10px;border-radius:4px 0 0 4px;border:none}
    .search button{padding:10px 14px;border:none;border-radius:0 4px 4px 0;background:#fff;color:var(--brand);font-weight:700;cursor:pointer}
    .nav-actions{display:flex;gap:10px;align-items:center}
    .container{max-width:1200px;margin:16px auto;padding:0 16px}
    .layout{display:grid;grid-template-columns:260px 1fr;gap:18px}
    .sidebar{background:var(--card);padding:12px;border-radius:8px;box-shadow:0 6px 18px rgba(0,0,0,0.04)}
    .filters h4{margin:8px 0}
    .category{padding:8px;border-radius:6px;cursor:pointer}
    .category.active{background:#e8f1ff;color:var(--brand)}
    .main{display:flex;flex-direction:column;gap:12px}
    .sort-row{display:flex;justify-content:space-between;align-items:center}
    .grid{display:grid;grid-template-columns:repeat(auto-fill,minmax(200px,1fr));gap:14px}
    .card{background:var(--card);padding:10px;border-radius:8px;box-shadow:0 6px 18px rgba(0,0,0,0.04)}
    .card img{width:100%;height:180px;object-fit:cover;border-radius:6px}
    .card h3{margin:8px 0 6px;font-size:15px}
    .price{color:var(--brand);font-weight:700}
    .rating{font-size:13px;color:#ffb400}
    .btn{background:var(--brand);color:white;padding:8px 10px;border:none;border-radius:6px;cursor:pointer}
    .btn.secondary{background:#fff;color:var(--brand);border:1px solid rgba(40,116,240,0.12)}
    .cart-drawer{position:fixed;right:18px;top:80px;width:360px;background:var(--card);padding:12px;border-radius:10px;box-shadow:0 20px 50px rgba(0,0,0,0.2);max-height:80vh;overflow:auto;display:none}
    .cart-item{display:flex;gap:10px;align-items:center;padding:8px 0;border-bottom:1px solid #eef2f8}
    .cart-item img{width:64px;height:64px;object-fit:cover;border-radius:6px}
    .footer{padding:18px;text-align:center;color:var(--muted)}
    .admin-panel{margin-top:12px;background:#fff;padding:10px;border-radius:8px}
    @media (max-width:900px){.layout{grid-template-columns:1fr}.sidebar{order:2}.cart-drawer{right:12px;left:12px;width:auto}}
  </style>
</head>
<body>
  <header>
    <div class="logo">PrabinKart</div>
    <div class="search">
      <input id="search-input" placeholder="Search for products, brands and more" />
      <button id="search-btn">Search</button>
    </div>
    <div class="nav-actions">
      <button id="open-cart" class="btn">Cart (<span id="cart-count">0</span>)</button>
      <button id="admin-mode" class="btn secondary">Admin</button>
    </div>
  </header>

  <div class="container">
    <div class="layout">
      <aside class="sidebar">
        <h3>Categories</h3>
        <div id="categories"></div>
        <div style="margin-top:12px" class="filters">
          <h4>Filters</h4>
          <label>Price under ₹<input id="price-filter" type="number" placeholder="e.g. 1000" style="width:120px;margin-left:6px"/></label>
          <h4 style="margin-top:8px">Sort</h4>
          <select id="sort-select" style="width:100%"><option value="new">Newest</option><option value="low">Price: Low to High</option><option value="high">Price: High to Low</option><option value="rating">Top Rated</option></select>
        </div>
        <div class="admin-panel" id="admin-panel" style="display:none">
          <h4>Admin: Add Product</h4>
          <input id="adm-title" placeholder="Title"/><br/><br/>
          <input id="adm-price" placeholder="Price" type="number"/><br/><br/>
          <textarea id="adm-desc" placeholder="Description"></textarea><br/><br/>
          <input id="adm-photo" type="file" accept="image/*"/><br/><br/>
          <input id="adm-cat" placeholder="Category"/><br/><br/>
          <button id="adm-add" class="btn secondary">Add Product</button>
        </div>
      </aside>

      <section class="main">
        <div class="sort-row">
          <div><strong id="result-count">0</strong> results</div>
          <div>
            <small>Showing products</small>
          </div>
        </div>

        <div class="grid" id="product-grid"></div>

        <div id="no-products" style="text-align:center;color:var(--muted);display:none;padding:20px">No products found.</div>
      </section>
    </div>
  </div>

  <div class="cart-drawer" id="cart-drawer">
    <h3>Your Cart</h3>
    <div id="cart-items"></div>
    <div style="margin-top:12px;display:flex;justify-content:space-between;align-items:center"><strong>Total: ₹<span id="cart-total">0</span></strong><button id="checkout-btn" class="btn">Place Order (COD)</button></div>
  </div>

  <footer class="footer">Flipkart-like demo • Client-side only • Data saved in browser (localStorage)</footer>

<script>
// Flipkart-like front-end demo (client-side only). Save this file as index.html.
let products = JSON.parse(localStorage.getItem('fk_products')||'[]');
let cart = JSON.parse(localStorage.getItem('fk_cart')||'[]');
const orders = JSON.parse(localStorage.getItem('fk_orders')||'[]');
const ADMIN_PASS = 'admin123';
let isAdmin=false;

// utilities
function uid(len=8){return Math.random().toString(36).slice(2,2+len)}
function save(){localStorage.setItem('fk_products',JSON.stringify(products));localStorage.setItem('fk_cart',JSON.stringify(cart));}
function escape(s){return (s||'').replaceAll('&','&amp;').replaceAll('<','&lt;').replaceAll('>','&gt;')}

// render categories
function renderCategories(){const out=document.getElementById('categories'); out.innerHTML=''; const cats = [...new Set(products.map(p=>p.category||'General'))]; if(cats.length===0){out.innerHTML='<div style="color:var(--muted)">No categories</div>';return;} cats.forEach(c=>{const d=document.createElement('div'); d.className='category'; d.textContent=c; d.onclick=()=>{document.querySelectorAll('.category').forEach(x=>x.classList.remove('active')); d.classList.add('active'); applyFilters();}; out.appendChild(d)})}

// render products
function renderProducts(list){const grid=document.getElementById('product-grid'); grid.innerHTML=''; const arr = list||products; if(arr.length===0){document.getElementById('no-products').style.display='block';} else {document.getElementById('no-products').style.display='none';}
  arr.forEach(p=>{const el=document.createElement('div'); el.className='card'; el.innerHTML = `
    <img src='${p.photo}' alt=''>
    <h3>${escape(p.title)}</h3>
    <div style='display:flex;justify-content:space-between;align-items:center'><div><div class='price'>₹${p.price}</div><div class='rating'>${renderStars(avg(p))} ${p.reviews && p.reviews.length? '('+p.reviews.length+')':''}</div></div><div><button class='btn secondary' onclick="viewProduct('${p.id}')">View</button> <button class='btn' onclick="addToCart('${p.id}')">Add</button></div></div>
  `; grid.appendChild(el); });
  document.getElementById('result-count').textContent = (arr||[]).length;
}
function renderStars(n){ if(!n) return '☆☆☆☆☆'; let full=Math.round(n); return '★'.repeat(full)+ '☆'.repeat(5-full); }
function avg(p){ if(!p.reviews||p.reviews.length===0) return 0; return p.reviews.reduce((s,r)=>s+r.rating,0)/p.reviews.length }

// filters
function applyFilters(){let list = [...products]; const q = document.getElementById('search-input').value.trim().toLowerCase(); if(q) list = list.filter(p=>p.title.toLowerCase().includes(q)||p.description.toLowerCase().includes(q)); const price = parseFloat(document.getElementById('price-filter').value||0); if(price>0) list = list.filter(p=>parseFloat(p.price)<=price);
  const activeCat = document.querySelector('.category.active'); if(activeCat) list = list.filter(p=>p.category===activeCat.textContent);
  const sort = document.getElementById('sort-select').value; if(sort==='low') list.sort((a,b)=>a.price-b.price); if(sort==='high') list.sort((a,b)=>b.price-a.price); if(sort==='rating') list.sort((a,b)=>avg(b)-avg(a)); if(sort==='new') list.sort((a,b)=> (b.created||0)-(a.created||0));
  renderProducts(list);
}

// cart
function updateCartUI(){document.getElementById('cart-count').textContent = cart.reduce((s,i)=>s+i.qty,0); const el=document.getElementById('cart-items'); el.innerHTML=''; let total=0; cart.forEach(ci=>{const p = products.find(x=>x.id===ci.id); if(!p) return; const d=document.createElement('div'); d.className='cart-item'; d.innerHTML = `<img src='${p.photo}'/><div style='flex:1'><strong>${escape(p.title)}</strong><div style='color:var(--muted);font-size:13px'>₹${p.price} • Qty: ${ci.qty}</div></div><div><div>₹${p.price*ci.qty}</div><div style='margin-top:6px'><button class='btn secondary' onclick="changeQty('${ci.id}','-')">-</button> <button class='btn' onclick="changeQty('${ci.id}','+')">+</button></div></div>`; el.appendChild(d); total+=p.price*ci.qty; }); document.getElementById('cart-total').textContent = total; save(); }
function addToCart(id){const ex=cart.find(c=>c.id===id); if(ex) ex.qty++; else cart.push({id,qty:1}); updateCartUI(); alert('Added to cart');}
function changeQty(id,op){const idx=cart.findIndex(c=>c.id===id); if(idx<0) return; if(op==='+') cart[idx].qty++; else {cart[idx].qty--; if(cart[idx].qty<=0) cart.splice(idx,1);} updateCartUI(); }

// product view
function viewProduct(id){const p = products.find(x=>x.id===id); if(!p) return; const modal = document.createElement('div'); modal.style.position='fixed'; modal.style.inset=0; modal.style.background='rgba(0,0,0,0.5)'; modal.style.display='flex'; modal.style.alignItems='center'; modal.style.justifyContent='center'; modal.innerHTML = `<div style='background:#fff;border-radius:8px;padding:18px;max-width:900px;width:100%;max-height:90vh;overflow:auto'><div style='display:flex;gap:12px;flex-wrap:wrap'><div style='flex:1 1 320px'><img src='${p.photo}' style='width:100%;height:360px;object-fit:cover;border-radius:6px'/></div><div style='flex:1 1 320px'><h2>${escape(p.title)}</h2><div style='color:var(--muted)'>₹${p.price}</div><p style='color:var(--muted)'>${escape(p.description)}</p><div style='margin-top:8px'><button class='btn' onclick="addToCart('${p.id}')">Add to Cart</button> <button class='btn secondary' id='close-modal'>Close</button></div><hr><h4>Reviews</h4><div id='rv-area'></div><h4>Add Review</h4><input id='rv-name' placeholder='Name' style='width:100%;padding:8px;margin-bottom:6px'/><input id='rv-rating' type='number' min='1' max='5' value='5' style='width:100%;padding:8px;margin-bottom:6px'/><textarea id='rv-comment' style='width:100%;padding:8px;margin-bottom:6px'></textarea><button id='rv-submit' class='btn'>Submit</button></div></div></div>`;
  document.body.appendChild(modal);
  modal.querySelector('#close-modal').addEventListener('click',()=>modal.remove()); renderReviews(p, modal.querySelector('#rv-area'));
  modal.querySelector('#rv-submit').addEventListener('click',()=>{const name=modal.querySelector('#rv-name').value||'Anonymous'; const rating=parseInt(modal.querySelector('#rv-rating').value)||5; const comment=modal.querySelector('#rv-comment').value||''; p.reviews=p.reviews||[]; p.reviews.push({name,rating,comment}); save(); renderReviews(p, modal.querySelector('#rv-area')); alert('Review added');}); }
function renderReviews(p,area){ area.innerHTML=''; if(!p.reviews||p.reviews.length===0) area.innerHTML='<div style="color:var(--muted)">No reviews</div>'; else p.reviews.slice().reverse().forEach(r=>{const d=document.createElement('div'); d.style.padding='6px 0'; d.innerHTML=`<strong>${escape(r.name)}</strong> <div style='color:#ffb400'>${'★'.repeat(r.rating)}${'☆'.repeat(5-r.rating)}</div><div style='color:var(--muted);font-size:13px'>${escape(r.comment)}</div>`; area.appendChild(d)}); }

// admin
function enableAdmin(){const p = prompt('Enter admin password'); if(p===ADMIN_PASS){isAdmin=true; document.getElementById('admin-panel').style.display='block'; alert('Admin enabled');} else alert('Wrong password'); }
async function addAdminProduct(){const title=document.getElementById('adm-title').value.trim(); const price=parseFloat(document.getElementById('adm-price').value); const desc=document.getElementById('adm-desc').value.trim(); const cat=document.getElementById('adm-cat').value.trim()||'General'; const file=document.getElementById('adm-photo').files[0]; if(!title||!price||!file) return alert('Provide title, price and photo'); const data = await fileToDataUrl(file); const p={id:uid(8),title,price,description:desc,photo:data,category:cat,created:Date.now(),reviews:[]}; products.unshift(p); save(); renderCategories(); applyFilters(); document.getElementById('adm-title').value='';document.getElementById('adm-price').value='';document.getElementById('adm-desc').value='';document.getElementById('adm-photo').value='';document.getElementById('adm-cat').value=''; alert('Product added'); }
function fileToDataUrl(file){return new Promise((res,rej)=>{const r=new FileReader(); r.onload=()=>res(r.result); r.onerror=()=>rej(); r.readAsDataURL(file);})}

// checkout (COD)
function checkout(){if(cart.length===0) return alert('Cart empty'); const name = prompt('Enter your name'); if(!name) return; const phone = prompt('Enter phone number'); if(!phone) return; const address = prompt('Enter address'); if(!address) return; const order = {id:'ORD-'+uid(6).toUpperCase(),name,phone,address,items:cart.slice(),total:cart.reduce((s,i)=>{const p=products.find(x=>x.id===i.id); return s + (p? p.price*i.qty:0)},0),method:'COD',created:Date.now(),status:'Pending'}; const all = JSON.parse(localStorage.getItem('fk_orders')||'[]'); all.unshift(order); localStorage.setItem('fk_orders',JSON.stringify(all)); cart=[]; save(); updateCartUI(); alert('Order placed. ID: '+order.id+' • Pay on delivery'); }

// init
function init(){ if(products.length===0){ // add demo products
  products = [
    {id:uid(6),title:'Men T-shirt',price:499,description:'Comfort cotton tee',photo:'https://placehold.co/600x400?text=T-shirt',category:'Clothing',created:Date.now(),reviews:[]},
    {id:uid(6),title:'Wireless Earbuds',price:1499,description:'Bluetooth earphones with mic',photo:'https://placehold.co/600x400?text=Earbuds',category:'Electronics',created:Date.now(),reviews:[]},
    {id:uid(6),title:'Backpack',price:899,description:'Durable travel backpack',photo:'https://placehold.co/600x400?text=Backpack',category:'Bags',created:Date.now(),reviews:[]}
  ]; save(); }
 renderCategories(); applyFilters(); updateCartUI(); }

// events
document.getElementById('search-btn').addEventListener('click',applyFilters);
document.getElementById('search-input').addEventListener('input',applyFilters);
document.getElementById('price-filter').addEventListener('input',applyFilters);
document.getElementById('sort-select').addEventListener('change',applyFilters);
document.getElementById('open-cart').addEventListener('click',()=>{const drawer=document.getElementById('cart-drawer'); drawer.style.display = drawer.style.display==='block'?'none':'block'});
document.getElementById('admin-mode').addEventListener('click',enableAdmin);
document.getElementById('adm-add').addEventListener('click',addAdminProduct);
document.getElementById('checkout-btn').addEventListener('click',checkout);

init();
</script>
</body>
</html>
# Daskart.github.io
